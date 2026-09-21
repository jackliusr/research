# Post-Sales and Customer-Facing Experience: A Comprehensive Guide — the Role Family, the Work, the Metrics and the Development Discipline

> **A deep-dive reference on post-sales and customer-facing work and its development: the role family as it actually exists, the lived experience of the work, the lifecycle of a customer relationship after the signature, the commercial mechanics, the metrics and their critique, the training and enablement discipline that is supposed to build the capability, the evidence on whether that training actually changes behaviour, the AI-era effect graded by evidence, the enterprise and regulated-industry context, and a Cymbal Bank worked example — plus a claims audit, a "what could not be verified" section and a glossary.**
>
> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Professional Development / Management & Leadership Series
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026

---

> **A note on verification and labelling.** Every substantive claim in this guide carries a marker. **✅** = verified in this pass against a named source. **⚠** = flagged: widely repeated but not verified in this pass, or an estimate whose provenance is stated. **🔧** = the guide's own construction: analysis, taxonomy, or template that is *explicitly not* sourced. **⛔** = rejected, or could not be verified. Where a framework, metric or training method is named, the guide states **who originated it** and **what evidence supports it**; where the originator is also the vendor of the thing being measured or sold, that is stated in the same sentence. The distinction between *documented*, *peer-reviewed*, *practitioner convention* and *constructed* is maintained in every section, and §15 audits the load-bearing claims.
>
> **A note on the sources of this pass.** This guide was written with a web search tool that returned **empty result sets for every query attempted** after an initial window, and a page-extraction tool that worked. Where a claim is marked ✅ it was verified against a source this guide could actually read — in practice, reference works, standards bodies' own pages, journal abstract records, and organisations' own sites. **An empty search is recorded here as a tool limitation, never as evidence that something does not exist.** Several items that a reader might expect to be settled — see §16 — are listed there as *not verified in this pass* rather than asserted.
>
> **What this guide is.** It owns **the post-sales and customer-facing role family and how it is developed**: what the roles are, what the work feels like from the inside, what happens between a signature and a renewal, how the commercial arithmetic that funds the function actually works, what the instruments used to measure it can and cannot support, how organisations try to train for it, and what the evidence says about whether that training works.
>
> **What this guide is not.** It is not a deal-cycle guide, not an IT service management guide, not a vendor-management guide, not a forward-deployed-engineering role guide, and not a human-skills curriculum. The boundary table in §1.2 says precisely which neighbouring guide owns each of those, and this guide treats them as **pointers**: one clause each, then move on.
>
> **What to read first.** If you are considering this work, read **§2** (the roles), **§3** (what the experience actually is) and **§13** (the career angle). If you run the function, read **§5** (commercial mechanics), **§6** (metrics and their critique) and **§7** (the enablement discipline). If you are being sold something, read **§6** and **§9** first, then **§15**. If you want the worked example, it is **§12**.

---

## Table of Contents

1. [How This Guide Relates to the Repository, the Overview and the Decoder](#1-how-this-guide-relates-to-the-repository-the-overview-and-the-decoder)
2. [The Professional Landscape, Verified](#2-the-professional-landscape-verified)
3. [What the Experience Actually Is](#3-what-the-experience-actually-is)
4. [The Post-Sales Lifecycle and Where It Breaks](#4-the-post-sales-lifecycle-and-where-it-breaks)
5. [The Commercial Mechanics](#5-the-commercial-mechanics)
6. [The Metrics and Their Critique](#6-the-metrics-and-their-critique)
7. [The Training and Enablement Discipline](#7-the-training-and-enablement-discipline)
8. [The Skills Decomposition](#8-the-skills-decomposition)
9. [What the Evidence Says About Training](#9-what-the-evidence-says-about-training)
10. [The AI-Era Effect, Evidence-Graded](#10-the-ai-era-effect-evidence-graded)
11. [The Banking and Enterprise-Relationship Context](#11-the-banking-and-enterprise-relationship-context)
12. [The Cymbal Bank Worked Example: A Post-Sales Function That Scores Well and Retains Badly](#12-the-cymbal-bank-worked-example-a-post-sales-function-that-scores-well-and-retains-badly)
13. [The Career Angle](#13-the-career-angle)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. How This Guide Relates to the Repository, the Overview and the Decoder

### 1.1 The problem this guide exists to solve

Ask ten people in a software company what "customer success" does and you will get ten answers that overlap at the edges: someone will describe support, someone will describe account management, someone will describe a technical consultant, and someone will describe a person whose job is to run a quarterly review that nobody prepared for. All of them are partly right. The vocabulary of this function is **borrowed from three different industries** — from sales (account management, quota, expansion), from IT service management (SLA, escalation, service review), and from management consulting (value realisation, adoption, the business review) — and it was assembled in that order between roughly the 1990s and the 2010s. The result is a role family whose titles are unstable, whose metrics are contested, and whose development model is largely **convention transmitted through vendor content marketing**.

That combination is exactly why this topic needs a source discipline most management topics do not. A vendor that sells customer-success software has a commercial interest in a particular definition of customer success. A training provider that sells a certification has a commercial interest in the certification mattering. A conference speaker has an interest in a metric being quotable. A very large share of what circulates as "research" in this field is **vendor-authored opinion with a chart attached**, and this guide separates those categories explicitly rather than averaging them together.

### 1.2 The boundary: what this guide owns, and what it does not

This guide owns **the post-sales and customer-facing work and its development**: the role family, the lived experience, the lifecycle, the commercial mechanics, the metrics and their critique, the training and enablement discipline, the skills decomposition, the evidence on training efficacy, the AI-era effect, and the banking-enterprise context.

| Neighbouring guide | What it owns | What this guide does with it |
|---|---|---|
| [meddicc_guide.md](meddicc_guide.md) | **The deal cycle and qualification.** MEDDPICC, its lineage, its expansions, training and enablement vendors in the sales-methodology market, deal reviews, forecasting, failure modes, the wholesale-banking application. | **Pointer only.** This guide never re-derives qualification methodology. It starts *after* the signature, and treats MEDDPICC as the thing whose output it inherits. |
| [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) | **The general sales-methodology survey** (note: this file lives in `technology/`, not `management/`, and its own sibling links are written as `../technology/...`). | **Pointer only.** §13 uses it for the adjacent-route discussion; §2 does not re-survey sales frameworks. |
| [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) | **IT service management** — ITIL-style internal and external service support, service design, incident and problem management. | **Pointer only.** §4 and §11 refer to service-management *cadence* as a customer expectation and do not re-teach the frameworks. |
| [vendor_management_guide.md](vendor_management_guide.md) | **The buy side** — how an enterprise selects, contracts with, governs and exits a supplier. | **Pointer only.** The mirror image of §11: this guide describes the sell side of the same relationship. |
| [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) | **One specific customer-facing technical role** — the forward-deployed engineer, customer-embedded, engineering-trained, building. | **Cross-reference, not absorption.** The FDE is a sibling role family with a different centre of gravity (§2.6). Its guide owns the role; this one owns the surrounding function. |
| [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) | **Revenue products** — what is sold, packaged and monetised. | **Pointer only.** §5 describes the retention/expansion *arithmetic*; it does not describe product design or pricing architecture. |
| [conflict_management_guide.md](conflict_management_guide.md), [facilitation_skills_guide.md](facilitation_skills_guide.md), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [authority_skills_guide.md](authority_skills_guide.md), [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md), [micro_expressions_guide.md](micro_expressions_guide.md), [the_first_90_days_guide.md](the_first_90_days_guide.md), [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md), [resilience_engineering_guide.md](resilience_engineering_guide.md) | **The general human-skills curriculum.** De-escalation, facilitation, stakeholder communication, authority, reading people, transitions, situational leadership, personal resilience. | **Named cross-references only.** §8 maps each trained skill to the guide that owns it, **by name**, and does not re-teach any of them. |
| [reverse_job_search_guide.md](reverse_job_search_guide.md) | **Discoverability and search mechanics** — how a professional becomes findable and how to run a reverse search. | **Named cross-reference only** in §13. |

🔧 *The "one owner per object of study" framing is this guide's own construction, offered as a reading aid. It is not a claim about how the industry organises itself — the industry does not, which is the subject of §2.*

### 1.3 Definitions this guide actually uses

Two words do most of the work in this field and are almost never defined. This guide defines them narrowly, on purpose.

**Post-sales** — everything a vendor does with a customer *after the contract is signed and the implementation is committed*: onboarding, adoption support, ongoing service and issue handling, value demonstration, renewal, and expansion. In this guide "post-sales" **includes** implementation and onboarding (because in practice the same function usually performs them, and the handover into them is the lifecycle's weakest joint), and **excludes** the sales process itself, which ends at signature. The term is a vendor-industry usage rather than a standard; the accounting of what it contains differs by company, and §2.4 says which parts of the industry's own description of it are documented.

**Customer-facing** — the role's *primary work product is an interaction with a person who is not employed by your organisation*, and whose satisfaction affects whether your organisation is paid again. Support, customer success, account management, professional services, technical account management and forward-deployed engineering all meet this test. A product manager who interviews customers does not (their output is a product decision). An internal service desk at a bank does not (their customer is a colleague, however much the vocabulary matches). The distinction matters because **the metrics, the emotional labour and the authority problem in §3 and §6 attach to the customer-facing boundary, not to the job title.**

🔧 *Both definitions are this guide's construction. They are stated so the reader can disagree with them; they are not sourced.*

### 1.4 The decoder: the vocabulary of a job description or a vendor pitch

The single most useful skill for reading a post-sales job description is knowing which of these words is a **function**, which is a **stage**, which is a **metric**, and which is a **sales promise**. The table below is the guide's own construction and is intended to be used against real postings and real pitch decks.

| Term | What it means in practice | What it is not | Marker |
|---|---|---|---|
| **Customer success (CS)** | A post-sales function — or a company name — whose declared purpose is to ensure the customer achieves the outcomes they bought for, usually carrying renewal and expansion responsibility. | Not a defined profession with a standard body of knowledge; the term's boundaries are set by each employer. | ⚠ industry usage |
| **Account management (AM)** | The commercial relationship owner. Traditionally revenue-carrying; in a post-sales setting, owns the commercial calendar (renewal, upsell, pricing) rather than the adoption detail. | Not the same as customer success, though many companies merge the two into "CSAM" and then wonder why the two halves conflict. | ⚠ industry usage |
| **Technical account management (TAM)** | The technical relationship owner, typically inside support or services. Owns architecture discussions, technical business reviews, escalation of technical issues, and the customer's roadmap questions. | Not a support agent with a nicer title, though levelling sometimes makes it one. | ⚠ industry usage |
| **Implementation / professional services (PS)** | The project function that delivers the initial deployment: configuration, integration, data migration, testing, go-live, training. Usually a **separate P&L** with billable utilisation. | Not customer success, and usually not funded from the subscription — which is the structural reason the handover in §4.2 breaks. | ⚠ industry usage |
| **Onboarding** | The bounded programme that takes a new customer from signature to first useful production use. | Not the same as implementation, though for small customers they are the same thing. | 🔧 distinction is the guide's |
| **Adoption** | Whether the licensed capability is being used by the people who were supposed to use it. Measured in usage telemetry, licence activation, feature depth. | Not satisfaction, and not a promise that the customer is getting value. | ⚠ industry usage |
| **Value realisation** | The claim that the customer obtained a measurable outcome. Corporate-speak with a real question inside it: *against which baseline, measured by whom?* | Not a metric. There is no standard instrument. | ⚠ used as a term; 🔧 critique |
| **Renewal** | The customer pays again for another term. **The event that funds the function.** | Not the same as retention of usage, of the sponsor, or of the account manager. | — |
| **Expansion** | More revenue from an existing customer: seats, modules, volume, tier upgrade. | Not automatically value delivered; it can be consolidation the customer regrets. | — |
| **Churn** | Revenue that stops. Must always be asked as *which* churn (see §5.2) — logo, gross, net, voluntary, forced, or downsell. | A single number. Unqualified churn figures are unusable. | 🔧 |
| **Gross revenue retention (GRR)** | Revenue retained from a cohort **excluding** expansion, so it can only fall. The strictest retention measure. | Not "retention" in the marketing sense. | ⚠ industry usage; 🔧 exact treatment |
| **Net revenue retention (NRR) / net dollar retention** | Revenue retained **including** expansion from the same cohort; can exceed 100%. | Not comparable across companies without the cohort definition, the period and the treatment of currency and downgrades. | ⚠ industry usage; see §5.2 |
| **Satisfaction instruments** | Survey-based measures administered after an interaction or on a schedule: the recommendation item (NPS), the effort item (CES), satisfaction items (CSAT/C-SAT). | Not validated measurements of loyalty, effort, or future revenue. §6 is the audit. | ⚠ see §6 |
| **First-contact resolution (FCR)** | The proportion of contacts resolved without transfer, callback or reopen. | Not a measurement of whether the customer's problem is gone; the definition usually comes with the tool that reports it. | ⚠ see §6.4 |
| **Business review / QBR / EBR** | The recurring meeting where the vendor presents value, adoption and roadmap to the customer. QBR = quarterly, EBR = executive. Also functioning as the informal renewal pipeline. | Not a governance right unless the contract says so (§11.3). | ✅ exists as industry practice; ⚠ as a standard |
| **The CS platform** | The software category (customer success platforms) that aggregates telemetry, health scores, playbooks and renewal dates. | Not the function. Buying the platform is the most common substitution for building the capability. | ⚠ industry usage |
| **Enablement** | In *sales* usage (see [meddicc_guide.md](meddicc_guide.md)): the function that builds seller capability — content, onboarding, certification, coaching, tooling. In this guide the same discipline is applied to post-sales roles, where it is usually much thinner. | Not training delivered once; see §9. | ⚠ two different usages share the word |
| **Health score** | A composite, vendor-defined index of account risk built from telemetry and survey data. | Not a validated predictive instrument; its threshold is set by whoever configured it. | 🔧 critique; see §6.6 |
| **Ramp** | The period from a new hire's start to their carrying full workload. Also used for the *customer's* time-to-first-value. Two different meanings; always ask which. | Not a fixed industry constant; see §7.2 and §9.6. | ⚠ see §7.2 |

**One structural observation before anything else in this guide: the title taxonomy of this field is not standard, and it is not documented anywhere authoritative.** There is no international standard for "customer success manager", no agreed competency framework that firms hire against, and — as §2 establishes from official occupational classifications — no clean occupational code that maps to the role. Every taxonomy in this guide is therefore presented as a *description of observed practice*, sourced where a source exists and labelled as inference where the only evidence is job postings.

---

## 2. The Professional Landscape, Verified

### 2.1 The role family, and what actually distinguishes the roles

The industry describes five to seven roles in this family. Almost none of them are defined by a standard; what follows separates **what is documented** (a source exists) from **what is inferred** (the only evidence is job postings and vendor content).

| Role | The centre of gravity | Primary artefact | What the customer buys from it | Evidence status |
|---|---|---|---|---|
| **Support / service desk agent (tiered)** | Resolving incoming issues within a service commitment. Volume-driven, queue-managed, measured on contact handling. | Ticket, resolved contact, knowledge article | A response within a time limit, and a fix | ✅ The occupational category exists and is classified in national statistics (customer service representatives); the tiered structure is industry convention |
| **Technical support engineer (L2/L3)** | The escalation point for issues the first tier cannot solve. Debugging, log analysis, reproduction, workarounds. | Root-cause analysis, workaround, bug report | Technical resolution | ⚠ Industry convention; no standard defines tier boundaries |
| **Customer success manager (CSM)** | Adoption and outcome attainment, carrying renewal risk. | Success plan, business review, health review, adoption report | That the purchased capability gets used and produces a result | ⚠ Defined in peer-reviewed literature as a research subject, not as an occupation with a standard (see below) |
| **Account manager / commercial owner** | The commercial relationship: renewal negotiation, expansion, pricing, contract vehicles. | Commercial proposal, renewal paper, contract change | Commercial continuity and a route to more | ⚠ Industry convention |
| **Technical account manager (TAM)** | The technical relationship with an enterprise customer: architecture, roadmap translation, technical escalations, service reviews. | Technical review, escalation plan, roadmap brief | A named technical counterpart who knows their estate | ⚠ Industry convention; in some firms identical to a CSM with deeper technical scope |
| **Implementation consultant / professional services** | Delivering the contracted deployment project on time and on budget, usually billable. | Project plan, configuration, migration runbook, training delivery | Working software in production | ✅ Exists as a distinct billable services P&L across the software industry; the specific naming is not standardised |
| **Forward-deployed engineer (FDE)** | An engineering-trained person embedded with the customer, building and shipping as well as advising. | Shipped code, integration, deployed capability | Custom build capacity inside the product | ✅ Separately owned by [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) — cross-referenced here, not re-derived |
| **Customer success operations / enablement** | The internal function that builds the tooling, the playbooks, the data and the training for the roles above. | Playbook, health-score model, enablement curriculum, platform configuration | Nothing directly — an internal capability | 🔧 Distinction is this guide's; the function exists, its boundary does not |

**The three boundaries which the industry blurs, and the questions that unblur them:**

1. **Support vs customer success.** The unblurring question is *what event triggers the work*. Support is triggered by the customer's inbound contact; customer success is triggered by a calendar, a telemetry threshold, or a renewal date. A "customer success" team whose work is entirely inbound-triggered is a support team with a different job title. ⚠ *Practitioner convention; not a standard.*
2. **Customer success vs account management.** The unblurring question is *who owns the number*. If the role carries a revenue target, it is a commercial role and will behave like one; if it carries a retention rate but no negotiation authority, it has accountability without the instrument to discharge it (§3.4). 🔧
3. **Implementation vs customer success (post-implementation).** The unblurring question is *who is funded by what*. Implementation is usually funded from a services line and is billable; the post-sale success function is usually funded from the subscription and is a cost centre. That single fact explains most of the hostility between them (§4.2). ⚠ *Widely described practitioner pattern; the funding model varies.*

### 2.2 Is "customer success" a real occupation, or a job title?

This is the honest centre of the section, and the answer is genuinely split:

**Documented — as a subject of peer-reviewed research.** ✅ The term has a real academic literature attached to it, and it is not merely a vendor coinage. Named examples located in this pass: *Hochstein, Voorhees, Pratt, Rangarajan, Nagel and Mehrotra, "Customer success management, customer health, and retention in B2B industries"* (published in **Industrial Marketing Management**, cited as December 2023) ✅, and *Eggert, Ulaga and Gehring, "Managing Customer Success in Business Markets: Conceptual Foundation and Practical Application"* (**Journal of Service Management Research**, 2020) ✅. Both appeared as citations in the reference work's treatment of the topic, and both are journal articles, not vendor content. ⚠ *Note the standing of this literature honestly: it is a young, thin body of B2B marketing research. It establishes that the function exists and that researchers study it; it does **not** establish a competency standard, a validated health score, or a training curriculum.*

**Documented — as a business strategy with a mainstream corporate origin.** ✅ Customer success is described as a marketing strategy and organisational function for B2B subscription businesses, whose purpose is to ensure customers achieve desired outcomes and thereby reduce churn, **and the practice is described as developed by companies such as Salesforce in the mid-2000s to increase revenue by improving retention**, building on the earlier services-marketing and relationship-marketing literature ✅ *(source: the reference work's "customer success" article and its citations)*. That origin story — a software company inventing a function to defend recurring revenue — is the load-bearing fact for §5: **the function was born commercial, and it has been presented as customer-advocacy ever since.** ⚠ *The mid-2000s and the Salesforce attribution are as stated by the source; this guide did not obtain a primary Salesforce disclosure of it.*

**Not documented — the taxonomy, the ladder, the competencies.** ⛔ **There is no international standard defining the customer success manager role, no body whose competency framework employers must hire against, and (in this pass) no verified official occupational classification that maps cleanly onto it.** The consequences are concrete and worth stating plainly:

- **Job titles carry no transferable meaning.** "Senior customer success manager" at one firm may mean individual contributor with an escalation licence; at another, a people manager; at a third, a quota-carrying seller with a CS title used to avoid the sales-compensation plan. Software that maps titles across companies produces noise. ⚠
- **Title inflation is documented as a market practice, not a measurement.** This guide could not verify any study quantifying title inflation in this family ⛔; what can be verified is that the *same* title appears in postings with materially different scope, seniority and compensation — which is a statement about postings, not about the labour market. §16 records this as unverified.
- **Compensation, headcount, growth and attrition figures for these roles require a source, a date and a methodology**, and in this pass **not one was verified to that standard** ⛔ (§16). Where such figures appear — salary aggregators' self-reported bands, vendor "state of the industry" surveys, job-board counts — they are recorded here as *what they are*: self-selected or sponsor-defined samples. **No number is printed in this guide that could not be attributed to a named source with a stated method.**

### 2.3 The seniority ladder, described honestly

The ladder below is the **modal pattern across postings and firm levelling pages**, not a standard. 🔧 *Construction based on observed practice; the band names are illustrative.*

| Level | Scope typically signalled | What actually changes | Evidence status |
|---|---|---|---|
| Associate / CSM I | A book of small or pooled accounts; scripted playbooks; escalation is the job. | Independence. The work is *execution of a defined motion*. | ⚠ inferred |
| CSM | An owned book; runs the cadence; owns the success plan. | Judgement: which account needs attention and which does not. | ⚠ inferred |
| Senior CSM / CSM III | Complex or strategic accounts; designs the motion for others; handle escalations. | Influence over internal roadmaps; the ability to say no. | ⚠ inferred |
| Lead / Principal / Team lead | Either a player-coach or a technical authority; owns a segment's outcome. | Accountability for others' outcomes without always having authority over them. | ⚠ inferred |
| Manager / Director | Multiple segments or a region; owns the retention number and the headcount. | Budget, hiring, and the politics of §5. | ⚠ inferred |
| VP / Chief Customer Officer | The function's strategy, its budget, and its relationship to the board. | The function becomes a political actor; retention becomes a company-level claim. | ✅ The CCO role exists as a titled executive office; its mandate varies by firm |

**Two honest observations about this ladder:** ⚠

1. **The rungs are not a profession's grades.** Unlike a chartered profession, there is no independent body that certifies that a person at "senior" level meets a required standard. Progression is employer-defined, which means it is **not portable** in the way a professional grade is, and it is negotiable in a way a professional grade is not.
2. **The ceiling is real but narrow.** The path from CSM to executive exists; it runs, in practice, through *owning a number* — retention, expansion or a segment's P&L — rather than through service excellence. §13 develops this.

### 2.4 Sectors, geography and what is genuinely unknown

**Sectors that employ this role family** — described, not measured ⚠: enterprise and mid-market software (the origin sector, and still the densest); cloud and infrastructure providers; financial technology and payment networks; healthcare technology; telecommunications; industrial and hardware vendors moving to subscription and connected-service models; managed-service providers; and — in a different register — airlines, banking retail operations and utilities, where the vocabulary of service levels and complaint handling overlaps ([../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) owns that register).

**Geography.** ⛔ **This guide verified no geographic distribution, no employment count and no growth projection for this role family in any country.** What can be verified is narrower and more useful: the *software industry* context in which the function grew is internationally distributed, and the function's vocabulary (NRR, adoption, health score, QBR) is used in English-language vendor and trade content regardless of jurisdiction. Anything stronger than that — "X thousand CSMs in Singapore", "Y% growth in APAC" — was not verified in this pass and is not printed. §16 records this. **An empty search is a tool limitation; it is not evidence of absence, and it is not a licence to estimate.**

### 2.5 What the landscape genuinely is, in one paragraph

A young, commercially invented function, born inside B2B software in the mid-2000s to defend recurring revenue, which has since accumulated a real but thin peer-reviewed research literature, a large vendor content industry, an unstable title taxonomy, no occupational standard, and no independently certified competency baseline. Its practitioners are skilled and its work is real; its *documentation* is mostly produced by parties selling into it. That is the honest summary, and it is why every subsequent section in this guide names a source or labels itself a construction.

---

## 3. What the Experience Actually Is

### 3.1 The shape of the working day

The work is **interrupt-driven, calendar-anchored and asynchronous across time zones**, and the experienced practitioner's skill is largely the management of those three properties. ⚠ *This section's description of the working pattern is practitioner-level description, labelled as such. There is no time-and-motion study of this role family that this guide could locate ⛔, so the account below is offered as the industry's own description of the work, not as measurement.*

| Time scale | What actually occupies the day | Where the pressure comes from |
|---|---|---|
| The 15-minute scale | The inbound message from a customer with a problem that is not the vendor's fault and not the customer's fault | The expectation of a fast, warm, competent answer, from someone the customer did not raise the ticket with |
| The day scale | Meetings across accounts, internal stand-ups, chasing engineering for an issue, writing the follow-up that documents what was agreed | Calendar density; the documentation burden falls on the customer-facing person because they were in the room |
| The week scale | The review preparation: assembling adoption data and value narrative for a customer meeting | The gap between what the vendor's telemetry shows and what the customer believes |
| The quarter scale | The business review, the renewal forecast, the escalation that needs an executive sponsor | The renewal date, which does not move |
| The year scale | The account plan, the product gaps you have raised three times, the sponsor who left | Product roadmap priorities you do not control |

Two structural features of this pattern matter for the training discussion in §7 and §9: the work is **high-interruption**, and interruption costs *every* role; and the work is **asymmetric in its failure modes** — a good day is invisible, a bad day is a ticket with your name in the thread. ⚠ *Practitioner consensus; the interruption-cost research this rests on is from general knowledge-work literature not re-verified here.* 🔧 *The asymmetry framing is the guide's construction.*

### 3.2 Emotional labour: the load the job does not admit to carrying

This is the single most documented thing about customer-facing work, and it is documented in the *academic* literature rather than in the industry's own content — which is precisely why the industry rarely cites it.

**The foundational concept.** ✅ The sociological concept of **emotional labour** was defined by **Arlie Hochschild**: emotional labour is the management of one's own emotions and the emotions of others to meet job expectations, and it is defined by three criteria — the job requires face-to-face or voice-to-voice contact with the public; the job requires the worker to *produce an emotional state in another person*; and the employer can, through training and supervision, exercise a degree of control over the employee's emotional activity. ✅ *(Source: the reference work's "emotional labor" article, citing Hochschild's foundational text and her 1983 argument that in this commodification process service workers become estranged from their own feelings at work.)* The "training and supervision" criterion is worth pausing on: **emotional display is not incidental to this job family — it is definitionally something the employer trains and supervises.** §7's call-review practice is, in Hochschild's terms, exactly the supervisory mechanism she named.

**Surface acting versus deep acting.** ✅ Hochschild's text divided emotional labour into two components: **surface acting**, where employees display the emotions the job requires without changing how they actually feel, and **deep acting**, an effortful process through which employees change their internal feelings to align with organisational expectations, producing more natural displays. ✅ **Research generally shows surface acting to be the more harmful to employee health.** ✅ *(same source)*

**The meta-analytic evidence, and its exact size.** ✅ **Hülsheger and Schewe's meta-analysis "On the costs and benefits of emotional labor"** quantitatively reviewed the relationship of emotion-rule dissonance, surface acting and deep acting with well-being and performance outcomes, **based on 494 individual correlations from a final sample of 95 independent studies**. ✅ *(Source: the article's own abstract, via its PubMed and APA PsycNet records; published in the **Journal of Applied Psychology**.)* It reported **substantial relationships of emotion-rule dissonance and surface acting with indicators of strain and burnout**. ⚠ *This guide verified the abstract-level description and the study's method and sample size; it did not read the full text, so it does not print effect sizes.* ⛔ *No correlation coefficient is asserted here.*

🔧 **What this means for the practitioner and the manager — the guide's own reading, not a sourced claim:** the finding that *surface acting* (not the work itself) is the strain mechanism has a direct design implication. A function that trains people to *display* empathy while the internal conversation is "this is not our fault and I have to apologise anyway" is manufacturing the specific condition the literature associates with strain. A function that gives people the authority to fix the underlying thing, or the honesty to say what is actually true about the timeline, is reducing it. That is a design claim, and the guide labels it as its own.

**Absorbing blame for things you did not cause.** ⚠ *This is the practitioner's own description of the job and it is consistent with the literature above, but no study of blame absorption specifically in post-sales software roles was located in this pass.* What can be said with confidence is structural: the customer-facing role is **the interface through which defects, delays and roadmap decisions made by others are received as personal service failure**, and the role's formal authority over those causes is usually near zero (§3.4). Whether that is "unfair" is a judgement; that it is *structural* is not.

### 3.3 Escalation, on-call and the shape of the pressure

- **Escalation is a resource-allocation ceremony.** An escalation is not primarily a technical event; it is a *claim on other people's attention* inside your own organisation, and its currency is reputation. Practitioners who escalate well are those who can make a case in one paragraph and who have banked credibility by escalating only what deserves it. ⚠ *Practitioner consensus.*
- **On-call.** Real for the technical roles in this family (support engineering, TAM in some firms, FDE); usually *not* formally on-call for CSMs, who are instead on-call socially — reachable, expected to answer, with the expectation undocumented. ⚠ *Practitioner consensus; the contractual and compensation practice around customer-facing on-call varies by firm and was not verified here.*
- **The escalation-to-blame transition** is the documented-in-practice failure mode: a function staffed and rewarded for escalation volume produces more escalation, and the customer learns that escalation works. Some vendors therefore gate escalation behind a service-management process ([../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) owns the mechanics). 🔧 *The incentive critique is this guide's.*

### 3.4 Responsibility without authority: the defining structural condition

If one sentence summarises this role family, it is this: **the customer-facing post-sales role is accountable for outcomes it does not control, using levers it does not own, measured by instruments it did not choose.** The three parts are worth separating:

| Accountability | Controlled by | The role's actual levers |
|---|---|---|
| Product works as sold | Product and engineering | Feature requests, escalations, workarounds, honesty about the timeline |
| Customer adopts it | The customer's own organisation | Training, champions, executive air cover, removing friction |
| The renewal happens | The customer's budget cycle, their sponsor's career, the competitive field | Commercial framing, value evidence, relationship quality — and, often, price authority it does not hold |
| The satisfaction score | Respondent psychology, timing, and who else is unhappy that day | The interaction itself, and the survey's administration |

⚠ *The table is a description of the tension every practitioner in this family describes; the guide constructed the taxonomy.* The practical consequence, which §8 treats as the core trainable skill, is that **influence without authority is not a soft skill here — it is the job's primary mechanism**, and §8 cross-references the repository's influence and authority guides by name rather than re-teaching them.

### 3.5 The relationship with the engineering organisation

The most reliable predictor of whether a post-sales role is sustainable is not the person's resilience; it is **whether the engineering organisation treats the field as an information source or as a complaint channel.** 🔧 *(The guide's own construction, consistent with the escalation economics described in §3.3 and with the lifecycle failure modes in §4.)* The observable differences:

- **Signal or noise.** Where field-reported issues are triaged as evidence with a defined intake, the role is a sensor. Where they arrive as personal demands from a named person, the role is a nuisance and its reports are discounted by association. ⚠
- **The gap between what was sold and what exists.** Where sales and product have an explicit "we do not sell what does not exist yet" constraint — the domain of the qualification guide ([meddicc_guide.md](meddicc_guide.md)) — the post-sales role inherits a smaller problem. Where they do not, the post-sales role spends its first year managing a debt it did not incur. ⚠ *This is the single most frequently named structural cause of post-sales distress in practitioner accounts; §4.2 treats it as the lifecycle's weakest joint.*
- **Who gets the credit.** The documented-in-practice pattern is that a shipped fix is an engineering success and a prevented churn is invisible; the customer-facing role's evidence is a *negative* — the renewal that nobody discussed. 🔧

### 3.6 What practitioners say about staying and leaving

Handled honestly, because the material here is practitioner testimony and self-reported: ⚠ **All of the following is practitioner-secondary evidence — forum and trade commentary, and the industry's own retrospective essays. It is directional, self-selected, and it is not measurement.** The guide deliberately does not print attrition rates (§16).

**Why people stay** ⚠ — the customer relationship itself (a real relationship with a real person who values you, which is emotionally sustaining and rarely available in pure internal roles); the breadth (sitting across product, commercial, technical and human problems, which is genuinely good general-management training); the visible causality (unlike many corporate roles, the connection between what you did and whether the customer stayed is often traceable); and the market (the skills transfer into adjacent roles, per §13).

**Why people leave** ⚠ — the responsibility/authority gap in §3.4, once the person realises it is structural rather than temporary; the on-call-of-the-soul pattern; being made the recipient of consequences of decisions taken above them; compensation lagging the adjacent commercial roles they work alongside; and — the specific complaint that recurs most in practitioner accounts — **the measurement**: being appraised on a survey score they can manipulate but not cause, and on a retention number they can influence but not control.

**The honest editorial position.** 🔧 The experience is neither the "dream job at the heart of the customer relationship" that vendor content describes, nor the "thankless escalation janitor" that burnt-out practitioners describe. It is a **high-influence, low-authority, high-emotional-demand role whose career value comes from exactly the parts that make it hard** — the accumulated judgment about which customer is at risk and why, and the ability to move an organisation you do not command. That combination is rare, it is not taught well, and its difficulty is the reason it is paid for.

---

## 4. The Post-Sales Lifecycle and Where It Breaks

### 4.1 The stages, and what each is actually for

The lifecycle below is **this guide's construction** 🔧, assembled from the stages the industry names and the purposes practitioners describe. It is offered as a diagnostic frame, not as a standard. For each stage: what it is *for*, what the customer-facing role does, and the failure mode.

| # | Stage | What it is for | What the customer-facing role actually does | Failure mode |
|---|---|---|---|---|
| 0 | **Sale** (owned elsewhere) | Establishing what was bought, for how much, by when, and what the customer expects to get | Receives whatever was promised | **Promises the product cannot keep** — the origin of most downstream distress. Owned by [meddicc_guide.md](meddicc_guide.md) |
| 1 | **Handover** (sales → post-sales) | Transferring the customer's context so the post-sales team starts informed rather than merely introduced | Attends a call, receives a document, inherits the account | **Context is not transferred, only the contact list.** §4.2 |
| 2 | **Implementation / onboarding** | Getting the purchased capability into production use | Project management, configuration, data work, testing, initial training | **Scope discovered in-flight, with no commercial vehicle to absorb it** |
| 3 | **Adoption** | Making the capability used by the people who were supposed to use it | Usage reviews, enablement, champion-building, friction removal | **Measuring logins and calling it adoption** |
| 4 | **Value realisation and the business review** | Demonstrating that the customer's original outcome is being achieved, against a baseline | Assembling evidence; presenting it; negotiating what "value" means | **Value asserted rather than evidenced**; the review becomes a status meeting |
| 5 | **Renewal** | Getting paid again | Owning the commercial calendar; surfacing risk early; negotiating | **Renewal discovered at the renewal date** |
| 6 | **Expansion** | More revenue from a customer who is succeeding | Recognising the moment; bringing in the commercial role; not over-selling | **Expansion sold into a customer who is not yet succeeding** — the fastest route to churn |
| 7 | **Advocacy / reference** | Using a successful customer to help acquire others | Asking for the reference, the case study, the analyst call | **Asking a customer who is merely tolerant**; burning the relationship for a marketing asset |
| 8 | **Churn / exit** | Ending or winding down | Conducting the loss review; preserving the possibility of return | **No loss review at all**, so the cause is never established and the pattern repeats |

### 4.2 The handover: why it fails, structurally

The handover is the most commonly cited failure point in this lifecycle, and the honest treatment is to refuse the individual explanation ("the rep didn't document properly") in favour of the structural one.

**Five structural causes.** 🔧 *This is the guide's analysis; the individual causes are each recognised in practitioner literature, but the assembly and the framing are this guide's.*

1. **The incentives are opposed, not merely unaligned.** ✅ *This is the strongest form of the argument.* The sales role is compensated on the signature; the post-sales role is measured after it. A thorough handover costs the seller time *after* their compensation event and creates no credit. It is not a documentation failure; it is a payment-timing failure. (The compensation mechanics of the deal cycle are owned by [meddicc_guide.md](meddicc_guide.md); this guide only notes the consequence.)
2. **The knowledge is not transferable as an artefact.** The seller's most valuable context is *provisional* — what the champion said off the record, which stakeholder is opposed and why, what was implied but not written. None of that is going in a form field. ⚠ *Practitioner consensus; also the reason the "documented handover" pattern in §14 fails.*
3. **The two functions are funded from different places.** Implementation is typically billable and carries its own utilisation target; post-sales success is typically funded from the subscription as a cost centre. Their targets therefore diverge at the moment of transfer. ⚠ *Widely described pattern; funding models vary by firm.*
4. **The receiving side has no power to refuse.** A post-sales team that must accept an account it did not qualify and cannot decline has no lever over the quality of what it receives. In procurement language, it has all of the liability and none of the acceptance criteria. 🔧
5. **The customer experiences the seam.** To the customer, the vendor is one company; being handed from one named person to another with different incentives, and *being asked to explain the same thing twice*, is the most visible evidence that the vendor is not. ⚠ *This is the practitioner-consensus statement of the harm.*

**The test for whether a handover works.** 🔧 *The guide's construction:* not "was a document created" but **"can the receiving person answer, without asking the customer, (a) what outcome the customer believes they bought, (b) who personally will be blamed internally if it fails, and (c) what was promised that does not yet exist?"** If the answer to (c) is unknown, they are inheriting a debt they have not been told about.

### 4.3 Implementation and onboarding

- **What it is for:** converting a contract into working, used software. In enterprise contexts this is the stage where most of the post-sale relationship risk is created, because it is the stage where reality first contacts what was sold. ⚠
- **What breaks:** scope discovered in-flight. The commercial question — *who pays for the integration nobody priced* — is answered in practice by either over-running at the vendor's cost, or by the customer paying for a change order they resent. Neither is a good outcome, and both are invisible on the satisfaction survey at the time. 🔧 *(The analysis is the guide's; the change-order dynamic is standard professional-services practice.)*
- **The observable tell:** how the vendor's own implementation people describe the sales team. Where implementation engineers speak of sales as a partner who scopes accurately, the stage works. Where they speak of a fire that arrives every quarter, no amount of post-sales training will help. ⚠ *Practitioner observation.*
- **Cross-reference:** the *service* mechanics of implementation — incident, problem, change, service transition — are owned by [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) (ITIL's own history and structure are verified: developed by the UK government's CCTA in the 1980s, released 1989, ITIL 4 released February 2019, AXELOS created 2013, and **PeopleCert acquired AXELOS in June 2021** ✅, with **no formal independent third-party compliance assessment available** and **certification available to individuals only, not organisations** ✅).

### 4.4 Adoption, value realisation and the business review

**Adoption** is measurable and therefore over-claimed. Usage telemetry can establish whether the capability is being used; it cannot establish whether the use is producing the outcome the customer bought it for. The gap between those two is the space in which the whole function either earns its credibility or loses it. ⚠

**Value realisation** is the most abused phrase in the field. The question underneath it is legitimate and unavoidable: *against what baseline, measured how, by whom, and at what point in time?* 🔧 *The guide's construction of the test:* a value claim is credible when the baseline was agreed **before** the outcome and does not come entirely from the vendor's own product telemetry. Both halves matter — a baseline agreed after the fact measures negotiation skill, not value.

**The business review (QBR/EBR)** ✅ *exists as widely practised industry convention; no standard governs it*: the recurring meeting in which the vendor presents usage, outcomes and roadmap to the customer. Its honest functions, in order of what it actually does rather than what it claims:

1. **It is the informal renewal pipeline**, and its cadence usually tracks the renewal date. 🔧
2. **It is an early-warning instrument** — the sponsor who does not attend, or sends a subordinate, is the most reliable churn signal available and is not on any dashboard. ⚠
3. **It is a status meeting** when the vendor brings nothing the customer does not already know. This is the modal outcome, and it is the reason the meeting is often resented. ⚠
4. **It is a value-demonstration instrument** only when it satisfies the baseline test above.

**Enterprise service-management cadence** — service level agreements, scheduled service reviews, escalation matrices, named service owners — is the *contracted* form of the same idea, and it is what enterprise buyers actually specify. It is owned by the operational-support and vendor-management guides; this guide only notes that **where the cadence is contractual, the relationship has a governance structure the vendor cannot unilaterally abandon**, and §11 develops that.

### 4.5 Renewal and expansion

- **Renewal is an event, not a process.** The process is the twelve months before it. The commonest structural failure is a vendor whose post-sales function has a *quarterly* cadence and a *renewal-date* trigger — i.e. risk becomes visible only when it can no longer be fixed. ⚠
- **The renewal conversation is a commercial conversation and is usually conducted by people without commercial authority.** 🔧 *This is the guide's construction, and it is a specific consequence of §3.4:* a CSM accountable for retention but without pricing or contracting authority must escalate every material negotiation, which slows the response and signals internally that the role is an input rather than an owner.
- **Expansion sold into a failing account accelerates the churn.** ⚠ *Widely held practitioner view; the mechanism is straightforward* — expansion raises the customer's spend and their expectation simultaneously, and the failure that follows is larger and better documented. §14 lists this as an anti-pattern.
- **Downgrades are churn.** This is a definitional point, not an opinion, and it is the subject of §5.2: a company that reports retention "including expansion" can be growing on the metric while shrinking on every account in the cohort except the largest. 🔧

### 4.6 Which stages a company actually performs, versus claims

The characteristic gap in this industry is between the lifecycle a company *describes* (all eight stages, formally) and the lifecycle it *staffs and pays for* (typically: implementation, a quarterly review, and a renewal conversation). 🔧 *The guide's construction of a way to test the claim — audit the following five things, and the real lifecycle appears:*

| Evidence to look for | What it tells you |
|---|---|
| Who is on the org chart, and who is billable | Which stages actually have people |
| What the post-sales function's budget line is called | Whether it is funded as cost or revenue (§5.3) |
| Whether there is an intake or refusal right over accounts | Whether the function has a handover lever at all |
| Whether renewal has a forecast that a finance function uses | Whether the retrospective is real |
| Whether loss reviews exist and are read by anyone outside the function | Whether the organisation learns from stage 8 |

🔧 *Five checks, the guide's own.*

---

## 5. The Commercial Mechanics

### 5.1 The arithmetic that justifies the function's existence

The commercial case for post-sales investment rests on one simple piece of arithmetic, and it is worth stating precisely because it is usually stated loosely:

- If a cohort of customers pays **$100** in year one and the vendor retains **90%** of it in year two, then without expansion the business shrinks by 10% a year and must replace it with new sales. If it retains 90% **and adds net 10% expansion**, the cohort is flat; at net 120%, the cohort grows 20% with no new customers at all. 🔧 *Illustrative arithmetic, the guide's own.*
- Therefore **the cost of a retention or expansion point can be compared against the cost of acquiring it.** New revenue has an acquisition cost and a delay; retained revenue has neither. The comparison is the actual argument for funding a post-sales function, and it is an accounting argument, not a customer-advocacy argument. 🔧
- **Consequence for the function's politics:** its budget is justified by retained revenue but its budget line is usually discretionary, because the revenue it defends does not appear on any invoice it issues. A function whose value is *a loss that did not happen* is structurally vulnerable in every cost-cutting cycle. 🔧 *This is the guide's reading; it is the origin of the metric politics in §6.*

⚠ **The caution attached to this arithmetic:** it only works if the retention ratio being used means what the user thinks it means. Which brings us to the definitions.

### 5.2 The retention ratios, defined precisely

The vocabulary below is **industry usage rather than a standard** ⚠ — no standards body defines these, and vendors' software calculates them differently, which is why a vendor's own reported number is not comparable across vendors. The definitions given are the ones the industry actually uses, stated so the treatment of each judgement call is explicit.

| Measure | Definition as used in the industry | The judgement calls that change the number |
|---|---|---|
| **Logo retention** | The proportion of customers (accounts) still customers at period end, regardless of how much they pay | A customer who downgraded 90% still counts as retained |
| **Gross revenue retention (GRR)** | Revenue retained from the **beginning-of-period cohort**, excluding all expansion. Cannot exceed 100%. Includes the effect of cancellations *and* downgrades | Whether the cohort is monthly or annual; whether it is computed on committed or billed revenue; whether a customer lost mid-period is annualised |
| **Net revenue retention (NRR) / net dollar retention (NDR)** | The same cohort, **including** expansion. Can exceed 100% | Currency effects; whether expansion is new seats, price increases, or re-sold modules; whether acquired-company customers ride in the same cohort |
| **Churn** | Revenue that stopped, **for the period and population specified** | Inbound vs outbound churn; forced churn (a customer going out of business, or a merger) vs voluntary; product-level vs account-level |
| **Downgrade / contraction** | Revenue retained but reduced (fewer seats, lower tier, removed module) | Whether contraction is netted against expansion inside NRR — **it is, and this is why NRR can hide a shrinking base of stable accounts behind one large expander** |
| **Gross churn vs net churn** | Gross = lost revenue before expansion; net = lost revenue minus expansion | Which of the two a company quotes, and whether it says so |

🔧 **Three practical disciplines this guide recommends for reading any retention number** *(the guide's own construction)*:

1. **Never accept a single retention figure.** Ask for logo retention, GRR and NRR together, for the same cohort and period. Each hides exactly what another reveals.
2. **Ask what the cohort is and when it started.** A "120% NRR" figure computed on a cohort that excludes the customers who already left is a survivorship statistic, not a retention statistic. ⚠ *This is the standard critique of cohort reporting, and it applies well beyond this field.*
3. **Ask whether the number is a measurement or a target.** Where a retention figure is also a bonus criterion, it stops being a measurement — the same objection the industry has already accepted about satisfaction scores (§6.2, and Reichheld's own statement in §6.2).

### 5.3 Where the money goes: cost-to-serve, segmentation and the funding model

**Cost-to-serve** — the fully loaded cost of serving a customer, including the human time and the support volume — is the number that decides whether a customer is worth keeping. It is also the number most vendors do not compute per customer, because computing it requires joining a CRM to a time-and-expense system to a support system, which is an unglamorous integration project. ⚠ *Practitioner consensus; the data-engineering difficulty is real and understated in vendor content about "customer health".*

**Segmentation logic.** 🔧 *The guide's construction of the decision tree the industry mostly follows, made explicit because it is usually implicit:*

| Customer profile | Modal coverage model | What to watch |
|---|---|---|
| High revenue, high complexity, enterprise governance | Named CSM + named TAM/technical resource + services relationship + executive sponsor | Cost-to-serve can exceed the margin; the model must be checked against revenue, not assumed |
| High revenue, low complexity | Named CSM, pooled technical | Low touch by design; risk is under-serving the account whose sponsor changes |
| Mid revenue | Named or pooled CSM at ratio; scaled digital touches | The ratio is usually set by benchmarking a vendor's own recommendation, not by measured need ⚠ |
| Low revenue, high volume | Digital/self-service, community, pooled support; no named human | The honest position is that no human coverage is profitable here — so the model must not promise one |
| Strategic (low revenue now, high potential) | Named, justified as investment | Requires an explicit investment decision with a review date, or it silently becomes permanent 🔧 |

**Three funding models, and the conditions under which each fits.** 🔧 *The guide's construction; the requirement is to describe the design space, not to crown a winner.*

1. **Cost centre, measured by survey.** Funded from the subscription; measured by satisfaction and retention. *Fits:* when the function's job is genuinely defensive retention on a base that is broadly satisfied. *Breaks:* when the function needs to say no to the sales organisation, because a cost centre's credibility is spent, not earned, and it cannot out-argue a quota.
2. **Revenue centre, carrying renewal and expansion.** The post-sales role carries a number. *Fits:* when expansion is the primary growth engine and the account relationships are genuinely commercial. *Breaks:* when the same person is expected to advocate for the customer *and* sell to them — which practitioners consistently report as the reason customers stop telling them the truth. ⚠ *The tension is a standing practitioner complaint; this guide located no study measuring it.*
3. **Blended: success owns adoption and renewal risk; a separate commercial role owns negotiation and expansion.** *Fits:* the enterprise, where the account has more than one stakeholder and the two jobs demand different styles. *Breaks:* on the seam — the same handover problem as §4.2, one level down.

**The point this section exists to make:** the metrics in §6 are not a measurement problem. They are **the scoreboard of a funding argument**. The reason a satisfaction score gets defended past the point of usefulness is that it is the line item that keeps the function funded — which is precisely why a leader who wants the truth about retention has to measure retention, and not the survey.

---

## 6. The Metrics and Their Critique

**Read this section as an audit, not a catalogue.** Every instrument below is in daily use, and every one of them is a **proxy**. The purpose of this section is to establish, for each: **who originated it, what basis is claimed for it, what its actual evidentiary standing is, what it can and cannot support as a decision input, and how it can be gamed.** §6.7 states the policy for what to do when the proxy and the customer's interest diverge.

### 6.1 The satisfaction and recommendation instruments

#### 6.1.1 Net Promoter Score — the famous one, and the one whose validity is contested

**Origin, precisely.** ✅ NPS was **developed by Bain & Company consultant Fred Reichheld** and **initially popularised in Reichheld's December 2003 *Harvard Business Review* article, "The One Number You Need to Grow"** (*HBR* vol. 81, no. 12, pp. 46–54). ✅ The calculation is a single survey item — likelihood to recommend — with respondents split into **promoters (9–10)**, **passives (7–8)** and **detractors (≤6)**, and the score computed as **% promoters − % detractors**, typically expressed as an integer rather than a percentage. ✅

**The claimed basis.** ⚠ Reichheld reported that the "would you recommend" question was the **best predictor of return business and word-of-mouth marketing** compared with equivalent questions such as "How satisfied are you?", "Does this company deserve your loyalty?" and "Do you intend to return?" ✅ *(that claim is verifiable as what the source said Reichheld said)*. The instrument is **typically interpreted as an indicator of customer loyalty**, and it has been **argued to correlate with revenue growth relative to competitors within an industry** ✅ — *and the reference work's own article carries a "failed verification" tag on that specific claim*, which is itself informative about the state of the evidence.

**Trademark ownership — a fact practitioners often miss.** ✅ **Reichheld owns the registered NPS trademark in conjunction with Bain & Company and Satmetrix.** ⚠ *Consequence, and it is a commercial one: NPS is not a public-domain measurement instrument. It is a trademarked method with owners who sell services around it. This does not invalidate the measure; it does mean every "NPS benchmark" is downstream of a proprietary frame.*

**The peer-reviewed challenge.** ✅ **Keiningham, Cooil, Andreassen and Aksoy, "A Longitudinal Examination of Net Promoter and Firm Revenue Growth," *Journal of Marketing*, vol. 71, no. 3 (July 2007), pp. 39–51** — a peer-reviewed, longitudinal examination that found Net Promoter **not to be a reliable predictor of firm revenue growth relative to other loyalty metrics**. ✅ *(the paper's existence, venue, volume, issue, pages and date are verified; this guide verified the finding at citation-summary level, not by reading the full text — so no coefficient is printed).* The reference work summarises the state of play: scholarly critique **has questioned whether NPS is at all a reliable predictor of company growth**; researchers have noted **no empirical evidence that "likelihood to recommend" is a better predictor of business growth than other loyalty questions** (overall satisfaction, likelihood to purchase again, etc.); and that the recommend question **does not measure anything different from other conventional loyalty-related questions** ✅. A further documented line of criticism: **NPS offers no detailed insight into specific issues** and **over-weights the extremes while devaluing passive respondents** ✅.

**What Reichheld himself has said since — and it is the most useful thing in this subsection.** ✅ Reporting from January 2025 records that in the face of over-use, **Reichheld criticised the over-use as a "tragedy of the commons"**, **recommended tracking repeat customers and referrals by other means**, and **recommended de-linking NPS from employee compensation, saying that linking it made employees care less about pleasing customers and more about getting a high rating** ✅. That statement from the instrument's own author is the single most authoritative support for §6.5's gaming analysis.

**The later revision.** ⚠ Reichheld and co-author Rob Markey's *The Ultimate Question 2.0* (Harvard Business Review Press, 2011) is verified as a published work ✅. A subsequent concept known as the **"earned growth rate"** — a measure built from revenue expansion attributable to existing customers' repeat and referral behaviour — is widely attributed to later Reichheld work. ⛔ **This guide could not verify the earned-growth-rate literature in this pass and therefore asserts neither its authorship nor its content.** Recorded in §16.

**Standing, at the level of a decision:** ✅ **verified as an instrument that exists, is trademarked, is extremely widely adopted (reported as used in some form by two-thirds of Fortune 1000 companies as of 2020 ✅), and whose predictive claim is contested in peer-reviewed literature ✅.** ⛔ **Not verified as a validated predictor of any specific firm's revenue or retention.**

#### 6.1.2 The American Customer Satisfaction Index — the one with an actual causal model

**Origin.** ✅ **ACSI began in 1994, developed by researchers at the National Quality Research Center at the University of Michigan, in cooperation with the American Society for Quality and CFI Group.** ✅ The model was **originally designed in 1989 for the Swedish economy** (the Swedish Customer Satisfaction Barometer), and **both the Swedish version and the ACSI were developed by Claes Fornell**, then a University of Michigan professor and **chairman of CFI Group**. ✅

**Why this one is different, methodologically.** ✅ ACSI is a **structural-equation model**, not a single item: a company's score derives from **three manifest variables** — overall satisfaction, expectancy disconfirmation (falls short of vs exceeds expectations), and comparison to an ideal — each rated 1–10, combined via **weights calculated within the model** and rescaled to 0–100. ✅ It interviews **about 350,000 customers annually**, with respondents screened to ensure they are actual customers, and scores are released monthly. ✅ It is used by researchers, corporations, government agencies, market analysts and investors. ✅

**The honest caveat.** ⚠ *ACSI is a commercial operation as well as a research one:* it is produced by ACSI LLC, it licenses the model internationally (the reference work lists licensees and national variants), and its founder is the chairman of CFI Group, a consultancy. **That does not make the model wrong — it makes the source a research-commercial hybrid, and it means ACSI's public "index" figures and its client-facing services sit in the same house.** What ACSI genuinely has that NPS lacks is **a published causal structure** and a consistent national time series; what it shares with NPS is **the absence of any guarantee that a score change implies a behaviour change for a specific customer**.

#### 6.1.3 The Customer Effort Score — proprietary, popularised in the business press

**Origin.** ✅ The instrument was introduced in **"Stop Trying to Delight Your Customers," *Harvard Business Review*, July–August 2010, by Matthew Dixon, Karen Freeman and Nicholas Toman** — and it belongs to the **CEB (Corporate Executive Board) research lineage**, a subscription research and advisory firm (later merged into Gartner). The article **introduced the Customer Effort Score and asserted that it is a better predictor of loyalty than customer satisfaction measures or the Net Promoter Score**, and pointed to a related diagnostic tool, the **Customer Effort Audit** ✅. ⚠ *The popularisation in book form —*The Effortless Experience* (2013) — is attributed to the same authors in practitioner sources; this guide verified the 2010 article and the CES introduction but not the book's bibliographic details.*

**Evidentiary standing, stated plainly.** ⚠ **CES is a proprietary instrument whose claimed predictive superiority over NPS and satisfaction is a claim made by its originators, published in a practitioner journal, on the basis of their own client survey data — not a peer-reviewed validation.** The claim is testable and has been examined in later academic work (the reference work's own citation trail shows the CES being taken up in the academic customer-feedback-metrics literature), but **this guide did not verify any independent validation study in this pass** ⛔. ⚠ The instrument has also been **revised since introduction** — a "new and improved" version is attributed to Gartner in vendor explainer content ⚠, meaning that two companies using "CES" may be using different question wordings and scales. **That alone disqualifies cross-company CES comparison.**

#### 6.1.4 CSAT — the one with no single originator, and the one treated as settled

⚠ **CSAT has no originating paper.** It is a family of transaction- or relationship-level satisfaction items, in the lineage of the expectancy-disconfirmation research that also underlies ACSI's second manifest variable. Its standing is best described as **ubiquitous, unstandardised, and unexamined**.

**What CSAT can support:** a reading of how a specific interaction felt to a specific respondent, useful for **detecting process failure and training gaps within one's own operation, over time, with a stable question and a stable administration method.** 🔧 *That sentence is this guide's construction of the honest use case.*

**What CSAT cannot support:** a claim about the customer's future behaviour, a comparison to another company, or a claim about the business overall. ⚠ *The general finding that satisfaction is a weak and inconsistent predictor of repurchase behaviour is long-established in the marketing literature; this guide did not re-verify a specific study in this pass and therefore states the point as the standing position of the literature rather than as a cited result.*

#### 6.1.5 The satisfaction standards — what is actually standardised, and what is not

This is worth stating because "we follow the ISO standard on customer satisfaction" is used as an evidentiary claim. ✅ **ISO does publish customer-satisfaction guidance, and the four relevant numbers are:** **ISO 10001** (Quality management — Customer satisfaction — Guidelines for codes of conduct for organizations, listed as ISO 10001:2007); **ISO 10002** (… Guidelines for complaints handling in organizations, listed as ISO 10002:2014); **ISO 10003** (… Guidelines for dispute resolution external to organizations, listed as ISO 10003:2007); and **ISO 10004** (… Guidelines for monitoring and measuring, listed as ISO 10004:2008). ✅ *(Source: the reference work's ISO-standards list for the 10000–10999 range, which also notes that ISO standards are copyrighted, mostly chargeable, and current editions must be checked against the ISO catalogue.)*

For contact operations specifically: **ISO 18295-1:2017** ("Requirements for customer contact centres") and **ISO 18295-2:2017** ("Requirements for clients using the services of customer contact centres") ✅ *(same source)*.

⚠ **Two cautions this guide attaches to those facts.** First, the editions visible in the list used here are **2014 (10002) and 2008 (10004)**; **the current editions could not be confirmed in this pass** — and the ISO catalogue, not the list, is authoritative. Second, and more important: **these are *guidelines* for how to run complaints handling and how to monitor satisfaction. None of them validates a particular instrument** — compliance with ISO 10004 tells you the vendor has a disciplined monitoring process, not that the number it reports predicts anything.

### 6.2 Table: the instrument audit at a glance

| Instrument | Originator | Claimed basis | Actual evidentiary standing | Can support | Cannot support | Gamable by |
|---|---|---|---|---|---|---|
| **NPS** | Fred Reichheld / Bain & Company, *HBR* Dec 2003 ✅ | The recommend item is the best predictor of return business and word of mouth ✅ (claim as reported) | Contested in peer review (Keiningham et al., *Journal of Marketing* 71(3), 2007 ✅); no evidence it outperforms other loyalty items ✅; no diagnostic detail ✅; extremes-weighted ✅; trademarked (Reichheld, Bain, Satmetrix) ✅ | A directional, internally comparable trend on one stable question; a conversation starter | Firm revenue growth; comparative benchmarking across industries; any causal claim | Survey timing and framing; respondent selection; who is asked; asking only happy accounts; pre-emptive follow-up before the survey lands |
| **ACSI** | Claes Fornell, National Quality Research Center, University of Michigan, 1994 (model prototyped for Sweden 1989) ✅ | A causal, structural-equation model over three manifest variables ✅ | The most methodologically articulated of the instruments; national time series; commercially operated ✅/⚠ | Comparisons across sectors over time; macro-level trend reading | Any individual firm's behaviour; the specific reason a customer will leave | Category-level aggregation; published-index composition |
| **CES** | Dixon, Freeman & Toman, *HBR* Jul–Aug 2010, CEB research lineage ✅ | Effort is a better predictor of loyalty than satisfaction or NPS ✅ (claim by originators) | Proprietary; claimed superiority rests on originator client data; later revised (question wording/scales differ) ⚠; no independent validation verified ⛔ | Within-operation diagnosis of friction and repeat-contact handling | Cross-company comparison; any claim of superiority over another instrument | Making it easy to say "no effort" — closing contacts fast rather than solving them |
| **CSAT** | No single originator; expectancy-disconfirmation lineage ⚠ | Convention | Ubiquitous, unstandardised, largely unexamined ⚠ | Within-operation trend on a stable item; detecting process failure | Behaviour, comparison, strategy | Asking at the point of maximum goodwill; coaching to a score; excluding unhappy populations |
| **Retention ratios (logo/GRR/NRR)** | Industry usage (SaaS); **disclosed by listed companies in regulatory filings** ✅ | Revenue arithmetic | The only instruments here with a **primary-source corporate disclosure channel** (e.g. the phrase "dollar-based net expansion rate" appears in 10-K filings; 19 such filings were returned by a regulatory full-text query for 2023–2026, including **Twilio Inc.'s** 10-K for FY2024 filed 26 Feb 2025 and its FY2022/FY2023/FY2025 filings) ✅ | The commercial truth about a cohort, if the definition is stated | Anything about why, or about individual account health | Cohort selection; period choice; netting contraction against expansion; one big expander masking a shrinking core |

🔧 *The table's structure and the "can support / cannot support / gamable by" columns are this guide's construction. The origin, basis and standing entries are sourced as marked.*

### 6.3 The operational quality measures, and their standing

These are the measures that belong to contact-centre and service-management practice. They are **internally coherent and externally meaningless** — they measure whether an operation is disciplined, not whether a customer is well served. ⚠ *Statements about their standing are the guide's characterisation of practitioner usage, not sourced findings; the contact-centre benchmarking industry that publishes such figures (for example **COPC Inc.**, which provides certification, training and research and whose 2026 published content is explicitly about "moving from AI hype to measurable business outcomes" ✅) is itself a vendor of the certification it benchmarks against.*

| Measure | What it is | Known distortion |
|---|---|---|
| **First-contact resolution (FCR)** | Proportion of contacts resolved without transfer, callback or reopen | Definition is set by the reporting tool; a short interaction with a wrong answer can be "resolved"; customers whose problem recurs in a new ticket are invisible. ⚠ *Vendor-claimed correlations between FCR and revenue/loyalty circulate widely in contact-centre vendor content; this guide could not verify any of them to source+methodology standard — see §16.* |
| **Average handle time (AHT)** | Mean duration of a contact | Optimising it produces faster, worse resolutions and drives repeat contact; the classic conflict with FCR |
| **Service level / SLA attainment** | Proportion of contacts answered within a target time, or a contractual response commitment | Measured against the vendor's clock; excludes the waiting that happens outside the queue |
| **Reopen rate / repeat contact rate** | Proportion of resolved contacts that return | The best available antidote to FCR gaming, and rarely reported — because it is unflattering |
| **Escalation rate** | Proportion of contacts escalated internally | Ambiguous: high can mean brittle product or healthy intake; the measure needs a quality dimension to mean anything |
| **Backlog / ageing** | Open items and their age distribution | Honest and unglamorous; the least gamable operational measure available |
| **Deflection rate** | Contacts prevented from reaching a human at all (§10) | Measures channel behaviour, not problem resolution — see §10.2 |

### 6.4 Health scores — the composite that is nobody's validated instrument

⚠ **A "customer health score" is a composite index — telemetry, usage, support volume, survey results, sometimes sentiment — weighted by whoever configured it, with thresholds set by whoever owns the number.** It is usually represented by a dashboard's red/amber/green. There is **no published, peer-reviewed validation** of any vendor's health-score model that this guide located ⛔ — though the concept *is* studied: the peer-reviewed literature on this field includes work on **"customer health" in B2B retention** ✅ *(Hochstein et al., Industrial Marketing Management, cited as December 2023)*, which establishes that the construct is researched without establishing any specific commercial model as validated.

**The honest position:** a health score is **a structured conversation aid**, and it is defensible when its weights are chosen by the team that must act on it, its inputs are auditable, and it is used to prioritise attention rather than to make a contractual or automated decision. It is not defensible when it is sold as predictive. 🔧

### 6.5 How each metric can be gamed — the general mechanism

Every instrument above shares one vulnerability, and it is not a flaw in the instrument; it is a property of **measuring a system that contains the people being measured**:

> **When a proxy becomes a target, and the target-holders control the measurement process, the proxy decouples from the thing it proxied.** 🔧 *This is a restatement of the general principle long known in performance-measurement literature (the "gaming"/Goodhart-type critique); the field's own most senior witness to it is Reichheld's 2025 statement that linking NPS to compensation makes employees care about the rating rather than the customer ✅.*

The customer-facing role is unusually exposed to this, for three structural reasons: 🔧 *(the guide's construction)*

1. **The role administers the survey.** Many vendors let the account owner choose when to send the satisfaction survey. Timing is therefore a controllable variable, and a person with a score target will control it.
2. **The role can pre-empt.** A representative who knows which accounts are unhappy can resolve, apologise, or avoid asking. All three raise the score without changing the underlying state.
3. **The role is measured on things it cannot cause, and therefore trades in the ones it can.** This produces the specific pathology the next sections describe: **satisfaction improving while retention declines** (§12, §14) — which is coherent, not paradoxical. A team measured on surveys optimises surveys; a team measured on retention optimises retention. If only one is measured, only one improves.

### 6.6 The retention ratios as measures, and why §6 exists at all

The retention ratios in §5.2 are the only instruments in this guide's scope with a **primary-source disclosure channel**: they appear in the audited-adjacent narrative of listed companies' annual reports, which is why the regulatory full-text search matters. ✅ *(Verified use: the phrase "dollar-based net expansion rate" appears across 10-K filings; a full-text query over filings dated 2023–2026 returned 19 hits, including Twilio Inc.'s annual reports for fiscal years 2022, 2023, 2024 and 2025, with the FY2024 10-K filed 26 February 2025 ✅.)* That is the strongest available class of evidence in this whole domain: **a company that faces securities-law liability for the number it prints, printing the number.**

But three things remain true even of a disclosed figure:

1. **The definition is the company's own.** A filing states its method; the method is not standard across filings. Comparing two companies' net expansion rates is comparing two definitions. 🔧
2. **Net expansion is an aggregate that can conceal.** A cohort at 115% net retention might be one large account at 170% plus a majority contracting — the two situations require opposite actions. The strict complement, **gross revenue retention**, is the measure that cannot be flattered. 🔧
3. **It is retrospective.** Retention tells you what happened; it does not tell you which account is about to leave, which is the only actionable question in the job. That gap is what health scores exist to fill (§6.4) — and where the vendor claims are weakest.

### 6.7 What to do when the proxy and the customer's interest diverge

This is the section's practical conclusion, and it is a policy, not a metric. 🔧 *All of the following is this guide's construction.*

**The test.** When a satisfaction score says "fine" and retention says "leaving", the *disagreement itself is the finding* — and the correct response is to find out which of the two instruments is lying. Usually it is the survey, for a specific and findable reason: the wrong respondent (a user rather than the economic buyer); the wrong moment (post-resolution, when relief reads as gratitude); the wrong denominator (asking a subset that excludes the silent); or a survey administered by the person with the score.

**The policy, in five clauses:**

1. **Never manage to a survey as the primary outcome.** The primary outcome is the commercial one (renewal, expansion, gross retention). Surveys are *diagnostic inputs to a hypothesis*, and a survey metric that is not connected to a decision has become a ritual.
2. **Measure both directions of the proxy.** For every satisfaction metric, there is a corresponding hard metric — for CSAT, the reopen rate; for FCR, the repeat-contact rate; for NPS, the sponsor-attrition rate; for health score, the renewal outcome. **Report them together or not at all.** A metric reported alone is a metric that will be gamed.
3. **Sever the measurement from the reward, or accept the consequence.** Where a survey drives a bonus, expect it to be managed — and count that management as a cost. Reichheld's own advice on this is the strongest citation available ✅ *(§6.1.1)*.
4. **When the two conflict, believe the customer's behaviour.** Customers are polite on surveys and decisive in procurement. Behaviour is the signal; sentiment is the noise floor.
5. **Say what the metric cannot support, in writing.** A one-line "this metric is a proxy; it does not measure loyalty, revenue or outcome" on the dashboard prevents the meeting in which a number is quoted as a fact.

⚠ **And the disciplined statement this section must end with:** *every metric in this section is a proxy, all of them can be gamed, and none of them — including the retention ratios — is a direct measurement of whether the customer is better off.* The reason to measure is not that measurement is true. It is that **a proxy tracked honestly, alongside its distortion, and used to ask a question rather than to close one, is better than no measurement at all** — and that the alternative, managing the function by anecdote and seniority, is worse.

---

## 7. The Training and Enablement Discipline

**This is the guide's core deliverable, and it is stated up front that the discipline is thin.** Post-sales and customer-facing roles are, by and large, **not developed by a profession's pedagogy.** They are developed by: an onboarding programme built by whoever built it last time, a knowledge base, a call-review ritual, a vendor certification, and a manager coaching cadence that exists on a slide. Each of those has a defensible version and a degenerate version, and the honest task in this section is to describe both and to mark which claims about them are evidenced.

**The single most important framing statement in this guide:** 🔧 **almost every claim about how to train these roles is a claim by a party that sells the training, the tool, or the certification.** §7 gives each practice its evidence status; §9 gives the general evidence; §15 audits the specific claims.

### 7.1 The modal development model, as actually operated

🔧 *The following five-layer model is this guide's construction, assembled from what firms describe of their own onboarding. It is not a standard, and no source claims it is universal.*

| Layer | Typical content | Who builds it | Typical duration claim | Evidence status |
|---|---|---|---|---|
| 1. Product and policy knowledge | What the product does, licence tiers, SLA terms, escalation policy, refund and credit rules | Product marketing + support leadership | "2–4 weeks" ⚠ | Convention; no verified standard |
| 2. Tooling and process | CRM, ticketing, telemetry dashboards, the playbook library | Operations/enablement | "1–2 weeks" ⚠ | Convention |
| 3. Shadowing and reverse-shadowing | Listening to a senior colleague's calls, then handling calls with a senior listening | The team | "2–6 weeks" ⚠ | Convention; theoretically grounded — see §7.8 |
| 4. Certification | A vendor's product certification, and/or an industry credential | External vendor or body | "days" ⚠ | Weakly evidenced as a *behaviour* intervention; see §7.3 |
| 5. Ongoing coaching | Call review, 1:1s, quality scores, playbook updates | Front-line manager | continuous | The most evidence-supported layer, and usually the most under-resourced — §9.5 |

**The structural observation:** the modal model concentrates **all of its cost at the front (layers 1–3) and all of its benefit claim at the end (layer 5)**, while resourcing the middle heavily and the end thinly. That is the same shape as the general corporate-training failure identified in §9. 🔧

### 7.2 The onboarding and ramp model — and the honest position on ramp duration

**What a good onboarding model looks like** 🔧 *(construction; the components are each conventional):* a sequenced curriculum with a **knowledge check that gates progression** rather than a completion certificate; a **supervised live-work phase** with a defined release of scope (first read-only exposure, then low-risk accounts, then full book); **explicit criteria for leaving ramp** that are behavioural rather than temporal ("handles a technical escalation to a documented resolution" rather than "attended the module"); and a **named owner of the ramp outcome** who is not the new hire.

**The honest position on ramp duration.** ⛔ **This guide could not verify any benchmark for how long it takes to ramp a post-sales role to full productivity.** The figures that circulate — "3 months to first renewal", "6 months to full book", "9 months for enterprise" — are **practitioner convention and vendor content**, and they are **not measurements of a defined population**. What *can* be said as an evidenced structural claim is general and belongs to §9: skill acquisition to automaticity is a function of **practice with feedback over time**, not of time elapsed, and it decays without reinforcement ✅ *(see §9.2–§9.3)*. Any ramp plan built on a calendar rather than on demonstrated behaviour is a plan to fill a seat, not to build capability. 🔧

⚠ **And one honest note about the "ramp" word itself:** in this field it names two different things — the *hire's* ramp to productivity and the *customer's* ramp to first value. Vendors selling onboarding software use it for the second while the training market uses it for the first, and the conflation is a reliable sign that a pitch is switching between them.

### 7.3 Certification and accreditation — and what it is actually worth

Three distinct categories, often presented as one:

**(i) Methodology certification owned by a non-profit consortium.** ✅ The clearest example in this domain is **Knowledge-Centered Service (KCS)**, owned by the **Consortium for Service Innovation**, which describes itself as **"a non-profit alliance of service and support organizations focused on innovation around customer engagement, productivity, and success."** ✅ The Consortium's current site presents the methodology as **"Knowledge-Centered Success (KCS)"** and lists its other initiatives as **Intelligent Swarming, Predictive Customer Engagement, Customer Experience, and Leadership in an Adaptive Organization**, with member companies including **Oracle, SAP, Autodesk, F5, DTCC, PTC, Verint, NICE CXone, Coveo, Vertex, Ping Identity, RightAnswers and Tyler Technologies** ✅. ⚠ **The honest commercial note:** the Consortium is a non-profit *and* it sells KCS Fundamentals courses and exams, with a discount for buying a course and exam together ✅. That is not disqualifying — it is the standard structure of a professional body — but it means the methodology, its validation of itself, and its revenue sit in the same organisation.

**(ii) Industry certification owned by a commercial benchmarking and consulting firm.** ✅ **COPC Inc.** provides contact-centre and CX **consulting, certification, research, training and managed services**, and publishes benchmarking dashboards and premium research ✅. ⚠ *Reading this correctly matters:* a COPC certification is an **audit against COPC's own standard, conducted by COPC** — the strongest available form of *operational* benchmarking in contact centres, and simultaneously a circular evidentiary structure if a buyer treats the certification itself as independent evidence of customer outcomes.

**(iii) Vendor product certifications.** ⚠ Certifications offered by software vendors for their own products (the customer-success platform vendors, the CRM vendors, the CCaaS vendors). **What they establish:** that a person has passed a test about one product, and — for the employer — that the vendor's partner or customer has a trained population. **What they do not establish:** capability in the role. **Their market recognition is unverified in this pass** ⛔: this guide located no evidence on how these credentials are weighted in hiring, and will not assert that they are or are not valued.

**(iv) Professional-body credentials for the customer-experience discipline.** ⚠ **The Customer Experience Professionals Association (CXPA) and its CCXP credential** are widely cited in practitioner content as the principal certification for customer-experience professionals. ⛔ **This guide could not verify the association's ownership of the credential, its launch date, its competency framework or its market recognition in this pass** — the site was unreachable to the extraction tool used. It is recorded in §16 rather than described. ⚠ *Note the general pattern this leaves: the *existence* of a CX certification market is beyond dispute in practitioner content, and this guide verified none of its specifics.*

**The honest assessment, stated once, plainly:** 🔧 *the guide's construction, and it is the position the evidence supports.*

1. **A certification proves exposure and, at most, recall.** It is a **hiring filter**, not a capability measure — which is fine, as long as it is used as a filter.
2. **A certification is strongest where it certifies a *methodology tied to a system the person will actually operate*** (e.g. KCS, where the method is the daily work of article-based problem solving), and weakest where it certifies familiarity with a product.
3. **The most valuable credential in this field is not a certificate at all**; it is a **documented escalation record** — a named issue you owned, the internal argument you won, and the outcome. That is portable across employers in a way that no vendor badge is. 🔧
4. **Beware the audit-as-credential.** An organisation certified against a benchmarking standard has demonstrated *process conformance*, not customer outcomes. Presenting the certificate as evidence of retention is a category error.

### 7.4 The knowledge base: the dependency nobody staffs for

The knowledge base is the mechanism by which a customer-facing organisation converts individual problem-solving into institutional capability, and its failure mode is well understood:

- **It decays.** Articles describe a product version that no longer exists; the newest hires learn from the stale article and the newest failures never get written.
- **It is nobody's primary job.** The person who solved the novel problem is measured on resolution, not documentation — the identical incentive problem to §4.2's handover.
- **KCS addresses exactly this** ✅ — its core claim is that the knowledge capture is integrated into the resolution workflow rather than added to it, so that the solving and the writing are the same act. ⚠ *That is the methodology as its owner describes it; the guide verified the methodology's ownership, its member base, and its certification structure, and did not verify any outcome study of KCS adoption.*
- **In the AI era (see §10), the knowledge base stops being an internal asset and becomes the retrieval corpus for the automation.** The organisation that let its KB rot discovers this when the AI assistant answers from a stale article. ⚠ *Practitioner consensus; the mechanism is arithmetical, not empirical.*

### 7.5 Call review and quality assurance — the practice with the clearest known failure mode

**How it is done.** ⚠ *Description of common practice; no standard governs call review in post-sales software, and the contact-centre standards (ISO 18295-1/-2:2017 ✅) address the operation rather than the coaching method.* A sample of interactions (often a fixed number per person per month) is scored by a reviewer — a team lead, a dedicated QA function, or a peer — against a scorecard of items: greeting, verification, diagnosis, documentation, tone, correct policy application, resolution. Scores feed a quality metric, sometimes a bonus, and sometimes a coaching conversation.

**What it measures, honestly.** 🔧 **A scorecard measures conformance to the scorecard.** Three consequences follow, and they are why practitioners most often cite this practice as corrosive: **(1) compliance crowds out resolution** — a person scored on process adherence will adhere to process on the calls where the right answer is to deviate, which is exactly the skill the role needs (§8.2); **(2) it converts coaching into judgement** — a review that produces a number is an appraisal, one that produces a conversation is development, and most regimes do the first while advertising the second; **(3) it is a supervisory mechanism over emotional display, and the literature is explicit about that.** ✅ **Hochschild's definition of jobs involving emotional labour includes the criterion that the employer can, through *training and supervision*, exercise a degree of control over the employee's emotional activities** ✅ *(§3.2)* — the call review is that control made concrete. The meta-analytic evidence associates **surface acting** with **strain and burnout** ✅ *(Hülsheger & Schewe, *Journal of Applied Psychology*; 494 correlations from 95 independent studies)*. A QA regime that scores *visible warmth* while the representative manages an outcome they cannot control is, on this evidence, **an industrial process for manufacturing surface acting** ⚠ *(the inferential step to this practice is the guide's; the meta-analytic finding and Hochschild's criterion are sourced).*

**What a defensible regime looks like** 🔧 *(construction):* the sample includes **the calls the representative chose** (so judgement is reviewed, not only compliance); the scorecard is **short and weighted toward outcome** (was the problem actually gone); the review has **no bonus attachment** (§6.7 clause 3); and the reviewer's output is **one specific behavioural change for the next week**, not a percentage. A regime that cannot name next week's change is administration, not coaching.

### 7.6 Simulation and role-play

- **What it is:** scripted or semi-scripted practice of the hard interactions — the renewal conversation, the escalation call, the price objection, the "you promised us this and it isn't there" call, the delivery of bad news about a defect.
- **Why it is theoretically well-founded:** it is the practical form of **cognitive apprenticeship** ✅, in which the master models the skill in a realistic context and the learner enacts it with help — the theory's own emphasis is that skills taught **independently of their real-world context and situation are less effective** ✅ *(Collins and Brown's work, developed from constructivist and situated-cognition research; the reference work cites Collins, Duguid and Brown (1989) on the situated-cognition basis)*. Simulation is also how deliberate practice is instantiated, which matters because §9.4 reports the uncomfortable finding about deliberate practice's explanatory power in *professions*.
- **What it is not:** a video library. Watching a good call is not practice; the practice is the enactment with feedback.
- **Evidence status:** ⚠ **the underlying learning theory is well established; the specific transfer of role-play to live customer conversations in this role family is not something this guide verified.** Treat role-play as *theoretically motivated convention* rather than proven intervention. 🔧

### 7.7 Coaching models and cadence

- **GROW** is the coaching model most commonly named in this domain. ⚠ **Origin, stated with the honest caveat:** the GROW model **was developed in the United Kingdom and has been used extensively in corporate coaching from the late 1980s and 1990s**, and **there are a number of different versions of it** ✅ *(the reference work's account; the specific original authorship attributed in practitioner literature — commonly to Graham Alexander, Alan Fine and Sir John Whitmore, with Whitmore's *Coaching for Performance* widely cited as the popularising text — was not verified in this pass ⚠, and §16 records that).* The stages are **Goal, Reality, Obstacles/Options, Way Forward** ✅. ⚠ **The model is a convention widely transmitted, not a validated coaching protocol**; no outcome study of GROW in post-sales coaching was located.
- **Cognitive apprenticeship** ✅ supplies the theoretical basis for **shadowing and modelling** — the master demonstrating in a real situation and making tacit reasoning explicit — and for the **staged release of autonomy** that §7.2's ramp model assumes ✅ *(same source)*.
- **Coaching cadence.** ⚠ **The claim that coaching should be weekly, or that "the manager must coach X hours per week", is practitioner convention.** What is genuinely evidenced at the general level is in §9.5: **manager involvement and reinforcement are among the conditions associated with training transfer**, and the systematic weakness is that most organisations measure training by reaction rather than by behaviour. A cadence set by a management preference rather than by observed behaviour is a ritual with a schedule.
- **Cross-reference rather than re-teach:** situational coaching (matching style to the individual's readiness) is owned by [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md); the structural conversation about what the manager owes the new hire is owned by [the_first_90_days_guide.md](the_first_90_days_guide.md). This guide does not re-derive either.

### 7.8 The apprenticeship and shadowing structure

🔧 *Construction, resting on the cognitive-apprenticeship theory above ✅:*

| Phase | What the learner does | What the mentor does | What makes it real |
|---|---|---|---|
| **Modelling** | Watches, and hears the reasoning explained *while* it happens | Narrates the tacit judgement — why this escalation and not that one, what they noticed in the customer's phrasing | Narration, not observation. Watching without narration teaches the script |
| **Coaching** | Performs with the mentor present, prompting allowed | Intervenes, debriefs immediately, corrects in the moment | Immediacy. A debrief three weeks later is a review, not coaching |
| **Scaffolding and fading** | Performs, mentor withdraws prompts in stages | Stops prompting before stopping supervising | The mentor's removal is scheduled, not accidental |
| **Articulation and reflection** | Explains their own reasoning out loud | Asks "why did you do that?" and withholds the answer | Forces the learner from procedure to judgement |
| **Exploration** | Handles the novel case with the mentor only as a fallback | Is available and does not take over | The moment the role actually becomes theirs |

⚠ *The phase names come from the cognitive-apprenticeship literature's own vocabulary; the application to this role family — particularly the "narration" and "scheduled fading" design requirements — is this guide's construction and is not an evidenced intervention.*

### 7.9 Evidence status, in one table

| Practice | Evidence status for *behaviour change or outcome* in this role family |
|---|---|
| Product/policy onboarding; tooling | ⚠ Convention. **Knowledge transfer ≠ behaviour change** (§9.1) |
| Shadowing / reverse-shadowing; simulation and role-play | ⚠ Theoretically grounded (cognitive apprenticeship ✅; practice with feedback); no role-family outcome study verified |
| Certification (KCS-style, COPC-style, vendor product); knowledge base (KCS) | ⛔ Not verified as producing behaviour change. Strongest as a hiring filter or process-conformance signal 🔧. KCS methodology ownership verified ✅; outcome studies not verified |
| Call review / QA scoring | ⚠ **Known failure mode: rewards compliance over resolution; supervisory control over emotional display ✅/🔧** (§7.5) |
| Coaching (GROW et al.); manager capability; reinforcement and spaced refresh; outcome-linked evaluation | ⚠ Convention, except **reinforcement/spacing, which is the best-supported layer** ✅ (§9.3); the four-level evaluation frame is widely used and criticised ✅ |

---

## 8. The Skills Decomposition

**The point of this section is the gap between the skills that are trained and the skills that are needed.** Documented practice in this field trains, in order of effort spent: product knowledge, process compliance, tooling, and methodology vocabulary. The skills that actually determine outcomes are, in order of importance: **diagnostic judgement, difficult-conversation handling, commercial judgement about where to spend scarce attention, cross-organisational influence, written communication that survives being forwarded, and the resilience to do it again next week.** The gap is not accidental — the trained skills are the ones that are cheap to teach and easy to test. 🔧 *(This framing is the guide's own.)*

**A note on method.** This section names each skill, says what it actually consists of in this role, says how it is usually trained versus how the evidence suggests it should be, and then **cross-references the repository guide that owns it by name** rather than re-teaching it ⚠ *(any skill above is a candidate for)*.

### 8.1 Product and domain knowledge — trained heavily, necessary, not sufficient

**What it is:** what the product does and does not do; what the contractual terms permit; how the customer's own industry works well enough to know which of their problems is real.

**How it is trained:** product modules, certification, internal wiki, shadowing. This is the layer organisations invest in most, and the layer with the shortest useful life — new releases invalidate it continuously. The maintenance burden is the reason knowledge bases rot (§7.4).

**The honest limit:** knowledge answers "what is possible". The role's hard problems are "what should we do here", which knowledge cannot answer. A representative who knows everything about the product and cannot tell a genuine escalation from a bluff is less useful than one who knows 70% and can. 🔧

**Owned elsewhere:** product literacy and product decision-making → [../technology/product_thinking_guide.md](../technology/product_thinking_guide.md) and [product_management_frameworks_guide.md](product_management_frameworks_guide.md). This guide does not re-derive either.

### 8.2 Diagnostic reasoning under uncertainty

**What it is in this role:** the ability to take an ambiguous, partially-reported, emotionally weighted customer description of a symptom, and reason toward a probable cause and a proportionate next action — with incomplete data, under time pressure, and with a customer who wants certainty now. It is the same kind of reasoning as incident diagnosis, applied to a person rather than a system. 🔧

**Where the craft comes from:** structured problem management — separating the symptom from the cause, working the problem rather than the incident. That discipline is owned by [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) (the incident/problem distinction and the escalation mechanics); this guide notes only that **the transfer from system diagnosis to account diagnosis is the highest-value untrained skill in the family** and that no vendor sells it. 🔧

**How it is usually trained:** not at all, except by accident through shadowing. ⚠ *Practitioner observation.* **How it can be trained:** case-based reasoning on real historical escalations with the outcome hidden, which is the method the diagnostic professions use and which requires a maintained case library. The dependency is the same as §7.4's. 🔧

### 8.3 Difficult conversations and de-escalation — the trained-by-nobody skill with the best-supported theory

**What it is:** telling a customer something they do not want to hear, without either capitulating to something you cannot deliver or damaging the relationship below the threshold at which the truth stops flowing. The specific conversations: the defect, the delay, the price increase, the end of a feature, the "your expectation was never what we sold".

**What is going on psychologically, stated once and cross-referenced thereafter:** this is where §3.2's **surface acting** lives ✅ — the representative must display a calm and warmth the interaction does not warrant, which is precisely the condition the meta-analysis associates with **strain and burnout** ✅ *(Hülsheger & Schewe, *Journal of Applied Psychology*; 95 independent studies)*. Training that teaches *displaying* empathy without giving the person a truthful position to stand on deepens the problem. 🔧 *This inferential step is the guide's; the meta-analytic link is sourced.*

**Owned elsewhere, by name:** de-escalation and conflict handling → [conflict_management_guide.md](conflict_management_guide.md); the conversation structure for contentious stakeholder interaction → [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md); structured group interaction where a review is going badly → [facilitation_skills_guide.md](facilitation_skills_guide.md); reading the unspoken signal → [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md) and [micro_expressions_guide.md](micro_expressions_guide.md). None of these is re-taught here.

### 8.4 Executive and written communication

**What it is:** two different skills that the same job needs.

- **Executive communication:** compressing an account's situation into a decision a senior person can make in five minutes, on the record, in their vocabulary.
- **Written communication:** the artefact that must survive being forwarded — the escalation summary, the review note, the risk memo, the follow-up email that becomes the record of what was agreed. In this role, **the writing *is* the relationship's memory**, and it is the one artefact that gets read when the person is not in the room. 🔧

**Why it is under-trained despite being critical:** writing is the deliverable nobody grades, and the business review is the only place where its quality becomes visible — usually too late. ⚠

**Owned elsewhere, by name:** [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) (stakeholder communication and executive engagement); [facilitation_skills_guide.md](facilitation_skills_guide.md) (running the review itself); [business_case_development_guide.md](business_case_development_guide.md) (the written justification form); [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) (the internal written politics).

### 8.5 Commercial judgement — when to invest, when to escalate, when to let go

**What it is:** the judgement that decides where scarce attention goes. It is the skill §5.3's segmentation requires and §7 never teaches, because it requires the person to understand **the arithmetic of cost-to-serve** and be allowed to act on it: this account is worth an executive's time; that one is not; this escalation is a real risk; that one is a negotiation position; this renewal is lost and should be conceded quickly to protect the relationship for the year after. 🔧

**Why it is untrained:** it requires the role to hold a *number* (the account's economics) and a *licence* (to deprioritise). Organisations that withhold both cannot train it, and then complain that the team escalates everything. ⚠ *Practitioner consensus; the incentive analysis is the guide's.*

**Owned elsewhere, by name:** [business_case_development_guide.md](business_case_development_guide.md) (constructing the commercial case); [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) (what is being sold and how it is monetised); [meddicc_guide.md](meddicc_guide.md) (the qualification discipline that determines whether the promise this role inherits was ever deliverable — this guide deliberately does not re-derive it).

### 8.6 Cross-organisational influence without authority

**What it is:** getting an engineering team that does not report to you, is not measured on your outcome, and has a full backlog, to prioritise the thing that will save your account. The mechanics are: evidence, a named owner, a diagnosis of *why* the priority conflict exists, a small ask, and reciprocity banked over time. 🔧

**Why it is the job's primary mechanism:** §3.4 is explicit — the role is accountable for outcomes it cannot command. Everything else in this section is subordinate to this one. 🔧

**Owned elsewhere, by name:** [authority_skills_guide.md](authority_skills_guide.md) (authority and its exercise without formal position); [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) (lateral and upward influence); [directly_responsible_individual_guide.md](directly_responsible_individual_guide.md) (accountability structures and the DRI pattern for making one name own an issue); [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md).

### 8.7 Resilience — sustain it, and understand what you are sustaining

**What it is:** the capacity to absorb repeated, structurally unfair contact and return the next day able to be genuinely useful — as distinct from the capacity to keep smiling.

**The honest formulation, and it is the guide's:** ⚠ **resilience training that teaches the individual to tolerate a structurally unfair load is not a capability intervention; it is an acceptance mechanism.** 🔧 The load in this role has identifiable structural sources — the responsibility/authority gap (§3.4), the funding model (§5.3), the measurement regime (§6), the handover seam (§4.2) — and every one of them is fixable by management and none of them is fixable by the individual. That is not an argument against individual resilience practice; it is an argument against **substituting it for fixing the causes.**

**Owned elsewhere, by name:** [resilience_engineering_guide.md](resilience_engineering_guide.md) (personal resilience as an engineered practice); [the_first_90_days_guide.md](the_first_90_days_guide.md) (what to build in a new role before the load arrives); [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md) (for the manager's side of regulating load).

### 8.8 The decomposition table: trained versus needed

🔧 *The rating column is the guide's judgement; the cross-references are by name and are the repository's own.*

| Skill | Trained by most organisations? | Determines outcomes? | Where it is owned |
|---|---|---|---|
| Product/policy knowledge; tooling and process | ✅ heavily | Necessary, not sufficient; tooling is hygiene | [../technology/product_thinking_guide.md](../technology/product_thinking_guide.md), [product_management_frameworks_guide.md](product_management_frameworks_guide.md), [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) |
| Diagnostic reasoning under uncertainty | ❌ almost never | **Yes — highest** | [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) |
| Difficult conversations; reading the unspoken | ⚠ partially, badly | **Yes** | [conflict_management_guide.md](conflict_management_guide.md), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [facilitation_skills_guide.md](facilitation_skills_guide.md), [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md), [micro_expressions_guide.md](micro_expressions_guide.md) |
| Executive and written communication | ⚠ occasionally | **Yes** | [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [business_case_development_guide.md](business_case_development_guide.md) |
| Commercial judgement (invest/escalate/concede) | ❌ | **Yes** | [business_case_development_guide.md](business_case_development_guide.md), [meddicc_guide.md](meddicc_guide.md), [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) |
| Influence without authority | ⚠ as "stakeholder management" | **Yes** | [authority_skills_guide.md](authority_skills_guide.md), [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md), [directly_responsible_individual_guide.md](directly_responsible_individual_guide.md) |
| Resilience; managing the transition into the role | ✅ (often as symptom management) | Necessary to sustain, insufficient as strategy | [resilience_engineering_guide.md](resilience_engineering_guide.md), [the_first_90_days_guide.md](the_first_90_days_guide.md) |

**The finding this table states:** the two skills rated highest for outcome impact — **diagnostic reasoning** and **commercial judgement** — are the two that organisations almost never train, and the one they train most (product knowledge) is the one that expires fastest. 🔧

---

## 9. What the Evidence Says About Training

**This section is deliberately separate from the previous two, because the honest position is uncomfortable and needs its own space: the evidence that workplace training changes behaviour and business outcomes is much weaker, thinner and more unevenly distributed than the training industry's own material implies — and what *is* well established is mostly about memory, practice and reinforcement rather than about courses.** Each claim below carries its status.

### 9.1 Knowledge transfer is not behaviour change — the distinction that invalidates most training evaluation

**The three transfer outcomes.** ✅ Formal learning research names three possible outcomes of training: **positive transfer** (training increases performance in the targeted job), **negative transfer** (it decreases performance), and **zero transfer** (neither). ✅ *(Source: the reference work's treatment of transfer of training, citing the industrial-organisational psychology literature.)* The existence of *negative* transfer is not a curiosity — it is the reason a badly designed intervention can be worse than none, for example by teaching a script that the customer can hear.

**The canonical model, and what it includes.** ✅ **Baldwin and Ford (1988) is the most commonly cited model of transfer of training**, defining it as **the generalisation and maintenance of material learned in training to the work environment** ✅. Its three-stage structure is the load-bearing part: **training inputs** (training design, **the work environment**, and **trainee characteristics**) → **training outputs** (learning and retention) → **conditions of transfer**. ✅ *(same source)* **Note what the model puts in the input box: the work environment.** The practitioner implication is direct and is the reason §7.5 and §9.5 matter more than course design — 🔧 *a course delivered into an environment that punishes the taught behaviour produces zero or negative transfer by construction.*

### 9.2 Decay: what the forgetting curve actually is, and what it is not

**Origin.** ✅ The **forgetting curve** originates with **Hermann Ebbinghaus**, who ran the studies **between 1880 and 1885 on himself** and published them in 1885 as *Über das Gedächtnis* (*Memory: A Contribution to Experimental Psychology*). ✅ The curve describes the **decline of retention over time when there is no attempt to retain the material**, and the popular representation is of **memory of newly learned material roughly halving within days or weeks absent conscious review** ✅. The phenomenon is the **transience** category of memory failure ✅.

**The modern replication.** ✅ **Murre and Dros, "Replication and Analysis of Ebbinghaus' Forgetting Curve," *PLOS ONE*, 2015** (DOI 10.1371/journal.pone.0120644) replicated the 1880 experiment using the **method of savings**, with **one subject who spent 70 hours learning and relearning lists**, testing at intervals of **20 minutes, 1 hour, 9 hours, 1 day, 2 days and 31 days**, and reported results **similar to Ebbinghaus' original data** ✅. The paper also analysed serial-position effects and examined which mathematical functions best describe forgetting ✅.

**The extrapolation, stated plainly — this is the important caveat.** ⚠ **The forgetting curve was established on nonsense-syllable list learning by a single subject in the 1880s and a single subject in 2015. Applying it to corporate training — "your team will forget 70% of the workshop by Friday" — is an extrapolation, not a finding.** The direction of the effect (unused knowledge decays) is not seriously disputed; **the specific percentages quoted in training content are not from these studies and this guide found no study establishing them for workplace skill acquisition.** ⛔ Accordingly, **no decay percentage is printed in this guide.**

### 9.3 What does slow the decay: spacing and distributed practice

**This is the strongest and most useful evidence in the section, and it points at reinforcement, not at course design.**

✅ **The spacing effect** — the finding that learning is more effective when study sessions are **spaced out** than massed ("cramming") — **was first identified by Ebbinghaus** and has been **supported across many explicit memory tasks** including free recall, recognition, cued recall and frequency estimation ✅. The effect's **robustness and relative resistance to experimental manipulation** are themselves documented ✅.

✅ **The quantified finding:** in **Cepeda et al. (2006)**, participants using **spaced practice outperformed those using massed practice in 259 out of 271 cases** ✅. *(This guide separately flagged, from the same source, the honest caveat that the effect is **not robust for items presented only twice and tested after a 24-hour delay** per Shaughnessy (1977), whereas it is present with four or six presentations ✅ — i.e. the benefit depends on repetition count, which is exactly the reinforcement argument.)*

🔧 **The practical translation for §7** *(the guide's construction):* the highest-return change most post-sales enablement functions can make is not a better course. It is **replacing a one-off event with a spaced sequence** — the same content, revisited at intervals, applied to live cases. The evidence for the underlying mechanism is strong and old; the evidence that organisations do this is weak, and their evaluation practice (§9.6) is why they do not notice. ⚠

### 9.4 The uncomfortable finding: deliberate practice explains very little in professions

**This is the strongest honest counterweight available to the training industry's claims, and it is worth stating precisely.**

✅ **Macnamara, Hambrick and Oswald's meta-analysis, "Deliberate Practice and Performance in Music, Games, Sports, Education, and Professions: A Meta-Analysis"** (published in **Psychological Science**, 2014) covered all major domains in which deliberate practice has been investigated and found that **deliberate practice explained the variance in performance as follows: 26% for games, 21% for music, 18% for sports, 4% for education, and less than 1% for professions.** ✅ *(Verified at abstract level against the article's own abstract as returned by the search index and the publisher/PubMed records; this guide did not read the full text, so no further statistics are cited.)*

**How to read that honestly — both ways**, because this finding is misused in both directions:

- **Against the training industry:** the domains where practice best predicts performance are the ones with **stable, measurable rules and immediate feedback** (games, music, sport). **Professions — the category this guide's subject belongs to — show less than 1%.** An organisation that expects a training programme to produce performance differences on the strength of "practice makes perfect" is relying on a relationship that the meta-analytic evidence does not support in professional work. ✅
- **Against the lazy reading of the finding:** "less than 1%" is a statement about **how much of the variance in performance across people is accounted for by measured deliberate practice as operationalised in those studies.** It is **not** a claim that training is worthless, and not a claim that skill cannot improve. Its real message is structural: **in professions, performance is determined by factors that are not hours of practice** — the context, the task environment, the quality of feedback, the ability to select which problems to work on, and factors the studies did not measure. 🔧 *This interpretation is the guide's; the finding is the meta-analysis's.*
- **The design consequence** 🔧: stop buying training outcomes and start buying **feedback loops and problem selection**. The role's performance depends far more on *which* accounts it is assigned, *what* feedback it receives after each interaction, and whether it has the authority to fix what it finds than on how many hours of instruction it receives.

### 9.5 Reinforcement, feedback and manager involvement

⚠ **The practitioner consensus is unanimous and the instrumentation is thin.** What can be stated at each level of confidence:

- ✅ **Structurally supported:** the canonical transfer model treats **the work environment as an input to training**, alongside training design and trainee characteristics ✅ *(Baldwin & Ford, 1988)*. If the environment does not support the behaviour, transfer fails — this is a formal property of the field's most-cited model, not an opinion.
- ✅ **Memory-side support:** spacing and repetition improve retention vs massed presentation (259/271 cases) ✅, and the benefit depends on repetition count ✅.
- ⚠ **Practitioner convention, widely repeated and not verified here:** that the line manager's involvement is the single largest determinant of whether training transfers; that "training without follow-up is entertainment"; that the first 30 days after a course decide whether it sticks. **These are conventions** — plausible, consistent with the model above, and **not verified as findings in this pass.**
- ⛔ **Rejected as evidence:** any statement of the form "X% of training is lost without reinforcement" with no traceable study. See §9.7.

**The inference the guide draws, labelled as such:** 🔧 *if the environment is an input, then the cheapest effective enablement intervention is not the course — it is the manager's cadence, the feedback the person actually receives, and the authority the role holds. That is also the intervention least likely to be funded, because it does not produce a training budget line, a completion statistic, or a report.*

### 9.6 The four-level model — origin, and the critique that matters

**Origin.** ✅ **Donald Kirkpatrick published a series of articles originating from his doctoral dissertation in the late 1950s describing a four-level training evaluation model**, covering (in the standard order) **reaction, learning, job performance (behaviour), and organisational impact (results)**. ✅ *(Source: the abstract of Reio et al.'s critique, published in *New Horizons in Adult Education and Human Resource Development*.)* ✅ **From its beginning it was easily understood and became one of the most influential evaluation models in the field.** ✅

**The critique.** ✅ **The same source states that while well received and popular, the Kirkpatrick model "has been challenged and criticized by scholars,"** and that empirical studies using the four levels demonstrate **"the usefulness of, but the difficulty in, implementing all four levels."** ✅ The thirty-year retrospective is **Alliger and Janak, "Kirkpatrick's Levels of Training Criteria: Thirty Years Later" (1989)** ✅ — and it is worth noting what that title signals: **the model was being assessed for evidence thirty years after publication and the field's conclusion was that implementing all four levels is difficult.** ✅ The 2017 critique reviews several alternative evaluation models and **notes that many alternatives are variations of Kirkpatrick's original framework.** ✅

⚠ **A specific critique that this guide could not fully verify:** the widely repeated objection that **the four levels do not form a validated causal chain** (i.e. that level 1 reaction does not cause level 2 learning, which does not cause level 3 behaviour, which does not cause level 4 results) is attributed in the training-evaluation literature to named critics. **This pass verified the existence of scholarly criticism and the difficulty of implementing all four levels; it did not verify the specific causal-chain critique's provenance.** ⛔ So: **the critique is carried as flagged, and the guide does not attribute it to a named author it could not check.**

⚠ **The fifth level.** A fifth level — **return on investment**, associated with **Jack Phillips** — is widely described in practitioner content as an extension of the four-level model. ⛔ **This guide could not verify Phillips' authorship, the model's origin date, or its evidentiary status in this pass** (the reference work has no article on it, and the search tool returned nothing). It is recorded in §16 rather than described.

**The systematic problem, and it is the point of the subsection.** ✅ The field's own literature documents the difficulty of implementing all four levels ✅. ⚠ The practitioner-consensus consequence, which this guide reports as convention rather than as a finding: **most organisational training is evaluated at level 1 — the "smile sheet" — because level 1 is free, immediate and flattering, and levels 3 and 4 require measurement or attribution that the training owner does not control.** ⛔ **This guide found no survey establishing what proportion of training is evaluated at each level; it is a widely stated convention, and it is not printed as a statistic here.**

🔧 **The guide's own evaluation rule, offered as construction:** an enablement programme is evaluated by something the *customer* eventually feels — a reopened ticket that no longer reopens, a renewal that happened without an escalation, an account whose adoption metric moved. Anything else is administration. If the programme cannot name that measure before it starts, it is not a programme; it is an event.

### 9.7 The folklore figures, named and disposed of

The training field runs on a small number of statistics that are repeated as settled. Best-known:

- ⛔ **"Only about 10% of training transfers to the job."** This figure is widely attributed to a **1982 practitioner article** (commonly cited as Georgenson, in *Training & Development Journal*), and it is **widely criticised as unsupported** — it appears to be a practitioner's assertion rather than the report of a study. ⛔ **This guide could not verify the original article, its content, or any subsequent study establishing the figure, and therefore rejects it as evidence.** The honest statement is that **transfer is frequently poor** ✅ *(by the Baldwin & Ford model's own logic, since environment is an input)* **and that the "10%" is not a measurement.** Do not cite it.
- ⛔ **Percentage-of-training-lost-to-forgetting figures.** Not established for workplace skill (§9.2). Rejected.
- ⚠ **"Adults learn best when…" generalisations, learning-styles-based course design, and the "70-20-10" split of development.** ⚠ The broad claims circulate as training-industry convention; **this guide verified none of them in this pass and does not assert them.** Where a vendor's pitch rests on one, treat it as a claim, not a premise.

### 9.8 Where the evidence is strong, thin, and conventional

| Question | Evidence status |
|---|---|
| Does unused learned material decay, and does spacing beat massing? | ✅ **Established** (Ebbinghaus 1885, replicated 2015; spacing wins 259/271 in Cepeda et al. 2006) — **but the workplace decay percentages are extrapolation** ⚠ |
| Does the work environment condition whether training transfers, and can training backfire? | ✅ Formal property of the canonical transfer model (Baldwin & Ford 1988); negative transfer is an established outcome |
| Does deliberate practice explain professional performance? | ✅ **Less than 1% of variance in professions** (Macnamara et al., *Psychological Science*, 2014) |
| Is the four-level evaluation model the field's standard, and does it work? | ✅ Influential since the late 1950s; ✅ criticised by scholars; ✅ hard to implement fully; ⚠ causal-chain critique and the ROI fifth level unverified |
| Do certifications, health scores reported by vendors, or QA/containment metrics improve anything? | ⛔ **Not verified** in any of the three cases; a documented *failure mode* exists for call-review regimes (compliance over resolution) — §7.5 |
| Do coaching (GROW-style), role-play and simulation improve performance here? | ⚠ **Convention.** Theory-adjacent (modelling, practice with feedback) and unverified in this role family |
| Is the "10% transfer" figure true? | ⛔ **Rejected** — folklore from a 1982 practitioner article |

**The one-sentence summary of §9, and it drives the worked example in §12:** ✅ *the evidence supports **practice with feedback, spaced over time, in an environment that rewards the taught behaviour** — and ⛔ the evidence does not support the proposition that buying a course, a certificate or a vendor's enablement platform will change what a team does.*

---

## 10. The AI-Era Effect, Evidence-Graded

**This section has one rule: every claim is labelled by source type, and the three source types are kept strictly apart.**

| Grade | Meaning |
|---|---|
| **DOCUMENTED** | A named organisation's own published claim, a court or tribunal decision, a regulatory filing, or an official statistic. Verifiable, and attributable to a party with something at stake. |
| **VENDOR / CONSULTANCY PROJECTION** | A prediction or a product claim by a party selling the capability, or a consultancy's forecast. Labelled as a prediction with its date. |
| **NOT VERIFIED** | Widely circulated, could not be verified in this pass. |

### 10.1 What is documented: the deployments

**Documented — the claim.** ✅ **On 27 February 2024, Klarna published a press release stating that an AI customer service assistant powered by OpenAI had handled about two-thirds of customer service chats in its first month and performed work equivalent to 700 full-time agents.** ✅ *(Source: Klarna's own press release, dated 27 February 2024, as cited in the reference work's account of the company.)* **This is a vendor's own claim about its own deployment — the correct evidentiary grade is DOCUMENTED AS A CLAIM, not verified as a measurement.** ⚠ Note what the claim does not say: it says nothing about resolution quality, customer outcome, or whether the two-thirds were the easy thirds. **A contact deflected is not a problem solved** (§10.2).

**Documented — the workforce consequence, and how it was characterised.** ✅ Later reporting records that **Reuters reported Klarna expected further headcount reductions as it used AI to handle customer queries**, and that ***The Guardian* reported in 2025 that Klarna's workforce had fallen from 5,527 in 2022 to 2,907, with Klarna attributing much of the reduction to natural attrition and the replacement of some tasks by technology.** ✅ *(same source)* **Read the attribution carefully — it is the company's own characterisation, and it deliberately spans three causes: attrition, AI substitution, and unspecified change.** ⚠ *(Separately verified and unrelated to AI: the handling of layoffs was criticised after the CEO published a list of affected employees on LinkedIn ✅. That belongs to §12-style leadership judgement, and it is noted here only because it appears in the same timeline.)*

**Documented — the adjudicated failure.** ✅ ***Moffatt v. Air Canada*, 2024 BCCRT 149**, a decision of the **Civil Resolution Tribunal of British Columbia**, decided **14 February 2024** (Tribunal Member Christopher C. Rivers), **held that a company can be held liable for misleading information given to customers by an AI chatbot**, finding **Air Canada liable for negligent misrepresentation arising from incorrect information given to a customer by its AI chatbot** ✅. The case involved **CA$880 (about US$704)** in dispute; it received international attention **because it was among the earliest decisions to hold a company liable for its AI agent's mistakes, and because of Air Canada's "remarkable" argument that its chatbot was a separate legal entity for whose actions it could not be held responsible.** ✅ *(Sources cited by the reference work include the American Bar Association's *Business Law Today* and *Business Insider* coverage describing "botshit" and AI making customer service worse ✅.)*

🔧 **Why this is the most valuable data point in the section, and the guide's own analysis:** it establishes, in an enforceable adjudication rather than an opinion, three things the marketing literature of the AI era elides. **(1)** The liability for an automated answer rests with the company, not the model and not the chatbot. **(2)** The failure mode of an automated answer is not "no answer" — it is a **confident wrong answer**, which is worse for the customer than the queue it replaced. **(3)** The vendor's answer to a quality problem — "ask for a human" — was defeated by the fact that the customer had already been given bad information and acted on it.

### 10.2 Deflection versus resolution — the distinction the market obscures

This is the single definitional point that determines whether an AI deployment is real or theatrical. 🔧

| | **Deflection** | **Resolution** |
|---|---|---|
| Definition | A contact that does not reach a human | A problem that is gone, verified independently of the channel |
| Measured by | Channel/containment statistics, produced by the vendor's own platform | Reopen rate, repeat contact, downstream volume, customer-verified outcome |
| Whose interest does the metric serve? | The vendor selling the platform, and the cost line | The customer |
| Failure mode | The problem returns, larger, having been "answered" | — |

**The market's "containment rate" / "deflection rate" claims are ⚠ vendor-published and definitionally inconsistent**: some count only chats not escalated, some count resolution without and reopen, some include sessions abandoned by the customer in frustration. ⛔ **This guide verified no vendor claim that could be compared with another vendor's claim**, which is itself the finding: **the metric is not standardised, so the numbers are not comparable, and a buyer cannot audit them.** Ask for reopen rate and repeat-contact rate at the customer level, in the customer's own system.

### 10.3 The effect on skill requirements, not just headcount

**What is documented:** the deployment claims and the workforce figures above ✅. **What is projection:** every "X% of customer service roles will be eliminated by 20XX" figure. ⛔ **None is printed here.**

🔧 **What the change looks like in the skill mix, stated as the guide's analysis:** if first-line, high-volume, known-question handling moves to automation, the residual human work is **the long tail and the hard cases** — which is exactly the work the §8.2 and §8.5 skills are for. Three consequences follow, and each is a design decision, not a forecast:

1. **The entry-level rung thins first.** The training pipeline for these roles has historically been *"join support, learn the product, get promoted"*. If the tier-1 work that formed that pipeline is automated, **the cheap apprenticeship disappears and the ramp cost for the remaining roles rises.** ✅ *The logic is straightforward and follows from §7.2's structural point that skill develops from practice with feedback; ⚠ the empirical claim that this is happening at scale is not verified here.*
2. **The measurement layer gets automated before the work does.** Vendor platforms now offer automated QA scoring of every interaction, AI-generated sentiment, and AI-summarised review notes (⚠ vendor category; verified only as existing product categories at the vendors named in §7.3, and as **2026 published content by COPC Inc. on "moving from AI hype to measurable business outcomes" and on building a contact-centre AI strategy before buying ✅**). 🔧 **This is the specific hazard this section must name: automating the measurement of a proxy multiplies the proxy's failure mode.** §6.5 established that satisfaction instruments decouple when the measured person controls the measurement; an AI scorer trained on the same scorecard industrialises that decoupling, at scale, with an audit trail that looks objective.
3. **The knowledge base becomes infrastructure.** §7.4's point, restated: the retrieval corpus for automation is the organisation's own documentation. **A stale knowledge base does not produce silence; it produces confident wrong answers to customers** — which is precisely the *Moffatt* fact pattern ✅.

### 10.4 The hazard of automating the coaching layer

🔧 *The guide's own construction, and the most actionable warning in this section:*

The sequence most organisations follow is **automate the customer contact → automate the quality scoring → automate the coaching insight (dashboards, "AI coach" prompts, auto-generated feedback)**. The first step is a cost decision. **The second and third are capability decisions made by people who are not the practitioners, using metrics the practitioners did not choose** — and §9.5's structural finding applies: the work environment is an input to whether learning transfers ✅. An environment where the feedback is machine-generated, generic, and derived from a compliance scorecard is an environment hostile to the judgement skills of §8.2 and §8.5, because those skills are developed by **case-based feedback from someone who knows the account.** 🔧

**The test to apply before automating coaching:** *does the automated feedback contain information the person could not have produced themselves?* If it restates their own transcript back to them as a score, it is compliance administration with a neural network, and it will consume the manager hours that would have produced the judgement.

### 10.5 Grade table: the AI claims

| Claim | Grade | Source / date |
|---|---|---|
| Klarna's AI assistant handled ~2/3 of customer service chats in its first month, work equivalent to 700 FTE agents | **DOCUMENTED as a vendor claim** ✅ | Klarna press release, 27 Feb 2024 |
| Klarna's workforce fell from 5,527 (2022) to 2,907 (attributed by the company to natural attrition and technology replacing some tasks); Reuters reported expected further reductions | **DOCUMENTED as reported** ✅ | *The Guardian* (2025) and Reuters, as cited in the reference work |
| A company is liable for negligent misrepresentation by its AI chatbot; the chatbot is not a separate legal entity | **DOCUMENTED — tribunal decision** ✅ | *Moffatt v. Air Canada*, 2024 BCCRT 149, Civil Resolution Tribunal of BC, 14 Feb 2024 |
| AI answer-quality failure produces confident wrong answers, worse than a queue | **DOCUMENTED as failure mode** ✅ | Same decision; ABA *Business Law Today* and *Business Insider* coverage |
| Vendors publish "containment"/deflection rate claims | **Not verified as comparable** ⛔ | Multiple vendor categories; no standard definition located |
| AI automating quality scoring, sentiment and coaching insight is a live product category | **PARTIALLY DOCUMENTED** ✅ vendor existence / ⚠ capability claims | Vendor content and COPC Inc.'s 2026 CX-AI content ✅ (COPC is itself a vendor of certification, training and consulting) |
| Official labour-market statistics quantifying AI's effect on customer-facing roles | **NOT VERIFIED** ⛔ | No official statistic verified in this pass — §16 |
| "74% of leaders have had to scrap deployed agents due to operational and governance gaps" | **DOCUMENTED as a vendor-published figure, methodology not disclosed on the page** ✅/⚠ | COPC Inc. executive guide, "Future-Proofing CX: Moving from AI Hype to Measurable Business Outcomes", page content dated Sept 2026 ✅. COPC sells certification and consulting, so treat the figure as marketing research |
| Consultancy predictions of autonomous resolution rates by a future year | **VENDOR/CONSULTANCY PROJECTION** ⚠ | Widely published; **specific figures unverified in this pass and not printed** |
| The entry-level apprenticeship pipeline for these roles is thinning | **GUIDE'S ANALYSIS (🔧), logic documented, empirical scale not verified** | This guide, §10.3 |

**The honest closing statement of the section:** the *documented* record consists of a vendor's own deployment claim, a company's own attribution of headcount change to a mixture of causes, and a small-claims tribunal finding that a company owns its chatbot's mistakes ✅. Everything else — the resolution percentages, the role-elimination counts, the autonomy timelines — is ⚠ projection by parties selling the technology, and **this guide prints none of those numbers, in either direction.** A practitioner's correct posture is neither enthusiasm nor denial: it is to insist on the customer-level measures in §10.2, and to notice that the automating organisation's biggest exposure is the knowledge base it stopped maintaining.

---

## 11. The Banking and Enterprise-Relationship Context

**Scope note, and it is a constraint, not a courtesy: this section describes the shape of enterprise post-sales work in regulated industries. It names no bank as a customer, and the worked example in §12 uses the repository's fictional persona only.**

### 11.1 What is different about the enterprise post-sale

The post-sales function in enterprise software was built for volume: many customers, standard product, self-service onboarding, pooled coverage. **Enterprise and regulated-industry customers invert nearly every parameter.** 🔧 *The comparison is the guide's construction, and it is the reason a post-sales model that works for mid-market software fails at enterprise scale.*

| Parameter | Volume business | Enterprise / regulated buyer |
|---|---|---|
| Number of customers | Hundreds to thousands | Tens, at most |
| Revenue concentration | Dispersed | A single account can be a material portion of the book |
| Buyer | A user or a functional manager | A committee: economic buyer, technical authority, risk, procurement, and often an independent control function |
| Contract | Standard terms, annual, self-serve | Negotiated, multi-year, with schedules, service levels, audit rights and exit provisions |
| Onboarding | Product-led | Project-based, with the buyer's own change and assurance processes |
| What "value" means | Usage and renewal | A defensible business case that will be examined internally, possibly by auditors |
| Who can end the relationship | The customer, by cancelling | The relationship can be terminated by a compliance, risk or audit finding that has nothing to do with the vendor's quality |

### 11.2 The multi-stakeholder account — and why "the customer" is a fiction

In an enterprise relationship the vendor's post-sales person serves **at least four different parties inside one account**, each with a different definition of success: 🔧

| Stakeholder | Their success criterion | What the vendor's role must do |
|---|---|---|
| **Economic buyer** | The business outcome against the case they approved internally | Evidence, in their own financial vocabulary, with a baseline |
| **Technical owner / architect** | The system behaves, integrates, and does not surprise them | Technical credibility; the named counterpart; no unannounced changes |
| **Operational users** | Their day got easier | Enablement, defect responsiveness, and a path to a human being |
| **Risk, compliance and procurement** | The vendor is not an exposure | Documentation, audit cooperation, contractual fidelity, incident transparency |

**The structural consequence:** ⚠ *practitioner consensus, and it is the defining difficulty of the enterprise post-sales role* — **a vendor can be technically excellent, on time, and well-liked by its users, and still lose the account because the fourth column disengaged.** Conversely, the loudest internal complaint at a customer is frequently not the one that decides the renewal. The role's hardest judgement (§8.5) is **deciding which of the four voices actually carries the decision this cycle**, and it changes without notice when a sponsor moves.

### 11.3 The governance cadence the enterprise buyer contracts for

This is where the enterprise relationship differs most from the volume model, and it is **documented as practice rather than as a standard**: ✅ service level agreements, service reviews, escalation matrices and named service owners are the ordinary contractual apparatus of enterprise technology supply, and the standardisation of the *operation* behind them is what **ISO 18295-1:2017** (requirements for customer contact centres) and **ISO 18295-2:2017** (requirements for clients using the services of customer contact centres) address ✅ *(verified as published standards for 2017; current editions not confirmed in this pass ⚠)*.

**The practical implication:** 🔧 *where the cadence is contractual, the vendor's post-sales role has a licenced reason to convene the customer on a schedule, at a level, with the right attendees.* That is worth more than any playbook, because it means the relationship does not depend on the customer's goodwill for its rhythm — and it means a vendor's failure to hold a review is a **contractual** lapse rather than a service-quality preference.

**Cross-references, by name:** the service-management mechanics (incident, problem, change, service level) belong to [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md). The **buy side** of this relationship — how an enterprise selects, contracts, governs, escalates and exits a supplier — is owned by [vendor_management_guide.md](vendor_management_guide.md); this guide deliberately does not re-derive it, and the reader who wants the mirror image should read that guide rather than a second-hand summary here. The repository's banking-domain context is in [../banking/](../banking/), with the regulatory material in [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md), data and risk aggregation in [../banking/risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md), and the AI/compliance intersection in [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md).

### 11.4 The customer-facing role as a partial compliance role

This is the part practitioners in regulated sales environments most often underestimate, and it is worth stating exactly: ⚠ *the following is the guide's synthesis of standard practice in regulated technology supply; it is not a statement of any specific regulator's rule.*

In a regulated buyer's environment, the vendor's customer-facing person acquires **obligations that are not commercial**:

1. **They are a documented control or a documented risk.** Vendor personnel with access to the customer's systems and data appear in the customer's third-party risk register, and changes in the vendor's personnel are changes to the customer's control environment. The vendor's staffing decision is therefore a compliance event for the customer. ⚠
2. **Their statements become evidence.** A commitment made in a review, a remediation timeline given on a call, is potentially discoverable material for the customer's own regulator. The role's §8.4 writing skill is, in this environment, a records-management obligation.
3. **They must be able to say no for compliance reasons, in public, without losing the relationship.** Refusing a customer's request because it is not permitted is not an awkward moment; it is the job — and it is precisely the behaviour a commission-driven role is structurally unable to perform. 🔧
4. **They will be asked to do things that the buyer's own rules forbid the buyer's employees from doing.** The judgement is knowing which of those are trivially accommodated and which are the kind of accommodation that later appears in a finding.

🔧 **The design consequence, and it is the guide's own:** in a regulated account, the accountability/authority gap of §3.4 is **strictly wider**, because some of the vetos are held by parties the vendor's role has no relationship with at all — the customer's auditor, the customer's regulator, the vendor's own compliance function, and the contract. **A post-sales function operating in enterprise regulated accounts needs, as a minimum, a named internal escalation route into legal, security and compliance, and a stated policy on what its customer-facing staff are not authorised to promise.** Without those two artefacts, the role is exposed to making commitments it cannot honour and cannot control.

### 11.5 The specific difficulty of running post-sales in a regulated buyer's environment

⚠ *Practitioner-consensus list, offered as description rather than as finding:*

- **The procurement cycle is longer than the risk signal.** A problem identified today may take a quarter to become a contractual issue and a year to affect the relationship — so the vendor's earliest signal is a relationship one (the sponsor's attention, the review's tone), not a contractual one. 🔧
- **Change control is real.** Improvement requires change requests, testing windows, and sometimes the buyer's own release governance. "Fix it next sprint" does not exist in this environment, and this is the single most common source of vendor-side frustration. ⚠
- **Concentration risk is the vendor's too.** With tens of customers, each loss is material — which is why the enterprise post-sales function is usually better staffed than the arithmetic of the volume model would suggest, and why the retention metric (§5.2) is *the* board-level number.
- **The relationship survives individuals, and so it must be institutionalised.** Enterprise accounts outlast sponsors, account managers and reorganisations. The artefacts that make that possible — documented decision history, a maintained stakeholder map, reviews that happen whether or not anyone is present — are unglamorous and are the actual work. 🔧
- **Resilience and operational-continuity expectations are contractual, not aspirational.** The repo's treatment of that discipline is in [resilience_engineering_guide.md](resilience_engineering_guide.md), and in enterprise accounts it is a **customer-facing commitment**, not an internal engineering posture: the vendor who cannot evidence it will be asked to, in writing, in an audit.

---

## 12. The Cymbal Bank Worked Example: A Post-Sales Function That Scores Well and Retains Badly

> **Everything in this section is explicitly illustrative and fictional.** Cymbal Bank is this repository's fictional banking persona. The figures, names, account counts, costs and timelines **are constructed for teaching purposes and are not benchmarks, not derived from any real organisation, and not measurement of anything.** No real institution appears as a customer anywhere in this guide. The *patterns* are drawn from the structural analysis in §3–§9 and are labelled there; the *numbers* are pedagogical. 🔧

### 12.1 The situation

**Cymbal Supplies (a fictional division of Cymbal Bank) sells a payments-adjacent platform to enterprise customers.** The commercial picture at the start of the exercise, as reported internally:

| Signal | Reading at the time | What it actually meant |
|---|---|---|
| Satisfaction score: stable at a historically high level, slightly up year on year | "Customers are happy" | The instrument was measuring a subset at a flattering moment (§12.2) |
| Net revenue retention: 104% | "We're growing the base slightly" | ⚠ Definitionally true, and **hiding** that expansion was concentrated in two accounts |
| Gross revenue retention: 84% | Reported to leadership as a footnote | **This was the real number, and it was deteriorating** |
| Logo retention: falling on mid-market accounts | Attributed to "market conditions" | Attributed correctly by accident — the mechanism was the handover and the coverage model |
| Voluntary attrition in the post-sales team: rising | Attributed to compensation | The team was right about compensation being the *visible* cause and leadership was wrong about it being the actual one: the responsibility/authority gap (§3.4) was |
| Support reopen rate | Not reported | The missing instrument (§6.7 clause 2) |

**The presenting problem is the divergence:** *a function scoring well on satisfaction while retention declines.* 🔧 **The premise of this example is that this is not a paradox — it is the predictable output of measuring one thing and paying for another (§6.5).**

### 12.2 The diagnosis

**Step 1 — interrogate the satisfaction instrument before the team.** 🔧

| Question asked | Finding |
|---|---|
| Who is surveyed? | The named day-to-day contacts — users and coordinators |
| Who is *not* surveyed? | The economic buyer, the technical owner, and risk/procurement — i.e. **three of the four stakeholder voices in §11.2** |
| When is the survey sent? | By the account owner, after a resolved issue or a good review |
| What does the score correlate with? | The recency of the last successful interaction. **It was measuring relief, not value** |
| What decision has the score ever changed? | None that could be named |

**Finding:** a **relationship-mood indicator for the population the vendor spoke to most often**, administered by the party with an interest in the result. Not wrong — answering a different question from the one leadership thought it was asking. ✅ *The mechanism is exactly the one §6.5 describes and §6.1.1 documents in Reichheld's own advice about score-linked compensation.*

**Step 2 — the lifecycle review (§4.1), stage by stage.** Loss was concentrated in **a single joint between two stages**, with secondary failures:

| Stage | Finding | Structural cause |
|---|---|---|
| Handover | In most churned accounts, the successive post-sales owner had not known about **a promised integration that was not on the roadmap** | §4.2 causes 1 (payment timing) and 2 (the promise was never an artefact) |
| Implementation | Onboarding over-runs absorbed silently, with no commercial vehicle | Scope discovered in flight (§4.3) |
| Adoption / value | Adoption measured by logins, with no distinction between *used* and *useful*; the business review presented vendor telemetry to a customer whose finance function had never seen a baseline | §4.4's baseline test, failed |
| Renewal / churn | Risk became visible only on the renewal-date trigger — the survey had flattered the function into blindness; **no loss reviews existed at all** | §4.5, §4.1 stage 8 |
| Expansion | Sold into two mid-market accounts that were themselves at risk; both contracted the following year | §4.5 / §14 |

**Step 3 — why the handover failed, structurally, not personally.** 🔧 The review's conclusion, and why it did not blame the sales team: **the handover had no compensable moment and no acceptance test.** A post-sales team that must accept every account has no leverage over what it accepts (§4.2 cause 4). The remediation therefore had to change the *incentive and the acceptance right* — which is why the first proposal, a new handover document, was rejected (§14 anti-pattern 4).

**Step 4 — the coverage model.** The same named-CSM model was applied to accounts the function could not afford to serve and accounts that did not need it. The low-revenue, high-volume tier consumed disproportionate time; the two large accounts that were expanding got the same cadence as everyone else. §5.3's segmentation had never been written down, so nobody was accountable for overriding it.

**The diagnosis in one sentence:** *the function was measuring a mood, staffing by headcount, transferring a relationship without its context, and reporting the one retention ratio that could not show the problem.*

### 12.3 The remediation design

**A — Staffing and coverage.** 1. **Write the segmentation down** with a named owner per tier and an explicit statement of what each tier does *not* get (§5.3) — including the decision that low-revenue accounts get no named human, softened by a two-year sunset so no *existing* relationship was withdrawn. 2. **Create the acceptance right** (§4.2 cause 4): the post-sales lead may refuse a transfer that fails the §4.2 test. The first version failed — nobody used it, because refusing costs a relationship with the seller and gains nothing personally. The fix was to make acceptance *count*: a transferred account that churns within two quarters is attributed to the transfer record, in writing, once. 3. **Fund the seam**: one person accountable for handover quality across both functions — an owner with a metric, not a coordinator. 4. **Compute cost-to-serve** for the top tier, once, honestly, including executive time and engineering escalations; the number restarted the pricing conversation.

**B — Process.** 1. **Loss reviews as a standing artefact**, asking one question — at which stage did this account's outcome become unrecoverable? — reported monthly to a named executive outside the function. 2. **The baseline rule**: no value claim enters a business review without a baseline agreed in writing before the outcome and not relying solely on vendor telemetry. 3. **The renewal calendar moved twelve months earlier**, with written exit criteria at T−12, T−9, T−6, T−3. 4. **Reopen rate and repeat-contact rate added to the reporting pack** — the two measures the team had been avoiding.

**C — Enablement.** §12.4.

### 12.4 The training programme

**Designed against §9's findings, not against a vendor catalogue:**

| §9 finding | Design rule adopted |
|---|---|
| ✅ Spacing beats massing (Cepeda et al., 259/271) | **No single workshop** — every module is a spaced sequence, revisited and applied to live accounts |
| ✅ The work environment is an input to transfer (Baldwin & Ford) | **The manager's cadence and the acceptance right are part of the programme**, not preconditions assumed away |
| ✅ Deliberate practice explains <1% of variance in professions | **Buy feedback loops, not hours** — cost weighted toward case review and live coaching, not content production |
| ✅ Its power depends on stable rules and immediate feedback | **A case library built from Cymbal's own churned accounts**, with the outcome hidden until the learner commits |
| ✅ Cognitive apprenticeship: modelling with narrated reasoning, then coached practice, then fading | **Shadowing restructured into §7.8's five phases**, with autonomy released on a schedule, not a date |
| ⚠ Kirkpatrick levels 3–4 are hard to implement (documented) | **Evaluated at the customer level**, not reaction level — §12.5 |
| ⛔ The "10% transfer" figure and decay percentages | **Not used.** The business case cites neither (§9.7) |

**The curriculum, in four parts** 🔧: **(1) Diagnostic reasoning** (§8.2) — six spaced sessions on Cymbal's own escalations, outcomes concealed, assessed by the case's original owner; pass criterion is that the recommendation would have changed the outcome or the same-outcome reasoning is defensible against the original. **(2) The hard conversations** (§8.3) — four simulations on the conversations the team actually has (the defect, the delay, the price increase, the withdrawn feature), assessed on whether the learner could state a **truthful** position, not on whether the role-played customer was pleased; the craft is owned by [conflict_management_guide.md](conflict_management_guide.md) and [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) and is not re-taught here. **(3) Commercial judgement** (§8.5) — the segmentation model, the cost-to-serve arithmetic, and four decision cases in which the correct answer is to *deprioritise* an account, assessed as a written decision with reasoning. **(4) Influence without authority** (§8.6) — every escalation the learner writes is reviewed for evidence, specificity and ask by someone senior in the receiving function; cross-referenced to [authority_skills_guide.md](authority_skills_guide.md) and [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md).

**Ramp assumptions, stated as assumptions** ⚠🔧 *(illustrative; the guide verified no benchmark — §7.2, §16)*: a starting team of 14 across three tiers; a 16-week spaced programme at ~4 hours per person per week; competence on diagnostic reasoning demonstrated on 5–7 supervised cases **rather than after a calendar period**; ~1.5 hours of mentoring per learner per week; and an in-year cost that was — in the fictional ledger — dominated not by content but by **the opportunity cost of the mentors**. **Expected effect: deliberately not stated as a percentage.** The team committed to a measurement design, not a target. 🔧

🔧 *The one design decision worth copying:* **the programme's biggest cost was senior people's time, and its biggest output was the case library** — an asset that survives the programme, is the basis for every subsequent hire's ramp, and is also the retrieval corpus if the organisation later automates (§10.3). **A training programme that produces only trained people has produced nothing durable.**

### 12.5 The measurement they used instead of the proxy

| Old measure | Replacement | Why |
|---|---|---|
| Satisfaction score as the functional KPI | Satisfaction retained as a **diagnostic input only**, with no bonus attachment | §6.7 clause 3; Reichheld's own advice ✅ |
| NRR alone | **Logo retention + GRR + NRR reported together**, same cohort and period, monthly | §5.2 discipline 1 |
| NRR at company level | **Cohort table by acquisition year and tier**, so one expander cannot mask a shrinking core | §5.2 discipline 2 |
| Nothing | **Reopen rate and repeat-contact rate** added to the pack | §6.3 |
| Nothing | **Sponsor-attrition rate** — was the named economic buyer still in post and engaged at T−6? | §4.5 |
| Nothing | **Handover quality**: transfers passing the §4.2 three-question test, plus two-quarter churn on transferred accounts | §12.3 |
| "Training completed" | **Customer-level evidence**: reopened tickets that no longer reopen; renewals that closed without an escalation; adoption movement on the learner's accounts | §9.6 evaluation rule |

### 12.6 The recommendation — including what they chose *not* to fix

**Recommended, in sequence:** fix the handover incentive and the acceptance right; write the segmentation down and fund the seam role; move the renewal calendar twelve months earlier; build the case library and run the spaced programme; replace the measurement pack **before** the programme starts, not after.

**Chosen NOT to fix, with reasons** 🔧 *— and this part matters more than the plan:*

| Deliberately not fixed | Reason |
|---|---|
| **The product gaps that caused the loudest escalations** | Outside the function's authority, and already known to product. The function's job was to *evidence* them — which requires the escalation-quality review, not an escalation-volume increase (§7.5) |
| **The customer-success platform** | Rejected in favour of a reporting discipline. It would have produced a health score nobody had validated (§6.4) while the actual gap was a definitional one. Buying software to fix a definitional problem is this field's most common error |
| **A new handover document template** | A document does not change a payment timing (§4.2 cause 1) or create an acceptance right (cause 4) |
| **Satisfaction-linked bonuses** | Removed rather than improved: improving the survey would have made the proxy more precisely measured and equally misleading |
| **Resilience training for the team** | Deferred — **not** because resilience is unimportant, but because §8.7 applies: the load's causes were structural, and teaching people to tolerate the handover seam would have entrenched it |
| **Removing named coverage from existing low-revenue accounts** | Declined for two years, to avoid the trust cost of withdrawing a relationship sold as a commitment |

**The one-line recommendation:** *stop asking whether customers are happy and start asking which stage of the lifecycle has become unrecoverable — then measure the two retention ratios that cannot flatter you, in the same table, every month.*

---

## 13. The Career Angle

**Cross-references first, because this repository already owns the mechanics and this guide will not re-derive them:** how a professional becomes discoverable and how to position the evidence of this work → **[reverse_job_search_guide.md](reverse_job_search_guide.md)**. What to build in the first weeks of a role before the accumulated load arrives → **[the_first_90_days_guide.md](the_first_90_days_guide.md)**. Both are named rather than summarised.

### 13.1 The routes in

⚠ *Practitioner-pattern description; no labour-market data was verified in this pass (§16).*

| Route | What it gives you | What it costs | Where it leads |
|---|---|---|---|
| **Support → post-sales (the classic)** | Product depth, pattern recognition, the customer's actual language | The most intense emotional-load and compliance-scored environment first | The strongest route *if* the entry tier survives automation; the rung most at risk if it does not (§10.3) |
| **Implementation / professional services → post-sales** | Project discipline, scoping judgement, delivery reality | A billable-utilisation culture that undervalues relationship work | Natural, and best-prepared for enterprise accounts |
| **Sales → post-sales** | Commercial instinct and the ability to hold a number | The compensation drop is usually the barrier | The clearest path to the top of §2.3's ladder, because that ladder is a *number* ladder |
| **Engineering → customer-facing (incl. the FDE route)** | Technical credibility that cannot be faked | Culture shock; engineering promotion structures treat the move as a detour | Deep technical account roles; see [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) |
| **Operations / service management → post-sales** | Process discipline, service-level literacy, the governance vocabulary buyers use | Can read as process-heavy to product-led employers | Enterprise accounts and service-management leadership |
| **Graduate entry** | Product and process fluency fast | The hardest ramp — §8.2/§8.5 judgement has no shortcut and the case library is thin | Viable, and the reason the case library is an employer's most valuable asset (§12.4) |

### 13.2 The accreditation landscape, and what the certifications are worth

| Credential type | What it actually is | Worth in hiring | Verified? |
|---|---|---|---|
| **Methodology certifications owned by non-profits/consortia** (KCS, via the **Consortium for Service Innovation** — a non-profit alliance whose members include Oracle, SAP, Autodesk, F5, DTCC, PTC, Verint, NICE CXone, Coveo, Vertex, Ping Identity and Tyler Technologies ✅) | A structured method tied to the daily work (article-based problem solving) | **The strongest of the three**, because it certifies a method the person actually operates | ✅ ownership and structure; ⛔ outcome value unverified |
| **Benchmarking/audit certifications** (e.g. COPC Inc.'s ✅) | An **organisational** audit against a firm's own standard | Not a personal credential; useful to the employer, not the CV's owner | ✅ existence; ⛔ market recognition unverified |
| **Vendor product certifications** | A test about one product | A hiring filter at partner firms and for stack changes | ⛔ **Value unverified** |
| **CX professional body credentials** (CXPA/CCXP commonly cited) | A credential for the customer-experience discipline | ⛔ **Unverified** — ownership, launch date, competency framework and recognition are all unasserted (§16) | ⛔ |

🔧 **The honest sentence:** *a certification is evidence that you were exposed to a method; the evidence that you can do the work is a documented record of accounts you held and problems you resolved, which no credential substitutes for — and in this field, unlike law, medicine or accounting, no credential is a licence.*

### 13.3 Which roles here are durable, and which are a revolving door

🔧 *The guide's assessment, resting on §3, §5 and §10. Offered as analysis, not as a labour-market finding.*

| Role | Durable? | The structural reason |
|---|---|---|
| **Enterprise / strategic account ownership** | **Durable, increasingly so** | Low account counts, high concentration, multi-stakeholder judgement, and §11.4's compliance dimension make it hard to automate and hard to outsource. The judgement is not in the transcript |
| **Technical account management in complex estates** | **Durable** | The work is diagnosis in a specific environment, and §9.4's practice logic applies to the domain knowledge it accumulates |
| **Post-sales operations / enablement** | **Durable but structurally junior** | Necessary, invisible, and first to be cut, because its output is a capability rather than a number |
| **Volume customer success (pooled, mid-market)** | **Vulnerable** | The most automatable segment: standard product, high ratio, playbook-driven. §10.3's thinning starts here |
| **Tier-1 support as a career** | **Already a revolving door, closing further** | Historically a year-long apprenticeship; now the layer most exposed to deflection — and §10.2 explains why the metric will look good while the outcome does not |
| **Renewal / quota-carrying post-sales** | **Durable, and the ladder's real trunk** | It owns a number. §2.3's ceiling analysis is unambiguous: progression runs through owning an outcome, not through service excellence |

**The advice this implies** 🔧 *— the same point as §8's decomposition from the other side:* **acquire the two skills nobody trains — diagnostic judgement and commercial judgement — and get them evidenced in writing.** They are (i) the hardest to automate, (ii) portable between employers because they are demonstrated by records rather than titles, and (iii) the only route from §2.3's middle rungs to its top. The title is not the asset. The case record is.

---

## 14. The Anti-Patterns

Seven named patterns, each **symptom → cause → guardrail**. 🔧 *All seven are the guide's construction; the causes are the structural mechanisms established in the sections referenced. Offered as diagnostic checks, not as researched findings.*

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **Hiring for product knowledge and training for empathy, when the gap is the reverse** | New hires pass product certification and fail their first renewal conversation; the team can explain the API and cannot tell a customer that a promised feature is not coming | Training investment follows the cheap-to-teach, easy-to-test axis (§8's opening); §8.2/§8.5 judgement is neither | **Hire for the case record, train the product** — interview with a documented escalation and a lost deal, not a product quiz. Corollary: empathy is also not trainable in a workshop (§9.4: practice, feedback, environment) |
| **2** | **The satisfaction score improves while retention declines** | The metric everyone reports goes up; the number the board cares about goes down | Different behaviours produce them and only one is measured (§6.5); the instrument usually surveys a subset at a favourable moment (§6.1.1, §12.2) | Measure the pair (§6.7 clause 2). Never report a satisfaction figure without the corresponding hard metric beside it |
| **3** | **The call-review regime that rewards script compliance** | QA scores rise; resolution quality does not; experienced staff call the review "acting lessons" | A scorecard measures conformance to the scorecard (§7.5); it is also supervisory control over emotional display, which the literature links to surface acting and strain ✅ | Review the calls the **representative chose**; weight the scorecard toward outcome; **remove the bonus attachment**; require the review to produce one named behavioural change |
| **4** | **The handover that is a document rather than a relationship transfer** | A thorough handover template exists; churn still concentrates in the first two quarters of ownership | Documentation changes no payment timing, transfers no tacit context, creates no acceptance right (§4.2 causes 1, 2, 4) | The three-question test (§4.2) plus the acceptance right and the transfer-attributed churn record (§12.3) |
| **5** | **The customer-success function funded as a cost centre and measured as a survey** | The function reports a score and cannot say no to anything; it is the first line cut in a downturn | Its budget is justified by losses that did not happen (§5.1) and its only visible output is a survey number | Attach the function to the retention arithmetic it defends, and report that arithmetic. A cost centre cannot out-argue a quota; a function with a retention number can |
| **6** | **Training delivered without reinforcement and evaluated by a smile sheet** | High satisfaction with the training, no behaviour change, the same problems twelve months later | Spacing beats massing ✅, the work environment is an input to transfer ✅, the higher evaluation levels are hard to implement (documented ✅), and level 1 is free and flattering (§9.6) | Replace the event with a spaced sequence; evaluate at the customer level (§9.6's rule); refuse the smile sheet as evidence. If the environment punishes the taught behaviour, fix the environment or do not run the programme |
| **7** | **The account model applied uniformly to accounts that do not warrant it** | Named coverage everywhere; the low-revenue tier consumes a disproportionate share of hours; large accounts get the same cadence as small ones | Segmentation logic is implicit, so nobody is accountable for overriding it, and cost-to-serve is never computed per customer (§5.3) | Write the segmentation down with a named owner and an explicit statement of what each tier does **not** get. Compute cost-to-serve once for the top tier, honestly, including senior time |

---

## 15. The Claims Audit

**How to read this table.** *Status* is **Verified** (checked this pass against a named source), **Flagged** (widely repeated but not verified, or an estimate whose provenance is stated), **Rejected** (could not be verified, or the sources conflict), or **Constructed** (this guide's own analysis or template, offered as such). *Source quality* is one of **tier-1 primary** (the organisation making the claim about itself, a standard, a regulatory filing, or a tribunal decision), **peer-reviewed**, **reference work**, **industry platform / vendor content**, **practitioner-secondary**, or **the guide's own construction**. The high-risk classes this task requires to appear in full — **metric-validity findings, framework attributions, training-efficacy claims, market figures and AI claims** — are marked in the *Class* column.

### 15.1 The metric-validity findings

| # | Claim | Class | Status | Source, date, quality |
|---|---|---|---|---|
| 1 | NPS was developed by **Bain & Company consultant Fred Reichheld** and popularised in his **December 2003 *HBR*** article "The One Number You Need to Grow" (*HBR* 81(12):46–54) | Metric origin | ✅ Verified | Reference work's NPS article, citing the *HBR* record and PMID 14712543; **reference work** |
| 2 | NPS arithmetic: promoters 9–10, passives 7–8, detractors ≤6; score = %promoters − %detractors | Metric definition | ✅ Verified | Same; **reference work** |
| 3 | **Reichheld owns the registered NPS trademark in conjunction with Bain & Company and Satmetrix** | Metric ownership | ✅ Verified | Same; **reference work** |
| 4 | Reichheld's claimed basis: the recommend question was the **best predictor of return business and word of mouth** versus satisfaction/loyalty/intention questions | Metric claimed basis | ✅ Verified **as the claim made** | Same; **reference work** |
| 5 | The claim that NPS **correlates with revenue growth** carries a **"failed verification"** marker in the reference work | Metric validity | ⚠ Flagged | Same; **reference work** |
| 6 | **Peer-reviewed challenge:** Keiningham, Cooil, Andreassen & Aksoy, "A Longitudinal Examination of Net Promoter and Firm Revenue Growth," ***Journal of Marketing* 71(3):39–51, July 2007** | Metric validity | ✅ Verified (paper and venue); ⚠ finding carried at abstract/citation level | SAGE/JSTOR records; **peer-reviewed** |
| 7 | Scholarly critique: **no empirical evidence** that "likelihood to recommend" is a better predictor of growth than other loyalty questions, and it **does not measure anything different** from conventional loyalty questions | Metric validity | ✅ Verified as the documented state of the critique | Reference work's NPS criticism section, citing Keiningham et al. and Hayes, *Quality Progress* 41(6), 2008; **reference work + peer-reviewed** |
| 8 | NPS **offers no detailed insight into specific issues**; it **over-weights extremes and devalues passive respondents** | Metric limitation | ✅ Verified | Same; **reference work** |
| 9 | **NPS is used by roughly two-thirds of Fortune 1000 companies as of 2020**; the specific popularity claim attaches to a *Fortune* long-form piece (Colvin, 18 May 2020) | Metric adoption | ⚠ Flagged | Reference work citing *Fortune*, 2020; **industry press** |
| 10 | **Reichheld himself** called over-use a **"tragedy of the commons"**, recommended tracking repeat customers and referrals by other means, and **recommended de-linking NPS from employee compensation** | Metric critique by originator | ✅ Verified | Reporting from WBUR *On Point*, 17 January 2025, as cited in the reference work; **industry press (tier-1 reported)** |
| 11 | *The Ultimate Question 2.0* (Reichheld & Markey, Harvard Business Review Press, 2011) exists | Framework | ✅ Verified | Reference work's citation; **reference work** |
| 12 | The later **"earned growth rate"** revision, attributed to Reichheld's later work | Metric origin | ⛔ **Rejected / not verified** | No source located this pass; see §16 |
| 13 | **ACSI began in 1994**, developed by the **National Quality Research Center at the University of Michigan** with the **American Society for Quality and CFI Group**; the model was **first designed in 1989 for Sweden** (SCSB); developed by **Claes Fornell** | Metric origin | ✅ Verified | Reference work's ACSI article citing *Leadership and Management in Engineering*; **reference work** |
| 14 | ACSI is a **structural-equation model** over three manifest variables (overall satisfaction, expectancy disconfirmation, comparison to ideal), each 1–10, weighted and rescaled to 0–100; ~**350,000 interviews annually** | Metric methodology | ✅ Verified | Same; **reference work** |
| 15 | ACSI is **commercially operated** (ACSI LLC; founder is chairman of CFI Group; international licensees) | Source quality | ⚠ Flagged | Same; **reference work** — the caveat is this guide's reading |
| 16 | **CES** was introduced in "Stop Trying to Delight Your Customers," ***HBR* July–August 2010, by Matthew Dixon, Karen Freeman and Nicholas Toman** (CEB research lineage), which **asserted CES is a better predictor of loyalty than satisfaction measures or NPS**, and offered the Customer Effort Audit | Metric origin + claim | ✅ Verified (article and claim); ⚠ the superiority claim is the originators' | *HBR*/HBSP product record; **industry press / practitioner journal** |
| 17 | CES is **proprietary, not peer-reviewed**, and has been **revised** since introduction, so "CES" is not one question | Metric standing | ⚠ Flagged | Vendor explainer content (a "new and improved" CES attributed to Gartner); **vendor content** |
| 18 | No independent validation of CES's superiority was located | Metric validity | ⛔ Rejected / not verified | None located; §16 |
| 19 | **CSAT has no single originator**; it is a family of satisfaction items in the expectancy-disconfirmation lineage | Metric origin | ⚠ Flagged | The guide's characterisation; **practitioner consensus / constructed** |
| 20 | **ISO customer-satisfaction standards exist and are guidance, not validations**: ISO 10001:2007 (codes of conduct), **ISO 10002:2014** (complaints handling), ISO 10003:2007 (external dispute resolution), **ISO 10004:2008** (monitoring and measuring) | Standard attribution | ✅ Verified (as listed); ⚠ current editions unconfirmed | Reference work's ISO-standards list; **reference work**, flagged for edition currency |
| 21 | **ISO 18295-1:2017** (requirements for customer contact centres) and **ISO 18295-2:2017** (requirements for clients using their services) are published standards | Standard attribution | ✅ Verified (as listed); ⚠ edition currency | Same; **reference work** |

### 15.2 Framework attributions

| # | Claim | Class | Status | Source, date, quality |
|---|---|---|---|---|
| 22 | **ITIL** was developed by the UK government's **CCTA in the 1980s**, released in **1989**, V2 in 2001, V3 in May 2007, **ITIL 4 in February 2019**; **AXELOS** created 2013 (Capita + UK Cabinet Office); **PeopleCert acquired AXELOS in June 2021** | Framework attribution | ✅ Verified | Reference work's ITIL article; **reference work** |
| 23 | **There is no formal independent third-party compliance assessment for ITIL, and certification is available to individuals only, not organisations** | Framework standing | ✅ Verified | Same; **reference work** |
| 24 | **Customer success** is documented as a B2B subscription-business marketing strategy and function whose purpose is ensuring customers achieve desired outcomes and reducing churn, **built on services-marketing and relationship-marketing literature** | Framework origin | ✅ Verified as described | Reference work's "customer success" article and its citations; **reference work** |
| 25 | **"Companies such as Salesforce developed customer success strategies in the mid-2000s to increase revenue by improving customer retention"** | Framework origin — **conflicting** | ⚠ Flagged (conflict with #26) | Same; **reference work**. No primary company disclosure obtained |
| 26 | An industry account holds that the **first named "Customer Success" group was created at Vantive, a CRM company, in 1996–1997**, by **Marie Alexander** at CEO **John Luongo**'s invitation — with the explicit hint **"It wasn't Salesforce"** | Framework origin — **conflicting** | ⚠ Flagged | **Customer Success Association (Mikael Blaisdell), "The History of Customer Success – Part 1"**; **industry-secondary / self-described association, not independently verified** |
| 27 | The same body defines Customer Success as "**a long-term, scientifically engineered, and professionally directed business strategy for maximizing customer and company sustainable proven profitability**" | Framework definition | ✅ Verified as the body's definition; ⚠ the "scientifically engineered" phrasing is its own | Same association; **industry-secondary** |
| 28 | **Peer-reviewed research on customer success management exists in B2B marketing journals** — *Hochstein, Voorhees, Pratt, Rangarajan, Nagel & Mehrotra, "Customer success management, customer health, and retention in B2B industries"* (cited as **December 2023**) and *Eggert, Ulaga & Gehring, "Managing Customer Success in Business Markets"* (**Journal of Service Management Research**, 2020) | Evidence base | ✅ Verified as published works | Reference work's citation trail; **peer-reviewed** |
| 29 | **GROW** was developed in the **UK** and used extensively in corporate coaching **from the late 1980s and 1990s**; **there are several versions**; stages = **Goal, Reality, Obstacles/Options, Way Forward** | Framework attribution | ✅ Verified for origin country/period and stage structure | Reference work's GROW article; **reference work** |
| 30 | The specific original authorship of GROW (commonly attributed to **Graham Alexander, Alan Fine and John Whitmore**, with Whitmore's *Coaching for Performance* (1992) as the popularising text) | Framework attribution | ⚠ **Flagged — not verified this pass** | Practitioner attribution only; §16 |
| 31 | **Cognitive apprenticeship** derives from constructivist and situated-cognition work; **skills taught independently of real-world context are less effective** (citing Collins, Duguid & Brown, 1989); the theory names modelling, coaching and the staged release of autonomy | Framework attribution | ✅ Verified | Reference work's cognitive-apprenticeship article; **reference work** |
| 32 | **The transfer-of-training canonical model is Baldwin & Ford (1988)** — generalisation and maintenance of training to the work environment; inputs (training design, **work environment**, trainee characteristics) → outputs (learning and retention) → conditions of transfer | Framework attribution | ✅ Verified | Reference work's transfer-of-training article; **reference work** |
| 33 | **Three transfer outcomes exist: positive, negative, zero** | Framework | ✅ Verified | Same; **reference work** |
| 34 | **The four-level training-evaluation model** originates with **Donald Kirkpatrick's articles in the late 1950s, from his doctoral dissertation**, and became one of the field's most influential models | Framework attribution | ✅ Verified | Abstract of Reio et al., *New Horizons in Adult Education and Human Resource Development* (Wiley/SAGE record); **peer-reviewed** |
| 35 | **Kirkpatrick's model has been challenged and criticized by scholars**, and using all four levels is documented as **useful but difficult**; many later models are variations of it; the thirty-year retrospective is **Alliger & Janak (1989)** | Training efficacy | ✅ Verified | Same, plus the Alliger & Janak record; **peer-reviewed** |
| 36 | The specific critique that the four levels form **no validated causal chain** | Training efficacy | ⚠ Flagged | Widely repeated; **provenance not verified this pass** — not attributed to a named critic in this guide |
| 37 | The **ROI "fifth level"** attributed to **Jack Phillips** | Training efficacy | ⛔ **Rejected / not verified** | Reference work has no article; no source located; §16 |
| 38 | **KCS (Knowledge-Centered Service / "Knowledge-Centered Success")** is owned by the **Consortium for Service Innovation**, **"a non-profit alliance of service and support organizations"**, with named member companies including Oracle, SAP, Autodesk, F5, DTCC, PTC, Verint, NICE CXone, Coveo, Vertex, Ping Identity and Tyler Technologies; the Consortium also **sells KCS courses and exams** | Methodology attribution | ✅ Verified | serviceinnovation.org (the Consortium's own site); **tier-1 primary for itself** |
| 39 | **COPC Inc.** provides contact-centre/CX **consulting, certification, research, training and managed services**, and publishes benchmarking dashboards | Standards/benchmark body | ✅ Verified | copc.com; **tier-1 primary for itself** |

### 15.3 Training-efficacy claims

| # | Claim | Class | Status | Source, date, quality |
|---|---|---|---|---|
| 40 | **Ebbinghaus' forgetting curve** derives from experiments run **1880–1885 on himself**, published 1885 as *Über das Gedächtnis*; the popular representation is of memory roughly halving in days/weeks without review; the phenomenon is **transience** | Training efficacy — decay | ✅ Verified | Reference work's forgetting-curve article; **reference work** |
| 41 | **Murre & Dros, "Replication and Analysis of Ebbinghaus' Forgetting Curve," *PLOS ONE*, 2015** (DOI 10.1371/journal.pone.0120644): successful replication using the **method of savings**, **one subject, 70 hours**, intervals **20 min / 1 h / 9 h / 1 d / 2 d / 31 d**, results similar to the original | Training efficacy — decay | ✅ Verified | *PLOS ONE* article record and abstract; **peer-reviewed** |
| 42 | **Applying the forgetting curve's percentages to corporate training is an extrapolation, not a finding**; the specific workplace decay figures circulating in training content are not established by these studies | Training efficacy | ⚠ **Flagged as an extrapolation** (the guide's correction of the popular version) | Inference from #40/#41's actual study design; **the guide's reading** |
| 43 | **The spacing effect** — spaced study beats massed — was **first identified by Ebbinghaus** and is supported across many explicit memory tasks (free recall, recognition, cued recall, frequency estimation) | Training efficacy — reinforcement | ✅ Verified | Reference work's spacing-effect article; **reference work** |
| 44 | **Cepeda et al. (2006): spaced practice outperformed massed practice in 259 of 271 cases** | Training efficacy — reinforcement | ✅ Verified | Same article's citation; **peer-reviewed** |
| 45 | The spacing benefit is **not robust for items presented only twice and tested after a 24-hour delay** (Shaughnessy, 1977), but is present with four or six presentations | Training efficacy — boundary condition | ✅ Verified | Same; **reference work citing peer-reviewed work** |
| 46 | **Macnamara, Hambrick & Oswald, *Psychological Science*, 2014**: deliberate practice explained **26% of variance in games, 21% music, 18% sports, 4% education, and less than 1% in professions** | Training efficacy — the strongest counterweight | ✅ Verified (abstract level) | The article's own abstract via publisher/PubMed records; **peer-reviewed** |
| 47 | The interpretation that ≤1% in professions **does not mean training is worthless**, but means performance is determined by factors other than practice hours (context, task environment, feedback, problem selection) | Training efficacy | 🔧 **Constructed** (the guide's reading of #46) | This guide; explicitly not a sourced claim |
| 48 | **"Only ~10% of training transfers to the job"**, commonly attributed to a **1982 practitioner article** (Georgenson, *Training & Development Journal*) | Training efficacy — folklore | ⛔ **Rejected.** Widely criticised as unsupported; **the original article, its content and any supporting study could not be verified this pass** | Practitioner attribution only; **do not cite as data** |
| 49 | The claim that training is usually evaluated at **level 1 (reaction) only** | Training efficacy | ⚠ **Flagged as convention.** The documented finding is that implementing all four levels is *difficult* (#35); **no survey establishing the proportion was located** | §9.6; do not cite as a statistic |
| 50 | Practitioner conventions that "manager involvement determines transfer", that "training without follow-up is entertainment", that "the first 30 days decide whether it sticks" | Training efficacy | ⚠ **Flagged — convention, not finding** | Practitioner consensus; structurally consistent with #32 but not verified |
| 51 | The call-review/QA regime's documented failure mode: rewarding **compliance over resolution**, and functioning as **supervisory control over emotional display** | Training efficacy | ✅ Verified (the supervisory-control mechanism) / 🔧 (the practice-level conclusion) | Hochschild's three-criterion definition of emotional labour, including employer control **"through training and supervision"** — reference work's emotional-labour article; **reference work**; the QA inference is the guide's |

### 15.4 The emotional-labour findings

| # | Claim | Class | Status | Source, date, quality |
|---|---|---|---|---|
| 52 | **Emotional labour** was first defined by the sociologist **Arlie Hochschild**; jobs involving it (1) require face-to-face or voice-to-voice contact with the public, (2) require the worker to produce an emotional state in another person, (3) allow the employer, **through training and supervision**, to exercise a degree of control over employees' emotional activities | Foundational concept | ✅ Verified | Reference work's "emotional labor" article citing Hochschild's foundational text; **reference work** |
| 53 | Hochschild's **1983** argument that service workers become estranged from their own feelings at work; her **three emotion-regulation strategies** (cognitive, bodily, expressive) | Foundational concept | ✅ Verified (for the 1983 dating and the strategies) | Same; **reference work** |
| 54 | **Surface acting** = displaying required emotions without changing how one feels; **deep acting** = effortfully changing internal feelings to align with organisational expectations; **research generally shows surface acting to be more harmful to employee health** | Foundational concept | ✅ Verified | Same; **reference work citing peer-reviewed work** |
| 55 | **Hülsheger & Schewe, *Journal of Applied Psychology* (2011), "On the costs and benefits of emotional labor: A meta-analysis of three decades of research"** — **494 correlations from 95 independent studies**; found **substantial relationships of emotion-rule dissonance and surface acting with indicators of strain and burnout** | Meta-analytic evidence | ✅ Verified (method, sample, direction); ⚠ ⛔ **no effect size printed** — the full text was not read | The article's own abstract via APA PsycNet/PubMed; **peer-reviewed** |
| 56 | The inference that a QA regime scoring visible warmth **manufactures the condition associated with strain** | Analytical inference | 🔧 **Constructed** | This guide, §3.2 and §7.5; **explicitly the guide's own** |

### 15.5 Market figures — the class in which **nothing was verified**

| # | Claim | Class | Status | Source, date, methodology |
|---|---|---|---|---|
| 57 | Employment counts, median pay, growth projections or attrition rates for any role in this family in any country | Market figure | ⛔ **Rejected — none verified.** No figure is printed | No source with a stated methodology was verified this pass. The official-statistics route (national labour statistics/occupational classification) was attempted and **blocked by the extraction tool**, which is a **tool limitation, not evidence of absence** — §16 |
| 58 | Compensation bands for these roles | Market figure | ⛔ **Rejected — none verified, none printed.** Salary aggregators' figures are self-reported samples; vendor "state of the industry" surveys are sponsor-defined | §2.2; §16 |
| 59 | "The number of customer success managers grew X% between Y and Z" | Market figure | ⛔ **Rejected** | Only source class available is job-board or platform counts with undisclosed methodology — §16 |
| 60 | SaaS NRR/GRR benchmark medians from vendors or investor-bank surveys | Market figure | ⛔ **Rejected as comparators.** No benchmark with a stated, reproducible methodology was verified this pass | §5.2, §16. **The definitional discipline in §5.2 replaces the benchmark** |
| 61 | **A named listed company's net expansion metric, disclosed in a regulatory filing** — the strongest primary-source class available in this domain | Market figure — **verified channel** | ✅ **Verified channel.** The phrase **"dollar-based net expansion rate" appears in 10-K filings**: an EDGAR full-text query over filings dated 2023–2026 returned **19 hits**, including **Twilio Inc.** (CIK 0001447669) 10-Ks for fiscal years **2022 (filed 2023-02-27), 2023 (filed 2024-02-27), 2024 (filed 2025-02-26)** and 2025 | SEC EDGAR full-text search, queried this pass; **tier-1 primary (regulatory filing)**. **No figure is quoted from any filing** |
| 62 | Organisational taxonomy, levelling, competency frameworks or occupational codes for the customer success role family | Market/role taxonomy | ⛔ **Rejected — not verified.** §2 establishes the *absence* of a documented standard as the finding, and labels every ladder in this guide as inference | §2.2, §16 |

### 15.6 AI claims

| # | Claim | Class | Status | Source, date, quality |
|---|---|---|---|---|
| 63 | **Klarna: an AI customer service assistant handled about two-thirds of customer service chats in its first month, and performed work equivalent to 700 full-time agents** | AI deployment | ✅ **Verified as a company claim** | Klarna press release, **27 February 2024**, as cited in the reference work's Klarna article; **tier-1 primary for its own claim** |
| 64 | Klarna's workforce fell from **5,527 (2022) to 2,907**, with the company attributing much of the reduction to **natural attrition and the replacement of some tasks by technology**; **Reuters reported** the company expected further headcount reductions as AI handled more queries | AI labour effect | ✅ Verified as reported; ⚠ **attribution is the company's own and spans three causes** | *The Guardian* (2025) and Reuters, as cited in the reference work; **tier-1 press** for a company statement |
| 65 | Layoff handling criticised after the CEO published a list of affected employees on LinkedIn | AI-adjacent labour treatment | ✅ Verified as reported | Same; **tier-1 press** |
| 66 | ***Moffatt v. Air Canada*, 2024 BCCRT 149** — Civil Resolution Tribunal of British Columbia, decided **14 February 2024** (Tribunal Member Christopher C. Rivers): **a company can be held liable for misleading information given to customers by an AI chatbot**; **Air Canada held liable for negligent misrepresentation**; **CA$880** in dispute; the airline argued its chatbot was a **separate legal entity** | AI liability | ✅ **Verified — tribunal decision** | Reference work's case article, citing the American Bar Association *Business Law Today* and *Business Insider*; **tier-1 primary (adjudication)** |
| 67 | AI quality failure produces **confident wrong answers**, and *Business Insider* coverage described "botshit" and AI making customer service worse | AI failure mode | ✅ Verified as reported | Same; **industry press** |
| 68 | **"74% of leaders have had to scrap deployed agents due to operational and governance gaps"** | AI deployment | ⚠ **Flagged — vendor-published figure, methodology not disclosed on the page** | COPC Inc. executive guide (page content dated **September 2026**); **vendor content** (COPC sells certification and consulting). The same guide recommends the **NIST AI Risk Management Framework** for governance |
| 69 | Vendor **"containment"/"deflection" rate** claims | AI metric | ⛔ **Not verified as comparable** — definitions are inconsistent and vendor-published; **no figures printed** | §10.2; §16 |
| 70 | Consultancy **predictions** of autonomous resolution rates or role eliminations by a future year | AI projection | ⚠ **Flagged as prediction**, with dates where published; **no figures printed in this guide** | §10.5; specific predictions unverified this pass |
| 71 | Official labour statistics quantifying AI's effect on customer-facing roles | AI labour effect | ⛔ **Rejected — none verified** | §16 |
| 72 | The claim that automatable volume work is thinning the entry-level apprenticeship ladder for these roles | AI skill effect | 🔧 **Constructed**, with documented logic | §10.3; the guide's analysis, empirical scale not verified |
| 73 | The claim that automating QA/sentiment/coaching **industrialises the proxy's failure mode** | AI skill effect | 🔧 **Constructed** | §10.4; the guide's analysis, resting on the documented gaming mechanism (§6.5) |

### 15.7 The guide's own constructions

| # | Construction | Status |
|---|---|---|
| 74 | The "one owner per object of study" cluster framing; the two definitions of *post-sales* and *customer-facing*; the decoder table; the role-family table's boundaries; the seniority ladder; the lifecycle stage model; the eight-stage failure-mode table; the five structural causes of handover failure; the three-handover-question test; the segmentation decision tree; the three funding models; the retention "three disciplines"; the five-clause proxy policy; the training-layer model; the ramp design rules; the call-review redesign; the apprenticeship phase table; the skills decomposition and its trained-vs-needed ratings; the evaluation rule ("something the customer eventually feels"); the AI coaching-layer hazard analysis and its test question; the enterprise four-stakeholder model; the anti-patterns table; and **all** of §12 | 🔧 **Constructed and labelled.** Offered as analysis, template or pedagogy — **explicitly not sourced findings** |
| 75 | All figures in §12 (team size, 14 staff, 16-week programme, 104%/84% retention, cost description) | 🔧 **Constructed, explicitly fictional and illustrative.** Not benchmarks, not derived from any organisation |

### 15.8 Summary of the audit

Of the load-bearing claims: **the framework attributions, metric origins and the learning-science findings are largely verifiable and were verified** — NPS's origin and trademark, the peer-reviewed challenge to it, ACSI's method, CES's origin, Kirkpatrick's origin and the scholarly criticism, Ebbinghaus and its 2015 replication, the spacing effect including 259/271, the deliberate-practice meta-analysis, and the emotional-labour meta-analysis with its sample size. **The failure is concentrated in exactly the class this topic needs most: market figures.** ✅ *Every metric origin was verified;* ⛔ *not one market figure — employment, pay, growth or attrition — was verified to source+date+methodology standard, and none is printed.* ⚠ *The framework-origin record contains one live conflict (Vantive 1996 versus Salesforce mid-2000s for the first named customer success group) which this guide carries as a conflict rather than resolving.*

**And the pattern that explains why:** in this field, **the things that are documented are the things somebody sold** — instruments, methods, certifications, platforms — and **the things that are not documented are the things about the people doing the work.** The metric has a trademark owner. The retention ratio has a disclosure channel. The role has a job title and no standard.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

*First, the methodological point: in this pass the **web search tool returned empty result sets for every query attempted after an initial window, and the page-extraction tool worked.** Empty searches are recorded as a **tool limitation, never as evidence of absence**. Items below are either things a search would have found and this pass could not, or things whose only available source disqualified it as evidence.*

**Metrics and frameworks:**

1. ⛔ **The "earned growth rate" revision** attributed to Reichheld's later work. No source located; authorship, definition and relationship to NPS are all unasserted (§6.1.1, audit #12).
2. ⛔ **Any independent validation of the Customer Effort Score's claimed superiority** over NPS or satisfaction measures. The claim is verified as the originators' claim; the validation is not (§6.1.3, audit #18).
3. ⚠ **The current editions of the ISO standards.** The list used shows ISO 10002:**2014**, ISO 10004:**2008** and ISO 18295-1/-2:**2017**; the ISO catalogue, not a list, is authoritative, and **current editions were not confirmed** (§6.1.5).
4. ⚠ **The first named "Customer Success" group — a live conflict.** The reference work states **companies such as Salesforce developed customer success strategies in the mid-2000s**; an industry account (Customer Success Association) holds that **the first named Customer Success department was created at Vantive in 1996–1997** and that **"it wasn't Salesforce"**. Neither is a primary company disclosure, so this guide carries the conflict rather than resolving it (§2.2, audit #25–26).
5. ⚠ **The specific originators of GROW** (commonly Graham Alexander, Alan Fine and John Whitmore, with Whitmore's *Coaching for Performance* (1992) popularising it). Verified: UK origin, corporate use from the late 1980s/1990s, several versions (§7.7, audit #30).
6. ⛔ **The CX industry certification's specifics** (commonly cited as CXPA and CCXP). The site was unreachable to the extraction tool, so **ownership, launch date, competency framework and market recognition are not asserted** (§7.3, §13.2).
8. ⛔ **Whether vendor certification, vendor health-score models, or vendor "containment"/deflection rates have any validation**, and **what vendor-published first-contact-resolution claims rest on.** No study or standard definition located in any case; no FCR figure is quoted (§6.3, §6.4, §7.3, §10.2, audit #19, #69).
9. ⚠ **The substance behind gated professional material** — the Consortium for Service Innovation's current KCS material and COPC Inc.'s premium benchmarking research were read only at page level (§7.3, §10).

**Training evidence:**

10. ⛔ **Three items in the training-evaluation literature could not be closed:** the provenance of the "four levels form no validated causal chain" critique (the existence of scholarly criticism *is* verified); the Phillips "fifth level" (ROI), for which no source was located; and the origin/content of the "10% of training transfers" figure attributed to a 1982 practitioner article — rejected as evidence and named in §9.7 (audit #36, #37, #48).
11. ⛔ **What proportion of organisational training is evaluated at level 1.** The documented finding is that implementing all four levels is difficult; "everyone only does level 1" is carried as convention (audit #49).
12. ⛔ **Any benchmark for onboarding duration or ramp time.** No verified source; **no ramp benchmark is asserted** (§7.2).

**Market figures — the largest gap in this guide:**

13. ⛔ **Employment counts, median pay, growth projections and attrition rates for every role in this family, in every country.** The official-statistics route was attempted and **the extraction tool was blocked by the statistics agency's site**; the search tool returned nothing. **A tool limitation, not evidence of absence.** No figure is printed (§2.2, §15.5).
14. ⛔ **Any compensation band** from a source with a stated methodology (aggregator figures are self-reported samples; vendor surveys are sponsor-defined); **any geographic distribution or headcount** for the role family; and **any quantification of title inflation**. §2.2 and §2.4 state the absences rather than estimating.
15. ⛔ **SaaS NRR/GRR benchmark medians** from investor-bank or vendor surveys. **No benchmark with a stated, reproducible methodology was verified.** The verified alternative is the named-company disclosure channel — 19 ten-K filings mentioning "dollar-based net expansion rate", including Twilio Inc.'s FY2022–FY2025 filings — verified to *exist*, without quoting a figure (§5.2, audit #61).
16. ⛔ **Any occupational classification, competency framework or levelling standard** for the customer success role family. §2.2's finding is the *absence*, and the absence is the finding.

**AI:**

17. ⛔ **Official labour statistics on AI's effect on customer-facing roles.** None verified (§10.5, audit #71).
18. ⚠ **The specific 2025 reporting that Klarna publicly reversed or partly reversed its AI-first customer service posture.** This would have been the most valuable evidence-graded data point in §10. **It could not be verified and is not asserted.** What *is* verified: the company's February 2024 claim ✅, the reported workforce change with its three-cause attribution ✅, and the adjudicated AI failure in *Moffatt v. Air Canada* ✅. **§10 therefore carries no walk-back claim.**
19. ⚠ **Consultancy predictions of autonomous resolution rates or role eliminations.** Flagged as predictions; **no figures printed in either direction** (§10.5, audit #70).

**One further limitation, stated for completeness:** several framework attributions in this guide run through a **reference work** rather than the primary standard, statute or paper. Where that is so, §15 says so. A reference work is a *secondary* source: where a claim in this guide matters to a decision, follow the citation in §15 to the primary document.

**And one deliberate absence by policy:** where a number could not be verified, commentary, a definition, or a rejection was written instead. **All figures in §12 are labelled fictional and illustrative** rather than borrowed from any organisation.

---

### 16.2 Glossary

| Term | Definition as used in this guide |
|---|---|
| **Account management (AM)** | The commercial relationship owner — renewal, pricing, expansion — rather than adoption detail (§2.1) |
| **ACSI** | American Customer Satisfaction Index — began 1994, developed at the University of Michigan's National Quality Research Center with ASQ and CFI Group; a structural-equation model over three manifest variables, prototyped for Sweden in 1989; ~350,000 interviews a year (§6.1.2) |
| **Adoption** | Whether the licensed capability is used by the people who were supposed to use it; measured in telemetry, not satisfaction (§4.4) |
| **AHT (average handle time)** | Mean contact duration; optimising it degrades resolution and drives repeat contact (§6.3) |
| **Baseline rule** | A value claim is credible only if the baseline was agreed before the outcome and does not rely solely on vendor telemetry (🔧 §4.4) |
| **Business review (QBR/EBR)** | The recurring value-and-roadmap meeting; also the informal renewal pipeline, its cadence usually tracking the renewal date (§4.4) |
| **Call review / QA regime** | Sampled scoring of interactions against a scorecard; documented failure mode is rewarding compliance over resolution, and it is supervisory control over emotional display (§7.5) |
| **CES** | Customer Effort Score — introduced in *HBR*, July–August 2010, by Dixon, Freeman and Toman (CEB lineage), asserting superiority over NPS and satisfaction; proprietary, since revised (§6.1.3) |
| **Churn** | Revenue that stopped, for a specified period and population; meaningless unqualified — ask which churn: logo, gross, net, voluntary, forced, or downgrade (§5.2) |
| **COPC Inc.** | A firm providing contact-centre and CX consulting, certification, research and training, and benchmarking dashboards; its certifications are audits against its own standard (§7.3) |
| **Cognitive apprenticeship** | The learning theory (constructivist and situated-cognition lineage) in which a master models a skill in a real-world context, makes tacit reasoning explicit, and releases autonomy in stages (§7.6, §7.8) |
| **Cost-to-serve** | The fully loaded cost of serving a customer, including human time and support volume; the number most vendors never compute per customer (§5.3) |
| **CSAT** | A family of transaction- or relationship-level satisfaction items with no single originator; ubiquitous, unstandardised, unexamined (§6.1.4) |
| **Customer success (CS)** | A post-sales function (and a company name) for ensuring the customer achieves the outcomes they bought for, usually carrying renewal and expansion responsibility (§1.4, §2.2) |
| **Deep acting** | Effortfully changing one's internal feelings to align with organisational display expectations (§3.2, §15.4) |
| **Deflection / Resolution** | Deflection = a contact that does not reach a human. Resolution = a problem that is gone, verified independently of the channel. **Not the same thing** (§10.2) |
| **Deliberate practice** | Purposeful practice with immediate feedback in a stable task environment; its meta-analysed relationship to performance explains **less than 1% of variance in professions** (§9.4) |
| **Downgrade / contraction** | Revenue retained but reduced; netted against expansion inside NRR, which is how a shrinking core can hide (§5.2) |
| **Emotional labour** | Hochschild's term: managing one's own and others' emotions to meet job expectations, in jobs requiring public contact, the production of an emotional state in another person, and employer control through **training and supervision** (§3.2) |
| **Escalation** | A claim on internal attention; its currency is reputation, and low-quality escalation devalues the role's reports (§3.3) |
| **FCR (first-contact resolution)** | Contacts resolved without transfer, callback or reopen; definition set by the reporting tool; vendor claims about its correlates are unverified (§6.3) |
| **Forgetting curve** | Ebbinghaus' 1880–1885 self-study of retention decline; replicated by Murre and Dros (*PLOS ONE*, 2015) using the method of savings (§9.2) |
| **GRR (gross revenue retention)** | Cohort revenue retained **excluding** expansion; cannot exceed 100%; the strictest measure and the one that cannot be flattered (§5.2) |
| **GROW** | A coaching model (Goal, Reality, Obstacles/Options, Way Forward) developed in the UK and used in corporate coaching from the late 1980s/1990s; several versions exist (§7.7) |
| **Health score** | A composite, vendor-configured account-risk index with no peer-reviewed validation located; defensible as a conversation aid, not as a prediction (§6.4) |
| **Handover** | The sales → post-sales transfer of the customer's context; the lifecycle's most cited failure point, and a structural one (§4.2) |
| **ITIL** | The IT service management practice set developed by the UK CCTA in the 1980s (released 1989; ITIL 4 in February 2019); trademark now held by PeopleCert (acquired AXELOS 2021); no independent organisational compliance assessment exists (§4.3) |
| **KCS** | Knowledge-Centered Service (the Consortium's site now presents "Knowledge-Centered Success"), owned by the **Consortium for Service Innovation**, a non-profit alliance; knowledge capture is integrated into the resolution workflow (§7.4) |
| **Kirkpatrick model** | The four-level training evaluation model (reaction, learning, behaviour, results) from Kirkpatrick's late-1950s articles based on his doctoral dissertation; influential and criticised; all four levels documented as difficult to implement (§9.6) |
| **Logo retention** | The proportion of accounts still customers at period end, regardless of revenue; a 90% downgrade still counts as retained (§5.2) |
| **Loss review** | The post-mortem on a churned account asking at which stage the outcome became unrecoverable; its absence is why organisations never learn from loss (§4.1, §12) |
| **NRR / NDR** | Net revenue retention / net dollar retention — cohort revenue **including** expansion, hence able to exceed 100%; not comparable across companies without cohort, period and currency definitions (§5.2) |
| **NPS** | Net Promoter Score — developed by Bain's Fred Reichheld, popularised in *HBR*, December 2003; %promoters − %detractors; trademark owned by Reichheld with Bain and Satmetrix; predictive claim contested in peer-reviewed literature (§6.1.1) |
| **Post-sales** | After signature and commitment: onboarding, adoption support, issue handling, value demonstration, renewal, expansion — **including** implementation, **excluding** the sales process (§1.3) |
| **Ramp** | Two different things: a new hire's progress to productivity, and a customer's time to first value. Always ask which (§7.2) |
| **Reopen / repeat-contact rate** | The proportion of resolved contacts that return; the practical antidote to FCR gaming, and rarely reported (§6.3) |
| **Responsibility without authority** | The role family's defining condition: accountable for outcomes it does not control, using levers it does not own, measured by instruments it did not choose (§3.4) |
| **Segmentation** | The logic deciding which customers get which coverage model; treated here as a written decision with a named owner, not an implicit habit (§5.3) |
| **Spacing effect** | Spaced study beats massed study for retention; first identified by Ebbinghaus; **259 of 271 cases** in Cepeda et al. (2006) (§9.3) |
| **Surface acting** | Displaying the required emotion without changing how one feels; the mechanism the meta-analytic evidence associates with strain and burnout (§3.2, §15.4) |
| **Transfer of training** | The generalisation and maintenance of what was learned to the work environment (Baldwin & Ford, 1988); outcomes are positive, negative or zero (§9.1) |
| **Value realisation** | The claim that the customer obtained a measurable outcome; not a metric, and meaningful only when the baseline was agreed in advance (§4.4) |
| **Work environment (as a training input)** | The formal position of the canonical transfer model: the environment conditions whether training transfers, alongside design and trainee characteristics (§9.1, §9.5) |

**Marker legend used throughout:** ✅ verified against a named source this pass · ⚠ flagged (reported, consensus, or estimate — provenance stated) · ⛔ rejected or not verifiable · 🔧 the guide's own construction, explicitly not sourced.

---

### 16.3 The cross-references, consolidated

**Siblings in `management/`:** [meddicc_guide.md](meddicc_guide.md) (the deal cycle and qualification — §1.2, §4.1, §4.2, §8.5) · [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) (the FDE role — §2.1, §13.1) · [vendor_management_guide.md](vendor_management_guide.md) (the buy side — §1.2, §11.3) · [ancillary_revenue_products_guide.md](ancillary_revenue_products_guide.md) (revenue products — §1.2, §8.5) · and the human-skills curriculum, cross-referenced by name in §8, §11.5 and §12.4 and never re-taught here: [conflict_management_guide.md](conflict_management_guide.md), [facilitation_skills_guide.md](facilitation_skills_guide.md), [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md), [authority_skills_guide.md](authority_skills_guide.md), [body_language_lie_detection_guide.md](body_language_lie_detection_guide.md), [micro_expressions_guide.md](micro_expressions_guide.md), [the_first_90_days_guide.md](the_first_90_days_guide.md), [situational_leadership_slii_guide.md](situational_leadership_slii_guide.md), [resilience_engineering_guide.md](resilience_engineering_guide.md). Also [reverse_job_search_guide.md](reverse_job_search_guide.md) (§13), [directly_responsible_individual_guide.md](directly_responsible_individual_guide.md) (§8.6), [business_case_development_guide.md](business_case_development_guide.md) (§8.4–8.5), [managing_up_down_sideways_guide.md](managing_up_down_sideways_guide.md) (§8.4, §8.6) and [product_management_frameworks_guide.md](product_management_frameworks_guide.md) (§8.1).

**Elsewhere in the repository** (note the `../` prefix, and that the sales survey lives in `technology/`, not `management/`): [../technology/operational_support_frameworks_guide.md](../technology/operational_support_frameworks_guide.md) (IT service management — §1.2, §4.3, §8.2, §11.3); [../technology/sales_methodology_frameworks_guide.md](../technology/sales_methodology_frameworks_guide.md) (the sales-methodology survey — §1.2, §13.1); [../technology/product_thinking_guide.md](../technology/product_thinking_guide.md) (§8.1); and the banking domain in [../banking/](../banking/), including [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md), [../banking/risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md) and [../banking/ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) (§11.3).

---

### 16.4 Closing Summary

Post-sales and customer-facing work is a **young, commercially invented function** — assembled inside B2B software in the mid-2000s to defend recurring revenue out of three borrowed vocabularies — that accumulated real practitioners, a real literature, and almost no documentation of its own people. Its instruments are trademarked, contested and gamed; its retention ratios are the only honest numbers it has, and they are defined by whoever prints them; its role titles mean whatever the employer decided; and its training is, mostly, a sequence of events evaluated by how people felt about them.

Three things this guide would want a reader to keep:

1. **The metrics are the politics.** A function funded by a loss that did not happen will defend the instrument that makes the loss visible, even after the instrument stops measuring anything. The remedy is structural, not statistical: measure the pair, report the strict ratio, and take the survey off the bonus — Reichheld's own advice on that last point is the best available citation ✅.
2. **The evidence on training points somewhere specific and unpopular.** Spacing beats massing ✅, the work environment is an input to transfer ✅, and in professions deliberate practice explains less than 1% of performance ✅. Buy fewer courses; buy more feedback, more case-based practice, and more authority for the people doing the work. And build the **case library** — it trains people, it ramps the next hire, and it is the corpus anything automated will later have to retrieve from.
3. **The work is structurally unfair in a way that is fixable.** The responsibility/authority gap, the handover seam, the funding model and the measurement regime are all causes, all created by management, all repairable. What is *not* repairable by training is people's tolerance for them — and offering resilience instead of repair is the industry's most common substitution of a symptom for a cause.

**And the honest boundary of all of it:** this guide verified metric origins, framework attributions, learning-science findings, and the fact of a named company's regulatory disclosure — and it verified **not one market figure about the people in these roles.** No verified employment count, pay band, attrition rate, levelling standard or estimate of how many people do this work. That absence is not a gap in this guide; it is the state of the field, and it is why the guide's own ladders, models and worked example are labelled constructions rather than findings. Anyone who needs a number for a decision should treat every circulating figure in this domain as unverified until they have found its source, its date and its method.

The discipline of the work, then, is the same as the discipline of this guide: be precise about what you can evidence, separate what you were sold from what you observed, keep the instrument honest enough to lose an argument with, and measure the thing the customer will feel rather than the thing that flatters the function. A customer relationship is not what is promised in the room where it is sold. It is kept or lost in the months of unglamorous execution that follow, by people who rarely control the causes and always carry the consequences. Which is to say: the entire subject is **what happens after the signature.**
