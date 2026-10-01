# Marketing Technology (MarTech): The Stack as an Architecture

*The stack as an architecture rather than a shopping list — the eight categories and what each one owns, the identity spine that decides whether the stack is one system or several, consent recorded as data rather than captured as a checkbox, journeys that react to events rather than campaigns that run to a schedule, and the measurement problem stated honestly: attribution is a model, not a measurement, and it systematically flatters the channel that is easiest to track — all resting on one thesis: without identity resolution, a stack is just tools that disagree about who the customer is.*

*Jack Liu Shurui, Solution Architect*

**Last Updated:** October 2026

**Purpose.** This guide is the dedicated deep-dive for **marketing technology (MarTech)** as a discipline — the categories of system that a marketing function assembles, the architectural relationships between them, and the failure modes that are structural rather than operational. It is written for the solution architect who has been handed a vendor list and asked to make it into a platform. It deliberately does **not** survey or rank products: the MarTech vendor landscape is enormous, turns over continuously, and any ranking published here would be stale before the ink dried. What does not go stale is the *category structure* — what a customer data platform is for versus what a CRM is for, what an identity spine does, what a consent record contains, where a campaign ends and a journey begins, and why an attribution report is an argument rather than a fact. That structure is the subject. This guide also does **not** re-derive the material owned by its sibling guides; §1.4 names those by filename and states the boundary.

**Verification-markers convention.** ✅ = verified against a primary source (standards body, regulator, open-source project documentation, or the platform's own published documentation) during this pass. ⚠ = flagged or partial — practice varies, the claim rests on industry-standard practice rather than a single primary source, or the source is dated. ❌ = could not be verified at source; recorded honestly in §16. The consolidated ledger is the Claims Audit (§15), and the source-by-source register is §15.4.

**How this guide is organised.** §1 is the overview, the decoder and the boundary. §2 is the stack map — the eight categories and what each owns. §3 is the identity spine, the guide's analytical centre. §4 is consent as data. §5 is the customer journey, orchestrated. §6 is the measurement problem — the guide's most valuable honest section. §7 is the data estate underneath. §8 is the personalisation and decisioning layer, kept short because a sibling owns it. §9 is the adtech boundary. §10 is the regulated-marketing constraints. §11 is the integration and operating model. §12 is a method for assessing a stack. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified and closes. The glossary and the cross-references sit after §16.

---

## Table of Contents

1. [The Overview, the Decoder and the Boundary](#1-the-overview-the-decoder-and-the-boundary)
   - 1.1 [The Thesis](#11-the-thesis)
   - 1.2 [The Scope — Structural, Not Commercial](#12-the-scope--structural-not-commercial)
   - 1.3 [The Decoder](#13-the-decoder)
   - 1.4 [The Boundary — Declared by Name](#14-the-boundary--declared-by-name)
   - 1.5 [The `market` Filename Trap](#15-the-market-filename-trap)
2. [The Stack Map](#2-the-stack-map)
   - 2.1 [The Eight Categories](#21-the-eight-categories)
   - 2.2 [What Each Category Owns, Stores and Overlaps](#22-what-each-category-owns-stores-and-overlaps)
   - 2.3 [The Paid-Media Category as the Boundary Case](#23-the-paid-media-category-as-the-boundary-case)
3. [The Identity Spine](#3-the-identity-spine)
   - 3.1 [Deterministic and Probabilistic Matching](#31-deterministic-and-probabilistic-matching)
   - 3.2 [Individual, Household and Account — Three Different Claims](#32-individual-household-and-account--three-different-claims)
   - 3.3 [What a "Unified Profile" Actually Claims](#33-what-a-unified-profile-actually-claims)
   - 3.4 [The Disagreement Failure Mode](#34-the-disagreement-failure-mode)
   - 3.5 [The Banking Finding — the Entity Model, Not Bad Data](#35-the-banking-finding--the-entity-model-not-bad-data)
4. [Consent as Data](#4-consent-as-data)
   - 4.1 [Consent Is a Record, Not a Flag](#41-consent-is-a-record-not-a-flag)
   - 4.2 [Withdrawal as a First-Class Event](#42-withdrawal-as-a-first-class-event)
   - 4.3 [Where Consent Actually Breaks](#43-where-consent-actually-breaks)
   - 4.4 [The Instruments](#44-the-instruments)
5. [The Customer Journey, Orchestrated](#5-the-customer-journey-orchestrated)
   - 5.1 [The Event and Trigger Model](#51-the-event-and-trigger-model)
   - 5.2 [The Channel Set](#52-the-channel-set)
   - 5.3 [Suppression and Frequency Capping](#53-suppression-and-frequency-capping)
   - 5.4 [The Batch Campaign That Calls Itself a Journey](#54-the-batch-campaign-that-calls-itself-a-journey)
6. [The Measurement Problem](#6-the-measurement-problem)
   - 6.1 [The Attribution Family Are Allocation Rules](#61-the-attribution-family-are-allocation-rules)
   - 6.2 [Marketing Mix Modelling — a Different Method, a Different Assumption Set](#62-marketing-mix-modelling--a-different-method-a-different-assumption-set)
   - 6.3 [The Flattery Finding](#63-the-flattery-finding)
   - 6.4 [Incrementality Testing Through Holdouts](#64-incrementality-testing-through-holdouts)
   - 6.5 [What None of Them Settles](#65-what-none-of-them-settles)
7. [The Data Estate Underneath](#7-the-data-estate-underneath)
   - 7.1 [The Stack Is a Data Estate](#71-the-stack-is-a-data-estate)
   - 7.2 [The Profile Store and the Resolution Pipeline](#72-the-profile-store-and-the-resolution-pipeline)
   - 7.3 [Freshness and Latency](#73-freshness-and-latency)
8. [The Personalisation and Decisioning Layer](#8-the-personalisation-and-decisioning-layer)
9. [The AdTech Boundary](#9-the-adtech-boundary)
10. [The Regulated-Marketing Constraints](#10-the-regulated-marketing-constraints)
11. [The Integration and Operating Model](#11-the-integration-and-operating-model)
12. [Assessing a Stack — a Method](#12-assessing-a-stack--a-method)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Decoder and the Boundary

### 1.1 The Thesis

A marketing technology stack is not a list of tools. It is an **architecture** — a set of components that exchange data, and whose behaviour is determined almost entirely by how faithfully they agree on *who* they are exchanging data about. Buy the best-in-class tool in every category, wire them together with email address as the join key, and you have built not a stack but a federation of systems that hold divergent beliefs about the same human being. The divergence is not a bug to be fixed later. It is the default outcome of anything assembled without an identity decision made first.

Everything else in this guide follows from that: consent propagated per purpose matters only if the propagation targets the same party; a journey that fires on an event matters only if the event and the profile describe the same person; an attribution report measures only the touchpoints that were keyed to an identity, which is why it flatters the channel that is easiest to instrument. Other disciplines inside this repo have their own theses — the mainframe is un-integrated, zero trust is a policy problem. MarTech's thesis is narrower and more punishing, because the unit of correctness is not the system but the *customer*:

> **Without identity resolution, a stack is just tools that disagree about who the customer is.**

### 1.2 The Scope — Structural, Not Commercial

This guide describes **categories** and their architectural relationships. It is not a buyer's guide. Three consequences of that, stated plainly:

- **No vendor is recommended, ranked or compared.** Real platforms and product categories may be named factually, because they are the subject matter and the reader needs the vocabulary — but naming a platform is not endorsing it, and no capability is attributed to any vendor beyond what the vendor's own published documentation states.
- **No market-size, adoption-rate, ROI-multiplier or "share of wallet" figure appears.** See §15.5 for why: the genre is saturated with figures whose provenance cannot be traced, whose definitions vary between publishers, and which change on a publication cadence faster than a guide can be revised. A structural guide does not need them, and a guide that asserts them becomes wrong quietly.
- **The unit of analysis is the architecture, not the deployment.** Whether the reader is assessing an existing estate (§12), designing a new one, or being sold to, the questions that matter are the same: what owns the relationship, what is the identity key, who knows the consent state, and what is measured versus modelled.

### 1.3 The Decoder

Twelve terms recur throughout this guide and in the vendor material the reader will encounter. Defined once here, used precisely after.

- **The stack** — the assembled set of systems a marketing function operates, plus the data flows and identity keys that connect them. The stack is the *integration graph*, not the licence list.
- **The CRM** (customer relationship management system) — the system of record for the customer *relationship*: the account, the contact, the interaction history, the pipeline or the servicing record. It is a record of the institution's dealings with a party. It is not, by default, an identity resolver.
- **The customer data platform (CDP)** — a system whose purpose is to collect customer data from multiple sources, resolve it into a unified profile, and make that profile available to other systems. The distinguishing claim is *persistence and activation*: the CDP is meant to be the place the unified profile lives, not merely a pipe. Many tools self-describe as CDPs while implementing a subset of that — the decoder matters because the label is marketing, not a specification.
- **The engagement or automation platform** — the system that executes communications and campaigns: it holds audiences, templates, schedules, triggers and delivery channels. Frequently called a marketing-automation platform; the category overlaps the journey-orchestration category (§5) and the boundary between them is a product decision, not an architectural one.
- **The consent platform** — the system that captures, records and serves the data subject's choices: what purposes are permitted, on what channels, with what expiry, and how to withdraw. Consent management platforms (CMPs) as sold for the web are one implementation of this category; the enterprise consent layer is broader.
- **The decisioning engine** — the system that selects what to offer, show or say to a given party in a given context: next-best-action, recommendations, ranking. Its mechanisms belong to a sibling guide (`personalization_engines_guide.md`); §8 covers only what the stack context adds.
- **The identity spine** — the mechanism, owned by someone, that decides when two records from two systems refer to the same party, and holds that decision with its confidence and provenance. It is the most important component in the stack and the one most often left to whoever wrote the most recent integration.
- **The profile** — the assembled record of a party: attributes, events, consents, scores, segment memberships. A profile is an *assertion* that a set of records belongs to one entity (§3.3).
- **The audience** — a selection of parties defined by criteria, materialised for activation: "eligible for X", "did not open Y in 30 days". An audience is a query result; it is only as correct as the profiles it selects.
- **The journey** — a stateful, event-driven sequence of actions applied per individual, with entry criteria, branching, waiting and exit conditions. Distinguished in §5.4 from a batch campaign, which is a repeated send to a selected list.
- **The attribution model** — a rule for allocating credit for a conversion among observed touchpoints. Last-touch, first-touch, linear, position-based, time-decay, and data-driven/Markov/Shapley-style models are all members of this family. They are **allocation rules**, not measurements (§6.1).
- **The holdout** — a randomly selected control population deliberately withheld from an intervention, so that the difference in outcome between the exposed and withheld populations estimates the intervention's incremental effect (§6.4). It is the closest thing in the discipline to an actual measurement.

### 1.4 The Boundary — Declared by Name

This guide owns **MarTech as a discipline** and nothing that a sibling already owns. The following are cross-referenced by filename, not re-derived:

- **Personalisation and decisioning** — owned by [`personalization_engines_guide.md`](personalization_engines_guide.md): what a personalisation engine is, its components and reference architecture, recommendation algorithms, ranking and re-ranking, its data infrastructure, cold start, and its measurement and evaluation. §8 here covers **only** what the stack context adds — where the decision sits in the journey and what it needs from the identity spine and from consent.
- **The CRM data model** — owned by [`data/crm_data_warehouse_modelling.md`](data/crm_data_warehouse_modelling.md): Kimball dimensional modelling, Data Vault, Inmon 3NF, slowly changing dimensions, common CRM metrics, source-system mapping and ETL/ELT. §7 here cross-references it rather than repeating a single modelling technique.
- **Customer lifetime value** — owned by [`customer_lifetime_value_prediction.md`](customer_lifetime_value_prediction.md): RFM, BTYD, machine-learning approaches, churn integration and evaluation.
- **Master data and entity resolution** — there is **no dedicated MDM guide in this repository**, despite a dispatcher path that claims `technology/master_data_management_guide.md`; that file does not exist (see the report accompanying this guide). The material actually lives in [`data/data_governance_framework.md`](data/data_governance_framework.md) — §2 core components including "Reference and Master Data", §7 data-quality dimensions including master-data, golden-record and deduplication matching rules, and §12 data classification — and in [`data/handling_duplicate_keys_data_warehousing.md`](data/handling_duplicate_keys_data_warehousing.md) — §1 the duplicate-candidate-key problem, §4 matching techniques (deterministic vs fuzzy vs probabilistic), and §5 survivorship and record-consolidation rules. §3 and §7 here cross-reference **both**.
- **The regulatory side of consent** — owned by [`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md) and, for Singapore, [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md). AML/KYC obligations are owned by [`../banking/aml_certifications_exam_content_guide.md`](../banking/aml_certifications_exam_content_guide.md) and [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md). Statistical method is owned by [`advanced_analytics_solutions_guide.md`](advanced_analytics_solutions_guide.md).
- **Also available and cross-referenced where relevant** — [`data/data_vault_2_modeling.md`](data/data_vault_2_modeling.md), [`data/data_fabric_guide.md`](data/data_fabric_guide.md), [`feature_store_guide.md`](feature_store_guide.md), and [`architecture/capability_engineering_guide.md`](architecture/capability_engineering_guide.md).

### 1.5 The `market` Filename Trap

A caution for anyone — human or automated — trying to measure this repository's coverage of marketing by search. **Every guide filename containing the token `market` in this repository is about FINANCIAL markets, not marketing.** The set is:

- [`../banking/capital_markets_architecture_guide.md`](../banking/capital_markets_architecture_guide.md)
- [`../banking/market_data_integrity_guide.md`](../banking/market_data_integrity_guide.md)
- [`../banking/market_making_singapore_guide.md`](../banking/market_making_singapore_guide.md)
- [`../banking/market_data_consumption_guide.md`](../banking/market_data_consumption_guide.md)
- [`../banking/singapore_private_markets_guide.md`](../banking/singapore_private_markets_guide.md)
- [`carbon_footprint_management_market_guide.md`](carbon_footprint_management_market_guide.md)

A `grep -i market` across this repository therefore **over-reports** marketing coverage badly: it returns capital markets, market data, market making, private markets and a carbon-software market guide. None of them is MarTech. The same applies to the token `attribution`, which is dominated by the prose sense ("attributed to") and by adjacent technical senses (SHAP feature attribution, P&L attribution, margin attribution, trade P&L attribution) rather than the marketing sense — see §15.2. Measuring a discipline by keyword is a trap; measuring it by *meaning* is the only honest method.

---

## 2. The Stack Map

### 2.1 The Eight Categories

The stack resolves into eight functional categories. They are defined by **what they own**, not by what they are called on a vendor's slide, because the naming is inconsistent and the ownership is not. The first six are the core; the seventh is content; the eighth is the boundary case treated in §2.3.

1. **The system of record for the relationship** — the CRM in its widest sense.
2. **The identity and profile layer** — resolution and the unified profile.
3. **The campaign and engagement execution layer** — audiences, templates, schedules, delivery.
4. **The decisioning layer** — what to offer, show or say.
5. **The consent layer** — capture, record, serve, withdraw.
6. **The analytics and measurement layer** — reporting, attribution, experimentation.
7. **The content and asset layer** — the assets and their variants.
8. **The paid-media layer** — the boundary case (§2.3).

### 2.2 What Each Category Owns, Stores and Overlaps

**(i) The system of record for the relationship.** Owns the *institutional* facts: the account, the contract, the servicing history, the holdings or policies, the contact preferences, and the record of what the institution has done with the party. Stores the durable relationship state. Overlaps the identity layer (it holds its own identifiers), the consent layer (it often holds a preference flag that is *not* a consent record), and the analytics layer (it is a source for both).

**(ii) The identity and profile layer.** Owns the *mapping*: this record and that record are the same party; this party is an individual, a household or an account (§3.2); this profile's attributes came from these sources at these times with this confidence. Stores the resolved profile and the provenance of every merge. Overlaps everything downstream, because everything downstream reads it — which is exactly why it must be a named, owned capability rather than an emergent artefact of integration.

**(iii) The campaign and engagement execution layer.** Owns the *composition and dispatch*: which audience, which template, which channel, which time, which frequency. Stores audiences (or references to them), templates, schedules, send history and engagement events. Overlaps the decisioning layer (a campaign may embed a decision) and the consent layer (dispatch must be gated on consent, and frequently is not).

**(iv) The decisioning layer.** Owns the *selection*: given this profile and this context, which option. Stores models, features or references to a feature store, decision logs. Overlaps the identity layer (a decision is only as good as the profile) and the execution layer (the decision is consumed by an action).

**(v) The consent layer.** Owns the *permission state*: which purposes are permitted, on which channels, until when, and the history of grants and withdrawals. Stores consent records and their lifecycle. Overlaps every layer that acts on a party, because every action needs a permission basis. This is the layer whose *architecture* matters most and whose *capture* is most often mistaken for its *propagation* (§4.3).

**(vi) The analytics and measurement layer.** Owns *what is known and what is inferred*: reporting, attribution, experimentation, modelled outcomes. Stores event history, aggregates, experiment assignments and measurement outputs. Overlaps the identity layer (measurement depends on keyed touchpoints) and, through reverse-ETL, the execution layer.

**(vii) The content and asset layer.** Owns the *assets*: images, copy, offers, versions, the metadata that makes them findable and the relationships between them. Stores the asset library and its rights and expiry metadata. Overlaps execution (the asset is what gets sent) and compliance (§10 — the asset is what gets approved).

**(viii) The paid-media layer.** Owned by a different discipline and treated here only at the boundary — see §9.

### 2.3 The Paid-Media Category as the Boundary Case

The paid-media category belongs in the stack diagram because marketing buys it, but it is architecturally different from the other seven in a way that the diagram must make visible: **it is the only category where the institution does not hold the customer.** Audiences are pushed out to a platform that then acts on them; outcomes come back as aggregates or pseudonymous events rather than as records the institution owns. That asymmetry — you push identifiers out and receive statistics back — is what makes the boundary (§9) an architectural boundary and not merely an organisational one. The stack map should draw paid media at the edge, with an arrow that crosses the consent layer, because that arrow is where the hardest failure mode lives.

---

## 3. The Identity Spine

### 3.1 Deterministic and Probabilistic Matching

Identity resolution decides when two records refer to the same party. The decision is made by one of two broad techniques, and each carries a characteristic consequence.

- **Deterministic matching** resolves on an exact shared key or key combination: a customer number, an authenticated account, a verified email, a government identifier. Its consequence is **precision with brittleness**. A resolved match is essentially certain. But it defeats itself on everything unkeyed: a form filled in before login, a device that has never been authenticated, a walk-in with a name and a phone number, a joint account where the second holder has no identifier of their own. Deterministic matching resolves the part of the estate that was already keyed and leaves the rest unresolved — and because the unresolved part is where the interesting, multi-channel behaviour lives, precision is bought at the cost of coverage. The matching-technique taxonomy is owned by [`data/handling_duplicate_keys_data_warehousing.md`](data/handling_duplicate_keys_data_warehousing.md) §4 and is not re-derived here.
- **Probabilistic matching** infers sameness from statistical evidence: fuzzy agreement across name, address, phone, device and behavioural signals, with a score. Its consequence is **recall at the cost of false merges** — and the nature of the error is asymmetric. A missed merge produces two partial profiles, both true, each incomplete; the failure is visible as "the customer keeps appearing twice". A false merge produces a *single* profile that is false: one person's history, consent state and value now attached to another person. **A false merge is a data-protection incident**, not a data-quality defect, because it discloses, or causes action on, one person's information in the context of another. The guardrail is not a better algorithm; it is a merge threshold the business is willing to defend, a survivorship rule that is explicit (the rules are owned by [`data/handling_duplicate_keys_data_warehousing.md`](data/handling_duplicate_keys_data_warehousing.md) §5), and the ability to *unmerge* — a resolved merge that cannot be reversed is a liability with a permanent half-life.

Most real estates use both, in a hierarchy: deterministic where a trusted key exists, probabilistic to extend coverage, with the two clearly separated so that a deterministic fact is never overwritten by a probabilistic guess.

### 3.2 Individual, Household and Account — Three Different Claims

A stack can claim to identify three different things, and confusing them is a common source of silent error:

- **The individual** — the natural person. The party on whose behalf consent is granted, whose mailbox is contacted, whose data protection rights attach.
- **The household** — a shared entity: one address, one relationship, often one product holding (a joint mortgage, a family policy, a shared plan). The household is a legitimate marketing unit and it is *not* the individual.
- **The account** — the product or financial construct: an account number, a policy, a card. Multiple accounts belong to one individual; one account can belong to several individuals; an account can outlive the individual who opened it.

A stack that resolves to account level and calls it customer level will over-count customers and under-count relationships. A stack that resolves to household level and calls it individual level will contact the wrong person and mis-attribute consent. The decoder term "unified profile" hides which of the three it means, and the reader should always press on that question before accepting the claim.

### 3.3 What a "Unified Profile" Actually Claims

A unified profile is not a fact about the world. It is an **assertion of sameness**: the system asserts that this set of records belongs to one entity. That assertion has three properties that must be stored with it rather than discarded:

- **A confidence** — how strong the evidence for the merge is. A profile assembled from an authenticated login and a customer number is near-certain; one assembled from a shared surname and a partial postcode is a guess. The confidence must be visible to the consumer of the profile, because some decisions (send a servicing notice) and some decisions (send a promotional offer) tolerate very different error rates.
- **A provenance** — which records were merged, from which sources, when, and by which rule or score. Without provenance the merge is unauditable and un-unmergeable.
- **A method** — deterministic or probabilistic, and by which key (§3.1).

A unified profile that claims certainty it does not have is worse than no profile at all, because it launders a guess into an apparent fact that every downstream tool then trusts.

### 3.4 The Disagreement Failure Mode

This is the failure mode the whole guide exists to name. When two tools resolve the same person differently, the stack does not error out — it silently forks. The consequences are concrete and compounding:

- **The suppression list diverges.** The tool that holds the suppression list suppresses the person; the tool that holds the other identity does not, and the person is contacted anyway.
- **The consent state diverges.** One profile records the withdrawal, the other does not, and the two halves of the same estate act on contradictory permission bases.
- **The value and eligibility diverge.** The customer lifetime value attached to identity A is not the value attached to identity B; the segment membership of A is not that of B; the offer made to A is made to the wrong version of the person.
- **The measurement diverges.** A conversion keyed to A cannot be joined to a touchpoint keyed to B, so the conversion is either credited to nothing or credited to the wrong journey.

None of these presents as "an identity problem". They present as a compliance breach, an angry customer, a campaign that "underperformed", or a report that contradicts another report. The root cause is upstream and invisible, which is why the identity spine is the first thing to build and the last thing anyone budgets for.

### 3.5 The Banking Finding — the Entity Model, Not Bad Data

In a financial institution the identity problem is sharper than in a consumer brand, and it is almost universally mis-diagnosed. **A bank's customer identity is deliberately fragmented by its own legal-entity and KYC structure.** This is a statement of architectural fact, not a data-quality complaint.

The same natural person is, legitimately and by design, **several parties**: several customer numbers across business lines, several legal-entity records where the retail bank, the wealth arm and a subsidiary each hold their own relationship with their own identifier, several jurisdictional records where the person is a client in more than one jurisdiction, and several KYC files where the identity has been verified separately for separate products under separate obligations. The fragmentation is not a defect to be removed. It **is** the legal and regulatory design: separate entities, separate licences, separate obligations and separate records of their own dealings. Two of the records being "duplicates" is often the correct state of affairs.

The consequence for MarTech is the point of this section: **the martech identity problem in a bank is the entity model, not bad data.** The question is not "how do we deduplicate these records" — they are not duplicates. The question is "what is our party model, and can we map every party to the natural person it represents, for the specific purposes that require that mapping, under controlled governance?" That mapping is itself governed: it touches KYC data, cross-border transfer constraints and the legitimate-interest and purpose limitations that the compliance guides own ([`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md), [`../banking/aml_certifications_exam_content_guide.md`](../banking/aml_certifications_exam_content_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md), and for Singapore [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md)).

Nothing here is resolved by "better deduplication". It is resolved by an explicit party/entity model with a known, governed mapping down to the natural person — built deliberately, owned by a named function, and consulted before any tool is allowed to resolve identities on its own.

---

## 4. Consent as Data

### 4.1 Consent Is a Record, Not a Flag

The consent layer is a **record-keeping** system, not a checkbox. A single boolean ("opted in") is an inadequate model of consent for three reasons that are structural rather than legalistic:

- **Per purpose.** Permission to contact someone about their existing account is not permission to market a different product to them, and neither is permission to share their data with a third party. Consent attaches to a *purpose*, and the stack must be able to evaluate each action against the purpose it serves.
- **Per channel.** Permission to email is not permission to call, and neither is permission to push a notification to an app. Consent attaches to a *channel*, because the channels carry different intrusiveness and different regulatory treatment.
- **Per time.** Consent has a validity. It may be granted for a period, it may need renewal, and it may lapse. A consent record therefore carries an **expiry or renewal notion**, and a stack that never expires a grant will, eventually, act on a permission the data subject no longer intends to give.

The strongest available anchor for treating consent as structured data is **ISO/IEC TS 27560:2023, "Privacy technologies — Consent record information structure"** (Edition 1, published 2023-08-08, 52 pages, ISO/IEC JTC 1/SC 27) ✅ — an interoperable, open and extensible information structure for recording consent, explicitly intended to support the provision of a record to the data subject, the **exchange of consent information between information systems**, and the **management of the life cycle** of the recorded consent ([iso.org/standard/80392.html](https://www.iso.org/standard/80392.html)). That the standard exists, and that its stated aims are exactly the three properties above, is the architectural argument: the industry's own standard says consent must be exchangeable and lifecycle-managed, which is precisely what a single flag in a CRM cannot do.

### 4.2 Withdrawal as a First-Class Event

Withdrawal must be modelled as an **event**, not a flag-flip. The difference is not cosmetic:

- A **flag flip** changes the state in one system, at one time, and leaves every other system holding the previous state until it next syncs — if it ever does. The downstream tool keeps acting on the old state, and the person who withdrew continues to receive the thing they withdrew from.
- A **withdrawal event** is published to the estate with a timestamp and a subject identifier, and every consumer is expected to act on it. It creates an obligation with an owner, an audit trail, and a measurable propagation latency.

A withdrawal event also has a property a flag does not: it can *arrive out of order* relative to actions already in flight. If a campaign was queued at 09:00 and the withdrawal arrives at 09:05 for a send scheduled at 09:10, the correct behaviour depends on whether the estate has a suppression check at send time — which is a design decision that must be made deliberately (§4.3).

### 4.3 Where Consent Actually Breaks

The honest statement, and the most useful one in this section: **the pipeline is where consent usually breaks, not the capture.** The banner, the preference centre, the app setting — these generally capture the choice correctly and write it to the consent platform faithfully. The failures are downstream and they are architectural:

- **Propagation.** The consent platform records the withdrawal, but the CRM, the engagement platform, the analytics tool and the paid-media integration each learn of it on their own schedule — or never. Consent coverage becomes partial and nobody can state, for a given tool, whether it knows the current state (§12 gives the method for establishing exactly that).
- **The race.** The withdrawal arrives after a campaign has been composed but before it has been sent. Without a send-time check against the consent state, the withdrawal loses the race and the communication goes out. This is not a bug in consent capture; it is a missing control in the execution layer.
- **Third-party onward-sharing.** The institution shared the audience before the withdrawal. The withdrawal propagates through the institution's own systems perfectly — and stops at the boundary, because the recipient platform is outside the consent record and, in the paid-media case, outside the institution's control entirely (§9). Consent that has already been shared onward cannot be un-shared by a propagation job.

The design consequence: consent must be enforced at the **point of action**, not only recorded at the point of capture, and every boundary crossing must carry the consent basis with it.

### 4.4 The Instruments

Where consent signals travel between organisations, they travel in named industry instruments. Named factually, with their publishing bodies and dates:

- **ISO/IEC TS 27560:2023** — the consent-record information structure (§4.1) ✅.
- **IAB Tech Lab Global Privacy Protocol (GPP)**, formerly the Global Privacy Platform — a protocol designed to streamline the transmission of privacy, consent and consumer-choice signals from sites and apps to adtech providers; it supports the IAB Europe TCF, the IAB Canada TCF, the MSPA US National string and a number of US state strings; the page was last updated 12 August 2026 and notes a rename from "Platform" to "Protocol" and an August 2026 update in public comment to 11 September 2026 ([iabtechlab.com/gpp/](https://iabtechlab.com/gpp/)) ✅.
- **IAB Europe Transparency & Consent Framework (TCF)** — a voluntary cross-industry standard; v1.1 launched 25 April 2018, v2.0 on 21 August 2019, v2.1 on 19 August 2020, v2.2 on 16 May 2023, and **v2.3 in April 2025**, which made the "Disclosed Vendors" section a mandatory part of the TC string and set participants a **28 February 2026** adoption deadline ([iabeurope.eu/transparency-consent-framework/](https://iabeurope.eu/transparency-consent-framework/)) ✅.

Two points about these instruments that the architect must hold clearly: first, they govern the **adtech** supply chain and are not a substitute for the institution's own consent record; second, an instrument's existence is not a compliance claim for any jurisdiction. §10 and §16 both stress that no jurisdiction's rule is asserted here without a retrieved source; the regulatory position belongs to [`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md).

---

## 5. The Customer Journey, Orchestrated

### 5.1 The Event and Trigger Model

A journey is built from five primitives. The clarity of their definition is what separates an orchestrated journey from a scheduled send:

- **An event** — something that happened, with a subject, a type and a time: an application was abandoned, a balance crossed a threshold, a card was used abroad, a statement was viewed. Events are the raw material; they arrive from the event stream (§11).
- **A trigger** — the condition on events that admits or moves an individual in the journey: "on this event type", "when this condition holds for this party". A trigger is evaluated per individual, in near-real time if the journey is to react.
- **A condition** — a filter applied before an action: is this party eligible, is this party suppressed, is consent present for this purpose and channel, is the frequency cap reached.
- **An action** — the thing done: send an email, show an in-app message, place a decision, suppress, wait.
- **An exit criterion** — what removes the individual from the journey: the goal was achieved, the person withdrew, the journey timed out, the person entered a conflicting journey.

The presence of **state** is what makes the set a journey. The system must remember, per individual, where they are in the sequence and what has already happened to them.

### 5.2 The Channel Set

Each channel is, architecturally, a different kind of thing, and the differences determine what the stack must do to support it:

- **Email** — asynchronous, addressable, high volume, with a deliverability dimension (the sending reputation is an operational asset) and a rich engagement-event stream (open, click) — though the reliability of those events varies by client and by privacy feature.
- **SMS** — asynchronous, high intrusiveness, tightly regulated in most jurisdictions, short-form; it carries a strong consent expectation.
- **Push notification** — asynchronous, delivered to a device via an app or platform, contingent on the app being installed and the permission being granted at the operating-system level — a permission that is *separate* from the institution's marketing consent and can be revoked without the institution being told.
- **In-app** — synchronous, in-context, requires the app to be open; the immediacy is the point.
- **Web and mobile personalisation** — synchronous, rendered on the page or screen, driven by the decisioning layer; the mechanism belongs to [`personalization_engines_guide.md`](personalization_engines_guide.md).
- **Paid media** — the boundary case (§9): the institution does not own the delivery channel.
- **Outbound and servicing** — the channel that is *not* marketing: statements, notices, collections contact, relationship-manager outreach. Its second-class treatment in a marketing stack is a serious architectural omission (§10).

### 5.3 Suppression and Frequency Capping

Suppression and frequency capping are **first-class controls**, not settings on a send. Suppression answers "must this person not be contacted, by this channel, for this purpose, right now?" — covering withdrawals, complaints, bereavement and vulnerability flags, regulatory exclusions, and internal do-not-contact states. Frequency capping answers "has this person already received too much?" — across all channels, per individual, in a rolling window.

Both reduce to an identity question: **a suppression list is only as good as the identity spine underneath it.** A suppression entry keyed to identity A does not suppress the person when the campaign resolves the same person to identity B (§3.4). The most robust suppression lists are the ones keyed to the entity the institution is most confident about, with the mapping to the marketing identity held explicitly rather than assumed.

### 5.4 The Batch Campaign That Calls Itself a Journey

An architecture distinction, stated without scorn because the batch campaign is often the correct and cheaper design:

- **A batch campaign** selects an audience **once**, at a point in time, and sends on a **schedule**. Its state is the send. Its correctness depends on the audience having been computed correctly at selection time. It cannot react to what the individual does after selection, and its suppression is only as fresh as the last audience build. This is a legitimate architecture for periodic, list-based communication.
- **A journey** reacts to **events, per individual, in near-real time, with state**. Entry is dynamic rather than computed once; the individual's path depends on what they do; exit is conditional; suppression and frequency capping are evaluated at each step rather than at selection.

Many arrangements described as journeys are the former with the latter's vocabulary: a flowchart that is really a schedule, a "trigger" that is really an overnight batch job, a "real-time" message that fires on the next run. The distinction is not about sophistication — it is about **what the system can actually observe and when**. An architect should be able to point at any "journey" in the estate and answer: does this react to an event or to a schedule, does it hold per-individual state, and how fresh is the suppression it honours? If the answer is "schedule, no state, daily", it is a batch campaign, and calling it a journey creates false expectations about latency and personalisation that the business will later hold the platform to.

---

## 6. The Measurement Problem

This is the guide's most important section, because measurement is where a MarTech stack most confidently misleads the business that bought it. The material here is conceptual and does not assert any uplift figure, benchmark or attribution result; statistical method belongs to [`advanced_analytics_solutions_guide.md`](advanced_analytics_solutions_guide.md).

### 6.1 The Attribution Family Are Allocation Rules

Attribution models answer the question "which touchpoints get credit for this conversion?" They are **allocation rules** — ways of dividing a fixed quantity of credit among observed touchpoints — and they are not, and do not claim to be, measurements of effect. The family:

- **Last-touch** — all credit to the final observed touchpoint before conversion.
- **First-touch** — all credit to the first.
- **Linear, position-based (U-shaped) and time-decay multi-touch** — credit divided among observed touchpoints by position, evenly, or weighted toward recency.
- **Data-driven / Markov-chain / Shapley-value** — credit allocated by a fitted model of the observed path, typically using the same observed touchpoints.

The shared property is the one that matters: every member of this family divides credit **only among touchpoints that were observed and keyed to an identity**. A touchpoint that was not observed, or not keyed, receives no credit — not because it had no effect, but because it does not appear in the data. The repo's own coverage note on this material is thin (§15.2): the marketing-attribution sense appears on a handful of lines across the repository, and where it does appear it is described as a modelling choice — [`advanced_analytics_solutions_guide.md`](advanced_analytics_solutions_guide.md) §16.3 lists "last-click, multi-touch (linear, time-decay, position-based), algorithmic (Shapley value, Markov chains, ML-based)" and notes that it "requires cross-channel data integration, conversion tracking, test/control methodology"; [`data/crm_data_warehouse_modelling.md`](data/crm_data_warehouse_modelling.md) §7 names "first-touch, last-touch, linear, U-shaped, data-driven" as a "key challenge"; and [`personalization_engines_guide.md`](personalization_engines_guide.md) §9.2 notes that revenue-per-user "requires proper attribution logic (last-click vs. multi-touch)" ✅ (in-repo measurement).

### 6.2 Marketing Mix Modelling — a Different Method, a Different Assumption Set

Marketing mix modelling (MMM) is a **distinct method** and is frequently confused with attribution. The confusion is a category error: attribution operates at the touchpoint level and allocates; MMM operates at the aggregate level and estimates. Its assumptions are different and must be stated with it:

- **Top-down.** MMM regresses an outcome (sales, account openings, a KPI) on marketing spend and other drivers, at an aggregate level — typically time series, often by geography.
- **Aggregate, not individual.** It uses aggregated data and, by design, **does not use cookie or user-level information**. This is stated explicitly by the open-source MMM framework `google/meridian`, whose documentation describes MMM as "a statistical analysis technique that measures the impact of marketing campaigns and activities to guide budget planning decisions", notes that it "uses aggregated data to measure impact across marketing channels and account for non-marketing factors that impact sales and other key performance indicators", and states plainly that "MMM is privacy-safe and does not use any cookie or user-level information" ([github.com/google/meridian](https://github.com/google/meridian)) ✅ — cited here as an available open-source MMM framework and as a documented statement of the method's aggregate nature, **not** as a recommended product.
- **Needs historical variation.** A regression over spend and outcome can only estimate the effect of variation that existed. If a channel's spend was constant, or always increased in the same season, the model has little to work with — the assumption is not that the model is wrong but that its inputs are uninformative for that channel.
- **Assumes a functional form.** The model assumes a shape for **diminishing returns** (saturation) and for **carry-over/adstock** (a spend's effect persisting beyond its period) — typically with parameters estimated or asserted. The estimate is conditional on that shape being right.
- **Controls for baseline and seasonality.** Non-marketing drivers (baseline demand, seasonality, price, distribution, macro factors) must be controlled for, or their variation is attributed to marketing.

MMM's distinctive virtue is that it is **immune to the tracking asymmetry that biases attribution** (§6.3): because it works from aggregate spend and aggregate outcome, it does not require each touchpoint to have been observed and keyed. Its corresponding limitation is that it **cannot resolve individual-level effects** — it describes channels in aggregate and cannot say what happened to a particular person, which is exactly what a journey-orchestration platform wants to know.

### 6.3 The Flattery Finding

The finding, stated as the conclusion it is:

> **Attribution is a model, not a measurement — and it systematically flatters the channel that is easiest to track.**

The mechanism is structural, not conspiratorial. Credit can be assigned only to touchpoints that were **observed** and **keyed to an identity**. The channels that instrument best are the digital, direct, owned ones: an app push is logged and keyed by construction; an email click is logged and keyed; a website visit to a logged-in session is logged and keyed. The channels that instrument worst — or not at all — are the off-line and the untracked: the branch conversation, the hand raised to the counter, the relationship manager's call, the referral from a friend, and the earned effect of press coverage, which was never a touchpoint the institution could key. These receive **no credit**, because they produce no keyed touchpoint, not because they produced no effect.

The consequence follows directly: **attribution's output is biased toward digital and direct channels by construction, and a stack's "best channel" is frequently an artefact of its instrumentation rather than a fact about the world.** The app push wins the report because the app push is measured; the branch referral loses because it is not. An organisation that reallocates budget on that report is reallocating toward the measurable, which is a different thing from reallocating toward the effective. This is the single most consequential way a MarTech stack misleads, and it misleads without any component being wrong.

### 6.4 Incrementality Testing Through Holdouts

If the business's real question is causal — "did this spend cause behaviour that would not otherwise have happened?" — then the family of methods that answers it is **incrementality testing through holdouts**, and it is the only family here that does.

- **Randomised holdout / control-group experiments.** A population is randomly split; the test group is exposed to the intervention and the control group is deliberately withheld; the difference in outcome over the test window estimates the incremental effect. Randomisation is what licenses the causal reading: because assignment is random, the difference is not confounded by who the people are.
- **Geo experiments (matched-market / geo-split).** The unit of randomisation is a geography rather than a person, used where individual-level holdout is impractical or where the intervention (e.g. a brand campaign) is inherently geographic. Markets are matched on prior behaviour and the difference in outcome between exposed and withheld markets is the estimate.
- **Pre/post with control.** Where randomisation is impossible, a before-and-after comparison in an exposed group is read against a control group that was not exposed, to net out the common time trend.

The honest cost, stated plainly: **holdouts cost measurable revenue in the short run.** Withholding contact from a segment that would have responded removes real, near-term conversions and shows up as a worse number in the quarter in which the test runs. Running a holdout therefore **fights the quarterly incentive** — the people who own the number are asked to give up revenue to learn whether the revenue was incremental. This is why holdouts are run less than they should be, and an architect should treat the presence of *any* holdout in the estate as a signal of unusual measurement seriousness rather than as a routine control.

### 6.5 What None of Them Settles

The honest accounting of each method's limits, kept together so the reader does not over-credit any of them:

- **Attribution cannot see the untracked** (§6.3). Its allocation is over observed touchpoints only, so its output is a statement about the instrumented channel mix, not about the whole causal picture.
- **MMM cannot see the individual**, and it is **only as good as the historical variation it has** (§6.2). Where spend did not vary, or the functional form is wrong, the estimate is weak — and it returns no per-person answer at all.
- **Holdouts answer a local causal question** for the exposure and period tested, and **do not generalise for free** (§6.4). A holdout that measured a specific channel, audience and window gives a credible estimate for that constellation; extending it to a different audience, a different season or a different creative is an assumption, not an inference.

The architect's discipline follows: **separate what is modelled from what is measured** in every report, and label the two differently, because a modelled number presented as a measured one is the failure mode this whole section describes.

---

## 7. The Data Estate Underneath

### 7.1 The Stack Is a Data Estate

The profiles, events and identities that the stack operates on are data, and the machinery that produces them is a data estate. The finding is short: **a MarTech stack is a data estate and fails like one** — the same failure modes (stale data, orphaned records, undocumented feeds, unclear ownership), the same ownership gaps (nobody owns the join key), and the same "who owns the pipeline" problem (the marketing function owns the outcome, the data function owns the pipes, and the pipeline is everybody's problem and nobody's job).

The modelling techniques for the warehouse that holds this estate are **not re-derived here**: dimensional modelling, Data Vault, Inmon 3NF and slowly changing dimensions are owned by [`data/crm_data_warehouse_modelling.md`](data/crm_data_warehouse_modelling.md); hash keys and hub/link/satellite design by [`data/data_vault_2_modeling.md`](data/data_vault_2_modeling.md); the fabric pattern by [`data/data_fabric_guide.md`](data/data_fabric_guide.md). For a banking reader, CRM-specific modelling is here in the repository and should be read before designing the profile store (see the in-repo coverage scan in §15.2, which records that facility as genuinely present).

### 7.2 The Profile Store and the Resolution Pipeline

The pipeline that produces a profile has a shape worth naming, because each stage is a place the estate fails:

1. **Ingestion** — events and attributes arrive from source systems, channels and partners. Failure here: the source declares a schema and silently changes it, or arrives with a key the pipeline does not expect.
2. **Keying** — each record is annotated with whatever identifiers it carries. Failure here: the record carries a different identifier set from its neighbours, so it cannot be joined.
3. **Matching** — candidate records are compared and grouped by identity (§3.1). Failure here: the thresholds are wrong, or the matching is done differently in two pipelines so the two produce different customers (§3.4).
4. **Survivorship** — where matched records conflict on an attribute, a rule decides which value wins. Failure here: the rule is implicit, so the profile silently takes the most recent, or the first-loaded, contrary to what the business believes. The rules belong to [`data/handling_duplicate_keys_data_warehousing.md`](data/handling_duplicate_keys_data_warehousing.md) §5, and the golden-record and deduplication responsibilities to [`data/data_governance_framework.md`](data/data_governance_framework.md) §7.
5. **Storage and serving** — the resolved profile is stored and made available to consumers, with the confidence and provenance of §3.3 preserved. Failure here: the store flattens the merge to a single "golden" record and discards the evidence, so the profile cannot be audited or unmerged.

### 7.3 Freshness and Latency

The question that connects the estate to the journey is: **how fresh must a profile be for a decision to be right?** The answer is decision-specific, and treating it as uniform is a common design error. A servicing notice may tolerate a profile that is a day old; a fraud-adjacent decision, or a suppression check at the moment of send, may not. A batch campaign can tolerate an audience built last night; an event-driven journey cannot tolerate a profile that has not yet learned that the event happened. The estate must therefore publish, per profile attribute and per consumer, a **freshness expectation** — and the journey engine must know which of its decisions depend on which attribute's staleness. A stack that cannot state the freshness of the profile it is acting on cannot state whether its decisions are correct.

---

## 8. The Personalisation and Decisioning Layer

This section is deliberately short: the **mechanisms** of personalisation belong to [`personalization_engines_guide.md`](personalization_engines_guide.md), which owns the engine's definition, its components and reference architecture, recommendation algorithms, ranking and re-ranking, its data infrastructure, cold start, and its measurement and evaluation. What this guide adds is only the **stack context** — three questions that a solution architect must answer about a decisioning engine that the engine's own documentation does not answer:

**1. Where does the decision sit in the journey?** A decision is either *inline* (evaluated at the moment of the action, needing low latency and a fresh profile) or *precomputed* (materialised ahead of time, e.g. a nightly next-best-action list, trading freshness for throughput and cost). The architectural consequences differ: an inline decision needs the identity spine and the profile store to answer within the action's latency budget; a precomputed decision needs the materialisation job to run, and it will act on a profile that is by definition as old as the last run. A stack that mixes the two without labelling which is which will produce decisions whose freshness is invisible to the business.

**2. What does it need from the identity spine?** A decision applies to *someone* — and the decision's correctness is bounded by the identity resolution's correctness (§3). If the engine resolves the party differently from the channel that will execute the decision, the decision is made about one version of the person and delivered to another (§3.4). The engine must consume the resolved identity, not re-derive its own.

**3. What does it need from consent?** A decision may be permissible to *make* and impermissible to *act on*. Consent gates the action, not the computation — but the stack must therefore be able to evaluate consent between the decision and the action, which means the decision layer and the execution layer must share the consent layer's answer rather than each holding their own copy (§4.3). A decisioning engine also inherits the purpose limitation on the data it uses: a profile assembled for one purpose may not be a lawful input for a decision serving another, which is a governance question owned by [`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md).

That is the whole of this guide's contribution to decisioning. Everything else — algorithms, ranking, cold start, evaluation — is the sibling's.

---

## 9. The AdTech Boundary

### 9.1 The Architectural Distinction

The cleanest statement of where MarTech ends and adtech begins:

> **MarTech addresses the customer the institution already knows; adtech buys the audience it does not.**

MarTech operates on a known party — a customer, a contact, a prospect who has identified themselves — and acts on that party through channels the institution owns or controls. Adtech operates on an audience the institution does not know individually: it buys the opportunity to show a message to people matching a description, through a platform that holds neither the institution's relationship nor its consent record. The institution supplies the description and, increasingly, the identifier set; the platform supplies the reach.

### 9.2 What Crosses the Boundary

Traffic crosses the boundary in both directions, and each direction carries its own obligation:

- **Outward: audience and identifier data.** The institution pushes an audience — or the identifiers that describe it — to a paid platform: hashed email or phone, device identifiers, a segment definition, a conversion signal. What leaves is, in effect, a set of customer references handed to a system outside the institution.
- **Inward: conversion and outcome data.** The platform returns measurement: impressions, clicks, conversions attributed by the platform's own model, and increasingly aggregate reports designed to preserve privacy. What comes back is a statistic about the audience the institution sent, not a record of individual customers.

### 9.3 The Consent Dependency and the Instruments

The consent signal that governs adtech is **a different instrument from the institution's own consent record**, and conflating them is the boundary's characteristic error. The institution's consent record (§4) says what *it* may do with *its* customer's data. The adtech consent string says what a chain of third parties may do in a browser or app context. They are different objects with different issuers, and the presence of one is not evidence of the other.

The industry instruments, named factually with their publishers and dates:

- **IAB Tech Lab Global Privacy Protocol (GPP)** — carries privacy, consent and consumer-choice signals through the digital supply chain, including the IAB Europe TCF string among others (§4.4) ✅.
- **IAB Europe Transparency & Consent Framework (TCF)** — the consent standard for the European digital-advertising supply chain, at **v2.3 as of April 2025**, with a **28 February 2026** adoption deadline (§4.4) ✅.
- **WICG Attribution Reporting API** — a **Web Platform Incubator Community Group** specification, published as a Draft Community Group Report (this version dated 11 May 2026), explicitly **"not a W3C Standard nor is it on the W3C Standards Track"**; it provides browser-side support for measuring and attributing conversions without cross-site identifiers, and supports event-level and aggregate reports ([wicg.github.io/attribution-reporting-api](https://wicg.github.io/attribution-reporting-api/), [github.com/WICG/attribution-reporting-api](https://github.com/WICG/attribution-reporting-api)) ✅. The status disclaimer matters: it is an incubating specification, not a settled standard, and its design intent is to make cross-site identifiers unnecessary rather than to replace the institution's own measurement.
- **IAB Tech Lab OpenRTB (Real-Time Bidding)** — the open API specification for the automated trading of digital media; the RTB Project (formerly the OpenRTB Consortium) assembled in **November 2010**; version line hosted at [github.com/InteractiveAdvertisingBureau/openrtb2.x](https://github.com/InteractiveAdvertisingBureau/openrtb2.x); the IAB Tech Lab page was last updated 23 January 2024 ([iabtechlab.com/standards/openrtb/](https://iabtechlab.com/standards/openrtb/)) ✅. (The page also carries a 2013 market-share statistic that is an artefact of its age; **no such figure is reproduced here** — see §15.5.)

### 9.4 The Failure Mode When the Boundary Is Crossed Without Consent

The failure mode, stated precisely: **an audience sent to a paid platform carries the institution's customer identifiers into a system that is outside its consent record and outside its control, and the withdrawal cannot be propagated.** Once the identifiers have crossed, a later withdrawal event (§4.2) propagates cleanly through every system the institution owns and **stops at the boundary**. The institution cannot un-send an audience, cannot recall a hashed identifier from a platform's stores, and cannot ask the platform to forget the association reliably or on demand. The consent dependency at the boundary is therefore not a formality to be engineered around — it is the last point at which the institution still has control, and a design that treats the outward push as a routine ETL job rather than a consent-gated act has already conceded the failure mode.

---

## 10. The Regulated-Marketing Constraints

A financial institution's marketing passes through a gate that a consumer brand's does not. This section states the *shape* of that gate — the categories of constraint — and cites named sources factually where they were retrieved, while asserting no jurisdiction's rule without one. Where practice varies between jurisdictions and between regulated activities, this guide says that it varies.

### 10.1 Financial-Promotion Rules

The first class of constraint is the **financial promotion**: a communication that invites or induces a person to engage in investment or regulated activity. Its regulatory shape, in the jurisdictions that impose it, is that the promotion must be **fair, clear and not misleading**, and that the *approval* of a promotion is a **control rather than a courtesy** — a named, qualified person signs off before the communication is used, and the sign-off is a record.

A retrieved primary source for one jurisdiction's approach is the **SEC's "Investment Adviser Marketing" final rule** — Release No. IA-5653, File No. S7-21-19, adopted 22 December 2020, effective **4 May 2021** ✅ — which merged the advertising and cash-solicitation rules into a single amended rule under the Advisers Act (17 CFR 275.206(4)-1) and amended the books-and-records rule (17 CFR 275.204-2), while rescinding the former solicitation rule (17 CFR 275.206(4)-3) ([sec.gov/rules/final/2020/ia-5653.pdf](https://www.sec.gov/rules/final/2020/ia-5653.pdf)). The rule's own structure — a definition of "advertisement", general prohibitions on untrue or unsubstantiated statements and on misleading implications, conditions attaching to testimonials and endorsements, requirements on performance advertising, and an explicit review-and-approval section — is itself an architectural specification of what a compliant marketing function must be able to *do with its content*, per asset, at the point of use.

The **FCA's** financial-promotions material could **not be retrieved this pass** — both `fca.org.uk/firms/financial-promotions` and `handbook.fca.org.uk/handbook/COBS/4/` returned nothing usable to the scraper. That is recorded as a **tool limitation** in §16, not as evidence of absence, and no FCA rule is asserted here without a retrieved source.

### 10.2 Record-Keeping — a Marketing Communication Is a Record

The second class is **record-keeping**: the institution may have to produce, years later, a record of a marketing communication. The shape of that obligation, where it applies, is that the record must let the institution answer: *what* was communicated, *to whom*, *when*, and *who approved it*. A primary source for one jurisdiction's approach is **FINRA Rule 2210, "Communications with the Public"** ✅, which defines "communications" as consisting of **correspondence, retail communications and institutional communications**; defines **"correspondence"** as written (including electronic) communication distributed or made available to **25 or fewer retail investors within any 30 calendar-day period**; defines **"institutional communication"** as written communication distributed or made available **only to institutional investors**; and defines **"retail communication"** as written communication distributed to **more than 25 retail investors within any 30-calendar-day period** ([finra.org/rules-guidance/rulebooks/finra-rules/2210](https://www.finra.org/rules-guidance/rulebooks/finra-rules/2210)) ✅. The rule imposes **principal approval** before first use for retail communications and a **record-keeping** obligation (referring to the retention period required by SEA Rule 17a-4(b)) whose required records include the communication itself and its dates of first and last use, the **name of the approving principal and the date of approval**, and the source of any statistical table, chart, graph or illustration used.

The architectural consequence is direct and easy to miss: an institution in scope of obligations of this shape must be able to reconstruct, per communication, the **audience, the approval and the date** — which means the marketing stack must retain those as first-class, queryable facts, not merely as a rendered artefact and a log line. A platform that can send a personalised message to a computed audience but cannot answer "who received this exact version, on what date, approved by whom" is not a stack the compliance function can rely on, whatever its send throughput.

### 10.3 Suitability and Fairness

The third class is **suitability and fairness**: a promotion that reaches an unsuitable audience can be a conduct problem even if every word of it is accurate. A promotion for a product that is inappropriate for the recipient, or that is distributed to a group the institution would not knowingly sell to, is a defect of *targeting* rather than of *copy*. This is why the identity spine and the audience definition are compliance-relevant architecture and not merely marketing convenience: if the stack cannot reliably exclude ineligible parties from an audience, it cannot reliably satisfy a suitability obligation, and the failure is invisible in the creative review that the institution does perform.

### 10.4 The Separation Between Marketing and Servicing Communication

The fourth class, and the one with the clearest architectural requirement, is the **separation between MARKETING consent and SERVICING communication**. A customer who withdraws marketing consent must still receive the communications that are not marketing: servicing notices, contractual communications, fee and rate-change notifications, and regulatory disclosures. Withdrawing from marketing is not a request to be cut off from one's own account.

The architectural requirement that follows, and the reason it belongs in this guide rather than in a legal appendix:

> **The stack must be able to tell which of its communications is which — per message.** It must know, for every communication it sends, whether that communication is marketing (and therefore gated on marketing consent) or servicing (and therefore owed regardless of marketing consent).

The failure is symmetric and both directions are serious. If the stack cannot make that distinction, it will either **keep marketing** to someone who withdrew (because it treats everything as consented, or because the withdrawal did not propagate — §4.3), or it will **suppress servicing** (because it applies the marketing consent state to everything, and a withdrawn customer stops receiving their rate-change notice). The first is a consent breach; the second is a servicing and possibly contractual failure. Neither is a wording problem, and neither is fixed by a better preference centre — both are fixed only by a **purpose taxonomy the stack enforces at send time**, with each message classified against it.

### 10.5 Where Practice Varies

Practice varies, and the guide says so plainly: the set of obligations that attach to a given communication depends on the jurisdiction, the regulated activity, the product, the channel and the audience. The three sources retrieved this pass (SEC IA-5653 ✅, FINRA Rule 2210 ✅, and the adtech instruments of §9.3 ✅) are named with their dates and are cited as **the shape of the constraint in the jurisdictions they govern**, not as a global rule. For the Singapore position, the reader is directed to [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md) and to MAS primary material at the time of reading, rather than to an assertion made here.

---

## 11. The Integration and Operating Model

### 11.1 The Mechanisms

The pieces of a stack connect by a small number of mechanisms, and naming them precisely is what allows an architect to reason about the identity and consent obligations each one carries:

- **APIs (REST/GraphQL over HTTPS) and webhooks.** Synchronous request/response for reads and writes; a webhook is the inverse — the system pushes an event to a URL when something happens. Webhooks are how "near-real-time" is usually implemented, and they carry an identity the receiver must be able to resolve.
- **Event streams (the event bus as the connective tissue).** A publish/subscribe channel where events are produced once and consumed by many. The event bus is the natural home of the withdrawal event (§4.2) and the journey trigger (§5.1); it is also where identity problems become *plural*, because every consumer resolves the event's subject independently unless the spine is shared.
- **Reverse-ETL (warehouse → operational tool).** The pattern of computing a result in the warehouse (a segment, a score, a suppression list) and *writing it back* into the operational tool that acts on it. Reverse-ETL is how a data team's model reaches the campaign platform; it is also a frequent source of identity divergence, because what is written back carries whatever identity the warehouse uses, which may not be the identity the tool uses.
- **File drops and batch feeds.** Still everywhere in banking. A nightly extract, a scheduled secure transfer, an SFTP directory of CSVs. These carry no schema negotiation, no delivery confirmation by default, and no identity semantics unless the file says so — which makes them the least observable and most fragile integration in most estates.

### 11.2 The Identity Key Each Integration Carries

The single most diagnostic question about any integration in a MarTech estate is: **what identity key does it carry?**

- An integration that ships the **party ID** (a resolved, institution-owned identifier) is a different integration from one that ships an **email address** as the join key.
  - The party-ID integration is stable under a change of email address, unambiguous across legal entities if the party model is right, and — crucially — reconcilable to the identity spine by construction.
  - The email-address integration breaks the moment the customer changes their email; it silently merges two people if an address is recycled; it cannot represent a party with no email; and it forces every downstream tool to re-do identity resolution from an unstructured key, each with its own rule, which is exactly how a stack ends up with divergent customers (§3.4).
- The same question applies to consent: an integration that ships the **consent basis** alongside the identifier is a different integration from one that ships data only. In an estate where most integrations ship data-only, consent coverage is partial by construction (§4.3), and no amount of capture-side improvement changes that.

### 11.3 The Integration Burden

The burden is multiplicative, and this is the arithmetic an architect should present when someone proposes adding a tool:

- **N tools × M integrations.** Each new tool does not add one integration; in a fully-connected stack it adds up to N. The cost of a tool is dominated by the number of connections it needs, not by its licence.
- **A per-integration consent obligation.** Every connection that moves personal data is a place consent must be evaluated, and a place a withdrawal must propagate to. Adding a tool adds a consent-propagation target.
- **A per-integration identity obligation.** Every connection is a place the identity key must be carried, resolved and kept consistent. Adding a tool adds an identity-divergence risk until it is reconciled to the spine.
- **Versioning, error handling and reconciliation.** Each integration has a contract that drifts, a failure path that must be handled (what happens when the webhook does not arrive, when the file is missing, when the API returns partial data), and a reconciliation job that proves the two sides agree. The reconciliation job is the integration's only evidence of correctness, and it is the first thing left unbuilt.
- **Undocumented feeds.** The integrations no one admits to are the ones that break silently: a spreadsheet upload by an analyst, a hand-run script, a partner's file that someone set up years ago.

### 11.4 Who Owns the Stack

Ownership in a MarTech estate is split in a way that guarantees the join keys are nobody's job:

- **Marketing owns the outcome** — the campaign, the response, the attributed revenue — and therefore has an incentive to add tools that improve the number.
- **Data or IT owns the pipes** — the warehouse, the pipelines, the integrations — and therefore carries the integration burden without owning the requirement.
- **Risk and compliance own the constraints** — the approval, the record-keeping, the consent obligations — and therefore hold the veto without owning the design.

Nobody owns the **join keys**. The identity mapping between the three parties is the artefact that determines whether all three functions are working with the same customer, and it is precisely the artefact that falls into the gap between their charters. This is the organisational form of §3.4, and it is the reason the identity spine must be established as a *named, funded capability* rather than assumed.

### 11.5 The Honest Finding

> **The integration work, not the licences, is where the cost and the fragility live.**

A stack's licences are a visible, budgeted line item. Its integrations are an accumulating, largely unbudgeted liability: N×M connections, each with a consent and identity obligation, each a potential silent failure, none of them owned end to end. This is why a stack assembled from best-in-class tools can cost more, over its life, than a smaller stack of adequate tools — and why the assessment method in §12 devotes a whole input to integration debt. The vendor sells a licence; the institution buys an integration graph, and the integration graph is the thing that has to be operated.

---

## 12. Assessing a Stack — a Method

This is the guide's practical contribution: a **method** for assessing an existing MarTech estate, presented as a sequence of inputs with a decision output. It deliberately produces **no score out of ten** — a score implies a benchmark, and there is no benchmark here that would be honest. What the method produces instead is a statement of *what must be built before anything else is bought or connected*. It has an order; the order matters, because each step's output is the next step's input.

### 12.1 Input (i) — The Inventory

*What it asks:* every tool in the estate, what it **stores**, what it is the **system of record** for, and **who owns it**.

*Why it comes first:* you cannot assess identity or consent coverage across tools you have not enumerated, and the enumerations routinely reveal tools the marketing function does not know the data team runs, and vice versa.

*The output:* a table of tool → data class stored → system-of-record claim → owning function. The **system-of-record column is the diagnostic one**: where two tools both claim to be the system of record for the same data class, you have found either a redundancy (§12.6) or a disagreement about who the customer is (§3.4), and you have found it before a single integration.

### 12.2 Input (ii) — The Identity Score

*What it asks:* **how many distinct customer identities exist across the tools**, whether they can be reconciled, and **by what key**.

*How to take it:* for each tool, record the identifier it keys its customers on. Then ask, pairwise, whether a join between tool A and tool B is possible without a mapping table — and if it needs a mapping table, who maintains it.

*The output:* a count of distinct identity spaces and a statement of the keys that separate them. A stack with one identity space keyed on a resolved party ID scores well; a stack with five identity spaces keyed variously on email, phone, account number and an internal surrogate ID has five customers for every one, and the finding is the *number and the keys*, not a quality judgement.

### 12.3 Input (iii) — Consent Coverage, per Purpose and per Channel

*What it asks:* for each tool in the inventory, **does it know the consent state, at what granularity, and how fresh?**

*How to take it:* the question is asked per purpose and per channel, because consent is dimensioned by both (§4.1). For each (tool, purpose, channel) cell: does the tool hold the state, or does it rely on the caller to have checked? When was its copy last synchronised? Does it learn of a withdrawal, and by what mechanism (§4.3)?

*The output:* a coverage map showing exactly which tools can act correctly on their own and which are trusting an upstream check that may not have happened. The uncovered cells are the estate's consent risk, and they are usually a large fraction of the map.

### 12.4 Input (iv) — The Attribution-Honesty Check

*What it asks:* **separate what is MODELLED from what is MEASURED**, and ask whether there is **any holdout anywhere in the estate**.

*How to take it:* read every recurring marketing report and mark each number as measured (from a controlled experiment) or modelled (from attribution or MMM). Then search the estate for a randomised holdout — a control population deliberately withheld from an intervention (§6.4).

*The output:* two lists. The modelled list is almost always much longer than the measured list, and the estate's *held-out* population count is frequently zero. A zero is not a failure to be hidden; it is the finding, and it reframes every number in the modelled list as an allocation rather than an effect (§6.3). If the estate has at least one holdout, this step establishes what it measured and for how long — and therefore how far its result can honestly be extended (§6.5).

### 12.5 Input (v) — Integration-Debt Assessment

*What it asks:* the **join keys**, the **failure paths**, the **reconciliation jobs**, and the **undocumented feeds** (§11).

*How to take it:* for each connection in the graph, record: which identity key it carries; what happens when it fails (and how the failure is detected); whether a reconciliation job exists and when it last ran clean; and whether the connection is documented anywhere. Then add the connections that appear in no diagram — the analyst's upload, the partner's file, the script in someone's home directory.

*The output:* a debt register. Its size is the estate's fragility, and it is typically invisible in the licensing budget while dominating the operating cost (§11.5).

### 12.6 Input (vi) — The Duplicate-Capability Question

*What it asks:* **which capability is bought twice, and why?**

*How to take it:* for each capability in the inventory (send email, hold a profile, resolve identity, capture consent, run an experiment), count how many tools provide it. Where the count exceeds one, find out how the second instance arose.

*The output:* the reason, which is almost always the same — **the first instance was not integrated**, so a second buyer solved the problem locally rather than connect to a system they could not reach. The duplicate is a symptom, and the fix is the integration, not the decommissioning. (§13.5 shows this question in use.)

### 12.7 The Order and the Decision Output

The method has an order because the inputs compound:

1. **Inventory** (§12.1) — you cannot measure what you have not enumerated.
2. **Identity score** (§12.2) — identity is the precondition for consent, measurement and integration being *meaningful*; consent coverage across divergent identities is not coverage.
3. **Consent coverage** (§12.3) — taken per identity space, because a consent state attached to an identity that the acting tool cannot resolve is not coverage.
4. **Attribution honesty** (§12.4) — taken per tool and per report, once you know which identities and consents the numbers rest on.
5. **Integration debt** (§12.5) — assessed last of the quantitative steps, because the debt is where the previous four findings live.
6. **Duplicate capability** (§12.6) — the summary question, asked last, whose answer is usually explained by the debt.

**The decision output is not a score.** It is a statement of the form: *the estate currently has N identity spaces; consent coverage is complete in X of Y tool-purpose-channel cells; measurement is modelled in M reports and measured in none; integration debt is D connections of which U are undocumented; and the duplicate capabilities are Q, all caused by un-integration.* From that statement the decision follows mechanically — **the identity spine and the party-to-person mapping are built and owned first, and no new tool is bought until the estate can state, per tool, which customer it thinks it is acting on.** That is the method's whole purpose: to convert an open-ended question ("is our stack good?") into a bounded one ("what must be true before we buy another tool?").

---

## 13. The Cymbal Bank Worked Example

*Cymbal Bank is a fictional institution used throughout this repository as the reference persona. It is explicitly illustrative: the scenario below is constructed to exercise the architecture, not to describe any real institution. No real bank is asserted to run, choose or fail at any capability described.*

### 13.1 The Scenario

Cymbal Bank has a scattered marketing estate assembled over several years, one tool at a time:

- a **CRM**, holding the customer record, the account relationships and the servicing history — the system of record for the relationship;
- an **engagement/automation tool**, holding audiences, templates and send history, with its own view of who the customer is;
- an **analytics tool**, holding the event stream and producing the recurring marketing reports.

Each holds **its own version of the customer**, and the three versions were never reconciled. There is no artefact in the estate that states whether Cymbal's engagement tool's "customer 4471" and Cymbal's CRM's "customer 4471" are the same person, or which of them the analytics tool keyed the campaign events to.

### 13.2 Working the Method — the Inventory and the Identity Reconciliation

Applying §12.1, Cymbal enumerates the three tools and the connections between them: CRM → analytics (nightly file), engagement → analytics (event API), CRM → engagement (a nightly audience sync keyed on **email address**). That last key is the first finding: the audience sync is a **different integration** from a party-ID sync (§11.2), and it means Cymbal's engagement tool re-resolves identity from an unstructured key on every run, with its own rule, while the CRM resolves it from the customer number.

Applying §12.2, Cymbal finds **three identity spaces**: the CRM's customer number, the engagement tool's email-keyed contact, and the analytics tool's device-and-session-keyed visitor. Pairwise, CRM↔engagement is joinable *only* through the email address (which breaks on change and on reuse); CRM↔analytics is joinable only for the fraction of events that carried a logged-in identifier, which Cymbal discovers is a minority.

The reconciliation then runs into the finding of §3.5, and this is the heart of the example. Cymbal's **retail bank**, its **wealth arm** and its **Singapore subsidiary** each legally hold a **different party for the same natural person**: three customer numbers, three legal entities, three KYC files, three sets of obligations. When the identity reconciliation proposes merging them, **the "duplicate" records are not duplicates.** They are the correct state of affairs — Cymbal's own entity structure, where the fragmentation is the legal and regulatory design. The obstacle to a unified profile is therefore **the entity model, not the data quality**, and no deduplication tool will resolve it, because there is nothing to deduplicate: there is a mapping to define, govern and maintain between the several legitimate parties and the one natural person.

Cymbal's conclusion at this step: the identity spine it needs is not a matching engine alone; it is a **party model** — an explicit statement of what a party is in Cymbal, plus a governed mapping from each party down to the natural person for the specific purposes that require it, with the cross-entity and cross-jurisdiction constraints the compliance function owns ([`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md), [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md)).

### 13.3 The Withdrawal That Reached Two of Three Systems

Cymbal next applies §12.3 and finds its consent coverage map has a hole. A retail customer withdraws marketing consent. The withdrawal reaches the **consent platform** (captured correctly) and the **CRM** (propagated by the nightly sync). It **never reaches the engagement tool**, because the audience sync pushes audiences *out* to the engagement tool and nothing pushes the consent state back (§11.1 — the reverse flow was never built).

The consequence follows in the same week: a promotional campaign was already composed and queued in the engagement tool, its audience built from the pre-withdrawal state, with **no send-time consent check**. The campaign goes out. The customer receives the marketing they withdrew from — not because the choice was missed at capture, but because **the pipeline is where consent broke** (§4.3), and because the race between the withdrawal and the queued campaign had no control to resolve it (§4.2).

Cymbal's finding: this is not a consent-capture defect to report as a training issue. It is a missing control — the send-time check — and a missing propagation path. Both are architectural, and both are fixed in the execution and consent layers rather than in the preference centre.

### 13.4 The Attribution Report That Flattered the App Push

Cymbal's analytics tool produces a recurring channel-performance report using last-touch attribution. It shows **app push** as the best-performing channel by a wide margin, and it shows the **branch referral** and **press coverage** as marginal or absent.

Applying §12.4 — the attribution-honesty check — resolves the contradiction. **The app push wins because it is keyed and logged**: every push is delivered through Cymbal's own app, attributed to a known contact, and its resulting in-app conversion carries the same identifier, so the touchpoint is observed and keyed by construction. **The branch referral is under-credited because it was never a keyed touchpoint**: the conversation happened, the customer opened the account, but no system recorded that the branch caused it, so it scores nothing. **The press coverage receives no credit at all**, because earned media was never a touchpoint Cymbal could observe, let alone key.

The report is therefore not wrong in its arithmetic; it is **biased by its instrumentation** (§6.3). Cymbal's "best channel" is an artefact of what Cymbal happened to be able to measure, and reallocating budget on that report would move spend toward the measurable rather than toward the effective. The modelled number and the world are different things.

So Cymbal runs the thing that answers the causal question instead: a **holdout** (§6.4). It randomly holds back a segment of the eligible population from an intervention — **no contact at all** — and measures the difference in account openings over the test window between the contacted population and the withheld population, **reporting the confidence interval** alongside the point estimate. The holdout costs Cymbal measurable openings in the quarter it runs, which is exactly the honest cost of the method, and Cymbal accepts it because it is the only result in the estate that answers "would this have happened anyway?" It also records what the holdout measured — one channel, one audience, one window — so that no one extends it beyond its scope (§6.5).

### 13.5 The Duplicate Capability, and the Decision

Applying §12.6, Cymbal asks which capability is bought twice. The answer: **audience selection** exists in both the CRM and the engagement tool, and **profile storage** exists in all three. The reason is the one the method predicts — the first instance was not integrated, so the second was bought to solve the problem locally (§12.6).

The decision Cymbal reaches is therefore not a purchase. Cymbal decides to **build the identity spine and the party-to-person mapping as an owned capability before buying another tool.** Concretely: to define the party model, to make the resolved party ID the key that every integration carries, to build the consent propagation path with a send-time check, to make the suppression list keyed to the entity Cymbal is most confident about, and to reconcile the three identity spaces to the spine rather than letting each tool resolve identity on its own. Only then does the estate have a defensible answer to the question every other capability depends on.

### 13.6 The Thesis, Reached

Cymbal's scattered estate was not failing because its tools were poor. It was failing because three tools held three versions of the customer, keyed three different ways, with three different beliefs about consent and one shared belief — that the other two were right. Every failure in the example traces to that: the withdrawal that did not propagate, the campaign that went out anyway, the channel that looked best because only it could be seen. The example ends where the guide's argument begins:

> **Without identity resolution, a stack is just tools that disagree about who the customer is.**

---

## 14. The Anti-Patterns

Each anti-pattern is given as **symptom → cause → guardrail**, because the symptom is what the organisation sees and the cause is what the architect has to fix.

### 14.1 The Stack Built Tool-First with No Identity Spine

- **Symptom.** Each tool's report counts a different number of customers; a segment built in one tool does not match the same-named segment in another; nobody can say which version of the customer is the authoritative one.
- **Cause.** The tools were bought and integrated before any decision was made about what a party is and what key identifies it. Identity was left to whatever the most recent integration happened to carry (§11.2).
- **Guardrail.** Establish the identity spine and the party model as a **named, owned capability** before adding tools; make the resolved party ID the key every integration carries; reject any integration proposal that cannot state which identity it ships (§12.2, §13.5).

### 14.2 Consent Captured but Not Propagated

- **Symptom.** A customer who withdrew still receives marketing; the consent platform shows the withdrawal, the tool that sent the message does not; the audit trail of *what the acting tool knew* is absent.
- **Cause.** Consent was treated as a capture problem. The propagation path, the withdrawal event and the send-time check were never built; tools hold their own copies and synchronise on their own schedules, or never (§4.3).
- **Guardrail.** Model withdrawal as a first-class event, build propagation as an owned path into every acting tool, and enforce consent **at the point of action** with a send-time check — not only at the point of capture (§4.2, §12.3).

### 14.3 Attribution Reported as Measurement

- **Symptom.** A "best channel" ranking drives budget decisions; the untracked channels (branch, referral, earned media) appear marginal; the report is presented without distinguishing modelled from measured numbers.
- **Cause.** An allocation rule (§6.1) was labelled a measurement, hiding the fact that it can only credit observed and keyed touchpoints, and therefore flatters the easiest-to-track channel by construction (§6.3).
- **Guardrail.** Label modelled and measured numbers differently in every report; run incrementality tests through holdouts where the causal question is the real one; and state the limits of each method alongside its result (§6.4, §6.5).

### 14.4 The Journey That Is a Batch Campaign

- **Symptom.** The "journey" fires on the next overnight run rather than on the event; suppression is only as fresh as last night's audience build; the business is surprised that the message arrived a day after the behaviour.
- **Cause.** A schedule with journey vocabulary (§5.4). The system selects an audience once and sends on a timetable; it holds no per-individual state and cannot react.
- **Guardrail.** Answer the three questions explicitly for every arrangement called a journey — does it react to an **event** or a **schedule**, does it hold **per-individual state**, and how **fresh** is the suppression it honours — and either build the reactive design or describe it accurately as a batch campaign.

### 14.5 A Duplicate Capability Bought Twice

- **Symptom.** Two tools can send the same message; two tools hold the same profile; the same experiment runs in two places; costs rise while the capability does not.
- **Cause.** The first instance was not integrated, so a second buyer solved the problem locally rather than connect to a system they could not reach (§12.6).
- **Guardrail.** Ask the duplicate-capability question on every renewal and every purchase; where a duplicate exists, fix the **integration** (the cause) rather than simply decommissioning the second tool (the symptom).

### 14.6 Marketing Consent Treated as Indistinguishable from Servicing Communication

- **Symptom.** Either a customer who withdrew marketing consent keeps receiving promotions, or a customer who withdrew marketing consent stops receiving their fee and rate-change notices. Both versions of the symptom appear in estates where the distinction is not modelled.
- **Cause.** The stack classifies communications by channel and asset rather than by **purpose**, so it cannot tell marketing from servicing per message, and it applies one consent state to everything (§10.4).
- **Guardrail.** Build a purpose taxonomy the stack enforces at send time, with every message classified against it, so that marketing is gated on marketing consent while servicing, contractual and regulatory notices are owed regardless of it (§10.4).

### 14.7 A Paid-Media Integration That Bypasses the Consent Layer

- **Symptom.** An audience is pushed to a paid platform by a pipeline that runs like any other ETL job; the consent state is not evaluated at the boundary; a later withdrawal cannot be propagated into the platform's stores.
- **Cause.** The outward push was treated as routine data movement rather than a consent-gated act, crossing the boundary at a point where the institution loses control (§9.2, §9.4).
- **Guardrail.** Gate every boundary crossing on the consent layer; treat the outward push as an act of the consent layer rather than a data pipeline; and recognise that after the crossing, the withdrawal cannot be recalled — so the decision must be made before it, not after (§9.4).

---

## 15. The Claims Audit

The audit is in five parts: the repository coverage scan this guide ran itself (§15.1); the false-friend investigations, recorded as **the dispatcher's measurement, corrected by this guide's own scan** (§15.2); the source ledger, with date and quality (§15.3); what was verified, flagged and rejected (§15.4); and the standing statement on the figures this guide refuses to state (§15.5).

### 15.1 The Repository Coverage Scan — This Guide's Own Run

Run over all **640** `.md` files in `/home/ubuntu/research` (✅, this pass). Discipline vocabulary, by number of files matching:

| Term | Files | Reading |
|---|---|---|
| `martech` | **0** | the discipline is absent by name |
| `marketing technology` | **0** | absent by name |
| `lifecycle marketing` | **0** | absent |
| `\bDMP\b` (data management platform) | **0** | absent |
| `marketing automation` | 2 | peripheral |
| `customer data platform` | 2 | peripheral, passing mentions only |
| `\bMMM\b` | 1 | peripheral |
| `email marketing` | 1 | peripheral |
| `adtech` | 1 | peripheral |
| `campaign management` | 3 | peripheral |
| `\bCRM\b` | 100 | dense — **but a different sense**: the CRM as a data-modelling and systems subject, not as MarTech |
| `customer journey` | 31 | dense — but mostly the journey as a metaphor or a servicing narrative, not as an orchestrated activation |
| `personalisation` | 17 | dense — owned by `personalization_engines_guide.md` |
| `consent management` | 15 | dense — mostly the regulatory/consent-platform sense |
| `identity resolution` | 9 | dense — but the entity-resolution sense in data engineering, not the marketing identity spine |
| `\bCDP\b` | 23 | **false-friend dominated — see §15.2** |
| `attribution` | 204 | **false-friend dominated — see §15.2** |
| `\bsegment` | 274 files / 1,504 lines | three senses, none of them the marketing-stack sense — see §15.2 |

**The corpus is dense but adjacent.** This repository has a great deal to say about CRM data modelling, personalisation engines, consent regulation, entity resolution and attribution-as-a-statistical-term. It has **no guide that owns MarTech as a discipline** — no guide that treats the stack as an architecture with an identity spine, consent as data, journey orchestration and a measurement problem. That is the gap this guide fills.

### 15.2 The False Friends — the Dispatcher's Measurement, Corrected by This Guide's Own Scan

Three tokens appear to signal marketing coverage and do not. In each case the scan below is this guide's own re-run; the dispatcher's counts are noted where they differ.

**`\bCDP\b` — 23 files, at least SEVEN distinct meanings; it is NOT marketing coverage.**

| Meaning | Where (this guide's scan) | Weight |
|---|---|---|
| **1. Central Depository (Pte) Limited** — SGX's central securities depository (Singapore) | `../banking/financial_infrastructure_guide.md`, `../banking/market_making_singapore_guide.md`, `../banking/online_investment_trading_platforms_guide.md`, `../banking/jump_trading_guide.md`, `../banking/hudson_river_trading_guide.md`, `../banking/singapore_trust_companies_guide.md`, `../banking/dbs_software_systems_guide.md`, `../banking/tower_research_capital_guide.md`, `../singapore/singapore-government-securities-guide.md`, `../singapore/singapore_government_tech_stack_guide.md` | **DOMINANT** |
| **2. Customer Data Platform** (the genuine marketing sense) | `../banking/universal_banking_model_guide.md` (lines 587, 589, 598, 631, 710 — incl. the definitional line "CDP (Customer Data Platform) — the system of record for client behavioral data that powers next-best-action cross-sell"), `../banking/customer_behaviour_modeling_guide.md`:194 ("the customer-data platform (CDP) world"), `personalization_engines_guide.md`:164 (Segment/RudderStack described as CDP-class tooling), `storylane_tech_stack_analysis.md`:64 & 166, `data/handling_duplicate_keys_data_warehousing.md`:428 ("Used by Salesforce CDP, ActionIQ"), `../singapore/starhub_software_systems_guide.md`:466 & 672 | **~6 files, and mostly one-line name-drops** |
| **3. Collateralized Debt Position** | `defi_guide.md`:659 (MakerDAO Vault) | 1 file |
| **4. Cloudera CDP (Cloudera Data Platform)** | `kafka_alternatives_guide.md`:207 | 1 file |
| **5. Carbon Disclosure Project** | `carbon_footprint_management_market_guide.md`:116 (SBTi partnership) | 1 file |
| **6. IBM Z Common Data Provider (CDP)** | `zero_trust_mainframe_guide.md` (several) | 1 file |
| **7. Chrome DevTools Protocol (CDP)** | `ai_llm/agent_harness_engineering_guide.md`:502; `ai_llm/agent_runtime_cache_design_guide.md`:268 (⚠ — acronym used without expansion) | 2 files |
| (coincidence) `CDP` as a **GL account code** | `../banking/oracle_flexcube_data_model_guide.md`:803, 805 | not a meaning at all |

**Reading:** the marketing-CDP thought appears in roughly **six files as a name-drop** and in **no guide as a discipline**. The dominant sense of the token in this repository is SGX's central securities depository.

**`attribution` — 204 files, false-friend dominated.**

The prose sense ("attributed to", "can be attributed") and adjacent technical senses (SHAP/feature attribution, P&L attribution, margin attribution, trade P&L attribution) dominate by a wide margin. The **marketing-attribution** sense appears on approximately **six lines across four files** (this guide's scan; the dispatcher estimated ~5 lines across seven files — the difference is a matter of which adjacent phrases are counted). The genuine occurrences are:

- `advanced_analytics_solutions_guide.md`:758 (heading "§16.3 Marketing Attribution") and :760 — "Models: last-click, multi-touch (linear, time-decay, position-based), algorithmic (Shapley value, Markov chains, ML-based). Requires cross-channel data integration, conversion tracking, test/control methodology."
- `data/crm_data_warehouse_modelling.md`:96 — "Key challenge: Attribution model (first-touch, last-touch, linear, U-shaped, data-driven)."
- `personalization_engines_guide.md`:235 — revenue-per-user "requires proper attribution logic (last-click vs. multi-touch)."
- `data/gaming_dw_bet_recommendation.md`:43 ("marketing attribution") and :511 ("Last-touch … bet-time channel").

Four other files matched the broad pattern but on **other senses** and are recorded as false positives for the marketing-attribution scan: `../banking/financial_fraud_detection_at_scale_guide.md`:443 ("multi-touch gestures"), `contech_construction_technology_guide.md`:561 ("multi-touchpoint process"), `a16z_big_ideas_2026_guide.md`:976 (a rejected-statistics audit row that mentions an "attribution model" in passing), `ai_llm/ai_verify_toolkit_guide.md`:597 (unrelated). **Reading:** `attribution` at 204 files is a severe over-report of marketing-measurement coverage.

**`segment` — 274 files / 1,504 lines, three senses, none of them the marketing-stack sense.**

- **Audience/behavioural segment** sense — the closest to marketing — appears on a minority of lines.
- **Market-structure/cohort** sense ("retail segment", "SME segment", "customer segment") accounts for a large share.
- **Technical** sense (network segment, data segment, TCP/storage/market-data segment, MPLS, segment file) accounts for a large share again.

The dispatcher's estimate was 276 files / 1,558 lines with the three senses "roughly balanced"; this guide's scan found **274 files / 1,504 lines** — a difference consistent with grep-engine and pattern-boundary variance, not a disagreement about the reading. **Reading:** `segment` is not marketing-stack coverage.

### 15.3 The Source Ledger

| # | Source | Date / version | Quality |
|---|---|---|---|
| 1 | ISO/IEC TS 27560:2023 "Privacy technologies — Consent record information structure" | Edition 1, published **2023-08-08**, 52 pages, ISO/IEC JTC 1/SC 27 | ✅ retrieved ([iso.org/standard/80392.html](https://www.iso.org/standard/80392.html)) |
| 2 | IAB Tech Lab **Global Privacy Protocol (GPP)** | page last updated **12 August 2026**; August 2026 update in public comment to 11 September 2026 | ✅ retrieved ([iabtechlab.com/gpp/](https://iabtechlab.com/gpp/)) |
| 3 | IAB Europe **Transparency & Consent Framework (TCF)** | v1.1 **25 Apr 2018**; v2.0 **21 Aug 2019**; v2.1 **19 Aug 2020**; v2.2 **16 May 2023**; v2.3 **April 2025**; adoption deadline **28 Feb 2026** | ✅ retrieved ([iabeurope.eu/transparency-consent-framework/](https://iabeurope.eu/transparency-consent-framework/)) |
| 4 | WICG **Attribution Reporting API** | Draft Community Group Report, this version **11 May 2026**; explicitly not a W3C Standard and not on the Standards Track | ✅ retrieved ([wicg.github.io/attribution-reporting-api](https://wicg.github.io/attribution-reporting-api/), [github.com/WICG/attribution-reporting-api](https://github.com/WICG/attribution-reporting-api)) |
| 5 | SEC **"Investment Adviser Marketing"** final rule | Release No. **IA-5653**, File No. **S7-21-19**, adopted **22 Dec 2020**, effective **4 May 2021** | ✅ retrieved ([sec.gov/rules/final/2020/ia-5653.pdf](https://www.sec.gov/rules/final/2020/ia-5653.pdf)) |
| 6 | **FINRA Rule 2210** "Communications with the Public" | current rulebook text; defines correspondence (≤25 retail investors / 30 calendar days), retail and institutional communication; principal approval and record-keeping | ✅ retrieved ([finra.org/rules-guidance/rulebooks/finra-rules/2210](https://www.finra.org/rules-guidance/rulebooks/finra-rules/2210)) |
| 7 | IAB Tech Lab **OpenRTB (Real-Time Bidding)** | RTB Project assembled **November 2010**; page last updated **23 January 2024**; version line at github.com/InteractiveAdvertisingBureau/openrtb2.x | ✅ retrieved ([iabtechlab.com/standards/openrtb/](https://iabtechlab.com/standards/openrtb/)) |
| 8 | **google/meridian** (open-source MMM framework) | README, **v2.1.0 / 2026**; states MMM "uses aggregated data… does not use any cookie or user-level information" | ✅ retrieved ([github.com/google/meridian](https://github.com/google/meridian)) — cited as an available framework and a documented statement of method, **not** a recommended product |
| 9 | FCA financial-promotions material | — | ❌ **not retrievable this pass** — `fca.org.uk/firms/financial-promotions` and `handbook.fca.org.uk/handbook/COBS/4/` both blocked the scraper (**tool limitation**, not absence of material) |
| 10 | `web_search` backend | — | ⚠ **intermittent** — returned results for some queries and empty sets for others this pass; recorded as a tool limitation, and compensated by direct `web_extract` on primary URLs |

### 15.4 Verified, Flagged, Rejected

- **Verified ✅ (this pass, at source).** The existence and dates of the eight sources in §15.3; the content of the FINRA Rule 2210 definitions (correspondence / retail / institutional communication, principal approval, the required records) as quoted; the SEC rule's adoption and effective dates, its merger of the advertising and solicitation rules, and its amendment of rule 206(4)-1 and rule 204-2; the ISO/IEC TS 27560 aims (record provision, exchange between systems, life-cycle management); the TCF version history and the v2.3 "Disclosed Vendors" change and deadline; the GPP's carriage of the TCF and other privacy strings; the WICG specification's Community-Group status and non-Standard disclaimer; OpenRTB's 2010 origin and current version line; Meridian's documented aggregate-data nature.
- **Flagged ⚠.** The broad statement that consent "usually breaks in the pipeline rather than at capture" is an argument from architecture and observed practice, not a measured claim, and no figure is attached to it; the claim that holdouts "cost measurable revenue in the short run" is a structural consequence of withholding contact, not a quantified one; the three-sense decomposition of the token `segment` is a qualitative reading of matched lines, not a per-line classification; `ai_llm/agent_runtime_cache_design_guide.md` uses the acronym `CDP` without expansion.
- **Rejected ❌.** The eMarketer 2013 market-share statistic carried by the OpenRTB page is **not reproduced** — it is a decade-old figure on a page whose own last-updated date is 2024, and it is exactly the kind of untraceable number §15.5 excludes. The dispatcher's file count for `segment` (276 files) is **corrected to 274** by this guide's scan, and its line count (1,558) to **1,504**. The dispatcher's path `technology/master_data_management_guide.md` is **wrong — no such file exists**, and the master-data/entity-resolution material is actually owned by `data/data_governance_framework.md` and `data/handling_duplicate_keys_data_warehousing.md`.

### 15.5 The Standing Statement on Figures This Guide Refuses to State

**No market-size figure, no adoption rate, no ROI multiplier, no vendor ranking and no product comparison appears anywhere in this guide.** No vendor is attributed a capability beyond what its own published documentation states, and no real bank is asserted to run, choose or fail at any capability. This is a deliberate editorial position, not an omission, and the reasons are three:

1. **Provenance.** The MarTech genre is saturated with figures whose primary source cannot be traced, whose definitions differ between the publishers who quote them, and which are frequently recycled past their vintage (the 2013 statistic on the OpenRTB page, rejected in §15.4, is the illustration).
2. **Volatility.** The vendor landscape changes continuously — products are renamed, merged, repositioned and discontinued, and category labels (including "CDP") are used loosely by vendors and analysts alike (as §15.2 demonstrates the token is used loosely even inside this repository).
3. **Irrelevance to the argument.** Every finding in this guide is structural. The identity spine matters whether the stack has three tools or thirty; attribution flatters the trackable channel whether the market is large or small; consent that does not propagate breaks regardless of the adoption rate. A figure would add the appearance of precision without changing a single conclusion — and would become wrong quietly, which is the worst property a claim in a reference guide can have.

Where a reader needs a quantitative figure for a business case, they should retrieve it from a primary source at the time of writing, and record its provenance and date in their own audit.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

Recorded honestly, as the marker convention requires:

- **The FCA's financial-promotions material.** `fca.org.uk/firms/financial-promotions` and `handbook.fca.org.uk/handbook/COBS/4/` could **not** be retrieved this pass — both blocked the scraper. This is a **tool limitation, not evidence of absence**. No FCA rule is asserted anywhere in this guide, and the reader should retrieve the FCA's own material before relying on any statement about UK financial-promotion rules (❌).
- **Web-search reliability.** The `web_search` backend returned empty result sets for a number of queries this pass while succeeding on others. Every source in §15.3 was therefore retrieved by **direct extraction from the publisher's own URL**, not via search (⚠ tool limitation).
- **No quantified claims about consent-propagation failure rates, holdout revenue cost, or attribution bias magnitude.** These are stated as architectural consequences, not measured effects, and no figure is attached. If a figure is wanted, it must be measured in the reader's own estate and sourced there (§15.5).
- **The Singapore regulatory position.** Not asserted here. The reader is directed to [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md) and to MAS primary material at the time of reading, not to any rule stated in this guide.
- **The MarTech vendor landscape, market size and adoption.** Not verified and deliberately not stated — see §15.5. Any figure encountered in vendor material is the vendor's claim and should be treated as such.

### 16.2 The Glossary

| Term | Definition |
|---|---|
| **Adtech** | The system of buying audiences through platforms the institution does not own; architecturally the boundary case of the stack (§9) |
| **Attribution model** | An **allocation rule** dividing credit for a conversion among *observed, keyed* touchpoints — last-touch, first-touch, linear, position-based, time-decay, data-driven/Markov/Shapley. A model, not a measurement (§6.1) |
| **Audience** | A selection of parties defined by criteria and materialised for activation; a query result, correct only to the extent the profiles it selects are correct (§1.3) |
| **Batch campaign** | A send to an audience selected **once**, dispatched on a **schedule**, holding no per-individual state. Distinguished from a journey (§5.4) |
| **CDP (customer data platform)** | A system whose purpose is to collect customer data, resolve it into a unified profile, and make it available for activation. The label is used loosely; the token `CDP` in this repository has at least seven meanings (§15.2) |
| **Consent record** | A structured record of permission dimensioned **per purpose, per channel and per time**, with a lifecycle — exchangeable between systems and managed over time, per ISO/IEC TS 27560:2023 (§4.1) |
| **Deterministic matching** | Identity resolution on an exact shared key — precise, but defeats itself on everything unkeyed (§3.1) |
| **GPP (Global Privacy Protocol)** | IAB Tech Lab's protocol for transmitting privacy, consent and consumer-choice signals through the digital supply chain; formerly the Global Privacy Platform (§4.4, §9.3) |
| **Holdout** | A randomly selected control population deliberately withheld from an intervention, so the measured difference estimates the incremental effect. The only family here that answers the causal question (§6.4) |
| **Identity spine** | The mechanism, owned by someone, that decides when two records refer to the same party and holds that decision with its confidence, provenance and method (§3) |
| **Incrementality** | The effect of an intervention that would not have occurred without it; measured by holdout/geo experiment, not by attribution (§6.4) |
| **Journey** | A stateful, event-driven sequence of actions applied per individual, with entry, branching, waiting and exit criteria (§5.1, §5.4) |
| **MMM (marketing mix modelling)** | A distinct top-down, aggregate method regressing an outcome on spend, assuming a functional form for diminishing returns and carry-over, controlling for baseline and seasonality; cannot resolve individual effects (§6.2) |
| **Party / natural person** | The legal/financial construct the institution holds (one party per entity/KYC relationship) versus the human being it represents; in a bank the mapping between them is governed, and the fragmentation is by design (§3.5) |
| **Probabilistic matching** | Identity resolution inferred from statistical evidence with a score — recall at the cost of false merges, and a false merge is a data-protection incident (§3.1) |
| **Profile (unified)** | An **assertion of sameness** with a confidence, a provenance and a method — not a fact (§3.3) |
| **Purpose limitation** | The principle that data collected for one purpose is not automatically lawful input for another; consent and data use both attach to purposes, not to persons in the abstract (§4.1, §8) |
| **Reverse-ETL** | Writing a warehouse-computed result (segment, score, suppression list) back into the operational tool that acts on it (§11.1) |
| **Servicing communication** | A non-marketing communication owed to the customer regardless of marketing consent — statements, fee and rate-change notices, contractual and regulatory disclosures (§10.4) |
| **Suppression** | The control that answers "must this person not be contacted, by this channel, for this purpose, now?" — only as good as the identity spine underneath it (§5.3) |
| **TCF** | IAB Europe's Transparency & Consent Framework, at v2.3 since April 2025, with a 28 February 2026 adoption deadline (§4.4, §9.3) |
| **WICG Attribution Reporting API** | A Web Platform Incubator Community Group specification for browser-side conversion measurement without cross-site identifiers; explicitly **not** a W3C Standard (§9.3) |

### 16.3 The Cross-References

From this guide's location (`technology/`), siblings are plain filenames, subdirectories are prefixed, and banking guides are prefixed `../banking/`.

- **Personalisation and decisioning — the mechanisms** — [`personalization_engines_guide.md`](personalization_engines_guide.md) (§8 here covers only the stack context).
- **The CRM data model** — [`data/crm_data_warehouse_modelling.md`](data/crm_data_warehouse_modelling.md) (§7 here cross-references rather than re-derives).
- **Customer lifetime value prediction** — [`customer_lifetime_value_prediction.md`](customer_lifetime_value_prediction.md).
- **Master data, entity resolution and data quality** — [`data/data_governance_framework.md`](data/data_governance_framework.md) (§2, §7, §12) and [`data/handling_duplicate_keys_data_warehousing.md`](data/handling_duplicate_keys_data_warehousing.md) (§1, §4, §5). **Note:** the dispatcher's path `technology/master_data_management_guide.md` does not exist.
- **Data Vault and the fabric pattern** — [`data/data_vault_2_modeling.md`](data/data_vault_2_modeling.md), [`data/data_fabric_guide.md`](data/data_fabric_guide.md).
- **Feature store** — [`feature_store_guide.md`](feature_store_guide.md); **capability ownership** — [`architecture/capability_engineering_guide.md`](architecture/capability_engineering_guide.md) (consent management and identity resolution as capabilities).
- **Analytics and statistical method** — [`advanced_analytics_solutions_guide.md`](advanced_analytics_solutions_guide.md) (§16.3 owns the marketing-attribution model list).
- **Consent's regulatory side** — [`data/data_compliance_frameworks.md`](data/data_compliance_frameworks.md); **Singapore** — [`../banking/mas_regulations_guidelines_guide.md`](../banking/mas_regulations_guidelines_guide.md).
- **AML/KYC and the entity constraint on identity** — [`../banking/aml_certifications_exam_content_guide.md`](../banking/aml_certifications_exam_content_guide.md), [`../banking/financial_risk_compliance_systems_guide.md`](../banking/financial_risk_compliance_systems_guide.md).
- **The `market` filename trap** — the banking guides named in §1.5 are all financial-markets guides and are **not** marketing coverage.

### 16.4 The Closing Summary

This guide set out to treat the marketing technology stack as an architecture rather than a shopping list, and it arrived at a single organising idea with a set of consequences.

The stack resolves into eight categories — the system of record for the relationship, the identity and profile layer, the execution layer, the decisioning layer, the consent layer, the analytics layer, the content layer, and paid media at the boundary (§2) — and the category structure, not the product list, is what lasts. Above all of them sits the **identity spine** (§3): the mechanism that decides when two records are the same party, holds that decision with its confidence and provenance, and keeps the estate from forking. In a bank, that problem is the **entity model, not bad data** — the same natural person is legitimately several parties because the fragmentation *is* the legal and regulatory design, and nothing is fixed by better deduplication (§3.5).

Consent is **data** — recorded per purpose, per channel and per time, with withdrawal as a first-class event, and enforced at the point of action rather than merely captured at the point of choice (§4). The journey is an **event-driven, stateful** arrangement, distinguished honestly from the batch campaign that wears its vocabulary (§5). The measurement problem is stated as it is: the attribution family are **allocation rules** over observed, keyed touchpoints; MMM is a distinct aggregate method with its own assumptions and no individual resolution; and **attribution flatters the channel that is easiest to track** by construction, because the untracked touchpoint — the branch referral, the hand raised to the counter, the press coverage — receives no credit for the effect it may have had (§6). Incrementality testing through **holdouts** is the only method here that answers the causal question, and it costs short-run revenue, which is why it is run less than it should be (§6.4).

Underneath it all is a **data estate** that fails like one, with the same ownership gaps and the same "who owns the pipeline" problem (§7); a decisioning layer whose inputs the spine and the consent record must supply (§8); an **adtech boundary** where the institution buys the audience it does not know and loses the ability to propagate a withdrawal the moment an identifier crosses (§9); a gate of **regulated-marketing constraints** — financial-promotion approval, record-keeping, suitability, and the separation of marketing from servicing communication that the stack must enforce per message (§10); an **integration graph** where the cost and the fragility actually live and where nobody owns the join keys (§11); and a method for assessing it all that ends not in a score but in a decision — build and own the identity spine and the party-to-person mapping before buying another tool (§12, §13).

Every one of those findings reduces to the same dependency. The consent that must propagate is consent about *a party*; the journey that must react is a journey for *an individual*; the report that flatters is a report about *keyed people*; the suppression that must hold is a suppression of *the right person*; and the integration that must be governed is an integration of *identities*. Get the identity wrong and none of the rest can be right, because the stack will be acting on customers who do not exist and missing the ones who do.

The guide therefore closes on the sentence it opened with, because it is the whole argument:

without identity resolution, a stack is just tools that disagree about who the customer is.
