# MEDDPICC — The Qualified Deal

**The MEDDPICC / MEDDICC Enterprise-Sales Qualification Methodology — the Verified Lineage (MEDDIC at PTC in the 1990s, Dunkel, Napoli and McMahon, the contested coinage, and the Andy Whyte / MEDDICC-company conflation), the Components (Metrics, Economic Buyer, Decision Criteria, Decision Process, Identify Pain, Champion, Competition, Paper Process — and the Implications-of-pain reading of the I), the Practice (Evidence-Gated Deal Reviews, Scorecards, MEDDPICC in the CRM, Forecast Discipline), the Complementary Methodologies (BANT, SPIN Selling, The Challenger Sale), and the Banking-Wholesale Application (RFP-Driven Corporate and Institutional Deals) with a Cymbal Bank Transaction-Banking Worked Example — Verified Against Primary Sources Wherever Possible, Flagged ⚠ Honestly Where Not**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Management / Industry Research — the dedicated deep-dive on the MEDDPICC family of sales-qualification methodologies: what the framework is and the qualification problem it solves (§1), the verified origin story at PTC (Parametric Technology Corporation) in the mid-1990s and the practitioner cast — Dick Dunkel, Jack Napoli, John McMahon — plus the conflicting accounts of who coined what (§2), the MEDDICC and MEDDPICC expansions and the contested attribution of the added letters, including the April 2026 US federal-court ruling that MEDDPICC is a generic term (§3), how the methodology spread through sales-training and enablement vendors — MEDDICC (the company), Force Management, Winning by Design, SalesHacker, Gong — with vendor marketing flagged as such (§4), the full component map, letter by letter, verified against the methodology sites and practitioner accounts (§5), the component interplays that close deals (§6), the qualification discipline — deal reviews, scorecards, CRM practice, forecasting — with the common failure modes (§7–§9), the complementary methodologies BANT, SPIN and Challenger with a comparison table (§10), the condensed banking-wholesale application (§11), and a full Cymbal Bank worked example: qualifying a regional corporate group's transaction-banking RFP letter by letter (§12). §13 is the claims audit, §14 "What Could Not Be Verified", §15 the glossary, sources and closing summary. The repository's general multi-framework survey — [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) — covers BANT/MEDDIC/SPIN/Challenger/Sandler at survey depth; this guide is the dedicated MEDDPICC deep-dive and deliberately goes deeper rather than duplicating it.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** meddicc.com (the MEDDICC company's "Who Created MEDDIC?" history article (13 Aug 2026), the "Backstory of Andy Whyte" (7 Apr 2026), the MEDDPICC-explained hub and the trademark-ruling release (28 May 2026)), salesmeddic.com (Monica Evans's first-person origin account, 11 Mar), revcentricpartners.com/about (Dick Dunkel and David Boyle's coaching firm), meddicc.com's Masters of MEDDICC podcast (Dunkel interview), forcemanagement.com (the Force Management MEDDICC offering page with client testimonials), winningbydesign.com (the SaaS/RevOps consulting firm), huthwaiteinternational.com (Neil Rackham's own account of the SPIN research origins, 6 Feb 2026), Wikipedia/Google Books/PR Newswire (The Challenger Sale publication and CEB research base), and practitioner blogs (Highspot, MeetGeek, closingfoundry.com, praiz.io, sellible.ai, prospeo.io). NOTE: this pass had **live web access** — the facts below were verified against the primary sources on 2026-09-03; claims that could only be checked against vendor marketing pages, single accounts, or memory are flagged ⚠ or ⚠-knowledge honestly. No dates, names or claims were invented; where accounts conflict, the conflict is described rather than smoothed over.
> **Last Updated:** September 2026
> **Companion guides (management/, same folder):** [Vendor Management Guide](vendor_management_guide.md) (the RFP/sourcing and selection process — §3 of that guide — which this guide's banking section cross-references rather than re-derives) · [Ancillary Revenue Products Guide](ancillary_revenue_products_guide.md) (sales-adjacent product economics and a Cymbal Bank sales-style worked example) · [Communication & Stakeholder Management Skills](communication_stakeholder_management_skills_guide.md) (stakeholder mapping and influencing without authority — the soft-skills layer under the champion/EB work) · [Facilitation Skills Guide](facilitation_skills_guide.md) (running the workshops and deal reviews that enforce the discipline) · [Business Case Development](business_case_development_guide.md) (building the quantified case behind the Metrics letter)
> **Companion guides (banking/, prefix `../banking/`):** [Payments Hub Guide](../banking/payments_hub_guide.md) (the transaction-banking product estate behind the worked example) · [Supply Chain Finance Technologies Guide](../banking/supply_chain_finance_technologies_guide.md) (trade and SCF products in the RFP scope)
> **Companion guides (technology/, prefix `../technology/`):** [Sales Methodology Frameworks Guide](../technology/sales_methodology_frameworks_guide.md) (the repo's general multi-framework survey of BANT, MEDDIC/MEDDPICC, SPIN, Challenger, Sandler — cross-reference §3–§6 of that guide for the survey view; this guide is the MEDDPICC deep-dive)

---

**How to use this guide:** §1 is the concept — the short answer, the qualification problem MEDDPICC exists to solve, and where a methodology sits relative to a sales process. §2 and §3 are the verified lineage: MEDDIC at PTC, the practitioner cast, the expansions to MEDDICC and MEDDPICC, the contested coinages, and the 2026 trademark ruling. §4 is the industry adoption story through the training and enablement vendors — read it with the marketing filter the section itself applies. §5 is the heart: the component map, every letter verified against primary sources, with the meaning, why it matters, the classic mistake, and variant differences for each, plus a variant table. §6 shows how the components chain together (champion → economic buyer → decision criteria; paper process as the champion's ammunition; pain → implications → metrics). §7–§9 are the practice: deal reviews, scorecards, the CRM, forecasting conventions, and the common failure modes. §10 covers the complementary methodologies — BANT, SPIN, The Challenger Sale — with a comparison table. §11 condenses the banking-wholesale application and cross-references the RFP and transaction-banking siblings. §12 walks a full Cymbal Bank transaction-banking RFP through every letter. §13 is the claims audit (✅/⚠/❌), §14 "What Could Not Be Verified", and §15 the glossary, the sources, and the closing. Cross-references follow the repository convention: sibling guides in `management/` are plain filenames, `banking/` guides are prefixed `../banking/`, `technology/` guides `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary source; ⚠ = flagged/approximate/unverified (vendor marketing or a single contested account); ⚠-knowledge = well-documented industry knowledge not re-verified this pass; ❌ = could not verify or conflicting sources.

---

**Table of Contents**

1. [The Concept — What MEDDPICC Is and Why It Exists](#1-the-concept--what-meddpicc-is-and-why-it-exists)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Qualification Problem](#12-the-qualification-problem)
   - 1.3 [Methodology vs Process — Where It Sits](#13-methodology-vs-process--where-it-sits)
   - 1.4 [What This Guide Adds](#14-what-this-guide-adds)
2. [The Lineage — MEDDIC at PTC](#2-the-lineage--meddic-at-ptc)
   - 2.1 [The Origin Story as the Primary Sources Tell It](#21-the-origin-story-as-the-primary-sources-tell-it)
   - 2.2 [The Cast — Dunkel, Napoli, McMahon, Evans](#22-the-cast--dunkel-napoli-mcmahon-evans)
   - 2.3 [The Conflicts — Who Coined What](#23-the-conflicts--who-coined-what)
   - 2.4 [Reading the Origin Story Critically](#24-reading-the-origin-story-critically)
   - 2.5 [The PTC Diaspora — How the Six Letters Left the Building](#25-the-ptc-diaspora--how-the-six-letters-left-the-building)
3. [The Expansions — MEDDICC and MEDDPICC](#3-the-expansions--meddicc-and-meddpicc)
   - 3.1 [Which Letter Each Variant Adds](#31-which-letter-each-variant-adds)
   - 3.2 [The Contested Coinage](#32-the-contested-coinage)
   - 3.3 [The 2026 Trademark Ruling — MEDDPICC Goes Generic](#33-the-2026-trademark-ruling--meddpicc-goes-generic)
   - 3.4 [The Wider Variant Zoo](#34-the-wider-variant-zoo)
4. [The Industry Adoption — the Training and Enablement Vendors](#4-the-industry-adoption--the-training-and-enablement-vendors)
   - 4.1 [MEDDICC the Company — and the McMahon Line](#41-meddicc-the-company--and-the-mcmahon-line)
   - 4.2 [Force Management](#42-force-management)
   - 4.3 [Winning by Design and the SaaS/RevOps Wave](#43-winning-by-design-and-the-saasrevops-wave)
   - 4.4 [The Evangelists — SalesHacker, Gong, Highspot](#44-the-evangelists--saleshacker-gong-highspot)
   - 4.5 [Where the Vendor Claims End and Verified Fact Begins](#45-where-the-vendor-claims-end-and-verified-fact-begins)
5. [The Component Map — Every Letter, Verified](#5-the-component-map--every-letter-verified)
   - 5.1 [Metrics](#51-metrics)
   - 5.2 [Economic Buyer](#52-economic-buyer)
   - 5.3 [Decision Criteria](#53-decision-criteria)
   - 5.4 [Decision Process](#54-decision-process)
   - 5.5 [Identify Pain — and the Implications of Pain](#55-identify-pain--and-the-implications-of-pain)
   - 5.6 [Champion](#56-champion)
   - 5.7 [Competition](#57-competition)
   - 5.8 [Paper Process](#58-paper-process)
   - 5.9 [The Variant Table — MEDDIC vs MEDDICC vs MEDDPICC](#59-the-variant-table--meddic-vs-meddicc-vs-meddpicc)
6. [The Component Interplays — the Chains That Close Deals](#6-the-component-interplays--the-chains-that-close-deals)
   - 6.1 [The Champion-Coaching Chain](#61-the-champion-coaching-chain)
   - 6.2 [The Paper-Process Chain](#62-the-paper-process-chain)
   - 6.3 [The Pain-Implications-Metrics Chain](#63-the-pain-implications-metrics-chain)
   - 6.4 [The Deal as a System](#64-the-deal-as-a-system)
7. [The Qualification Discipline — Deal Reviews and Scorecards](#7-the-qualification-discipline--deal-reviews-and-scorecards)
   - 7.1 [Evidence-Gated Deal Reviews](#71-evidence-gated-deal-reviews)
   - 7.2 [Scorecards and Opportunity Scoring](#72-scorecards-and-opportunity-scoring)
   - 7.3 [MEDDPICC in the CRM](#73-meddpicc-in-the-crm)
   - 7.4 [The Vendor-Specific Practice Caveat](#74-the-vendor-specific-practice-caveat)
   - 7.5 [Standing the Discipline Up — the First 90 Days](#75-standing-the-discipline-up--the-first-90-days)
8. [Forecasting — Commit, Upside, and the MEDDPICC Signal](#8-forecasting--commit-upside-and-the-meddpicc-signal)
   - 8.1 [The Commit-vs-Upside Conventions](#81-the-commit-vs-upside-conventions)
   - 8.2 [What MEDDPICC Adds to a Forecast](#82-what-meddpicc-adds-to-a-forecast)
   - 8.3 [The Caveats](#83-the-caveats)
9. [The Common Failure Modes](#9-the-common-failure-modes)
   - 9.1 [Champion Misidentification — Coach vs Champion](#91-champion-misidentification--coach-vs-champion)
   - 9.2 [The Economic-Buyer Gap](#92-the-economic-buyer-gap)
   - 9.3 [The Missing Paper Process](#93-the-missing-paper-process)
   - 9.4 [Competition Blindness](#94-competition-blindness)
   - 9.5 [Pain That Is Not Quantified](#95-pain-that-is-not-quantified)
   - 9.6 [The Failure-Mode Table](#96-the-failure-mode-table)
10. [The Complementary Methodologies — BANT, SPIN, Challenger](#10-the-complementary-methodologies--bant-spin-challenger)
    - 10.1 [BANT](#101-bant)
    - 10.2 [SPIN Selling](#102-spin-selling)
    - 10.3 [The Challenger Sale](#103-the-challenger-sale)
    - 10.4 [The Comparison Table](#104-the-comparison-table)
    - 10.5 [How They Fit With MEDDPICC](#105-how-they-fit-with-meddpicc)
11. [The Banking-Wholesale Application](#11-the-banking-wholesale-application)
    - 11.1 [The Shape of a Wholesale-Banking Deal](#111-the-shape-of-a-wholesale-banking-deal)
    - 11.2 [The Economic Buyer in a Bank Deal](#112-the-economic-buyer-in-a-bank-deal)
    - 11.3 [Decision Criteria and the RFP Scorecard](#113-decision-criteria-and-the-rfp-scorecard)
    - 11.4 [Paper Process — Credit, Onboarding, Legal](#114-paper-process--credit-onboarding-legal)
    - 11.5 [The Multi-Stakeholder Committee](#115-the-multi-stakeholder-committee)
    - 11.6 [Cross-References to the Sibling Guides](#116-cross-references-to-the-sibling-guides)
    - 11.7 [The Banking Timeline Map — Letters Across an RFP Cycle](#117-the-banking-timeline-map--letters-across-an-rfp-cycle)
12. [The Cymbal Bank Worked Example — Qualifying a Transaction-Banking RFP](#12-the-cymbal-bank-worked-example--qualifying-a-transaction-banking-rfp)
    - 12.1 [The Scenario](#121-the-scenario)
    - 12.2 [The MEDDPICC Map — Letter by Letter](#122-the-meddpicc-map--letter-by-letter)
    - 12.3 [The Qualification Scorecard](#123-the-qualification-scorecard)
    - 12.4 [The Deal Review](#124-the-deal-review)
    - 12.5 [The Lessons](#125-the-lessons)
    - 12.6 [The Anti-Example — the Same Deal, Unqualified](#126-the-anti-example--the-same-deal-unqualified)
13. [The Claims Audit](#13-the-claims-audit)
    - 13.1 [Verified This Pass](#131-verified-this-pass)
    - 13.2 [Flagged or Approximate (⚠ and ⚠-knowledge)](#132-flagged-or-approximate--and--knowledge)
    - 13.3 [Could Not Verify or Conflicting](#133-could-not-verify-or-conflicting)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [The Glossary, the Sources, and the Closing](#15-the-glossary-the-sources-and-the-closing)
    - 15.1 [The Glossary](#151-the-glossary)
    - 15.2 [The Sources](#152-the-sources)
    - 15.3 [The Closing Summary](#153-the-closing-summary)

---

## 1. The Concept — What MEDDPICC Is and Why It Exists

### 1.1 The Short Answer

**MEDDPICC is a qualification framework for complex, multi-stakeholder B2B sales.** It is an acronym whose letters name the eight things a seller must *know with evidence* before a big deal is worth betting pipeline, forecast and reputation on: **M**etrics, **E**conomic Buyer, **D**ecision Criteria, **D**ecision Process, **P**aper Process, **I**dentify Pain, **C**hampion, **C**ompetition. The older six-letter form, **MEDDIC**, lacks the two added letters; the seven-letter form, **MEDDICC**, adds only Competition (see §3 for the letter-level archaeology). ✅ The MEDDICC company's own current explanation defines the framework's purpose as qualifying "your deal at the earliest opportunity" and keeping it relevant "throughout the deal's evolution," with the letters as "critical information" that lets a team "assess the health of every deal" ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)).

A useful one-sentence summary, widely echoed by practitioners: MEDDPICC answers three questions about an opportunity — *is it real, can we win it, and is it worth winning?* ⚠-knowledge: that exact phrasing is standard in the sales-training literature (for example the practitioner framing that qualification decides "whether a complex deal is real, winnable, and worth pursuing before [the rep] invests a full sales cycle in it," seen across Highspot and other practitioner guides surfaced this pass). The letters are not a checklist to recite at the buyer; they are a **memory system for deal evidence** — what to know, who to ask, and what to verify before a forecast number is attached to a name.

### 1.2 The Qualification Problem

MEDDPICC exists because of a structural fact about enterprise selling: **the cost of pursuing the wrong deal is enormous, and the cost of misreading the right deal is fatal.** ✅ The origin account published by MEDDICC (the company) describes the problem the framework was built to solve inside PTC: when the sales organization reached roughly 300 reps, attrition became a serious threat and the company "could not hire fast enough to replace the people it was losing"; Dick Dunkel and Jack Napoli were tasked with studying past and present opportunities to answer three questions — *why did PTC win deals, why did PTC lose deals, and why did deals slip?* — and the pattern they found traced every time to the six areas that became MEDDIC ([meddicc.com/resources/who-created-meddic](https://meddicc.com/resources/who-created-meddic)). In other words: the methodology is not a theory of persuasion; it is an **empirical taxonomy of win/loss/slip causes**, distilled from a data set of real deals.

That origin matters for how you use the framework today. A qualification methodology is only useful if it changes behavior before resources are sunk: it tells the rep which unknowns are *dangerous* (no economic buyer, no champion, criteria unknown) versus merely *incomplete*, and it gives the sales manager a shared language to challenge a rep's optimism. Without it, deal reviews degenerate into vibes — "the customer loves us" — which is precisely the failure mode the PTC study was chasing.

### 1.3 Methodology vs Process — Where It Sits

A useful distinction, covered in the sibling survey guide at [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) (§2.2) and followed here: a **sales process** is the sequence of stages a deal moves through (prospect → qualify → propose → close), while a **sales methodology** is the thinking discipline applied *within* those stages. MEDDPICC is a methodology in that sense: it does not tell you what stage to be in; it tells you what evidence must exist at every stage. It is also, in the taxonomy of the sibling survey, a **qualification framework** — a specialized methodology for the go/no-go and health-assessment decisions — rather than a messaging, negotiation, or account-strategy system. That is why it composes with other methodologies rather than replacing them: SPIN gives you the question technique for discovery, Challenger gives you the message shape, MEDDPICC gives you the evidence ledger those conversations must fill (see §10.5).

### 1.4 What This Guide Adds

The repository's general survey ([../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md), §4) already sketches MEDDIC's six dimensions, the variants, and a deal-review one-pager. This guide is the dedicated deep-dive and does not duplicate that survey: it verifies the *lineage* against primary and first-person sources (§2–§3), documents the *industry-adoption machinery* of training and enablement vendors with marketing flagged (§4), builds the *full component map* with per-letter meaning, rationale, classic mistake and variant notes (§5), explains the *interplays* (§6), covers the *practice discipline* — deal reviews, scorecards, CRM, forecasting — and the failure modes (§7–§9), verifies the *complementary methodologies* and compares them (§10), applies the framework to *wholesale banking* (§11), and closes with a full *Cymbal Bank worked example* on a transaction-banking RFP (§12).

## 2. The Lineage — MEDDIC at PTC

### 2.1 The Origin Story as the Primary Sources Tell It

The framework was born inside **PTC — Parametric Technology Corporation**, the Massachusetts software company whose Pro/Engineer CAD software defined the mid-1990s enterprise-software boom. ✅ The MEDDICC company's history article (13 Aug 2026) states: "MEDDIC was created in 1996 inside PTC by Dick Dunkel, working under SVP John McMahon alongside teammate Jack Napoli," and credits PTC's growth "from $0 to $1 billion in ten years" partly to the discipline MEDDIC brought ([meddicc.com/resources/who-created-meddic](https://meddicc.com/resources/who-created-meddic)). The same page frames the origin as an internal study — winners, losers, slippers — that resolved into six recurring patterns: Metrics, Economic Buyer, Decision Criteria, Decision Process, Identify Pain, and Champion.

A second, independent first-person account comes from **Monica Evans**, a PTC sales-development veteran who says she was present when the framework was first written down. Her account (published on salesmeddic.com) is more nuanced about authorship: "MEDDIC/MEDDICC/MEDDPICC is John McMahon's sales brain codified; Dick Dunkel was the first to write it down; and then Dick and Jack told everybody at PTC… So that being said, the actual 'biological' father of MEDDIC is Dick Dunkel." Evans also gives the supporting cast: PTC leadership under Sam Geisberg (creator of Pro/Engineer), CEO Steve Walske, CRO Dick Harrison and VP of Sales Mike McGuiness "had tasked Dick Dunkel and I" with creating an onboarding program, documenting the sales process, and running a worldwide "Demo Tour" — the documentation task being the one that produced the written method ([salesmeddic.com/blog/origin-of-meddic](https://www.salesmeddic.com/blog/origin-of-meddic)).

### 2.2 The Cast — Dunkel, Napoli, McMahon, Evans

- **Dick Dunkel** — the "biological father" per Evans and the creator per MEDDICC's account; the man who wrote MEDDIC on the whiteboard. ✅ Dunkel later built training businesses around the method: **RevCentric Partners**, which he co-founded with David Boyle (described as "the first MEDDIC instructor"), coaching enterprise sales teams "in live deals using the methodology we invented" ([revcentricpartners.com/about](https://revcentricpartners.com/about)); and in the 2020s he joined **MEDDICC** (the company) as CSO ([meddicc.com/resources/meddicc-welcomes-dick-dunkel](https://meddicc.com/resources/meddicc-welcomes-dick-dunkel)). ✅ Andy Whyte's own career bio records that Whyte "learned MEDDPICC with Dick Dunkel" while at Sprinklr (2015–17) — direct evidence that Dunkel ran live MEDDPICC training engagements into the SaaS era ([meddicc.com/resources/the-backstory-of-andy-whyte](https://meddicc.com/resources/the-backstory-of-andy-whyte)).
- **Jack Napoli** — Dunkel's collaborator in the original documentation task; both MEDDICC's account and Evans's credit him as the co-teacher who "told everybody at PTC." ⚠ Napoli's independent footprint online is thin; most of what is published about his role comes through the MEDDICC company's own retelling, so the depth of his contribution is hard to verify beyond the two accounts above.
- **John McMahon** — the sales leader under whom MEDDIC emerged. ✅ Multiple independent profiles confirm McMahon's career: five-time CRO at public software companies including PTC, GeoTel, Ariba, BladeLogic and BMC, later a board member at Snowflake and MongoDB (among others), and author of the sales-leadership book *The Qualified Sales Leader* (echai.ventures GTM profile; sellinglikeachampion.com; a Force Management interview). Third-party sources variously call him "creator of MEDDIC" or "originator of the MEDDIC sales qualification framework" ⚠ — which conflicts with the Dunkel-as-creator accounts above; see §2.3.
- **Monica Evans** — PTC sales-development veteran and self-described "godfather" of MEDDIC who says she witnessed the first whiteboard and trained an estimated 3,000 customer-facing PTC employees in it. Her account is the most granular first-person source on the origin available this pass ✅ ([salesmeddic.com/blog/origin-of-meddic](https://www.salesmeddic.com/blog/origin-of-meddic)).

### 2.3 The Conflicts — Who Coined What

The origin story has real, documentable conflicts, and an honest guide must surface them:

1. **Dunkel vs McMahon as "creator."** The MEDDICC company's official history names Dick Dunkel as the creator (with Napoli collaborating, under McMahon). Evans splits the difference: McMahon's *brain*, Dunkel's *pen* — "McMahon's sales brain codified; Dunkel was the first to write it down." Independent third-party blogs and profiles frequently collapse this into "John McMahon, creator of MEDDIC" (e.g., antoinebuteau.com's "Lessons from John McMahon, creator of MEDDIC"; eChai's profile noting "PTC is where the MEDDIC qualification framework originated" under his CRO tenure). ⚠ The truth is likely the Evans version — a sales leader whose philosophy was codified by a gifted subordinate — but the label "creator" is applied to both men in different sources, and the primary accounts are themselves published by parties with commercial interests (see §2.4). The conflict is real and unresolved in the public record.
2. **The Andy Whyte conflation.** A commonly repeated claim in secondary sources is that MEDDIC was created by "Dick Dunkel, John McMahon **and Andy Whyte**" at PTC — and variants that Andy Whyte coined the acronym. ✅ The primary evidence contradicts this. Whyte's own published backstory shows he was *not* at PTC in the 1990s at all: his career ran from a DJ/emcee job at a holiday resort (2001–04) through door-to-door sales (2004–06), a first B2B SaaS role at ECI (2006–08), ES Tech Group (2008–13), Oracle (2013–15), Sprinklr (2015–17 — where he "learned MEDDPICC with Dick Dunkel"), Tealium, Poq, and Branch (2019–21), before **founding MEDDICC™ (the company) in 2020** ([meddicc.com/resources/the-backstory-of-andy-whyte](https://meddicc.com/resources/the-backstory-of-andy-whyte)). Whyte is the founder of the *MEDDICC training company* and author of the *MEDDICC* book — not a PTC-era co-creator of the acronym. ⚠ The likely mechanism of the conflation: Whyte's company brand (MEDDICC) became synonymous with the methodology, so later writers retro-projected him into the origin story. Neither MEDDICC's own history page nor Evans's first-person account mentions Whyte as a creator. Any source crediting Whyte with coining MEDDIC at PTC should be treated as unverified secondary conflation.
3. **The date.** MEDDICC's account says 1996. Evans's account gives no year. The trademark-ruling summary (see §3.3) refers to MEDDIC being "developed at PTC in the early 1990s," while third-party guides say "mid-1990s." ✅/⚠: 1996 (MEDDICC's primary account) is consistent with the "mid-1990s" framing and PTC's growth timeline, but the court summary's "early 1990s" shows even the general era is stated loosely in some records. The mid-1990s dating is safe; the exact year rests on a single commercial source.

### 2.4 Reading the Origin Story Critically

⚠ Two of the three richest origin sources are commercially motivated. MEDDICC (the company, founded 2020) sells MEDDPICC training, certification and a platform, and its history page ends in a sales pitch ("Ready to put it into practice?… Get certified through the MEDDPICC Masterclass"). It has also hired Dick Dunkel as CSO — so its "Dunkel is the creator" telling doubles as brand legitimacy for the Dunkel-endorsed training product. RevCentric Partners is Dunkel's own firm and calls him "author of MEDDIC." None of this makes the accounts false — Evans's independent first-person testimony corroborates the core (PTC, mid-1990s, Dunkel wrote it down, McMahon's brain, Napoli helped spread it) — but it explains why "creator" attributions vary with the publisher's commercial interest. The **corroborated core** is: PTC, mid-1990s, a win/loss/slip study, six letters, Dunkel's pen, McMahon's sponsorship, Napoli's evangelism, and the PTC diaspora spreading it. Everything beyond that core is attribution politics ⚠.

### 2.5 The PTC Diaspora — How the Six Letters Left the Building

MEDDIC escaped PTC the way most enterprise-sales innovations escape: through people. ✅ Monica Evans estimates that "approximately 3,000 customer-facing employees learned MEDDIC" through PTC's internal education during her tenure — a large, trained, quota-carrying population that then rotated through the wider software industry as PTC's own alumni network spread ([salesmeddic.com/blog/origin-of-meddic](https://www.salesmeddic.com/blog/origin-of-meddic)). ✅ MEDDICC's history page makes the same point from the company side: "PTC's focus on qualification, and its development of MEDDIC, sent thousands of MEDDIC-proficient salespeople out into the wider workforce. Look at many of today's most respected sales leaders and you can trace their sales lineage straight back to PTC" ([meddicc.com/resources/who-created-meddic](https://meddicc.com/resources/who-created-meddic)). The mechanism matters for the adoption story (§4): MEDDIC did not spread because a book or a training company marketed it in the 1990s — it spread because hiring managers who had sold at PTC imported the vocabulary into their next companies, and those companies' reps carried it onward. The later vendor ecosystem (MEDDICC, Force Management, Winning by Design) industrialized a diffusion that the diaspora had already started by hand. ⚠ The "thousands of alumni" framing is consistent across the two sources but neither publishes a roster; the qualitative claim — that a PTC-trained generation carried MEDDIC into the SaaS/enterprise era — is corroborated by the number of senior sales figures whose public bios cite PTC tenure and by the framework's observable presence a generation later.

## 3. The Expansions — MEDDICC and MEDDPICC

### 3.1 Which Letter Each Variant Adds

The letter-level evolution is itself documented at the primary source. ✅ MEDDICC's comparison section states it plainly: "MEDDIC is the original framework. MEDDICC added Competition. MEDDPICC added Paper Process — making it the most complete version for modern, complex B2B sales" ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)). Letter by letter:

| Variant | Letters | What was added | Position |
|---|---|---|---|
| **MEDDIC** | 6 — M E D D I C | — (the original) | — |
| **MEDDICC** | 7 — M E D D I C C | **C = Competition** | appended after Champion |
| **MEDDPICC** | 8 — M E D D P I C C | **P = Paper Process** (plus the Competition C) | P inserted between Decision Process and Identify Pain; C appended at the end |

So the mnemonic order of the full MEDDPICC is: Metrics, Economic Buyer, Decision Criteria, Decision Process, **Paper Process**, Identify Pain, Champion, Competition. ✅ That ordering is confirmed by MEDDICC's own letter list ("Metrics, Economic Buyer, Decision Criteria, Decision Process, + Paper Process, Implicate Pain, Champion, + Competition") and its per-letter definitions ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)). Note the wrinkle: the P is inserted *before* the I, even though conceptually paper work happens after the decision — the acronym's letter order reflects when the letters were *added*, not the chronological flow of a deal, and practitioners are unanimous that the letters are a memory device, not a sequence to execute. ⚠-knowledge: some training materials re-order the letters into a deal-flow narrative (Metrics → … → Paper Process last); that is a teaching choice, not the canonical spelling.

✅ The modern MEDDICC company also re-glosses the **I** in the 8-letter form as "**Implicate** the Pain" (their definition: "you have both Identified, Indicated, and Implicated the Pain your solution solves"), an elaboration of the original six-letter "Identify Pain" that absorbs the "implications of pain" concept — see §5.5 for the full treatment. This is the source of the "some vendors append Implications of pain" observation: the I in extended variants is often expanded into a three-step Identify–Indicate–Implicate chain rather than a single letter.

### 3.2 The Contested Coinage

Who coined the *extended* acronyms is contested and partly vendor-specific ⚠:

- **MEDDICC (the 7-letter variant).** The extra C for Competition is associated with **John McMahon's** school of the methodology — the variant name itself became the brand of the MEDDICC company (founded by Andy Whyte, 2020), which teaches MEDDPICC but brands itself MEDDICC. ⚠ No primary source located this pass states *who first appended the C* or when; the attribution "championed by John McMahon" is plausible (McMahon's *The Qualified Sales Leader* and his coaching line teach Competition as a core discipline) but I could not verify a document in which McMahon claims the coinage. Treat "McMahon coined MEDDICC" as ⚠ unverified; "the MEDDICC brand popularized the Competition letter" as ✅ (the company's own comparison text credits the variant with adding Competition).
- **MEDDPICC (the 8-letter variant).** The P for Paper Process is the differentiator. ✅ The April 2026 federal court ruling summary states MEDDPICC was "coined in 2005 by former Parametric Technology Corporation (PTC) employees" (§3.3) — i.e., a post-PTC generation of alumni added Paper Process roughly a decade after the original six letters. ⚠ The individual(s) who coined the 2005 form are not named in the sources verified this pass, and the MEDDICC company (which litigated the trademark) has an interest in the "former PTC employees, not the trademark holder" framing. Attribution of the P to any specific person is therefore ⚠ unverified.
- **"Implications of pain."** This phrase appears in vendor and practitioner expansions of the I (the Identify–Indicate–Implicate chain above, and in some SPIN-influenced framings that map MEDDPICC's I onto Rackham's Implication questions). ⚠-knowledge: no single originator is credited in the literature reviewed; it is a practitioner elaboration, not part of the original six letters.

### 3.3 The 2026 Trademark Ruling — MEDDPICC Goes Generic

A genuinely notable 2026 development, and the only piece of *case law* in the lineage story. ✅ Per MEDDICC's release (28 May 2026): on **21 April 2026**, the United States District Court for the Eastern District of Pennsylvania (Civil Action No. 24-1836, *MEDDICC Ltd v. 01 Consulting LLC*; Chief Judge Wendy Beetlestone) granted summary judgment to MEDDICC Ltd, ruling that **MEDDPICC is a generic term** and ordering the USPTO to cancel Registration No. 6,489,058 — a trademark held by Darius Lahoutifard of 01 Consulting LLC, doing business as MEDDIC Academy. The court found Lahoutifard "was not involved in MEDDPICC's conception," yet "sixteen years after its genesis, he federally registered the term as his trademark," and dismissed his counterclaims with prejudice ([meddicc.com/resources/meddpicc-trademark-cancelled-federal-court-ruling](https://meddicc.com/resources/meddpicc-trademark-cancelled-federal-court-ruling)). ⚠ This account is MEDDICC's own press release — the party that won — and I did not verify the docket independently this pass; but it names the court, the case number, the judge, the registration number and the date, and it is consistent with the observable fact that MEDDPICC training is offered by many unaffiliated vendors (which is what "generic term" means commercially). The ruling's practical effect, as the release argues: MEDDPICC "is a general sales methodology" usable by anyone, and the term cannot be owned.

### 3.4 The Wider Variant Zoo

Beyond the big three, the practitioner literature (and this pass's searches) surfaced a genuine zoo of near-variants ⚠-knowledge, mostly vendor-specific spellings and misspellings that have taken on lives of their own:

- **MEDDPIC** (7 letters) — MEDDIC + Paper Process without the Competition C; used by some vendors who insert the P but skip the second C.
- **MEDDICC with Implications of pain** — some vendors teach "MEDDICC" where the I is expanded to "Implications of pain" (or "Identify/Implicate pain"), producing extended glosses beyond the strict 7 letters.
- **MEDDPICC with a second I** — occasional renderings add "Implications of pain" as a ninth element in the letter list (Metrics, Economic Buyer, Decision Criteria, Decision Process, Paper Process, Identify Pain, **Implications of pain**, Champion, Competition), which is where the "nine disciplines" reading of the framework comes from even though the acronym itself is 8 letters.
- **SPICED** — Winning by Design's SaaS-flavored cousin (Situation, Pain, Impact, Critical event, Decision criteria, Economic buyer) ✅ per winningbydesign.com's own description of the frameworks it created (see §4.3); included here only to note that MEDDPICC's DNA is visible in adjacent SaaS methodologies.

⚠ The practical guidance from the vendor-comparison literature this pass reviewed (closingfoundry.com, praiz.io, orm-tech.com, nimitai.com) is consistent: the letters you *need* track to your deal's risk profile — Paper Process matters most where procurement/legal gate-keeps the signature; Competition matters most where you are in a bake-off; and the core six are table stakes everywhere. The variant you choose is a policy decision for the sales organization; the underlying disciplines are the same.

## 4. The Industry Adoption — the Training and Enablement Vendors

MEDDPICC's spread is not the organic diffusion of an idea alone; it is a story of **training and enablement vendors** who productized, branded and distributed the method. This section maps the main carriers and separates what is verifiable from what is marketing.

### 4.1 MEDDICC the Company — and the McMahon Line

✅ The most visible carrier is **MEDDICC** (meddicc.com), founded by Andy Whyte in 2020 after he implemented MEDDIC at Branch (EMEA MD, 2019–21) and learned MEDDPICC from Dick Dunkel at Sprinklr (2015–17) ([meddicc.com/resources/the-backstory-of-andy-whyte](https://meddicc.com/resources/the-backstory-of-andy-whyte)). The company sells the **MEDDPICC Masterclass** (certification, "8 in-depth modules — one per letter"), a platform ("mOS") that puts MEDDPICC fields "right where your reps work every day," call recording and opportunity-management tools, and corporate membership. It reports "25,000+ professionals certified," "2,500+ active teams," "500+ organizations in the registry" and "80,000+ books sold," with customer logos including CrowdStrike, AWS, Cloudflare, Darktrace, Zapier and Cognism. ⚠ All of those figures and logos are the company's own marketing claims — plausible for a category leader, but not independently audited this pass. Notably, Dick Dunkel joined MEDDICC as CSO (✅ per the company's announcement), which gives the vendor a direct line to the origin story, and the company runs the "Masters of MEDDICC" podcast that features Dunkel recounting the PTC history.

**The McMahon line** is the other carrier of the original DNA. ✅ John McMahon's book *The Qualified Sales Leader* (⚠-knowledge: widely dated to 2021) codifies his version of the discipline — deal inspection, qualification, A-player management — and McMahon himself remains the most-cited living authority on MEDDIC practice (five-time CRO at PTC, GeoTel, Ariba, BladeLogic, BMC; board roles at Snowflake and MongoDB per the profiles verified this pass). Force Management publishes an interview with him, and the "Selling Like a Champion" podcast profile describes him as "one of the most respected voices in enterprise sales leadership." Whether McMahon runs a formal training company of his own could not be verified this pass ⚠; his vehicle appears to be the book, speaking, board work, and the ecosystem of firms (MEDDICC, Force Management, RevCentric) that teach his philosophy.

### 4.2 Force Management

✅ **Force Management** — the B2B sales training and enablement firm best known for Command of the Message — explicitly offers MEDDICC as its qualification approach ("MEDDICC is an acronym that outlines our sales qualification approach… Using MEDDICC as an internal sales qualification tool helps drive consistent discovery and efficient qualification of opportunities") and pairs it with its Command of the Message messaging methodology ([forcemanagement.com/offerings/meddicc](https://www.forcemanagement.com/offerings/meddicc)). Its page carries named client testimonials — Intercom's Head of NA & APAC Sales crediting "Command of the Message® and MEDDPICC" with ~4x average revenue per account growth, and MindTickle's Head of Sales Enablement praising the reinforcement package. ⚠ These are vendor-published testimonials: directional marketing evidence that large tech firms buy MEDDPICC-based training from Force Management, not audited outcome data. What is verifiable is the *adoption pattern*: a major independent training firm embedding MEDDICC-family qualification into its methodology stack is itself evidence of industry adoption.

### 4.3 Winning by Design and the SaaS/RevOps Wave

✅ **Winning by Design** (winningbydesign.com), the global GTM consultancy founded by Jacco van der Kooij, is a central carrier of the methodology into SaaS: the firm's own site describes it as creating the **SPICED** sales methodology, the Bowtie revenue framework and the REKS coaching framework, and its client work spans recurring-revenue companies (publicly named alumni include Uber Eats, DocuSign, MURAL and OwnBackup per its Amazon author bio ⚠-knowledge). SPICED — Situation, Pain, Impact, Critical event, Decision criteria, Economic buyer — is recognizably MEDDPICC's skeleton re-cut for SaaS (pain and economic buyer retained, paper process dropped, urgency added as "Critical event"). ⚠-knowledge: Winning by Design has long taught MEDDPICC-derived qualification in its SaaS Sales Method curricula and its "Blueprints" library; the firm's own site this pass confirmed the SPICED creation claim and the SaaS Sales Method book (van der Kooij & Pizarro, *The SaaS Sales Method: Sales As a Science*), but I did not re-verify specific MEDDPICC course pages.

**The RevOps/Gong wave** ⚠-knowledge: the 2018–2024 period saw revenue-intelligence platforms — Gong being the most prominent — build MEDDPICC into deal-inspection workflows, coaching scores and "MEDDPICC for sales managers" content, and the methodology's vocabulary (champion, economic buyer, paper process) is now routine in RevOps job descriptions and enablement stacks at SaaS companies. Gong's own MEDDPICC materials and adoption metrics were not directly verified this pass (searches returned vendor-neutral pages rather than Gong primary content) ❌, so this specific claim is flagged rather than asserted as fact; the *general* pattern — MEDDPICC as the default enterprise qualification language of the SaaS era — is corroborated by the volume of independent practitioner literature (Highspot, MeetGeek, Livespace, and the dozens of vendor comparison pages surfaced in this pass's searches) that treats it as the incumbent standard.

### 4.4 The Evangelists — SalesHacker, Gong, Highspot

Beyond the paid training vendors, the methodology spread through **content evangelism**: SalesHacker (the community site now part of Pavilion ⚠-knowledge) has published MEDDIC explainers and practitioner takes for a decade; Highspot, Gong and a long tail of enablement platforms publish MEDDIC/MEDDPICC "complete guides" (Highspot's and MeetGeek's were among the top results this pass); and LinkedIn is dense with MEDDICC-certified profiles (the MEDDICC certification is explicitly designed to be "shown off on LinkedIn" per the company's FAQ ✅). The pattern is a classic category-building loop: training vendors certify practitioners → certified practitioners evangelize on content platforms → employers adopt the vocabulary → more demand for training. ⚠ Where the marketing line exceeds verifiable fact is in claims of *outcome*: "teams using MEDDPICC forecast better / grow faster" is the universal vendor pitch, but the controlled-evidence base for any sales methodology's ROI is thin (see §7.4).

### 4.5 Where the Vendor Claims End and Verified Fact Begins

A disciplined summary of the adoption story:

- ✅ **Verified:** MEDDIC originated at PTC in the mid-1990s (§2); the variants added Competition and Paper Process (§3.1); a 2020-founded company named MEDDICC sells MEDDPICC training at scale and hired Dick Dunkel as CSO (§4.1); Force Management and Winning by Design — independent, well-known firms — embed MEDDICC-family qualification in their offerings (§4.2–4.3); a 2026 federal court treated MEDDPICC as a generic industry term (§3.3).
- ⚠ **Marketing, not audited fact:** headcount figures ("25,000+ certified," "2,500+ teams"), customer rosters, "80,000+ books sold," client ROI testimonials, and any claim that MEDDPICC adoption *causes* better forecasts or faster growth.
- ⚠-knowledge: the specific mechanics of Gong's MEDDPICC tooling and SalesHacker's article archive were not re-verified this pass; the general adoption direction they represent is well documented.

## 5. The Component Map — Every Letter, Verified

This section defines every component. For each letter: the ✅ verified meaning (glossed from the methodology's own sources or from convergent practitioner definitions), why it matters, the classic mistake, and any variant differences. Component glosses marked ✅ are from MEDDICC's 2026 per-letter definitions ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)); where a definition is instead drawn from convergent practitioner literature it is marked accordingly.

### 5.1 Metrics

- **Verified meaning:** ✅ MEDDICC: "Metrics are the quantifiable measures of value that your solution can provide," and the company teaches a breakdown into "M1s, M2s and M3s" (⚠ that M1/M2/M3 sub-taxonomy is vendor-specific — the original MEDDIC simply said Metrics). Convergent practitioner definition ⚠-knowledge: the business metrics the customer cares about and the numbers your solution moves — revenue, cost, time, risk — expressed as current state, target state, and the gap your product closes.
- **Why it matters:** a deal sold on feelings cannot be defended internally by your champion, priced with conviction, or forecast with confidence. Quantified value is the *currency* of the rest of the framework: it feeds the pain case (§5.5), gives the champion ammunition (§6.3), and justifies the price to the economic buyer.
- **Classic mistake:** quoting *product* metrics (throughput, features, uptime) instead of *business* metrics the customer's CFO would recognize; or accepting the customer's vague "we'll save money" without agreeing the baseline number and the measurement method.
- **Variant notes:** present in all variants from the original six letters onward. ⚠-knowledge: Winning by Design's SPICED renames the spirit of this letter "Impact," and the modern "M1/M2/M3" framing is a MEDDICC-company pedagogy, not part of the 1996 original.

### 5.2 Economic Buyer

- **Verified meaning:** ✅ MEDDICC: "The Economic Buyer is the person with the overall authority in the buying decision." Practitioner gloss ⚠-knowledge: the individual who can say *yes* to the money — who owns the budget and whose approval cannot be appealed over. Not necessarily the person with the biggest title and never the person who merely signs.
- **Why it matters:** every stakeholder can say *no*; only the economic buyer can say *yes* over the objections of everyone else. Until the EB is identified, engaged and aligned, the deal is a coalition of preferences without a decision-maker — and competitors with EB access will win even with worse products.
- **Classic mistake:** mistaking the *user* or *IT gatekeeper* for the EB, or assuming the highest title you have met is the EB. Also common: ignoring the EB until the end ("we'll bring them in at the proposal stage") — by which point the criteria are fixed and the budget allocated.
- **Variant notes:** E is constant across MEDDIC, MEDDICC, MEDDPICC and SPICED (where it is the final letter).

### 5.3 Decision Criteria

- **Verified meaning:** ✅ MEDDICC: "The Decision Criteria are a set of principles, guidelines and requirements that an organization uses to make a decision about your solution." Practitioner gloss ⚠-knowledge: the *actual* scorecard — published or hidden — on which you will be judged: functional requirements, commercial terms, risk appetite, references, incumbent bias.
- **Why it matters:** criteria determine whether your strengths are weighted heavily or irrelevant. The sales job is not only to meet criteria but to *shape* them while they are still being written — helping the customer write criteria your differentiators satisfy — because once the RFP is published, the game is largely scored.
- **Classic mistake:** assuming the published requirements are the real criteria (the real ones often include "lowest risk," "trusted incumbent," "internal preference," and other unwritten items), or discovering the criteria only at the proposal stage when they are frozen.
- **Variant notes:** constant across variants; in banking it maps directly onto the RFP scorecard (§11.3).

### 5.4 Decision Process

- **Verified meaning:** ✅ MEDDICC: "The Decision Process is the steps that the buyer will take to decide whether they will buy your solution." Practitioner gloss ⚠-knowledge: the sequence — who evaluates, in what order, with what gates, on what timeline — including the internal steps (shortlist, demos, references, business case, committee vote) that lead to the decision.
- **Why it matters:** a deal without a mapped decision process is a deal you cannot navigate or time. Knowing the process tells you which stakeholder to arm for which gate, when the decision actually lands (vs when the seller hopes it lands), and what events you can influence.
- **Classic mistake:** confusing *your* sales process (demo → proposal → negotiation) with *their* buying process; assuming the person you are talking to knows or tells the truth about the internal process; ignoring the difference between the decision date and the signature date.
- **Variant notes:** constant across variants. Distinguished from Paper Process (below) as the *decision* steps vs the *contractual/administrative* steps after the decision ✅ per MEDDICC's pairing of the two definitions.

### 5.5 Identify Pain — and the Implications of Pain

- **Verified meaning:** ✅ the original sixth-letter **I** of MEDDIC is **Identify Pain**. MEDDICC's modern 8-letter gloss renames it "**Implicate the Pain**": "Implicating the Pain means you have both Identified, Indicated, and Implicated the Pain your solution solves" — an explicit three-step chain (identify the problem → indicate its consequences → implicate the cost of not acting). This is the canonical home of the "**implications of pain**" concept that some vendors append as an extra letter: the recognition that *pain alone does not sell; pain with consequences and a deadline does*.
- **Why it matters:** the pain is the motive force of the entire deal. If the pain is small, vague or unacknowledged by the EB, every other letter collapses — there is no reason to change, no urgency, no budget. The identify→implicate expansion matters because buyers routinely admit *problems* ("our reconciliation takes too long") while resisting the *implications* ("and that costs us X per month in float and errors"); the seller's job is to make the implications vivid and quantified.
- **Classic mistake:** stopping at problem-awareness ("yes, they have pain") without quantifying it or connecting it to a business metric and a timeline; or selling to pain the *customer* does not feel (your pain, not theirs).
- **Variant notes:** the letter position and spelling differ across the family — "Identify Pain" (MEDDIC/MEDDICC), "Implicate the Pain" (MEDDICC's MEDDPICC gloss), and "Implications of pain" as a separately-spelled ninth element in some extended vendor lists (§3.4). ⚠ The three-step Identify–Indicate–Implicate framing is MEDDICC-company pedagogy; the underlying "pain → implications" logic is the SPIN research's Implication-question insight (§10.2) absorbed into MEDDPICC practice.

### 5.6 Champion

- **Verified meaning:** ✅ MEDDICC: "The Champion is a person who has power, influence, and credibility within the customer's organization." Practitioner gloss ⚠-knowledge adds the active ingredient: a champion is someone who *wants your solution to win and is actively selling it inside* — not merely someone who likes you. Power/influence/credibility is what makes their advocacy effective.
- **Why it matters:** the seller can only be in the room occasionally; the champion is in the room all the time — in the hallway, at the budget meeting, in the criteria workshop. The champion is the channel through which the seller shapes criteria (§6.1), the internal voice that translates your value into their language, and the early-warning system for competitive moves and process changes.
- **Classic mistake:** confusing a *coach* (friendly informant who tells you what is happening) with a *champion* (advocate who makes it happen) — see §9.1; or assuming the person who likes you most is the person with the influence to change votes.
- **Variant notes:** C is constant across all variants; the *second* C (Competition) follows it in MEDDICC/MEDDPICC, which is a mnemonic accident of the acronym's growth, not a claim that Competition comes after Champion chronologically.

### 5.7 Competition

- **Verified meaning:** ✅ MEDDICC (7-letter MEDDICC and 8-letter MEDDPICC) adds "Competition" — the discipline of knowing who else is in the deal, on what terms, with what relationships, and how the customer frames the choice. MEDDICC's own comparison text: MEDDICC "helps teams navigate competitive landscapes and differentiate more effectively." (⚠ the company's per-letter page had a copy/paste error repeating the Champion definition under Competition; the component's meaning is nonetheless unambiguous from the variant-comparison text and every practitioner source reviewed.)
- **Why it matters:** most big deals are contested, and the contest is often *not* head-to-head product vs product — it is incumbent inertia, do-nothing, an internal alternative, or a competitor with better stakeholder access. Knowing the real competitive set tells you where to differentiate and where you are actually losing.
- **Classic mistake:** assuming you know the competitor because you know the product ("we beat Vendor X in the bake-off"), while the real competition is the incumbent relationship or the status quo; or ignoring competition until late, by which time the competitor has already shaped the criteria (§5.3).
- **Variant notes:** absent from the original 6-letter MEDDIC; added in MEDDICC; carried into MEDDPICC. ⚠ Its addition date and author are not documented in any primary source verified this pass (§3.2).

### 5.8 Paper Process

- **Verified meaning:** ✅ MEDDICC: "The Paper Process is the series of steps that follow the Decision Process in how you will go from Decision to signature." Practitioner gloss ⚠-knowledge: the internal administrative machinery between a verbal yes and an executed contract — procurement paperwork, legal review, security/risk assessment, vendor onboarding, credit approval, works-council or board sign-off — each with its own owner, queue and failure mode.
- **Why it matters:** the paper process is where deals die *after* they are won in principle — the "deal slipped into next quarter" is almost always a paper-process event, and the "we lost after they said yes" is usually a paper-process competitor ambush (someone's paperwork got expedited). MEDDICC's positioning is explicit: Paper Process is "the step most deals get lost in," and the 8-letter form exists "for complex enterprise sales with legal and procurement in the loop" ✅.
- **Classic mistake:** treating signature as the finish line and ignoring the paper process until the champion asks "where's your vendor onboarding pack?" — by which point the close date has already slipped; or letting the champion carry the paper alone when the seller should be arming them with every artifact the internal process demands.
- **Variant notes:** the distinguishing letter of MEDDPICC; absent from MEDDIC and MEDDICC. For banks and other regulated buyers it is the *largest* letter in the acronym (§11.4).

### 5.9 The Variant Table — MEDDIC vs MEDDICC vs MEDDPICC

| Letter | MEDDIC (6) | MEDDICC (7) | MEDDPICC (8) | Notes |
|---|---|---|---|---|
| M — Metrics | ✅ | ✅ | ✅ | Original letter |
| E — Economic Buyer | ✅ | ✅ | ✅ | Original letter |
| D — Decision Criteria | ✅ | ✅ | ✅ | Original letter |
| D — Decision Process | ✅ | ✅ | ✅ | Original letter |
| P — Paper Process | — | — | ✅ | Added 2005-era (⚠ dating per court summary, §3.3) |
| I — Identify Pain | ✅ | ✅ | ✅ (glossed "Implicate") | Expanded in some vendor lists to include Implications of pain |
| C — Champion | ✅ | ✅ | ✅ | Original letter |
| C — Competition | — | ✅ | ✅ | Added in the MEDDICC expansion (⚠ exact coinage unattributed) |

✅ Fact base for the table: MEDDICC's comparison section, which lists the letters of each variant exactly as above ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)).

## 6. The Component Interplays — the Chains That Close Deals

The letters are not independent boxes. The methodology's power comes from the **chains** between them — the causal pathways through which a well-qualified deal actually closes. Three chains deserve names.

### 6.1 The Champion-Coaching Chain

**Champion → economic-buyer access → decision-criteria shaping.** The champion is the pivot. A champion with power, influence and credibility (✅ MEDDICC's definition) can get the seller in front of the economic buyer — and, critically, can *prepare* that conversation so the EB hears the right message in the right language. That EB access is then spent on the decision criteria: while criteria are still being drafted, the champion (armed by the seller) feeds the evaluation the requirements and weightings that favor the seller's differentiators. Each link feeds the next: no champion → no warm EB access → criteria get written by someone else (often the incumbent or a competitor's champion). This is why §5.6's coach-vs-champion distinction is existential: a coach gives you information but cannot open the EB door or shape the scorecard; a champion does both.

### 6.2 The Paper-Process Chain

**Internal approval paper as the champion's ammunition.** The paper process (§5.8) looks like bureaucracy, but in MEDDPICC practice it is the *champion's sales kit*. Every internal approval step — the business case, the vendor assessment, the risk review, the security questionnaire, the credit application — is a document someone inside the customer must complete and defend. A seller who has mapped the paper process can pre-build each artifact with the champion: the quantified business case (from Metrics and pain), the risk-mitigation answers (from the security/legal conversations), the implementation plan (from the delivery team). The champion then walks into each internal gate with a finished argument rather than a blank form. The chain inverts the naive view: instead of paper being the obstacle after the decision, **the seller who masters the paper process uses it as the mechanism that makes the decision stick** — and as a competitive moat, because the incumbent's paperwork is already done and yours is not (§9.3).

### 6.3 The Pain-Implications-Metrics Chain

**Pain → implications → metrics: the quantification ladder.** The raw material of the whole deal is pain (§5.5). The identify→implicate expansion turns pain into *consequences* ("this costs us, this risks us, this delays us"), and the Metrics letter turns consequences into *numbers the champion can sell internally* ("$2.1M annual float cost, 40% of month-end overtime, three audit findings"). The chain explains why Metrics is listed first in the acronym even though pain discovery happens first in time: **metrics are the pain's final, weaponized form.** A champion cannot win a budget meeting with "the current process is painful"; they can win it with "this project pays back in nine months on float and error-cost savings alone." Every letter downstream of the I in a live deal — the EB's yes, the criteria's weightings, the paper process's business case — is an argument that runs on this quantified fuel. The common failure mode (§9.5) is breaking the chain early: pain identified but never implicated, or implicated but never metered, leaving the champion with a feeling instead of a number.

### 6.4 The Deal as a System

Read together, the chains make MEDDPICC a **system with feedback loops**: the champion (C) is recruited through pain (I) and armed with metrics (M); the champion opens the EB (E) and shapes criteria (D); the criteria and process (D) determine what the paper process (P) will demand; the competition (C) is the external force testing every link. A gap anywhere propagates: missing metrics starve the champion; a missing champion leaves the EB unreachable; an unreachable EB leaves the criteria hostile; hostile criteria make the paper process a formality of defeat. This systemic view is the difference between using MEDDPICC as a checklist and using it as a methodology — and it is the view that deal reviews (§7) are designed to enforce.

## 7. The Qualification Discipline — Deal Reviews and Scorecards

A methodology that nobody enforces is a vocabulary, not a discipline. This section covers the enforcement machinery: deal reviews, scorecards, and the CRM — with the vendor-specific practices flagged.

### 7.1 Evidence-Gated Deal Reviews

The core ritual of MEDDPICC practice is the **deal review gated on evidence**: a recurring session (weekly for the pipeline, deep-dive for stage changes) in which the rep must *demonstrate* each letter rather than assert it. ⚠-knowledge: this is the practice described consistently across the enablement literature (MEDDICC's platform is explicitly built around "Opportunity Manager" deal evaluation; the sibling survey's deal-review one-pager at [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) §4.6 sketches the same artifact). The gate rule has two halves:

1. **Evidence, not adjectives.** "The champion loves us" is not evidence; "the champion, the treasury manager, has pre-circulated our business case to the CFO with her own cover note" is. Each letter maps to a required proof artifact: Metrics → an agreed baseline/target document; EB → a meeting held or scheduled with the named budget owner; Decision Criteria → the draft scorecard or requirements list in hand; Decision Process → a written step list with dates; Paper Process → the named internal approval owners and their packs; Pain → a quantified statement the customer agrees with; Champion → a concrete advocacy act; Competition → named alternatives and their standing.
2. **Stage-gated progression.** A deal may not move to proposal without the D's; may not enter negotiation without the EB's alignment; may not be forecast as commit without the paper process mapped ⚠-knowledge: the exact gate mapping is an organizational policy choice, and firms differ on which letters gate which stages. What is universal is the *form*: the review asks "what do we know, how do we know it, and what is the next evidence to collect" — turning sales management from storytelling into auditing.

✅ The methodology's own framing supports the evidence standard: MEDDICC describes the pre-methodology failure as "guesswork about a deal's outcome" and the post-methodology state as being "aligned on how success is measured," with the letters as "critical information" tracked on every deal ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)). The sibling survey frames the same discipline as the deal-review one-pager — a single page per deal carrying the letters' evidence — which is the artifact this section assumes.

### 7.2 Scorecards and Opportunity Scoring

**Scorecards** operationalize the deal review: each letter is scored (typical scales: red/amber/green, or 0–5/0–10), the scores roll up into an overall qualification grade, and the grade drives pipeline decisions (invest / fix gaps / qualify out) and forecasting weight. ⚠-knowledge with a specific caution: the *scoring rubrics* in circulation are vendor-specific inventions layered on the shared letters — MEDDICC's platform scores qualification gaps and "evaluate deals"; Winning by Design teaches scoring tied to its SaaS metrics; Gong's deal-inspection templates attach MEDDPICC scores to call evidence; and most CRM/Salesforce MEDDPICC field packs define their own 1–5 rubrics. None of these rubrics is part of the 1996 methodology; the original MEDDIC specified letters, not Likert scales. ⚠ Any specific cutoff ("a deal must score ≥7/10 to enter forecast") is an organizational policy, not a MEDDPICC rule — treat vendor scorecard templates as useful starting points and the weights as your own calibration problem.

The evidence for the scorecard practice itself: ✅ MEDDICC's platform materials describe members using the tooling "to evaluate deals, identify gaps in qualification, and apply MEDDICC consistently across their pipeline," which is scoring in all but name ([meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc)); the broader "score every letter, review weekly, gate by evidence" pattern is convergent across the practitioner literature surfaced this pass ⚠-knowledge.

### 7.3 MEDDPICC in the CRM

In practice, MEDDPICC lives in the CRM as a **structured field set on the opportunity record**: one field (or section) per letter, populated with evidence notes and links, feeding pipeline views, forecast reports and manager coaching. ⚠-knowledge: the standard implementation pattern is a Salesforce (or equivalent) opportunity layout with MEDDPICC fields — champion name and advocacy evidence, EB name and last contact, decision criteria attachment, process steps with dates, paper-process owner list, metrics document link, competition picklist — often enforced by making certain fields required at certain stages. Vendor tooling goes further: MEDDICC's mOS platform and call recorder attach conversation evidence to the letters ✅ (per the company's product description), and the enablement-platform ecosystem (Force Management's Opportunity Manager, Gong, Highspot) sells MEDDPICC-shaped deal governance ⚠-knowledge.

Two implementation cautions, both ⚠-knowledge but near-universal in the practitioner literature: (1) **fields without evidence standards rot** — a MEDDPICC section full of adjectives is worse than no section, because it manufactures false qualification; the field should require the artifact link or the specific named fact, not a free-text vibe. (2) **The CRM is the system of record, not the methodology** — firms that succeed treat the CRM fields as the *output* of deal reviews, not the input; the review conversation happens with people in the room, and the rep updates the record afterward.

### 7.4 The Vendor-Specific Practice Caveat

A consolidated honesty note. The *letters* are a shared, court-confirmed-generic public methodology (§3.3); the *practice stack around them* — M1/M2/M3 metrics pedagogy, scoring rubrics, platform fields, certification curricula, "MEDDPICC in 10 minutes" explainers — is commercial product. ⚠ Claims that a specific vendor's practice layer produces measurably better forecasts or faster ramps are marketing: no vendor page verified this pass presented audited, controlled outcome data, and the testimonials that stand in for it (Force Management's Intercom quote; MEDDICC's customer logos) are selection-biased by construction. The honest position: MEDDPICC's disciplines are widely attested to improve *conversation quality and deal transparency* — which is itself the mechanism by which forecasting improves (§8) — but the magnitude is organizational and unmeasured in the public record.

### 7.5 Standing the Discipline Up — the First 90 Days

Adoption is where methodologies die, and the practitioner literature is consistent that MEDDPICC fails when it is announced as a policy rather than installed as a practice ⚠-knowledge. A pragmatic 90-day rollout, synthesized from the practice material in §7.1–7.3 (and the facilitation skills in the [Facilitation Skills Guide](facilitation_skills_guide.md) sibling, which covers running the working sessions this sequence depends on):

1. **Weeks 1–2 — shared language.** Train the team on the letters and the evidence standard (what counts as proof for each letter — §7.1's artifact list). The training can be vendor certification (MEDDICC's Masterclass), an internal session, or both; what matters is that the *evidence standard*, not the acronym, is the message. ⚠ Do not let this become the vendor's pitch: the letters are generic (§3.3); the pedagogy is optional.
2. **Weeks 3–4 — the CRM gets teeth.** Add the MEDDPICC field set to the opportunity record with the evidence-standard help text (§7.3), and make the fields required from the stage where qualification starts. Define the rubric — the score scale, the roll-up, the gate rules — as organizational policy (§7.2), and publish it so "why is this deal a commit?" has a written answer.
3. **Weeks 5–8 — the review cadence bites.** Run weekly evidence-gated deal reviews on the live pipeline (§7.1), with the manager asking the detection questions of §9.6's table. The first reviews will be uncomfortable — that is the point; the discomfort is the old storytelling culture being audited for the first time.
4. **Weeks 9–12 — calibrate and socialize.** Compare the scorecard grades against outcomes starting to come in, adjust the rubric where it mis-ranks, and publish the first "qualified out" wins — deals the discipline killed early that would previously have consumed the quarter. Nothing sells the methodology to a team like watching it save them from a deal that was never real (§1.2's qualification problem, made personal).

⚠-knowledge: the 90-day shape above is this guide's synthesis of convergent rollout advice in the enablement literature (train → instrument → review → calibrate), not a single canonical source; the sibling survey's implementation guidance ([../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) §15) covers organizational adoption at survey depth and is the natural companion read.

## 8. Forecasting — Commit, Upside, and the MEDDPICC Signal

### 8.1 The Commit-vs-Upside Conventions

Enterprise sales forecasting operates on a layered convention that predates MEDDPICC and is general to the industry ⚠-knowledge (it is standard sales-operations practice, taught in every sales-management curriculum and encoded in forecasting tools, though no single canonical source exists):

- **Commit (or "closed/committed")** — the revenue the rep is *accountable* to deliver in the period. A commit number is a personal obligation: the rep is expected to be able to defend it deal by deal, and misses are review events. Conservative by construction.
- **Upside (or "best case")** — revenue that could land but that the rep is not yet willing to be accountable for: deals with gaps, late-stage risks, or unproven timing. Forecasts are typically reported as commit + separate upside, and leadership plans against commit while watching upside.
- **Pipeline** — everything earlier, unweighted or weighted by stage probability; the raw material from which commits and upside are drawn.

The disciplines connect: stage probability weights, deal age, and coverage ratios (pipeline ÷ quota) are the quantitative layer, and the qualitative layer is exactly what a qualification methodology supplies — the judgment that a deal *deserves* its weight.

The forecasting ritual that binds the layers together is the **manager's commit review**: each period, the rep defends every commit deal in front of the manager, deal by deal, and the manager's signature on the commit number is an endorsement of the evidence — not of the rep's optimism. ⚠-knowledge: in well-run enterprise sales organizations the commit review and the MEDDPICC deal review are the same meeting, because both ask the same question — *what do we actually know about this deal?* In banking-wholesale settings the review has an extra audience: the mandate's paper queues (§11.4) mean the commit number is partly owned by operations and credit teams who were never in the sales meeting, so the forecast review is also a cross-functional handshake, not a sales-only ceremony. That is a practice convention of this guide's synthesis, flagged ⚠-knowledge like the rest of §8, but it follows directly from the verified definition of the P letter as the steps "from Decision to signature" — the steps only some of which the seller controls.

### 8.2 What MEDDPICC Adds to a Forecast

MEDDPICC's contribution to forecasting is that it **gives the commit/upside distinction a defensible evidence base** ⚠-knowledge framing, but grounded in the verified component definitions:

- A deal belongs in **commit** only when the evidence threshold holds: the EB is aligned (✅ "person with the overall authority"), the decision criteria are known and winnable, the decision process has a date, and the **paper process is mapped** (✅ MEDDICC: the steps "from Decision to signature") — because in enterprise sales, the difference between "the customer said yes" and "the revenue lands this quarter" is precisely the paper process. A commit without a mapped paper process is a hope.
- A deal belongs in **upside** when the commercial decision is plausible but evidence is missing: no named champion act, criteria unwritten, competition unidentified, or paper owners unknown.
- The **letters act as an early-warning system**: deterioration in any letter (champion leaves, criteria change, a competitor appears in procurement) is a forecast event that should move the deal down before the quarter-end surprise.

⚠ The MEDDICC company's marketing goes further — its flagship use case is literally "Forecast Confidence," claiming the methodology produces "more accurate and efficient communication [leading] to better forecasting" ✅ (that is their stated positioning) — but any *quantified* claim (X% forecast-accuracy improvement) is vendor marketing, and the causal chain (qualification transparency → forecast defensibility) is the mechanism, not a measured guarantee. Cross-reference the repo's sales-adjacent material: the pipeline/coverage mechanics belong to sales-operations practice ⚠-knowledge and are not re-derived here; the sibling survey ([../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md)) treats forecasting only lightly, so this section's conventions rest on standard industry practice rather than a repo sibling.

### 8.3 The Caveats

Three honest caveats on forecasting and MEDDPICC. First, **forecast accuracy is a team property, not a methodology property**: a firm with weak discovery or a culture of optimism will mis-forecast with or without MEDDPICC — the methodology merely makes the weakness visible earlier. Second, **paper-process timing dominates quarter-end risk**: even a fully-qualified deal can slip on a customer's internal legal queue, which is why the P letter — the last one added historically — is often the most important one for forecasting in regulated industries (§11.4). Third, ⚠ the "evidence-gated commit" standard is a *practice norm* synthesized from the enablement literature and this guide's synthesis of the component definitions; no single canonical rulebook was located that says "a commit requires all eight letters green," and organizations legitimately calibrate their own thresholds.

## 9. The Common Failure Modes

The failure modes below are the deal-killers the methodology exists to catch. Each maps to one or more letters, and each is stated here with its signature symptom so it can be recognized in a deal review.

### 9.1 Champion Misidentification — Coach vs Champion

**The failure:** treating a friendly insider as a champion. The distinction (from §5.6): a **coach** tells you what is happening inside (who decides, what they fear, where the landmines are); a **champion** *acts* — they advocate for you in rooms you cannot enter, feed your value into the criteria, and carry your paper through internal gates. ⚠-knowledge: practitioner literature is emphatic that most reps over-estimate their champion coverage precisely because they have never defined the difference; the standard test is behavioral — "what has this person done for us inside, that cost them something, in the last two weeks?" A coach is valuable (they make you smart); a champion is necessary (they make you win). The MEDDICC definition — "power, influence, and credibility within the customer's organization" ✅ — is the benchmark: a true champion has the standing to spend political capital on you.

**Signature symptoms:** "our champion loves us" said without a single concrete advocacy act; the supposed champion has no line to the EB; the champion is the person who likes you, not the person with influence.

### 9.2 The Economic-Buyer Gap

**The failure:** selling to users, IT, or a department head without ever reaching (or even identifying) the person with authority over the money — then losing at the end to an EB-level objection (price, priority, risk) that no one in your coalition could have overruled. The E letter (✅ "the person with the overall authority in the buying decision") is the only letter that cannot be delegated by the customer and cannot be substituted by the seller. ⚠-knowledge: in enterprise deals the EB gap usually coexists with a champion gap — the rep's insider is a user-level enthusiast whose own access to the EB is limited, so the rep is blind at exactly the level where the decision is made. Note the banking variant of this failure is structural: the EB in a bank deal is often two or three levels above the treasury team running the RFP (§11.2), and the gap is the *default* state, not an accident.

**Signature symptoms:** every meeting is with implementers or evaluators; nobody has ever asked the rep about price or budget from a position of authority; the deal "keeps getting delayed" at a stage the insiders cannot explain.

### 9.3 The Missing Paper Process

**The failure:** winning the decision and losing the quarter — or the deal — in the paperwork. The P letter exists because between "the customer's committee chose us" and "the contract is signed and revenue books," there is an internal machine: procurement terms, legal review, security assessment, vendor onboarding, credit approval, entity setup (✅ MEDDICC: "the series of steps that follow the Decision Process… from Decision to signature"). Firms that do not track P treat every late-stage slip as bad luck; firms that track it treat it as a managed pipeline with named owners and dates. ⚠-knowledge: in the vendor literature the P is described as "the step most deals get lost in" (MEDDICC's own phrase ✅), which matches the experience of long-cycle sellers: the competitive battle is often re-fought inside the customer's approval chain, where the incumbent's completed paperwork is an unfair advantage and the challenger's empty onboarding queue is a silent killer.

**Signature symptoms:** the close date keeps slipping in week-sized increments; the rep cannot name the customer's legal reviewer, security assessor or vendor-onboarding owner; "it's just signatures left" has been true for six weeks.

### 9.4 Competition Blindness

**The failure:** misreading the competitive set. Two species: (1) **named-competitor blindness** — assuming you know the rival's product strengths and ignoring their *relationships* (their champion in the customer, their incumbent status, their seat at the criteria table); (2) **status-quo blindness** — the far more common enterprise killer, where the real competition is "do nothing / stay with the incumbent / fix it internally," which never appears in a win-loss report as a named vendor. ⚠-knowledge framing grounded in the verified letter: Competition (added in MEDDICC) exists because deals are lost to *alternatives*, and the alternative is often inaction. The criteria (D) usually encode the real competition: scorecards written around the incumbent's strengths, or around "safe," disqualify challengers before they can differentiate.

**Signature symptoms:** the rep names one competitor and no alternatives; the customer has never mentioned the status quo as an option; the criteria document, if obtained, reads like the incumbent's feature list.

### 9.5 Pain That Is Not Quantified

**The failure:** breaking the pain→implications→metrics chain (§6.3). The team knows the customer "has pain" but cannot state its size, its cost, or its deadline — so the champion has nothing to carry into the budget meeting, the business case in the paper process is written by the customer's finance team *without your numbers*, and the price conversation is a tug-of-war over a number with no anchor. The I letter (✅ Identify Pain; the modern gloss adds "Indicated, and Implicated") and the M letter (✅ "quantifiable measures of value") are a pair: pain is the motive, metrics are the measure, and a deal missing either half runs on sentiment. ⚠-knowledge: unquantified pain also correlates with the other failures — it is hard to recruit a champion for a vague cause, and impossible to justify urgency to an EB who has not been shown the cost of delay.

**Signature symptoms:** the business case is being written by the customer without your input; nobody can say what the problem costs per month; the "why now" answer is a feeling about a deadline rather than a number attached to one.

Unquantified pain has a second, subtler cost beyond the lost argument: it **forces the price down**. When the value case is a feeling, the only number in the conversation is the price — and a price with no value anchor against it is a price to be discounted. The quantified deal (§6.3's ladder complete) gives the negotiator something to defend: "you agree the float and FX leakage is US$3–4M a year; the mandate fee is a fraction of the first year's capture" is a defensible position, while "our platform will save you money" invites "then discount it." This is why the Metrics letter is not a marketing exercise but a pricing and margin instrument — a point the [Business Case Development](business_case_development_guide.md) sibling develops from the buyer's side (how the customer's own business case will value your offer, and what evidence it will demand), which the seller should read to know what the champion will have to submit internally.

### 9.6 The Failure-Mode Table

| Failure mode | Primary letters | Detection question for the deal review |
|---|---|---|
| Champion misidentification (coach vs champion) | C | What has the champion *done* for us inside, at personal cost, recently? |
| Economic-buyer gap | E | Has the budget owner heard our pitch from us, not through a filter? |
| Missing paper process | P | Can we name every internal approval owner between yes and signature? |
| Competition blindness | C (second) | What is the status quo's hold, and who inside is its champion? |
| Unquantified pain | I, M | What number does the customer's own finance team attach to this problem? |

⚠-knowledge: the five failure modes above are the ones consistently named across the practitioner and enablement literature reviewed this pass (MEDDICC's platform copy lists the pre-methodology symptoms as "no visibility on why you win or lose," "guesswork about a deal's outcome," and "not being sure on the next steps" ✅-corroborated); the table's framing is this guide's synthesis for review use.

## 10. The Complementary Methodologies — BANT, SPIN, Challenger

MEDDPICC is one methodology in a crowded field. This section verifies the origins of the three most frequently paired with it, then compares them.

### 10.1 BANT

**Budget, Authority, Need, Timeline** — the oldest qualification shorthand, commonly attributed to **IBM**. ✅ The attribution to IBM is consistent across the sources reviewed this pass, but the *dating* is not: sources variously place its origin in "the 1950s," "the 1960s," or "the 1950s–60s" (prospeo.io, demandnexus.io, realgrowthmatters.com, getgangly.com all attribute it to IBM with dates spanning that range). ⚠ No primary IBM document was located this pass, and none of the secondary sources cite one; the exact year — and even whether IBM originated it or merely popularized it — is ⚠ unverified. The substance is uncontested: four quick filters — does the prospect have Budget, Authority, Need, and Timeline? — designed for fast triage of simple, single-decision-maker sales. Its modern reputation is the foil for MEDDPICC: BANT is seller-centric, shallow on stakeholders, and notoriously gamed ("the customer always says they have budget"). ⚠-knowledge: modern vendor literature (and the sibling survey at [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) §3.1, which covers BANT's replacements FAINT/ANUM/GPCT) treats BANT as a lead-qualification filter, not an enterprise deal methodology — which is exactly the gap MEDDPICC fills.

### 10.2 SPIN Selling

**Situation, Problem, Implication, Need-payoff** — the four question types of consultative discovery, from Neil Rackham's *SPIN Selling* (1988). ✅ The origins are well verified: Rackham himself recounts (Huthwaite International blog, 6 Feb 2026) that he began as a research fellow at Sheffield University studying measurable interactions; a chance question — why do the top 10% of a sales force outsell the bottom 10% by three to one? — turned him to sales research; and **Xerox** funded the initial work, seeking a European sales model, which became a study of Xerox selling in thirteen countries. ✅ The book and the research scale are documented: *SPIN Selling* (McGraw-Hill, 1988) is "the result of the Huthwaite Corporation's 12-year, one million dollar research into effective sales performance" (Google Books/WorldCat publisher summaries), and the ⚠-knowledge "35,000 sales calls studied" figure appears in the standard publicity but was not re-verified word-for-word this pass. The four question types: **Situation** (facts about the customer's world — asked sparingly), **Problem** (difficulties and dissatisfactions), **Implication** (the consequences of those problems — the questions that create urgency), and **Need-payoff** (the value of solving them — questions that get the customer to state the benefit). The research finding: small sales respond to closing techniques, but major sales are won on the question sequence — needs discovery and implication development, not closes.

### 10.3 The Challenger Sale

**The Challenger Sale** (Matthew Dixon and Brent Adamson, CEB — the Corporate Executive Board — 2011) argues that the winning rep profile is not the relationship builder but the **Challenger**: one who *teaches* the customer something new about their business, *tailors* the message to the customer's economics, and *takes control* of the sale. ✅ Publication verified: published 10 November 2011 by Portfolio/Penguin (Wikipedia); the underlying CEB research studied "more than 6,000 sales reps" (PR Newswire release; ~90 companies per multiple secondary sources ⚠-knowledge on the exact company count) and sorted reps into five profiles — Challenger, Hard Worker, Problem Solver, Relationship Builder, Lone Wolf — with Challengers over-represented among top performers, especially in complex sales. ✅ The study's scale claim ("6,000+ reps") is corroborated by the PR Newswire announcement of the research, which is a primary-adjacent source. ⚠-knowledge: CEB was later acquired by Gartner (2017), and the Challenger insights were extended in *The Challenger Customer* (2015); neither was re-verified in detail this pass. The Challenger's commercial teaching — reframing the customer's problem in a way that challenges their status quo — is the message-layer complement to MEDDPICC's evidence-layer discipline.

### 10.4 The Comparison Table

| Dimension | MEDDPICC | BANT | SPIN Selling | The Challenger Sale |
|---|---|---|---|---|
| **Origin / attribution** | PTC, mid-1990s (✅ per MEDDICC: Dunkel 1996, under McMahon, with Napoli; ⚠ creator labels contested) | IBM (✅ attribution; ⚠ exact decade 1950s/60s unverified) | Neil Rackham / Huthwaite, 1988 book out of 12-year research incl. Xerox studies (✅) | CEB — Dixon & Adamson, 2011 book out of 6,000-rep study (✅) |
| **Core focus** | Qualification evidence — know the deal (metrics, people, process, paper, pain, competition) | Lead triage — four quick filters (budget, authority, need, timeline) | Discovery questioning — four question types that build need and value in major sales | Message and profile — teach, tailor, take control; challenge the status quo |
| **Stage fit** | Qualification and deal governance across the whole cycle; strongest mid-to-late | Very early — qualify leads in/out fast | Early-to-mid discovery conversations | Early messaging and executive conversations; positioning |
| **Strengths** | Depth on stakeholders and process; evidence-based forecasting; coachable, auditable | Simple, fast, memorable, universal | Research-backed; turns discovery into a skill; handles the "no visible need" case | Differentiates in commoditized markets; reframes value at the top |
| **Weaknesses** | Heavy for small deals; can become bureaucracy; no message/content guidance | Seller-centric; one-decision-maker assumptions; easily gamed; shallow for complex deals | Demands skilled questioning; no stakeholder/process map; no paper/approval tracking | Research methodology debated ⚠; rep-profile typing can become labeling; light on process mechanics |
| **Relation to MEDDPICC** | — | MEDDPICC is what BANT grows into when the deal is complex: BANT's "budget/authority" become E + P; "need" becomes I + M; "timeline" becomes D-process | SPIN's Implication questions are how you *fill* MEDDPICC's I and M — the questioning technique under the evidence ledger | Challenger's teaching message is what the champion carries; MEDDPICC supplies the stakeholder/process map for it to travel through |

⚠-knowledge note on the comparison: the "strengths/weaknesses" rows synthesize the standard critique in the sales literature (the sibling survey covers each methodology in its own section — BANT §3.1, SPIN §5, Challenger §6 of [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) — and this table deliberately compresses rather than duplicates).

### 10.5 How They Fit With MEDDPICC

The four are complementary layers, not rivals — a point the sibling survey makes for the general stack and this guide applies to MEDDPICC specifically:

- **BANT** is the *triage layer*: at the top of the funnel it filters inbound noise in minutes. MEDDPICC takes over when the deal is real enough to matter — BANT answers "should we talk?", MEDDPICC answers "should we bet on this?"
- **SPIN** is the *discovery technique*: MEDDPICC tells you which evidence to collect; SPIN's Situation/Problem/Implication/Need-payoff questions are how you collect it. The overlap is visible inside the framework itself — MEDDPICC's modern "Implicate the Pain" gloss (✅ per MEDDICC) is SPIN's Implication stage wearing MEDDPICC's letters (see §5.5), and MEDDICC publishes its own "SPIN Selling vs MEDDPICC" comparison article ⚠-vendor-content.
- **Challenger** is the *message layer*: the teaching pitch that reframes the customer's economics is exactly the content a champion needs to carry and an EB needs to hear; MEDDPICC supplies the map of who must hear it, when, and through which paper gates.

The integration pattern used by practitioners ⚠-knowledge: qualify with MEDDPICC, discover with SPIN questions, message with Challenger-style commercial teaching, triage the top of funnel with BANT — one methodology per job, with MEDDPICC as the spine because it is the only one that tracks the deal's evidence across the full cycle into the forecast.

## 11. The Banking-Wholesale Application

MEDDPICC transfers to wholesale banking with remarkably little modification — because a bank's corporate client buying banking services is structurally the same animal as an enterprise buying software: a multi-stakeholder committee, an RFP-driven evaluation, a long cycle, and a heavy internal approval machinery. This section condenses the application; the transaction-banking product substance it presupposes lives in the sibling banking guides, cross-referenced rather than re-derived.

### 11.1 The Shape of a Wholesale-Banking Deal

A wholesale-banking deal — a corporate group awarding its cash management, payments, trade or supply-chain-finance mandate to a bank — has the canonical MEDDPICC shape: high value (annual fee pools plus balances), multi-stakeholder (treasury, finance, procurement, IT/operations, risk, legal), RFP-driven (published scorecards, fixed timelines), and brutally long (six to eighteen months from first touch to boarding). ⚠-knowledge: the six-to-eighteen-month cycle and the committee structure are standard transaction-banking experience; the repo's banking siblings describe the product estates — [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) for the payments platform the client is evaluating, [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md) for trade and SCF — while the mechanics of running a formal selection are covered in the [Vendor Management Guide](vendor_management_guide.md) (§3, "Vendor Selection and Sourcing": RFI → RFP → RFQ → tender, selection criteria, the selection decision), seen from the buyer's side. The seller's-side discipline for the same process is exactly MEDDPICC.

What differs from software selling is not the framework but the *content of the letters*: the "product" is a regulated balance-sheet relationship (credit, deposits, liquidity), the "champion" is usually a treasury manager with real but bounded influence, the "economic buyer" is often two levels above the evaluation team, and the "paper process" is not one queue but three parallel ones — the client's own vendor-onboarding, the client's credit/risk review of the bank as a counterparty, and the bank's own internal approvals (credit, legal, onboarding, compliance) that must clear before the bank can even sign. MEDDPICC's P letter, built for "legal and procurement in the loop," earns its keep here more than anywhere.

### 11.2 The Economic Buyer in a Bank Deal

In a corporate banking RFP, who is the economic buyer? The clean answer: **the person who owns the money and whose signature commits the group** — in most regional-corporate structures, the **Group CFO** or the **Group Treasurer** with delegated budget authority, *not* the procurement lead running the tender and *not* the treasury manager chairing the evaluation. ⚠-knowledge: the treasury/procurement split is the classic EB trap of banking deals. Procurement owns the *process* (the RFP, the scorecard, the negotiation, the terms) but rarely owns the *economic decision* — the mandate's value (fee pool, balance-sheet cost, working-capital benefit) is a finance/treasury decision, and the risk/relationship considerations reach the CFO and sometimes the board or group finance committee. The seller who treats the procurement manager as the EB is running §9.2's economic-buyer gap inside a bank deal; the seller who maps the real authority — often the Group CFO for a regional group, the Group Treasurer for a cash-rich multinational — can shape the finance-level criteria that procurement then encodes in the RFP.

Two banking-specific wrinkles worth stating. First, **the bank is also a buyer of risk**: the client's treasury evaluates the bank's creditworthiness, so the "EB" conversation includes the client's risk function assessing *your* bank — an inversion software sellers rarely face (the vendor is not usually credit-reviewed by the customer). Second, **relationship seniority matters**: in wholesale banking the real EB conversation is often banked at the country/regional head or the coverage MD level, meeting the client CFO peer-to-peer, while the RFP itself is run at the treasury-manager level. MEDDPICC maps this cleanly: the EB letter names the CFO/Group Treasurer; the champion letter names the treasury manager; the two are connected by the champion-coaching chain (§6.1), and the coverage senior's job is to work the EB channel the champion opens.

### 11.3 Decision Criteria and the RFP Scorecard

In banking the Decision Criteria letter is unusually *tangible*: the RFP scorecard. ✅-knowledge framing: an RFP evaluation matrix with weighted criteria — functional coverage of the required products (payments, collections, liquidity, trade), technical fit (connectivity, file formats, APIs, ISO 20022 readiness), pricing (fee schedule, FX margins, deposit rates), service and implementation model, risk and stability (ratings, regulatory standing), and reference/incumbent status — is the published face of the criteria. The MEDDPICC discipline applies in two moves:

1. **Discover the unpublished criteria.** The published scorecard never contains everything: incumbent inertia ("why change banks?"), relationship comfort, the credit appetite of the bank for the group's facilities, and the internal preferences of individual committee members are all real criteria that never appear in the RFP document. The champion and coaches are the source for these (see the stakeholder-mapping methods in the [Communication & Stakeholder Management Skills](communication_stakeholder_management_skills_guide.md) sibling, which covers power/interest mapping and influence without authority).
2. **Shape the criteria before they freeze.** The window between "the group decides to tender" and "the RFP is published" is where criteria get written. A bank with a champion inside (the treasury manager who wants Cymbal Bank's solution, §12) can feed the requirements and weightings — API-based integration, multi-entity visibility, SCF capability — that match its strengths, exactly the decision-criteria-shaping chain of §6.1. Once the RFP is out, the scorecard is a scoring exercise; before it is out, it is a persuasion exercise.

Cross-reference: the [Vendor Management Guide](vendor_management_guide.md) §3 documents the RFP mechanics from the issuer's side (RFI → RFP → shortlist → selection), and its §3.2 on selection criteria shows how buyers weight and score — the seller reading this guide should read that section to see the scorecard from the other side of the table.

### 11.4 Paper Process — Credit, Onboarding, Legal

The P letter is where banking deals are won and lost after the mandate decision. In a bank deal the paper process is genuinely three interlocking machines ⚠-knowledge (standard industry structure, described here from experience and the vendor-management sibling's treatment of bank onboarding):

1. **The client's own vendor-onboarding** — the corporate's supplier/vendor management process for adding a new bank: security questionnaires, data-protection review, entity setup in their ERP/payment systems, authorized-signatory updates, treasury-system connectivity forms. This is the mirror of the supplier-onboarding machinery the [Vendor Management Guide](vendor_management_guide.md) describes from the buyer side (its §5 covers onboarding: KYC-style data collection, contracts, SLAs, access).
2. **The client's credit/risk review of the bank** — the corporate's treasury policy typically requires counterparty approval of the bank itself: ratings review, concentration limits, board/treasury-committee approval for a new banking relationship. A mandate can be won commercially and die here if the bank's rating or the relationship size trips a policy limit.
3. **The bank's own internal approvals** — before Cymbal Bank can sign, its own machinery must clear: credit approval for any facilities, legal review of the mandate documents, client-onboarding/KYC (including the AML/sanctions screening that is the subject of the repo's compliance guides ⚠-knowledge), product-approval and pricing sign-offs, and implementation/boarding resource planning.

The MEDDPICC move is the same as §6.2: map every queue, name every owner, and pre-build every pack — the security answers, the credit pack, the onboarding forms — so that when the client says yes, the paper is already 80% assembled. In banking, the seller who treats "they chose us" as the end of the sale loses the quarter to the paper; the seller who treats the paper as a parallel sale manages it like one.

### 11.5 The Multi-Stakeholder Committee

Banking RFP evaluation committees are textbook MEDDPICC stakeholder maps. Typical membership ⚠-knowledge: **Group Treasury** (the daily users and often the champion's home), **Group Finance/CFO office** (economics, and the EB channel), **Procurement** (process and commercial terms), **IT/Operations** (integration, file formats, connectivity, STP rates), **Risk/Compliance** (counterparty and data risk), and sometimes **Tax/Legal** (cross-border structures, documentation). Each has a veto and a different criterion: treasury cares about visibility and control; finance about price and balance-sheet; procurement about contract terms and process compliance; IT about integration effort and reliability; risk about the bank's standing and the data flows. The repo's stakeholder-mapping content in [Communication & Stakeholder Management Skills](communication_stakeholder_management_skills_guide.md) (power/interest grid, §5.1 of that guide) is the tool for this map; MEDDPICC supplies the deal-level structure: who is the champion in this committee, who is the EB behind it, what criteria does each member carry into the scorecard, and whose paper gates the signature.

### 11.6 Cross-References to the Sibling Guides

- [Vendor Management Guide](vendor_management_guide.md) — §3 (RFP/sourcing process, selection criteria) and §5 (onboarding) describe the buyer-side machinery this guide's banking section works against; read together they give both sides of the same transaction.
- [Ancillary Revenue Products Guide](ancillary_revenue_products_guide.md) — a sales-adjacent product deep-dive with a Cymbal Bank commercial worked example; its structure (concept → economics → mechanics → worked example) is the model this guide's §12 follows, and its treatment of how a bank packages and prices product value feeds the Metrics letter.
- [Communication & Stakeholder Management Skills](communication_stakeholder_management_skills_guide.md) and [Facilitation Skills Guide](facilitation_skills_guide.md) — the stakeholder-mapping, influencing and workshop skills underneath the champion/EB work and the deal-review cadence (§7).
- [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) and [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md) — the transaction-banking product estates the worked example's RFP is about.
- [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) — the general survey this guide deep-dives; its §13 (banking/financial-services context) is the survey-level companion to §11 here.

### 11.7 The Banking Timeline Map — Letters Across an RFP Cycle

A condensed map of how the letters load onto a wholesale-banking RFP timeline — the seller's answer to "what should we know, and when?" for a deal shaped like §12's. The phases are the buyer's own process (per the [Vendor Management Guide](vendor_management_guide.md) §3 RFI → RFP → shortlist → selection sequence); the letters column is what MEDDPICC demands the seller have in hand at each point. ⚠-knowledge synthesis (phase timing is typical, not universal; the letters' loading order reflects how banking evidence actually accumulates):

| Buyer's phase (typical timing) | Letters loading | The evidence the file must hold by phase-end |
|---|---|---|
| Pre-tender / relationship (months −6 to −3) | I, M, C | Pain identified and quantified with the treasury contact; first champion candidate confirmed by an advocacy act |
| Tender announced, RFI/RFP drafting (months −3 to −1) | D (criteria), E | Unpublished criteria mapped via champion/coaches; EB named; criteria-shaping done before the RFP freezes |
| RFP issued, written responses (month 1–2) | D (process), C (competition) | Published scorecard in hand; decision process step list with dates; competitors named incl. incumbent and status quo |
| Shortlist, demos, site visits (month 3) | C, M (refresh) | Champion actively carrying the demo narrative; metrics refreshed and defended in the demo workstream |
| Commercial negotiation (month 4) | E, D | EB (CFO/Group Treasurer) engaged peer-to-peer; commercial criteria and pricing anchors agreed at EB level |
| Decision committee → mandate (month 5) | P | Paper process mapped with named owners; first packs (security, credit, onboarding) pre-built |
| Signature, boarding, implementation (month 6–9) | P (execution) | All three paper queues (§11.4) cleared; KYC/onboarding across entities; revenue books per the mandate letter |

Two readings of the map. First, it makes the *early* letters (I, M, C) the ones that must exist before the RFP is even published — which is why banking coverage teams that treat the RFP as the start of the sale have already lost the criteria-shaping window (§11.3). Second, it shows the P letter spanning the widest calendar arc: it starts loading at month 5 but executes through month 9, which is exactly why the worked example's scorecard (§12.3) refuses to call the deal a commit while P scores 2.

## 12. The Cymbal Bank Worked Example — Qualifying a Transaction-Banking RFP

### 12.1 The Scenario

**Cymbal Bank** — the Singapore-based bank persona used throughout this repository — is a mandated transactional bank for mid-market and regional corporates across ASEAN, with a transaction-banking platform spanning payments, collections, liquidity and trade (the product estate described in the [Payments Hub Guide](../banking/payments_hub_guide.md) and [Supply Chain Finance Technologies Guide](../banking/supply_chain_finance_technologies_guide.md)). The deal: **Meranti Group**, a regional consumer-goods conglomerate headquartered in Kuala Lumpur with subsidiaries in Singapore, Indonesia, Thailand and Vietnam — roughly 40 legal entities, 25 banking relationships, and a treasury function that has run out of patience with fragmented visibility and manual reconciliation. Meranti has issued an **RFP for a regional cash-management and payments mandate**: consolidated multi-entity account structure, regional payments and collections, liquidity sweeping, host-to-host/API connectivity, and a trade/SCF component for its supply chain. Three banks are invited: Cymbal Bank and two regional competitors (one of which, **Meranti's incumbent relationship bank in Malaysia**, holds the largest single share of the group's balances today). Evaluation runs over five months, with a decision committee of treasury, finance, procurement, IT and risk. Cymbal Bank's coverage team has a warm entrée — Cymbal Bank already runs Meranti's Singapore payroll and one subsidiary's collections — but the mandate is contested and the incumbent has inertia on its side. The relationship manager opens the MEDDPICC qualification file in week one.

### 12.2 The MEDDPICC Map — Letter by Letter

**M — Metrics.** The qualification file's first entry is the quantified case, built jointly with Meranti's regional treasury manager. The pain, metered: Meranti runs ~85,000 cross-border transactions a year across 25 bank portals; treasury staff spend an estimated 60% of month-end on reconciliation; the group carries an estimated US$40–60M of idle subsidiary balances that liquidity sweeping could concentrate; FX conversion happens at subsidiary level at unmanaged retail-style margins, estimated to cost US$1.8–2.4M a year versus a consolidated approach; and the SCF programme could release an estimated US$30M of supplier-chain working capital. The M-file records current state, target state and the gap, agreed in writing with the treasury manager: **"consolidate to one regional structure; cut reconciliation effort ~50%; concentrate and sweep balances; centralize FX; launch SCF in year one."** These numbers become the ammunition for every later letter — the champion's business case, the EB's payback conversation, the RFP pricing justification.

**E — Economic Buyer.** Two candidates present themselves; the file names the real one. The **Group CFO** (Singapore-based, group finance committee chair) owns the mandate's economics and the bank-relationship policy; the **Group Treasurer** (Kuala Lumpur) runs treasury day-to-day and chairs the treasury workstream. Procurement runs the process but does not own the economic decision. The file's EB entry: **the Group CFO**, with the Group Treasurer as the finance-side co-owner — and the coverage MD's job is defined as the peer-level CFO channel, opened through the champion (§6.1), with the conversation framed in the CFO's language: balance-sheet efficiency, counterparty-risk policy (Cymbal Bank's Singapore regulation and rating vs the policy thresholds), and the group's bank-reduction strategy (the CFO has publicly wanted "fewer, deeper" bank relationships — a criterion that favors Cymbal Bank's regional scope over single-country banks).

**D — Decision Criteria.** The file tracks both scorecards. The *published* RFP matrix (weighted): functional coverage of payments/collections/liquidity (30%), technology and integration — APIs, ISO 20022, host-to-host (20%), pricing and commercial terms (20%), implementation and service model (15%), risk and stability (10%), sustainability/ESG credentials (5%). The *unpublished* criteria, gathered via champion and coaches: incumbent comfort in Malaysia (the current bank's regional treasurer relationship), the CFO's bank-reduction agenda, IT's fear of integration effort, and risk's counterparty-policy limits. The shaping move: before the RFP froze, the champion fed the treasury workstream Cymbal Bank's differentiators — API-first connectivity, a single regional platform, SCF embedded — so the published weightings (20% on technology, and the SCF scope item) reflect strengths Cymbal Bank actually has. (See §11.3 for why this window matters.)

**D — Decision Process.** The mapped steps, with dates, in the file: (1) RFP issued (Month 1) → (2) written responses + scoring by treasury/procurement (Month 2) → (3) shortlist of two + solution demos to the treasury/IT workstream (Month 3) → (4) pricing negotiation and commercial terms with procurement (Month 4) → (5) reference calls and risk review → (6) decision committee recommendation to the Group CFO (Month 5) → (7) board/treasury-committee ratification and mandate letter. Known gates: IT's integration assessment feeds the shortlist; risk's counterparty review feeds the final recommendation. The file flags the two dates that matter for forecasting: the **decision date** (Month 5 committee) and the **signature date** (Month 6–7, after paper — see P).

**P — Paper Process.** The largest section of the file, with three queues and named owners (§11.4): **Meranti's own vendor-onboarding** — security questionnaire (Cymbal Bank's ISO 27001 and MAS-regulated posture pre-answers most of it), data-protection review, ERP/treasury-system connectivity forms, signatory updates across 40 entities; **Meranti's counterparty-risk review of Cymbal Bank** — rating check against treasury policy, concentration limits, treasury-committee approval of a new regional banking relationship; **Cymbal Bank's internal approvals** — credit approval for the SCF facility line, legal review of the mandate and cross-border documentation, KYC/onboarding across the 40 entities (with the AML/sanctions screening the repo's compliance guides cover ⚠-knowledge), pricing sign-off, and implementation resource planning. The pre-build starts in Month 2, not after the win: the file tracks each pack's owner, status and due date so that a Month-5 yes can become a Month-6 signature.

**I — Identify Pain (and its implications).** The I-file is the most heavily evidenced. Identified: fragmented visibility (25 portals), manual reconciliation, idle balances, unmanaged FX, supplier cash strain. Indicated/implicated, with the treasury manager's own numbers: the reconciliation hours (60% of month-end), the float cost of idle balances (estimated US$1.5M a year at prevailing rates), the FX leakage (US$1.8–2.4M), and the working-capital cost of the unmanaged supply chain. The deadline that makes it urgent: Meranti's regional ERP upgrade (Month 9) forces the connectivity decision anyway, and the CFO's bank-reduction target gives the mandate a "why now" with a name and a date. The pain is not Cymbal Bank's thesis about Meranti; it is Meranti's own numbers, written into the file with the treasury manager's agreement.

**C — Champion.** The file names the champion and the evidence: **the regional treasury manager** (Kuala Lumpur) — the daily victim of the fragmentation pain, the author of the internal "why we must consolidate" note, and the person who pre-circulated Cymbal Bank's regional-structure proposal to the Group Treasurer *with her own cover note* — an advocacy act that cost her effort and reputation capital. She has power (runs the treasury workstream), influence (credible with the Group Treasurer and IT), and credibility (she wrote the pain memo the CFO quoted). She is not merely a coach: she has acted. The file also names the coaches (the IT integration lead who leaks the demo shortlist criteria; the procurement analyst who flags the commercial-term thresholds) and is explicit that coaches inform while the champion advocates (§9.1).

**C — Competition.** The file names three competitors, not one: **the Malaysian incumbent** (holds the largest balances; its champion is the regional treasurer of the legacy relationship; its weapon is inertia and the cost/risk of switching); **the second invited regional bank** (strong pricing, weaker platform; its weapon is the commercial terms); and — the one that never appears in the RFP — **the status quo** ("do nothing, fix it with the ERP upgrade alone"), which is the CFO's fallback if the business case does not clear the payback bar. The file's competitive read: the incumbent is beatable on the criteria Cymbal Bank helped write (regional platform, API, SCF), and the status quo is beatable only by the quantified M/I case — which is why the metrics work is the competitive strategy, not an admin chore.

### 12.3 The Qualification Scorecard

The weekly deal review scores the file (per §7.2's practice; the rubric is Cymbal Bank's own calibration, ⚠ not a MEDDPICC rule). A representative mid-cycle snapshot:

| Letter | Score (0–5) | Evidence on file | Gap / next action |
|---|---|---|---|
| M — Metrics | 5 | Agreed baseline/target document, signed off by treasury manager | Maintain; refresh FX numbers quarterly |
| E — Economic Buyer | 3 | CFO identified; MD-level meeting scheduled Month 3 | Secure the CFO session via champion; prepare payback case |
| D — Decision Criteria | 4 | Published RFP matrix + unpublished criteria mapped | Monitor weighting changes at shortlist |
| D — Decision Process | 4 | Full step list with dates and owners | Verify shortlist date with procurement coach |
| P — Paper Process | 2 | Queues mapped; packs not yet started | **Pre-build security + credit packs in Month 3** — the weakest letter |
| I — Identify Pain | 5 | Quantified, agreed, with deadline (ERP upgrade) | — |
| C — Champion | 4 | Advocacy act documented (cover note to Group Treasurer) | Deepen: champion to brief IT lead before demo |
| C — Competition | 3 | Three competitors named with weapons | Incumbent watch: track its pricing response |

The gate decision from the review: the deal may proceed to the demo stage (D's and I strong), but it **may not be forecast as commit** until the P score clears 4 — because in a banking mandate the paper queues, not the committee, determine the quarter (§8.2). The file's overall grade: *winnable, with a paper-process gap to close and the EB meeting still to land* — which is exactly the kind of sentence an evidence-gated review is supposed to produce.

### 12.4 The Deal Review

The fortnightly review conversation, in the discipline of §7.1, goes: *What do we know? How do we know it? What is the next evidence to collect?* The coverage MD challenges the champion claim ("what has she done this fortnight?" — answer: she circulated the demo agenda to the IT workstream with Cymbal Bank's integration approach embedded); the product lead defends the criteria read ("how do we know SCF stays in scope?" — the champion's note to the Group Treasurer); the file's P section gets the hard question ("who owns Meranti's counterparty review, and have we sent the rating pack?"). The output of each review is not a feeling about the deal but a dated list of evidence to collect — the demo-date confirmation from the procurement coach, the CFO meeting locked, the security pack sent. When the committee eventually recommends Cymbal Bank in Month 5, the file's P-queues are already 70% pre-built, and the mandate letter becomes a Month-6 signature — a commit that lands in the quarter it was promised, because the qualification file said it would.

### 12.5 The Lessons

Five takeaways from the worked example, each tracing to a section of this guide: (1) **the metrics case is the strategy** — every other letter ran on the agreed numbers (§6.3); (2) **the EB was two levels above the evaluation team**, and reaching her was the champion's job, not the rep's cold call (§11.2); (3) **criteria were shaped before the RFP froze**, turning the published scorecard into an ally (§11.3); (4) **the paper process was treated as a parallel sale** with named owners and pre-built packs — the difference between a Month-5 win and a Month-6 signature (§11.4); and (5) **competition included the status quo**, and the CFO's bank-reduction agenda was a criterion the file exploited rather than a risk it feared (§9.4). The same file structure applies to any Cymbal Bank wholesale deal — trade finance mandates, regional treasury consolidations, SCF programmes — because the shape of the deal, not the product, is what MEDDPICC qualifies.

### 12.6 The Anti-Example — the Same Deal, Unqualified

The mirror image of §12.2–12.4 shows what the Meranti mandate looks like when the file is filled with adjectives instead of evidence — a cautionary walk through §9's failure modes in one deal. **M:** the team quotes platform features ("ISO 20022, API, host-to-host") and never agrees the float and FX baseline with the treasury manager — so no number anchors the price discussion. **E:** the relationship manager treats the procurement lead as the decision-maker (she runs the RFP, after all) and never meets the Group CFO — so the commercial negotiation happens against a procurement benchmark, not the CFO's bank-reduction economics. **D:** nobody learns that IT's integration fear is an unwritten criterion until the demo day, and the incumbent's regional treasurer has already fed the scorecard's "relationship stability" weighting — the criteria freeze hostile. **D:** the team maps its own sales steps (proposal → demo → negotiation) but not Meranti's gates, and is surprised when risk's counterparty review — a gate the file never recorded — delays the recommendation by a month. **P:** the packs start only after the committee's yes; Meranti's vendor-onboarding queue and Cymbal Bank's own credit approval for the SCF line each take six weeks, and the mandate letter slips two quarters — the deal "wins" in month 5 and books in month 12, missing two forecast cycles. **I:** the pain is acknowledged ("treasury is frustrated") but never implicated with the treasury manager's own numbers, so the business case inside Meranti is written by finance *without* Cymbal Bank's input and comes out lukewarm. **C:** the friendly IT integration lead — a coach — is logged as the champion; nobody notices that no insider has spent any political capital, and the incumbent's champion quietly wins the hallway. **C:** the competitive file names the second invited bank only; the incumbent's inertia and the CFO's do-nothing fallback are invisible until the loss. The post-mortem writes the same sentence §1.2 predicted: the deal was lost to ignorance — of the money owner, the real criteria, the paper queues and the status quo — not to a better competitor. The discipline of §12.2 exists precisely so that post-mortem never has to be written.

## 13. The Claims Audit

### 13.1 Verified This Pass

| Claim | Source |
|---|---|
| MEDDIC was created inside PTC in 1996 by Dick Dunkel, working under SVP John McMahon alongside Jack Napoli; it came out of a win/loss/slip study when the sales org reached ~300 reps | [meddicc.com/resources/who-created-meddic](https://meddicc.com/resources/who-created-meddic) |
| First-person account: MEDDIC is "McMahon's sales brain codified," Dunkel "was the first to write it down," Dunkel and Napoli taught it at PTC; witnessed by Monica Evans | [salesmeddic.com/blog/origin-of-meddic](https://www.salesmeddic.com/blog/origin-of-meddic) |
| Andy Whyte was NOT a PTC-era creator: his career ran 2001–2020 (DJ → door-to-door → ECI → ES Tech Group → Oracle → Sprinklr [learned MEDDPICC from Dunkel] → Tealium → Poq → Branch), founding MEDDICC the company in 2020 | [meddicc.com/resources/the-backstory-of-andy-whyte](https://meddicc.com/resources/the-backstory-of-andy-whyte) |
| MEDDICC (variant) added Competition; MEDDPICC added Paper Process; letter order of MEDDPICC is M E D D P I C C | [meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc) |
| MEDDICC's per-letter definitions: Metrics "quantifiable measures of value," EB "person with the overall authority," DC "principles, guidelines and requirements," DP "steps the buyer will take," PP "from Decision to signature," Implicate the Pain = "Identified, Indicated, and Implicated," Champion "power, influence, and credibility" | [meddicc.com/what-is-meddpicc](https://meddicc.com/what-is-meddpicc) |
| Dick Dunkel co-founded RevCentric Partners (with David Boyle, "first MEDDIC instructor") and later joined MEDDICC as CSO | [revcentricpartners.com/about](https://revcentricpartners.com/about); [meddicc.com](https://meddicc.com/resources/meddicc-welcomes-dick-dunkel) |
| John McMahon: five-time CRO (PTC, GeoTel, Ariba, BladeLogic, BMC), author of *The Qualified Sales Leader*, board roles incl. Snowflake/MongoDB | [echai.ventures](https://echai.ventures/gtm/people/john-mcmahon); [forcemanagement.com](https://www.forcemanagement.com/interview-john-mcmahon); [sellinglikeachampion.com](https://www.sellinglikeachampion.com/johnmcmahon-1) |
| April 21, 2026: US District Court (E.D. Pa., Civil Action No. 24-1836) ruled MEDDPICC generic, ordered cancellation of Reg. No. 6,489,058 (Lahoutifard / 01 Consulting, dba MEDDIC Academy); court: MEDDPICC coined 2005 by former PTC employees | [meddicc.com (ruling release)](https://meddicc.com/resources/meddpicc-trademark-cancelled-federal-court-ruling) ⚠ single-party account |
| Force Management offers MEDDICC as its qualification approach, paired with Command of the Message; client testimonials on page | [forcemanagement.com/offerings/meddicc](https://www.forcemanagement.com/offerings/meddicc) |
| Winning by Design created SPICED, Bowtie, REKS; SaaS Sales Method book (van der Kooij & Pizarro) | [winningbydesign.com](https://winningbydesign.com) |
| SPIN Selling origins: Rackham's own account — Sheffield research fellow; Xerox-funded research for a European sales model; studied in 13 countries; book = "12-year, one million dollar research" (1988, McGraw-Hill) | [huthwaiteinternational.com](https://www.huthwaiteinternational.com/blog/origins-of-spin-selling); Google Books/WorldCat summaries |
| The Challenger Sale: published Nov 10, 2011 (Portfolio/Penguin); CEB study of 6,000+ sales reps; five profiles | [Wikipedia](https://en.wikipedia.org/wiki/The_Challenger_Sale); [PR Newswire](https://www.prnewswire.com/news-releases/the-rise-of-the-challenger-sale-corporate-executive-board-research-confirms-the-demise-of-relationship-selling-133831648.html) |
| BANT attributed to IBM | Multiple secondary sources (prospeo.io, demandnexus.io, realgrowthmatters.com, getgangly.com) — attribution consistent, dating not (see 13.3) |

### 13.2 Flagged or Approximate (⚠ and ⚠-knowledge)

- **⚠ Who "created" MEDDIC:** MEDDICC's account says Dunkel (with Napoli, under McMahon); Evans splits "McMahon's brain / Dunkel's pen"; third-party profiles call McMahon "creator/originator." All three accounts are commercially interested or secondary. Corroborated core: PTC, mid-1990s, six letters, Dunkel's pen, McMahon's sponsorship (§2.3–2.4).
- **⚠ The "Whyte coined MEDDIC" claim:** contradicted by Whyte's own backstory; treated as secondary-source conflation of the MEDDICC company brand with the PTC origin (§2.3).
- **⚠ Exact coinage of the MEDDICC/MEDDPICC additions:** no primary source names who appended Competition or Paper Process, or when (beyond the court summary's "coined in 2005" for MEDDPICC); "championed by McMahon" plausible but unverified (§3.2).
- **⚠ MEDDICC company marketing figures** ("25,000+ certified," "2,500+ teams," "500+ organizations," "80,000+ books sold," customer logos) and Force Management's testimonial ROI claims — vendor marketing, not audited (§4.1–4.2, §7.4).
- **⚠ M1/M2/M3 metrics pedagogy and the "Implicate the Pain" gloss** are MEDDICC-company teaching, not the 1996 original (§5.1, §5.5).
- **⚠-knowledge:** Gong's MEDDPICC tooling specifics; SalesHacker archive specifics; CEB's ~90-company count; the "35,000 sales calls" SPIN publicity figure; the 2021 dating of *The Qualified Sales Leader*; Winning by Design's MEDDPICC course pages; scorecard rubrics as organizational policy (§4.3–4.4, §7.2, §10.2–10.3).
- **⚠-knowledge:** commit/upside forecasting conventions, banking deal cycles and committee structures, onboarding/credit paper structure — standard industry knowledge not re-verified against a single primary source this pass (§8, §11).

### 13.3 Could Not Verify or Conflicting

| Claim | Status |
|---|---|
| Exact year/decade of BANT's origin at IBM (1950s vs 1960s; IBM origin vs popularization) | ❌ conflicting secondary datings; no primary IBM document located (§10.1) |
| Gong's own MEDDPICC product content and adoption metrics | ❌ not directly verified this pass (search results pointed to third-party pages) (§4.3) |
| The identity of the individual(s) who coined the MEDDPICC name in 2005 | ❌ not named in any source verified (§3.2) |
| Whether John McMahon operates a formal training company | ❌ could not verify (§4.1) |

## 14. What Could Not Be Verified

This section collects, honestly, what this pass could not pin down despite live web access — each item is either single-sourced, commercially interested, or simply absent from the accessible record:

1. **The "Andy Whyte coined MEDDIC" claim.** The task's working assumption — that Whyte is commonly credited with coining the MEDDIC acronym at PTC — could not be verified against any primary source; Whyte's own published biography contradicts it (he was not in enterprise software until 2006 and not at PTC at any point), and the two deepest origin accounts (MEDDICC's history page and Monica Evans's first-person essay) do not mention him as a creator. Secondary blogs repeating the tripartite "Dunkel, McMahon and Whyte" origin story exist, but none surfaced this pass with evidence behind it. The most defensible reading: the conflation happened because Whyte's company brands itself MEDDICC — but this guide cannot prove the mechanism, only document the contradiction.
2. **The precise date of MEDDIC's creation.** 1996 rests on MEDDICC's single (commercial) account; Evans gives no year; the 2026 court summary says "early 1990s." Mid-1990s is safe; 1996 specifically is single-sourced.
3. **Who appended the C (Competition) and the P (Paper Process), and when.** No primary source names the individuals. The court summary dates MEDDPICC's coinage to 2005 by former PTC employees but does not name them, and it is a one-party account (MEDDICC won the case). The "McMahon championed MEDDICC" attribution is plausible but unproven.
4. **The 2026 trademark ruling's details beyond MEDDICC's own release.** The docket (Civil Action No. 24-1836) was not independently checked; the release could overstate or spin details, though the core outcome (generic term, cancellation ordered) matches the observable marketplace.
5. **BANT's exact origin.** IBM attribution is consistent across secondary sources; the decade (1950s vs 1960s) is not, and no primary IBM document was found. Whether IBM originated or merely popularized BANT is unresolved.
6. **Gong's MEDDPICC product specifics and adoption metrics**, and the SalesHacker article archive — searches this pass returned third-party pages rather than the primary content.
7. **The "35,000 sales calls" SPIN publicity figure** and CEB's exact company count (~90) — standard publicity numbers not re-verified word-for-word.
8. **Whether John McMahon runs a formal training company** — his book, podcast appearances and board roles are documented; a McMahon-branded training operation was not found.
9. **Any controlled evidence that MEDDPICC adoption improves forecast accuracy or win rates.** The vendor literature asserts it; none of it publishes audited, controlled outcomes. This is a genuine gap in the public record, not a criticism of the methodology — the same is true of every sales methodology's ROI claims.

None of these gaps undermines the corroborated core of the guide (§2.4): PTC, mid-1990s, a win/loss/slip study, six letters, Dunkel's pen, McMahon's sponsorship, Napoli's evangelism, and a 2020-founded company that turned the extended 8-letter form into a global training brand.

## 15. The Glossary, the Sources, and the Closing

### 15.1 The Glossary

- **Champion** — a person inside the customer with power, influence and credibility who actively advocates for the seller's solution; distinct from a coach (who informs but does not act). ✅ MEDDICC definition, §5.6.
- **Coach** — a friendly insider who provides information about the deal (people, process, politics) but does not advocate; valuable but not a substitute for a champion. §9.1.
- **Commit** — forecast revenue a rep is accountable to deliver in the period; the conservative layer of the forecast. §8.1.
- **Decision Criteria** — the principles, guidelines and requirements (published or hidden) the customer uses to judge the solution. ✅ §5.3.
- **Decision Process** — the steps the customer will take to decide whether to buy. ✅ §5.4.
- **Economic Buyer (EB)** — the person with overall authority in the buying decision; the one who can say yes to the money. ✅ §5.2.
- **Implicate the Pain** — MEDDICC's modern gloss on the I: having identified, indicated and implicated the pain — including its consequences and cost. ✅ §5.5.
- **Implications of pain** — the consequences of an identified problem; the concept some extended variants add as a ninth element or fold into the I. §5.5.
- **MEDDIC** — the original six-letter framework: Metrics, Economic Buyer, Decision Criteria, Decision Process, Identify Pain, Champion. ✅ §3.1.
- **MEDDICC** — the seven-letter variant adding Competition. ✅ §3.1.
- **MEDDPICC** — the eight-letter variant adding Paper Process (and Competition): Metrics, Economic Buyer, Decision Criteria, Decision Process, Paper Process, Identify Pain, Champion, Competition. ✅ §3.1.
- **Metrics** — the quantifiable measures of value the solution provides; the customer's business numbers the deal moves. ✅ §5.1.
- **Paper Process** — the steps that follow the decision, taking the deal from decision to signature (procurement, legal, security, credit, onboarding). ✅ §5.8.
- **Qualification** — the discipline of determining whether a deal is real, winnable and worth pursuing, on evidence rather than optimism. §1.1.
- **RFP (Request for Proposal)** — the formal tender document through which corporate clients run competitive banking mandates; the published face of the Decision Criteria in wholesale banking. §11.3.
- **Scorecard** — an operationalization of MEDDPICC in which each letter is scored and rolled up into a qualification grade; an organizational tool, not part of the original methodology. §7.2.
- **Upside** — forecast revenue that could land but is not yet committed; reported separately from commit. §8.1.

### 15.2 The Sources

Primary and practitioner sources verified or consulted this pass (accessed 2026-09-03):

1. MEDDICC — "Who Created MEDDIC? The History Behind the Framework" (13 Aug 2026): https://meddicc.com/resources/who-created-meddic
2. MEDDICC — "MEDDIC / MEDDPICC Sales Methodology and Process" (the what-is-meddpicc hub, per-letter definitions and variant comparison): https://meddicc.com/what-is-meddpicc
3. MEDDICC — "The Backstory of Andy Whyte" (7 Apr 2026): https://meddicc.com/resources/the-backstory-of-andy-whyte
4. MEDDICC — "Meddicc Welcomes Dick Dunkel, Creator of Meddic Framework, to The Team": https://meddicc.com/resources/meddicc-welcomes-dick-dunkel
5. MEDDICC — "US Federal Court Rules MEDDPICC is a Generic Term, Orders Cancellation of Trademark" (28 May 2026): https://meddicc.com/resources/meddpicc-trademark-cancelled-federal-court-ruling
6. Monica Evans (Sales MEDDIC Group) — "The Origins of MEDDIC" (11 Mar): https://www.salesmeddic.com/blog/origin-of-meddic
7. RevCentric Partners — "About — Built by Dick Dunkel, Author of MEDDIC": https://revcentricpartners.com/about
8. Force Management — "MEDDICC Sales Qualification Methodology & Process": https://www.forcemanagement.com/offerings/meddicc
9. Force Management — "Interview with Author & Sales Veteran John McMahon": https://www.forcemanagement.com/interview-john-mcmahon
10. eChai Ventures — "John McMahon · GTM people": https://echai.ventures/gtm/people/john-mcmahon
11. Selling Like a Champion — John McMahon profile: https://www.sellinglikeachampion.com/johnmcmahon-1
12. Huthwaite International (Neil Rackham) — "The origins of SPIN — the most researched sales model in the world" (6 Feb 2026): https://www.huthwaiteinternational.com/blog/origins-of-spin-selling
13. Wikipedia — "The Challenger Sale": https://en.wikipedia.org/wiki/The_Challenger_Sale
14. PR Newswire — "The Rise of 'The Challenger Sale': Corporate Executive Board Research Confirms the Demise of Relationship Selling": https://www.prnewswire.com/news-releases/the-rise-of-the-challenger-sale-corporate-executive-board-research-confirms-the-demise-of-relationship-selling-133831648.html
15. Winning by Design — corporate site (SPICED/Bowtie/REKS attribution): https://winningbydesign.com
16. Practitioner and vendor-comparison literature surfaced this pass: Highspot — "The MEDDIC Sales Methodology" (https://www.highspot.com/blog/meddic-sales-methodology/); MeetGeek — "A Complete Guide to the MEDDIC Sales Methodology" (https://meetgeek.ai/blog/meddic-sales-methodology); Closing Foundry — "MEDDIC vs MEDDPICC vs MEDDICC" (https://www.closingfoundry.com/insights/meddic-vs-meddpicc); praiz.io — "MEDDICC and MEDDPICC: what the extra letters change" (https://www.praiz.io/blog/meddicc-and-meddpicc-what-the-extra-letters-change); Sellible — "MEDDPICC Financial Services: Banking & Insurance Guide" (https://blog.sellible.ai/meddpicc-sales-methodology-guide-for-financial-services/); prospeo.io — "BANT IBM: Origin, Framework & How to Use It" (https://prospeo.io/s/bant-ibm); demandnexus.io, realgrowthmatters.com and getgangly.com BANT-origin pages.
17. Sibling repository guides (cross-referenced, not sources for external facts): [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md), [Vendor Management Guide](vendor_management_guide.md), [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md), [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md), [Communication & Stakeholder Management Skills](communication_stakeholder_management_skills_guide.md), [Facilitation Skills Guide](facilitation_skills_guide.md), [Ancillary Revenue Products Guide](ancillary_revenue_products_guide.md).

### 15.3 The Closing Summary

MEDDPICC is, at bottom, a memory system with an attitude: it assumes that big deals are lost to ignorance, not to bad luck — ignorance of the money owner, the real criteria, the internal process, the paperwork, the pain's true cost, the champion's actual advocacy, and the competition that never shows up in the RFP. Born at PTC in the mid-1990s as six letters distilled from a win/loss study, carried out of the company by a diaspora of trained sellers, extended by later generations with Competition and Paper Process, turned into a global training brand in 2020, and declared a generic term by a US federal court in 2026, the framework survived every stage of its own commercialization because the underlying discipline is vendor-independent: know your deal, quantify the value, map the people and the paper, and never let a forecast rest on a feeling. In wholesale banking — where the evaluation is a committee, the buyer is a CFO two levels up, the criteria are an RFP scorecard, and the signature waits on three interlocking paper queues — that discipline is not a sales nicety; it is the difference between a mandate that lands in the promised quarter and one that dies somewhere between the committee's yes and the boarding team's inbox. The champions, the metrics, the mapped paper and the shaped criteria of the Cymbal Bank worked example are not decorations on the methodology; they are the methodology, applied where the stakes are highest. Qualified well, a deal is not a hope — it is the qualified deal.
