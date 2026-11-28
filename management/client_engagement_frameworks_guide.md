# Client Engagement Frameworks and Methodologies: The Model and the Method Are One Bet

**This is the guide to the object that sits between a client and a delivery team and is almost never written down in one place: the engagement. It owns the commercial form the work takes (fixed price, time and materials, capped time and materials, outcome-based, managed service, staff augmentation, retainer), the phases the engagement runs through (pursuit, proposal, contract, discovery, delivery, handover, close), the delivery methodology chosen to run it, and the governance that holds model and method together. Its argument is narrow and testable: the engagement model and the delivery methodology are two expressions of one assumption - how much is knowable before the work starts - and choosing them separately is choosing them wrongly. It measures the repository gap that left this unowned, decodes ten overloaded terms, prints a risk-allocation table for the seven models, walks the lifecycle phase by phase, treats the statement of work as an instrument rather than a formality, and audits every claim at the end. Standards are cited only where verified: ISO 20700:2017 clauses 4.1.2, 4.1.3, 5, 6 and 7, and SS 680:2021's published scope. SS 680's internal clause structure is paywalled and is marked unverified throughout.**

> **Author:** Jack Liu Shurui - Solution Architect at Cymbal Bank, Singapore
> **Context:** Management / Practice Guide (management/) - owns the engagement itself: the commercial form it takes, the phases it runs through, the method it is delivered by, and the governance that holds model and method together.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** ISO 20700:2017(E), *Guidelines for management consultancy services* - clause 4.1.2 (typical consulting process: contracting, execution, closure), clause 4.1.3 (responsibility allocation), and clauses 5, 6 and 7, quoted here as recorded and verified in [ss680_management_consultants_guide.md](ss680_management_consultants_guide.md) section 3.4 (read 28 Nov 2026); SS 680:2021, *Specification for management consultants*, ICS 03.080.20, public scope statement only, via the Singapore Standards eShop product page and the NLB catalogue as recorded in the same guide (internal clause structure paywalled, not used); US Federal Acquisition Regulation 16.101, acquisition.gov, FAC 2026-01 effective 13 Mar 2026, read 28 Nov 2026 (contract type as an allocation of responsibility for performance cost); and repo measurements run by the author on this repository on 28 Nov 2026, reproduced in section 1.3.
> **Last Updated:** November 2026
> **Companion guides (management/, same folder):** [Management Consulting Skills Guide](management_consulting_skills_guide.md) (its section 6 is the delivery skeleton and the client-management craft; this guide's sections 4, 7 and 8 cross-reference it and do not re-teach hypothesis-driven problem solving) | [Consulting & Advisory Experience Guide](consulting_advisory_experience_guide.md) (owns the career ladder, advisory vs delivery, positioning and the profession's economics; section 11 here cross-references it for the buyer's-eye view of a consulting career) | [SS 680 Management Consultants Guide](ss680_management_consultants_guide.md) (owns the standard, its history, ISO 20700 and the certification ecosystem; this guide cites its verified clause record and never restates it - sections 2, 4 and 15) | [Workshop Methodology Guide](workshop_methodology_guide.md) (owns workshop design and facilitation; section 6 here says what discovery must produce and hands the how-to-run-it to that guide) | [Business Development Experience Guide](business_development_experience_guide.md) (owns origination as a role - motions, deal types, partner lifecycle, attribution; section 4.2 here covers only the pursuit artefact, not the sales craft) | [Business Case Development Guide](business_case_development_guide.md) (owns the business-case artefact and its financials; section 11 here explains why that artefact's approval logic forces a price before discovery, and does not rebuild the case) | [Vendor Management Guide](vendor_management_guide.md) (owns the buy side - managing a supplier once engaged; section 12 here owns the contract's risk allocation, and defers supplier performance management to it) | [Communication & Stakeholder Management Skills Guide](communication_stakeholder_management_skills_guide.md) (owns stakeholder mapping, influence and difficult conversations; section 8 here owns the formal governance body those skills are exercised inside) | [Forward Deployed Engineering Guide](forward_deployed_engineering_guide.md) (owns the FDE variant of the engagement, the one neighbour with statement-of-work vocabulary; section 3.9 and section 16.2 cross-reference it and do not re-derive it) | and, as the nearest false overlap on the words "engagement model", [McKinsey Approach Guide](mckinsey_approach_guide.md) (its section 4 is one firm's lifecycle and its economics; section 1.5 states the boundary - this guide owns the generic taxonomy, the SOW instrument and governance across all models, and quotes no firm's rates).
> **Companion guides (banking/, prefix `../banking/`):** [Operational Resilience Framework Guide](../banking/operational_resilience_framework_guide.md) (important business services and exit expectations, cross-referenced from section 12) | [Enterprise Risk Management Guide](../banking/enterprise_risk_management_guide.md) (risk taxonomy and appetite, section 12) | [GRC in Banking Guide](../banking/grc_in_banking_guide.md) (the control environment an engagement's deliverables must fit, section 12) | [MAS Regulations & Guidelines Guide](../banking/mas_regulations_guidelines_guide.md) (the regulatory register, section 12). There is no vendor/third-party risk guide in `banking/` at the time of writing - checked by directory listing on 28 Nov 2026 - so the third-party risk material in section 12 stands on its own and is referenced to the Vendor Management Guide instead.

---

**How to use this guide:** Section 1 is the map - what the engagement is, the ten terms that get confused, the measured gap that proves nobody in this repository owns it, the neighbour map, and the thesis with its concession. Read it first. Section 2 is the argument: the model and the method are one decision, argued pairing by pairing. Section 3 is the reference table you will actually use - the seven commercial forms and what each allocates. Section 4 is the lifecycle in sequence; sections 5 and 6 zoom into its two most consequential artefacts, the statement of work and discovery. Section 7 is the methodology section, written as conditions under which each method is wrong rather than as a menu. Sections 8 and 9 are the two things that decide whether the work survives: governance and handover. Section 10 is the failure taxonomy. Section 11 flips the chair and describes how consulting is bought, because the buyer's approval logic is what produces most mispaired engagements. Section 12 is the regulated-bank overlay. Section 13 is the anti-pattern list; section 14 is the fictional worked example; section 15 is the claims audit and the could-not-verify ledger; section 16 is the glossary, cross-references and closing.

**Integrity convention:** ✅ = verified this pass against a primary source; ⚠ = flagged, approximate or unverified; ⚠-knowledge = well-documented industry knowledge not re-verified this pass; ❌ = could not verify, or sources conflict. The three marks are the only non-ASCII characters in this file. Everything else is ASCII: " - " where a dash is wanted, straight quotes, no arrows (-> is used instead).

---

**Table of Contents**

1. [The Overview, the Decoder, and the Gap](#1-the-overview-the-decoder-and-the-gap)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Decoder: Ten Terms in Three Registers](#12-the-decoder-ten-terms-in-three-registers)
   - 1.3 [The Measured Gap](#13-the-measured-gap)
   - 1.4 [The Neighbour Test: Two Files Measured](#14-the-neighbour-test-two-files-measured)
   - 1.5 [The Neighbour Map and the McKinsey Boundary](#15-the-neighbour-map-and-the-mckinsey-boundary)
   - 1.6 [The discovery False Friend](#16-the-discovery-false-friend)
   - 1.7 [The Thesis and the Concession](#17-the-thesis-and-the-concession)
   - 1.8 [The Boundary: What This Guide Does Not Own](#18-the-boundary-what-this-guide-does-not-own)
2. [The Two Axes Are One Axis](#2-the-two-axes-are-one-axis)
   - 2.1 [The Two Axes, Named](#21-the-two-axes-named)
   - 2.2 [Knowability Is the Only Variable That Matters](#22-knowability-is-the-only-variable-that-matters)
   - 2.3 [The Pairings, Argued One by One](#23-the-pairings-argued-one-by-one)
   - 2.4 [The Pairing Table](#24-the-pairing-table)
   - 2.5 [The Concession](#25-the-concession)
   - 2.6 [What Follows for a Bank Architect](#26-what-follows-for-a-bank-architect)
3. [The Engagement Models and What Each Allocates](#3-the-engagement-models-and-what-each-allocates)
   - 3.1 [The Risk-Allocation Table](#31-the-risk-allocation-table)
   - 3.2 [Fixed Price](#32-fixed-price)
   - 3.3 [Time and Materials](#33-time-and-materials)
   - 3.4 [Capped Time and Materials](#34-capped-time-and-materials)
   - 3.5 [Outcome-Based and Value-Based](#35-outcome-based-and-value-based)
   - 3.6 [Managed Service](#36-managed-service)
   - 3.7 [Staff Augmentation](#37-staff-augmentation)
   - 3.8 [Retainer](#38-retainer)
   - 3.9 [What the Table Cannot Show](#39-what-the-table-cannot-show)
4. [The Lifecycle, Phase by Phase](#4-the-lifecycle-phase-by-phase)
   - 4.1 [The Seven Phases and Their Artefacts](#41-the-seven-phases-and-their-artefacts)
   - 4.2 [Pursuit, Qualification, Proposal and RFP Response](#42-pursuit-qualification-proposal-and-rfp-response)
   - 4.3 [Contract and Statement of Work](#43-contract-and-statement-of-work)
   - 4.4 [Discovery and Delivery](#44-discovery-and-delivery)
   - 4.5 [Handover, Close and Lessons](#45-handover-close-and-lessons)
   - 4.6 [The Skipped-Phase Pattern](#46-the-skipped-phase-pattern)
5. [The SOW as an Instrument](#5-the-sow-as-an-instrument)
   - 5.1 [What a Statement of Work Is and Is Not](#51-what-a-statement-of-work-is-and-is-not)
   - 5.2 [The Clause Set](#52-the-clause-set)
   - 5.3 [Assumptions and Dependencies](#53-assumptions-and-dependencies)
   - 5.4 [Acceptance Criteria](#54-acceptance-criteria)
   - 5.5 [Exclusions: Where Engagements Are Won or Lost](#55-exclusions-where-engagements-are-won-or-lost)
   - 5.6 [Payment Milestones and Change Control](#56-payment-milestones-and-change-control)
   - 5.7 [Reading a SOW as a Risk Document](#57-reading-a-sow-as-a-risk-document)
6. [Discovery as the Pivotal Phase](#6-discovery-as-the-pivotal-phase)
   - 6.1 [What Discovery Must Produce](#61-what-discovery-must-produce)
   - 6.2 [What Discovery Costs, in Time and Access](#62-what-discovery-costs-in-time-and-access)
   - 6.3 [The Compression Finding](#63-the-compression-finding)
   - 6.4 [Discovery Under a Fixed Price](#64-discovery-under-a-fixed-price)
7. [The Methodologies and When Each Is Wrong](#7-the-methodologies-and-when-each-is-wrong)
   - 7.1 [Waterfall](#71-waterfall)
   - 7.2 [Iterative and Agile](#72-iterative-and-agile)
   - 7.3 [Hybrid](#73-hybrid)
   - 7.4 [Design-Led](#74-design-led)
   - 7.5 [Value-Led Change](#75-value-led-change)
   - 7.6 [The Selection Test](#76-the-selection-test)
8. [Governance and the Economics of Scope Creep](#8-governance-and-the-economics-of-scope-creep)
   - 8.1 [The Three Instruments](#81-the-three-instruments)
   - 8.2 [The Steering Committee: What It Can and Cannot Decide](#82-the-steering-committee-what-it-can-and-cannot-decide)
   - 8.3 [Change Control and Escalation](#83-change-control-and-escalation)
   - 8.4 [Scope Creep Is a Pricing Problem Wearing a Delivery Costume](#84-scope-creep-is-a-pricing-problem-wearing-a-delivery-costume)
9. [Handover](#9-handover)
   - 9.1 [What a Transition Actually Transfers](#91-what-a-transition-actually-transfers)
   - 9.2 [The Six Requirements](#92-the-six-requirements)
   - 9.3 [The Warranty Period and the Named Successor](#93-the-warranty-period-and-the-named-successor)
   - 9.4 [Why the Cut Phase Decides Whether Benefit Survives](#94-why-the-cut-phase-decides-whether-benefit-survives)
10. [What Actually Causes Failure](#10-what-actually-causes-failure)
   - 10.1 [The Taxonomy](#101-the-taxonomy)
   - 10.2 [The Three Clusters](#102-the-three-clusters)
11. [The Client's Side](#11-the-clients-side)
   - 11.1 [The Budget Cycle](#111-the-budget-cycle)
   - 11.2 [Procurement](#112-procurement)
   - 11.3 [Sponsor Incentives](#113-sponsor-incentives)
   - 11.4 [Why the Buyer's Approval Logic Drives the Model](#114-why-the-buyers-approval-logic-drives-the-model)
12. [The Regulated and Banking Angle](#12-the-regulated-and-banking-angle)
   - 12.1 [Why the Contract Is Longer Than the Scope](#121-why-the-contract-is-longer-than-the-scope)
   - 12.2 [Third-Party Risk and Outsourcing](#122-third-party-risk-and-outsourcing)
   - 12.3 [Audit, Exit Rights, Concentration and Subcontracting](#123-audit-exit-rights-concentration-and-subcontracting)
   - 12.4 [What This Means for the Architect](#124-what-this-means-for-the-architect)
13. [The Anti-Patterns](#13-the-anti-patterns)
   - 13.1 [The Ten](#131-the-ten)
   - 13.2 [Why They Cluster](#132-why-they-cluster)
   - 13.3 [The Test](#133-the-test)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
   - 14.1 [The Scenario](#141-the-scenario)
   - 14.2 [How the Model and the Method Disagreed](#142-how-the-model-and-the-method-disagreed)
   - 14.3 [What It Cost](#143-what-it-cost)
   - 14.4 [The Guardrail and the Amended Thesis](#144-the-guardrail-and-the-amended-thesis)
15. [The Claims Audit and What Could Not Be Verified](#15-the-claims-audit-and-what-could-not-be-verified)
   - 15.1 [The Claims Audit](#151-the-claims-audit)
   - 15.2 [What Could Not Be Verified](#152-what-could-not-be-verified)
16. [The Glossary, the Cross-References and the Closing Summary](#16-the-glossary-the-cross-references-and-the-closing-summary)
   - 16.1 [The Glossary](#161-the-glossary)
   - 16.2 [The Cross-References](#162-the-cross-references)
   - 16.3 [The Closing Summary](#163-the-closing-summary)

---

## 1. The Overview, the Decoder, and the Gap

### 1.1 The Short Answer

An engagement is not a contract and it is not a project plan. It is the design that says, in one breath, three things: what commercial form the work takes, what sequence of phases it will move through, and what method will be used to do the work inside those phases. Almost every painful engagement in a bank has one of these three settled and the other two inherited by accident - the contract type chosen by procurement, the method chosen by the vendor's delivery practice, and the lifecycle chosen by nobody.

This guide owns all three, and it owns the claim that joins them. The claim is testable, which is why it is worth a guide: if the commercial form asserts that the answer is knowable now (a price fixed before the work starts) and the delivery method asserts that the answer is only discoverable as you go (discovery-led, iterative), the engagement is carrying two contradictory beliefs about the same unknown. One of them will be paid for twice - once in the price's contingency, and again in the rework when the discovery contradicts the price.

That is the whole thesis in one paragraph. The remaining fifteen sections unpack it, bound it, and audit it.

### 1.2 The Decoder: Ten Terms in Three Registers

Ten words carry most of the confusion in this territory, because each of them is used in at least two registers - a commercial register where it allocates risk, and a delivery register where it describes activity - and the two registers are rarely distinguished in the same conversation. The decoder below is the one to keep open while reading a statement of work.

| Term | What it means in this guide | The confusion it causes |
|---|---|---|
| **Engagement** | The whole commercial arrangement: form, phases, method, governance. Not a synonym for "project". | People say "engagement" when they mean the contract, or the relationship, or the piece of work - so nobody can tell which one is being changed. |
| **Engagement model** | The commercial form: fixed price, time and materials, capped time and materials, outcome-based, managed service, staff augmentation, retainer. | Often used to mean "how we work together" (a relationship style) or one firm's proprietary lifecycle. Both readings lose the risk allocation, which is the only part that binds. |
| **SOW (statement of work)** | The instrument that converts an agreed commercial form into a bounded, testable obligation. | Treated as a formality attached to the contract, when it is the document that decides who pays for being wrong. |
| **Discovery** | A priced, bounded phase whose product is knowledge that changes the plan. | Used for any early activity, including sales calls, and therefore never resourced as a phase with its own acceptance criteria. |
| **Workstream** | A vertical slice of delivery with its own lead, plan and dependencies, inside one engagement. | Confused with phase (sequence) and with team (people); a workstream is neither. |
| **Steering committee** | The governance body that owns trade-offs the delivery team cannot make: scope, sequence, funding release, escalation. | Used as a status meeting with senior attendees, which produces visibility without decisions. |
| **Change control** | The mechanism that reprices or re-scopes when reality diverges from the SOW. | Either absent (so change is absorbed silently) or weaponised (so every change becomes a negotiation). |
| **Acceptance criteria** | The test that says a deliverable is done, written before it is built. | Replaced by "client sign-off", which is a person's mood rather than a test. |
| **Handover** | The transfer of knowledge, decision rights and run capability to a named successor owner. | Reduced to a document pack and a closing meeting, which transfers neither knowledge nor rights. |
| **Outcome-based** | A commercial form in which payment is tied to a measured outcome rather than to effort or to a deliverable. | Confused with "outcome-focused", which is a good intention, not a contract type with an allocation. |

### 1.3 The Measured Gap

The claim that this territory is unowned is not an impression. It was measured on this repository on 28 November 2026, across all `*.md` files, counting files that contain each pattern (case-insensitive, alternation as a grouped regex, word boundaries where the term is short enough to be a substring of something else).

| Pattern searched (repo-wide, `*.md`) | Files containing it |
|---|---|
| `client engagement` | 7 |
| `engagement model` OR `engagement framework` | 11 |
| `SOW` OR `statement of work` | 8 |
| `outcome-based` OR `value-based` | 32 |
| `steering committee` | 30 |
| `handover` OR `transition plan` | 44 |
| `discovery` | 226 (see 1.6 - the count is misleading) |

`management/` alone holds 57 guides. Seven files mention "client engagement" and eleven mention an engagement model or framework, across fifty-seven management guides and hundreds of files repo-wide. The vocabulary exists; the ownership does not. The two counts that look healthy - thirty-two for value-based language and thirty for steering committees - are dominated by engineering and product guides that use the words for something else entirely (value-based care, value-based pricing of a product, a steering committee mentioned once as an audience). A keyword count is evidence of vocabulary, not of coverage.

### 1.4 The Neighbour Test: Two Files Measured

The two closest neighbours were measured directly, because a claim of absence has to survive the strongest counter-example. Both are the guides a reader would go to first for this material.

| File | Lines | What was measured inside it | Reading |
|---|---|---|---|
| [management_consulting_skills_guide.md](management_consulting_skills_guide.md) | 987 | Section 6, "Client Management and Project Delivery", lines 543-630, is 88 lines. Inside section 6: `SOW`/`statement of work` = 0, `discovery` = 0, `value-based`/`outcome-based`/`fixed-price` = 0, `RFP`/`proposal` = 0, `engagement model`/`engagement lifecycle` = 1. Whole file: SOW 0, discovery 0, value/outcome/fixed-price 2, RFP/proposal 0. | It has a delivery skeleton (kickoff -> diagnosis -> analysis -> synthesis -> recommend -> handover) and teaches client management as a soft skill. It has no commercial machinery: no contract form, no SOW, no discovery phase, no handover method. |
| [ss680_management_consultants_guide.md](ss680_management_consultants_guide.md) | 620 | Whole file: `engagement lifecycle`/`engagement model` = 0, `SOW`/`statement of work` = 0, `discovery` = 0, `outcome-based`/`value-based`/`fixed-price`/`time and materials` = 0, `handover` = 0, `steering committee` = 0, `change control` = 0. Section 5, "The Consultancy Process", lines 353-395 (43 lines), measures 0 on SOW, discovery, price models and handover. | It owns the standard, not the engagement. ISO 20700's contracting-execution-closure is cited as the international frame, but the guide does not describe how a contract is formed, priced, scoped or closed out in practice. |

This is the decisive measurement. The guide that teaches the consulting craft ends its client-management section without a single mention of a statement of work or a discovery phase; the guide that owns the professional standard cites the process clause without describing the process. Between them they cover the craft and the standard, and leave the commercial object unexamined.

### 1.5 The Neighbour Map and the McKinsey Boundary

| Guide | Lines | What it owns | The boundary this guide draws |
|---|---|---|---|
| [management_consulting_skills_guide.md](management_consulting_skills_guide.md) | 987 | Hypothesis-driven problem solving, structured thinking, data analysis, storytelling, case interviews; its section 6 is client management plus a delivery lifecycle. | It owns the thinking. This guide does not teach how to structure a problem; it teaches what commercial form the structured work is sold under. |
| [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) | 812 | The career and the profession's economics: ladder, advisory vs delivery, conversion and positioning, independent and fractional routes, the AI-era effect, Singapore. | It owns the person. This guide owns the arrangement the person works inside. |
| [ss680_management_consultants_guide.md](ss680_management_consultants_guide.md) | 620 | The standard: SS 680 history, relation to ISO 20700, the certification ecosystem, competency, ethics, CPD. | It owns the standard. This guide cites its verified clause record and adds the commercial machinery the standard does not specify publicly. |
| [workshop_methodology_guide.md](workshop_methodology_guide.md) | 785 | The workshop instrument: design, diverge/sensemake/converge technique libraries, facilitation craft, retrospectives. | It owns the room. Section 6 here says what discovery must produce; that guide says how to run the session that produces it. |
| [business_development_experience_guide.md](business_development_experience_guide.md) | 873 | Origination as a role: BD vs sales, motions and deal types, partner lifecycle, metrics and attribution, the ladder. | It owns winning the work. This guide starts at the point where the work has been won or is being qualified, and owns what is then signed. |
| [business_case_development_guide.md](business_case_development_guide.md) | 712 | The business-case artefact: frameworks, structure, financials, options appraisal, risk. | It owns the justification artefact. Section 11 explains why that artefact's approval logic forces a price before discovery - the mechanism, not the method. |
| [vendor_management_guide.md](vendor_management_guide.md) | 925 | The buy side: managing suppliers, performance, relationships, reviews. | It owns the supplier after signature. This guide owns the instrument that is signed and the governance that sits above both parties. |
| [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) | 554 | Stakeholder communication: mapping, influence, difficult conversations, managing up. | It owns the conversation. Section 8 here owns the formal body - the steering committee and the change board - in which those conversations take decisions. |
| [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) | 707 | The FDE engagement variant: embedded pods, discovery -> deployment -> adaptation -> handover. | The only neighbour with SOW vocabulary (seven lines, zero in the two consulting guides), used to contrast FDE ("beyond the SOW") with professional services ("SOW compliance"). Section 3.9 cross-references it; this guide does not re-derive the FDE model. |
| [mckinsey_approach_guide.md](mckinsey_approach_guide.md) | 715 | One firm: its history, its problem-solving approach, and its section 4, titled "The Engagement Model", which describes that firm's lifecycle narratively, including its economics. | This is the closest false overlap on the words "engagement model". That guide owns one firm's model and its economics. This guide owns the generic commercial-form taxonomy, the SOW instrument and engagement governance across all models. No firm's model is re-described here and no rate or price figure is quoted anywhere in this file. |

The McKinsey boundary deserves one more sentence, because it is the place a reader is most likely to think this guide is redundant. Section 4 of that guide answers "how does this firm run an engagement". This guide answers "what are the available commercial forms, what does each one allocate, and what happens when the form and the method disagree". A reader who wants one firm's practice should read that guide; a reader who has to choose, price, scope or govern an engagement - in either chair - should read this one. The two do not overlap in object, only in vocabulary.

### 1.6 The discovery False Friend

`discovery` matches 226 Markdown files repo-wide. That number is not evidence of anything, and saying so is part of the argument. A sample of contexts pulled at random from those files: MCP Discovery, A2A Agent Card discovery, DNS discovery, OpenID Connect discovery, "capability discovery", registry and runtime service discovery, a security guide's "discovery is the mechanism that converts exposure", and "algorithm-driven discovery" in an ecommerce guide. The word is a standard term in protocols, security, service meshes and product analytics before it is ever an engagement phase.

Measured properly, the engagement sense is small:

| Measure | Count | What it shows |
|---|---|---|
| Files matching `discovery` (all senses) | 226 | Mostly protocol and technical senses; the raw count is misleading. |
| Files matching `discovery` in `management/` at all | 19 | Of 226, only nineteen are even in the management folder. |
| Files matching `discovery` immediately followed by phase, workshop, sprint, assessment, report or stage | 8 | The strict engagement sense - a discovery that is a named, bounded activity. |
| Files matching `scoping` | 98 | Almost all API scoping or authorisation scopes, not engagement scoping. |
| Files matching `scoping` immediately followed by workshop, exercise, document or session | 3 | The engagement sense of scoping is three files. |
| Files matching `proposal` | 116 | Of which 26 are in `management/`; most elsewhere are technical proposals in engineering guides. |

The lesson generalises beyond this guide: in a repository this size, the engagement sense of a term has to be measured by context, not by keyword count. A word that appears 226 times and means the engagement phase eight times is a word nobody owns. That is the gap, stated in the only way that survives scrutiny.

### 1.7 The Thesis and the Concession

The thesis, stated precisely: **the engagement model and the delivery methodology are two expressions of the same assumption - how much is knowable before you start - so choosing them separately is choosing them wrongly.** The sharpest form of it: a fixed price paired with a discovery-led method buys the same risk twice.

The concession, which is printed here and again in section 2.5 and section 14.4, because a thesis that cannot be falsified is marketing: engagements also fail for reasons orthogonal to knowability. A sponsor leaves. Funding is withdrawn at a quarter boundary. A procurement rule changes mid-flight. A key person resigns and the replacement has a different mandate. None of those are model-method mismatches and this guide's thesis does not explain them. **The claim is about structural failure - failure caused by the arrangement - not about all failure.** Sections 10 and 15 keep that distinction honest: the taxonomy in section 10 includes orthogonal causes explicitly, and section 15 records that no dataset was found that would let anyone put a rate on how often structural mismatch is the cause.

### 1.8 The Boundary: What This Guide Does Not Own

This guide does not teach problem solving, facilitation, business-case construction, stakeholder influence, supplier performance management, origination, the consulting career, the FDE model, or any firm's proprietary lifecycle. Each of those has a guide in this folder, and each is cross-referenced at the point where the reader would otherwise expect it here.

What it does own, and what no other guide in the repository owns:

- **The commercial-form taxonomy** - the seven ways an engagement can be priced and risk-allocated, and what each one makes the client unable to ask for.
- **The SOW as an instrument** - the clause set, and the argument that exclusions and acceptance criteria are where the engagement is actually won or lost.
- **The lifecycle as a sequence with artefacts and decisions** - not a narrative of how one firm works, but the phase table with the artefact and the decision each phase must produce.
- **Governance across all models** - the steering committee, change control and escalation as risk-allocation machinery, not as meeting types.
- **Handover as a phase with requirements** - the six things a transition must transfer, and why the phase most often cut is the one that decides whether the benefit survives.

---

## 2. The Two Axes Are One Axis

### 2.1 The Two Axes, Named

The first axis is the **commercial form**: how the engagement converts work into an obligation and an entitlement to be paid. The seven forms are set out in section 3. What matters for this section is that every form is, mechanically, an allocation of two distinct risks - estimation risk (the risk that the effort or duration turns out to be different from what was assumed) and scope risk (the risk that what must be delivered turns out to be different from what was assumed).

The second axis is the **delivery methodology**: how the work is sequenced and how learning enters the plan. Waterfall learns once, at the end of each phase, and treats a late change as a defect in the requirement. Iterative methods learn continuously and treat a late change as information. Hybrid methods learn at defined boundaries. Design-led methods learn through artefacts that stakeholders can react to. Value-led methods learn through measured outcomes on a live system.

The two axes look independent because they are usually chosen by different people at different times - procurement chooses the form, the vendor's practice chooses the method - and because the vocabulary does not overlap. They are not independent. Both are answers to one question.

### 2.2 Knowability Is the Only Variable That Matters

Ask of any commercial form: *what does this form assume is known at signature?* A fixed price assumes scope and effort are knowable now, to within a tolerance the supplier is willing to carry. Time and materials assumes they are not knowable now and will be discovered, so the client keeps the risk and pays for the discovery. Outcome-based assumes the outcome is knowable and measurable now, and that the path to it is not. Managed service assumes the service definition is knowable and that the demand is forecastable within a band. Staff augmentation assumes nothing about the work at all - it transfers none of it.

Now ask of any delivery method: *when does this method assume learning happens?* Waterfall says: at phase gates, and a change after a gate is a scope event. Iterative says: every cycle, and a change is an input to the next cycle. Discovery-led says: before the plan is fixed, in a distinct phase whose output is the plan.

Put the two answers side by side and the coherence test is mechanical. If the form assumes the answer is known now and the method assumes the answer arrives later, the engagement is financed on a belief the method is designed to disprove. That is the entire argument, and it does not depend on any claim about which form or which method is better.

There is a second-order version worth naming, because it is where experienced people get caught. A time-and-materials contract paired with waterfall is not incoherent in the same way - both can coexist - but it is *wasteful in a specific direction*: the client has paid for the option to change (that is what time and materials buys), and the method delays the moment at which the option can be exercised until the phase gate, by which time the cost of change has risen. The client holds an option and the method refuses to let them exercise it while it is cheap. Coherence is not only about contradiction; it is also about whether the instrument you bought can actually be used.

### 2.3 The Pairings, Argued One by One

**Fixed price + waterfall.** Coherent, and the most common coherent pairing in regulated delivery. Both assume knowability. The form puts estimation and scope risk on the supplier; the method front-loads requirement definition, which is what makes the fixed price defensible. The failure mode is not incoherence but *brittleness*: the price is only as good as the requirement document, and the change mechanism becomes the whole relationship. This pairing is right when the requirement is genuinely stable - a regulatory reporting change with a fixed taxonomy, a platform upgrade with a known target state - and wrong when the requirement is stable only in the document.

**Fixed price + discovery-led.** The incoherent pairing, and the one this guide is named after. The price is set before discovery and the method's first act is to discover things that change the price. There are only three ways this ends: discovery is run as confirmation and produces nothing that could change the price (so the client paid for a phase that was designed not to find anything); discovery finds something and the price must be renegotiated, which converts the fixed price into a negotiation with a sunk-cost anchor; or discovery finds something and nobody renegotiates, so the supplier delivers the original scope at a loss and the quality of everything after discovery absorbs it. All three are the same error: the risk was bought twice, once in the contingency inside the price and once in the rework after discovery.

**Time and materials + waterfall.** Not contradictory, but self-defeating in the way described above. The client bears both risks and buys the right to change course, then adopts a method that makes changing course expensive. This pairing usually appears when a client wants control (waterfall gates, sign-offs) and flexibility (time and materials) at the same time, and has not noticed that the two desires are in tension. It is defensible only when the gates are being used for funding release rather than for change control.

**Time and materials + agile.** Coherent. The client bears estimation and scope risk, and the method gives them the instrument to steer - a backlog they can reorder, a cadence at which they can stop or redirect. The failure mode is governance, not design: without a steering committee that actually trades scope, time and materials plus agile becomes an open-ended burn with a demo every two weeks. The form is honest; the discipline has to be supplied.

**Capped time and materials + agile.** Coherent on the way up and deceptive at the boundary. Below the cap, the client bears estimation risk and the method gives them steering. At the cap, the arrangement silently converts into a fixed-price engagement with none of a fixed price's protections: the supplier stops, the client must choose between more funding and less scope, and the choice is made under time pressure with a half-built product. The cap is a real instrument, but it requires the governance to pre-agree what happens at the boundary - which scope is sacrificed, decided in advance, not at the moment of exhaustion.

**Outcome-based + iterative.** Coherent, and the most demanding pairing to make work. Outcome-based requires a measurable baseline and an attribution rule at signature; iterative delivery is what makes the outcome reachable when the path is unknown, because it lets the engagement steer toward the metric. The failure mode is measurement, not method: if the outcome metric is one the supplier does not control (revenue, customer retention, a regulator's timing), the supplier has accepted a risk they cannot manage, and will either price it as a lottery or manage it by arguing about attribution. Outcome-based is coherent with iterative delivery only when the outcome is defined as something the engagement's own outputs move.

**Managed service + waterfall.** Coherent if the two are separated in time, incoherent if they are conflated. A managed service is a steady-state promise over a term; the transition into it is a project, and a waterfall transition into a service is entirely normal. The mistake is applying the project mental model to the service term - treating the service as a long deliverable rather than as a standing obligation with a service definition, a catalogue and a variation mechanism.

**Retainer + continuous.** Coherent: a retainer buys availability and access over time, and continuous advisory is what availability is for. The failure is a retainer used to fund a project without project discipline - work arrives as a queue, nobody owns the queue, and the retainer becomes an unmeasured subscription. A retainer needs a demand mechanism as much as a contract.

**Staff augmentation + any method.** Coherent with any method because it transfers no outcome risk at all: the client's own method applies, the client's own plan governs, and the supplier supplies capacity. The clarity is real and it is also the trap - staff augmentation is often chosen precisely to avoid the conversation about knowability, and the conversation does not go away, it just moves inside the client's own plan where nobody is accountable for the estimate. ISO 20700 clause 4.1.3 is unambiguous about where accountability sits in that arrangement: the provider is responsible for its resources and work, and "the ultimate accountability for decisions, outcomes, deliverables and the impacts on stakeholders lies with the client" ✅.

### 2.4 The Pairing Table

| Pairing | Verdict | The risk that gets bought twice, or the option that goes unused |
|---|---|---|
| Fixed price + waterfall | Coherent | None doubled; brittle if the requirement is stable only on paper. |
| Fixed price + discovery-led | Incoherent | Knowability bought twice: contingency inside the price, rework after discovery. |
| Time and materials + waterfall | Coherent but self-defeating | The right to change is bought and then made expensive to use. |
| Time and materials + agile | Coherent | None; the discipline must come from governance. |
| Capped time and materials + agile | Coherent until the cap | At the cap, scope risk reappears with no pre-agreed trade. |
| Outcome-based + iterative | Coherent, measurement-dependent | Attribution risk if the metric is outside the engagement's control. |
| Managed service + waterfall | Coherent if transition and term are separated | The project model applied to a standing obligation. |
| Retainer + continuous | Coherent | Demand mechanism absent, so availability is bought and unspent. |
| Staff augmentation + any | Coherent, because nothing is transferred | The knowability question is not answered, only relocated. |

### 2.5 The Concession

The thesis is about structural failure and nothing else. It says: when an engagement pairs a commercial form that asserts knowability with a method that asserts the opposite, the arrangement contains a contradiction that will surface as rework, renegotiation or silent quality loss. It does not say that most engagements fail for this reason, and it cannot, because no dataset was found that would support such a claim (see section 15.2).

The orthogonal failures are real and common: a sponsor leaves and the successor does not own the outcome; funding is withdrawn or re-phased; a procurement rule changes and the contract must be re-papered; a key person resigns; a regulator changes a deadline; an acquisition redirects the roadmap. In each of those cases the engagement can be perfectly coherent and still fail, and a reader who uses this guide to explain such a failure would be misusing it. The thesis earns its keep only where the failure is traceable to the arrangement itself - and section 10's taxonomy marks, row by row, which causes are structural and which are orthogonal.

### 2.6 What Follows for a Bank Architect

A solution architect in a bank is usually not the person who chooses the commercial form. That is procurement's and the sponsor's decision, and section 11 explains why their approval logic pushes toward fixed price almost regardless of knowability. The architect is, however, almost always the person who knows first whether the thing is knowable - because they are the one who has read the source systems, seen the data, and can tell whether the target state is a configuration or a research project.

That puts the architect in a specific position: the only person in the room able to say, before signature, "this scope is not knowable yet, so either the form changes or the method must be allowed to change it". Saying it is a commercial act, not a technical one, and it is the single highest-leverage sentence available to an architect in an engagement. The rest of this guide is largely about being able to say it with evidence.

## 3. The Engagement Models and What Each Allocates

### 3.1 The Risk-Allocation Table

Everything a commercial form does is in this table. Read it as an allocation, not a menu: the rows are not better or worse than each other, they place two risks on different parties and reward different behaviour as a consequence. The last column is the one people forget - a form does not only allocate risk, it also takes something off the table, and the client who has not noticed what has been taken away will spend the engagement trying to ask for it.

| Model | Bears estimation risk | Bears scope risk | Behaviour it rewards | What it makes the client unable to ask for | Method it implies |
|---|---|---|---|---|---|
| **Fixed price** | Supplier | Supplier, within the stated scope | Precise scoping, contingency, disciplined change control | Exploration, and "more of the same" without a variation | Waterfall, or iterative delivery with a frozen backlog |
| **Time and materials** | Client | Client | Throughput, responsiveness, utilisation | A price certain before the work starts | Any; governance must supply the discipline the form does not |
| **Capped time and materials** | Client up to the cap, supplier above it | Shared, and undefined at the boundary | Burning to the cap, then renegotiating | Unlimited reprioritisation without a boundary decision | Iterative, with a pre-agreed trade at the cap |
| **Outcome-based** | Supplier, against the outcome definition | Shared, and definition-dependent | Measurable outcomes, honest baselines, steering toward the metric | Effort-based changes, and outputs unrelated to the metric | Iterative, with a measurement instrument |
| **Managed service** | Supplier, within the service definition | Supplier, within the catalogue | Service levels, cost-to-serve, automation | Scope beyond the service catalogue without a variation | Continuous service management; a project only for the transition |
| **Staff augmentation** | Client | Client | Filling seats, responsiveness to direction | Any outcome accountability from the supplier | The client's own method, whatever it is |
| **Retainer** | Shared, and usually left ambiguous | Client | Availability, advisory access, responsiveness | A delivery commitment, or a defined output | A continuous advisory cadence with a demand mechanism |

### 3.2 Fixed Price

The supplier converts an uncertain estimate into a commitment and carries the difference. This is the only form in which the client can budget a single number before the work starts, which is why it dominates procurement (section 11.2 explains the mechanism). It is the right form when the requirement is knowable to a tolerance the supplier can price, and when the cost of a late change is genuinely high enough that the discipline of a frozen scope is worth paying for.

Its characteristic failure is not overrun but **scope distortion**: because the supplier carries scope risk only within the *stated* scope, the incentive is to state the scope narrowly and to treat every ambiguity as a variation. The engagement that was supposed to solve a problem delivers a contract-shaped subset of it, and the client discovers at acceptance that the thing they needed was in the gap between the scope and the exclusions. That is why section 5.5 treats exclusions as the decisive clause.

### 3.3 Time and Materials

The client keeps both risks and buys the ability to redirect. The supplier is paid for effort, so the incentive is toward utilisation and responsiveness rather than toward a defensible scope boundary. This is the right form when the work is genuinely exploratory - a diagnostic, an architecture study, an incident-driven remediation - and when the client has the governance capacity to steer, because the form supplies no steering of its own.

Its characteristic failure is **drift without a decision**: work expands to fill the available mandate, no single moment exists at which anyone chooses the scope, and the engagement ends when the budget ends rather than when the problem is solved. Note that this is a governance failure, not a form failure, and it is the reason section 8 exists.

### 3.4 Capped Time and Materials

A cap converts an open-ended arrangement into a bounded one without freezing the scope. Up to the cap, the client keeps both risks and the flexibility. Above the cap, the supplier does not work for free and does not carry the risk, so the engagement stops and a decision must be made.

Its characteristic failure is **the boundary surprise**. Because the cap is usually set as a number rather than as a rule, nobody agrees in advance what happens when it is reached. The decision is then made in the worst possible conditions - the team is mid-build, the deadline is fixed, and the only options are more funding or less scope. The correction is procedural and cheap: write the boundary rule into the SOW (which scope is sacrificed first, who decides, by when) at the same time as the cap. A cap without a boundary rule is a fixed price with worse protections.

### 3.5 Outcome-Based and Value-Based

Payment is tied to a measured outcome rather than to effort or to a deliverable. This transfers estimation risk to the supplier and shares scope risk, because the supplier now has an interest in the outcome being reached rather than in the scope being delivered. It is the right form when three conditions hold: the baseline is measurable before the work starts, the outcome is attributable to the engagement's outputs, and the client will accept the measurement regime the form requires.

Its characteristic failure is **attribution argument**. If the metric is revenue, retention, cost-to-serve or a regulatory date, the supplier has accepted a risk they cannot manage, and the engagement turns into a dispute about what caused what. Outcome-based works when the outcome is defined as something the engagement's own outputs move - a data quality threshold, a cycle-time reduction inside a process the engagement rebuilt, an adoption rate inside a system the engagement delivered - and fails when it is defined as a business result the client's own operation controls. The form is not a substitute for the measurement discipline; it is a bet that depends on it.

### 3.6 Managed Service

The supplier promises a steady state over a term, within a service definition and catalogue. Estimation risk sits with the supplier inside the definition and with the client outside it, which makes the definition of the boundary - what is in the catalogue, what is a variation - the whole commercial argument. This is the right form for anything that will still be running after the project ends: a platform operation, a monitoring capability, a managed integration.

Its characteristic failure is **the definition gap discovered in operation**. The catalogue was written for the steady state the client imagined, the demand turns out to be different, and every difference is a variation. The engagement then has a project's governance arguing about a service's boundary. The correction is to write the demand forecast and the variation mechanism into the contract, not just the service levels.

### 3.7 Staff Augmentation

The supplier supplies capacity and the client retains every risk. This is the cleanest allocation available and the most honest about what is being bought: people, directed by the client, inside the client's plan. It is the right form when the client has the method, the plan and the management capacity, and needs throughput rather than answers.

Its characteristic failure is **accountability laundering**. Because the form transfers no outcome risk, it is often chosen to avoid the knowability conversation entirely - and the estimate then lives inside the client's own plan, owned by nobody, with a supplier who is contractually correct and operationally useless when the plan is wrong. Staff augmentation with a client who has no plan is not a low-risk form; it is a form that hides where the risk went. ISO 20700 clause 4.1.3 puts the accountability exactly there, on the client, which is honest but not comfortable ✅.

### 3.8 Retainer

The client buys access and availability over a period. This is the right form for advisory work whose demand cannot be forecast - a standing architecture council, a regulatory change watch, an independent review function - and it is the only form that prices availability rather than output.

Its characteristic failure is **the unmeasured subscription**. The retainer is signed, the calendar fills with meetings, and nobody can say what the availability produced. A retainer needs an explicit demand mechanism (who can call on it, for what, with what notice) and a periodic review of what the access actually bought. Without those, it degrades into a relationship subsidy.

### 3.9 What the Table Cannot Show

Three things sit outside the table and each is a warning about using it mechanically.

First, **the forms combine**. A single engagement is often a managed service with a fixed-price transition project inside it, or a time-and-materials discovery leading into a fixed-price build. That composition is legitimate and common; what is not legitimate is composing it without noticing that the discovery is being paid for by a form that has already committed to its answer. The composition is where the double-buy of section 2.3 usually happens.

Second, **the FDE variant does not appear as a row because it is not a commercial form**. Forward-deployed engineering is a delivery model - an embedded pod, discovery through deployment to adaptation to handover - and it is sold under several of the forms above. That guide uses statement-of-work language only to contrast FDE ("beyond the SOW") with professional services ("SOW compliance"), which is a claim about the working relationship, not about risk allocation (see [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md), sections 3.2 and 6). This guide does not re-derive it; the point of the cross-reference is that a delivery model can be coherent with several commercial forms, and the coherence test still has to be run per engagement.

Third, **the table describes the contract, not the behaviour**. A fixed price with generous change control behaves like time and materials with a paperwork step; a time-and-materials engagement with a hard governance ceiling behaves like a cap. The instrument on paper and the instrument in practice diverge, and the divergence is where section 8's economics live.

---

## 4. The Lifecycle, Phase by Phase

### 4.1 The Seven Phases and Their Artefacts

A phase is not a period of time. It is a decision that must be taken before the next phase can start, and it is only real if it has an artefact that the decision can be taken on. The table below is the lifecycle in that form - the artefact each phase must produce, the decision that artefact enables, and the failure that appears when the phase is skipped.

| Phase | Artefact | Decision it enables | The failure that skips it |
|---|---|---|---|
| **Pursuit and qualification** | A qualification note: the problem, the buyer, the budget reality, the knowability assessment | Whether to bid at all, and under which form | Bidding on everything, then discovering at proposal stage that the scope is unknowable and the buyer has no budget |
| **Proposal and RFP response** | The response: approach, team, plan, assumptions, and the commercial form proposed | Which form and method are being offered together | Answering the RFP's form mechanically, so the form is inherited rather than chosen |
| **Contract and SOW** | The signed instrument: scope, exclusions, acceptance criteria, dependencies, change control, milestones | What is actually owed, and by whom, and what happens when reality diverges | Signing a contract whose schedule references a scope nobody wrote down |
| **Discovery** | The findings: current state, data reality, constraints, decision rights, baseline, options with feasibility | Whether the plan, the estimate and the form survive contact with the facts | Compressing discovery to protect a price (section 6.3) |
| **Delivery** | Working increments, decisions log, risk register, change requests | Whether to continue, re-scope, escalate or stop | Reporting progress against a plan that discovery invalidated |
| **Handover and transition** | The transition pack, the run-capability evidence, the named successor, the warranty terms | Whether the client can operate without the team | Treating handover as a document drop (section 9) |
| **Close and lessons** | Acceptance record, closure of open items, lessons note | Whether the engagement is complete and what the next one inherits | Closing on delivery rather than on acceptance, so the last items are never resolved |

Read the table down the "failure" column and a pattern appears: every skipped phase produces a failure that surfaces two phases later, in a different vocabulary, and gets attributed to the wrong cause. A skipped qualification shows up as an unpriced discovery. A skipped discovery shows up as rework in delivery. A skipped handover shows up as a benefit that evaporates a quarter after the consultants leave. That two-phase lag is why post-mortems misdiagnose so consistently.

### 4.2 Pursuit, Qualification, Proposal and RFP Response

Qualification is the phase where the knowability question is cheapest to ask and most often avoided. The qualification note should carry four things: the problem as the buyer states it, the buyer's actual authority and budget position, an honest assessment of whether the scope is knowable before the work starts, and the commercial form that assessment implies. The last two are the ones that get dropped, because they are the ones that might end the pursuit.

The proposal is where the form and the method are offered together, and where an RFP has usually already fixed one of them. A bank RFP typically specifies the contract type in the commercial section and asks bidders to propose a methodology in the technical section, which structurally guarantees that the two are chosen separately - the exact error this guide is about. The bidder's only defence is to make the coherence argument inside the response: to state, in the technical section, what the proposed method assumes about knowability, and to state, in the commercial section, what the proposed form assumes, and to show that the two match. A bidder who does that either wins on coherence or loses to someone who will discover the mismatch later. That is a real commercial risk and it is not this guide's job to resolve it; the point is that the argument exists and can be made in writing.

The proposal's assumptions section is the seed of the SOW's exclusions section, and the two should be written by the same person. Most of the exclusions that save an engagement at month seven were visible as assumptions at bid stage and were softened because they might have cost the bid.

### 4.3 Contract and Statement of Work

The contract carries obligations, liability, termination, and - in a bank - the third-party risk apparatus of section 12. The SOW carries the work. The two are usually drafted by different functions, reviewed by different committees, and never read side by side, which is how a contract can promise "the services described in Schedule 2" while Schedule 2 promises outcomes that the SOW's acceptance criteria cannot test.

The decision this phase enables is narrow and essential: *what exactly is owed, and what is explicitly not owed*. Everything else in the SOW is machinery for answering that question when it is disputed. Section 5 takes the SOW apart clause by clause; here it is enough to note that this phase is the last point at which the commercial form can be changed cheaply, and that after signature the only remaining instrument is change control.

### 4.4 Discovery and Delivery

Discovery is treated in full in section 6, so this section only fixes its place in the sequence: it sits between the contract and the plan, which is exactly the position that makes a fixed price incoherent with it. Delivery is the phase where the method becomes visible: increments, decisions log, risk register, and change requests. The one discipline worth stating here is that the **decisions log and the change log are different documents** - a decision is a choice made inside the agreed scope, a change is a modification of it - and merging them is how an engagement loses the ability to say whether it is on plan.

### 4.5 Handover, Close and Lessons

Handover is section 9. Close is the phase almost everyone treats as administrative and almost nobody gets right, and ISO 20700 clause 7 makes it a substantive obligation rather than a formality: after completion, including delivery of all deliverables and assessment of the results, the assignment is closed by resolving unfinished items and meeting all contractual and legal obligations, and "It is critical that the MCSP gains acceptance from the client that the assignment is complete" ✅. Two things follow. Acceptance is a positive act by the client, not the absence of complaint. And closure requires the unfinished items to be resolved, which means the close phase has to be planned with time in it - an engagement that ends on the last day of delivery has no closure, only a stop.

The lessons note is the artefact that makes the next engagement cheaper, and it is the one with no owner. A useful lessons note answers three questions: which assumptions in the SOW turned out to be false, which of them were knowable at bid stage, and what the qualification note should have said. That is a knowability audit, not a blame exercise, and it is the only mechanism by which an organisation learns to price the second engagement better than the first.

### 4.6 The Skipped-Phase Pattern

The phases are skipped in a predictable order, and the order is informative. Qualification is skipped when the pipeline is thin. Proposal assumptions are softened when the competition is strong. Discovery is compressed when the price is already quoted. Handover is cut when the next engagement is starting. Close is cut when the budget line is exhausted.

Every one of those is a decision made under a different pressure from the one the phase exists to relieve, which is why the phase loses. The structural correction is not discipline in the abstract but **making the phase an artefact with an owner and an acceptance criterion** - a discovery with a defined output and a named acceptor cannot be compressed to nothing without someone visibly refusing to accept it. Section 6.3 makes the same argument for discovery specifically, because it is the phase where the cost of skipping is largest and least visible at the time.

## 5. The SOW as an Instrument

### 5.1 What a Statement of Work Is and Is Not

A statement of work is the conversion of a commercial form into a testable obligation. It is not a proposal, not a project plan, and not a schedule of activities. Its job is to make four things unambiguous: what will be delivered, what will not, how "done" will be tested, and what happens when reality diverges from the document.

The test for whether a SOW is doing its job is simple and brutal: hand it to two people who were not in the negotiation and ask them whether a specific disputed item is in or out. If they disagree, the SOW is not an instrument, it is a record of an intention. Most SOWs reviewed after a dispute fail this test on the item that is being disputed, which is not a coincidence - the ambiguity was not an accident, it was the price of agreement.

### 5.2 The Clause Set

Nine clauses do the work. The set is standard documented practice, not a standard's requirement - no clause list is published in either SS 680 or ISO 20700, and any claim that a particular clause order is mandated would be false.

| Clause | What it fixes | What happens without it |
|---|---|---|
| **Scope** | The boundary of the work, described as outcomes and covered entities, not as activities | Scope is whatever the reader remembers; activity lists get read as entitlements |
| **Assumptions** | The conditions the estimate depends on, stated as falsifiable propositions | Assumptions become implicit, so when one fails nobody can say it failed |
| **Exclusions** | What is explicitly not included | The gap between the scope and the client's expectation is filled by argument at delivery |
| **Deliverables** | The named outputs and their form | Deliverables become whatever was produced, and acceptance has no object |
| **Acceptance criteria** | The test for each deliverable, written before it is built | Acceptance becomes a person's opinion, exercised late and under pressure |
| **Dependencies** | What the client must supply, and when, and what happens if they do not | Client-side delays become supplier-side slippage with no record of the cause |
| **Payment milestones** | What event releases what portion of the contracted value | Progress becomes a matter of narrative rather than of evidence |
| **Change control** | How a divergence is raised, assessed, decided, and recorded | Change is absorbed silently or fought over informally; both destroy the plan |
| **Termination** | The conditions and consequences of ending early | Exit is either impossible or catastrophic, which is a risk in itself (section 12.3) |

### 5.3 Assumptions and Dependencies

An assumption is a proposition the estimate depends on, stated so that it can be shown false. "The source system extract is available by the end of the first month" is an assumption. "The client will support the project" is not - it cannot be falsified, so it cannot be invoked. The discipline is to write assumptions in the falsifiable form and then to *track* them: an assumptions register that is reviewed at each steering committee converts the SOW's assumptions from a legal shield into an early-warning instrument. The legal shield is still there when needed; the point is that it should rarely be needed, because the assumption was being watched.

Dependencies are the subset of assumptions that require an action by the client. They deserve their own clause because they carry a consequence: if the client owes an input and does not supply it, the schedule must move, and the SOW has to say so before the slippage happens rather than after. A dependency clause with no stated consequence is a note, not a clause.

### 5.4 Acceptance Criteria

Acceptance criteria are the only part of the SOW that gets tested, so they are the only part that ultimately matters. Three rules make them usable.

First, **write them before the build, not at acceptance**. Criteria written at acceptance describe what was built. Criteria written at signature describe what was needed, and the difference between the two is the entire value of the clause.

Second, **make each criterion binary**. "The reconciliation report is accurate" is not a criterion; "the reconciliation report matches the control total for each of the defined account classes on each of the defined test dates, with exceptions listed and explained" is. A criterion that requires judgement at the moment of acceptance will be judged by whoever has the least time.

Third, **name the acceptor and the evidence**. Acceptance criteria without a named acceptor default to whoever is in the room, which in a bank is usually someone from a different function with no stake in the outcome. The clause should say who accepts, on what evidence, within what period, and what happens if no decision is made in that period - silence must not be acceptance by default, and it must not be indefinite rejection either.

There is a fourth rule that is really a warning: acceptance criteria are where a fixed-price engagement hides its quality risk. If the criteria are weak, the supplier can meet them and deliver something the client cannot use, and the contract will be satisfied. Weak acceptance criteria do not make an engagement easier; they move the failure from the delivery phase to the benefit phase, where it is attributed to the client.

### 5.5 Exclusions: Where Engagements Are Won or Lost

The proposal is won on scope and the engagement is lost on exclusions. That asymmetry is worth stating plainly because it explains a behaviour that otherwise looks irrational: bidders soften exclusions to win, and the softening is invisible at award and decisive at delivery.

An exclusion does three things at once. It **bounds the price** - the supplier can only commit to a number because the number excludes what has not been estimated. It **bounds the argument** - when the client asks for the excluded thing, the conversation is about a variation rather than about whether the thing was ever included. And it **records the disagreement** - a well-written exclusion is a sentence that says, in effect, "we know you want this and we are telling you now that it is not in the price". That last function is the one that wins engagements rather than losing them, because the client who sees the exclusion at bid stage either funds it or accepts the gap, and either way does not discover it at acceptance.

The structural argument is this: scope is what the client asked for, exclusions are what the supplier cannot promise, and the engagement fails in the space where the two were never reconciled. Exclusions are the only clause that forces that reconciliation to happen before signature. A SOW with a strong scope section and a thin exclusions section is a document that has agreed what will be done and deferred the question of what will not.

Two practical tests for an exclusions section. Is every exclusion paired with the assumption it protects? (An exclusion with no assumption behind it is a negotiating position, not a boundary.) And would the client recognise each exclusion as something they had asked for? (An exclusion the client never wanted is noise that dilutes the ones that matter.)

### 5.6 Payment Milestones and Change Control

Payment milestones do two jobs. The obvious one is cash timing, which is out of scope here. The structural one is that **milestones define what "progress" means**, and an engagement whose milestones are dates rather than events has no way to distinguish progress from passage of time. Milestones tied to acceptance-tested deliverables make the payment schedule an independent check on the delivery narrative, which is why suppliers prefer date-based milestones and clients should not.

Change control is the mechanism that keeps the SOW alive. A workable change clause has five parts: how a change is raised (a written request, by whom), how it is assessed (by when, against what criteria), who decides (a named role, not a committee), how it is recorded (a register, with the decision and its effect on scope, schedule and acceptance), and what happens if no decision is made (a default, usually that the change is not approved and the original obligation stands). The last part is the one that is almost always missing, and its absence is why change registers fill with items that were discussed and never decided.

### 5.7 Reading a SOW as a Risk Document

Read a SOW four times with four questions, and the document's real position becomes visible.

1. **Read the exclusions first.** They tell you what the supplier is afraid of, and therefore what the estimate does not cover.
2. **Read the acceptance criteria second.** They tell you what "success" will actually be tested against, and therefore what quality the supplier has committed to rather than promised.
3. **Read the dependencies and their consequences.** They tell you which risks have been moved to the client, and whether the client has noticed.
4. **Read the change clause last.** It tells you what happens when the first three turn out to be wrong, which they will.

A SOW that survives those four readings is a risk allocation. A SOW that does not is a purchase order with paragraphs.

---

## 6. Discovery as the Pivotal Phase

### 6.1 What Discovery Must Produce

Discovery is not "understanding the client". It is a bounded phase that produces specific artefacts, each of which changes a decision that would otherwise be taken blind. If a discovery phase ends without these, it was a series of meetings.

| Output | The decision it changes |
|---|---|
| **Current-state map** - systems, processes, data flows, and the interfaces that will hurt | Whether the target architecture is a modification or a replacement, and therefore whether the estimate is the right order of magnitude |
| **Data reality** - what exists, in what quality, owned by whom, with what latency | Whether the migration or integration is a build task or a remediation project, which is the single biggest source of estimate error in bank work |
| **Constraint register** - regulatory, contractual, platform, and political constraints | Which options survive, and which parts of the plan are not negotiable |
| **Decision-rights map** - who can approve what, and who can stop what | Whether the governance in the SOW is real, and who must be in the steering committee for it to have authority |
| **Baseline** - the measured starting point for whatever the engagement intends to change | Whether the outcome claimed at the end is a change or a coincidence |
| **Options with feasibility** - at least two, with the evidence that distinguishes them | Whether the plan is a choice or a rationalisation of a decision already taken |
| **Estimate revision** - the plan and estimate restated against what was found | Whether the commercial form still holds, which is the point of the whole phase |

The last row is the one that makes discovery a commercial instrument rather than a technical one. A discovery that does not revise the estimate has not been allowed to find anything.

### 6.2 What Discovery Costs, in Time and Access

Discovery is not free and its cost is not primarily financial. It costs three things, and each is a reason it gets cut.

**Elapsed time.** Discovery sits on the critical path before delivery and produces nothing a stakeholder can see working. In a programme with an announced date, elapsed time before visible progress is the most expensive currency there is.

**Access.** Discovery needs time with people who are busy and who do not report to the engagement: source-system owners, data stewards, operations staff, control functions, and the users who know what the process actually does rather than what the procedure says. Access is a political resource, and it is granted in inverse proportion to how early in the programme it is requested.

**Political capital.** Discovery can find that the sponsor's preferred option is not feasible. That is the phase working correctly, and it is also the reason sponsors who have already announced an option are reluctant to fund a phase designed to test it.

The honest accounting is that discovery's cost is front-loaded, visible and attributable, while its benefit is deferred, invisible and counterfactual ("we avoided the rework"). That asymmetry is enough to explain its fate without any appeal to bad faith.

### 6.3 The Compression Finding

The finding, stated as plainly as it can be: **discovery is the phase most often compressed to protect a price that was already quoted.** The mechanism is not mysterious. Once a price exists, discovery has only three possible outcomes - it confirms the price, it raises it, or it lowers it. Two of the three are commercially awkward, and the third is rare. So the phase is scoped down until its most likely outcome is confirmation, and the organisation tells itself it is being efficient.

The tell is structural, and it is visible in the artefacts rather than in the behaviour. A compressed discovery has these properties:

- **Its acceptance criterion is a document, not a finding.** It ends when the report is delivered, not when a decision changes.
- **Its duration is set by the delivery plan rather than by the questions.** It is sized to fit the gap before build starts.
- **Its team is the delivery team.** The people who will build are the people who scoped, so the discovery's incentive is to confirm the scope they will be measured against.
- **Its estimate revision column is empty.** No option is ever priced as infeasible.

Any one of those is survivable. All four together mean discovery is a ritual, and the engagement is being delivered on a plan that nobody tested.

The correct framing for a sponsor is not "discovery reduces risk", which is unfalsifiable and sounds like a request for time. It is: "discovery exists to tell you whether the price you have been quoted corresponds to the thing you asked for, before you are committed to it". That is a question every sponsor wants answered and almost none is offered the chance to ask.

### 6.4 Discovery Under a Fixed Price

The specific pathology of discovery inside a fixed-price engagement deserves its own treatment, because it is the clearest instance of the thesis and it is extremely common in bank programmes.

Under a fixed price, the supplier has converted an uncertain estimate into a commitment. Discovery, if run honestly, is the phase most likely to produce the information that makes the commitment unprofitable. The supplier therefore has three options, and all three are rational from inside the arrangement:

1. **Run discovery as confirmation.** The phase is staffed, documented, and produces a report that supports the price. Nobody is lying; the questions asked are simply the ones whose answers are already known.
2. **Run discovery honestly and renegotiate.** The findings are real, the price must move, and the renegotiation happens with the client already committed and the supplier already sunk. The relationship absorbs the cost of the original mispricing.
3. **Run discovery honestly and absorb the difference.** The supplier delivers the original scope at a loss, and the shortfall is recovered from quality, from team continuity, or from the next engagement. The client gets the scope and not the outcome.

Every one of those outcomes is a consequence of the pairing, not of anyone's competence. The fix is structural and has two forms. **Separate the discovery from the commitment**: a priced, separately contracted discovery phase whose output is the input to the fixed price, so the price is set after the information exists. Or **let the form follow the method**: keep discovery inside the engagement but make the commercial form one that can absorb what it finds - a capped time-and-materials discovery followed by a fixed-price build, with the boundary written into the SOW. Both are the same idea, which is that the price must be allowed to be a consequence of the discovery rather than a constraint on it.

## 7. The Methodologies and When Each Is Wrong

### 7.1 Waterfall

Waterfall assumes the requirement can be specified before it is built, that the cost of change rises steeply with time, and that a phase gate is a genuine decision point. When those hold, it is not old-fashioned - it is correct, and it is the only method that produces a defensible fixed price.

It is a mistake in four conditions. When **the requirement is a learning instrument rather than a specification** - the client does not yet know what they need, so the specification phase produces a document that encodes the client's current guess with the authority of a signed-off requirement. When **the artefact is exploratory** - a diagnostic, an options appraisal, an architecture study - because there is nothing to waterfall through. When **the gates are ceremonial** - the phase document is approved by people who will not be held to it, so the gate transfers no accountability and only adds elapsed time. And when **the change mechanism is absent**, because waterfall without change control does not prevent change, it just makes every change a crisis.

The tell that waterfall is being used wrongly is a requirement document that nobody defends. If the business sponsor cannot answer "is this still what you want" without deferring to the analyst who wrote it, the specification is a artefact of the method rather than an input to it.

### 7.2 Iterative and Agile

Iterative delivery assumes the requirement will be discovered, that a working increment is a better basis for a decision than a document, and that someone with authority will reorder the work as learning arrives. When those hold, it is the only method that converts delivery into steering.

It is a mistake in five conditions. When **nobody has the authority to reorder** - a product owner who cannot say no is a backlog administrator, and the method degrades into waterfall with shorter, more frequent status reports. When **the work is an irreversible cutover** - a core migration, a regulatory reporting switch - where the value of a small increment is low and the cost of a half-migrated state is high; iterative planning can still surround such a cutover, but the cutover itself is a single event. When **the contract has no capacity for re-prioritisation**, because then every sprint boundary is a change request and the cadence becomes a negotiation cycle. When **"agile" is being used to avoid committing to scope** in a fixed-price deal - the form promises a fixed scope and the method promises to discover it, which is the double-buy of section 2.3 wearing a ceremonial costume. And when **the team is staff augmentation without an empowered client owner**, because then the method's central mechanism has no operator.

The tell is the same in all five: the team can describe its cadence but cannot describe a decision the cadence produced. A cadence that has never changed the order of work is a reporting rhythm.

### 7.3 Hybrid

Hybrid is treated here as a method rather than as a compromise, because it has its own failure mode. The legitimate form is **a boundary between kinds of work**: a waterfall transition and a fixed target state for the irreversible parts, iterative delivery for the parts whose requirements are still forming, and an explicit interface between them. That is coherent and common in banks, where the migration of a ledger and the building of the surrounding services have genuinely different knowability.

It is a mistake when it is **the absence of a method with two governance bodies**. The symptoms are two definitions of done, two sets of phase gates, a team that reports to both, and a plan that cannot be reconciled because the iterative half has no end date and the waterfall half has no flexibility. Hybrid chosen as a compromise between two camps is not a method; it is a truce, and truces expire.

The test: can someone draw the boundary and say which work is on which side, and why? If the boundary is "whatever the vendor does is agile and whatever we do is waterfall", the boundary is organisational, not technical, and it will produce the worst of both.

### 7.4 Design-Led

Design-led methods assume that the constraint is understanding what the user needs, that artefacts stakeholders can react to produce better requirements than documents, and that the experience is where the value is. They are right surprisingly often in customer-facing work, and they are the correct method when the requirement is genuinely unknown and the cost of building the wrong thing is high.

It is a mistake in three conditions. When **the problem is a control or compliance obligation** rather than an experience - a reconciliation, a reporting change, a limit check - where the requirement is externally specified and prototyping it is a way of avoiding the specification. When **prototyping outruns the ability to industrialise** - the design is validated, the pilot delights, and the platform cannot run it at volume, so the engagement delivers a proof of concept and calls it a product. And when **the design phase becomes the deliverable**, because a design system, a journey map and a prototype can absorb an entire engagement and leave nothing in production.

The tell: the engagement can show a beautiful artefact and cannot show an operational process.

### 7.5 Value-Led Change

Value-led change assumes the benefit is measurable, that the benefit owner is inside the engagement, and that changing behaviour is part of the work rather than an afterthought. It is the right frame when the engagement's purpose is a measurable shift in how an organisation operates, and it is the only frame that makes handover meaningful, because the benefit has to survive the team leaving.

It is a mistake in three conditions. When **the baseline is unmeasurable** - the benefit was asserted in the business case and never measured before the work started, so nobody can tell afterwards whether it moved. When **the benefit owner is not in the room** - the engagement is accountable for a number that another function controls, which is the same attribution problem as outcome-based pricing (section 3.5), arriving through the method rather than the contract. And when **it becomes a change-management programme bolted on after go-live**, which is the most common form of the error: the technology is delivered by one team, the behaviour change is commissioned from another, and the second starts when the first ends, which is precisely when the organisation's attention has moved on.

### 7.6 The Selection Test

The method is not a preference and not a house style. It is a claim about when learning happens, and it has to be tested against the same question the commercial form answers.

| Question | If the answer is yes | If the answer is no |
|---|---|---|
| Can the requirement be specified to a tolerance someone will commit to? | Waterfall, or a frozen-backlog iterative delivery | Discovery first, then iterative |
| Can the work be delivered in increments that are individually useful? | Iterative | Waterfall, with iterative planning around it |
| Is there an owner with authority to reorder the work? | Iterative is viable | Iterative will degrade into reporting |
| Is the change irreversible at a single point? | Sequence the irreversible part as an event; iterate around it | Iterative throughout |
| Is the benefit measurable and owned inside the engagement? | Value-led framing is honest | Value-led framing is a claim nobody can test |

A method chosen without that table having been filled in is a method inherited from the last engagement, and the last engagement's knowability is not this one's.

---

## 8. Governance and the Economics of Scope Creep

### 8.1 The Three Instruments

Governance in an engagement is three instruments, and they are usually confused with three meeting types.

**The steering committee** owns trade-offs: scope against time against quality against funding release. It is the only body with the authority to change the engagement's shape, and it exists because the delivery team cannot make those trades and the sponsor should not make them alone.

**Change control** is the mechanism that converts a divergence into a decision with a record. It is not a committee; it is a process with a named decider, a register, and a default when no decision is made.

**Escalation** is the path a decision takes when the level that owns it cannot decide. It needs a time bound, because an escalation without a deadline is a decision that has been deferred to whoever runs out of patience first.

The instruments are distinct, and an engagement that has a steering committee but no change control has a body that meets monthly and no mechanism between meetings; an engagement with change control but no steering committee has a mechanism with nobody to escalate to when the trade is genuinely above the delivery lead's authority.

### 8.2 The Steering Committee: What It Can and Cannot Decide

A steering committee can decide four things: whether to release the next tranche of funding, whether to change the scope, whether to change the sequence, and whether to stop. It cannot decide how to design something, and it should not try - a committee that designs is a committee that has stopped governing, because it has consumed its authority on a decision that should have been made two levels down and has none left for the trade it exists to make.

The design of a working steering committee is unglamorous and specific:

- **A named chair with authority to decide, not to convene.** A chair who must consult after the meeting is a chair whose committee does not decide.
- **A standing agenda of trades, not of status.** Status is a paper. The agenda item is "what must be traded this month and what is the recommendation".
- **One decision log, shared with the delivery team.** The most common governance failure is a committee whose decisions are recorded in minutes that the team never reads, so the same trade is re-argued at the next meeting.
- **An explicit escalation route upward** for the trades the committee cannot make, with a date attached.

The failure mode to name is the **status committee**: senior people, a monthly cadence, a deck of red-amber-green, no decisions. It reads as governance because the room is senior and the meeting happens. It produces visibility without authority, which is worse than no committee, because it creates the impression that the trade-offs have been considered.

### 8.3 Change Control and Escalation

Change control fails in two directions, and both are common.

**Change control that is absent** means divergence is absorbed. The team does the extra work, the plan slips quietly, and the slippage is attributed to delivery performance rather than to the change. In a fixed-price engagement this is the supplier absorbing a mispriced scope; in a time-and-materials engagement it is the client paying for work they never chose.

**Change control that is weaponised** means every ambiguity becomes a variation, the relationship becomes a series of negotiations, and the client experiences the contract as a series of small defeats. This is what a supplier does when the fixed price was mispriced and the only route back to a viable engagement is the change log.

The workable middle is procedural: a change is raised in writing, assessed within a stated period against stated criteria (does it change the scope boundary, the acceptance criteria, or the dependencies), decided by a named role, and recorded with its effect on scope, schedule and acceptance. The register is reviewed at every steering committee, so that the accumulation is visible as a trend rather than discovered as a total.

Escalation needs the same specificity. A ladder with three levels and no time bounds is a description of an organisation chart. A ladder that says "if the delivery lead and the client owner have not agreed within five working days, the decision goes to the steering committee chair, who decides at the next meeting or in writing before it" is a mechanism. The time bound is the whole clause.

### 8.4 Scope Creep Is a Pricing Problem Wearing a Delivery Costume

This is the section's argument and it is worth stating as a proposition: **scope creep is usually a pricing problem wearing a delivery costume.** The costume is a delivery narrative - weak requirements, poor change discipline, an accommodating team, a client who keeps adding. The pricing problem underneath is that the scope which was priced was not the scope which was knowable, and the difference has to go somewhere.

Follow the difference. If the engagement is fixed-price and the supplier absorbs it, the difference is paid in quality, in team continuity, or in the next engagement's price. If the engagement is time-and-materials and the client absorbs it, the difference is paid in a budget that ends before the problem does. If the engagement has change control, the difference is paid in variations, which is the honest version - the repricing is explicit and the client chooses it. In all three cases the same quantity is being settled; only the mechanism differs.

That reframing changes what a guardrail should be. The conventional guardrail is discipline: better requirements, stricter change control, a firmer project manager. Discipline helps, but it treats the symptom. The structural guardrails are these three:

1. **Price after discovery, or price a discovery.** The gap between priced scope and knowable scope is the source of the creep. Shrinking the gap shrinks the creep, and there are only two ways to shrink it: learn before pricing, or price the learning as its own phase.
2. **Make the change mechanism cheap and neutral.** A change process that costs a week and a negotiation per item will be bypassed. A process that is a form, a named decider and a two-day turnaround will be used, and a used change log is a repricing instrument that both parties can see.
3. **Govern the trend, not the item.** One change is a decision; twelve changes in a quarter is a signal that the original scope was wrong. The steering committee's job is to read the trend and re-baseline, which is a decision only it can take - and one it almost never takes, because re-baselining is an admission.

The reason this matters to an architect more than to anyone else is that the trend is visible in the technical artefacts first. The interfaces that were not in the exclusions, the data that was worse than the assumption, the control that nobody mentioned - these appear in design long before they appear in a change log. An architect who reports the pattern rather than the item is doing governance work, whether or not they are in the governance meeting.

## 9. Handover

### 9.1 What a Transition Actually Transfers

Handover is not the delivery of a document set. It is the transfer of four things, and the fourth is the one that is almost always missing.

**Knowledge** - not what the system does, but why it does it that way: the decisions that were taken, the options that were rejected, the constraint that made the ugly part necessary. Knowledge lives in the people who made the decisions, which is why a handover pack cannot contain it and why the only reliable transfer is time spent with those people before they leave.

**Decision rights** - the authority to change the thing. A team that can operate a system but cannot approve a change to it has not received the system; it has received custody of it, and every change will route back to whoever still holds the authority, which is usually the departing supplier.

**Run capability** - the ability to detect, diagnose and recover: monitoring that means something, runbooks that were used rather than written, an on-call path that has been exercised, and a rollback that has been tested.

**Ownership** - a named person in the client organisation who is accountable for the thing after the engagement closes. Not a team, not a function, a name. Without it, the other three decay at the rate of staff turnover.

### 9.2 The Six Requirements

| Requirement | What it means in practice | How it is evidenced |
|---|---|---|
| **Knowledge transfer** | Structured sessions with the people who made the decisions, plus a decision log that records why | The successor team can explain a past design choice without the supplier in the room |
| **Decision rights** | Named client roles hold the authority to approve changes to the delivered system | A change is raised and approved after the engagement closes, with no supplier involvement |
| **Run capability** | Monitoring, alerting, runbooks, on-call path and rollback, all exercised | A simulated incident is handled by the client team alone |
| **Documentation** | As-built design, interface contracts, data lineage, operational procedures | An engineer who has never seen the system can trace a transaction end to end from the documents |
| **Warranty period** | A defined window after acceptance in which defects are corrected under the engagement's terms | The window is written into the SOW, with a definition of defect and a response path |
| **Named successor owner** | A person, in the client organisation, accountable for the system after close | The name is in the SOW and in the closure record, and the person has signed for it |

Read the "evidenced" column and the difference between a handover and a document drop becomes obvious. Every row is a test that can only be passed by the client doing something without the supplier. A handover pack cannot pass any of them, which is why the pack is necessary and never sufficient.

### 9.3 The Warranty Period and the Named Successor

The warranty period is the only part of handover that is a commercial instrument rather than a practice. It does two things: it gives the client a route to correction after acceptance, and it gives the supplier an incentive to make acceptance honest. Its failure mode is definitional - "defects" that are actually missing scope, or missing scope that is argued to be a defect - so the SOW has to define a defect as a failure to meet an acceptance criterion, and nothing else. A warranty without that definition is a second negotiation with a time limit.

The named successor is the cheapest and most effective clause in the entire engagement, and it is the one most often omitted. Naming a person creates an obligation on the client side - the person must be available, must attend the transfer, must sign - and it converts handover from a supplier deliverable into a joint one. An engagement with a named successor who has signed for the system behaves differently from month one, because the successor starts asking questions during delivery rather than at the end.

### 9.4 Why the Cut Phase Decides Whether Benefit Survives

Handover is the phase most likely to be cut, for a structural reason: it produces no new functionality, it sits at the point where the budget is most exhausted and the attention has moved to the next thing, and its benefit is entirely in the future. It is also the phase that determines whether every other phase's benefit survives.

The mechanism is simple. The benefit of an engagement is realised after the engagement ends. If the client cannot operate, change and improve the thing without the supplier, then the benefit is capped at whatever the supplier continues to be paid to sustain - and the engagement has not delivered a capability, it has created a dependency with a delivery date. That is the difference between a project that ends and a project that never quite does.

This is why the phase is worth arguing about at signature rather than at the end. Handover requirements written into the SOW at bid stage cost nothing and are almost impossible to add later, because by then the budget is committed and the supplier's remaining obligation is the scope, not the transition. The counter-intuitive consequence: the clause that protects the benefit is cheapest at the moment when the benefit is least visible.

---

## 10. What Actually Causes Failure

### 10.1 The Taxonomy

The table below is a taxonomy of symptoms, not of blames. Each row starts from something a reader would actually observe in a struggling engagement, names the structural cause underneath it, and names the guardrail that would have caught it before it became observable. The "kind" is marked in the prose after the table - some causes are the structural mismatch this guide is about, and some are orthogonal, and the distinction matters (section 2.5).

| Symptom | Structural cause | Guardrail that would have caught it |
|---|---|---|
| Discovery completes and the plan is unchanged | Price fixed before discovery, with no authority to revise the estimate | A priced discovery phase whose acceptance criterion is a revised estimate, not a report |
| Every month ends in a change negotiation | The priced scope was narrower than the knowable scope | Exclusions written at bid stage, plus a pre-agreed boundary rule for changes |
| Deliverables are accepted and the benefit does not appear | Acceptance criteria test outputs rather than the outcome the case was built on | A measured baseline in discovery, and at least one acceptance criterion tied to it |
| Two teams build the same interface twice | No architecture decision rights in the governance | A decision-rights map from discovery, and a named architecture authority in the SOW |
| The steering committee meets monthly and decides nothing | Status on the agenda, no trade, no chair with authority | A standing trade item, a named deciding chair, and a decision log the team reads |
| The handover pack is delivered and nobody can operate the system | Handover defined as documentation rather than as transfer | Run-capability evidence and a simulated incident handled by the client alone |
| The same supplier people stay for years with no transfer | A staff-augmentation form chosen to avoid the commitment conversation | An explicit end state in the SOW: what the client will be able to do without the supplier |
| Spend concentrates in the final third of the engagement | Contingency held in the price rather than released in tranches | Tranche funding with a steering decision at each release |
| Quality degrades quietly in the last two months | The supplier is absorbing a mispriced scope | Acceptance evidence per milestone, and a change-log trend read at every committee |
| An audit finds supplier access nobody was tracking | Third-party risk not represented in the SOW | An access register, a subcontractor clause, and a named owner for third-party obligations |
| The project ends and the supplier never leaves | No exit criteria and no run capability | Exit plan in the contract, transition criteria in the SOW, successor named |
| The sponsor leaves and the engagement continues on inertia | No re-confirmation gate at a change of sponsor | Sponsor change treated as a formal decision point in the governance calendar |
| The team is busy and the backlog never shrinks | Iterative cadence with no owner empowered to reorder | A named product owner with authority written into the SOW's governance section |

### 10.2 The Three Clusters

The thirteen rows cluster into three causes, and only the first is what this guide's thesis explains.

**Knowability failures** (rows 1, 2, 3, 8, 9): the arrangement assumed something was known that was not. These are the structural failures, and the guardrail in every case is a mechanism that lets the plan be a consequence of learning rather than a constraint on it.

**Authority failures** (rows 4, 5, 7, 11, 13): someone had the responsibility but not the authority, or the authority sat in a body that would not use it. These are governance failures and they are orthogonal to knowability - a perfectly coherent engagement can have a steering committee that will not decide.

**Continuity failures** (rows 6, 10, 12): the engagement ended, or the sponsor changed, or the access was never governed, and nothing was designed for the moment after. These are also orthogonal.

The point of separating them is diagnostic. A post-mortem that attributes a knowability failure to a governance failure will fix the wrong thing and repeat the engagement; a post-mortem that attributes an authority failure to the contract will re-paper a document that was never the problem. Section 15.2 records the honest limitation: this taxonomy is built from documented practice and inference, not from a dataset of engagement post-mortems, so it is a diagnostic aid rather than a measured distribution.

## 11. The Client's Side

### 11.1 The Budget Cycle

Consulting is bought inside a budget cycle, and the budget cycle has a shape that predates the engagement and outlives it. In a bank the shape is roughly annual: a planning round in which business cases are submitted, a funding decision, a release of funds against an approved case, and a year-end in which unspent funds are a problem and overspend is a bigger one.

Three consequences follow, and all three push toward the same commercial form.

First, **a business case needs a number before a solution exists**. The case is approved on a cost estimate and a benefit estimate, both produced before anyone has done discovery, because the funding decision is what would pay for discovery. The estimate is therefore a guess with a governance process around it, and the fixed price is the form that turns the guess into something a committee can approve.

Second, **the approval is for a scope, not for a question**. A case that says "we will spend a quarter finding out whether this is feasible" is much harder to approve than a case that says "we will deliver a data platform". The incentive is to describe the work as a delivery, which then attracts a delivery contract, which then requires a delivery method, which is then applied to work whose shape was never established.

Third, **the cycle creates deadline pressure that is unrelated to the work**. Funds released late in the year must be committed or lost; a project that starts in the last quarter inherits a date that was set by an accounting calendar. Discovery is the phase that absorbs that pressure, because it is the only phase that can be compressed without anyone being able to point at a missing deliverable.

### 11.2 Procurement

Procurement's job is to make purchases comparable, contestable and defensible. That job is legitimate and it has a structural consequence: **you cannot compare bids you cannot define**, so the process rewards a defined scope, and a defined scope is exactly what a fixed price requires. The two are not independent preferences; they are the same preference expressed in two vocabularies.

The mechanism is visible in the documents. A bank RFP has a commercial section that specifies the contract type and a technical section that asks for a methodology, and the two are evaluated by different people against different criteria. The commercial evaluation rewards certainty and comparability; the technical evaluation rewards a credible approach. Nothing in the process asks whether the two are consistent, so the winning bid is the one that scores well on both axes independently - which is how a fixed price and a discovery-led method end up in the same award.

Procurement rules also constrain the answer in advance. Framework agreements and standing terms often pre-approve a small set of contract types, so the form is inherited rather than chosen. Evaluation weightings that score "price certainty" explicitly make a time-and-materials bid uncompetitive regardless of whether it is the honest form. And a procurement rule that changes mid-flight - a new threshold, a new approval layer - can invalidate a form that was coherent when it was proposed, which is one of the orthogonal failure causes in section 2.5.

The anchor for all of this is documented public procurement practice rather than anything specific to consulting. The US Federal Acquisition Regulation states the principle in one sentence: contract types vary according to "the degree and timing of the responsibility assumed by the contractor for the costs of performance" ✅. That is the same claim this guide makes in section 3 - the contract type is a risk allocation - stated by a procurement rulebook rather than by a consulting text. Procurement professionals know this. What they are not usually asked to do is check the allocation against the delivery method that the technical section proposes.

### 11.3 Sponsor Incentives

A sponsor is not a neutral buyer. A sponsor is a person with a mandate, a date, and a personal position that depends on the programme being seen to progress. Those three facts explain most of the behaviour that looks irrational from the delivery side.

**A mandate** means the sponsor has already told someone the direction. A discovery that finds the direction unfeasible is not neutral information to that sponsor; it is a threat to a commitment already made. This is why sponsors so often want discovery to be fast, and why the fastest discovery is the one that confirms.

**A date** means the sponsor is managing an announcement, not a learning curve. Announcements are made at the start; the learning curve happens after. The commercial form that fits an announcement is a fixed price with a delivery date, because it is the only form that lets the sponsor say "this will be delivered by then" without qualification.

**A personal position** means the sponsor's exposure is asymmetric. Being late is visible and attributable. Having scoped the work wrongly is diffuse and hard to attribute. A rational sponsor therefore optimises for a defensible plan rather than a correct one, and a defensible plan is one that a committee approved on a number.

None of this is corruption and none of it is unique to banking. It is the ordinary incentive structure of someone who owns a delivery and is measured on it, and it explains why the buyer's approval logic - not the supplier's sales behaviour - is the main producer of mispaired engagements.

### 11.4 Why the Buyer's Approval Logic Drives the Model

| Buyer's constraint | What it forces | What it costs structurally |
|---|---|---|
| The case must be approved on a number | A price before discovery | The knowability question is answered by assumption, not by evidence |
| Bids must be comparable | A defined scope before the bid | The scope is defined by the RFP author, who is the least informed party |
| Evaluation rewards certainty | A fixed price, or a cap that behaves like one | The supplier carries risk they cannot price, and prices it defensively |
| Funds are released in tranches | A plan with milestones that a committee can read | Milestones become dates rather than events, so progress is unfalsifiable |
| The sponsor is measured on the announcement | A date before the method is chosen | The method is chosen to fit the date, not the knowability |
| Procurement rules are fixed in advance | An inherited contract type | The form and the work are never tested against each other |

Read the table from the other chair and the guide's central claim becomes less a criticism of suppliers and more a description of a system. The buyer is not choosing badly; the buyer is choosing under constraints that make the incoherent pairing the locally rational one. That is why the correction has to be offered as something the buyer wants - a way to know whether the price corresponds to the thing they asked for - rather than as a request for more time.

The practical use of this section for an architect is diagnostic. When an engagement is mispaired, the question is not "who chose badly" but "which constraint forced it". The answer usually names the artefact to change: the business case's estimate, the RFP's commercial section, the evaluation weighting, or the tranche structure. Changing the artefact is possible; changing the incentive is not.

---

## 12. The Regulated and Banking Angle

### 12.1 Why the Contract Is Longer Than the Scope

A bank's engagement contract is routinely longer than the work it describes, and the disproportion is not bureaucratic inflation. The bank is carrying obligations to a regulator that do not disappear when it delegates the work. Every one of those obligations has to be pushed down into the contract, because the contract is the only instrument through which the bank can control a third party's behaviour after signature.

That is the structural explanation: **the contract carries the risk the scope cannot**. The scope describes what will be built. The contract describes what happens if the supplier fails, if the data is mishandled, if the service stops, if the supplier is acquired, if the regulator asks for access, if the bank wants to leave. None of that is in the scope, and all of it is the bank's problem whether or not it is in the contract.

The practical consequence for an architect is that the technical design and the contractual obligations are the same subject seen from two sides. A design decision about where data is processed is a contractual question about residency and subcontracting. A design decision about a managed service is a contractual question about exit and concentration. Section 12.4 turns that into a checklist.

### 12.2 Third-Party Risk and Outsourcing

A bank's third-party risk regime typically has four stages, and each one constrains the engagement before a line of code exists: risk assessment and due diligence before onboarding, contractual requirements that must be present in the agreement, ongoing monitoring during the term, and exit planning. The material is generic documented practice in banking supervision and the specifics vary by jurisdiction; what is stable across jurisdictions is the shape.

**Due diligence** assesses the supplier as an institution, not as a delivery team: financial standing, control environment, security posture, subcontracting, jurisdiction. It happens before the engagement is signed, and it can conclude that a technically excellent supplier is not an acceptable third party, which is a decision the delivery team never sees.

**Contractual requirements** are non-negotiable clauses the bank must include: audit and access rights, data protection and confidentiality, breach notification, business continuity, subcontracting approval, termination and exit assistance. These are frequently in tension with the supplier's standard terms, and the negotiation is a commercial one that has nothing to do with the scope of work.

**Ongoing monitoring** means the engagement is subject to review after it starts: service performance, security attestations, financial health, concentration. A supplier that fails a monitoring review can be constrained mid-engagement, regardless of delivery performance.

**Outsourcing materiality** matters because a material outsourcing arrangement usually attracts a higher bar - more diligence, more contractual requirements, sometimes notification or approval before signing. Whether an arrangement is material is a determination the bank makes, not the supplier, and it can change the commercial timeline substantially. The jurisdiction-specific detail is out of scope here and is summarised in [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md); what this guide asserts is only the shape, and only as documented practice ⚠-knowledge.

### 12.3 Audit, Exit Rights, Concentration and Subcontracting

Four contractual themes recur in every bank engagement and each has an architectural counterpart.

**Audit and access.** The bank, and in many jurisdictions its regulator, must be able to inspect the supplier's controls and records. The architectural counterpart is that the supplier's environment must be inspectable: logs retained, access logged, evidence producible. A design that cannot produce evidence on request is a design that will fail an audit right the bank has already promised.

**Exit.** The contract must give the bank a way out: termination rights, transition assistance, data return and deletion, and enough documentation to move. The architectural counterpart is portability - data formats, interface specifications, and the absence of proprietary encoding in the places that matter. Exit is a design property before it is a clause.

**Concentration.** A bank that has the same supplier in too many critical places has a concentration risk that is a supervisory concern independent of any single engagement's performance. The architectural counterpart is that "use the existing platform" is not automatically the right answer, because every additional use deepens a dependency that is already being monitored.

**Subcontracting.** A supplier that subcontracts part of the work introduces a fourth party the bank did not diligence. Contracts normally require approval before subcontracting and flow-down of the bank's obligations. The architectural counterpart is that the subcontractor's environment and access are part of the design, and a design that assumes the prime contractor controls everything is a design with an unexamined dependency.

### 12.4 What This Means for the Architect

Six questions, asked at design time rather than at contract review, catch most of the regulated-engagement failures:

1. **Where does the data go, and who can see it?** Residency, access paths and subcontractor visibility are contractual obligations expressed as architecture.
2. **Can the bank leave?** If the answer requires the supplier's cooperation to be true, the exit clause needs to say so and the design needs a fallback.
3. **Can the bank prove what happened?** Audit rights are empty without retained evidence. Ask what the design produces, not what the contract promises.
4. **Is this engagement material?** If it touches an important business service, the timeline includes diligence and approval that the delivery plan probably does not.
5. **Who is the fourth party?** Any component operated by someone other than the contracting supplier is a subcontracting question, including managed cloud services.
6. **What does the SOW say about the control environment?** If the engagement will produce a system that must satisfy the bank's control framework, the acceptance criteria should reference it, or the gap becomes a post-implementation finding.

The general point is that a bank's engagement is a three-cornered arrangement - supplier, bank, regulator - and the contract is where the third corner is represented. An architect who designs only for the first two will produce something that works and cannot be signed off.

## 13. The Anti-Patterns

### 13.1 The Ten

Each of these is a recognisable move that a competent person makes for a locally good reason. That is what makes them anti-patterns rather than mistakes: they are rational responses to the pressure of the moment, and they produce the same structural failure every time.

| Anti-pattern | The failure it produces | The correction |
|---|---|---|
| **Price first, discover later** | The price encodes assumptions nobody tested; discovery becomes confirmation or renegotiation | A priced discovery phase, or a price explicitly conditional on a discovery gate |
| **The methodology in the appendix** | The method is written to satisfy the technical evaluation and never referenced again; delivery runs on the plan | Name the method in the governance section, and make the steering committee review its assumptions |
| **Discovery as a sales motion** | Unpaid discovery is scoped to win the work, so it finds what the bid needs and nothing else | Separate the discovery contract from the delivery contract, and price it |
| **The status committee** | Senior attendance, no trades, no decisions; the engagement's shape changes without anyone choosing | A standing trade item, a chair with authority, and a decision log the team reads |
| **The change log nobody reads** | Divergence is recorded and never governed; the trend appears as a total at the end | Review the register at every committee, and re-baseline when the trend is a pattern |
| **The staff-augmentation escape** | The knowability question is avoided rather than answered, and the estimate lives in the client's plan with no owner | Ask what the client will be able to do without the supplier, and write it into the SOW |
| **Handover by document drop** | The pack is complete, the capability is absent, and the benefit ends when the team does | Run-capability evidence, a simulated incident, and a named successor who signs |
| **The outcome you do not control** | Payment or credit is tied to a metric another function owns, and the engagement becomes an attribution argument | Define the outcome as something the engagement's own outputs move, and measure the baseline first |
| **The cap without a boundary rule** | At the cap, the trade is made under pressure with no pre-agreed answer | Write the boundary rule into the SOW at the same time as the cap |
| **Acceptance criteria written at acceptance** | The criteria describe what was built, so they always pass | Write them at signature, make each one binary, and name the acceptor and the evidence |

### 13.2 Why They Cluster

All ten are the same move in different costumes: **defer the knowability question to a phase that cannot answer it.** Price first defers it to delivery. The appendix methodology defers it to the plan. Unpaid discovery defers it to a phase that was never resourced to find anything. The status committee defers it to a governance body that will not decide. The document-drop handover defers it to the client, after the team has gone.

That is why they co-occur. An engagement that has one usually has three, because the same pressure - a date, a number, an approval - is being relieved in every phase by the same technique: make the commitment now, discover later, and hope the discovery is cheap. The technique works often enough to be reinforced, which is the definition of an anti-pattern.

### 13.3 The Test

One question tests an engagement for all ten: **"what happens, mechanically, on the day the discovery contradicts the price?"**

If the answer is a defined process - a re-baseline decision at the steering committee, with a named decider and a date - the engagement is coherent regardless of its form. If the answer is "we would have to renegotiate", the form and the method are in conflict. If the answer is "that won't happen", the discovery has been designed not to find anything, and the question has been answered by avoiding it.

---

## 14. The Cymbal Bank Worked Example

### 14.1 The Scenario

*This scenario is fictional and illustrative. Cymbal Bank is the only institution in it; no real engagement, supplier or individual is described.*

Cymbal Bank's payments division wants a transaction analytics platform. The stated need is unremarkable and real: the division cannot answer, quickly and consistently, questions about payment volumes, failure reasons and settlement timing, because the answers live in three operational systems that were never designed to be joined. A programme is approved in the annual planning round with a business case that estimates a cost and a benefit, and the case is approved on the strength of a target date.

Procurement runs a competitive process. Because the case was approved on a number, the RFP specifies a fixed price for a defined scope: build the platform, ingest from the three named source systems, deliver six named reports, hand over to the data engineering team. The technical section asks bidders to propose a methodology. A systems integrator wins with a proposal that scores well on both - a competitive fixed price, and a modern proposal describing two-week discovery sprints, iterative build, and continuous stakeholder validation.

Nobody in the evaluation notices that the two halves of the winning bid assume different things. The price assumes the source systems contain what the RFP says they contain. The method assumes the team will find out what they contain as it goes.

### 14.2 How the Model and the Method Disagreed

The engagement starts with a discovery that was written into the plan as two weeks of two-week sprints. It finds, within the first three weeks, three things that the price did not assume.

**The transaction status field has fourteen variants across the three systems**, and only four of them are documented. The other ten are produced by operational workarounds, two of which are load-bearing for the reconciliation the finance team runs daily. The RFP's scope said "ingest the transaction status"; it did not say which of the fourteen was authoritative, because nobody in the RFP process knew there were fourteen.

**Two of the three systems have no reliable event timestamp.** Their records carry a processing date, which is not the same as the event time, and the difference is large enough that any settlement-timing analysis built on it is wrong. Fixing that is source-system remediation, which is not in the scope, not in the price, and owned by a different division.

**One of the six named reports cannot be produced from the data that exists**, at any level of effort, without a control change in the general ledger. The report was specified by a stakeholder who assumed the data was captured; it never was.

Each of these is a discovery finding of exactly the kind the phase exists to produce. Each of them contradicts the price.

### 14.3 What It Cost

The disagreement surfaced as three distinct costs, none of them financial in this account, and all of them structural.

**Rework.** The data model was designed in week two against the documented four statuses. It was rebuilt in week nine against the fourteen. The rebuild was not a failure of engineering; it was the predictable consequence of designing before the discovery that would have informed the design. The team did the work twice because the plan required the first version before the information arrived.

**Escalation.** The supplier raised the source-system remediation as a change. The bank's delivery lead refused it, correctly, on the grounds that the scope said "as-is source data" and the RFP had said the same. The refusal was contractually sound and operationally fatal: the platform could not deliver the settlement analysis without the fix, and the fix belonged to a division that was not in the room. The dispute went to the steering committee, which had been meeting monthly on status and had never been asked to make a trade. It took three meetings to establish what the trade actually was, because the committee had no practice at making one.

**Trust.** This is the cost that outlasted the engagement. The payments division's sponsor had announced the platform at a town hall. The discovery had found that one of the six reports could not exist. By the time the finding reached the sponsor it had been through two escalation layers and had become a supplier dispute, so the sponsor heard "the vendor cannot deliver" rather than "the data was never captured". The relationship with the supplier never recovered its early goodwill, and the next engagement in that division was staffed with a different integrator for reasons that had nothing to do with capability.

The engagement completed. The platform was delivered, the settlement analysis was descoped, the source remediation was funded as a separate piece of work nine months later, and the handover was cut to two weeks because the budget was exhausted - so the data engineering team inherited a platform whose data model had been rebuilt under pressure and whose documentation had been written by people who were leaving.

### 14.4 The Guardrail and the Amended Thesis

Three guardrails would each have caught this, and the cheapest one is the second.

**A discovery gate before the price.** The RFP could have specified a fixed-price discovery of defined duration whose deliverable was a revised estimate and an evidenced scope, with the delivery price set afterwards. The bank would have paid for four weeks of work and avoided a rebuild, an escalation and a descoped report. This is the structural correction and it is politically hard, because it requires the business case to be approved without a delivery number.

**Exclusions that named the sources.** Even inside a fixed price, the SOW could have excluded source-system remediation and data-quality correction, and stated that the report specifications were valid only for data captured in the source systems as described. The exclusion would have been visible at bid stage, the sponsor would have had to fund it or accept the gap, and the argument at week nine would have happened at week zero when it was cheap. This is the cheapest correction and it costs nothing but the willingness to write it down.

**A steering committee that had made a trade before.** The escalation took three meetings because the committee had never decided anything. A committee that had traded scope against time once, early, on something small, would have had a decision mechanism when the real trade arrived.

The amended thesis, with the concession attached: **the model and the method were two expressions of the same assumption, and the assumption was wrong**. The fixed price asserted the sources were knowable; the discovery-led method asserted they were not; the engagement paid for both beliefs. That is the structural failure this guide explains, and it is not the whole story of the engagement - the sponsor's announcement, the division boundary around the source system, and the budget exhaustion at handover were all orthogonal causes, and each of them would have damaged the engagement even if the model and the method had been coherent. A coherent engagement is not a safe engagement. It is an engagement whose failures are honest ones.

## 15. The Claims Audit and What Could Not Be Verified

### 15.1 The Claims Audit

Every substantive claim in this guide appears below with its source, the date it was checked, what kind of claim it is, and its mark. "Documented practice" means a convention that is widely used and written about but not codified in a standard; "inference" means this guide's own reasoning from documented practice, which a reader should treat as an argument rather than a fact.

| Claim | Source | Date | Kind | Mark |
|---|---|---|---|---|
| ISO 20700:2017 clause 4.1.2: "The consulting process includes three typical activities: contracting, execution and closure" | ISO 20700:2017(E), as recorded in ss680_management_consultants_guide.md section 3.4 | Checked 28 Nov 2026 | Standard | ✅ |
| ISO 20700 clause 5: client and provider reach a clear agreement on scope and approach and commit to working together | ISO 20700:2017(E), same record | 28 Nov 2026 | Standard | ✅ |
| ISO 20700 clause 6: execution in accordance with the agreed approach, with collaboration "with confidence, fairness and mutual respect" | ISO 20700:2017(E), same record | 28 Nov 2026 | Standard | ✅ |
| ISO 20700 clause 7: closure requires resolution of unfinished items, contractual and legal obligations, and client acceptance - "It is critical that the MCSP gains acceptance from the client that the assignment is complete" | ISO 20700:2017(E), same record | 28 Nov 2026 | Standard | ✅ |
| ISO 20700 clause 4.1.3: the provider is responsible for its resources and work; ultimate accountability for decisions, outcomes, deliverables and stakeholder impacts lies with the client | ISO 20700:2017(E), same record | 28 Nov 2026 | Standard | ✅ |
| SS 680:2021 is a Singapore Standard, ICS 03.080.20, "Specification for management consultants", replacing TR 43:2015, whose published scope covers the consultancy process, competence, ethics and CPD | Singapore Standards eShop + NLB catalogue, as recorded in ss680_management_consultants_guide.md section 1 | 28 Nov 2026 | Standard | ✅ |
| SS 680's internal clause structure, phase names and clause numbering | None available - the standard is paywalled | 28 Nov 2026 | Not verifiable | ❌ |
| Contract type is an allocation of responsibility for performance cost | US Federal Acquisition Regulation 16.101(a)(1), acquisition.gov, FAC 2026-01 | 28 Nov 2026 | Documented practice (public procurement) | ✅ |
| There are seven canonical commercial forms for consulting and delivery work | Practitioner literature and common contract practice; no standard codifies the list | 28 Nov 2026 | Documented practice | ⚠-knowledge |
| A standard SOW carries nine clause families (scope, assumptions, exclusions, deliverables, acceptance, dependencies, milestones, change control, termination) | Common contract practice; no standard mandates a clause list | 28 Nov 2026 | Documented practice | ⚠-knowledge |
| The engagement model and the delivery method both express an assumption about knowability, so they must be chosen together | This guide's argument, built on the risk-allocation logic above | 28 Nov 2026 | Inference | ⚠ |
| A fixed price paired with a discovery-led method produces one of three endings: confirmation-only discovery, renegotiation, or silent absorption | This guide's reasoning from the allocation in section 3.1 | 28 Nov 2026 | Inference | ⚠ |
| Discovery is the phase most often compressed to protect an already-quoted price | This guide's observation from the artefacts (section 6.3); no measured base | 28 Nov 2026 | Inference | ⚠ |
| Scope creep is usually a pricing problem rather than a delivery-discipline problem | This guide's argument in section 8.4 | 28 Nov 2026 | Inference | ⚠ |
| Bank third-party risk regimes run through risk assessment, contractual requirements, ongoing monitoring and exit planning | Documented banking supervisory practice; jurisdictional detail varies | 28 Nov 2026 | Documented practice | ⚠-knowledge |
| Material outsourcing arrangements attract a higher bar, sometimes including notification or approval before signing | Documented banking supervisory practice; the specific rule is jurisdictional | 28 Nov 2026 | Documented practice | ⚠ |
| Repo counts in section 1.3 and section 1.6 (7, 11, 8, 32, 30, 44, 226, 19, 8, 98, 3, 116, 26, and 57 management guides) | Measured on this repository, `grep -rlEi --include='*.md'`, 28 Nov 2026 | 28 Nov 2026 | Measurement | ✅ |
| management_consulting_skills_guide.md section 6 spans lines 543-630 (88 lines) and measures 0 on SOW, discovery, price models and RFP/proposal | Measured on this repository, 28 Nov 2026 | 28 Nov 2026 | Measurement | ✅ |

### 15.2 What Could Not Be Verified

**1. SS 680's internal structure.** The standard is a paid publication. Its published scope is public and is quoted above; its clause numbering, phase names and requirement detail are not. Nothing in this guide depends on them, and no clause number for SS 680 appears anywhere in this file. Any secondary source that offers SS 680 clause numbers should be treated as unverified until checked against a purchased copy.

**2. The formal relationship between SS 680 and ISO 20700.** Both cover the consultancy process and both sit in ICS 03.080.20, and the overlap is real. Whether SS 680 is formally aligned to, derived from, or merely informed by ISO 20700 is not publicly documented, and [ss680_management_consultants_guide.md](ss680_management_consultants_guide.md) section 3.2 already marks it as unverified. This guide inherits that position rather than resolving it.

**3. No dataset on how often model-method mismatch causes failure.** The thesis is structural and the argument is deductive. No study was found that measures the incidence of mispaired engagements, and this guide does not estimate one. The failure taxonomy in section 10 is a diagnostic aid built from documented practice, not a distribution.

**4. The compression claim is an inference.** Section 6.3 asserts that discovery is the phase most often compressed to protect a price. The evidence offered is the artefact pattern - a document-based acceptance criterion, a duration set by the delivery plan, a delivery team doing the discovery, no estimate revision - not a measured sample. A reader who wants the claim as fact should treat it as untested.

**5. The seven-form taxonomy is a convention, not a standard.** No standard publishes a list of commercial forms. The seven used here are the ones that recur across practitioner literature and contract practice; other forms exist (gain-sharing, subscription, framework call-off, joint venture) and are either variants of the seven or outside the scope of a single engagement. The taxonomy is a thinking tool, and its exhaustiveness is unverified.

**6. Procurement behaviour is described as a mechanism, not measured.** Section 11 asserts that evaluation weightings, comparability requirements and budget cycles push toward fixed price. That is a reading of documented public procurement practice and of how approval processes work; it is not a measurement of any institution's behaviour, and no bank's procurement rules are quoted.

**7. The procurement anchor is jurisdictional.** FAR 16.101 is US federal acquisition regulation. It is cited because it states the risk-allocation principle in one clean sentence, not because it governs a Singapore bank. Other jurisdictions express the same principle differently and with different constraints, and none of them is quoted here.

**8. No rates, no benchmarks, no market data.** This is deliberate. The guide's arguments are structural and do not need them, and the price and rate figures that circulate in consulting commentary could not be verified at primary source. Their absence is a limitation of the evidence base as well as a constraint on the writing.

---

## 16. The Glossary, the Cross-References and the Closing Summary

### 16.1 The Glossary

| Term | Definition as used in this guide |
|---|---|
| **Acceptance criteria** | The test that a deliverable is complete, written before it is built, binary, with a named acceptor and named evidence |
| **Baseline** | The measured starting point of whatever the engagement intends to change; without it no outcome claim is testable |
| **Capped time and materials** | A time-and-materials arrangement with a ceiling; the client keeps both risks below the cap, and the cap creates a boundary decision |
| **Change control** | The process that turns a divergence from the SOW into a recorded decision with a named decider and a default |
| **Decision rights** | The authority to approve a change to the delivered thing; distinct from the ability to operate it |
| **Discovery** | A bounded phase whose product is knowledge that changes the plan, the estimate, or the commercial form |
| **Engagement** | The whole arrangement: commercial form, phase sequence, delivery method, and governance |
| **Engagement model** | The commercial form and its risk allocation; not a relationship style and not one firm's lifecycle |
| **Escalation** | The path a decision takes when the level that owns it cannot decide, with a time bound attached |
| **Estimation risk** | The risk that the effort or duration differs from what was assumed |
| **Exit plan** | The contractual and technical means by which the client can leave, including transition assistance, data return and portability |
| **Handover** | The transfer of knowledge, decision rights, run capability and ownership to a named successor |
| **Knowability** | How much of the requirement and the effort can be established before the work starts; the variable both axes answer |
| **Managed service** | A steady-state obligation over a term, within a service definition and catalogue |
| **Orthogonal failure** | A failure whose cause is unrelated to knowability - sponsor change, funding withdrawal, rule change, resignation |
| **Outcome-based** | A form in which payment is tied to a measured outcome, requiring a baseline and an attribution rule |
| **Scope risk** | The risk that what must be delivered differs from what was assumed |
| **SOW (statement of work)** | The instrument converting a commercial form into a bounded, testable obligation |
| **Stage gate** | A decision point at which a phase's artefact is accepted and the next phase is authorised |
| **Staff augmentation** | A form supplying capacity in which the client retains every risk and the method |
| **Steering committee** | The body that owns trades - scope, time, quality, funding release, stop - and nothing below them |
| **Workstream** | A vertical slice of delivery with its own lead, plan and dependencies, inside one engagement |

### 16.2 The Cross-References

| Guide | Section of this guide that cross-references it | What it owns |
|---|---|---|
| [management_consulting_skills_guide.md](management_consulting_skills_guide.md) | 1.4, 1.5, 4.4, 7.6 | The practitioner craft: hypothesis-driven problem solving, structured thinking, analysis, storytelling, case interviews; section 6 is client management plus a delivery skeleton (lines 543-630) |
| [consulting_advisory_experience_guide.md](consulting_advisory_experience_guide.md) | 1.5, 11.3 | The career and the profession's economics: ladder, advisory vs delivery, conversion and positioning, independent and fractional routes, the AI-era effect, Singapore |
| [ss680_management_consultants_guide.md](ss680_management_consultants_guide.md) | 1.4, 2.3, 15.1, 15.2 | The standard: SS 680 history, ISO 20700 relationship, certification ecosystem, competency, ethics, CPD - and the verified clause record this guide cites |
| [workshop_methodology_guide.md](workshop_methodology_guide.md) | 1.5, 6.1 | The workshop instrument: design, diverge/sensemake/converge libraries, facilitation craft, retrospectives |
| [business_development_experience_guide.md](business_development_experience_guide.md) | 1.5, 4.2 | Origination as a role: BD vs sales, motions and deal types, partner lifecycle, metrics and attribution, the ladder |
| [business_case_development_guide.md](business_case_development_guide.md) | 1.5, 11.1 | The business-case artefact: frameworks, structure, financials, options appraisal, risk |
| [vendor_management_guide.md](vendor_management_guide.md) | 1.5, 12.2 | The buy side: managing suppliers once engaged, performance, reviews, relationships |
| [communication_stakeholder_management_skills_guide.md](communication_stakeholder_management_skills_guide.md) | 1.5, 8.2 | Stakeholder communication: mapping, influence, difficult conversations |
| [forward_deployed_engineering_guide.md](forward_deployed_engineering_guide.md) | 1.5, 3.9 | The FDE engagement variant: embedded pods, discovery to deployment to adaptation to handover |
| [mckinsey_approach_guide.md](mckinsey_approach_guide.md) | 1.5 | One firm's model, its problem-solving approach and its economics; the nearest false overlap on the words "engagement model" |

### 16.3 The Closing Summary

The engagement is a single design with four parts - the commercial form, the phase sequence, the delivery method, and the governance - and the repository spent fifty-seven management guides and several hundred files without owning it. Seven files mention "client engagement". The two guides closest to this territory measure zero on SOW, zero on discovery, and zero on price models. The word "discovery" appears in 226 files and means the engagement phase in eight of them.

The commercial forms are seven, and each one is a different allocation of estimation risk and scope risk, with a different behaviour rewarded and a different thing the client can no longer ask for. The methods are five families, and each one is a claim about when learning happens. The lifecycle is seven phases, and each phase exists to produce an artefact that a decision can be taken on - which is why the phases are skipped in a predictable order and why the failure always surfaces two phases later in a different vocabulary.

The governance is three instruments, and scope creep is what happens when the gap between the priced scope and the knowable scope is settled through a mechanism nobody designed. Handover is six requirements, and it is the phase most likely to be cut and the one that decides whether the benefit survives the team leaving. The failures are thirteen symptoms in three clusters, and only the first cluster is the thesis - the rest are orthogonal, and a reader who uses this guide to explain a sponsor change has misused it.

**the engagement model and the delivery methodology are two expressions of the same assumption - how much is knowable before you start - and a fixed price paired with a discovery-led method buys the same risk twice.**
