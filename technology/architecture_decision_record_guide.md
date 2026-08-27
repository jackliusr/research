# Architecture Decision Records: The Decision Trail — A Comprehensive Guide

*The dedicated deep-dive on Architecture Decision Records (ADRs) — the short, self-contained documents that capture one significant architecture decision each, in three parts: Context, Decision, Consequences. From Michael Nygard's 2011 article that popularized the practice, through the template family (Nygard, MADR, Y-statement), the statuses and the lifecycle, the numbered ADR log and its file-naming conventions, the practice of when to write one and how ADRs sit on the documentation spectrum (ADR vs RFC vs design doc vs wiki), the supersession chains, the tooling (adr-tools, Log4brains, MADR), the organizational angle (governance, guardrails, decision review gates, fitness functions, Team Topologies), the regulated-banking angle (the decision trail as audit evidence under BCBS 239 and MAS expectations), and a complete worked example — a Cymbal Bank ADR mini-series for the event-backbone decision.*

> **Author:** Jack Liu Shurui, Solution Architect — Cymbal Bank, Singapore
> **Context:** the decision-recording discipline guide of the technology series — the practice that binds the repository's architecture guides together (every sibling guide is, in effect, a long-form companion to the ADRs a working architect would write). It sits alongside [AI Platform Engineering](ai_platform_engineering_guide.md) (the platform-team/ownership angle, cross-ref'd in §9), [Kafka](../banking/kafka_guide.md) (the event-backbone platform behind the worked example, cross-ref'd in §11), and the banking/ risk cluster ([Enterprise Risk Management](../banking/enterprise_risk_management_guide.md), [MAS Regulations](../banking/mas_regulations_guidelines_guide.md), [Risk Data Aggregation / BCBS 239](../banking/risk_data_aggregation_guide.md), all cross-ref'd in §10)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** this pass had **live web access** (web_search + web_extract). Verified at primary sources: Nygard's original article (thinkrelevance.com, 15 November 2011; now archived at cognitect.com/blog/2011/11/15/documenting-architecture-decisions), the adr-tools README and its src/ tree and raw command scripts (github.com/npryce/adr-tools), the MADR template and README (github.com/adr/madr — the madr.dev domain itself was observed suspended, see §13), the Log4brains README (github.com/thomvaill/log4brains), and adr.github.io (definitions, background, media). Everything verified this pass is marked ✅ in the [Claims Audit](#12-the-claims-audit); anything that could not be verified is flagged ⚠ or ❌ there and in [What Could Not Be Verified](#13-what-could-not-be-verified). Nothing in this guide is fabricated; where the brief demanded verification and the tools could not deliver it, the guide says so plainly.
> **Last Updated:** August 2026

### Series Context: Where This Guide Sits

This guide is the **decision-recording anchor** of the research repo. The repository is a curated knowledge base of long-form technical guides; ADRs are the short-form complement — the one-page records an architect actually writes *during* a project, of which each sibling guide is the deep-dive. The division of labour:

- **The practice guide** — this guide owns the *discipline*: what an ADR is, where it came from, the templates, the statuses, the log, the tooling, and how the practice plugs into governance and regulation.
- **The platform angle** — [AI Platform Engineering](ai_platform_engineering_guide.md) owns the *platform-team* canon (IDP, golden paths, Team Topologies team types). This guide cross-refs it in §9.4 for decision ownership instead of re-deriving it.
- **The banking context** — [Kafka](../banking/kafka_guide.md) owns the event-backbone platform behind the §11 worked example; [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md), [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md), and [Risk Data Aggregation](../banking/risk_data_aggregation_guide.md) own the regulated-banking detail that §10 references rather than re-derives.
- **The author's angle** — written from the Cymbal Bank architecture desk: the worked example (§11) is deliberately Cymbal Bank-flavored, and the governance framing (§9.5) reflects how a bank's Architecture Review Board actually consumes ADRs.

**Verification convention used throughout:** ✅ = verified this pass against a primary source (or already verified in a cross-referenced sibling guide's ledger); ⚠ = flagged (not re-verified this pass, single-source, or held from the author's knowledge base); ❌ = disputed or false as stated (with the correction in the text). Unmarked statements are structural or methodological knowledge presented as such. The consolidated list is the [Claims Audit](#12-the-claims-audit); the residual gaps are in [What Could Not Be Verified](#13-what-could-not-be-verified).

**How to use this guide:** read §1–§2 for the concept and its origin (the mental model every later section assumes), §3–§5 for the form (templates, statuses, the log), §6–§7 for the practice (when to write, supersession), §8 for the tooling, §9–§10 for the organizational and regulatory angles, and §11 for the complete worked example — the three-ADR mini-series that shows the whole lifecycle in one sitting. §12 is the audit, §13 the honest gaps, §14 the glossary. The one-page summary is §1.1; the closing paragraph is the last thing you read.

---

## Table of Contents

1. [The Overview: What an ADR Is](#1-the-overview-what-an-adr-is)
2. [The Origin: Nygard's "Documenting Architecture Decisions" (2011)](#2-the-origin-nygards-documenting-architecture-decisions-2011)
3. [The Template Family](#3-the-template-family)
4. [The Statuses and the Lifecycle](#4-the-statuses-and-the-lifecycle)
5. [The ADR Log: Numbering, Naming, and the Repository Convention](#5-the-adr-log-numbering-naming-and-the-repository-convention)
6. [The Practice: When to Write an ADR](#6-the-practice-when-to-write-an-adr)
7. [The Supersession Chains](#7-the-supersession-chains)
8. [The Tooling](#8-the-tooling)
9. [The Organizational Angle: Governance, Guardrails, and Ownership](#9-the-organizational-angle-governance-guardrails-and-ownership)
10. [The Regulated-Banking Angle: The Decision Trail as Audit Evidence](#10-the-regulated-banking-angle-the-decision-trail-as-audit-evidence)
11. [The Worked Example: The Cymbal Bank Event-Backbone Mini-Series](#11-the-worked-example-the-cymbal-bank-event-backbone-mini-series)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [The Glossary](#14-the-glossary)

---

## 1. The Overview: What an ADR Is

### 1.1 The Short Answer

An **Architecture Decision Record (ADR)** is a short, self-contained document that captures **one** significant architecture decision and the reasoning behind it, in three parts: the **Context** (the forces at play), the **Decision** (the response to those forces), and the **Consequences** (the resulting context, positive and negative). It is stored in version control next to the code it shapes, numbered sequentially, and treated as immutable except for its status. The collection of ADRs in a project is its **decision log** — a running, reviewable history of *why* the architecture is the way it is.

The definition of the underlying concept comes from the ADR community (adr.github.io): an **Architectural Decision (AD)** is "a justified design choice that addresses a functional or non-functional requirement that is architecturally significant" ✅, and an **Architecturally Significant Requirement (ASR)** is "a requirement that has a measurable effect on the architecture and quality of a software and/or hardware system" ✅. An ADR captures a single AD and its rationale ✅ — "along with its trade-offs and consequences" ✅. The adr.github.io site is explicit that ADR usage "can be extended to design and other decisions ('any decision record')" ✅.

Three properties define the form, and all three trace back to Nygard's 2011 article (§2):

1. **Short and self-contained.** "The whole document should be one or two pages long" ✅. Nobody reads large documents, and "large documents are never kept up to date" ✅ — so the unit of documentation is the smallest document that can stand alone: one decision, one file, one or two pages.
2. **One decision per record.** "One ADR describes one significant decision for a specific project" ✅. Not a design, not a plan, not a status report — a single decision, so that each record can be accepted, superseded, or deprecated independently.
3. **Written for the future.** "We will write each ADR as if it is a conversation with a future developer" ✅ — full sentences, value-neutral context, active voice in the decision, and every consequence listed, not just the flattering ones.

### 1.2 The Two Failure Modes the ADR Exists To Prevent

Nygard's motivation is best read in his own words ✅. When a new person meets a past decision they do not understand, they have only two options, both bad:

- **Blindly accept the decision** — fine if the decision is still valid, but if the context has changed and the decision should be revisited, the team accumulates dead weight; "the development team becomes afraid to change anything and the project collapses under its own weight" ✅.
- **Blindly change it** — fine if the decision needed reversing, but "changing the decision without understanding its motivation or consequences could mean damaging the project's overall value without realizing it" ✅ (the classic case: the decision quietly supported a non-functional requirement nobody has tested yet).

An ADR gives the future developer a third option: **understand the decision, then either keep it or reverse it deliberately.** That is the entire value proposition — the record is the antidote to both blind acceptance and blind reversal.

### 1.3 What Belongs in an ADR

Nygard's decision statement defines the scope: the team "will keep a collection of records for 'architecturally significant' decisions: those that affect the structure, non-functional characteristics, dependencies, interfaces, or construction techniques" ✅. Five triggers, all in the original article:

| Trigger | Example |
|---|---|
| **Structure** | Monolith vs microservices; layered vs hexagonal; where the boundaries are |
| **Non-functional characteristics** | The event backbone's durability and ordering guarantees; the latency budget; the recovery-time objective |
| **Dependencies** | Adopting Kafka, a schema registry, a managed cloud service; retiring a legacy broker |
| **Interfaces** | The contract between systems — Avro schemas, API versions, message formats |
| **Construction techniques** | Build vs buy, framework adoption, the team's delivery mechanics |

The counter-definition is equally important: if a choice does not affect at least one of these five, it is probably not an ADR. A library choice with no architectural ripple, a code-style preference, a sprint-level implementation detail — those belong in the code review and the commit message, not in the decision log.

### 1.4 The Decision Log and Its Properties

The **decision log** is the aggregate: "The collection of ADRs created and maintained in a project constitute its decision log" ✅. Three properties make the log trustworthy, and they all come from Nygard's original prescriptions:

- **Numbered, sequentially and monotonically.** "ADRs will be numbered sequentially and monotonically. Numbers will not be reused" ✅ — a deleted decision leaves a gap, never a renumbering, so references stay stable forever.
- **Immutable except for status.** The record's content never changes after acceptance; only its status does (Proposed → Accepted → Superseded/Deprecated/Amended — §4). Log4brains states the doctrine crisply: "an ADR is immutable. Only its status can change. Thanks to this, your documentation is never out-of-date! ... it was at least true one day!" ✅
- **Reversal is recorded, not erased.** "If a decision is reversed, we will keep the old one around, but mark it as superseded. (It's still relevant to know that it *was* the decision, but is *no longer* the decision.)" ✅

This last property is what turns the log into a **decision trail** — a chronological, auditable record of what was decided, why, and what replaced it. It is the same property that makes ADRs valuable as audit evidence in a regulated bank (§10): a superseded decision is not a failure to hide, it is a data point that proves the governance process worked.

---

## 2. The Origin: Nygard's "Documenting Architecture Decisions" (2011)

### 2.1 The Article and the Date

The practice of ADRs was popularized by Michael Nygard's blog post **"Documenting Architecture Decisions," published 15 November 2011** on the Relevance (thinkrelevance.com) blog — the article's canonical URL is http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions, and it is now archived at cognitect.com/blog/2011/11/15/documenting-architecture-decisions (both locations observed and extracted this pass) ✅. The date is verified from the article page itself ("Michael Nygard — November 15, 2011") ✅, and it is corroborated by two independent sources: adr.github.io calls it "the blog post from 2011 by Michael Nygard that popularized the concept" ✅, and the Log4brains README states "The term ADR became popular in 2011 with Michael Nygard's article" ✅. The article is published under a CC0 public-domain dedication ✅.

The article is, fittingly, **itself formatted as an ADR** — it walks through Context, Decision, Status ("Accepted."), and Consequences, and then adds an experience report ✅. This is the genre's founding gesture: the method's own origin document demonstrates the form.

### 2.2 What the Article Prescribes

The Decision section of the article contains the original specification ✅, and every element of today's practice is in it:

- **Storage:** "We will keep ADRs in the project repository under doc/arch/adr-NNN.md" ✅ — in version control, next to the code, from day one.
- **Format:** "We should use a lightweight text formatting language like Markdown or Textile" ✅.
- **Numbering:** "ADRs will be numbered sequentially and monotonically. Numbers will not be reused" ✅.
- **Reversal:** reversed decisions are kept and marked superseded ✅ (quoted in §1.4).
- **The parts:** Title, Context, Decision, Status, Consequences — with precise instructions for each (quoted in §3.1).
- **Length and voice:** "The whole document should be one or two pages long"; "write each ADR as if it is a conversation with a future developer"; full sentences, active voice ("We will …"); bullets only for visual style, never as an excuse for sentence fragments — "(Bullets kill people, even PowerPoint bullets.)" ✅.
- **Statuses:** "A decision may be 'proposed' if the project stakeholders haven't agreed with it yet, or 'accepted' once it is agreed. If a later ADR changes or reverses a decision, it may be marked as 'deprecated' or 'superseded' with a reference to its replacement" ✅.

The article's experience report adds the earliest evidence that the practice worked: Nygard's teams had used the format "since early August" 2011, six to ten developers had rotated through ADR-using projects, and "all of them have stated that they appreciate the degree of context they received by reading them" ✅. The report also answers the obvious objection — that version control makes records less accessible to non-developers — with the observation that GitHub renders Markdown automatically, "so it looks just as friendly as any wiki page would" ✅.

### 2.3 The Lineage: What Came Before and After

Nygard did not invent the idea of recording architectural decisions; he invented the *lightweight record* that made the idea practical. The lineage, verified at primary/secondary sources this pass:

- **Philippe Kruchten** — Nygard's article thanks Kruchten "for discussing the importance of architecture decisions," linking to his IEEE Software work (the linked DOI is 10.1109/MS.2009.52) ✅. Kruchten's "Architectural Decisions: The Making Of" (2020) is listed by adr.github.io as providing "a history on architecture decision recording since the late 1990" ✅.
- **Zdun et al.** — the adr.github.io organization states its work "is based on the guidelines and principles in 'Sustainable Architectural Decisions' by Zdun et al." (InfoQ), which introduced the **Y-statement** format ("In the context of …, facing …, we decided … to achieve …, accepting …") ✅.
- **The adr GitHub organization** (github.com/adr) — formed to motivate AD capture, strengthen tooling, and provide public knowledge pointers ✅. Its aims are listed on adr.github.io ✅.
- **The template literature** — a WICSA 2015 paper compares seven ADR templates ✅ (referenced from adr.github.io); Michael Keeling wrote about ADRs in IEEE Software (2022, "Love Unrequited: The Story of Architecture, Agile, and How Architecture Decision Records Brought Them Together") ✅; and the Azure Well-Architected Framework adopted ADRs, with adr.github.io itself listed as a reference (adr.github.io notes the WAF feature dated 2024-11-10) ✅.
- **The tooling wave** — Nat Pryce's adr-tools CLI (2016-era, GPL-3.0, 5.6k stars at observation) ✅, Oliver Kopp / Anita Armbruster / Olaf Zimmermann's MADR template (ZEUS 2018 paper, "Markdown Architectural Decision Records: Format and Tool Support") ✅, and Thomas Vaillant's Log4brains (2020, Apache-2.0) ✅ — all covered in §8.

### 2.4 Why the Practice Stuck

The durability of ADRs is not an accident of fashion; it follows from the design of the artifact. The article's Context section diagnoses the problem that every other documentation approach fails: agile projects make decisions continuously, "not all of them will be done when the project begins" ✅, and the two candidate documentation strategies both fail — the big specification document is never read and never updated, and no documentation at all leaves the team at the mercy of blind acceptance and blind reversal. The ADR is the minimal unit that breaks both failure modes: small enough to maintain, explicit enough to preserve rationale, and versioned enough to be trustworthy. Every later development — MADR's structured options analysis, Log4brains' published knowledge base, the governance and audit uses of §9–§10 — is an elaboration of that same bet.

---

## 3. The Template Family

An ADR is defined less by its exact headings than by its discipline — one decision, three movements (context, decision, consequences). But in practice the community has converged on a small family of templates, and most teams pick one and standardize. The two canonical members are Nygard's original template (the genre's default) and the MADR template (the structured, options-analysis variant). A team that has chosen a template should treat it as a local standard: template choice is itself a decision worth an ADR (adr-tools' very first generated ADR is "Record architecture decisions" — the meta-record that the log exists ✅).

### 3.1 Nygard's Original Template

The five parts, in Nygard's own words from the 2011 article ✅:

- **Title** — "These documents have names that are short noun phrases. For example, 'ADR 1: Deployment on Ruby on Rails 3.0.10' or 'ADR 9: LDAP for Multitenant Integration'" ✅.
- **Context** — "This section describes the forces at play, including technological, political, social, and project local. These forces are probably in tension, and should be called out as such. The language in this section is value-neutral. It is simply describing facts" ✅.
- **Decision** — "This section describes our response to these forces. It is stated in full sentences, with active voice. 'We will …'" ✅.
- **Status** — "proposed" until stakeholders agree, "accepted" once agreed; later ADRs may mark it "deprecated" or "superseded" with a reference to its replacement ✅.
- **Consequences** — "This section describes the resulting context, after applying the decision. All consequences should be listed here, not just the 'positive' ones" ✅.

The canonical template file as used by adr-tools (its default template "follows the style described by Michael Nygard" ✅ — verified in the `adr new` source script):

```markdown
# NUMBER. TITLE

Date: DATE

**Status**

STATUS

**Context**

The forces at play — technological, political, social, and project-local —
probably in tension. Value-neutral language; just the facts.

**Decision**

We will <the response to those forces, active voice>.

**Consequences**

What becomes easier, what becomes harder, and what is now required.
Positive, negative, and neutral — all of them.
```

Note the header convention: adr-tools fills in NUMBER, TITLE, DATE, and STATUS from the command line (default STATUS is "Accepted") ✅ — so the Status line carries the current lifecycle state, and the record's history of status changes is tracked via links to superseding ADRs rather than by rewriting the file ✅. (The fenced example above uses bold section labels in the running text of this guide's mini-records for compactness; the section structure is identical.)

### 3.2 The MADR Template

**MADR — "Markdown Architectural Decision Records"** — is the structured variant maintained by the adr GitHub organization (github.com/adr/madr), created by Oliver Kopp, Anita Armbruster, and Olaf Zimmermann (the format and its tool support were presented at ZEUS 2018) ✅. Its home page is adr.github.io/madr/ ✅; the project README notes that the user documentation lives there ✅. The template files are `adr-template.md` (all sections, with explanations), `adr-template-minimal.md` (mandatory sections only), and bare versions of both ✅. The project is dual-licensed MIT OR CC0-1.0 ✅ and is published to npm as `madr` ✅. ⚠ One verification gap: the **madr.dev domain itself was observed to be suspended** during this pass (the extraction returned an account-suspended page) — the template below was verified from the GitHub repository's raw template file instead, which is the authoritative source ✅.

The MADR template, verified from the repository's `template/adr-template.md` ✅, has this structure: an optional YAML front-matter block (status, date, decision-makers, consulted, informed — the RACI-style metadata), then Context and Problem Statement, optional Decision Drivers, Considered Options, Decision Outcome (chosen option + justification), optional Consequences (Good/Bad), optional Confirmation (how compliance with the decision will be verified), optional Pros and Cons of the Options, and optional More Information. The status vocabulary in the front matter is explicitly open-ended: `{proposed | rejected | accepted | deprecated | … | superseded by ADR-0123}` ✅.

```markdown
---
status: proposed
date: {YYYY-MM-DD when the decision was last updated}
decision-makers: {list everyone involved in the decision}
consulted: {list everyone whose opinions are sought}
informed: {list everyone who is kept up-to-date on progress}
---

# {short title, representative of solved problem and found solution}

**Context and Problem Statement**

{Describe the context and problem statement, e.g., in free form using two to
three sentences or in the form of an illustrative story. Make the scope of the
decision explicit, for instance, by calling out structural architecture
elements (components, connectors, ...).}

**Decision Drivers**

* {decision driver 1, for instance, a desired software quality, faced concern,
  constraint or force}
* {decision driver 2}

**Considered Options**

* {title of option 1}
* {title of option 2}

**Decision Outcome**

Chosen option: "{title of option 1}", because {justification, e.g., only
option which meets k.o. criterion decision driver | resolves force | comes out
best (see below)}.

### Consequences

* Good, because {positive consequence}
* Bad, because {negative consequence}

### Confirmation

{Describe how the implementation / compliance of the ADR can/will be
confirmed. Is there any automated or manual fitness function? If so, list it
and explain how it is applied. A design/code review or a test with a library
such as ArchUnit can help validate this.}

**Pros and Cons of the Options**

### {title of option 1}

* Good, because {argument a}
* Neutral, because {argument c}
* Bad, because {argument d}

### {title of option 2}

* Good, because {argument a}
* Bad, because {argument b}

**More Information**

{Additional evidence/confidence for the decision outcome; team agreement;
when/how the decision should be realized; if/when it should be re-visited.}
```

(All section text above is condensed from the actual template's guidance ✅; the heading structure is verbatim ✅.)

The MADR template's three distinctive contributions: (1) **Considered Options** forces the author to show that alternatives were actually evaluated, not just that one was chosen; (2) **Decision Drivers** makes the forces (the Nygard Context) explicit as a checklist; (3) **Confirmation** — remarkable for a template — explicitly asks "Is there any automated or manual fitness function? If so, list it and explain how it is applied" ✅, which is the direct bridge to the architectural-fitness-function practice of §9.3.

### 3.3 Other Templates in the Family

- **The Y-statement** (Zdun et al., via adr.github.io ✅): a single-sentence template — "In the context of ⟨situation⟩, facing ⟨concern⟩, we decided to ⟨option⟩ to achieve ⟨quality⟩, accepting ⟨downside⟩." Excellent as a one-liner summary; often used as the first line of a longer record.
- **The lightweight/agile variant** (Joel Parker Henderson's widely-used collection, github.com/joelparkerhenderson/architecture_decision_record — referenced by Log4brains ✅): Title/Status/Context/Decision/Consequences with the option to add "Alternatives" and "Positive/Neutral/Negative consequences" bullets. Log4brains ships MADR as its default but supports custom templates ✅.
- **Mark Richards' framing** ("ADRs and Architecture Stories," listed by adr.github.io ✅) treats the ADR as the story of a decision, starting from Nygard's template.
- **The seven-template comparison** (WICSA 2015, referenced by adr.github.io ✅) is the scholarly survey if a team wants to choose deliberately.

### 3.4 Choosing a Template

The choice is a governance decision, not a taste decision, and the defaults are sensible: **start with Nygard's original** (lowest ceremony, matches adr-tools out of the box, sufficient for most records) and **move to MADR** when the team needs the options-analysis discipline — typically in regulated or high-stakes contexts where a regulator or an architecture board will want to see that alternatives were considered and rejected with reasons (§10). The repo's house style for this guide's worked example (§11) uses the Nygard template, matching adr-tools' default; a MADR rendering of the same decisions would differ only in structure, not in content.

---

## 4. The Statuses and the Lifecycle

### 4.1 The Status Vocabulary

The status field is the only part of an ADR that changes after creation (Log4brains: "an ADR is immutable. Only its status can change" ✅). The vocabulary, verified against Nygard's article ✅ and the MADR template's front matter ✅:

| Status | Meaning | Source |
|---|---|---|
| **Proposed** | Drafted; stakeholders have not yet agreed. The decision is on the table for review. | Nygard 2011 ✅; MADR front matter ✅ |
| **Accepted** | Agreed by the stakeholders; the decision is now live architecture. | Nygard 2011 ✅; adr-tools default (new ADRs start "Accepted" ✅) |
| **Superseded** | A later ADR replaced this decision; the record is kept but is no longer the decision. Includes a reference to the replacement. | Nygard 2011 ✅; MADR "superseded by ADR-0123" ✅ |
| **Deprecated** | The decision is no longer recommended (e.g., the technology was abandoned upstream), without a specific replacement ADR yet. | Nygard 2011 ✅ (listed as an alternative to superseded); MADR front matter ✅ |
| **Rejected** | Considered and explicitly not chosen (MADR front matter ✅). Useful for recording options the team decided *against*, so the reasoning does not get re-litigated. | MADR ✅ |
| **Amended** | Partially modified by a later ADR while the core decision stands. adr-tools models this as a typed link (`adr link 12 Amends 10 "Amended by"` ✅) rather than a status; teams using MADR-style front matter can also write "Amended by ADR-XXXX" (⚠ team convention, not a fixed standard). | adr-tools link syntax ✅ |

### 4.2 The Lifecycle

The canonical lifecycle is a small state machine:

```text
                 +---------------------------+
                 |                           v
[Drafted] -> Proposed -> Accepted -> Superseded
                 |           |         ^
                 |           |         |
                 +-----------+---> Deprecated
                 |           |
                 +------> Rejected (never accepted)
                 |
                 +------> Amended (partial change; core decision stands)
```

- **Proposed → Accepted** is the normal path: a decision is drafted, reviewed (a decision review gate, §9.2), and agreed. adr-tools' `adr new` creates records already at "Accepted" ✅ — the tool assumes the author has done the review before creating the file, or will flip the status by editing it; teams that want a formal Proposed phase simply edit the status line before merging.
- **Accepted → Superseded** is the reversal path: a later ADR explicitly replaces this one, and both directions of the link are recorded (the old ADR says "Superseded by ADR-N", the new one says "Supersedes ADR-M") — this is exactly what adr-tools' `adr new -s M` does automatically ✅ (§7).
- **Accepted → Deprecated** is the abandonment path: the decision is no longer valid (vendor end-of-life, regulatory change) but no single replacement has been recorded yet.
- **Amended** is the partial-change path: the core decision stands but a detail changed; the amendment is recorded as a typed link (adr-tools' `-l "5:Amends:Amended by"` ✅), keeping the original record intact.

### 4.3 Immutability and Why It Matters

The immutability rule — *content never changes after acceptance; only status changes* — is the load-bearing wall of the whole practice. It is what makes the log a **decision trail** rather than a wiki: because records are never rewritten, the log preserves exactly what was decided at each point in time, in the words of the people who decided it. That property is what regulators (§10.1), successors (§10.2), and the architecture board (§9) all rely on. The discipline this imposes on the author: an ADR must be written well enough *before* acceptance that it never needs editing afterward — which is why Nygard demanded full sentences and a value-neutral Context ✅.

### 4.4 The Status of the Superseded Record

A common misunderstanding is that a superseded ADR is a failed ADR. The opposite is true: **supersession is the lifecycle working as designed.** The superseded record remains in the log, still numbered, still readable, still linked — it documents that the decision *was* made, *why* it was made, and *what replaced it*. Nygard's own words: "It's still relevant to know that it *was* the decision, but is *no longer* the decision" ✅. The §11 worked example shows this in practice: ADR-0001 ends up superseded by ADR-0003, and the log is *better* for it — the Kafka-adoption decision stands, the legacy-broker clause is visibly withdrawn, and a future architect can reconstruct the entire event-backbone history from three files.

---

## 5. The ADR Log: Numbering, Naming, and the Repository Convention

### 5.1 The Numbered Log

The log is a flat, numbered sequence of files. Nygard prescribed the numbering in 2011 ("ADRs will be numbered sequentially and monotonically. Numbers will not be reused" ✅), and the modern tooling implements it mechanically: adr-tools computes the next number as `max(existing numbers) + 1` and formats it zero-padded to four digits (`printf "%04d"`) ✅ — so the log reads 0001, 0002, … 0012, 0013. The zero-padding is what keeps the log alphabetically sortable as plain filenames: `ls` and `find | sort` (which is literally what `adr list` does ✅) present the records in decision order without any index file.

Numbering rules that matter in practice:

- **Never reuse a number** (Nygard ✅). If a proposed ADR is withdrawn before acceptance, the number is retired — the gap is a feature, because any reference to "ADR-0017" stays unambiguous forever.
- **Numbers are references.** In adr-tools, any command that takes an ADR argument accepts "a reference (number or partial filename)" ✅ — so `adr new -s 12 …` and `adr link 12 Amends 10 "Amended by"` ✅ both address records by number.
- **The log is append-only.** New records get the next number; nothing is ever inserted in the middle. This is what makes the log a *trail*: the numbering order is the decision order.

### 5.2 File Naming: NNNN-title-with-dashes.md

The de facto standard filename is `NNNN-title-with-dashes.md`. The mechanics are verified in the adr-tools `adr new` script ✅: the title is slugified — non-alphanumeric characters become dashes, uppercase becomes lowercase, leading/trailing non-alphanumerics are stripped — and the file is written as `<number>-<slug>.md`. Concretely, `adr new Use MySQL Database` produces `0001-use-mysql-database.md` ✅ (the script's own usage example). The MADR project uses the same shape: "For each ADR, copy the template to `nnnn-title.md` and adapt" ✅, stored under `docs/decisions` ✅.

| Command | Resulting file |
|---|---|
| `adr new Adopt Kafka as the event backbone` | `0001-adopt-kafka-as-the-event-backbone.md` |
| `adr new -s 1 Retire the legacy message broker` | `0002-retire-the-legacy-message-broker.md` (and 0001's status is updated) ✅ |
| `adr new Use MySQL Database` (adr-tools' own example) | `0001-use-mysql-database.md` ✅ |

### 5.3 Where the Log Lives: The Repository Convention

The records live **in the repository, next to the code** — that was Nygard's original prescription (`doc/arch/adr-NNN.md` ✅), and it is the reason ADRs survive: they travel with the codebase, they are reviewed in the same pull requests, and they are versioned by the same git history. The three common directory conventions, all observed in the wild:

- `doc/adr/` — **adr-tools' default** ("ADRs are stored in a subdirectory of your project as Markdown files. The default directory is `doc/adr`" ✅; `adr init` lets you override, e.g. `adr init doc/architecture/decisions` ✅).
- `docs/decisions/` — **MADR's convention** ("Copy it into `docs/decisions`" ✅).
- `docs/adr/` — the widely used variant of the adr-tools default (⚠ convention, not tool-enforced; any directory works since `adr init <dir>` takes the path ✅).

For a bank's architecture-governance context (§9.5) the directory convention is usually fixed by the architecture board — a single repository-wide `docs/adr/` for platform-level decisions, with product repositories running their own logs. The key invariant is the same everywhere: **the log must be in the same version-control unit as the code it governs**, so that a checkout of any release contains the decisions that shaped it.

### 5.4 The Log's Self-Description and Index

A well-formed log carries its own meta-records:

- **ADR-0001 is usually the meta-record.** `adr init` "will create a directory … containing the first ADR, which records that you are using ADRs to record architectural decisions and links to Michael Nygard's article" ✅ — the canonical "Record architecture decisions" ADR. The log documents its own existence before it documents anything else.
- **A generated table of contents** keeps the log navigable: `adr generate toc` ✅ produces a Markdown index of all records, and `adr generate graph` ✅ produces a DOT graph of the links between records — both typically wired into the docs build (§8.1).
- **The log index can be a single README** in the ADR directory, maintained by hand or by the tool's output — the choice is house style; the requirement is that a newcomer can see the whole log and the current status of every record at a glance (the §11.5 table is the worked example of such an index).

---

## 6. The Practice: When to Write an ADR

### 6.1 The Trigger: Significant, Hard-to-Reverse, at the Point of Commitment

An ADR is written at the **point of commitment** of a decision that is **architecturally significant** (affects structure, non-functional characteristics, dependencies, interfaces, or construction techniques — Nygard's scope ✅) and **hard to reverse** (the cost of undoing it grows with every day it stands). The test is a conjunction of three questions:

1. **Is it significant?** Does it change the structure, a non-functional property, a dependency, an interface, or a construction technique (§1.3)? If none of the five, it is not an ADR.
2. **Is it hard to reverse?** Would undoing it later cost more than writing one page now? A choice that is trivially reversible (a library swap isolated behind an interface) can wait for the code review; a choice that shapes the platform for years (the event backbone, the schema strategy, the deployment model) cannot.
3. **Is this the point of commitment?** The record must be written *when the decision is made*, not after the fact. Nygard's own experience report notes that ADRs captured "longer-term intentions" so that teams "don't inadvertently make those future changes harder" ✅ — the record is a commitment device, and a retrospective ADR is an archaeology project.

The discipline generalizes as: **if a future developer will ask "why?", write the record now.** The cost of an ADR is one or two pages and a review slot; the cost of its absence is a guessing game played by every successor and every auditor who touches the system (§10).

### 6.2 What Is Not an ADR

Equally important is what stays *out* of the log, to keep the signal high:

- **Design descriptions** — how a component works internally belongs in design docs and code comments; ADRs record *why it is the way it is*.
- **Status reports and plans** — ADRs are immutable; anything that changes weekly is not a decision.
- **Trivial choices** — a reversible library pick, a naming convention, a sprint decision (§6.1).
- **Everything-is-an-ADR sprawl** — a log with hundreds of records about minor choices drowns the few that matter; the governance failure mode of ADRs is noise, not silence.
- **Meeting minutes** — the log records *decisions*, with their rationale; the discussion belongs in the issue tracker or the review record.

### 6.3 The Documentation Spectrum: ADR vs RFC vs Design Doc vs Wiki

ADRs are one rung on a documentation spectrum, and the craft is choosing the right instrument per question. The spectrum, and when each fits:

| Instrument | Unit | Immutable? | Audience | Fits when… |
|---|---|---|---|---|
| **ADR** | One decision + rationale | Yes (except status) | Future developers, architects, auditors | A significant, hard-to-reverse decision at the point of commitment; the "why" must survive |
| **RFC (Request for Comments)** | A proposal with discussion | No — it *is* the discussion | The deciding team, reviewers | The decision is genuinely open and needs structured debate before commitment; the RFC's history *becomes* the rationale |
| **Design doc** | A design (often several decisions) | No — living document | Implementers | The question is "how will this work?" — the design's shape, not a single decision |
| **Wiki** | Anything | No — edited freely | The whole organization | Reference knowledge, glossaries, runbooks, tribal knowledge that needs a home; never for decisions that must be auditable |

The relationship is complementary, not competitive: an RFC proposes and debates, the ADR records the commitment that ends the debate, the design doc describes the implementation, and the wiki carries the ongoing reference material. The decision trail lives only in the ADR column — which is why, in a regulated context (§10), the ADR is the only one of the four that functions as audit evidence: it is the only one that is immutable, dated, numbered, and linked.

### 6.4 Lightweight vs Heavyweight: The Cost-Benefit

The entire ADR movement is a bet against heavyweight documentation — Nygard's Context section is an argument that "large documents are never kept up to date" and "nobody ever reads large documents" ✅. The economics, made explicit:

- **Cost of an ADR:** one or two pages (Nygard ✅), written by the person making the decision, reviewed at the existing review gate (§9.2), stored in the existing repo. Marginal cost: roughly the length of a good pull-request description.
- **Cost of no ADR:** every future reader re-derives the rationale from code archaeology; every reversal is done blind (the two failure modes of §1.2); every audit and onboarding question is answered from memory.
- **The threshold rule:** if a decision is significant and hard to reverse, the ADR is cheaper than the alternative *even if it is never read* — because its existence (and the discipline of writing it) is what forces the rationale to exist at all. The record's value is realized the first time someone does *not* have to ask.

The heavyweight alternatives (full design docs, architecture specifications) still have their place — they are the right instrument for designing a system; they are the wrong instrument for recording why a decision was made. ADRs do not replace design; they replace the *loss* of design rationale.

---

## 7. The Supersession Chains

### 7.1 The Mechanics: ADR-X Supersedes ADR-Y

Supersession is the mechanism by which the decision trail stays honest when the architecture moves. The mechanics, exactly as the tooling implements them (verified in the adr-tools scripts ✅):

1. A new ADR is created that explicitly names its predecessor: `adr new -s 12 Use PostgreSQL Database` — "a reference (number or partial filename) of a previous decision that the new decision supercedes" ✅.
2. **The new ADR receives a "Supercedes" link** to the old record (inserted into its Status section as a Markdown link) ✅.
3. **The old ADR receives a "Superceded by" link** to the new record, and its "Accepted" status is removed ✅ (in adr-tools, the `_adr_remove_status` helper strips the old status line so the record now reads "Superceded by ADR-N").
4. Multiple predecessors are supported — `adr new -s 3 -s 4 -l "5:Amends:Amended by" …` supersedes ADRs 3 and 4 while amending ADR 5 in one command ✅.

So a supersession is not a hand-wave; it is a pair of bidirectional, typed links plus a status change on the old record. The old record is **never deleted and never rewritten** — only its status line and its link set change (Nygard: "we will keep the old one around, but mark it as superseded" ✅). The result, rendered in the log:

```text
0001-use-mysql-database.md        Status: Accepted
0002-use-postgresql-database.md   Status: Accepted        (created with `adr new -s 1`)
0001-use-mysql-database.md        Status: Superceded by ADR-0002   (status updated automatically)
```

### 7.2 What Happens to the Superseded ADR's Status

The superseded record's status becomes **"Superseded by ADR-N"** — a status that *carries its own replacement reference*, which is exactly the vocabulary Nygard prescribed ("marked as 'deprecated' or 'superseded' with a reference to its replacement" ✅) and the MADR front matter models ("superseded by ADR-0123" ✅). Three properties follow:

- **The old decision remains fully readable.** Its Context, Decision, and Consequences are untouched — a future reader can still reconstruct what was decided, why, and what it cost.
- **The status makes the current state unambiguous.** Nobody reading the log can mistake a superseded decision for a live one: the status line says so, and says *what replaced it*.
- **The link makes the chain walkable.** From any record you can follow "Superseded by" forward and "Supercedes" backward, reconstructing the full evolution of a decision area — the §11 mini-series is a complete worked example of walking such a chain.

### 7.3 Supersession vs Deprecation vs Amendment

The three "later-state" statuses are often conflated; the distinction is what keeps the log precise:

| Later state | Meaning | When to use | Example |
|---|---|---|---|
| **Superseded** | Replaced by a specific later decision; the chain has a next link | The team made a new decision that displaces the old one | "Superseded by ADR-0003" |
| **Deprecated** | No longer valid/recommended, but no replacement recorded yet | The old decision's foundation collapsed (vendor EOL, regulation change) before a successor was chosen | "Deprecated: the vendor announced end-of-life" |
| **Amended** | Core decision stands; a detail changed | A partial change that must not rewrite the original record | `adr link 5 Amends 3 "Amended by"` — ADR-5 amends ADR-3 ✅ |

The heuristic: **superseded = replaced, deprecated = abandoned, amended = adjusted.** All three preserve the original record; none of them rewrites history.

### 7.4 Supersession in Practice: Chains, Trees, and Anti-Patterns

Real logs are rarely linear; they are chains and trees:

- **A chain** — ADR-0001 → ADR-0004 → ADR-0009 — records successive replacements of one decision area (the event backbone's evolution, say).
- **A tree** — one decision superseded by two successors (the decision area split), or one successor superseding two predecessors (`adr new -s 3 -s 4` ✅).
- **The anti-patterns:** (1) *superseding a superseded record* — possible and sometimes legitimate (the chain just grows), but usually a sign the team should look for the *current* record instead; (2) *rewriting a superseded record to "fix" it* — destroys the trail; (3) *silent supersession* — changing the architecture without an ADR, which leaves the log lying; (4) *supersession without consequences* — the new ADR must carry its own Consequences, because the supersession itself (the migration, the dual-run, the decommissioning) is a decision with costs. The §11.4 worked example shows a well-formed superseding ADR: it names its predecessor, records the migration, and lists what the supersession costs.

---

## 8. The Tooling

The tooling story is deliberately thin: an ADR is a Markdown file, and the tools are conveniences around the file format, not platforms that own the records. The three tools verified this pass — adr-tools, MADR, and Log4brains — each take a different slice of the workflow.

### 8.1 adr-tools (npryce/adr-tools)

**adr-tools** is "a command-line tool for working with a log of Architecture Decision Records (ADRs)" ✅ — a set of bash scripts (GPL-3.0, by Nat Pryce) that manage the numbered log. Verified against the README, the src/ tree, and the raw command scripts this pass ✅. The command set (user-facing commands in the current master's `src/`):

| Command | What it does (verified) |
|---|---|
| `adr init <dir>` | Creates the ADR directory and the first ADR ("records that you are using ADRs … and links to Michael Nygard's article") ✅. Default directory is `doc/adr`; `adr init doc/architecture/decisions` shows the override ✅. |
| `adr new <title>` | Creates the next numbered ADR file (`NNNN-slug.md`), fills the template (NUMBER/TITLE/DATE/STATUS), opens it in `$VISUAL`/`$EDITOR`, prints the filename ✅. New ADRs default to Status "Accepted" ✅. |
| `adr new -s <n> <title>` | Creates a new ADR that supersedes ADR *n*: inserts the "Supercedes" link in the new record, the "Superceded by" link in the old one, and changes the old record's status ✅. Multiple `-s` allowed ✅. |
| `adr new -l <n>:<link>:<reverse>` | Links the new ADR to an existing one with typed links, e.g. `-l "5:Amends:Amended by"` ✅. Multiple `-l` allowed ✅. |
| `adr link <src> <link> <tgt> <reverse>` | Adds a typed link between two existing ADRs, e.g. `adr link 12 Amends 10 "Amended by"` ✅. |
| `adr list` | Lists the ADR files in number order (a sorted `find` over the ADR directory) ✅. |
| `adr generate <report>` | Generates summary documentation; `adr generate` lists the available report types; `adr generate toc` produces a table of contents; `adr generate graph` produces a DOT graph of the links; `adr help generate toc` explains a report ✅. |
| `adr help` / `adr help <command>` | Built-in help ✅. |
| `adr config` | Outputs the tool's configuration (used internally to bootstrap every command) ✅. |
| `adr upgrade-repository` | Upgrades older-format repositories ✅. |
| Custom templates | If the ADR directory contains `templates/template.md`, `adr new` uses it; otherwise the default template "follows the style described by Michael Nygard" ✅. |

⚠ **One correction to the commonly-circulated command list:** the current master of adr-tools has **no `adr status` command and no `adr edit` command** as user-facing tools — the src/ tree contains `adr-config, adr-generate, adr-help, adr-init, adr-link, adr-list, adr-new, adr-upgrade-repository` (plus internal `_adr_*` helpers) ✅, and the README documents init/new/help ✅. The status *view* is `adr list` (file listing) plus reading the records; the *edit* step is your editor, invoked by `adr new`. The `_adr_status` helper exists but is an internal function, not a CLI command. Teams that remember "adr status" from older write-ups are remembering a different tool or an older branch — worth knowing before a demo.

### 8.2 Log4brains

**Log4brains** (Thomas Vaillant, Apache-2.0, 2020) is "a docs-as-code knowledge base for your development and infrastructure projects" that logs ADRs "right from your IDE" and publishes them "automatically as a static website" ✅. Verified features from the README ✅: ADRs stored in git next to the code; interactive ADR creation from the CLI; local preview with hot reload; static-site generation for GitHub/GitLab Pages or S3; a timeline menu; search; ADR metadata "automatically guessed from its raw text and git logs"; no enforced markdown structure; no required file numbering schema ("avoids git merge issues"); a customizable template with **MADR as the default**; and multi-package project support ✅. The workflow, from the README ✅:

```shell
npm install -g log4brains
log4brains init        # interactive setup; creates template files and the first ADR
log4brains preview     # local web UI with hot reload
log4brains adr new     # create a new ADR from the template
log4brains build --basePath /<repo>/log4brains   # static site for CI publishing
```

Log4brains' own README restates the doctrine in one line: "an ADR is immutable. Only its status can change" ✅ — and its goals list is a good summary of the whole practice: avoid blind acceptance and blind reversal, speed up onboarding, and "formalize a collaborative decision-making process" ✅.

### 8.3 MADR (the Project and the npm Package)

MADR is both a template (§3.2) and a tooling project: it ships the four template variants (`adr-template.md`, `-minimal`, `-bare`, `-bare-minimal`) ✅, is published to npm as `madr` ✅, and its user documentation lives at adr.github.io/madr/ ✅. The workflow it prescribes is template-copy, not scaffolding: "Copy it into `docs/decisions`. For each ADR, copy the template to `nnnn-title.md` and adapt" ✅ — a zero-dependency approach that fits any CI and any editor. Teams that want the MADR structure with command-line convenience typically pair the templates with adr-tools (custom `templates/template.md` ✅) or with Log4brains (MADR is its default ✅).

### 8.4 The Rest of the Ecosystem (light)

Two adjacent tools observed in the primary sources, for completeness: **adr-viewer** (mrwilson) — a static-site renderer that Log4brains credits as inspiration for its site generation ✅; and **Joel Parker Henderson's `architecture_decision_record` collection** — "a collection of ADR templates and examples" ✅ (credited by Log4brains ✅), useful as a pattern library for status transitions and link conventions. (⚠ Neither was deep-verified this pass; both are cited here as observed in the Log4brains acknowledgments.)

### 8.5 The Tooling Invariants

Whatever the tool, three invariants hold: **the record is a plain Markdown file in the repo** (no proprietary store — Nygard's "lightweight text formatting language" ✅); **the number and the links are the structure** (the log is a graph, and the tools exist to maintain the graph edges); and **the tools are optional** — a team with no tooling at all can run a perfect log by following §3–§7 by hand. The tools buy convenience and publishing; the discipline is the product.

---

## 9. The Organizational Angle: Governance, Guardrails, and Ownership

### 9.1 ADRs as Architectural Governance and Guardrails

An ADR log is, in organizational terms, the **memory and the boundary of the architecture function**. As governance, the log is how the architecture is made visible and reviewable: every significant decision has a named record, a named status, and a named set of links — so "the architecture" stops being whatever the last conversation decided and becomes a queryable artifact. As guardrails, the log is how the organization constrains future choices without stopping them: a decision is not a cage, it is a recorded position that a later ADR may deliberately reverse — but only *deliberately*, through the review process, with the supersession link making the change visible to everyone. That is the difference between governance by ADR and governance by decree: the decree forbids; the ADR requires *reasoned* change.

Three properties make the log governable, not just readable:

- **Status as control state.** The Architecture Review Board (ARB) grants and revokes "Accepted"; a record's status is the single source of truth for whether a decision is live (§4).
- **Links as dependency visibility.** Supersession and amendment links show which decisions depend on which — the impact of revisiting ADR-0001 is visible as the set of records that link to it (§7).
- **The log as the review agenda.** "Proposed" records are, by construction, the open decisions of the organization — the board's backlog, generated for free (§4.1).

### 9.2 Decision Review Gates

The natural integration point for ADRs is the **decision review gate**: a defined step in the workflow where a proposed decision is reviewed and either accepted or sent back. In practice this is usually one of three shapes:

- **The pull-request gate** — the ADR is a file in the repo, so the proposal *is* a pull request; the review is the code review; "Accepted" means merged. Lowest ceremony, highest velocity, and the review record is preserved by the version-control system itself. This is the shape adr-tools assumes (a new ADR opens in your editor, you commit it when done ✅).
- **The architecture-board gate** — significant or cross-team decisions are tabled at the ARB; the ADR is the submission document, the board's decision is recorded as the status change (Proposed → Accepted or Rejected), and the minutes reference the ADR number. This is the shape a bank's architecture-governance function typically requires (§9.5).
- **The hybrid gate** — routine decisions flow through the pull-request gate; a defined class of decisions (regulatory impact, data-residency impact, cross-domain interfaces) is escalated to the board. The ADR's status line records which gate it passed.

The gate and the ADR reinforce each other: the gate gives the ADR an enforcement mechanism, and the ADR gives the gate a durable, reviewable artifact. A gate with no ADR produces decisions with no rationale; an ADR with no gate produces a log that records decisions nobody agreed to.

### 9.3 Architectural Fitness Functions

The MADR template's **Confirmation** section asks the question explicitly: "Is there any automated or manual fitness function? If so, list it and explain how it is applied" ✅ — connecting the decision record to the verification of its own implementation. An **architectural fitness function** is an automated (or manual) check that the running system still conforms to a decision: a test that the event backbone's messages carry schemas, a lint rule that forbids a banned dependency, a performance test that enforces the latency budget an ADR promised. The pattern that makes this powerful is the **ADR-to-fitness-function link**: each ADR's Consequences or Confirmation section names the check that will fail if the decision is violated, so the log is not a museum but a set of executable commitments. When the check fails, the organization is forced to do what the log is designed to force: either fix the drift, or write the superseding ADR that deliberately changes the decision (§7). The repository's own house style is a manual instance of the same idea — this guide's Claims Audit (§12) is a fitness function over the guide's factual claims.

### 9.4 Decision Ownership under Team Topologies

Under Team Topologies (Skelton & Pais), decisions are owned by the team type whose cognitive load they affect — and the ADR log is the mechanism that makes ownership visible. The cross-reference here is deliberate and shallow: [AI Platform Engineering](ai_platform_engineering_guide.md) owns the platform-team canon (IDP, golden paths, the four team types, platform-as-a-product) in this repository — this section only applies its ownership model to the decision log:

- **Stream-aligned teams** own the decisions inside their stream: product-domain ADRs, interface contracts with their consumers, their local schema choices. Their log is local to their codebase.
- **Platform teams** own the decisions that reduce everyone else's cognitive load: the event backbone, the schema registry, the golden path itself. Platform ADRs are the platform's "product decisions" — written like a product spec, consumed like an API contract.
- **Enabling teams** own *advice*, not decisions: their output is the proposed ADR, handed to the team that owns the decision, who accepts or rejects it in their own log.
- **Complicated-subsystem teams** own the decisions inside their subsystem; the *interface* to the subsystem is jointly owned and recorded in both teams' logs via links.

The rule that falls out: **an ADR's owner is the team whose cognitive load the decision serves** — and the log's link structure records the ownership boundaries. A platform decision accepted in a stream team's log, or a stream decision accepted in the platform log, is an ownership smell the board should catch at the review gate (§9.2). For the full Team Topologies model — team types, interaction modes, and the platform-as-a-product operating model — read `ai_platform_engineering_guide.md` §2, which owns that ground.

### 9.5 The Cymbal Bank Architecture-Governance Context

The recurring context of this repository — Cymbal Bank, a global bank with its Singapore APAC hub — exercises the governance model as follows (this is the repo's architectural-governance frame, consistent with how the sibling banking guides describe the Cymbal Bank operating context):

- **The Architecture Review Board** is the acceptance authority for platform-level ADRs; product-domain ADRs are accepted at the delivery-team level with the ARB informed.
- **The decision classes that must reach the board:** anything with regulatory impact (§10), anything that changes a platform-wide dependency (the event backbone, the schema registry, the core banking interfaces), anything with data-residency or outsourcing implications, and anything that reverses or amends an existing board-accepted ADR.
- **The repository conventions are fixed:** platform decisions live in the platform repository's `docs/adr/`, named `NNNN-title-with-dashes.md` (§5.2), with `adr-tools` as the standard CLI and the generated TOC published with the docs build (§8.1).
- **The gate is the pull request plus the board agenda:** ADRs arrive as pull requests with a standard template, the ARB reviews the monthly decision agenda (the "Proposed" set), and the status change to Accepted is recorded in the same merge that carries the code.
- **Fitness functions are named in every board-level ADR** (§9.3), and the platform team's CI runs them — a drifted decision fails the build before it fails the audit.

None of this is load-bearing regulation; it is the governance frame the worked example (§11) assumes, and it is the bridge to the regulated-banking angle (§10): the same records that govern the architecture are the evidence that the regulators and the auditors will ask for.

---

## 10. The Regulated-Banking Angle: The Decision Trail as Audit Evidence

### 10.1 The Decision Trail as Audit Evidence

In a regulated bank, "the decision trail" stops being a metaphor. The ADR log — immutable, numbered, dated, linked, version-controlled — is a **contemporaneous record of decisions and their rationale**, which is precisely the class of evidence that audits, supervisory reviews, and internal risk functions rely on. The properties that make ADRs good engineering documentation (§1.4) are the same properties that make them good evidence:

- **Contemporaneity** — the record is written at the point of commitment (§6.1), not reconstructed after the fact; an auditor can tell the difference, and so can a successor.
- **Immutability** — records are never rewritten, only superseded (§4.3); the log cannot be quietly edited to match the present.
- **Rationale preservation** — the Context section records the forces, the Decision records the choice, the Consequences record the known trade-offs; the "why" survives the people who made the decision.
- **Lineage** — supersession links reconstruct the evolution of any decision area (§7), which is the decision-side analogue of the data lineage that BCBS 239 demands on the data side (§10.3).

The audit value is not hypothetical: the alternative — decisions reconstructed from email, meeting minutes, and memory — is exactly the "blind acceptance or blind reversal" failure mode of §1.2, scaled up to regulatory scale.

### 10.2 "Why" Preservation for Regulators and Successors

The single most valuable thing an ADR preserves is the **"why"**. For successors, the "why" is what turns a baffling architecture into a coherent one (Nygard's original motivation ✅); for regulators, the "why" is what turns a technical choice into a *defensible* choice. When a supervisor asks why the bank routes a particular class of payment events over a particular backbone, or why a control was placed where it was, the answer is not the design doc and not the vendor slide — it is the ADR: here is the context we were in, here are the options we considered, here is the decision we made, here is what we knew it would cost. A bank whose significant decisions are all recorded this way answers "why" questions in minutes; a bank without the log answers them in weeks of archaeology. The preservation requirement is therefore not a documentation nicety but an operational capability — one that the repo's banking siblings document in depth and that this guide only needs to reference: the ERM guide's treatment of the risk-appetite-to-decision chain ([Enterprise Risk Management](../banking/enterprise_risk_management_guide.md), its §6–§7), the MAS guides' treatment of board and management oversight expectations ([MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md)), and the BCBS 239 guide's treatment of lineage and documentation ([Risk Data Aggregation / BCBS 239](../banking/risk_data_aggregation_guide.md), its §2 and §6).

### 10.3 BCBS 239: Documentation and Lineage Expectations

BCBS 239 — the Basel Committee's *Principles for effective risk data aggregation and risk reporting* (January 2013, 14 principles, G-SIB compliance deadline 1 January 2016; facts carried by the sibling guide ✅) — is the regulatory anchor for the data side of the decision trail. The standard's governance-and-infrastructure principles require a bank's risk data architecture and IT infrastructure to be documented, aligned, and able to support risk aggregation and reporting in normal times and under stress; its implementation expectations reach down to **data lineage** — knowing where each data element comes from, how it is transformed, and who owns it — and to **documentation of the aggregation and reporting process** itself. The full principle-by-principle treatment is the sibling guide's ground ([Risk Data Aggregation](../banking/risk_data_aggregation_guide.md), which owns the 14 principles, the P1–P14 numbering, the data-architecture layer, and a complete BCBS 239 programme for a Cymbal Bank) — this section only draws the connection: **the ADR log is the decision-side lineage.** Just as BCBS 239 requires data lineage from source to report, the ADR log provides decision lineage from requirement to architecture: which decision produced this interface, which supersession changed that flow, which accepted ADR stands behind this control. A BCBS 239 programme that documents data lineage but not decision lineage has traced the numbers without tracing the choices that produced them. The two lineages meet in the schema registry and the event backbone of the §11 example: the Avro schemas (ADR-0002) are data lineage; the record that chose Avro is decision lineage.

### 10.4 MAS Expectations

The Monetary Authority of Singapore's expectations operate on the same logic, from the technology-risk side: sound technology risk management requires documented system architecture, controlled change management, and the ability to demonstrate that changes were reviewed and approved — expectations the repo's MAS guide carries in detail ([MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md), which owns the TRM Guidelines, the Notices, and the Cymbal Bank obligations × controls map). For the ADR practice, three MAS-relevant implications follow:

- **Change management is evidenced by the log.** An accepted ADR merged with its code is a controlled change with a documented rationale; the supersession chain (§7) shows that reversals went through the same control as the original decision.
- **Outsourcing and third-party decisions are decision records.** The choice to run the event backbone on a managed cloud service, or to outsource a component, is exactly the class of architecturally significant, hard-to-reverse decision that §6.1 says must be an ADR — and the class that MAS's outsourcing expectations say must be documented, assessed, and approved. The ADR is the natural vehicle for the architecture half of that record.
- **Board and management oversight needs a readable trail.** MAS expectations run to board-level visibility of technology risk; a board that can read a one-page ADR (Nygard's length limit ✅) can exercise oversight over decisions that a 200-page design doc would bury. The log's TOC and status view (§5.4) is the oversight surface.

### 10.5 The Risk-Management Frame: Decisions Inside the Appetite

Finally, the ADR log is the place where architecture decisions meet the bank's risk appetite. The ERM discipline ([Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) — which owns the COSO/ISO frameworks, the risk taxonomy, the three lines of defence, and the appetite-to-ICAAP chain) frames the connection: every significant architecture decision is a risk decision — it accepts some operational risk (a new dependency, a migration window, a concentration of vendors), trades one risk for another, or mitigates one. Recording the decision with its consequences is recording the risk acceptance. A bank whose ADRs state the risk implications of each decision has, in effect, an architecture-level risk register that is updated at every review gate (§9.2) and auditable at every supervisory review — the decision trail as a risk trail, which is what the §11 worked example's Consequences sections demonstrate.

---

## 11. The Worked Example: The Cymbal Bank Event-Backbone Mini-Series

### 11.1 The Scenario

Cymbal Bank (the repository's recurring bank persona — a global bank with its Singapore APAC hub) needs an event backbone: the platform that carries payments, trade-lifecycle, reference-data, and risk events between its core systems. The scenario is deliberately familiar — it is the platform documented in depth by [Kafka](../banking/kafka_guide.md) (whose §9 covers the banking context for running Kafka as the event backbone, §10 a complete Cymbal Bank event-platform design, and §6 the Schema Registry) — but here the focus is not the platform: it is the **three-record decision trail** that a working architect would write around it. The mini-series exercises every mechanism this guide has described: the Nygard template (§3.1), the statuses and lifecycle (§4), the numbered log and naming (§5), the practice triggers (§6), and — in ADR-0003 — a full supersession chain (§7).

The narrative in one paragraph: Cymbal Bank's event backbone was the **legacy message broker** — a pre-ADR, decision-by-default choice never recorded anywhere (§7.4's "silent" state). ADR-0001 records the decision to adopt Kafka, and in doing so supersedes the original legacy-broker choice; its accepted form retains the legacy broker through a migration window. ADR-0002 adds the schema registry with Avro, governing the backbone's contracts. ADR-0003 completes the story: the migration is done, and the legacy broker is retired — superseding the last remnant of ADR-0001's original choice of the legacy broker (the retention clause), and leaving the log with a clean, walkable chain. All dates are pedagogical constructions (flagged in §12 and §13); the mechanics are real.

### 11.2 ADR-0001: Adopt Kafka as the Event Backbone (Proposed → Accepted)

The record below is shown in the Nygard template (§3.1) with bold section labels, as the repository's mini-records render it. Note the **status transition shown in the Status section** — the lifecycle in the record itself — and note that the record's draft revision happened *while Proposed*, which is the one legitimate edit window (immutability binds at acceptance, §4.3):

```markdown
# ADR-0001: Adopt Kafka as the Event Backbone

Date: 2026-06-02 (proposed) · 2026-06-16 (accepted)

**Status**

Proposed (2026-06-02) → Accepted (2026-06-16, Architecture Review Board).

Draft history: the original proposal (2026-06-02) recommended the legacy
message broker's next major version as the backbone; the board review
(2026-06-16) revised the decision to adopt Apache Kafka and accepted the
revision. The original legacy-broker direction survives only in the migration
clause below, and that clause is retired by ADR-0003.

**Context**

Cymbal Bank's event backbone is the legacy message broker — a pre-ADR,
decision-by-default choice. It carries the payments, trade-lifecycle,
reference-data, and risk events between the core systems. It is reliable but
aging: it is licensed per message, which makes the volumes projected for the
next three years expensive; its replay capability is weak, which makes
reconciliation and regulatory-data rebuilds slow; its consumer model couples
consumers to the broker's clustering topology; and its vendor has announced a
migration to a new product line, forcing a decision now.

The forces in tension: (1) least-change — the legacy broker is proven and the
teams know it; (2) cost — per-message licensing does not scale with projected
event volumes; (3) resilience — the bank's operational-resilience expectations
(MAS TRM-aligned, per the MAS guide) demand replay, isolation, and
recoverability that the legacy broker provides poorly; (4) skills — the
platform team's Kafka capability is strong, and the repository's Kafka guide
documents the target platform in depth (banking/kafka_guide.md §9–§10).

**Decision**

We will adopt Apache Kafka as Cymbal Bank's event backbone, replacing the
legacy message broker for all new event traffic and for the migration of
existing flows over a defined window (see ADR-0003). The legacy broker remains
in service during the migration window for settlement-critical flows and for
consumers not yet migrated; that retention clause is the last remnant of the
original legacy-broker choice and is itself superseded by ADR-0003.

We will run Kafka with a replication factor of 3, acks=all, and the KRaft
metadata quorum (banking/kafka_guide.md §3), and we will treat topics as
governed artifacts with named owners.

**Consequences**

Positive: per-message licensing cost disappears; replay and retention become
strong, so reconciliation and regulatory-data rebuilds get faster; consumers
decouple from the broker topology; the platform team operates the backbone it
already knows; the backbone becomes the foundation for the schema discipline
(ADR-0002) and for audit-grade event retention (BCBS 239-aligned lineage).

Negative: a migration of existing flows with dual-run costs; a new operational
surface (broker fleet, monitoring, capacity planning); exactly-once semantics
must be designed per flow, not assumed; the legacy broker must be kept healthy
for the whole migration window.

Neutral: the broker topology becomes an owned, governed platform component
with named owners and a fitness function (backbone latency and durability
checks in the platform CI).
```

### 11.3 ADR-0002: Schema Registry with Avro (Accepted)

```markdown
# ADR-0002: Schema Registry with Avro for the Event Backbone

Date: 2026-06-20 (proposed) · 2026-07-01 (accepted)

**Status**

Proposed (2026-06-20) → Accepted (2026-07-01, Architecture Review Board).
Related to: ADR-0001 (the backbone this record governs).

**Context**

ADR-0001 commits the bank to Kafka as the event backbone. Kafka itself is
schema-agnostic — it stores bytes — so the backbone's contracts are whatever
producers and consumers agree on. Without a schema strategy, Cymbal Bank would
repeat the legacy broker's failure mode: undocumented payloads, drift between
producer and consumer expectations, and break-on-deploy integration failures
between the payments hub and the risk systems. The bank's data-governance
expectations (BCBS 239-aligned lineage, per banking/risk_data_aggregation_guide.md) require that the events carrying payments and risk data have
documented, versioned, auditable structures.

Forces in tension: (1) time-to-adopt — a schema strategy adds ceremony to every
event contract; (2) compatibility — consumers must keep reading old events
during rolling deployments; (3) governance — the schema is the lineage anchor
for every event; (4) skills — the platform team's streaming experience is
strongest with the toolchain documented in banking/kafka_guide.md §6.

**Decision**

We will adopt a schema registry with Apache Avro as the event format for the
event backbone. Every topic's payload is defined by an Avro schema, registered
in the schema registry, and versioned; producers and consumers resolve
compatibility through the registry (backward compatibility as the default,
per banking/kafka_guide.md §6). Schema changes are reviewed at the same
decision gate as the ADRs that own them.

**Consequences**

Positive: contracts are explicit, versioned, and auditable — the decision-side
lineage (this record) meets the data-side lineage (the schemas); incompatible
changes fail at registration instead of at deploy; new consumers self-serve
the schemas from the registry; the payments and risk systems integrate without
point-to-point payload agreements.

Negative: every event contract now carries schema-governance overhead; Avro
tooling must be adopted by all producers and consumers; the registry is a new
critical dependency that must be operated to the same standard as the broker.

Neutral: the choice is consistent with the Kafka ecosystem's default toolchain,
so hiring and training align with the platform decision (ADR-0001).
```

### 11.4 ADR-0003: Retire the Legacy Message Broker (the Superseding ADR)

ADR-0003 is the record that closes the chain. Its Status section carries the **supersession links in both directions** — "Supersedes: ADR-0001, in part" (the retention clause, i.e. the last remnant of ADR-0001's original choice of the legacy broker) and "Related to: ADR-0002" — and its Context and Consequences record the cost of the supersession, which §7.4 demands:

```markdown
# ADR-0003: Retire the Legacy Message Broker

Date: 2026-08-05 (proposed) · 2026-08-19 (accepted)

**Status**

Proposed (2026-08-05) → Accepted (2026-08-19, Architecture Review Board).
Supersedes: ADR-0001, in part — specifically the migration-window retention
clause, the last remnant of ADR-0001's original choice of the legacy broker.
Chain: pre-ADR legacy-broker choice → ADR-0001 as accepted (Kafka plus the
retention clause) → ADR-0003 (Kafka only). Related to: ADR-0002 (the schemas
the migrated flows must adopt).

**Context**

ADR-0001 accepted Kafka as the backbone but retained the legacy message broker
in service during the migration window for settlement-critical flows. The
migration is now complete: all event traffic — payments, trade lifecycle,
reference data, risk events — flows over the Kafka backbone with Avro schemas
(ADR-0002). The legacy broker now carries only residual traffic: a handful of
unmigrated consumers and the settlement-critical flows the retention clause
protected. Its per-message licensing cost is pure overhead, its operation
consumes platform-team capacity, and its continued presence creates a
dual-backbone risk profile — two systems to secure, two to recover, two to
audit — that the bank's operational-resilience expectations no longer justify.

Forces in tension: (1) risk — decommissioning a settlement-era component is
irreversible; (2) cost — dual operation is now pure overhead; (3) confidence —
the backbone's durability and exactly-once posture (banking/kafka_guide.md
§3–§4) must be proven sufficient for the settlement-critical flows before the
legacy broker is switched off; (4) audit — the decommissioning must itself be
recorded, dated, and evidenced.

**Decision**

We will retire the legacy message broker. The remaining consumers migrate to
the Kafka backbone under ADR-0002 schemas; the settlement-critical flows are
validated against the backbone's durability and ordering guarantees for one
full quarter; then the legacy broker is decommissioned, its configuration and
message history are archived for audit retention, and its licensing is
terminated.

This decision supersedes the retention clause of ADR-0001 — the last remnant
of ADR-0001's original choice of the legacy broker. ADR-0001's Kafka adoption
decision stands; only its legacy-broker clause is withdrawn. ADR-0001's status
is therefore updated to "Superseded by ADR-0003 (in part)".

**Consequences**

Positive: the dual-backbone risk profile disappears; per-message licensing
ends; platform-team capacity returns to the backbone; one system to secure,
recover, and audit instead of two; the decision trail now shows a clean chain
from the original legacy-broker choice to the Kafka-only state.

Negative: decommissioning is irreversible — a rollback would require
re-procurement; the quarter-long validation window delays the cost savings;
the archived message history must be operated for the retention period.

Neutral: the migration playbook and the validation evidence become the
reference record for future platform retirements — the audit trail now
includes a complete retirement, start to finish (§10.3–§10.4).
```

### 11.5 The Resulting Log: Chain Links and Final Statuses

After ADR-0003 is accepted, the log reads as follows — the resulting status of each record, and the links that make the chain walkable:

| ADR | Title | Resulting status | Links in the log |
|---|---|---|---|
| 0001 | Adopt Kafka as the event backbone | Accepted → **Superseded by ADR-0003 (in part)** — the Kafka adoption stands; the legacy-broker retention clause is withdrawn | "Superseded by ADR-0003 (in part)" (Status section) |
| 0002 | Schema registry with Avro | **Accepted** (unchanged — the supersession does not touch it) | "Related to ADR-0001" (Status section) |
| 0003 | Retire the legacy message broker | **Accepted** | "Supersedes ADR-0001 (in part)" and "Related to ADR-0002" (Status section) |

The chain, rendered as the graph a `adr generate graph` would draw (§8.1):

```text
[pre-ADR] legacy message broker as the event backbone
  (decision by default — never recorded, silent state per §7.4)
      │  superseded by
      ▼
ADR-0001  Adopt Kafka as the event backbone        Status: Superseded by ADR-0003 (in part)
      │    (Kafka adoption stands; retention clause withdrawn)
      │  retention clause superseded by
      ▼
ADR-0003  Retire the legacy message broker         Status: Accepted
      │  governs
      ▼
ADR-0002  Schema registry with Avro                Status: Accepted
      │  related to
      └──────────────► ADR-0001 (the backbone it governs)
```

Note what the log deliberately does *not* show: no record was deleted, none was rewritten after acceptance, and the original legacy-broker choice is still visible — as the pre-ADR state in ADR-0001's Context and as the superseded clause in ADR-0003's Status. That is the decision trail doing its job (§10.1): a future architect, auditor, or regulator can reconstruct the entire event-backbone story — what was chosen, why, what it cost, and what replaced it — from three files.

### 11.6 Lessons from the Mini-Series

- **The status transition is part of the record.** ADR-0001 shows Proposed → Accepted in its own Status section; the board's acceptance is dated and attributable. The record *is* the governance evidence (§9.2).
- **Draft revision happens before acceptance.** ADR-0001's original direction (the legacy broker's next major version) was revised during review, while the record was still Proposed — the one legitimate edit window. After acceptance, only status and links change (§4.3).
- **Supersession can be partial.** ADR-0003 supersedes ADR-0001 *in part*: the Kafka adoption survives, the legacy clause dies. The status line says exactly that ("Superseded by ADR-0003 (in part)") rather than forcing a false binary (§7.3).
- **The superseding record carries its own cost.** ADR-0003's Consequences record the irreversibility, the validation window, and the archive obligation — because the supersession itself is a decision with consequences (§7.4).
- **The links are the structure.** Every record names its neighbours; the graph in §11.5 reconstructs itself from the Status sections alone — which is why adr-tools' link machinery (§8.1) is the tooling that matters.
- **The mini-series is audit evidence.** Dated, immutable, linked, and reviewed — the three records are the §10 argument made concrete: a Cymbal Bank supervisor or internal audit asking "why is the legacy broker gone?" gets the answer from ADR-0003's Context, with the chain back to ADR-0001 and the schema lineage from ADR-0002.

---

## 12. The Claims Audit

The house rule of this repository: never present an unverified claim as fact. This audit lists every load-bearing factual claim in the guide, with its verification status. **✅ = verified this pass at a primary source** (URLs given); **⚠ = flagged** (single-source, not re-verified, or held from the author's knowledge base); **❌ = disputed or false as stated** (correction given in the text).

| # | Claim | Status | Evidence |
|---|---|---|---|
| 1 | Nygard's "Documenting Architecture Decisions" was published 15 November 2011 on thinkrelevance.com, archived at cognitect.com | ✅ | Article page extracted this pass (cognitect.com/blog/2011/11/15/documenting-architecture-decisions: "Michael Nygard — November 15, 2011"); original thinkrelevance URL cited in the adr-tools README, adr.github.io, and the article itself |
| 2 | The article is itself formatted as an ADR (Context / Decision / Status / Consequences) and is CC0 public-domain | ✅ | Extracted article text this pass |
| 3 | The article prescribes doc/arch/adr-NNN.md, sequential non-reused numbering, Markdown/Textile, "superseded" marking for reversed decisions, one-to-two-page length, "conversation with a future developer" | ✅ | Extracted article text this pass (all quotations verbatim) |
| 4 | The article's status vocabulary: proposed / accepted / deprecated / superseded "with a reference to its replacement" | ✅ | Extracted article text this pass |
| 5 | adr.github.io definitions: AD = "justified design choice… architecturally significant"; ASR definition; "collection of ADRs… constitute its decision log" | ✅ | adr.github.io extracted this pass |
| 6 | adr.github.io calls Nygard's post "the blog post from 2011… that popularized the concept" | ✅ | adr.github.io extracted this pass |
| 7 | adr-tools is "a command-line tool for working with a log of Architecture Decision Records"; default directory doc/adr; `adr init`, `adr new`, `adr new -s`, `adr help` documented in README | ✅ | github.com/npryce/adr-tools README extracted this pass |
| 8 | adr-tools full user command set in master: adr-config, adr-generate, adr-help, adr-init, adr-link, adr-list, adr-new, adr-upgrade-repository | ✅ | src/ tree listing extracted this pass |
| 9 | `adr new` numbering/slugging: `printf "%04d"`, slugified title → NNNN-title.md; template placeholders NUMBER/TITLE/DATE/STATUS; default STATUS "Accepted"; opens $VISUAL/$EDITOR | ✅ | Raw src/adr-new script extracted this pass |
| 10 | `adr new -s N` adds "Superceded by"/"Supercedes" links and removes the old record's Accepted status; multiple -s/-l supported; `-l "5:Amends:Amended by"` example | ✅ | Raw src/adr-new script extracted this pass |
| 11 | `adr link SOURCE LINK TARGET REVERSE-LINK`, e.g. `adr link 12 Amends 10 "Amended by"`; `adr list` = sorted file listing; `adr generate toc` / `adr generate graph` | ✅ | Raw src/adr-link, src/adr-list, src/adr-generate scripts extracted this pass |
| 12 | "adr-tools has an `adr status` command" (and an `adr edit` command) | ❌ | Not in the current master src/ tree (verified this pass); the internal `_adr_status` helper is not a CLI command; status is viewed via `adr list` — corrected in §8.1 |
| 13 | MADR = Markdown Architectural Decision Records, by Kopp, Armbruster, Zimmermann; ZEUS 2018 paper; docs at adr.github.io/madr/; templates adr-template[-minimal][-bare].md; "copy the template to nnnn-title.md" in docs/decisions; MIT OR CC0-1.0; npm package "madr" | ✅ | github.com/adr/madr README extracted this pass |
| 14 | MADR template structure: front matter (status/date/decision-makers/consulted/informed), Context and Problem Statement, Decision Drivers, Considered Options, Decision Outcome, Consequences, Confirmation, Pros and Cons, More Information; status vocabulary "{proposed \| rejected \| accepted \| deprecated \| … \| superseded by ADR-0123}"; Confirmation asks about fitness functions | ✅ | Raw template/adr-template.md extracted this pass (heading structure verbatim) |
| 15 | madr.dev is the MADR homepage and is reachable | ⚠ | The domain was observed **suspended** (account-suspended page) during this pass; MADR facts verified from the GitHub repo instead — flagged in §3.2 and §13 |
| 16 | Log4brains: docs-as-code ADR knowledge base by Thomas Vaillant, Apache-2.0, 2020; commands init/preview/adr new/build; MADR as default template; "an ADR is immutable. Only its status can change"; inspired by adr-tools and adr-viewer | ✅ | github.com/thomvaill/log4brains README extracted this pass |
| 17 | BCBS 239: January 2013 standard, 14 principles, 1 January 2016 G-SIB deadline | ✅ | Carried by the sibling guide's ledger (banking/risk_data_aggregation_guide.md), per the repo's cross-verification convention |
| 18 | MAS technology-risk expectations (TRM Guidelines, outsourcing expectations, board oversight) | ✅ | Carried by the sibling guide's ledger (banking/mas_regulations_guidelines_guide.md), per the repo's cross-verification convention |
| 19 | ERM framing (COSO/ISO, three lines of defence, appetite-to-ICAAP chain) | ✅ | Carried by the sibling guide's ledger (banking/enterprise_risk_management_guide.md), per the repo's cross-verification convention |
| 20 | Kafka operational facts used in the worked example (replication factor, acks=all, KRaft, Schema Registry, compatibility defaults) | ✅ | Carried by the sibling guide's ledger (banking/kafka_guide.md §3, §4, §6), per the repo's cross-verification convention |
| 21 | Worked-example dates (2026-06-02 … 2026-08-19) and the Cymbal Bank governance frame (ARB, decision classes, gate shape) | ⚠ | **Pedagogical constructions**, consistent with the repo's Cymbal Bank persona; not facts about any real bank — flagged in §11.1 and §13 |
| 22 | Repository conventions (docs/adr/, NNNN-title-with-dashes.md, adr-tools as standard CLI) | ⚠ | House-style choices of this repository's Cymbal Bank context, not external standards |
| 23 | Azure Well-Architected Framework features ADRs (adr.github.io referenced; dated 2024-11-10 per adr.github.io) | ✅ | adr.github.io media list extracted this pass |
| 24 | Keeling, "Love Unrequited…", IEEE Software Vol. 39 Issue 4 (2022); WICSA 2015 seven-template comparison; Zdun et al. "Sustainable Architectural Decisions" (InfoQ) | ✅ | adr.github.io background list extracted this pass |

---

## 13. What Could Not Be Verified

The honesty section — the items this pass could not pin down, and exactly why:

- **madr.dev content.** The domain returned an account-suspended page during extraction, so nothing was verified *from* madr.dev itself. The task brief named madr.dev as a primary source; the guide fell back to the authoritative GitHub mirror (github.com/adr/madr, including the raw template file) and says so (§3.2). If madr.dev comes back, the template and the version history there should be cross-checked against what is cited here.
- **`adr status` and `adr edit`.** The task brief listed these as adr-tools commands to document. The current master has neither (verified via the src/ listing — see audit #12). They may exist in older versions, in forks, or in other ADR CLIs; this guide documents the verified set and flags the discrepancy rather than guessing.
- **adr-viewer and the joelparkerhenderson collection.** Both are cited only as observed in Log4brains' acknowledgments; neither was deep-verified this pass (audit-adjacent ⚠, §8.4).
- **The exact wording of MADR's full template guidance text.** The heading structure and the section *intent* are verbatim from the raw template; the explanatory sentences inside the fenced example are condensed (the fence says so). A team adopting MADR should copy the template file from the repository, not from this guide.
- **MADR's current version number and changelog state.** Not verified this pass; the guide cites the template as of the observed develop branch.
- **The worked-example dates and the Cymbal Bank governance frame.** Explicitly pedagogical (§11.1, audit #21). Nothing in §9.5 or §11 describes a real bank's governance process; Cymbal Bank is this repository's recurring fictional persona.
- **Any real-world adoption statistics for ADRs.** None are claimed in this guide; the empirical claims that *are* made (the "six to ten developers" of Nygard's 2011 experience report) come verbatim from the primary source ✅.
- **The 2016-era dating of adr-tools' creation.** The README and repository do not state a creation date on the pages extracted; "2016-era" in §2.3 is held from the author's knowledge base and should be treated as ⚠ — the *commands* are all verified against the current master regardless of the project's age.

---

## 14. The Glossary

| Term | Definition |
|---|---|
| **ADR (Architecture Decision Record)** | A short, self-contained document capturing one significant architecture decision — Context, Decision, Consequences — stored in version control, numbered, and immutable except for its status (§1) |
| **AD (Architectural Decision)** | "A justified design choice that addresses a functional or non-functional requirement that is architecturally significant" (adr.github.io ✅, §1.1) |
| **ASR (Architecturally Significant Requirement)** | "A requirement that has a measurable effect on the architecture and quality of a software and/or hardware system" (adr.github.io ✅, §1.1) |
| **Decision log** | The collection of ADRs maintained for a project — the numbered, linked, auditable history of its decisions (§1.4) |
| **Decision trail** | The decision log considered as evidence: contemporaneous, immutable, linked records of what was decided, why, and what replaced it — the audit-facing property of the log (§10) |
| **Context** | The ADR section describing the forces at play — technological, political, social, project-local — in value-neutral language (Nygard ✅, §3.1) |
| **Consequences** | The ADR section listing the resulting context after the decision — positive, negative, and neutral (Nygard ✅, §3.1) |
| **Proposed / Accepted** | Lifecycle statuses: drafted-but-not-agreed / agreed-and-live (Nygard ✅, §4.1) |
| **Superseded / Deprecated / Amended** | Lifecycle statuses: replaced by a named later ADR / abandoned without a replacement / partially adjusted while the core stands (§4.1, §7.3) |
| **Supersession chain** | The linked sequence of records replacing one another in a decision area — walkable via "Supersedes"/"Superseded by" links (§7) |
| **MADR** | "Markdown Architectural Decision Records" — the structured template family (Kopp, Armbruster, Zimmermann) with Considered Options, Decision Outcome, and Confirmation sections (§3.2) |
| **Y-statement** | The one-sentence decision template: "In the context of …, facing …, we decided … to achieve …, accepting …" (Zdun et al., §3.3) |
| **adr-tools** | Nat Pryce's bash CLI for managing the numbered ADR log: init, new, link, list, generate, help (§8.1) |
| **Log4brains** | Thomas Vaillant's docs-as-code ADR knowledge base with a web UI and static-site publishing; MADR is its default template (§8.2) |
| **Decision review gate** | The workflow step where a proposed decision is reviewed and accepted or sent back — pull-request, architecture-board, or hybrid (§9.2) |
| **Architectural fitness function** | An automated or manual check that the running system still conforms to a decision; the MADR Confirmation section asks for it explicitly (§9.3) |
| **ARB (Architecture Review Board)** | The acceptance authority for significant decisions in the Cymbal Bank governance frame (§9.5) |
| **BCBS 239** | The Basel Committee's *Principles for effective risk data aggregation and risk reporting* (January 2013, 14 principles) — the regulatory anchor for data lineage and documentation (§10.3) |

---

*End of guide. The ADR is the smallest document that can hold a decision and its reasons; the log is the trail that holds them all. Written at the point of commitment, kept immutable, linked in supersession chains, and reviewed at a gate, the records of this repository's architecture stop being opinions and become evidence — for the next architect, the next auditor, and the next regulator who asks why. Every decision recorded is a question answered for someone who has not asked it yet, and that is the whole practice: the decision trail.*
