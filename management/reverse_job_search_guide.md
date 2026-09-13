# The Reverse Job Search — the published candidate

> **A deep-research guide to the inverted job search: the model in which the candidate publishes a profile, a proof-of-work record, or a single self-hosted page, and the employer does the searching — covering what the term actually means and where it comes from, the verified platform landscape from the Hired/Vettery auctions to the niche board graveyard and the candidate-pays tools that replaced them, what each "open to work" signal really exposes and to whom, the candidate-side playbook with its honest limits, the AI-era collision of generated applications and machine screening, the recruiter's side of the same inversion, what the labour-economics evidence does and does not support, the recruitment-fraud surface that publishing creates, and the Singapore work-pass and fair-hiring constraints that shape all of it.**

**Author:** Jack Liu Shurui · **Role:** Solution Architect, Cymbal Bank  
**Series:** Career Engineering · Market Mechanics · **Domain:** Hiring Platforms · Talent Markets · Singapore Technology Employment  
**Reading time:** ~35 minutes  
**Companion guides:** [contract_job_smart_move.md](../technology/contract_job_smart_move.md) (the engagement-structure decision this guide deliberately does *not* re-derive — that guide answers *what type of engagement*, this one answers *how the candidate gets discovered*), [professional_networking_guide.md](../technology/professional_networking_guide.md) (the repo's personal-branding content, pointed to rather than repeated), [google_system_design_interview_guide.md](../technology/google_system_design_interview_guide.md) · [system_design_interview_insiders_guide.md](../technology/system_design_interview_insiders_guide.md) · [ml_system_design_interview_guide.md](../technology/ml_system_design_interview_guide.md) (the interview cluster — a later phase, pointed to only), [the_first_90_days_guide.md](the_first_90_days_guide.md) · [authority_skills_guide.md](authority_skills_guide.md) · [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) · [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) · [grow_team_guide.md](grow_team_guide.md) (the human-skills cluster — presumed prerequisites, not re-derived), [agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md) · [agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md) · [agentops_guide.md](../technology/ai_llm/agentops_guide.md) · [llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md) · [prompt_injection_guide.md](../technology/ai_llm/prompt_injection_guide.md) · [ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) · [coding_agents_research.md](../technology/ai_llm/coding_agents_research.md) (the agent cluster — referenced from §6, mechanics not re-derived)

---

## Table of Contents

- [Sourcing convention used throughout this guide](#sourcing-convention-used-throughout-this-guide)

1. [What the Reverse Job Search Is (and Is Not)](#1-what-the-reverse-job-search-is-and-is-not)
2. [The Platform Landscape, Verified](#2-the-platform-landscape-verified)
3. [Signals and 'Open to Work' on the Major Networks](#3-signals-and-open-to-work-on-the-major-networks)
4. [Niche, Industry-Specific and Self-Hosted Options](#4-niche-industry-specific-and-self-hosted-options)
5. [The Candidate-Side Playbook](#5-the-candidate-side-playbook)
6. [The AI Era](#6-the-ai-era)
7. [The Employer Side](#7-the-employer-side)
8. [The Evidence Base](#8-the-evidence-base)
9. [Risks and Costs](#9-risks-and-costs)
10. [The Singapore and Asia Context](#10-the-singapore-and-asia-context)
11. [Worked Example: Cymbal Bank's Reverse Talent Model](#11-worked-example-cymbal-banks-reverse-talent-model)
12. [One-Page Summary](#12-one-page-summary)
13. [Claims Audit](#13-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [References](#16-references)

---

## Sourcing convention used throughout this guide

Every factual claim in this guide carries one of three markers. The topic makes this discipline unusually important: almost every number in the recruit-tech space is published by a party that sells access to candidates, and the single most-quoted statistic in the field (the "hidden job market") is a fifty-year-old arithmetic artefact that has been laundered into folklore.

| Marker | Meaning |
| --- | --- |
| ✅ | **Verified at a primary source.** The claim appears in a first-party artefact — the platform's own site or help documentation, a regulator's or government's own page, a peer-reviewed or named-institution publication — or is directly documented at the entity concerned. |
| ⚠ | **Single-source, vendor-published, press-relayed, or promotional.** The claim appears in material published by a party with a commercial or reputational interest, in one place only, or relayed by a directory or blog. Treat as a claim, not a measurement. This guide says *why* each such claim is weak. |
| ❌ | **Not verifiable.** No public source states it. These are collected in [§14 What Could Not Be Verified](#14-what-could-not-be-verified) rather than inferred, estimated, or invented. |

Blocks labelled **Guide's analysis** or **Practice** are the author's own reasoning or practitioner convention, not sourced findings. They are meant to be argued with. Where a number is labelled **constructed by this guide**, it is an illustrative metric definition, not a measured value.

### The sources, and what each is

| Source | Date | Authorship | Interest |
| --- | --- | --- | --- |
| [LinkedIn Help — "Let recruiters know you're Open to Work"](https://www.linkedin.com/help/linkedin/answer/a507508) | Product doc, updated ~2026 | LinkedIn (first party) | **Vendor.** Describes its own feature; unusually candid about the privacy limit |
| [Ministry of Manpower — Fair Consideration Framework](https://www.mom.gov.sg/employment-practices/fair-consideration-framework) | Updated 8 Aug 2025 | Singapore Government | Primary; regulator describing enforceable rules |
| [MOM — Eligibility for Employment Pass](https://www.mom.gov.sg/passes-and-permits/employment-pass/eligibility) · [S Pass](https://www.mom.gov.sg/passes-and-permits/s-pass/eligibility) | Updated 2026 | Singapore Government | Primary; salary and COMPASS thresholds |
| [FBI IC3 2024 Internet Crime Report](https://www.ic3.gov/AnnualReport/Reports/2024_IC3Report.pdf) | 2025 | FBI (government) | Primary, but self-reported complaint data — see the caveats in §9 |
| [Kuhn & Skuterud, *Internet Job Search and Unemployment Durations* (IZA DP 613; AER 94(1), 2004)](https://ftp.iza.org/dp613.pdf) | 2002 / 2004 | Academic economists | Peer-reviewed; the best available counterweight in §8 |
| [BLS *Occupational Outlook Quarterly* — "Focused jobseeking"](https://www.bls.gov/careeroutlook/2011/spring/art01.pdf) | Spring 2011 | US Bureau of Labor Statistics | Named institution, but an advice article quoting practitioners, not a study |
| [Cavuno — "What Is a Reverse Job Board?"](https://cavuno.com/blog/what-is-a-reverse-job-board) | 10 Jul 2026 | Job-board software vendor | **Vendor.** Sells board software; the most detailed public history of the category, and its own listing of dead boards is checkable |
| [Wellfound](https://wellfound.com/) · [Glints](https://glints.com/sg) · [ElixirDevs](https://elixirdevs.com/) · [Simplify](https://simplify.jobs/) · [LinkedIn Recruiter](https://business.linkedin.com/talent-solutions/recruiter) | 2026 | Platform marketing pages | **Vendor.** Usable for *model and monetisation*, not for outcomes |
| [Learning Curves — "Hidden Origins of the Hidden Job Market Pt. 2"](https://learningcurves.org/hidden-origins-of-the-hidden-job-market-pt-2/) | 2026 | Employment specialist, Epilepsy Toronto | Advocacy-adjacent but archival; traces the 80% figure to a named 1966 survey |
| [Wikipedia — Hired (company)](https://en.wikipedia.org/wiki/Hired_(company)) · [SourcingTools — Hired review](https://www.sourcingtools.org/tools/hired/) | 2026 | Community/directory | Relays TechCrunch/Reuters; used only where corroborated |

**The structural warning up front.** Nobody publishes audited data on reverse-job-search outcomes. The platforms publish inventory counts and marketing conversion animations; the press repeats the platform's funding and its shutdown; the labour economists studied *internet* job search in 1998–2000 and never revisited it for candidate-publishing markets. Read §8 before quoting anything in this field as evidence.

---

## 1. What the Reverse Job Search Is (and Is Not)

### 1.1 One paragraph

The **reverse job search** is the practice of making yourself findable instead of making applications: you publish a durable, searchable representation of your capability — a profile on a network, a listing on a candidate-side marketplace, a repository history, or a single self-hosted page — and you rely on someone on the hiring side to search for you, match you, and initiate contact. The candidate becomes the listing. The searches run in the other direction. Everything else in this guide is about what that inversion actually buys, what it costs, and where the evidence stops.

### 1.2 The two directions, side by side

| | Ordinary (outbound) search | Reverse (inbound) search |
| --- | --- | --- |
| Who publishes | Employer publishes a requisition | Candidate publishes a profile, page, or artefact |
| Who searches | Candidate searches postings | Employer searches a pool or the open web |
| Who initiates contact | Candidate (application) | Employer or recruiter (InMail, message, email) |
| Unit of inventory | Job posting, life ~30–60 days, replaced every cycle ⚠ (Cavuno, vendor) | Candidate profile, life *until hired* — the best profiles exit fastest ⚠ (Cavuno) |
| Failure mode | The application is never read | The profile is never found, or is found by the wrong searcher |
| Scaling constraint | Candidate attention | Candidate supply, and its perishability |
| Cost bearer | Candidate's time; employer's posting fee | Employer's seat/licence fee, or the candidate's subscription |

That "who publishes" row is the entire redefinition, and the *failure mode* row is why the model disappoints people who assume it is passive. An unpublished application fails quietly; an unfound profile fails silently. Both are failures of a different kind, and only one of them can be fixed by writing better.

### 1.3 It is not merely "being on LinkedIn"

This distinction matters more than any other in the guide, because conflating the two is how people conclude the reverse search "already works" and stop investing in it.

- **Being on LinkedIn is a default state.** A profile exists because you have an account. Recruiters can search it, but you have published nothing deliberate, made no claim, and produced no artefact. You are *indexed*, not *published*.
- **The reverse job search is a deliberate publication.** It means choosing a niche narrow enough to be searched by name, publishing verifiable work, structuring the profile so that a Boolean query for that niche returns you, and then handling the inbound that results.

The functional test is not "did I get an InMail this month" — most professionals do, on the strength of a job title alone. The test is: **can a competent hiring manager, who has never met me and does not know my name, find me by describing the problem they need solved?** If the answer is no, you are indexed but not published.

### 1.4 The term is loose, and this guide will not pretend otherwise

Honest provenance, because the previous generations of career writing did not do this and it is the whole reason §8 exists.

- **The phrase "reverse job search" is not a settled industry term.** A job-board software vendor's 2026 guide — the most detailed public treatment located for this repo — defines it explicitly as *"a tactic, not a platform… the job seeker's practice of targeting companies directly instead of applying to postings. No operator, no marketplace."* ⚠ (Cavuno, vendor-published). Note the divergence immediately: on that definition the reverse job search has no platform at all, whereas the popular use of the phrase is precisely about platforms.
- **Practitioner usage is inconsistent and mostly marketing.** The phrase surfaces in recruiter-agency blog posts and LinkedIn Pulse articles meaning "employers come to you" ⚠ (e.g. the Work in Virtual Pulse post and a cluster of career-coach blogs located in the first search pass), and in others meaning "find the human behind the posting and message them first" — a *front-door* tactic, which is closer to outbound networking than to publishing. Two opposite practices share one label.
- **The adjacent term with firmer provenance is "reverse job board."** That one has a documented origin artefact: a developer named Andrew Horner built a "reverse job application" in 2012 — a single page listing himself and inviting companies to apply to *him* ⚠ (Cavuno). The artefact itself is gone. The category became real in 2021 when RailsDevs launched, described in §2.2.
- **Verdict.** ✅ for the *existence* of the practice and for its platform history; ❌ for any claim that "reverse job search" is an established term with a standard definition, and ❌ for any claim about how many people practise it or how well it works. Those are the two gaps this guide cannot close, and §14 lists them.

### 1.5 The candidate-as-product framing, and where it comes from

The framing is not a career-coach invention; it is a transplant from two-sided-marketplace literature, and the transplant is worth naming because it carries a warning.

- From marketplaces: the operator's job is to make candidate supply the inventory and employer access the product — *"The candidates are the listings. Employer access is the product."* ⚠ (Cavuno, vendor).
- From the platform side, the same structure is visible in the products themselves: LinkedIn's own recruiter page sells access to "1B+ professionals" and a shortlist workflow ✅ (LinkedIn, first party), and Wellfound's page sells "10M+ candidates who opted in to hear from startups" ⚠ (vendor, unverifiable count).
- **The warning, in platform-economics terms.** As the Cavuno analysis puts it, candidate profiles are **perishable inventory**: a posting is useful for its whole life and the employer will repost next quarter, whereas a profile is useful until the candidate is hired, and the profiles employers most want to reach exit the pool *fastest* — *"Success consumes your inventory."* ⚠ (Cavuno). For a candidate this is two-sided news: a working reverse market for your niche means you get contacted and then leave; a market that has stopped working for your niche is one where the board has quietly run out of employer attention, not where your profile became bad.

### 1.6 What this guide does not claim

- It does not claim the reverse search is superior to applying, networking, or contracting. No study supports that (§8), and the platform history in §2 is a graveyard.
- It does not claim it substitutes for skill, or for the workplace skills in the `../management/` cluster. Those are prerequisites. A published candidate who cannot hold a difficult conversation is a well-indexed problem.
- It does not re-derive the engagement-structure decision — contract versus permanent, day-rate premium, entity structuring. That is [contract_job_smart_move.md](../technology/contract_job_smart_move.md)'s territory, and §10 points there rather than repeating it.

---

## 2. The Platform Landscape, Verified

The category is small enough, and recent enough, that its full arc is checkable. It reads: a venture-funded auction, a venture-funded screening marketplace, an acquisition, an absorption into a staffing firm, an open-source niche board that worked and closed anyway, and a handful of living niche boards that monetise the employer. Verify any "active reverse job boards" list against this before trusting it — the category churns faster than the roundups.

### 2.1 The original reverse marketplaces: DeveloperAuction/Hired and Vettery

Two different companies, two different mechanics, one long merging sequence. Keep them separate or the history will not parse.

**The original Hired (San Francisco, 2012).** Founded in 2012 by Matt Mickiewicz, Douglas Feirstein, and Allan Grant, and launched as **DeveloperAuction** — a marketplace in which software engineers auctioned their skills to bidding employers, i.e. the literal "talent auction" mechanic ⚠ (secondary aggregators relaying a 2014 TechCrunch report; the original TechCrunch article URL returned 404 on retrieval, see §14). The company renamed to Hired, raised venture capital across several rounds — figures cited in different secondary sources diverge ($133M at Tracxn; "over $150M" at another aggregator) ⚠, and was reported as once valued around $500M ⚠ (Cavuno). It came close to winding down in 2020 ⚠ (Cavuno).

**Vettery (New York, 2013).** Founded March 2013 in New York by Brett Adcock and Adam Goldstein — two former hedge-fund analysts who had covered the recruiting industry ✅/⚠ (Wikipedia, relaying TechCrunch and the New York Post). Launched in the US in June 2015 after a **$1.7M seed** in June 2015; raised **$9M Series A** in 2016 led by Raine Ventures and Greycroft; **$11.9M total over three rounds** ⚠ (Wikipedia, relaying TechCrunch/AlleyWatch). UK launch May 2018 ✅ (Onrec, 14 May 2018 — a trade announcement relaying company statements: candidate CVs are "assessed and rigorously screened" and *"only five percent of candidates who apply through the site are accepted"*, after which *"clients [approach] potential employees directly for interview"*). **The 5% figure is ⚠: it is a company claim in a launch press release, uncorroborated and un-auditable.**

**The sequence.**

| Date | Event | Evidence |
| --- | --- | --- |
| Feb 2018 | Vettery acquired by Adecco Group; TechCrunch reported a price "a little over $100 million", terms not disclosed | ⚠ Wikipedia relaying TechCrunch; Reuters reported the acquisition without price |
| Nov 2020 | Vettery (as part of Adecco) acquired the competing tech marketplace Hired, undisclosed price | ⚠ Wikipedia relaying TechCrunch |
| Mar 2021 | The two products were combined and rebranded under the **Hired** name | ⚠ Wikipedia relaying Hired's own blog |
| 14 Jun 2024 | Hired folded into **LHH Recruitment Solutions** (Adecco); `hired.com` redirects to LHH, self-serve sign-up closed | ⚠/✅ Wikipedia; sourcingtools.org review corroborates the redirect and closure |
| 2026 | No standalone Hired marketplace exists; the matching technology is reportedly used inside LHH's recruiting practice, priced as a service engagement (estimated $10K–$40K/yr historically) | ⚠ third-party directory estimate, explicitly labelled "directional" by its publisher |

**What the Hired mechanic actually was.** Candidates were vetted, published a profile with **declared salary expectations** and a signal that they were actively interviewing; employers browsed and requested interviews, competing in the open rather than negotiating blind ⚠ (sourcingtools.org, third-party). That two-sided compensation transparency is the part practitioners still miss most, and it is the part that died with the marketplace.

### 2.2 RailsDevs: the case study worth reading in full

The best-documented reverse job board is also the one that proves the model's economics are hostile even when the product is loved.

- Launched **November 2021** by Joe Masilotti for Ruby on Rails developers; open-sourced; run in public ⚠ (Cavuno, which cites Starter Story's case study and the operator's public notes).
- **Roughly 100 developer profiles in the first week** — largely from the founder's existing audience; **600+ developers and 30+ hires by the end of year one**; **~$146K in first-year revenue**, averaging around **$10,000/month at its best** ⚠ (Cavuno relaying Starter Story — two removes from the source).
- Monetisation, per Cavuno's account of RailsDevs' published numbers: browsing free, **messaging a developer required a paid company subscription**, and **hiring fees made up roughly 90% of revenue** ⚠. The implication Cavuno draws is sharp and should be quoted: *"The best-known 'reverse job board' was economically a lightweight recruiting agency with a profile directory attached."* ⚠ (vendor's analysis).
- **Shut down; codebase archived July 2025; domain offline** ⚠ (Cavuno, "verified July 2026"). The operator's farewell note reportedly cited no operational cause, only that it was time to make space ⚠. **There is no published post-mortem** — see §14.

**Guide's analysis.** The RailsDevs arc is the strongest available argument that the reverse model's ceiling is structural, not promotional: an operator with an audience, an open codebase, and four years of goodwill still converged on recruiting-agency economics (success fees) rather than marketplace economics (subscriptions), because the counterparties can simply take the conversation off-platform once matched. Any candidate strategy that depends on a single niche board surviving should be built with that convergence in mind.

### 2.3 Living niche boards, and the ones that are not

**Live.** **ElixirDevs** — a directory of Elixir developers that is **free for developers and paid for companies**, who pay for deeper profile access and to start contact; it also documents a two-state availability flag ("actively looking" versus "open to interesting opportunities") ✅ (vendor site; the 900+ profile count is ⚠ marketing). **Vue Developers** — a Vue.js directory on a subscription model, live but quiet ⚠ (single source).

**Dead or gone.** **RailsDevs** (archived Jul 2025), **HireThePivot** (site offline; descendant code still public), the 2012 **"reverse job application"** origin page (long gone) — all ⚠ (Cavuno) — and **Hired** itself as a marketplace (folded into LHH, Jun 2024; see §2.1) ⚠/✅.

Note what is *not* in that table: any live, venture-scale, candidate-publishes-everything marketplace in 2026. The surviving candidate-first boards are **niche, small, and employer-funded**. That is the honest state of the art, and it is the fact that most "reverse job search" listicles omit.

### 2.4 The giant generic databases: the reverse model at scale

The most consequential reverse marketplace is the one nobody calls one.

- **LinkedIn Recruiter.** Employer-pays, seat-licensed; search across "1B+ professionals"; 40+ advanced filters, keywords, and Boolean; up to **150 InMails per month per seat**; ATS/CRM integration ✅ (LinkedIn's own Recruiter product page for the mechanics; the outcome claims on the same page — see §7 — are ⚠). Candidates publish profiles; employers pay to search and contact. That is the reverse model, industrialised.
- **Indeed Resume** and equivalent résumé databases. Employer-pays access to candidate databases, broadly described in the same category ⚠ (Cavuno, vendor). **The Indeed employer pricing page could not be retrieved during research (fetch failed); no Indeed pricing or model claim is made in this guide** — see §14.
- **Wellfound (formerly AngelList Talent).** Candidate-side: *"Creating a profile and applying to jobs on Wellfound is always free for candidates"* ✅ (vendor's own FAQ). Employer-side: free unlimited job posts, paid promotion, paid AI sourcing ("Reach"), and managed hiring ("Autopilot") — third-party tracking estimates Autopilot at ~$500/month per role plus a ~10% placement fee ⚠ (sourcingtools.org estimate). Inventory claims: 27,000+ startups, 10M+ opted-in candidates, plus 500M+ enriched external profiles for AI sourcing ⚠ (vendor marketing; unauditable).
- **Glints (Southeast Asia).** Employer-pays job posts; free to post, or managed hiring; positions itself on Southeast Asian early-career and mid-level talent: *"Post jobs for free or let Glints build and run your team anywhere in the world"*; claims 55,000+ organisations and 150+ countries, plus an AI-matching and built-in ATS layer ✅ (vendor's own site for the model; ⚠ for the scale claims). Relevant to §10.

**Guide's analysis.** For a candidate, the giant databases and the niche boards are different bets with the same downside. On LinkedIn you are findable in a pool of a billion, which means your *specific* discoverability depends on keyword discipline and on the recruiter's search being good. On a niche board you are findable in a pool of a thousand, which means discoverability is easier but employer attention is scarce and perishable. Neither is passive, and neither substitutes for §5.

### 2.5 The comparison table

| Platform | Model | Monetisation | Status (2026) | Evidence quality |
| --- | --- | --- | --- | --- |
| **Hired (original, 2012)** | Candidate published profile + salary expectation; employers requested interviews; auction origin | Employer pays | Dead; folded into LHH (Adecco) 14 Jun 2024 | ⚠ aggregated press; ✅ for the LHH absorption |
| **Vettery (2013)** | Vetted candidate database; employers approach; ~5% acceptance claimed | Employer subscription/success fee | Merged into Hired (Mar 2021) | ⚠ press + company press release |
| **RailsDevs (2021)** | Candidate directory, employer pays to message | Subscription + ~90% from hiring fees | Dead; archived Jul 2025 | ⚠ vendor blog relaying Starter Story |
| **ElixirDevs** | Candidate directory, developer free | Employer pays for access/contact | Live | ✅ vendor site |
| **Vue Developers** | Candidate directory | Employer subscription | Live but quiet | ⚠ single source |
| **Wellfound** | Candidate profiles + employer posts; AI sourcing added | Employer pays (posts free; Reach/Autopilot paid); candidate free | Live | ✅ candidate-free claim; ⚠ inventory counts |
| **LinkedIn Recruiter** | Candidate profiles, employer search + InMail | Employer pays per seat | Live, dominant | ✅ mechanics (vendor first-party); ⚠ outcome claims |
| **Glints** | Employer posts + AI matching; candidate profiles | Employer pays | Live (SEA) | ✅ model; ⚠ scale claims |
| **Indeed Resume** | Candidate database, employer pays | Employer pays | Live | ❌ not verified in this pass |
| **Triplebyte** | Vetted tech-candidate marketplace | — | Not verified | ❌ see §14 |
| **MyCareersFuture (SG)** | Government job portal; employer must advertise | Publicly funded; free to employers | Live, mandatory for EP/S Pass advertising | ✅ MOM |

---

## 3. Signals and 'Open to Work' on the Major Networks

A "signal" is a deliberate marker that you are available. The design question for every one of them is the same: **who can see it, and does seeing it help you or cost you?** Only the largest network documents its settings to a standard that survives scrutiny, so this section is short and specific on purpose.

### 3.1 LinkedIn #OpenToWork: what it actually does

Verified against LinkedIn's own help documentation for the feature ✅:

- Enabling it lets *"recruiters and your network"* know you are open, and — the mechanically important part — *"we'll help your profile show up in search results when recruiters look for suitable job candidates."* ✅ (LinkedIn Help). So the signal is not decorative; it feeds recruiter search.
- You choose the audience, and there are exactly three options ✅:
  1. **All LinkedIn Members** — includes recruiters *and people at your current company*, and adds the **#OpenToWork photo frame** to your profile picture.
  2. **Recruiters only** — visible to *"People using LinkedIn Recruiter only."*
  3. **Visible only to you** — job preferences are used to tailor your recommendations.
- Fields you supply (job titles, location types, locations, start date, employment types, visibility) are what the search matching runs against ✅.
- **India-specific:** notice period / availability to join and expected annual salary can be specified and are **visible to recruiters only, regardless of your Open To Work visibility setting** ✅ (LinkedIn Help). If you are not in India these fields are not part of the feature.
- **Automatic decay:** *"If we notice that you stop responding to InMails from recruiters, we'll email you to confirm that you're still open to work. If we don't receive confirmation, we'll automatically remove the #OpenToWork feature"* ✅ (LinkedIn Help). This is the platform's own admission that a stale signal degrades the market for everyone, and it is a maintenance obligation on the candidate, not a fire-and-forget switch.

### 3.2 The privacy limit, stated by the platform itself

Buried in the same document is the sentence that should govern how every candidate thinks about publishing availability:

> *"To protect your privacy, we take steps to prevent LinkedIn Recruiter users who work at your company from seeing your shared career interests. We take the current company on your LinkedIn profile marked as 'I am currently working here' to know who to hide your Open to Work status from, however, **we can't guarantee complete privacy**."* ✅ (LinkedIn Help, emphasis added)

Two operational consequences follow, and both are the candidate's problem, not the platform's:

1. **The hiding rule depends on you keeping the "I am currently working here" flag accurate.** If your current employer is listed as a past role, or listed as a subsidiary when the recruiter seat sits at the parent, the suppression may not apply as you expect.
2. **Contingent staffing breaks the rule by construction.** If the recruiter searching you is an agency seat staffed *into* your employer, they are not "a LinkedIn Recruiter user who works at your company" in the flag's sense. Publishing availability while contract-placed is therefore a materially different risk decision from publishing it while permanently employed — an engagement-structure consequence, and the reason §10 points at [contract_job_smart_move.md](../technology/contract_job_smart_move.md) rather than treating the two cases as identical.

### 3.3 Backlash: an honest gap

There is a widely repeated practitioner narrative that the public `#OpenToWork` frame is stigmatising and reduces reply rates. **This guide could not verify it.** The searches run for documented backlash returned no primary source, no platform data, and no study; only repeated assertion in coach blogs ❌ (see §14). Treat the claim as folklore unless you can find a measurement. What *is* documented is narrower and more useful: the frame is **only** added in the "All LinkedIn Members" setting ✅, so the visible-frame trade-off is a discrete, reversible choice rather than an unavoidable property of the feature.

### 3.4 Other networks

- **Indeed and other boards** offer résumé-visibility settings in the same spirit. **Not verified in this pass** (the primary pages did not return content) ❌. Do not assume their semantics mirror LinkedIn's; there is no reason they would.
- **GitHub, Stack Overflow, and community platforms.** These are not availability signals at all; they are *work* signals (§4). Their visibility semantics are those of any public repository: permanently public, globally indexed, and effectively irreversible once forked or cached.
- **Salary-transparency fields.** Where a platform exposes expected compensation to recruiters, engaging with it trades privacy for match quality. LinkedIn documents this only for India ✅; platform-by-platform behaviour elsewhere is ❌ unverified.

---

## 4. Niche, Industry-Specific and Self-Hosted Options

When the generalist platforms are too noisy and the historical marketplaces are dead, the reverse search moves to two places: a **niche board** with a curated pool, or **infrastructure you own**.

### 4.1 Niche reverse boards

The mechanism is uniform across survivors: candidate profiles are the inventory, browsing is free or cheap, **contacting is the paid act**, and the niche is narrow enough that an employer's search intent maps almost one-to-one onto membership.

- **ElixirDevs** is the cleanest living example, and its FAQ states the economics plainly: *"ElixirDevs is always free for developers"*; paid plans exist so that *"companies… [can] access more detailed information about candidates and start the communication"* ✅ (vendor site). It also documents the exact two-state signalling model worth copying: developers indicate whether they are *actively looking* or merely *open to interesting opportunities* ✅.
- **RailsDevs** performed the same function for Rails until July 2025 ⚠ (§2.2).
- Niche boards exist for other communities as RailsDevs forks ⚠ (Cavuno); most died faster than the original.

**What to do with this.** The practitioner test (Practice, not sourced): find whether a reverse board exists for *your* stack, then check three things in order — (1) does the employer side pay for contact, which is the only signal that employer attention is real; (2) when were profiles most recently added or marked hired, which is the supply freshness test; (3) is the operator a person with a public track record, because the base rate of these boards is closure without post-mortem.

### 4.2 The self-hosted "hire me" page

The most durable artifact in this guide is the one you control: a single page that states who you are, what you are for, and what you have built, on a domain you own, that no platform can delist or price-gate.

**What is verified, and what is convention:**

- ✅ **Verified:** the practice has an origin artefact — the 2012 "reverse job application" page built by Andrew Horner, a single page listing himself and inviting companies to apply to him ⚠ (Cavuno; the artefact is gone, so its *form* is documented while its *content* is not). GitHub operates **The ReadME Project**, an official programme for developer profiles and career stories ✅ (github.com/readme) — evidence that the platform treats profile-as-content as a first-class thing, though it documents no "README-as-CV" convention.
- ❌ **Not verified:** that "README-as-CV" is a documented, named convention with known mechanics and outcomes. It is a real practice with real anecdotes, but this guide found no primary description of it as a convention, and it is listed as a gap in §14.
- **Practice (practitioner convention, not sourced):** the page that works is boring. One sentence of positioning naming the niche; three to five artefacts with links; a short "currently" line with availability and location; one contact route; structured data (§4.3). Pages that fail usually fail by being a résumé in HTML — a history rather than an argument.

### 4.3 The machine-readable profile

If the point is to be *found* rather than read, the profile should exist in a form a machine can consume without ambiguity. Two standards are verifiable, and both are real:

| Standard | What it is | Evidence |
| --- | --- | --- |
| **JSON Resume** | An open-source, community-driven standard representing a résumé as a single structured JSON document — sections including `basics`, `work`, `education`, `skills`, `projects` — so career data is authored once and rendered/validated/hosted by any compatible tool | ✅ jsonresume.org (first party); ✅ schema repository, MIT-licensed, ~2.4k stars, archived at the old repo and *moved* into the `jsonresume.org` monorepo (`packages/schema`), npm package `@jsonresume/schema` unchanged |
| **schema.org `Person`** | The vocabulary search engines and crawlers understand for a person: `name`, `jobTitle`, `knowsAbout`, `alumniOf`, `hasCredential`, `sameAs`, `address`, `email` | ✅ schema.org/Person (first party); ✅ usage reported as 10M+ domains based on Google's web index (July 2026) |

**Guide's analysis.** JSON Resume is a *portable authoring format*; schema.org `Person` is a *crawlable claim*. They do different jobs and both are cheap: authoring in JSON gives you one source of truth for every surface, and emitting `Person` JSON-LD gives search engines an unambiguous reading of your title, expertise, and identity links. Neither is a hiring channel. Treat them as discoverability plumbing, and note that the standard's own README documents a repository migration and an archived predecessor ✅ — a reminder that every third-party surface in this guide has a lifespan.

### 4.4 The personal-site discoverability stack

**Practice (constructed by this guide; each element is a verifiable convention, the combination is the author's proposal):**

1. **A domain you own, and a static site on it** — a repository, a build step, and hosting (GitHub Pages, Netlify, Cloudflare Pages). Identity is the asset and hosting is rented; the build history is itself evidence that you ship.
2. **One canonical profile page** with a stable URL, written in the searcher's vocabulary rather than yours — plus `Person` JSON-LD pointing at `sameAs` identities (GitHub, LinkedIn, talks, publications), and a schema-valid `resume.json` versioned in the same repository.
3. **An indexable surface** — a sitemap, descriptive `<title>`/meta, and at least one inbound link from somewhere a crawler already trusts (a conference page, a community listing, a repository README).
4. **A "now"/"currently" line** with a machine- and human-readable availability state, mirroring the two-state pattern ElixirDevs documents ✅: *actively looking* versus *open to interesting things*.

The stack costs a weekend and, unlike every platform in §2, it cannot be discontinued underneath you. Its limitation is equally structural: **nobody searches it unless you are already known or already ranked.** The personal site is the destination of a reverse search, not its cause — which is precisely why §5 spends its length on making the searcher's query land on you.

---

## 5. The Candidate-Side Playbook

This section is the operational core. Each element is marked ✅ where there is evidence behind it, and **Practice** where it is practitioner convention that this guide asserts without a source. Where a metric is invented here, it is labelled **constructed by this guide** — it is a way to keep score, not a finding.

### 5.1 Positioning: the niche-narrowing argument

**Practice, with one evidentiary anchor.** The anchor is BLS's advice article, which reports that specialised boards — those serving "a particular group, such as a university's student body or alumni or members of a specific trade or professional association" — attract fewer competing candidates and give jobseekers *"a greater chance of success,"* and that *"the more specialized an online job board, the more targeted the specific group or location"* ⚠ (BLS OOQ 2011, an advice article quoting named practitioners rather than a study). The same article records the counter-argument from a career-services director: *"It's difficult to differentiate yourself online"* ⚠.

The generalisation to publishing is the guide's, not BLS's: **a published candidate is only findable by a query, and queries contain terms.** If your published identity is "senior engineer," the query that finds you returns thousands and nobody pages past the first fifty. If it is "payments reconciliation on legacy mainframe rails, migrating to event streams," the query that finds you is one a hiring manager actually types, and the answer set is small enough to read.

The discipline is therefore to choose a niche small enough to be searchable and large enough to hire: one industry, one problem class, one stack. Then publish the same niche phrase in the same words on every surface — profile headline, repository description, page title, `knowsAbout`, conference bio. Consistency is the discoverability mechanic; variation is noise.

### 5.2 The proof-of-work portfolio

**Practice.** The reverse search trades on artifacts a stranger can evaluate without trusting you. The hierarchy, weakest to strongest:

| Artefact | Why it works | Cost |
| --- | --- | --- |
| Job title and tenure | Default state; comparable and searchable | Near zero — and near-zero signal |
| Written explanation of a hard problem | Demonstrates reasoning and communication; the human-skills cluster in `../management/` is where that capacity is developed | Hours |
| A public repository with real history | Commit cadence, review quality, issue triage — process evidence, not just a result | Weeks |
| A running system with users | The only artefact that proves delivery under constraints | Months |
| Published talk, paper, or specification | Third-party endorsement by a programme committee | Weeks |
| Reproduced result by a stranger | Someone else validated you; the strongest possible claim | Rare, earned |

**The verifiability rule (Practice):** every claim on a published profile should resolve to an artefact a stranger can open. "Led a migration" is a claim. A migration runbook in a public repo with a postmortem is evidence. The reason to prefer evidence is not modesty; it is that the reverse search is *read by people who did not ask for it*, so the burden of proof is higher than in an interview, where you can answer follow-up questions.

### 5.3 Discoverability mechanics

**Where the evidence exists:** LinkedIn Recruiter documents that employer search runs on *"40+ advanced filters, keywords, and Boolean"* ✅ (LinkedIn's own product page), and that the #OpenToWork signal influences recruiter search results ✅ (LinkedIn Help). So the mechanics are not mysterious — a recruiter types a query, and a ranked list appears.

**Practice (the query-design checklist):**
1. **Enumerate the query strings** a hiring manager would type for your niche — 5 to 10 phrasings, including the ones you dislike.
2. **Place each phrase** where the search actually indexes: headline, About, skills, repository descriptions, page title, `knowsAbout`.
3. **Use the market's vocabulary, not your employer's.** Internal system names are worthless as search terms unless you also publish the generic equivalent.
4. **Re-check the ranking surface**: if you cannot find yourself with your own query, neither can anyone else. Do this logged out, in a private window.
5. **Maintain the freshness signal** — some contact, some commit, some update. Both LinkedIn's auto-removal of stale availability ✅ and the perishable-inventory dynamic ⚠ (Cavuno) point the same way.

**What to leave out (Practice):** the internal project name, the client name if confidentiality applies, the exact architecture of a production system you are not authorised to describe, and any metric that is not public. A published candidate is a permanently public candidate; assume the current employer will read it eventually and write so that this is unremarkable.

### 5.4 Signalling design: what to publish, what to withhold

| Publish | Withhold | Why |
| --- | --- | --- |
| Your niche, in the market's words | Your current employer's confidential system names | Niche attracts; confidential detail repels and risks |
| Availability state (looking / open) | Your precise compensation floor on public surfaces | Availability is the signal; a floor is a negotiation handicap on a public page |
| Named artefacts with links | Names of unshipped or internal products | Evidence is the point; unverifiable claims invite doubt |
| Location and work-authorisation status (where relevant) | Personal contact details in machine-readable form | Authorisation is a hard filter for recruiters; scraped contact data is a phishing asset (§9) |
| A professional contact route | Anything you would not say to your current manager | Contact needs to be reachable, not exhaustive |

Two documented specifics to fold in: LinkedIn's **India-only** recruiter-visible salary/notice fields ✅ show that platforms may expose compensation data to recruiters even when you set visibility narrowly; and the `#OpenToWork` **photo frame appears only in the fully public setting** ✅, so the frame is a choice rather than a side effect.

### 5.5 The inbound-handling workflow

**Practice (constructed by this guide).** Being findable converts a search problem into a triage problem. Without a workflow, the reverse search feels worse than applying, because the interruptions arrive while you are employed.

1. **Triage on four questions, in order, stopping at the first "no":** is this a named role at a named employer, or an agency fishing for CVs? Does it match the niche you published for? Is the engagement type right (the decision itself lives in [contract_job_smart_move.md](../technology/contract_job_smart_move.md))? Is the work-authorisation path viable (§10 for Singapore)?
2. **Respond in three tiers.** A template for the qualified inbound (agree, ask two qualifying questions, propose a 20-minute call); a template for the near-miss (decline, state the niche, ask to be kept in mind — a warm pool costs one message); silence for the rest, with a rule that stops you reading them at all after the first week.
3. **Time-box.** One scheduled block per week for inbound, plus a stated response time on the published page. Unbounded inbound handling is the failure mode that makes people switch #OpenToWork off.
4. **Keep one outcome log** with dates and channels — the input to §5.6.
5. **Never** complete a form, provide bank details, identity documents, or a "pre-employment equipment purchase" before a signed offer from a verifiably real employer. §9 explains why in numbers.

### 5.6 Metrics a candidate should track

**Every metric in this table is constructed by this guide.** None of them is a sourced benchmark, and no platform publishes comparable figures. They exist so that a reverse search is measurable rather than mood-based.

| Metric | Definition | Why it matters |
| --- | --- | --- |
| **Discoverability hit rate** | Queries you tested ÷ queries that returned you in the top results, measured logged out | Tests §5.3 directly; should approach 1.0 |
| **Inbound volume per month** | Qualified inbound contacts, excluding spam and agency blasts | The market's read on your niche |
| **Qualified ratio** | Qualified inbound ÷ total inbound | Low ratio = your signal is attracting the wrong searchers; fix positioning, not volume |
| **First-response latency** | Median hours from inbound to your reply | Documented platform behaviour penalises non-response ✅ (LinkedIn's auto-removal) |
| **Conversion to first conversation** | Conversations ÷ qualified inbound | Measures how well your page survives scrutiny |
| **Conversion to interview** | Interviews ÷ conversations | Separate from the above: a page that gets conversations but not interviews has a claims problem |
| **Artefact engagement** | Referrals to your page from named artefacts (talk bio, repo, publication) | Identifies which artefact actually carries the search |
| **Stale-signal incidents** | Times your availability state was out of date | The cheapest failure to prevent |

### 5.7 The honest limits, by seniority and geography

**Practice, with the evidence gaps stated.** This is where most guides oversell, so it is worth being blunt:

- **Early career.** The reverse search works poorly. There is little published work to find, and the query "junior developer" returns an unreadable pool. The evidence-adjacent point is BLS's: *"Any candidate who relies on mass-marketed job listings to get in the door is almost certainly doomed to failure"* ⚠ (BLS OOQ, quoting a recruiter) — and the reverse search at junior level is a different mass market, not an escape from one.
- **Mid-career specialist.** The sweet spot. You have artefacts, a nameable niche, and enough experience that a narrow query returns few competitors.
- **Senior and staff-plus.** Positions stop being advertised as roles and start being decided as hires — which is where the reverse search has real leverage, and also where discretion matters more than visibility. Publishing availability at this level is a reputational decision as much as a search one.
- **Geography.** Discoverability is a global mechanic; *hirability* is a local one. Being found by a foreign employer does not create a work-authorisation path, and for Singapore specifically the pass framework in §10 constrains who can actually be hired regardless of how findable they are. The reverse search does not route around immigration law, and any guide implying it does is wrong.
- **The unmeasured part.** Whether reverse-search candidates find *better* roles — measured by pay, tenure, or satisfaction — is ❌ not verifiable. Kuhn and Skuterud's finding that internet job search showed no employment-rate advantage once selection was accounted for, and possibly lengthened spells ⚠✅ (§8), is the closest available prior, and it should temper enthusiasm rather than fuel it.

---

## 6. The AI Era

Two things changed at once: the candidate side got automation, and the employer side got matching. Both are documented; the *interaction* between them is mostly asserted. This section keeps the two apart deliberately, because the conversation about "AI versus AI hiring" is currently about 90% speculation.

### 6.1 Auto-application and job-search agents: what exists, verified

These tools are the *opposite* of the reverse search — they industrialise the outbound application — but they belong in this guide because they are what a candidate is choosing between, and because they alter the noise floor of every channel in §2.

| Tool | Documented capability | Monetisation | Evidence |
| --- | --- | --- | --- |
| **Simplify** | Resume upload and ATS-style scoring; per-role rewrite of résumé bullets; autofill of application forms via a browser extension; an application tracker; "Network Copilot" surfacing people you know at the company; job matching across postings it indexes from *"50+ job boards and company career pages daily"*; claims to help *"2 million job seekers"* | *"Simplify is free for job seekers — matches, autofill, and the tracker included. We make money from Simplify+ subscriptions and from employers who hire through us."* Data policy: profile shared with employers *"only when you apply or choose to make it visible"*, never sold | ✅ vendor's own site for the mechanics and the monetisation statement; ⚠ for the user count and for every outcome-flavoured claim |
| **Provider listicles** (JobCopilot, LoopCV, Jobright, FastApply and others) | Auto-apply, job matching, résumé optimisation, tracking; typically ranked in single-vendor "best tools" listicles that sell in the same category | Subscription | ⚠/❌ — these appeared only in single-vendor listicles during research; no independent verification of their behaviour was obtained, and **the vendors named in listicles are not endorsed by this guide** |

**The honest statement of what is verified here:** exactly one public tool's mechanics and monetisation were verifiable at first party ✅ (Simplify). The category clearly exists. The claims the category makes about outcomes are ⚠ marketing and should be treated as such. **Nothing in this pass verified that any auto-apply tool improves callback rates** ❌.

### 6.2 The AI-versus-AI dynamic: documented, or asserted?

The appealing story is a loop: candidates use LLMs to generate applications, employers use AI to screen them, so volume rises and differentiation collapses. Two halves of it are verifiable, and the loop itself is not.

**Verified half — employer-side automation exists and is sold on time saved.** LinkedIn's own Recruiter page documents an AI agent ("Hiring Assistant") that searches, reviews, and delivers a shortlist, sends prescreening questions, and drafts outreach ✅ (vendor first-party that the feature exists), with outcome claims attached: *"less than 5 minutes on average to find and engage with a qualified candidate,"* *"saving an average of 4+ hours per user, per role,"* *"62% fewer profiles"* reviewed, *"37% less likely to leave in year one,"* and *"55% higher InMail acceptance"* for AI-assisted messages — every one of these footnoted to *"LinkedIn data (2024)"* ⚠. Wellfound's own marketing renders an AI sourcing dashboard with counts like *"241 evaluated · 117 matched · 12 pitched · 3 replied"* ✅ (vendor site, as a product animation) — i.e. an illustrative funnel, not published outcome data ⚠.

**Verified half — candidate-side generation tools exist.** LLM writing assistants and the tools in §6.1 are public products ✅/⚠.

**Asserted half — that AI-written applications systematically meet AI screening and that this is why response rates fell.** Searches for documented evidence of the loop returned no study, no platform dataset, and no named report in this pass. The "244 applications per posting" figure that circulates in the same breath as this claim could not be traced to a primary source either ❌ (see §14). The loop is plausible, widely repeated, and **unverified in this guide**.

**Guide's analysis, clearly labelled as such.** If the loop is real, its consequence for a reverse-search strategy is favourable and specific: automation increases the *supply of applications*, which raises the value of the one input automation cannot manufacture — a stranger-verifiable artefact (§5.2). It also makes the *discovery* channel noisier, because AI sourcing engines (§7) will pitch more candidates at the same roles. Both effects push the same way: publish fewer, better artefacts rather than more, faster applications.

### 6.3 LLM-era candidate presence: machine-readable profiles and matching

- **Verified:** structured, machine-readable personal data has real standards and real adoption ✅ (§4.3 — JSON Resume; schema.org `Person` used on 10M+ domains per schema.org's own reporting).
- **Verified:** employer-side matching consumes profile data at scale — LinkedIn's page states its AI layer is *"based on 1+ billion members and real-time interactions that reveal job seekers' intent and preferences – not just experience"* ⚠ (vendor marketing for a verified product).
- **Not verified:** that any ATS or matching engine parses `resume.json` or schema.org `Person` markup preferentially, or that publishing them improves matching. **No ATS vendor documentation confirming either was located** ❌. The claim that "ATS-friendly" formatting decisions are decisive is one of the most repeated and least sourced claims in career writing.
- **Practice:** publish structured data because it is cheap and unambiguous to *crawlers*; do not pay for it, do not expect a hiring benefit you can measure, and do not let it substitute for a human-readable page (§5.2).

Cross-reference for the agent mechanics that underpin this section, rather than re-deriving them here: [agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md), [agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md), [agentops_guide.md](../technology/ai_llm/agentops_guide.md), [llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md), [prompt_injection_guide.md](../technology/ai_llm/prompt_injection_guide.md), [ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md), [coding_agents_research.md](../technology/ai_llm/coding_agents_research.md). The security relevance is direct: §9's fraud surface is an untrusted-input problem aimed at a public profile, and [prompt_injection_guide.md](../technology/ai_llm/prompt_injection_guide.md) is the frame for treating recruiter messages as hostile input to any assistant you point at your inbox.

### 6.4 The fraud and slop problem: documented cases

This is the part of §6 that *is* documented, at government primary sources, and it is the strongest argument for caution in the whole guide. It is developed with numbers in §9.4; the summary here is that the FBI's IC3 recorded **20,044 employment-fraud complaints and $264,223,271 in losses in 2024** in the United States alone ✅ (IC3 2024 Internet Crime Report, its own crime-type table), that the same report's public-service-announcement appendix lists dedicated warnings on work-from-home scams and on state actors using US-based individuals to defraud employers ✅, and that Singapore's government job portal carries its own scam warning banner aimed at job seekers ✅ (MyCareersFuture homepage, pointing to the national ScamShield job-scam guidance).

### 6.5 Documented versus speculation — the split, explicitly

| Claim | Status |
| --- | --- |
| Public AI job-search and auto-apply tools exist with documented mechanics and monetisation | ✅ (one tool verified at first party) |
| Employer AI sourcing/matching products exist and are sold | ✅ (vendor first-party) |
| Those products' outcome claims (time saved, fewer profiles, retention) | ⚠ vendor claims, footnote-sourced to the vendor's own data |
| AI-written applications meet AI screening at scale, degrading outcomes | ❌ asserted, not verified |
| "244 applications per posting" | ❌ untraceable in this pass |
| ATS parsing of JSON Resume or schema.org markup improves hiring outcomes | ❌ no vendor documentation found |
| Recruitment fraud targeted at job seekers is documented and quantified | ✅ IC3 2024; ✅ Singapore ScamShield/MyCareersFuture |
| Deepfake video-interview fraud is a documented, quantified pattern | ❌ not verified (see §14) |

---

## 7. The Employer Side

The inversion is invisible from the candidate's chair, and misunderstood from the recruiter's. This section is written from the employer's side of the same desk, using only vendor-documented facts where they exist, flagged as claims where the vendor is also the beneficiary.

### 7.1 What the recruiter actually sees

- **A search interface and a ranked list.** LinkedIn Recruiter documents 40+ filters, keywords, and Boolean search, with AI assistance generating a shortlist ✅ (vendor first-party).
- **A quota of outreach.** Up to **150 InMails per month per seat** ✅ (vendor first-party) — a hard, documented ceiling that shapes recruiter behaviour: outreach is rationed, so unqualified candidates are not messaged at all, and the "no reply" case is often "never contacted" rather than "rejected."
- **A pipeline they must feed repeatedly.** Because candidate inventory is perishable ⚠ (Cavuno), sourcing is a standing activity rather than a project, which is precisely why recruiters search continuously and why a stale signal is worse than no signal.
- **A sourcing tool that must justify itself.** LinkedIn's own page sells AI on *"saving an average of 4+ hours per user, per role"* ⚠, and Wellfound sells candidates as *"the ones other searches miss"* ✅/⚠ (both vendor marketing, both shaped to justify the seat cost).

### 7.2 The vendor facts, and which ones are marketing

| Vendor claim | Source | Marker |
| --- | --- | --- |
| Recruiter has 40+ filters, keywords, Boolean; 150 InMails/seat/month; ATS/CRM integrations; 1B+ member network | LinkedIn Recruiter product page | ✅ mechanics; ⚠ "1B+" as marketing |
| AI-assisted messages drive "55% higher InMail acceptance" | LinkedIn Recruiter product page, footnote "LinkedIn data (2024)" | ⚠ vendor, non-independent |
| "LinkedIn hires are 37% less likely to leave in year one, compared to other sources" | Same page | ⚠ vendor, unexplained methodology |
| "It takes less than 5 minutes on average to find and engage with a qualified candidate" | Same page | ⚠ vendor |
| Wellfound: 10M+ opted-in candidates, 500M+ enriched external profiles for AI sourcing, 27,000+ startups | Wellfound site (FAQ) | ⚠ vendor, unauditable counts |
| Wellfound candidate side is free; employer side is free to post with paid Reach/Autopilot | Wellfound site (FAQ) | ✅ first-party pricing statement |
| Glints: hire in 150+ countries; 55,000+ organisations; free job posting with AI matching and built-in ATS | Glints site | ✅ model; ⚠ scale claims |
| Indeed employer pricing and sourcing modules | — | ❌ page fetch failed; nothing asserted |

### 7.3 What an inbound-heavy funnel does to screening

**Guide's analysis.** Moving from outbound applications to inbound sourcing changes the *shape* of the work, not its volume:

- **Funnel entry moves earlier.** With applications, screening starts after a candidate self-selects. With sourcing, the employer screens candidates who never asked to be screened, so precision matters more than recall and *rejection is silent by default*.
- **The resumé stops being the first artefact.** A sourced candidate is evaluated first on the searchable surface — headline, skills, profile — and only later on a CV. That is the same asymmetry §5.3 exploits, seen from the other side.
- **Sourcing cost replaces posting cost.** A job posting is a fixed, published price; a sourcing seat is a licence plus recruiter hours. Employers therefore concentrate sourcing on **scarce roles where the posting market fails**, and use postings for volume roles. A candidate in a role an employer believes is abundant will not be sourced, however well published.
- **AI shortlisting moves the bottleneck downstream.** LinkedIn claims *"62% fewer profiles"* reviewed with AI assistance ⚠ — if true, the effect is fewer, more confident conversations and a harder gate for candidates who do not resemble the shortlist criteria. The relevant risk is *adverse impact*, which §11.6 addresses for a regulated employer.

### 7.4 AI matching and sourcing engines: claims versus evidence

- **Verified:** the engines exist and are sold ✅ (LinkedIn Hiring Assistant; Wellfound Reach and Autopilot; Glints AI matching).
- **Verified:** they are marketed on agentic framing — Wellfound describes *"AI sourcing agents"* that *"identify and engage relevant candidates"* ✅, and LinkedIn describes an agent that *"search[es], review[s], and deliver[s] a summarized shortlist"* ✅.
- **Weak:** every published performance number is vendor-sourced ⚠. No independent audit of a sourcing engine's precision, recall, or bias was located in this pass ❌.
- **Guide's analysis.** For a candidate, the operative question is not whether the engine is good but **what corpus it searches**. LinkedIn's AI runs over LinkedIn; Wellfound's runs over 500M+ *external* enriched profiles ⚠ (i.e. data about people who never opted in) plus 10M+ opt-in profiles ⚠. A directly relevant governance question — how the enrichment data was collected and on what basis — is one employers increasingly have to answer, and §11.6 treats it as a compliance constraint rather than a technical footnote.

### 7.5 What evidence exists on outcomes for employers?

Thin, and honestly so. The published numbers are:

1. vendor claims on vendor pages ⚠ (§7.2);
2. one platform's public history of hiring *through* a reverse marketplace (RailsDevs: 30+ hires in year one, on ~600 profiles) ⚠, which is an existence proof rather than an outcome study;
3. a documented failure arc: two venture-funded reverse marketplaces absorbed into a staffing firm and one open-source board archived, with **no published post-mortem** ⚠/❌.

**There is no published, independent study of reverse-marketplace hiring outcomes** located for this guide ❌. Employers adopting these products are, on the public evidence, buying a sourcing channel on vendor-reported performance — which is exactly the position in which an internal pilot with a pre-registered metric (§11.7) is the responsible move. The `../management/` cluster (in particular the sourcing and vendor-evaluation habits in [vendor_management_guide.md](vendor_management_guide.md)) is the repo's home for that discipline; this guide does not repeat it.

---

## 8. The Evidence Base

This section exists because the reverse job search is usually sold with a statistic. Two statistics in particular — "80% of jobs are never advertised" and "most hires come from networking" — are load-bearing in nearly every article on the topic, and both dissolve under inspection. Getting this right is more useful than any tactic in §5.

### 8.1 What the platforms and the press claim

**Vendor-side, checkable but promotional.** The clearest vendor claim about the *model* is that a reverse board's candidate inventory is perishable and that success consumes the supply ⚠ (Cavuno). Note that this claim is *against* the vendor's own interest as a software seller, which makes it more credible than the outcome numbers on the same page. The scale claims — membership counts, candidate pools, startup counts — are ⚠ across the board and unauditable.

**Press-side.** The funding and shutdown history in §2.1 is reported by TechCrunch and Reuters and relayed by Wikipedia ⚠; the Hired/DeveloperAuction founding and auction mechanic appear in aggregators that cite a 2014 TechCrunch piece whose URL now 404s ❌ (see §14). Press coverage is history, not evidence of effectiveness.

### 8.2 What the labour-economics literature actually says

The best available primary source located for this guide is **Peter Kuhn and Mikal Skuterud, "Internet Job Search and Unemployment Durations"** (IZA Discussion Paper 613, October 2002; published in *American Economic Review* 94(1), March 2004, pp. 218–232) ✅ — a peer-reviewed study using the December 1998 and August 2000 CPS Computer and Internet Supplements matched to subsequent CPS files. Its findings, in its own terms:

- Internet job searchers are **positively selected on observables** — better educated, previously in occupations with lower unemployment — and **negatively selected on unobservables** ✅.
- Naive means show internet searchers more likely to be employed a year later; **once observable differences are held constant, that advantage disappears entirely** ✅.
- Adjusting further for unemployment durations and length-biased sampling yields estimates that are *"counterproductive, i.e. internet job search appears to lengthen unemployment spells"* ✅.
- The authors are careful: a beneficial causal effect can only be consistent with their estimates *"if negative selection on unobservables is especially strong, in other words only if the population of on-line resumes is strongly adversely selected"* ✅.

They also situate the work in an older literature comparing search methods directly — Holzer (1987, 1988), Bortnick and Ports (1992), Osberg (1993), Addison and Portugal (2001) ✅ (cited in the paper) — which is the literature a rigorous reverse-search guide would draw on, and which no reverse-search marketing page cites.

**What this means, carefully.** The paper is from 2002–2004, measures *internet* job search (which then meant résumé posting and board browsing, not a curated profile with published artefacts), and cannot transfer to 2026 markets without qualification. But its core warning transfers perfectly: **the apparent advantage of online/published job search is largely a selection effect**, and the visible winners are not a random sample of practitioners. Anyone citing a LinkedIn-success story as evidence is committing exactly the error Kuhn and Skuterud isolated.

**A named-institution source that is not a study.** The US Bureau of Labor Statistics' *Occupational Outlook Quarterly* article "Focused jobseeking" (Spring 2011, Dennis Vilorio, BLS economist) ⚠/✅ reports practitioner consensus rather than measurement: *"Organizations tend to hire people they know or who are referred to them by someone they trust"*; *"Career experts say that organizations fill many openings through this 'hidden,' or unadvertised, job market"*; *"Any candidate who relies on mass-marketed job listings to get in the door is almost certainly doomed to failure"* (quoting a recruiter); and that specialised boards give jobseekers *"a greater chance of success"* ⚠. Note the careful hedging — *career experts say* — and note that the article is advice framed by practitioner interviews, not a study. It is still the most citable institutional source in this field, which is itself the headline finding of §8.4.

### 8.3 The "80% of jobs are never advertised" statistic: traced to its provenance

This is the guide's most important verification, because the number is recycled endlessly and is not what it appears to be. The archival account (Jesse Preston, Learning Curves, Parts 1 and 2) ⚠ — a lay-accessible but source-quoting account, not a peer-reviewed one — traces it as follows:

1. **The number originates with Bernard Haldane**, who in a 1966 article on job-seeker re-employment wrote: *"according to a Ford Foundation survey, 80 percent of existing job openings are hidden and never show up in the employment office or want ad listings"* ⚠ (relayed).
2. **The survey is real and specific**: a Ford Foundation–funded pilot project conducted by the National Industrial Conference Board to gather labour-market information from businesses in **Rochester, New York**, with results in 1966 ⚠. The account argues this is provably the source because it was the only employer-side survey of its kind at the time, referenced in the 1965 *Proceedings of the Interstate Conference on Labour Statistics* ⚠.
3. **The arithmetic is reconstructive.** In the Rochester data, about **25.1%** of hires by the 27 surveyed companies appeared in the newspaper; service occupations were the bulk of that, and **excluding service and unskilled roles leaves about 18.9% advertised** — hence "about 80% not advertised" for career-type roles ⚠.
4. **The fatal context, in the report's own words:** *"In Rochester, one would not expect a close relationship between the help-wanted index and job vacancies because the larger employers mostly engaged in manufacturing are affiliated with the Industrial Management Council which requests its members not to advertise for employees in the Rochester Newspapers"* ⚠ (quoted). The 80% measures **a deliberate local no-advertising arrangement in one city's manufacturing sector in the mid-1960s**, not a law of labour markets.
5. **Haldane had the number before the survey.** His own earlier writing used 80% in a different sense — jobs filled by personal recommendation — and a 1960 book attributes the breakdown to jobs at a specific salary level ⚠. The statistic was, in this account, retrofitted to a survey that appeared to support a conclusion already held.
6. **The number then propagated.** Later authors, including Richard Irish (1973) and *What Color Is Your Parachute?* (1977 edition), repeated it ⚠, and forty years of career writing repeated them — *"completely divorced from its context"* ⚠.

**Verdict: ❌ as a fact about the present.** The honest statement is that "80% of jobs are never advertised" is a **1966 Rochester artefact** with a local institutional cause, traced by an employment specialist rather than a peer-reviewed study, and that its wide circulation is evidence about career-advice publishing, not about hiring. **Do not use it.** Where a current figure is wanted, the same source points to a 2021 CERIC national business survey (employee referrals used by 67% of businesses; internal candidates identified by 52%; recruitment/search firms by 24%) ⚠ — a survey, relayed, about *methods used*, not about proportions of hires.

**Two related numbers that fare no better.** Wikipedia's job-hunting article reports *"as of 2010, less than 10% of U.S. jobs are filled through online ads"* — cited to a newspaper column which, by Wikipedia's own footnote, *"references uncited studies"* ❌. And the ubiquitous networking figure (50%+, sometimes 85%, of jobs found through networking) appears in the same article as an "estimated" 50%+ attributed to a BLS article that does not measure it ⚠/❌. **The generic "most jobs are hidden" family of statistics is unsourced in every instance this guide could trace.** The "85% through networking" variant is flagged in §14.

### 8.4 Where the evidence is thin — stated plainly

1. **No study measures the reverse job search.** Not the platforms, not the boards, not the self-hosted page pattern. Nil. ❌
2. **The closest literature is 20 years old and about a different behaviour** (posting a résumé on a board), with a result that is unfavourable to naive enthusiasm ✅.
3. **No platform publishes cohort outcomes.** Counts yes; hires, pay, or retention no.
4. **No post-mortem exists for the two flagship failures** ⚠/❌, so the category's lessons are inferred from business-model economics rather than from the operators' own account.
5. **The channel statistics everyone quotes are unsourced or misattributed** ❌ (§8.3).
6. **What *is* solid:** the mechanics of the products (✅ vendor first-party), the legal constraints in Singapore (✅ government), the fraud volumes (✅ IC3), the platform history (⚠ press), and one peer-reviewed warning about selection effects ✅.

**Guide's analysis.** That list is not a reason to avoid the reverse search; it is a reason to stop treating it as a strategy with a known effect size. The defensible claim is narrow: **publishing makes you findable to searchers who are already looking, at low marginal cost, with a documented fraud and privacy downside.** Nothing more is supported.

---

## 9. Risks and Costs

The reverse search's costs are not the ones people expect. They are not "time spent applying" — there is less of that. They are noise, exposure, and being a visible target.

### 9.1 Noise, spam, and the attention tax

**Practice.** The documented mechanics guarantee noise: outreach is a rationed, quota'd activity (150 InMails per seat per month ✅), agencies source continuously, and a profile with in-demand keywords appears in many lists. The candidate-side consequences are (a) an attention tax while employed, (b) a signal-quality problem — a first-rate inbound and a blast look identical in an inbox, and (c) **staleness enforcement by the platform**, since LinkedIn will automatically remove `#OpenToWork` if you stop responding to InMails ✅. The mitigation is a workflow (§5.5), not a filter.

### 9.2 Privacy and data exposure

- **Platform-documented limits.** *"We can't guarantee complete privacy"* ✅ (LinkedIn Help, on hiding career interests from your own employer's Recruiter users). Treat any expectation of airtight suppression as unmet.
- **Public permanence.** A repository, a talk recording, a page, and a structured profile are all indexed indefinitely; forks and caches make withdrawal unreliable.
- **Aggregation risk (Guide's analysis).** The interesting exposure is not the profile itself but its combination: name + employer + niche + availability + a contact address, assembled across surfaces, is exactly the dataset a targeted recruiter-fraud attempt needs. §9.4 shows the volume of that fraud.
- **Enriched external profiles.** Vendor marketing describes sourcing over 500M+ *enriched external* profiles ⚠ (Wellfound) — i.e. data about people beyond those who opted in. A candidate cannot opt out of a corpus they are not in, and an employer cannot easily explain its provenance. This is a governance question as much as a privacy one (see [ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md)).

### 9.3 Reputational risk with a current employer

**Practice, with one documented anchor.** The platform's own account of its suppression mechanism depends on a profile field *you* maintain — the company marked "I am currently working here" ✅. Three failure modes follow: an inaccurate employer flag; a parent/subsidiary mismatch between the recruiter's seat and your employer's legal entity; and the contingent case, where an agency seat staffed into your employer is not "a user who works at your company" and may see the signal. For contract-placed readers this is a material risk decision and not a cosmetic setting — the engagement, payment, and renewal structure that determines how exposed you are is the subject of [contract_job_smart_move.md](../technology/contract_job_smart_move.md), and this guide deliberately defers to it rather than duplicating it.

### 9.4 Recruitment fraud, quantified

The clearest documentation of the fraud aimed at findable candidates comes from the FBI's IC3, whose 2024 annual report is government primary source material ✅:

| IC3 2024 figure | Value |
| --- | --- |
| Employment-fraud complaints, 2024 | **20,044** |
| Employment-fraud losses, 2024 | **$264,223,271** |
| Employment complaints, prior years shown in IC3's three-year comparison | 15,443 (2023); 14,946 (2022) |
| Employment losses, same comparison | $70,234,079 (2023); $52,204,269 (2022) |
| All IC3 complaints and losses, 2024 | 859,532 complaints; **$16.6 billion** in losses, a 33% increase on 2023 |
| IC3's own definition of the crime type | *"Employment Fraud: An individual believes they are legitimately employed and loses money, or launders money/items during their employment"* |

Read the trend: employment-fraud losses reported to IC3 grew roughly **5×** between 2022 and 2024 ✅ (IC3's own comparison table). Caution: IC3 data is *self-reported complaints*, so it measures reporting as much as incidence ⚠ — a documented caveat, and the guide does not treat these as a prevalence estimate.

IC3's 2024 public-service-announcement appendix lists the specific patterns ✅: *"Scammers Defraud Individuals via Work-From-Home Scams"* (4 June 2024); *"North Korea Leverages U.S.-Based Individuals to Defraud U.S. Businesses"* (16 May 2024) — the IT-worker/infiltration pattern; *"Criminals Use Generative Artificial Intelligence to Facilitate Financial Fraud"* (3 December 2024); and *"Business Email Compromise: The $55 Billion Scam"* (11 September 2024).

**Singapore-side warnings are on the government's own platforms** ✅: MyCareersFuture's homepage carries the banner *"Government officials will NEVER ask you to transfer money or disclose bank log-in details over a phone call. Call the 24/7 ScamShield Helpline at 1799 if you are unsure if something is a scam,"* and links directly to the national job-scams guidance ✅.

**The patterns a published candidate should assume will arrive (Practice, grounded in the above):** fake recruiters using real employer names; interviews and offers without a matching, verifiable requisition; requests for identity documents, bank details, or a "personal reference" early; cheque-overpayment and equipment-purchase schemes; and money-mule recruitment, which IC3's definition names explicitly ✅. **Not verified:** deepfake video-interview fraud as a quantified pattern — the searches returned nothing primary, and it is listed ❌ in §14 rather than described.

### 9.5 Mitigations, in priority order

**Practice.** (1) Verify the employer independently — company register, own domain email, a named requisition, and a person you can reach on a channel you chose. (2) Never move money or financial identity data before a signed offer from a verified entity — the IC3 definition ✅ turns precisely on this. (3) Keep a canonical channel: a page or profile you control, referenced in your outbound replies, so the "real you" is the one with the stable URL. (4) Publish availability, not identifiers: no national ID, bank references, references' contact details, or home address in machine-readable form. (5) Treat inbound messages as untrusted input to any AI assistant you point at your mailbox — see [prompt_injection_guide.md](../technology/ai_llm/prompt_injection_guide.md). (6) Review your own exposure quarterly against §5.4's publish/withhold table.

---

## 10. The Singapore and Asia Context

The mechanics in §1–§9 are global. The *hirability* is not. Singapore is a small, tightly regulated market with a state-run job portal and a points-based foreign-hiring regime, and both of those change what a reverse search can achieve. This section is kept concrete because the guide's author's own domain is Singapore banking technology.

### 10.1 The local landscape, verified or flagged

| Property | What it is | Model | Evidence |
| --- | --- | --- | --- |
| **MyCareersFuture** | Singapore government job portal ("An Adapt and Grow initiative"), with job matching, career guidance, and employer-side posting; carries a national scam-warning banner | Publicly operated; **free for employers**; **mandatory advertising venue** for EP/S Pass applications under the Fair Consideration Framework | ✅ (portal; ✅ MOM's FCF page names it as the required advertising channel) |
| **JobStreet (SEEK)** | Asia-focused job board operating in Singapore (`sg.jobstreet.com`); self-describes as "Asia's favourite career…" | Employer-pays job postings | ✅ presence; ⚠ model detail — the SEEK corporate relationship was not extracted in this pass |
| **Glints** | Southeast Asian talent platform, Singapore-headquartered presence; free job posting with AI matching and a built-in ATS; managed hiring in 150+ countries | Employer-pays | ✅ vendor's own site for the model; ⚠ scale claims |
| **LinkedIn (Singapore)** | The regional instance of the global network; #OpenToWork semantics are global and documented ✅ | Employer-pays (Recruiter/Jobs) | ✅ feature docs; ❌ no Singapore-specific data located |
| **NodeFlair** | Singapore tech job/salary platform referenced in the local market | Not verified | ❌ — extraction failed; see §14 |
| **Contract/staffing market** | Agency-mediated contract placements are a large share of local technology hiring | Agency margin on the placement; candidate is employed by the agency or by an employer-of-record | ⚠ market norms, not a verified statistic; the structure itself is treated in [contract_job_smart_move.md](../technology/contract_job_smart_move.md) |

**Honest flag:** this guide found **no verified Singapore-specific reverse job board** operating on the candidate-publishes/employer-pays-contact model ❌. The local equivalents are general job boards, the government portal, and agencies. That is a real gap in the market and a real limit on §2's tactics locally.

### 10.2 The statutory frame that shapes everything: FCF

Verified at MOM, the Fair Consideration Framework ✅:

- All Singapore employers are expected to adhere to the **Tripartite Guidelines on Fair Employment Practices**, and must not discriminate on non-job-related characteristics such as age, sex, nationality, or race ✅.
- Employers submitting **Employment Pass or S Pass** applications **must first advertise on MyCareersFuture** and fairly consider all candidates, with a **mandatory advertising duration of at least 14 consecutive days**; **no job offer may be made during that period** ✅.
- Advertising exemptions exist, and employers are *"strongly encouraged"* to advertise on MyCareersFuture even where exempt ✅.
- MOM names the abuse pattern explicitly: *"some employers go through the motion of advertising on MyCareersFuture after pre-selecting a foreigner, and do not review the applicants"* — a breach that can lead to debarment ✅.
- Since January 2020 penalties were raised: minimum **work-pass debarment 12 months**, up to **24 months** for egregious cases, now covering renewals as well as new applications; and **false declarations that all candidates were fairly considered** carry imprisonment of up to 2 years, a fine up to $20,000, or both, under the Employment of Foreign Manpower Act ✅.

### 10.3 Pass thresholds: the numbers a candidate must clear

Verified at MOM's own eligibility pages ✅. EP is a **two-stage** test: qualifying salary, then the points-based **COMPASS** (candidates need **40 points** to pass; those failing stage 1 are ineligible regardless of COMPASS).

| Pass | Sector | Current minimum qualifying salary | New applications from 1 Jan 2027 (renewals expiring from 1 Jan 2028) |
| --- | --- | --- | --- |
| **Employment Pass** | All except financial services | **$5,600**/month, rising progressively with age from 23 to **$10,700** at 45+ | **$6,000**, up to **$11,500** at 45+ |
| **Employment Pass** | Financial services | **$6,200**, up to **$11,800** at 45+ | **$6,600**, up to **$12,700** at 45+ |
| **S Pass** | All except financial services | **$3,300**, up to **$4,800** at 45+ | **$3,600**, up to **$5,100** at 45+ |
| **S Pass** | Financial services | **$3,800**, up to **$5,650** at 45+ | **$4,000**, up to **$5,650** at 45+ |

COMPASS scoring, as documented ✅: **C1 Salary** (20 points at or above the 90th percentile of local PMET salary benchmarks for the sector; 10 points between the 65th and 90th; 0 below the 65th) and **C2 Qualifications** (20 points for degree-equivalent qualifications from top-100 QS universities, Singapore's autonomous universities, or agency-endorsed institutions; 10 for other degree-equivalent qualifications; 0 otherwise; **no points needed from this criterion if you do not need them**, and verification proof is required only when claiming points) ✅. Criteria C3 (diversity) and C4 (support for local employment) are computed from the employer's own PMET composition ✅. Employers can pre-check eligibility with MOM's **Self-Assessment Tool** ✅, and for S Pass MOM states that a positive SAT outcome is followed by a pass roughly 90% of the time ✅.

**What this means for the reverse search, concretely.** A Singapore employer sourcing a foreign candidate must still satisfy the advertising requirement before an EP/S Pass application, and must still clear stage 1 salary and COMPASS. So the honest statement is: **for many roles, no amount of candidate-side discoverability creates a hire that the pass framework forbids — and for roles that do clear it, the candidate's published profile is a sourcing aid, not a legal route.** Conversely, for Singapore citizens and PRs, the FCF requirement creates a genuine structural advantage on MyCareersFuture: the role *must* be advertised there for at least fourteen days before a pass application, which makes the government portal the one job surface where local candidates are guaranteed a look-in ✅ (a claim directly supported by the framework's design and MOM's statement of its purpose ✅).

### 10.4 Recruiter-market norms and the contract market

**Honest scope note:** the local recruiter-market norms this guide could *verify* are thin. What is verified is structural rather than statistical — that MOM holds agencies to the FCF and the Tripartite Guidelines ✅, that MyCareersFuture is the mandatory advertising venue ✅, and that the agency/contract structure dominates a meaningful share of technology hiring (⚠ practitioner knowledge, no verified statistic located). Practical implications (Practice): expect agency-mediated inbound to dominate your qualified contacts locally; expect the "which entity employs me" question to arrive earlier than in other markets; and read [contract_job_smart_move.md](../technology/contract_job_smart_move.md) for the day-rate premium, entity structuring, and tax position, because this guide's boundary stops at discovery.

### 10.5 Where local evidence is thin — stated, not generalised

1. **No published Singapore data on reverse-search or candidate-publishing effectiveness** ❌.
2. **No verified local reverse job board** ❌ (§10.1).
3. **No verified local recruiter-sourcing statistics** (e.g. share of hires sourced versus posted) ❌.
4. **No verified data on how COMPASS interacts with sourcing channels** — only the published criteria and points ✅, not outcomes.
5. **Regionally,** the Glints and JobStreet/SEEK models are verified at the vendor site level ✅/⚠; **no comparable primary source was found for Indonesia, Malaysia, Vietnam, or Thailand markets** ❌, and this guide does not generalise from Singapore to the region.

---

## 11. Worked Example: Cymbal Bank's Reverse Talent Model

> **This section is illustrative and fictional.** No such programme exists, and **every number in it is hypothetical and constructed for the example** — conversion rates, counts, costs, and timelines. The *regulatory and statutory references* are real and marked ⚠/✅; the *design* is the author's proposal from the employer's chair. Read it as a template an employer could pilot, not as a report of anything that happened.

### 11.1 The hiring problem, stated as a search problem

**Setting (fictional):** Cymbal Bank's technology organisation needs people who can work on two scarce seams at once — **mainframe-to-event-stream modernisation** on the core banking side, and **agent-platform engineering** on the AI side (the harness/evaluation/observability discipline covered in [agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md) and [agentops_guide.md](../technology/ai_llm/agentops_guide.md)). Two properties make this a good reverse-search case (Guide's analysis):

1. **The candidate set is small and publicly identifiable.** Practitioners in both seams publish — repositories, postmortems, conference talks, standards discussions.
2. **Posting markets fail for it.** A generic "senior engineer" advert attracts volume and no signal; a narrow advert attracts almost nobody because the population is tiny. Sourcing is the only channel that scales against a small population.

### 11.2 The talent-market map: where these people actually publish

| Population | Where they publish | Searchable surface |
| --- | --- | --- |
| Mainframe/modernisation specialists | Legacy-community forums, vendor user groups (COBOL/assembler/IMS/CICS communities), migration postmortems, conference archives | Named talks, repo READMEs with domain vocabulary, forum handles |
| Agent-platform engineers | GitHub repositories, evaluation and observability write-ups, model/harness docs, the agent-engineering literature the `ai_llm/` cluster indexes | Repos, commit history, issue threads, published harness and eval designs |
| Both, if senior | Personal sites, talks, standards and working groups, professional-association venues | The §4.4 stack: page, structured data, artefacts |

Note the §1.4 discipline: on the open web, the *publication venue is the directory*. There is no single reverse board for either seam ❌ (§10.1), which is why the employer-side design below is built on **searcher discipline plus a curated pool**, not on a purchased inventory.

### 11.3 The published proof-of-work signals Cymbal would search for

**Guide's analysis (constructed).** These are query-shaped artefacts, chosen because they are verifiable by a stranger and specific enough to search:

- a **migration postmortem** naming the failure mode (e.g. batch reconciliation breaks under event replay) with real, non-confidential detail;
- a **public harness or evaluation design** — a scoring rubric, a golden-set policy, a tracing schema — of the kind the repo's `ai_llm/` cluster treats as first-class engineering;
- **sustained contribution history** in a relevant repository, readable as process rather than a single commit;
- a **talk or written specification** endorsed by a programme committee;
- a **working system with named constraints** — throughput, latency, failure behaviour — that a reviewer can reproduce.

Deliberately **not** searched for: job titles, employer brands, years of experience, or credentials alone. Those are the fields everyone has and therefore the fields that carry no discriminative power (Guide's analysis).

### 11.4 The sourcing funnel, with illustrative conversion rates

**All figures below are hypothetical.** They are structured as a funnel with explicit stage definitions so a pilot could replace each with a measured value (§11.7).

| Stage | Definition | Illustrative count | Illustrative conversion |
| --- | --- | --- | --- |
| Published-artefact search | Searcher runs the §11.3 queries across public surfaces | 400 candidate-artefacts reviewed | — |
| Longlist | Candidate matches niche *and* published proof-of-work | 60 | 15% of artefacts reviewed |
| Verified contact | Named, reachable, with a public route (no scraped personal data) | 30 | 50% of longlist |
| Qualified conversation | 20-minute call; niche, engagement type, work-authorisation path clear | 12 | 40% of verified contacts |
| Assessment gate | §11.5 technical gate passed | 5 | ~42% of conversations |
| Panel and offer | Hiring panel; offer made | 2 | 40% of assessments |
| Hire | Offer accepted, pass application (where needed) approved | 1 | 50% of offers |

**Guide's analysis of the shape.** The funnel's cost is concentrated in stages 1–3 and consumed by the funnel's *own* perishability: candidates contacted at stage 3 are the same candidates every competitor is contacting, and the qualified ones exit fastest ⚠ (the perishable-inventory dynamic of §2.2). A pilot's real cadence target is therefore *time from artefact discovered to first conversation*, not the number of artefacts reviewed.

### 11.5 The screening and assessment gates

**Guide's analysis (constructed, and deliberately artefact-anchored):**

1. **Public-work review** — a reviewer reads the published artefact and drafts questions from it. Pass criterion: the candidate can defend the published design under questioning, including its failures.
2. **Scoped technical exercise on non-production data** — for the modernisation seam, a small migration/replay problem; for the agent seam, a harness or evaluation design with an explicit failure-mode analysis. Same exercise for all candidates on a given track, scored against a written rubric with anchored levels.
3. **Systems and risk conversation** — the regulated-industry component: change control, data handling, and what the candidate does when a verification fails.
4. **Panel, then decision** — structured, with the rubric retained as evidence (which also produces the fairness record MOM expects, see §11.6).

Note the ordering choice: the **published artefact is the entry ticket and the first evidence**, not a substitute for the exercise. This is how a reverse-search employer avoids the "impressive public work, unverifiable under constraints" failure.

### 11.6 Compliance and fairness constraints a regulated employer must observe

Real instruments, verified or flagged as required:

- **Fair Consideration Framework (FCF), MOM** ✅ — advertising on MyCareersFuture for at least 14 consecutive days before an EP or S Pass application, fair consideration of all candidates, no offer during the mandatory advertising window, and penalties including 12–24 month work-pass debarment plus prosecution for false declarations (up to 2 years' imprisonment, or a fine up to $20,000, or both) ✅. **Design consequence:** a reverse-sourced foreign candidate cannot skip the advertising requirement, and the sourcing programme must be *additive* to the advertised process, with records showing the advertised applicants were fairly considered. MOM itself names the "advertise after pre-selecting" pattern as the breach to avoid ✅.
- **Tripartite Guidelines on Fair Employment Practices (TAFEP)** ✅ — the standard MOM expects all employers to apply; sourcing must not filter on age, sex, nationality, or race, which directly constrains the search criteria in §11.3 (search on *artefacts*, never on protected characteristics).
- **Adverse-impact discipline on AI screening (Guide's analysis, flagged as practice rather than a verified requirement).** Where AI assistance ranks or filters candidates (as the §7.4 products do), a regulated employer should be able to show the criteria used, the data used, and the pass rates by group. **No Singapore-specific statutory instrument imposing AI hiring audits was verified in this pass** ❌ — the requirement here is the employer's own governance position, and the repo's frame for it is [ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md) and the bias-red-teaming material in the `ai_llm/` cluster.
- **Personal Data Protection Act 2012 (PDPA)** ✅ as a statute (Singapore Statutes Online, `sso.agc.gov.sg/Act/PDPA2012`) — with the specific obligations most relevant to sourced candidates (collection, use, notification, retention limitation, and consent) documented in the Act's own provisions ✅; the PDPC's published guidance on the personal data of **job applicants and employees** exists as a first-party PDF ⚠ (located but not extracted in this pass). **Design consequence:** sourcing from *publicly published* candidate artefacts is not the same as sourcing from a purchased enrichment corpus, and the latter is where consent, notification, and retention questions bite. The 500M+ *enriched external profiles* model described by one vendor ⚠ is exactly the pattern a regulated employer should stress-test before buying.
- **MAS instruments — an honest flag.** The commonly cited Singapore banking-conduct instrument, the **MAS Guidelines on Fair Dealing**, is a *customer* fair-dealing standard; the URL consulted returned 404 in this pass, and **this guide could not verify that any MAS instrument imposes employment-fairness duties on hiring** ❌. The employment-side duties here are TAFEP/MOM's ✅. Do not cite MAS as a hiring-fairness authority without a checked source.
- **Data-protection and residency habit (Guide's analysis).** Candidate data gathered through sourcing should land in the employer's own records with a defined retention period, not in a recruiting tool's tenancy with no exit. That is a procurement requirement before it is a technical one — see [vendor_management_guide.md](vendor_management_guide.md).

### 11.7 The metrics Cymbal would track

**All definitions in this table are constructed by this guide** for the fictional programme. They are chosen so that each has an owner and a falsifying threshold.

| Metric | Definition | Illustrative target | What failure means |
| --- | --- | --- | --- |
| **Artefact-to-conversation rate** | Qualified conversations ÷ artefacts reviewed | 3% | Search criteria are wrong, not the market |
| **Sourcing latency** | Days from artefact discovery to first conversation | ≤ 14 | Slow process → the best candidates are gone (perishable inventory) |
| **Advertised-applicant fairness record** | Share of advertised applicants screened, with criteria, per requisition | 100% recorded | FCF exposure ✅ |
| **AI-screening transparency** | Criteria documented, pass rates by group reviewable | Maintained | Governance gap (§11.6) |
| **Assessment discrimination** | Spread of assessment scores within a track | Comparable bands across sources | Rubric is not discriminating |
| **Offer acceptance** | Acceptances ÷ offers on sourced candidates | Tracked, not targeted | Either a salary problem or an engagement-type mismatch — see [contract_job_smart_move.md](../technology/contract_job_smart_move.md) |
| **Time-to-productivity** | Weeks to first independent contribution | Tracked against posted-hire cohort | The real test of whether sourcing produced a better hire |
| **Cohort comparison** | Attrition and performance at 12 months, sourced vs posted | Tracked | The only comparison that ages the claim |

**What would falsify the programme (Guide's analysis):** if the cohorts do not differ at 12 months, or if the advertised-applicant fairness record cannot be produced on demand, the reverse model is an expensive detour and should be stopped. That is the same discipline the repo applies in [vendor_management_guide.md](vendor_management_guide.md) and the same one the vendor-side numbers in §7.2 conspicuously lack.

---

## 12. One-Page Summary

**The model.** The reverse job search inverts the funnel: the candidate publishes, the employer searches. It is *not* merely having a LinkedIn profile — that is being indexed, not published. The functional test is whether a hiring manager who does not know your name can find you by describing their problem. The *term itself* is loose and contested ❌; the adjacent term "reverse job board" has firmer provenance (a 2012 one-page "reverse job application"; a real category from 2021), and practitioner use of "reverse job search" spans two opposite practices — publishing yourself, and messaging the human behind a posting first.

**The landscape.** The original Hired (San Francisco, 2012, launched as DeveloperAuction) pioneered the auction mechanic ⚠; Vettery (New York, 2013) ran a vetted candidate database ⚠/✅; Vettery acquired Hired in Nov 2020, merged the brands in Mar 2021, and the whole thing folded into LHH (Adecco) on **14 June 2024** — no standalone marketplace survives ⚠/✅. **RailsDevs** (Nov 2021) worked and closed anyway: ~600 developers, 30+ hires in year one, ~$146K first-year revenue, and **~90% of revenue from hiring fees rather than subscriptions** ⚠ (archived July 2025). Living exemplars are niche and employer-funded (ElixirDevs: free for developers, companies pay ✅). The reverse model *at scale* is LinkedIn Recruiter and the résumé databases — employer-pays search over candidate-published profiles ✅.

**Signals.** LinkedIn documents `#OpenToWork` as three visibility options, feeding your profile into recruiter search, with the photo frame only in the fully public setting; India-only notice/salary fields are recruiter-visible regardless of your setting; the feature is **automatically removed** if you stop responding; and the platform states plainly that it *"can't guarantee complete privacy"* from your own employer's Recruiter users ✅.

**The playbook.** Narrow to a niche small enough to be searched by name; publish stranger-verifiable artefacts; place the market's own vocabulary where recruiter search and crawlers index it; publish availability but not identifiers; run inbound through a time-boxed triage workflow; and keep score with the constructed metrics in §5.6. The personal site plus structured data (JSON Resume ✅, schema.org `Person` ✅) is the durable destination; no platform in §2 should be treated as permanent.

**The AI era.** Auto-apply tools and employer AI sourcing both verifiably exist ✅; the "AI versus AI" loop is asserted, not verified ❌; vendor outcome claims are ⚠. The consequence is to compete on artefacts, because artefact supply is the one input automation does not inflate.

**The evidence base — the guide's most important conclusion.** There is **no study of reverse-job-search outcomes** ❌. The closest peer-reviewed work (Kuhn & Skuterud, AER 2004) found internet job search's apparent advantage **disappears once selection is controlled for**, and possibly lengthens spells ✅. The "80% of jobs are never advertised" statistic traces to a **1966 Rochester survey shaped by a local employers' no-advertising agreement**, retrofitted to a number its author already used ❌ — do not cite it. The channel statistics in wide circulation are unsourced.

**Risks.** Noise and an attention tax (mitigate with workflow); privacy limits the platform states itself ✅; reputational exposure to a current employer, worst for contingent workers; and quantified recruitment fraud — IC3 recorded **20,044 employment-fraud complaints and $264.2M in losses in 2024**, roughly 5× the 2022 loss figure, with dedicated advisory notices ✅. Never move money or financial identity data before a verified, signed offer.

**Singapore.** The FCF requires advertising on MyCareersFuture for **at least 14 consecutive days** before EP/S Pass applications, with debarment of 12–24 months and prosecution for false declarations ✅; EP requires a qualifying salary (**$5,600**, or **$6,200** in financial services; rising to **$6,000/$6,600** for applications from 1 Jan 2027) **plus** 40 COMPASS points, and S Pass requires **$3,300** (**$3,800** FS; rising to **$3,600/$4,000**) ✅. Discoverability does not route around these, and there is no verified Singapore reverse job board ❌.

**The verdict.** The reverse search is cheap, real, and unmeasured. Publish deliberately, narrow hard, handle inbound with a process, expect the noise and the fraud, and do not let anyone sell you a statistic.

---

## 13. Claims Audit

Every load-bearing claim in this guide, with its marker and source. Read the ⚠ rows as claims, not measurements.

| Claim | Marker | Source | Note |
| --- | --- | --- | --- |
| "Reverse job search" is not a settled industry term; a vendor guide defines it as a tactic with no operator or marketplace | ⚠ | Cavuno (vendor blog), 10 Jul 2026 | Vendor source, but the definition works against its own product interest |
| The 2012 "reverse job application" one-pager is the origin artefact of the category | ⚠ | Cavuno | Artefact itself gone; unverifiable at first party |
| Candidate profiles are perishable inventory; success consumes supply | ⚠ | Cavuno | Vendor analysis; agrees with the RailsDevs arc |
| The original Hired was founded 2012 in San Francisco as DeveloperAuction, with the auction mechanic | ⚠ | Aggregators relaying a 2014 TechCrunch report | Source article URL 404s (§14) |
| Vettery founded Mar 2013 in New York; launched US Jun 2015; $1.7M seed; $9M Series A 2016; $11.9M total | ⚠ | Wikipedia relaying TechCrunch/AlleyWatch | Press-relayed, internally consistent |
| Vettery claims ~5% of applying candidates were accepted | ⚠ | Onrec, 14 May 2018 (launch release) | **Company claim in a press release**; unauditable |
| Vettery acquired by Adecco Feb 2018 at "a little over $100 million" | ⚠ | Wikipedia relaying TechCrunch | Terms never disclosed; Reuters reported without price |
| Vettery acquired Hired Nov 2020; brands merged Mar 2021 | ⚠ | Wikipedia relaying TechCrunch | Corroborated across sources |
| Hired folded into LHH Recruitment Solutions on 14 Jun 2024; hired.com redirects to LHH; self-serve closed | ⚠/✅ | Wikipedia; sourcingtools.org review | Directory corroborates redirect; LHH page fetch was blocked |
| RailsDevs: launched Nov 2021; ~100 profiles week one; 600+ developers and 30+ hires in year one; ~$146K first-year revenue; ~$10K/month best | ⚠ | Cavuno relaying Starter Story | Two removes from the operator's own data |
| ~90% of RailsDevs revenue came from hiring fees, not subscriptions | ⚠ | Cavuno | The category's defining economic fact |
| RailsDevs archived Jul 2025; no published post-mortem | ⚠/❌ | Cavuno | Operator's farewell note reportedly gave no operational cause |
| ElixirDevs is live, free for developers, paid for companies, 900+ profiles | ✅/⚠ | elixirdevs.com | Model and pricing first-party; count is vendor marketing |
| Wellfound candidate side is free; employer posting free; Reach/Autopilot paid | ✅ | wellfound.com FAQ | First-party pricing statement |
| Wellfound: 10M+ opted-in candidates; 500M+ enriched external profiles; 27,000+ startups | ⚠ | wellfound.com | Unauditable vendor counts |
| LinkedIn Recruiter: 40+ filters/Boolean, 150 InMails per seat per month, ATS/CRM integrations | ✅ | LinkedIn Recruiter product page | First-party mechanics |
| LinkedIn AI outcome claims: <5 min to engage a candidate; 4+ hours saved; 62% fewer profiles; 37% lower attrition; 55% higher InMail acceptance | ⚠ | Recruiter page, footnote "LinkedIn data (2024)" | Vendor's own data, no methodology published |
| `#OpenToWork`: three visibility settings, the photo frame only in the public setting, influence over recruiter search, automatic removal after non-response, and India-only salary/notice fields visible to recruiters regardless of setting | ✅ | LinkedIn Help | One platform document supplies all five facts |
| LinkedIn suppresses your signal from your current employer's Recruiter users but "we can't guarantee complete privacy" | ✅ | LinkedIn Help | The single most important privacy disclosure in this guide |
| JSON Resume (open structured-résumé standard) and schema.org `Person` are open, adopted structured-profile vocabularies | ✅ | jsonresume.org; GitHub repo; schema.org/Person | First-party; the 10M+ domains figure is the publisher's own usage statistic |
| Simplify: free for candidates, monetises via Simplify+ subscriptions and employers, indexes 50+ job boards | ✅/⚠ | simplify.jobs | Monetisation statement first-party; user count is vendor marketing |
| IC3 2024: 20,044 employment-fraud complaints and $264,223,271 in losses (2023: 15,443/$70,234,079; 2022: 14,946/$52,204,269); totals 859,532 complaints and $16.6B losses, +33% on 2023; advisories on work-from-home scams, state-actor infiltration of US employers, and generative-AI-enabled fraud | ✅ | IC3 2024 Internet Crime Report | Government primary; **self-reported complaint data ⚠**; advisory titles and dates as published |
| MyCareersFuture carries a national scam warning and links to ScamShield job-scam guidance | ✅ | mycareersfuture.gov.sg | Government platform |
| FCF: advertise on MyCareersFuture ≥14 consecutive days before EP/S Pass applications; no offer during the window | ✅ | MOM FCF page, updated 8 Aug 2025 | Regulator primary |
| FCF penalties: 12 months' minimum work-pass debarment, up to 24; false declaration → up to 2 years' imprisonment and/or fine up to $20,000 | ✅ | MOM FCF page | Regulator primary |
| EP: qualifying salary $5,600 (all other sectors) or $6,200 (financial services), rising to $6,000/$6,600 for applications from 1 Jan 2027, **plus** COMPASS at 40 points (salary C1 and qualifications C2 each scored 20/10/0) | ✅ | MOM EP eligibility, updated 28 Apr 2026 | Regulator primary; age-graduated tables apply |
| S Pass: $3,300 (other sectors) or $3,800 (financial services), rising to $3,600/$4,000; a positive Self-Assessment Tool outcome is followed by a pass roughly 90% of the time | ✅ | MOM S Pass eligibility, updated 1 Sep 2026 | Regulator primary |
| Kuhn & Skuterud: internet job search advantage disappears on controls; estimates become counterproductive; conclusion conditional on strong negative selection | ✅ | IZA DP 613 (2002); AER 94(1) 2004 | Peer-reviewed; **dated 1998–2000 data** |
| BLS OOQ reports organisations fill many openings through an unadvertised "hidden" job market, and that specialists boards improve odds | ⚠ | BLS OOQ, Spring 2011 | Institutional publisher, but advice quoting practitioners, not a study |
| "80% of jobs are never advertised" traces to a 1966 Ford Foundation–funded Rochester survey shaped by a local employers' no-advertising agreement — a figure its author (Bernard Haldane) had already been using before the survey, later propagated via Irish (1973) and *What Color Is Your Parachute?* (1977) | ⚠ | Learning Curves, Parts 1–2 (employment specialist) | Secondary archival account, heavily sourced but not peer-reviewed |
| "As of 2010, less than 10% of U.S. jobs are filled through online ads" | ❌ | Wikipedia, citing a newspaper column that "references uncited studies" | Unsourced at origin |
| "244 applications per posting" | ❌ | Untraceable | Circulates only in vendor blogs |
| AI-written applications meeting AI screening degrades outcomes | ❌ | No source located | Plausible, unverified |
| Deepfake video-interview fraud is a quantified pattern | ❌ | No primary source located | Listed in §14 |
| No study measures reverse-job-search outcomes | ❌ | — | The guide's central negative finding |
| Cymbal Bank reverse talent model: all numbers, rates, and targets | ❌ (hypothetical) | Constructed for §11 | **Explicitly fictional; illustrative only** |

---

## 14. What Could Not Be Verified

The ❌ list, stated as gaps rather than glossed. Each of these is an honest hole in the guide, not an inference presented as a finding.

1. **The number of people who practise the reverse job search,** in any market, at any time. No operator publishes it.
2. **Any measured outcome of the reverse job search** — hires, pay, tenure, or satisfaction — for candidates or employers. No study, no platform cohort data, no independent audit.
3. **The original 2012 "reverse job application" page's content, and whether "README-as-CV" is a documented, named convention** — the artefact is gone, and the convention has anecdotes but no primary description of its mechanics or results.
4. **The 2014 TechCrunch article on DeveloperAuction/Hired (its URL 404s) and the original Hired's exact funding and peak valuation** — aggregators give "$133M" and "over $150M", and the ~$500M valuation is single-source ⚠.
5. **The exact price and terms of the Vettery and Hired acquisitions.** "A little over $100 million" for Vettery was reported from an unnamed source; Hired's price was undisclosed.
6. **Any Hired, Vettery, or RailsDevs post-mortem.** No operator published a causal account of failure or closure.
7. **Triplebyte's history, pivot, and shutdown, and NodeFlair's model and status** — searches and extraction returned nothing usable for either.
8. **Indeed's employer pricing and its résumé-database monetisation.** The pricing page failed to retrieve; no Indeed claim is made beyond the existence of the category ⚠.
9. **Any Singapore-specific reverse job board, and any Singapore recruiter-sourcing statistics** (share of hires sourced versus posted; agency share of placements).
10. **Any regional (non-Singapore) Southeast Asian primary source** on reverse-marketplace or candidate-publishing practice.
11. **"244 applications per posting" and the "85% of jobs are found through networking" variant** — no primary source located for either; the networking figure traces only to an "estimated 50%+" assertion attributed to a BLS article that does not measure it.
12. **Documented backlash against the public `#OpenToWork` frame, and evidence that AI-written applications systematically meet AI screening at scale** — both are widely asserted, and neither has a primary source, platform dataset, or study behind it.
13. **Any documented case of ATS parsing of `resume.json` or schema.org `Person` markup affecting an outcome.** No ATS vendor documentation found.
14. **Deepfake or synthetic-video interview fraud as a quantified pattern.** No primary, quantified source located in this pass.
15. **Whether any MAS instrument imposes employment-fairness duties on hiring** (the page consulted returned 404), **and the PDPC's guidance PDF on job applicants' and employees' personal data** (located by URL, not extracted; cited ⚠).
16. **Any auto-apply tool's measured effect on callback rates, and whether candidate-publishing improves *either* party's outcomes relative to alternatives** — unmeasured by definition (§8.4).

**Tooling failures encountered.** Web search returned empty result sets for several queries (rate-limiting); those were retried with different phrasings or replaced by direct extraction of primary URLs, consistent with the documented fallback. Direct extraction failed for: the LHH Hired page (blocked as a private/internal address), the Indeed employer pricing page, nodeflair.com, sg.jobstreet.com, the PDPC PDPA page, gobusiness.gov.sg, the MAS fair-dealing guidelines URL (404), the TechCrunch DeveloperAuction article (404), and bls.gov's Monthly Labor Review index. Nothing was substituted for these failures.

---

## 15. Glossary

| Term | Definition |
| --- | --- |
| **Reverse job search** | The practice of publishing a durable, searchable representation of your capability and relying on the hiring side to search, match, and initiate contact. Distinguished in this guide from *being indexed* (having a profile) and from *front-door outreach* (finding the human behind a posting and messaging them first). |
| **Reverse job board** | A marketplace whose inventory is candidate profiles rather than job postings; candidates list themselves, employers search and pay to make contact. Origin: the 2012 one-page "reverse job application"; category: RailsDevs (2021–2025) ⚠. |
| **Inbound recruiting** | Hiring in which the employer initiates contact with candidates it has sourced, rather than processing candidates who applied. |
| **Candidate-as-product** | The framing, transplanted from marketplace economics, in which candidate supply is the inventory and employer access is the product ⚠. Carries the warning that candidate inventory is perishable and that success consumes it. |
| **Talent marketplace** | A managed matching platform that inserts itself into matching, vetting, and often payment, taking a share — e.g. Hired at its peak, Toptal, A.Team ⚠. Thicker than a reverse job board: it sells matchmaking, not access. |
| **Sourcing** | The employer-side activity of finding candidates who did not apply, typically via search over a profile database, an AI matching engine, or public artefacts. |
| **ATS (applicant tracking system)** | The system of record for applications and candidates; in this guide, notable for the **unverified** claim that résumé formatting is decisive because an ATS "reads" it. |
| **Talent pool / CRM** | A retained, searchable store of candidates an employer has previously encountered or sourced, integrated with the ATS. Vendor-documented as an integration surface ✅ (LinkedIn Recruiter). |
| **Open to work / #OpenToWork** | LinkedIn's availability signal: three visibility settings, influence over recruiter search, an optional public photo frame, automatic removal after non-response, and an explicit privacy caveat ✅. |
| **Proof-of-work** | A published artefact a stranger can evaluate without trusting the candidate: code with history, a written postmortem or design, a talk, a reproducible result. The currency of the reverse search (§5.2). |
| **Employer branding** | The employer-side mirror of the candidate's published profile: the deliberate public representation of what it is like to work somewhere, aimed at making sourcing easier. Out of scope here beyond the mirror-image observation. |
| **The hidden job market** | In this guide, **folklore**. A phrase for roles filled or discovered outside formal postings; its canonical "80% of jobs" statistic traces to a 1966 Rochester survey shaped by a local employers' no-advertising agreement ❌ (§8.3). Defined so that it cannot be used as evidence. |
| **FCF (Fair Consideration Framework)** | Singapore's framework requiring employers to consider the local workforce fairly; for EP/S Pass applications it mandates advertising on MyCareersFuture for at least 14 consecutive days and prohibits offers during that window ✅. |
| **COMPASS** | Singapore's points-based Complementarity Assessment Framework for EP applications; candidates must earn 40 points across salary, qualifications, diversity, and local-employment criteria ✅. |
| **EP / S Pass** | Singapore's employment passes for foreign professionals (EP: $5,600 minimum, $6,200 in financial services, plus COMPASS) and mid-skilled workers (S Pass: $3,300, $3,800 in financial services), each graduating with age ✅. |
| **PDPA** | Singapore's Personal Data Protection Act 2012, governing collection, use, and disclosure of personal data by organisations ✅; the framework around candidate data gathered through sourcing (§11.6). |
| **Scraped / enriched profile** | Candidate data assembled from public and commercial sources rather than supplied by the candidate. Described in vendor marketing as a 500M+ profile corpus ⚠; a compliance problem before it is a capability. |
| **Perishable inventory** | The model property that a candidate profile is useful only until the candidate is hired, and that the most desirable profiles leave the pool fastest ⚠. |
| **Structural data / machine-readable profile** | Career data in a consumable format: JSON Resume (a structured résumé document ✅) and schema.org `Person` (a crawlable vocabulary ✅). Discoverability plumbing, not a hiring channel. |

---

## 16. References

**Primary — first-party platform and product documentation (model and mechanics; outcome claims within them are ⚠)**

1. LinkedIn Help. *"Let recruiters know you're Open to Work."* <https://www.linkedin.com/help/linkedin/answer/a507508> — the three visibility settings, the photo-frame rule, the auto-removal policy, the India-specific fields, and the *"we can't guarantee complete privacy"* caveat. **The most important candidate-side source in this guide.**
2. LinkedIn. *"LinkedIn Recruiter + Hiring Assistant."* <https://business.linkedin.com/talent-solutions/recruiter> — 40+ filters and Boolean, 150 InMails per seat per month, ATS/CRM integrations, and the AI outcome claims (each footnoted to LinkedIn data). **Vendor marketing; mechanics usable, percentages not.**
3. Wellfound. <https://wellfound.com/> — candidate-free and employer-pays structure, 10M+ opted-in candidates, 500M+ enriched external profiles, Reach and Autopilot. **Vendor; counts unauditable.**
4. Glints (Singapore). <https://glints.com/sg> — free job posting, AI matching, built-in ATS, managed hiring across 150+ countries. **Vendor.**
5. ElixirDevs. <https://elixirdevs.com/> — the living niche reverse board: free for developers, paid for companies, two-state availability signalling, 900+ profiles. **Vendor; the model statement is the useful part.**
6. Simplify. <https://simplify.jobs/> — résumé scoring, autofill, tracker, 50+ indexed boards, and its own monetisation statement. **Vendor.**
7. JSON Resume. <https://jsonresume.org/> · schema repository <https://github.com/jsonresume/resume-schema> — the open structured-résumé standard; the old repo is archived and development moved to the monorepo, npm package unchanged.
8. schema.org. *"Person."* <https://schema.org/Person> — the `Person` vocabulary and its reported usage (10M+ domains, Google web index, July 2026).

**Primary — government and regulator**

9. Ministry of Manpower (Singapore). *"Fair Consideration Framework (FCF)."* <https://www.mom.gov.sg/employment-practices/fair-consideration-framework> — the 14-day advertising requirement, the offer prohibition, the named abuse pattern, and the 2020 penalty increases. Updated 8 Aug 2025.
10. MOM. *"Eligibility for Employment Pass."* <https://www.mom.gov.sg/passes-and-permits/employment-pass/eligibility> — two-stage eligibility, qualifying salaries by sector and age, COMPASS scoring. Updated 28 Apr 2026.
11. MOM. *"Eligibility for S Pass."* <https://www.mom.gov.sg/passes-and-permits/s-pass/eligibility> — S Pass thresholds and the SAT outcome statement. Updated 1 Sep 2026.
12. Federal Bureau of Investigation, Internet Crime Complaint Center. *2024 Internet Crime Report.* <https://www.ic3.gov/AnnualReport/Reports/2024_IC3Report.pdf> — employment-fraud complaint and loss totals, the three-year comparison, the crime-type definition, and the advisory appendix. **Self-reported complaint data ⚠.**
13. MyCareersFuture (Singapore). <https://www.mycareersfuture.gov.sg/> — the government job portal, its scam-warning banner and ScamShield job-scam link. **Mandatory advertising venue under the FCF.**
14. Personal Data Protection Act 2012, Singapore Statutes Online. <https://sso.agc.gov.sg/Act/PDPA2012> — the statute governing candidate data in sourcing (§11.6). Page located via index; PDPC's job-applicant guidance PDF located but not extracted ⚠.
15. TAFEP, *Tripartite Guidelines on Fair Employment Practices* (linked from the MOM FCF page) — the non-discrimination standard MOM enforces.

**Primary — peer-reviewed and named-institution research**

16. Kuhn, Peter, and Mikal Skuterud. *"Internet Job Search and Unemployment Durations."* IZA Discussion Paper 613, October 2002; *American Economic Review* 94(1), March 2004, 218–232. <https://ftp.iza.org/dp613.pdf> — positive selection on observables, negative on unobservables, no employment advantage on controls, counterproductive duration estimates. **The evidence base's centre of gravity.**
17. Vilorio, Dennis (BLS). *"Focused jobseeking: A measured approach to looking for work."* *Occupational Outlook Quarterly*, Spring 2011. <https://www.bls.gov/careeroutlook/2011/spring/art01.pdf> — practitioner-quoted advice on focusing a search, specialised boards, and the "hidden" job market. **Institutional, but not a study ⚠.**

**Secondary — provenance, history, and trade coverage**

18. Preston, Jesse. *"Hidden Origins of the Hidden Job Market," Parts 1 and 2.* Learning Curves. <https://learningcurves.org/hidden-origins-of-the-hidden-job-market-pt-2/> — the archaeological trace of the 80% statistic to Haldane, the Ford Foundation–funded Rochester survey, and the Rochester Industrial Management Council's no-advertising request. **The single most useful source for §8.3; secondary, and not peer-reviewed ⚠.**
19. Cavuno. *"What Is a Reverse Job Board? How the Model Works, Who Tried It, and What Operators Should Learn."* 10 July 2026. <https://cavuno.com/blog/what-is-a-reverse-job-board> — the category history, the RailsDevs revenue composition, the perishable-inventory analysis, and the definitional distinction between reverse job boards, reverse recruiting, and the reverse job search. **Vendor-published, but its analysis cuts against its own product pitch.**
20. Wikipedia. *"Hired (company)."* <https://en.wikipedia.org/wiki/Hired_(company)> — the Vettery/Hired acquisition sequence with TechCrunch and Reuters citations. **Tertiary; used only where corroborated.**
21. SourcingTools.org. *"Hired review."* <https://www.sourcingtools.org/tools/hired/> — the LHH absorption, the closed self-serve sign-up, and the historical pricing estimate. **Third-party directory; pricing explicitly labelled directional.**
22. Onrec. *"Vettery launches in the UK."* 14 May 2018. <https://www.onrec.com/news/launch/vettery-launches-in-the-uk> — the screened-candidate model and the 5% acceptance claim. **Trade coverage of a company press release.**
23. TechCrunch. *"Vettery acquires Hired to create a 'unified' job search platform."* 23 November 2020. <https://techcrunch.com/2020/11/23/vettery-acquires-hired/> — the acquisition announcement.

**Cross-referenced sibling guides in this repository (pointers, not sources)**

24. [../technology/contract_job_smart_move.md](../technology/contract_job_smart_move.md) — the engagement-structure decision (contract vs permanent, day-rate premium, SG structuring, portfolio careers). **The explicit boundary of this guide:** that guide answers *what type of engagement*, this one answers *how the candidate is discovered*. Cross-referenced from §1.6, §5.5, §9.3, §10.1, §10.4, §11.7.
25. [../technology/professional_networking_guide.md](../technology/professional_networking_guide.md) — the repo's personal-branding content; §1.4 and §5.1 assume it rather than repeating it.
26. The `../management/` human-skills cluster — [authority_skills_guide.md](authority_skills_guide.md), [the_first_90_days_guide.md](the_first_90_days_guide.md), [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [grow_team_guide.md](grow_team_guide.md) — the skills the published candidate must already have (§1.6, §5.2); — the interview cluster ([../technology/google_system_design_interview_guide.md](../technology/google_system_design_interview_guide.md), [../technology/system_design_interview_insiders_guide.md](../technology/system_design_interview_insiders_guide.md), [../technology/ml_system_design_interview_guide.md](../technology/ml_system_design_interview_guide.md)) is a later phase, of which §11.5 borrows the assessment-discipline point only.
27. [vendor_management_guide.md](vendor_management_guide.md) — the sourcing and vendor-evaluation discipline §7.5 and §11.6 defer to.
28. The `../technology/ai_llm/` agent cluster — [agent_harness_engineering_guide.md](../technology/ai_llm/agent_harness_engineering_guide.md), [agentic_engineering_guide.md](../technology/ai_llm/agentic_engineering_guide.md), [agentops_guide.md](../technology/ai_llm/agentops_guide.md), [llm_agents_failures_production_guide.md](../technology/ai_llm/llm_agents_failures_production_guide.md), [prompt_injection_guide.md](../technology/ai_llm/prompt_injection_guide.md), [ai_governance_framework_guide.md](../technology/ai_llm/ai_governance_framework_guide.md), [coding_agents_research.md](../technology/ai_llm/coding_agents_research.md) — cross-referenced from §6.3, §9.2, §11.1, §11.3, §11.6 rather than re-derived.

**Sourcing note for future readers.** The platform history in §2 will age fastest: check whether ElixirDevs and the surviving niche boards are still live, and re-check `hired.com` — the redirect to LHH is the current state of a brand that existed as a marketplace for twelve years. The regulatory numbers in §10 should be re-read at MOM directly, because the EP and S Pass thresholds move on a published schedule (1 January 2027 for the values quoted here). The labour-economics position in §8.2 will *not* age — it is a twenty-year-old study that nobody has replaced, and until someone does, that absence is the finding.

---

**Last word.** The reverse job search is not a strategy with a proven effect size; it is a low-cost, high-optionality *publication* whose entire upside depends on being findable by someone whose query you cannot predict, and whose entire risk is being equally findable by someone whose intentions you would not have chosen. Publish the niche, publish the artefacts, publish the availability, withhold the identifiers, and run the inbound like an operations queue. Then accept that no one — not the platforms, not the boards, not the forty years of recycled statistics — can tell you what it is worth, because nobody has ever measured it. What can be measured is simpler and more useful: whether a stranger who needs exactly what you do, and has never heard your name, can find you before they stop looking. Everything in this guide exists to make the answer yes, and to make the noise, the exposure, and the fraud that come with it survivable — because the alternative is the search that never has to be made, the published candidate.
