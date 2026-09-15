# The FINOS Common Domain Model (CDM): A Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Capital Markets / Banking Technology — Post-Trade Data Standards, Derivatives Lifecycle Modelling, Regulatory Reporting (DRR), Semantic Interoperability, Collateral, Tokenisation, Open-Source Governance (FINOS / Linux Foundation)
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Evidence cut-off:** every version, date, licence and adoption claim in this guide was checked at a primary source on **15 September 2026**. Where a claim is analysis rather than a sourced fact, it is labelled as such. Governance and version facts in this domain change several times a year — re-verify before quoting.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [Origin and Governance](#2-origin-and-governance)
3. [Model Architecture](#3-model-architecture)
4. [The DSL and Code Generation](#4-the-dsl-and-code-generation)
5. [Scope and Coverage](#5-scope-and-coverage)
6. [The Use Cases](#6-the-use-cases)
7. [The Interoperability Question](#7-the-interoperability-question)
8. [The Adoption Reality](#8-the-adoption-reality)
9. [The AI and LLM Angle](#9-the-ai-and-llm-angle)
10. [The Business Case and the Honest Limits](#10-the-business-case-and-the-honest-limits)
11. [The Comparison](#11-the-comparison)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. The Overview

The FINOS Common Domain Model (CDM) is a **semantic and process model of financial transactions**, expressed in a purpose-built domain-specific language, distributed as **executable code in multiple programming languages**, and governed as an open standard. In FINOS's own words it is "a standardised, machine-readable and machine-executable blueprint for how financial products are traded and managed across the transaction lifecycle. It is represented as a domain model and distributed in open source" (CDM repository README, github.com/finos/common-domain-model, accessed 15 September 2026).

### 1.1 What the CDM actually is

Three artefacts, bundled together, are what people mean when they say "the CDM":

| Layer | What it is | Where it lives |
|---|---|---|
| **The model** | Types, enums, choices, qualifications and functions describing products, events, legal agreements, processes and reference data | `.rosetta` source files under `rosetta-source/src/main/rosetta/`, organised as namespaces |
| **The language** | The **Rune DSL** — syntax, semantics and rules for expressing data *and* logic in a technology-agnostic way | `github.com/finos/rune-dsl` (Apache-2.0) |
| **The generated artefacts** | Executable code produced automatically from the model by code generators, published as versioned distributions | CDM release artefacts (Java complete; other targets partial) |

So the CDM is best classified as: **a domain model + a DSL + a code-generation pipeline + an open-standard governance process**. It is a *specification that ships as a library*.

### 1.2 What the CDM is not

This matters more than the definition, because most CDM disappointment in the industry comes from category error:

- **Not a product.** Nobody sells you "the CDM". You implement it, or a vendor implements it inside their product. FINOS explicitly states that it "encourages the adoption of CDM by software providers but does not endorse any CDM application component" (cdm.finos.org/docs/cdm-overview, v7.0.0 docs, accessed 15 September 2026).
- **Not a database and not a golden store.** The CDM is a representation standard. It does not store your trades, and it does not resolve your data-quality problems on its own.
- **Not a messaging standard.** There is no "CDM message" you send to a counterparty in the way you send an ISO 20022 message or an FpML document. Messaging formats are *targets the model maps to*, not the model itself.
- **Not a rule book or a legal opinion.** It encodes an industry interpretation of rules and documents; the interpretation is a separate, human-governed artefact (this is exactly what the DRR peer-review process exists to manage).
- **Not a vendor product in disguise** — but the concentration of tooling around one vendor's platform is a real critique, discussed in §10.

### 1.3 The problem it addresses

The CDM's founding premise is that the post-trade stack is **semantically fragmented**, and that the fragmentation carries no competitive advantage:

- **Per-firm proprietary models.** Each institution represents the same interest rate swap's lifecycle events differently in its own systems. ISDA's own framing: "There is no commercial advantage to organizations maintaining their own representations, as it results in firms having to continually reconcile their trades to make sure they have the same information – a big drain on resources" (isda.org/cdm, accessed 15 September 2026).
- **Reconciliation cost.** Every divergence in event representation becomes a reconciliation break, a manual investigation, and an operational-risk item. The CDM's stated purpose is to enable interoperability and straight-through processing "across firms, solutions and platforms, reducing the need for reconciliation caused by variations in how each firm records trade lifecycle events" (CDM README, accessed 15 September 2026).
- **Regulatory reporting burden.** Reporting regimes are rewritten every few years (CFTC, EMIR REFIT, UK EMIR, JFSA, MAS, ASIC, HKMA, CSA). Each firm re-interprets each rule set separately, producing divergent data for the same trade. The DRR initiative (§6) is the CDM's answer: publish the industry interpretation once as executable code.

### 1.5 Who runs it now

- **Host:** FINOS (Fintech Open Source Foundation), the financial-services umbrella of the Linux Foundation. The CDM has been a FINOS open-standard project "starting in February 2023" (GOVERNANCE.md and README, accessed 15 September 2026).
- **Sponsoring trade associations:** ISDA, ICMA and ISLA — they commission the roadmap and, per the README, "have commissioned REGnosys to provide the CDM Release Management Service" (CDM README, accessed 15 September 2026).
- **Governance model:** the Community Specification process, with Maintainers, Editors and Participants, and consensus-based decision-making (§2.4).
- **Repo health signal:** the CDM repository shows **6,915 commits**, **57 branches**, **1,408 tags**, 278 stars and 134 forks on the master tree viewed on 15 September 2026. The README carries a **FINOS "Graduated" lifecycle badge** while the docs site still displays an "Incubating" badge on the same day — a small but instructive example of documentation drift in this project (§14).

---

## 2. Origin and Governance

### 2.1 The ISDA origin (2017-2022)

The CDM began life as an **ISDA** project, not a FINOS one. ISDA's own CDM resource chronology places its earliest public artefact in November 2017, with REGnosys appointed in February 2018 to build the "digital" version, an openly published iteration in June 2018, and **ISDA CDM 2.0** released to the whole market in March 2019 (isda.org/cdm, resource listing, accessed 15 September 2026).

Crucially, the CDM was born *because of* a regulatory-reporting problem, not a post-trade processing problem. ISDA describes the model as "developed in response to regulatory changes, high costs associated with current manual processes and a demand for greater automation across the industry" (ISDA press release, 21 May 2019). The UK Digital Regulatory Reporting pilot run with the FCA and the Bank of England is the direct ancestor of today's DRR.

Scope expansion beyond derivatives came through the other two associations: **ISDA, ICMA and ISLA signed a Memorandum of Understanding on the CDM on 2 August 2021**, bringing repo, bonds and securities lending into the model's remit (isda.org, accessed 15 September 2026).

### 2.2 The transfer to FINOS — exact date and terms

This is the single most mis-stated fact about the CDM. The verified sequence is:

| Date | Event | Source |
|---|---|---|
| 10 May 2022 | ISDA, ICMA and ISLA publicly invite third parties to propose hosting the CDM repository | isda.org press release |
| **8 September 2022** | The three associations **announce the appointment of FINOS** to provide the repository, with "a view to migrate to FINOS by the start of 2023" | isda.org/2022/09/08/... |
| **February 2023** | CDM formally becomes "an open standard project hosted under FINOS … starting in February 2023" | CDM README / GOVERNANCE.md |
| 23 February 2023 | Repository licence file updated to the Community Specification License (PR #1962), with a NOTICE entry for versions before 4.0.0 | CDM repo commit history |
| **24 June 2024** | REGnosys contributes the **Rune DSL** (formerly the Rosetta DSL) to FINOS, completing the open-sourcing of the language layer | REGnosys press release, 24 June 2024 |

**The terms, read carefully.** The September 2022 announcement states that after migration "the model will then be distributed on an **Apache license** standard for open-source software". The *actual* licence file in the repository today is **not** Apache: it is the **Community Specification License 1.0** (§2.3). The 2022 sentence is therefore stale — treat any deck that quotes it as out of date. What the CSL does say is that *source code* in a Working Group repository takes the repository's source-code licence, and "if no source code license is included, the source code will be subject to the Apache License, version 2" (LICENSE.md §4, accessed 15 September 2026).

The governance arrangement that resulted is a three-way split:

- **Trade associations (ISDA, ICMA, ISLA)** set direction, fund the roadmap, and commission release management.
- **FINOS / Linux Foundation** hosts the repository, runs the IP framework (ICLA/CCLA via EasyCLA), and provides the open-standard process.
- **A vendor (REGnosys)** performs the operational release-management function under contract to the associations — an unusual and much-discussed arrangement for a supposedly neutral standard.

### 2.3 The licence, quoted from the file

| Component | Licence | Evidence (accessed 15 September 2026) |
|---|---|---|
| CDM specification | **Community Specification License 1.0** | `LICENSE.md` in finos/common-domain-model. Full text begins: "# Community Specification License 1.0 … This License sets forth the terms under which 1) Contributor will participate in and contribute to the development of specifications … 2) how the materials developed under this License may be used. It is not intended for source code." |
| CDM copyright notice | "Copyright 2021 FINOS and CDM Participants" | CDM README, License section |
| CDM versions **before 4.0.0** | **ISDA CDM™ License** | `NOTICE.md`: "Previous versions are licensed under the ISDA CDM™ License and are available here [REGnosys/rosetta-cdm, tag 3.0.0]" |
| Rune DSL (language + default generators) | **Apache License 2.0**, "Copyright 2019 REGnosys", SPDX `Apache-2.0` | finos/rune-dsl LICENSE / README |
| DRR (the reporting logic) | **Proprietary**, free for non-commercial use; free to use for ISDA and non-ISDA members; "Currently DRR is not open source" | drr-docs.isda.org/docs (Governance); TradeHeader DRR FAQ, 10 February 2026 |



### 2.4 Governance as it operates today

- **Framework:** "Community Specification Governance Policy 1.0". Three roles per Working Group — **Maintainers** (drive consensus, approve and merge changes), **Editors** (codify ideas into the specification; presently individuals from TradeHeader, Fragmos Chain and FT Advisory hold Editor status), **Participants** (anyone contributing under a signed CSL CLA) (GOVERNANCE.md, accessed 15 September 2026).
- **Decision-making:** consensus-based; "while the agreement of all Participants is preferred, it is not required for consensus". If consensus fails, Maintainers may call a simple majority vote of Participants. Decisions are appealable via issue or pull request.
- **Steering Working Group:** defines major releases and shapes their content, sets the strategic roadmap, vets proposed changes, and approves Working Group formation; the first Steering Group of each year re-ratifies the governance measures themselves.
- **Maintainer model:** Maintainers are either **Organisational Representatives (OrgReps)** or **private individuals**; each organisation gets one vote; new maintainers need proven data-modelling experience and a contribution track record, and are approved by consensus of Participants plus existing Maintainers.
- **Maintainer roster (14 names, as published 15 September 2026):** ICMA (Gabriel Callsen, Tom Healey), ISDA (Tabish Ahmed, Eleonora Acuna, Lyteck Lynhiavu), ISLA (Adrian Dale, Chris Rayner), REGnosys (Leo Labeis, Hugo Hills, Jayasri Radhakrishnan), TradeHeader (Manuel Martos, Marc Gratacos), JPMorgan Chase & Co. (Nicholas Moger), plus one private individual (Minesh Patel). **Observation for readers:** of 14 maintainers, **six represent the vendor and consultancy sides** and exactly **one** represents a bank. That composition is a legitimate input to any governance critique.
- **Active Working Groups:** Steering; Contribution Review; Collateral; Technology Architecture; Physical Risk; **Tokenized Assets**; Derivatives (formerly Derivatives Products and Business Events); plus ISLA working groups on securities lending, GIL/ trading and GMSLA document digitisation, and an ICMA SteerCo for repo and bonds (CDM README, accessed 15 September 2026).

### 2.5 Release and versioning practice

Versioning is **semantic**, per the repository's own release instructions: "Increment that semantic version and enter in the Select Tag box" (CDM README, accessed 15 September 2026). The current release-schedule policy (SCHEDULE26.md, updated 9 September 2026) defines four states and a hard concurrency limit: **"At any point we want a maximum of one centrally supported development version, one production version, and one maintenance version."**

| State | Meaning | Intended lifetime |
|---|---|---|
| Development | New designs from `main`, still evolving; tests must pass | Until promoted |
| **Production** | "The latest and greatest" stable version; backwards-compatible enhancements | **~1 year** (explicitly to "alleviate upgrade costs to consumers") |
| Maintenance | Critical bug fixes and critical regulatory changes only | Until the next Production release |
| Unsupported / EOL | No fixes | Forever |

The 2026/27 timeline in the same file shows **CDM 7 in Production from July 2026, CDM 6 in Maintenance, CDM 8 in Development from July 2026, and CDM 9 entering Development around June 2027.**

### 2.6 Dated timeline

| Date | Event | Source (all accessed 15 Sep 2026) |
|---|---|---|
| 30 Nov 2017 | ISDA publishes "What is the ISDA CDM?" — earliest CDM item on ISDA's own resource list | isda.org/cdm |
| 15 Feb 2018 | ISDA appoints REGnosys to develop the digital CDM | isda.org |
| 20 Mar 2019 | **ISDA CDM 2.0** published; access opened to the entire market | isda.org |
| 21 May 2019 | ISDA CDM 2.0 deployed to support **FCA / Bank of England DRR pilot phase two** | isda.org |
| 2 Aug 2021 | ISDA, ICMA and ISLA sign an MoU on the CDM | isda.org |
| 22 Nov 2022 | **ISDA DRR 1.0** launched, open to the entire market | isda.org |
| 5 Dec 2022 | CFTC reporting rule changes take effect — first DRR target regime | isda.org DRR InfoHub |
| 9 Nov 2022 | CDM tag **3.0.0** (last ISDA-licensed line, pre-FINOS go-live) | GitHub API, tag date |
| 8 Sep 2022 | FINOS appointed to host the CDM repository | isda.org |
| **Feb 2023** | CDM goes live as a FINOS-hosted open standard | CDM README / GOVERNANCE.md |
| 23 Jun 2023 | CDM **4.0.0** | GitHub API |
| 29 Apr 2024 | DRR extended to **EU EMIR** reporting rules | ISDA DRR InfoHub |
| 10 Jun 2024 | VERMEG integrates the CDM into its COLLINE collateral management system | isda.org |
| **24 Jun 2024** | REGnosys contributes the **Rune DSL** to FINOS (renamed from Rosetta DSL) | REGnosys press release |
| 21 Oct 2024 | DRR extended to **Australia (ASIC)** and **Singapore (MAS)** | ISDA DRR InfoHub |
| 15 Jan 2025 | **JSCC** announces it is the first CCP and first Japanese entity to adopt DRR/CDM in a production environment; production-parallel reporting to JFSA and CFTC from June 2025 | FINOS press release / JSCC |
| 24 Jan 2025 | CDM **6.0.0** ("second major update since the model found its home at FINOS", >100 enhancements) | GitHub API; FINOS blog |
| 30 Oct 2025 | ISDA launches an RFQ for a DRR traceability tool | isda.org |
| 17 Mar 2026 | **LSEG TradeAgent** integrates the ISDA DRR solution | isda.org |
| 30 Mar 2026 | ISDA selects **Gentek AI** to build the DRR traceability tool (AI-extracted from working-group minutes) | isda.org |
| 17 Jul 2026 | **CDM 7.0.0** — current Production line | GitHub API |
| 9 Sep 2026 | CDM **7.3.0** (production line) and **8.0.0-dev.5** (development line) | GitHub API |
| 14 Sep 2026 | CDM **6.28.1** — newest tag on the Maintenance line | GitHub API |

### 2.7 How an institution actually participates

1. **Join FINOS** (membership) — the CDM README notes the model "is maintained by FINOS members and the wider open source in finance community"; membership is how most banks get sustained influence.
2. **Sign the Community Specification CLA** (corporate CCLA, or ICLA for individuals) so contributions pass the EasyCLA check. "Commits from individuals not covered under an ICLA or CCLA will be flagged and blocked by the Linux Foundation EasyCLA tool."
3. **Enrol as a Participant** and attend Working Groups — Steering, Contribution Review, Technology Architecture, Collateral, Tokenized Assets, Derivatives, or the association-hosted ones.
4. **Contribute via pull request** — either through GitHub directly (fork, branch, PR) or through REGnosys's Rosetta Design web application, which "can be used to contribute to the CDM without setting up any development environment".

---

## 3. Model Architecture

Everything in this section is checked against the CDM v7.0.0 documentation site, the `master` branch source tree and release notes, all accessed 15 September 2026.

### 3.1 The five primary dimensions

The CDM documentation defines the model's primary dimensions as:

| Dimension | Documentation definition | Practical meaning |
|---|---|---|
| **Product** | "describes a thing (e.g. contract or instrument) that is used to transfer financial risk between two parties" | The economic terms: instrument type, payoff, legs, schedules, quantities |
| **Event** | "provides data structures to represent the lifecycle events of financial transactions" | What happens to a trade: execution, allocation, clearing, reset, payment, exercise, amendment, termination |
| **Legal Agreement** | The contractual/documentation layer (master agreements, ISDA/ICMA/ISLA documentation, additional terms, and — as of 2026 — umbrella agreements and contract amendments) | Which legal terms attach to the trade and how they evolve |
| **Process** | "lays the foundation for the standardisation, automation and inter-operability of industry processes" | The sequencing and state machine around events, including new instruction-composition work |
| **Reference Data** | "describes underlying business entities such as parties, legal entities, floating rate indexes, and so on" | Party, LEI, identifiers, taxonomies, codes |

### 3.2 The layers as they exist in the repository

The source tree confirms the modularisation principle ("**Modularisation** into logical layers", one of the five published design principles). Namespaces follow a `<layer>-<domain>-<aspect>` convention, and — importantly for anyone reading older material — **the file extension is still `.rosetta` and the source directory is still `rosetta-source/src/main/rosetta/`** even though the *language* has been renamed (§4.1).

File families visible on `master` (verified 15 September 2026) include:

```text
base-datetime-{type,func,enum}.rosetta        date/time arithmetic, day counts, business-day conventions
base-math-{type,func,enum}.rosetta            quantities, arithmetic, unit handling
base-staticdata-asset-rates-*.rosetta         rate asset definitions
base-staticdata-codelist-*.rosetta            external code lists (e.g. ISO/FpML taxonomies)
base-staticdata-identifier-*.rosetta          identifiers: UTIs, LEIs, product IDs
legaldocumentation-master-{isda,icma,isla}-*.rosetta   master-agreement modelling per association
legaldocumentation-transaction-additionalterms-*.rosetta
```

The 8.0.0 development line adds a new **instruction-composition** capability, evidenced in the 8.0.0-dev.5 release notes: namespaces `cdm.event.instructioncomposition` and `cdm.event.instructioncomposition.reset`, with types such as `CompositionStepInstructions`, `ResetInstructionState` and `DetermineUnadjustedCalculationPeriodInstruction` implementing an **8-step Interest Rate Reset processing workflow** with explicit state mutation and a "Golden Record" concept. This is the CDM moving from *representing* an event to *executing a deterministic lifecycle sequence* — a meaningful architectural shift from where the model was in the 4.x/5.x era.

### 3.3 How an instrument and its lifecycle are represented

The core modelling idiom is:

1. **Product** — an economic-terms object built compositionally. Derivatives products are assembled from legs; legs from payoffs; payoffs from observable/pricing components. This is the "**Composability** where objects are composed and **qualified** from the bottom up" principle in action: rather than one giant `Swap` type per product, the model composes a small number of primitives and *qualifies* them (attach attributes) to arrive at specific instruments.
2. **Trade / execution** — ties a product to the parties, dates and identifiers of an actual transaction.
3. **Business event** — the primitive of the lifecycle. Instead of mutating a trade object, the CDM expresses *the event that happened* (a reset, a payment, a termination). Lifecycle evolution is reconstructed by applying events, which is precisely why the model suits reporting, reconciliation and stateful processing.
4. **Execution instruction / state (new in 8.x)** — the instruction-composition framework adds the ordered, stateful layer that deterministically produces the post-event state (§3.2).
5. **Reference data** ties the above to real-world identifiers so the model is an actual record and not an abstract schema.

### 3.4 Extensibility and qualification

- **Qualification** is the model's native extension mechanism: you compose existing types and qualify them rather than inventing parallel hierarchies. The published design principles are explicit that composability is "bottom up".
- **Namespacing** is the governance extension boundary: the documented working practice is to keep customisations in a **separate namespace/file from the public model** "to ensure clarity and organisation as well as facilitating version upgrades" (TradeHeader DRR FAQ, 10 February 2026).

### 3.5 Serialisation

- The CDM is specified as a model, not a wire format; the serialisation targets that matter in practice are **JSON** (native representation of the model) and **XML** through the mapping layer to FpML/ISO 20022-derived formats.
- The **7.x line enables a Rune-native serialiser** — commit "[7-dev] Enable Rune serialiser" (PR #4803) references a `RuneJsonObjectMapper` and Rune-format samples, i.e. serialisation is being aligned to the language rather than to legacy JSON conventions.
- The project also runs a **CDM schema publication** pipeline (PR #4435, "moved hard-coded versions to separate, centralized file", per-version landing pages) so that machine-readable schemas for each released version are downloadable from the docs site.

### 3.6 Where structure changed across versions

- **Pre-4.0.0** was the ISDA-licensed line; 4.0.0 (June 2023) is the baseline for FINOS-era licensing and documentation.
- **7.0.0 (July 2026)** moved to a Java 21/current-DSL baseline and enabled the Rune serialiser; the CDM release notes for the 7.x and 8.x lines reference DSL bundle versions in the 11.x/12.x range, confirming the language and the model now version independently.
- **8.x (development)** introduces instruction composition and stateful reset processing (§3.2).
- **Legal layer** was still being extended as recently as September 2026: ISDA published "Expanding Legal Agreement Coverage in the CDM: Umbrella Agreements and Contract Amendments" on 4 September 2026, describing "two of the most significant, and previously undeveloped, areas of its legal agreement model".

---

## 4. The DSL and Code Generation

### 4.1 The naming history you must not get wrong

This is the highest-risk stale-content area in the whole topic. The machinery has been renamed twice.

| Era | What it was called | Status today |
|---|---|---|
| ~2018 - mid-2024 | **Rosetta DSL** — the language; **Rosetta** — REGnosys's platform/SDK | Renamed |
| **24 June 2024** | REGnosys contributes the language to FINOS: "The language – formerly known as the Rosetta DSL (Domain-Specific Language) and **renamed the Rune DSL** – is now available under the FINOS open source governance" | Current language name |
| Today (2026) | **Rune DSL** = the open-source language and default code generators. **Rosetta** = REGnosys's *proprietary platform* used as an execution engine and dev-kit | Both names coexist, for different things |

Practical consequences of the rename that trip people up:

- The repository is `github.com/finos/rune-dsl`; **`github.com/finos/rosetta` returns a 404** (verified 15 September 2026). Older links to a FINOS "rosetta" repo are dead.
- Model source files are **still `.rosetta`** and still live under **`rosetta-source/`**. The rename did not reach file extensions or directory names.
- The CDM docs still say the model is "contained into a set of `.rosetta` files organised as namespaces" and still link to `docs.rosetta-technology.io` for language documentation — while the FINOS language documentation lives at **rune.finos.org**.
- The DRR glossary defines **"Rosetta: REGnosys's proprietary platform used as an execution engine for DRR"** and separately **"Rune DSL: the domain-specific language built on Java that DRR uses for its logic"**. If you ever see these two used interchangeably in a vendor deck, the deck predates June 2024.
- The Rune DSL repository carries a **FINOS "Incubating"** lifecycle badge (rune-dsl README, accessed 15 September 2026) — unlike the CDM itself, which carries a **"Graduated"** badge.

### 4.2 What the DSL does

Per the FINOS Rune documentation (rune.finos.org/docs/get-started/overview, accessed 15 September 2026), Rune is "a coding language that expresses **both data and logic** – so you model not just **what** information exists, but **how** it behaves and how rules apply to it". It is explicitly designed to be human-readable "so that domain experts without programming experience can write fully functional regulatory logic directly – a bit like in Excel". Design intent, from the language's own README: "The Rune DSL allows to represent data and business logic in a system- and technology-agnostic way into a cohesive domain model."

Three capability claims are worth separating:

- **Representation:** types, enums, choices, cardinalities, qualifications, references.
- **Logic:** functions, conditions, validations, calculations — business logic lives *in the model*, not hidden in an implementation.
- **Execution:** the model generates runnable code, so logic that was "buried in code" becomes a reviewable artefact.

### 4.3 The code-generation pipeline

Repository evidence for the toolchain (finos/rune-dsl module list, accessed 15 September 2026): `rune-lang`, `rune-ide`, `rune-runtime`, `rune-tools`, `rune-maven-plugin`, `rune-generator-api`, `rune-testing`, `rune-profiling`, `rune-xcore-plugin-dependencies`, plus `rune-integration-tests` and a `website`. The presence of a **Maven plugin** and a **generator API** is what makes Rune usable as a build step rather than a curiosity.

### 4.4 Which targets exist today — and the honest caveat

The CDM documentation is refreshingly blunt here:

- "**Only the Java executable code distribution is complete**: i.e. it represents the entire CDM as defined in Rosetta (plus any associated default implementation). **Other distributions may only capture parts of the model**: for instance, the TypeScript distribution includes the complete data model, but not the functions' executable code." (cdm.finos.org/docs/cdm-overview, v7.0.0, accessed 15 September 2026)
- The language "includes one default code generator into Java". Broader targets come from the community code-generator project, historically hosted at `REGnosys/rosetta-code-generators`.
- On the model side, the CDM repository carries a `python/` directory, and a commit on 30 June 2026 is titled "**[master] Re-enables CDM's Python code generation using the new python generator/runtime (#4898)**" — evidence that Python generation was re-enabled on the master line after a gap. Treat non-Java targets as improving but still secondary.
- The **DRR distribution is Java only**: "DRR is currently distributed in Java. The model is developed using a functional language called Rune DSL and an open source code generator creates the Java distribution automatically" (TradeHeader DRR FAQ, 10 February 2026).

**Practical translation:** if your bank is not a JVM shop, you will be writing a mapping layer or relying on a vendor's non-Java distribution. That is a real adoption constraint, not a footnote.

### 4.5 Developer experience

| Activity | Tooling |
|---|---|
| Contributing without a local environment | **Rosetta Design** web app — create a workspace, make the change, and "Contribute the change in Rosetta Design which will create a PR in the FINOS GitHub" |
| Building/integrating | Maven (the Java distribution "is designed to be built and used with Maven"; repository settings ship in the CDM jar's `settings.xml`), plus the Rune Maven plugin |
| Validating | Model validation, cardinality/only-exists/type-format validators (generated by dedicated generator modules), regression test cases as part of change control |

**Baseline skills**, drawing on the maintainer/editor profiles: Xtext/Eclipse modelling literacy, functional modelling discipline, Java 21, Maven, and — critically — a derivatives business analyst who can read a rule and a model. Java 21 became the baseline when the DSL was upgraded to 9.54.0 (PR #3734, 30 May 2025).

### 4.6 Versioning the language


---

## 5. Scope and Coverage

### 5.1 The published scope statement

The authoritative sentence, from the CDM documentation's Scope section: **"The CDM covers OTC derivatives, cash securities, securities financing, commodities, and can expand to include other Capital Markets products and Asset Classes."** (cdm.finos.org/docs/cdm-overview, v7.0.0, accessed 15 September 2026).

Note how much wider that is than the marketing shorthand "derivatives model", and how much narrower than "everything in capital markets". Both errors are common.

### 5.2 Coverage in practice

| Area | Status | Evidence |
|---|---|---|
| OTC derivatives (rates, credit, FX, equity, commodity) | Core origin; deepest coverage | Model namespaces, DRR rule sets, ISDA documentation |
| Repo and bonds | In scope via ICMA; dedicated ICMA CDM SteerCo | CDM README working-group list; ISDA/ICMA/ISLA MoU 2021 |
| Securities lending | In scope via ISLA; dedicated ISLA CDM working groups (incl. GMSLA document digitisation) | CDM README; ISLA "CDM: The Road to Adoption" (Feb 2025) |
| Commodities | Stated in scope; commodity asset namespaces exist | CDM docs Scope; `base-staticdata-asset-commodity-*` |
| Collateral and margin | Active workstream: eligible collateral schedules, margin call workflows, collateral validation, digital documentation across OTC, repo, securities lending and cleared markets | ISDA collateral initiatives; CDM Collateral Use Case Overview deck (2026) |
| Legal documentation | Master agreements per association (ISDA/ICMA/ISLA), transaction additional terms, and — as of Sept 2026 — umbrella agreements and contract amendments | Source tree; ISDA paper 4 Sep 2026 |
| Tokenised assets / digital assets | Active extension work: token standards, blockchain protocol attributes on Asset and Settlement structures; atomic settlement and intra-day count fractions under consideration | CDM Tokenized Assets WG page; CDM Tokenization Use Case Overview (2026) |
| Stateful event processing | New and developing: instruction composition (8.x dev line) | 8.0.0-dev release notes |

### 5.3 Lifecycle processes modelled

The event model covers the trade lifecycle as *events*: execution and allocation, clearing and clearing submission, resets and rate fixings, payments and settlements, exercises, novations, amendments and terminations, accounting/valuation-related events, and legal-agreement-affecting events. ISDA's framing of the original design goal was to enable "firms meeting both position-based and transaction-based reporting requirements from the same trade data, and harmonize reporting triggers so firms report the same information at the same time" (ISDA, 21 May 2019).

### 5.4 What is explicitly out of scope

- **Messaging**: the CDM is not a wire format; it maps to FpML and ISO 20022 rather than replacing them (§7).
- **Reporting eligibility determination**: "currently reporting eligibility is not covered by DRR" — deciding *whether* a transaction must be reported to a jurisdiction is a firm-side determination (TradeHeader DRR FAQ, 10 February 2026). This is a very commonly underestimated gap.
- **Storage, orchestration and operational workflow** beyond the model's process layer: the CDM does not schedule your collateral calls or store your trade repository submissions.
- **Banking generally**: retail, payments processing, core banking ledgers and financial-crime domains are outside the model. For those, the relevant reference points in this library are BIAN and ISO 20022, not the CDM (§11).
- **Anything undocumented**: asset classes not listed in the Scope sentence should be treated as "not currently covered" until a primary source states otherwise.
---

## 6. The Use Cases

### 6.1 Regulatory reporting via the DRR

**The concept.** Instead of each firm reading a rulebook and writing its own reporting logic, the industry produces **one "golden source" interpretation** of each rule set, reviewed by a peer-review group, and publishes it as **machine-executable code that runs against CDM data**. ISDA's framing: "ISDA's Digital Regulatory Reporting (DRR) solution uses the open-source Common Domain Model (CDM) to transform an industry-agreed interpretation of new or amended regulatory reporting rules into unambiguous, machine-executable code" (isda.org DRR InfoHub, accessed 15 September 2026).

**The mechanics, end to end:**

```text
regulatory text (CFTC / EMIR / JFSA / MAS / ASIC / HKMA / CSA ... )
        |
        v
industry interpretation agreed by an industry committee, coded in Rune DSL
        |            (peer-review group approves pull requests)
        v
DRR model + generated Java libraries  <-- built on the CDM product/event model
        |
        v
firm's own trade data --> mapped to CDM --> DRR validation + report generation
        |
        v
ISO 20022 message (most jurisdictions) or DTCC Harmonized XML (Canada)
        |
        v
trade repository / regulator
```

Two deployment modes matter operationally: the DRR can be used **as the primary reporting engine**, or as a **control function** validating the firm's existing reporting logic. ISDA also allows vendors to embed it in their reporting products.

**Structure and governance:** an **ISDA DRR Steering Committee** made up of participating financial institutions defines scope and priorities; the **ISDA Peer Review working group** approves model pull requests (TradeHeader DRR FAQ, 10 February 2026).

**Jurisdictional status — verified individually, because the programme's progress is very uneven:**

| Jurisdiction | Regime(s) | DRR coverage live since | Status assessment | Source |
|---|---|---|---|---|
| **US (CFTC)** | CFTC rewritten reporting rules | 5 Dec 2022 (first DRR target) | **LIVE** — coverage at launch; named firms implemented DRR as primary reporting (BNP Paribas; JPMorgan Chase publicly describes implementing DRR "as a primary reporting mechanism") | ISDA DRR InfoHub; ISDA 2 Nov 2022; ISDA webinar page |
| **US (SEC)** | SEC security-based swap reporting | Listed within ISDA's 14-rule-set commitment | **COVERED** — no named production adopter verified | isda.org/cdm; DRR InfoHub |
| **EU** | **EMIR** | 29 Apr 2024 (EMIR REFIT date) | **LIVE** | ISDA DRR InfoHub |
| **EU** | MiFIR, SFTR | — | **IN DEVELOPMENT** ("currently in the process of being further extended") | ISDA DRR InfoHub |
| **UK** | **UK EMIR** | 30 Sep 2024 | **LIVE**; the UK is also the *origin* jurisdiction of DRR (FCA/BoE pilot phases 1 and 2, 2018-2019) | ISDA DRR InfoHub; ISDA 21 May 2019 |
| **UK** | MiFIR, SFTR | — | **IN DEVELOPMENT** | ISDA DRR InfoHub |
| **Japan** | JFSA rules | 1 Apr 2024 | **LIVE**, and the strongest production proof point: **JSCC** announced on 15 Jan 2025 that it is the **first CCP and first Japanese entity to adopt DRR and CDM in a production environment**, with production-parallel reporting to the JFSA and CFTC from June 2025 | ISDA DRR InfoHub; FINOS/JSCC press release |
| **Singapore** | MAS reporting rules | 21 Oct 2024 | **LIVE coverage**; MAS and ASIC jointly held a DRR kick-off webinar in January 2024. No Singapore-based production adopter has been verified for this guide | ISDA DRR InfoHub; ISDA webinar listing |
| **Australia** | ASIC rules | 21 Oct 2024 | **LIVE coverage** | ISDA DRR InfoHub |
| **Canada** | CSA rules | 25 Jul 2025 | **LIVE coverage**, plus active regulator engagement: the **Ontario Securities Commission** and ISDA are jointly reviewing DRR code against the CSA Technical Manual and coding upcoming revisions, targeting UAT in mid-April 2027 (ISDA explicitly labels these timings estimates) | ISDA DRR InfoHub |
| **Hong Kong** | HKMA rules | 29 Sep 2025 | **LIVE coverage** | ISDA DRR InfoHub; 15 Oct 2025 press release |
| **Switzerland** | FINMA | — | **ENGAGED / IN DEVELOPMENT** — no coverage date announced | ISDA DRR InfoHub; TradeHeader FAQ |
| **South Africa** | New derivatives reporting regime | — | **PRE-LAUNCH PILOT** — **Strate**, the sole licensed trade repository, "will publish the regime's reporting rules as machine-executable code using ISDA DRR. South African reporting has yet to commence, so this will be the first time rules are published as machine-executable code ahead of the initial implementation." | ISDA DRR InfoHub |

**How to state the DRR status honestly:**

- ISDA is "committed to supporting **14 core regulatory reporting regimes across nine jurisdictions**: Australia, Canada, the EU …, Hong Kong, Japan, Singapore, Switzerland, the UK … and the US" (ISDA DRR InfoHub, accessed 15 September 2026).
- As of 30 March 2026 ISDA described the DRR as having "been applied to **eight** sets of reporting rules globally" — eight covered, fourteen committed. That gap is the programme's own measured progress bar.
- Two exceptions must **not** be generalised: **eligibility determination is out of scope**, and the **DRR code is proprietary, not open source** (free to use, including for non-ISDA members).
- Regulator *use* of machine-readable rules remains the exception, not the rule: the clearest examples are the **Ontario Securities Commission** reviewing/coding rules with ISDA and **Strate** publishing rules as code. Most regulators are engaged, not operational, in rules-as-code.

### 6.2 Post-trade processing

The value claim is the strongest and the least contested at the *concept* level: a shared representation of lifecycle events removes the class of reconciliation breaks caused by firms recording the same event differently, enabling straight-through processing "across firms, solutions and platforms" (CDM README, accessed 15 September 2026).

What is **evidenced**:

- The model represents execution, allocation, clearing, reset, payment, exercise and termination events, with a new stateful instruction-composition layer arriving in the 8.x line (§3.2).
- DTCC's Global Trade Repository was connected to REGnosys's Rosetta platform (April 2024) to allow firms to test DRR-generated trade reporting submissions — a post-trade-adjacent production infrastructure link.

What is **analysis, not evidenced**:

- Published, quantified *affirmation/confirmation* benefits attributable to the CDM. No primary source was found that measures a specific firm's affirmation or confirmation cycle-time improvement from CDM adoption. Treat vendor claims here as vendor claims.

### 6.3 Collateral and margin

Collateral is, after reporting, the **most institutionally active** CDM workstream:

- Two dedicated working groups (CDM Collateral WG; ISLA collateral-adjacent groups) and an ICMA SteerCo.
- ISDA frames the objective as "increase automation in collateral management processes to improve efficiency, reduce the time it takes to get a client relationship up and running and cut operational and liquidity risks", with work alongside **ICMA and ISLA** to "develop collateral representations across derivatives, repo and securities lending, which will reduce onboarding time and improve interoperability and collateral optimization processes" (isda.org/cdm, accessed 15 September 2026).
- ISDA publishes a **CDM Collateral Start-up Guide** (modules A-G) explicitly aimed at *business users*, not developers — a signal that this workstream targets real operational deployments.
- The 2026 CDM collateral deck describes support for "standardised eligible collateral schedules, margin call workflows, collateral validation, and digital documentation across OTC, repo, securities lending, and cleared markets" (cdm.finos.org/docs/cdm-overview, accessed 15 September 2026).
- **Named vendor production evidence:** VERMEG announced on 10 June 2024 that it had **integrated the CDM into its COLLINE collateral management system** — a shipped product integration, not a PoC.

### 6.4 Tokenisation and digital assets

**Verdict: the CDM is a plausible but still-immature semantic layer for tokenised workflows.** Verified position:

- There is a dedicated **CDM Tokenized Assets Working Group**. Its own statement of intent: "This initiative will begin by introducing **new attributes to the current Asset and Settlement structures**, allowing for specification of token standards, blockchain protocol and other token details while maintaining the integrity of the existing CDM structure. Further work — including exploration of **atomic settlement and intra-day count fractions** — will also be considered." (cdm.finos.org/docs/CDM-Tokenized-Assets-WG, accessed 15 September 2026). That is *extension work*, not a finished tokenisation model.
- The 2026 "CDM Tokenization Use Case Overview" deck describes "both asset-level and settlement-level tokenisation models, helping firms represent cryptoassets, security tokens, and tokenized securities consistently across legal and operational frameworks" — a documentation artefact, indicating active design rather than broad production.
- The ISDA "CDM Smart Contracts Overview" (2026) frames machine-executable post-trade workflows (e.g. interest rate resets) as supporting "future tokenized workflows" — future tense.

**Do not assert** that the CDM "is the semantic layer for tokenisation". The defensible statement is: the CDM is the only industry-governed candidate for that role in derivatives and SFTs, and the model layer is being extended for it now.

---

## 7. The Interoperability Question

The honest framing: the CDM does not eliminate the mapping problem, it **moves it**. The design principles say so — "**Mapping** to existing industry messaging formats" is one of the five. The question for a bank is *where* the mapping lives and *how many times* you pay for it.

### 7.1 CDM and FpML

| Aspect | Position |
|---|---|
| Relationship | Complementary, not competing. FpML is a *messaging/representation format* for derivatives; the CDM is a *model* of products, events and processes. ISDA owns both historically, which is why convergence is a policy choice rather than a market accident |
| Translation state | An **open-source standard FpML→CDM translation — "Translate 2.0" — was published in Q4 2025 and is included in the CDM distribution** (TradeHeader DRR FAQ, 10 Feb 2026). The CDM 6.28.0 release notes also reference a `rune-fpml` mapping bundle (v2.7.0/2.8.0), confirming active maintenance |
| Practical mapping options | Four are documented: (1) use and extend open-source Translate 2.0; (2) use REGnosys's commercial translation service; (3) use a commercial ETL tool; (4) build your own |
| The catch | Translation is *structural*, not *semantic*. A syntactically valid FpML→CDM translation can still be wrong where your FpML usage is idiosyncratic or your data is incomplete. Translation frameworks move data; they do not fix missing fields |

### 7.2 CDM and ISO 20022

- ISO 20022 is a **messaging standard** for the payloads regulators and infrastructures exchange. The DRR uses it as an **output** target: "Reports generated in DTCC's Harmonized XML for Canada or the **ISO 20022 format** for other reporting jurisdictions, which can be submitted directly to trade repositories" (ISDA DRR InfoHub, accessed 15 September 2026).
- So the realistic architecture is **CDM in the middle, ISO 20022 at the edge**: model and compute on CDM data, emit ISO 20022 to report. The convergence work that matters is the mapping CDM→ISO 20022 message fields, including identifier population (UTI, UPI, LEI) — a live area of CDM change, e.g. release 6.28.0 added party-identifier scheme mapping so FpML party IDs are populated with the right scheme.
- For a bank already invested in ISO 20022 for payments, the CDM does **not** replace that investment, and the CDM should not be sold internally as "our ISO 20022 strategy".

### 7.3 CDM and FIX

- FIX governs pre-trade and execution messaging while the CDM governs post-trade lifecycle and product semantics, so the two largely do not collide. Where they touch (execution-time identifiers, allocation, and the handoff from execution to post-trade lifecycle), the mapping is again the firm's problem. The CDM's governance explicitly expects maintainers to have exposure to "other data standards (such as ISO) and messaging protocols (such as FIX, FpML or Swift)" — a recognition that the CDM lives in a protocol neighbourhood. For the FIX side, see `banking/fix_protocol_guide.md`.

### 7.4 CDM, internal firm models and market-infrastructure formats

- **Internal models are the real competitor.** No firm will rewrite its product master. The pragmatic pattern is a **translation layer at the boundary**: internal model → CDM for reporting and interoperability, CDM → internal for inbound messages. The cost is not the translation itself but **sustaining** it across regulatory change, model releases and internal platform change.
- **Migration burden is cumulative, not one-off.** With CDM Production releases intended to last about a year and a new major version following every 12-18 months (SCHEDULE26.md), a firm adopting the CDM should budget for an **annual regression-and-upgrade cycle** as a standing cost.

### 7.5 The mapping burden, stated plainly

- Three mappings exist for most firms and only the middle one is standardised: your internal model to the CDM (bespoke, and yours to own), the CDM to FpML and ISO 20022 (partly open source), and internal enrichment at the reporting edge. The CDM standardises the representation and the reporting logic; it standardises neither your source data nor your destination formats. Any business case that assumes otherwise will under-deliver.

---

## 8. The Adoption Reality

### 8.1 Named adopters, with the pilot/production distinction enforced

| Organisation | Type | What they have done | Stage | Source (accessed 15 Sep 2026) |
|---|---|---|---|---|
| **JSCC** (Japan Securities Clearing Corporation) | CCP | First CCP and first Japanese entity to adopt DRR + CDM in a production environment; production-parallel reporting to JFSA and CFTC from June 2025 | **PRODUCTION** | JSCC/FINOS press release, 15 Jan 2025 |
| **JPMorgan Chase** | Bank | Implementing ISDA DRR "as a primary reporting mechanism using the open-source CDM"; a maintainer (OrgRep) and DRR contributor | **PRODUCTION (self-described, via ISDA webinar)** | isda.org DRR InfoHub; CDM maintainer list |
| **BNP Paribas** | Bank | Successfully tested DRR for CFTC rules (Nov 2022); described as using CDM/DRR in production | **PRODUCTION** | ISDA 2 Nov 2022; TradeHeader FAQ, 10 Feb 2026 |
| **Pictet Group** | Bank / asset manager | Named among firms using CDM and DRR in production | **PRODUCTION** | TradeHeader FAQ, 10 Feb 2026 |
| **Standard Chartered** | Bank | DRR development contributor and institutional supporter | **CONTRIBUTOR** (production status not stated publicly) | ISDA DRR InfoHub; REGnosys, Jun 2024 |
| **Natixis CIB** | Bank | Adopted the ISDA DRR solution | **ADOPTION ANNOUNCED, 8 Apr 2026** (production vs pilot not specified in the announcement) | isda.org, 8 Apr 2026 |
| **DTCC** | Market infrastructure | Collaboration with REGnosys to connect Rosetta to DTCC's Global Trade Repository for testing DRR-generated submissions | **PLATFORM INTEGRATION / TESTING** | DTCC/REGnosys, 10 Apr 2024 |
| **LSEG** (TradeAgent) | Market infrastructure / vendor | Integrated the ISDA DRR solution into TradeAgent | **PRODUCT INTEGRATION, 17 Mar 2026** | isda.org, 17 Mar 2026 |
| **VERMEG** (COLLINE) | Vendor | Integrated the CDM into its collateral management system | **PRODUCT INTEGRATION, 10 Jun 2024** | isda.org, 10 Jun 2024 |
| **TradeHeader** | Consultancy / vendor | CDM co-creator; chairs/guides working groups; ISDA partner for CDM/DRR advisory and training; generated multiple CDM maintainers | **PRODUCTION SERVICES** | TradeHeader FAQ, 10 Feb 2026; CDM maintainer list |
| **REGnosys** | Vendor | Rosetta platform; commissioned by ISDA/ICMA/ISLA to provide CDM Release Management; holds maintainer seats; contributes the Rune DSL | **PRODUCTION SERVICES / CORE DEPENDENCY** | CDM README; REGnosys, 24 Jun 2024 |
| **Tokenovate, Fragmos Chain, FT Advisory** | Vendors | DRR contributors (Tokenovate); Editors on the CDM (Fragmos Chain, FT Advisory) | **CONTRIBUTOR** | GOVERNANCE.md; TradeHeader FAQ |
| **Strate** | Trade repository (South Africa) | Will publish South African reporting rules as machine-executable DRR code ahead of the regime's go-live | **PRE-LAUNCH PILOT** | ISDA DRR InfoHub |
| **Ontario Securities Commission (OSC)** | Regulator | Joint review of DRR code against the CSA Technical Manual; coding of revisions for UAT targeted mid-April 2027 | **PILOT / ENGAGED (timings ISDA-labelled as estimates)** | ISDA DRR InfoHub |
| **ISDA + Digital Asset** | Vendor pilot | CDM clearing pilot using DAML (Oct 2020); CDM tooling (Apr 2019) | **PILOT (historical)** | isda.org, Oct 2020 / Apr 2019 |
| **ISDA + BNP Paribas** | Bank pilot | Successfully tested DRR against CFTC rules | **PILOT (then production)** | isda.org, 2 Nov 2022 |
| **FCA + Bank of England** | Regulators | UK DRR pilot phases 1 (2018) and 2 (2019) built on ISDA CDM 2.0 | **PILOT (historical, foundational)** | isda.org, 21 May 2019 |

### 8.2 What the pattern shows

- **The centre of gravity is regulatory reporting, not post-trade processing.** Every named *production* deployment verified here is a reporting deployment. Collateral has shipped vendor integrations. Tokenisation has working groups and decks.
- **The production list is short and banked around three or four majors.** BNP Paribas, JPMorgan Chase, Pictet and JSCC are the recurring names. That is a real list — but it is not "the industry".
- **Infrastructure and CCP involvement is the strongest signal.** JSCC produced the first genuine CCP production commitment; DTCC connected to the ecosystem at the testing layer. Production adoption that matters systemically will come through CCPs and trade repositories, not through bilateral bank enthusiasm.
- **Vendors are adopting faster than banks**, because embedding the CDM is a product differentiator for them and a cost centre for banks.
- **News flow is healthy but thin in numbers:** through 2026 ISDA's DRR announcements are integration and adoption announcements at a rate of roughly one per quarter, not an avalanche.

### 8.3 The gap, characterised honestly

- **Membership ≠ adoption.** FINOS has surpassed 100 members (2025), and ISDA has over 900 member institutions; ISDA's DRR contributor logos number around a dozen. Wide membership and thin production adoption co-exist comfortably in standards initiatives.
- **Coverage ≠ deployment.** Eight rule sets have DRR coverage; the number of institutions running DRR as their primary reporting engine is single-digit in public evidence.
- **Contribution ≠ commitment.** Many firms contribute to the model (and can point to working-group participation) without having any CDM-dependent system in production.
- **The adoption bottleneck is not the model; it is the mapping and the change-management cycle** (§7). That is where the cost lands and where programmes stall.

---

## 9. The AI and LLM Angle

This section cross-references the library's existing AI/LLM work — particularly `technology/ai_llm/llm_agent_use_cases_finance_guide.md`, `technology/ai_llm/llm_agents_quantitative_finance_guide.md` and `technology/ai_llm/ai_governance_framework_guide.md` — rather than re-deriving LLM mechanics.

### 9.1 What is actually documented (verified)

| Initiative | Who | Stage | Source (accessed 15 Sep 2026) |
|---|---|---|---|
| **DRR traceability tool** | ISDA client-side; **Gentek AI** developing | **ANNOUNCED, IN DEVELOPMENT.** ISDA launched an RFQ on 30 Oct 2025 and selected Gentek AI on 30 Mar 2026. The tool "will allow users to look back at the history of DRR decision-making and pinpoint when and why coding choices were made… Using artificial intelligence (AI), the tool will extract information from **minutes of ISDA working group meetings**". Stated purpose: "an open, auditable and AI-driven traceability framework that links regulatory text, working group decisions and DRR code" | ISDA press releases, 30 Oct 2025 and 30 Mar 2026 |
| **AI-assisted development of the standard's own toolchain** | FINOS / Rune DSL community | **IN USE.** The `rune-dsl` repository carries `CLAUDE.md` and a `.claude/skills/migrate-xtend-generator` skill directory, i.e. an AI-assistant workflow specifically for migrating Xtend-based code generators to Java; recent generator migrations are explicitly titled "Convert RuleGenerator, ReportGenerator and FunctionGenerator to Java and migrate them to the fluent API" | finos/rune-dsl tree and release notes, 15 Sep 2026 |
| **AI-assisted governance/documentation upkeep in the CDM repo itself** | CDM maintainers | **IN USE.** Maintainer-roster and documentation pull requests on the CDM repository in September 2026 carry `Co-Authored-By: Claude Opus 5` trailers and a `Claude-Session:` link, with the summary text explicitly noting what was applied to which documentation pages | CDM repo commit metadata, 11-15 Sep 2026 |
| **FINOS AI programme context** | FINOS | **ACTIVE.** FINOS AI now runs two pillars — **Governance As Code** and **Agentic FSI Specifications**; the **FINOS AI Fund and Governing Board launched June 2026** with DTCC, Morgan Stanley, NatWest and RBC as founding members; a first-draft AI Governance Framework was released at OSFF NY | ai.finos.org, accessed 15 Sep 2026 |

### 9.2 What is *not* documented

- No ISDA or FINOS artefact was found that ships **natural-language → CDM model** generation as a product, nor any published accuracy benchmark for it. Claims that an LLM can "read a rulebook and produce valid Rune DSL" are, on the evidence available on 15 September 2026, **vendor positioning or research demos**, not shipped capability.
- The DRR traceability tool is explicitly a *decision-audit* tool: it mines meeting minutes to explain why code is the way it is. That is not rule interpretation, and it should not be sold internally as such.
- No AI workstream was found with the CDM as its formal scope inside FINOS project governance. The AI-relevant CDM activity sits inside the ISDA DRR programme and inside the DSL toolchain, not under a CDM "AI Working Group".

### 9.3 Why a deterministic model is a useful substrate for probabilistic processing

Labelled as **analysis**, grounded in the verified facts above:

1. **Grounding and hallucination containment.** An LLM interpreting a reporting rule has an open-ended output space. Anchored to a fixed model with typed attributes, enumerated code lists and cardinality constraints, its output space becomes *checkable*: generated logic either validates against the CDM or it does not. The validator does what the reviewer cannot.
2. **The model becomes the specification of record.** Because the CDM expresses both data and logic, you can ask an LLM to produce logic *in* the model rather than in a bespoke pipeline — the artefact is reviewable by a business analyst, which is exactly the design intent of Rune's human-readable syntax.
3. **Traceability is the missing control.** The reason the Gentek tool matters is that AI-assisted rule interpretation is only usable in a regulated context if every coding decision traces back to a provision of the rule text and a committee decision. The CDM's rule-reference structures (`regulatoryReference` with `provision`, `segment`, `rationale` — verified in the DRR glossary) exist precisely to make that link machine-readable.

### 9.4 The honest state of play

- **Shipped:** AI assistance inside the standard's *toolchain* and *documentation process* (verified in repositories). AI-based decision traceability (in development, announced).
- **Announced but not shipped:** AI traceability tool for DRR users.
- **Not evidenced at all:** LLM-driven regulatory rule interpretation producing CDM/Rune artefacts at production quality, natural-language-to-model mapping products, and any measured reduction in interpretation effort from AI. Treat all three as unverified until a primary source ships them.

---

## 10. The Business Case and the Honest Limits

### 10.1 The value argument, and what backs it

| Value driver | Claim | Evidence status |
|---|---|---|
| **Reconciliation cost** | Shared event representation removes reconciliation breaks caused by divergent per-firm event definitions | **Sourced as intent** (CDM README, ISDA): "reducing the need for reconciliation caused by variations in how each firm records trade lifecycle events". **Not quantified publicly** |
| **Regulatory reporting cost** | Interpret each rule set once, industry-wide, instead of once per firm per rule set; future rule changes delivered via central code updates | **Sourced as intent and widely repeated** by ISDA, FINOS, TradeHeader. ISDA/Capgemini published "Industry Perspectives on the ISDA DRR" (10 Nov 2025) based on interviews with adopting firms, described as exploring "the benefits they have realized" — the closest thing to an impact study; this guide treats its contents as vendor-sponsored evidence and does not quote figures from it |
| **Data quality for regulators** | Cleaner, more comparable data improves systemic-risk analysis; fewer misreporting penalties | **Sourced as intent**; consistent with the DRR's validation-rule coverage |
| **Time-to-market for new rules** | DRR updates land ahead of compliance dates | **Sourced with dates** — each coverage date in §6.1 precedes or matches its compliance date. Strongest concrete benefit in the whole file |
| **Quantified cost savings** | Any specific percentage or dollar figure | **NOT VERIFIED.** No primary-source, attributable, methodology-described figure was located. Treat every such number you see in a deck as derived from a vendor or consultancy model |

### 10.2 The barriers and critiques — labelled

**Evidenced (structural, visible in the artefacts themselves):**

1. **Network-effect dependency.** The value of a shared model scales with the number of counterparties and infrastructures using it. With four or five named production adopters and one CCP, a bank cannot yet bank on counterparties speaking CDM. This is a *measured* thinness, not a hypothesis.
2. **Java-only complete distribution.** "Only the Java executable code distribution is complete" (CDM docs). For non-JVM firms, the adoption cost is front-loaded and real.
3. **DRR is not open source.** The most valuable layer for a bank — the reporting logic — is proprietary ISDA code, free to use. Firms get "free", not "open", and they carry licence-compliance obligations for it. Note the contrast with the model itself.
4. **Vendor concentration.** REGnosys acts as CDM release manager (under contract to the associations), holds three of fourteen maintainer seats, owns the Rosetta platform named in CDM documentation as the application layer, provides the DRR execution engine per the DRR glossary, and is the reference commercial translation service. Meanwhile exactly **one** of fourteen maintainers represents a bank. Both facts are verifiable from primary sources and both are legitimate governance questions for a bank's risk review.
5. **Release cadence creates standing cost.** One production version, intended to last about a year, then maintenance-only; a new major every 12-18 months. Budget an annual regression cycle.
6. **Model complexity.** The 8.x development line adds stateful instruction composition, deliberate step sequencing and overlay processing — more power, more to learn, more to get wrong. Skills scarcity is evidenced by the same handful of firms (REGnosys, TradeHeader, ISLA, ICMA, ISDA) supplying most maintainers and editors.
7. **Scope creep risk.** The published scope already spans OTC derivatives, cash securities, securities financing and commodities, "and can expand to include other Capital Markets products and Asset Classes" — with active working groups on collateral, tokenised assets, physical risk and legal documentation. A model that grows in every direction is harder to keep coherent, and creates a queue of change requests that competes with your own.
8. **Reporting eligibility gap.** Deciding *whether* to report remains the firm's problem, so DRR does not remove end-to-end reporting effort.

**Analysis (reasoning, not sourced):**

9. **Internal-model resistance and migration cost.** Product masters are load-bearing and politically defended. The CDM competes with the internal model that already works, and the migration cost lands on the firm while some of the benefit accrues to counterparties.
10. **"Standards that never get adopted" risk.** The CDM's eight-year history is long by standards terms but the production-adoption list remains short. A bank should size its exposure so that failure of the initiative is survivable — i.e. adopt CDM *at the boundary* (reporting, mapping) with the option to deepen later, rather than re-platforming the core on it.
11. **Governance asymmetry risk.** Sponsorship by three trade associations plus release management by one vendor is workable, but it means roadmap priority broadly tracks association member interests (derivatives, repo, securities lending documentation) rather than any single bank's priorities.
12. **Open-source-governance optics.** Community Specification licensing, CLAs, working groups and consensus decision-making look open, and are. But the *most commercially valuable* generated logic (DRR) is closed, the *tooling* is largely one vendor's, and the maintainer roster is vendor-heavy. A bank's OSS review board should examine the whole chain, not just the repo licence.

---

## 11. The Comparison

Positioning exercise only. Each comparator has (or should have) its own treatment elsewhere in this library: BIAN in `technology/architecture/bian_standard_guide.md` and `banking/bian_banking_architecture_guide.md`; ISO 20022 in `banking/iso_20022_core_processes_guide.md`; FIX in `banking/fix_protocol_guide.md`; tokenised assets in `banking/tokenized_assets_guide.md`; banking data models generally in `banking/data_models_banking_insurance_guide.md`.

### 11.1 The axes table

| Axis | **FINOS CDM** | **BIAN** | **ISO 20022** | **FpML** | **Closed vendor models** |
|---|---|---|---|---|---|
| **What it fundamentally is** | Semantic + process domain model of financial transactions, with embedded logic, shipped as generated code | Reference architecture / service-landscape metamodel for a whole bank (capability and service domains) | Message-definition standard (syntax + dictionaries) for exchanging financial data | Message schema standard for OTC derivative transactions and lifecycle events | Proprietary data models inside a vendor platform (e.g. a trading/collateral system's internal object model) |
| **Scope** | OTC derivatives, cash securities, securities financing, commodities; expanding to other capital-markets products | Whole-bank: retail, payments, lending, deposits, cards, capital markets, risk, operations | Payments, securities, trade services, cards, FX, and increasingly reporting | OTC derivatives only (rates, credit, FX, equity, commodity), plus associated lifecycle messages | Whatever the vendor's product covers |
| **Granularity** | Deep on transaction economics, lifecycle events, legal agreements and reporting logic; shallow-to-none on bank operations, ledgers, customers | Coarse-to-medium on business capabilities and services; deliberately not a data model of individual trades | Fine on message fields and code lists; silent on internal logic and process semantics | Fine on trade economics and message structure | Varies; typically deep where the product is deep and absent elsewhere |
| **Governance** | FINOS (Linux Foundation) under Community Specification; sponsored by ISDA, ICMA, ISLA; release management commissioned from REGnosys; maintainers vendor-heavy (1 of 14 from a bank) | BIAN e.V., a membership association of banks and vendors with a formal standards release cycle | ISO TC68/SC9 with registration-authority and maintenance governance; slow, high-consensus, regulator-backed | Historically ISDA (ISDA owns FpML); working groups dominated by dealers and vendors | Single vendor, unilateral change control |
| **Licence** | Specification: Community Specification License 1.0. Code: Apache-2.0 by default. Generated artefacts: per CDM release. **DRR logic: proprietary, free to use.** Pre-4.0.0 CDM: ISDA CDM™ Licence | BIAN membership/licence terms for the specification and artefacts | Purchasable ISO standard; message schemas circulated under ISO terms | ISDA/FpML licence (free to use under licence terms) | Commercial licence, no reuse rights |
| **Adoption stage** | 8 of 14 committed reporting rule sets covered; verified production adopters number in the single digits (JSCC, JPMorgan Chase, BNP Paribas, Pictet); strongest in regulatory reporting, thinner in post-trade and collateral | Entrenched as a *reference* architecture in bank architecture practices; implementation is interpretive and firm-specific | **Highest adoption of the four by a wide margin** — mandated or adopted for payments, securities and reporting globally | Long-established in derivatives messaging; deep installed base, now coexisting with ISO 20022 reporting requirements | Ubiquitous by default — every firm has one, because every platform has one |
| **Who should care at a bank** | Regulatory reporting, post-trade operations, collateral, market data/product reference, capital-markets architecture | Enterprise architecture, capability mapping, service design | Payments, reporting, messaging/integration, securities operations | Derivatives technology, trade capture, confirmation | Anyone integrating the platform |

### 11.2 Short contrasts, deliberately brief

- **CDM vs BIAN.** Different categories that are frequently and unhelpfully conflated. BIAN answers "*what services does a bank have?*" at the capability level; the CDM answers "*what exactly happened to this swap, and what does the regulation require us to do with it?*". BIAN is a reference architecture you map your estate to; the CDM is a model you generate code from and ship. A bank can — and typically would — use both, at different layers, with no overlap.
- **CDM vs ISO 20022.** Complementary and hierarchical: CDM *computes*, ISO 20022 *transmits*. The DRR emits ISO 20022. Confusing them leads to the false belief that adopting ISO 20022 gets you reporting logic — it gets you a message format.
- **CDM vs FpML.** Same asset class, different eras and purposes. FpML is a representation/transport schema for derivative transactions; the CDM is a model with embedded executable logic. The open-source "Translate 2.0" FpML→CDM translation (Q4 2025) makes them formally interoperable, which is the constructive way to hold both.
- **CDM vs closed vendor models.** The vendor model will always be deeper for that vendor's product and always be non-portable. The CDM's value is portability and comparability; the vendor model's value is shipping now. The realistic posture is CDM **at the boundary** of the vendor estate (reporting, interoperability, analytics exchange), not inside it.

---

## 12. The Cymbal Bank Worked Example

> **Everything in this section is fictional and illustrative.** Cymbal Bank is a fictional institution. All numbers, dates, effort estimates and success criteria are invented to demonstrate an assessment method and must not be quoted as market data. Real entity names appear only where the guide is citing real sources elsewhere.

### 12.1 The bank, the pain, and the question

**Cymbal Bank** is a fictional wholesale and transaction bank headquartered in Singapore, with derivatives booking in Singapore, Hong Kong, London and New York, roughly 40,000 OTC derivative trades executed or cleared per year, and a post-trade estate that has grown by acquisition.

**The pain points we actually own:**

| Pain | Current state (illustrative) | Owner |
|---|---|---|
| Regulatory reporting change cycles | Six reporting regimes in scope (MAS, HKMA, ASIC, JFSA, EU EMIR, UK EMIR), each with its own bespoke interpretation and code; 2-4 rule-change releases a year; ~14 FTE-equivalent across compliance change and technology | Head of Regulatory Reporting |
| Post-trade reconciliation backlog | ~3,200 lifecycle-event breaks per month against counterparties and CCPs; ~60% attributed to event representation and timing differences | Head of Post-Trade Operations |
| Collateral processing | Manual eligible-collateral schedule capture and collateral-eligibility disputes during onboarding; new ISDA/CSA onboarding takes ~6 weeks | Head of Collateral Management |
| Tokenisation ambition | Board-level digital-asset ambition; two PoCs on tokenised money-market funds; no agreed semantic layer | Chief Digital Officer |

**The question put to architecture:** *should Cymbal Bank pilot the FINOS CDM, and if so, where — and what would justify the second stage?*

### 12.2 Use-case selection against our pain

Scored by the bank's architecture function (1 = poor, 5 = strong). Illustrative.

| Use case | Pain severity | CDM maturity for this case | Effort to first value | Dependency on others (network effect) | Score |
|---|---|---|---|---|---|
| Multi-jurisdiction regulatory reporting (DRR) | 5 | 5 (8 covered regimes; production evidence exists) | 3 (mapping + DRR integration) | 2 (works alone) | **21** |
| Post-trade event reconciliation | 4 | 3 (model supports it; production evidence thin) | 2 | 5 (needs counterparties) | 14 |
| Collateral and eligible-collateral schedules | 4 | 4 (working groups; vendor integration shipped) | 2 | 4 (needs counterparties/CSAs) | 14 |

**Selection: regulatory reporting first.** It is the only case where the CDM is mature, the bank is the primary beneficiary, and the network effect is *not* a prerequisite — the regulator and the trade repository are already the counterparties. Everything else is gated behind evidence.

### 12.3 Target operating model implications

- **A CDM capability, not a CDM project.** A small permanent team: one product/trade-data architect, two Rune/Java modellers, one reporting business analyst, one regulatory change analyst, plus vendor support. Seconded from existing change budgets, not net-new headcount in the steady state.
- **Internal model stays.** The bank does not re-platform booking. A translation service at the boundary owns internal → CDM and CDM → internal mapping, versioned alongside the internal product master.
- **Mapping is a governed asset.** The mapping between the internal product model and the CDM is code, reviewed, tested and version-pinned, with the same change control as any regulated system.
- **Upgrade calendar is a standing item.** Production releases last roughly a year and majors arrive every 12-18 months; the bank budgets an annual CDM regression and upgrade cycle in its change calendar from day one.
- **Skills.** Java 21, Maven, Rune DSL, plus derivatives product knowledge. Assume 6-9 months for a competent Java engineer with derivative exposure to become independently productive in the model (illustrative estimate; no public benchmark was found — see §14).
- **Governance participation.** Two named individuals as Participants, one attending the Contribution Review and Derivatives Working Groups, with a route to Editor status if the bank contributes meaningfully. Membership of FINOS is the enabling step.

### 12.4 The mapping exercise, concretely

| Step | Activity | Output | Illustrative effort |
|---|---|---|---|
| 1 | Inventory the internal product master and identify in-scope products (rates and FX vanilla plus a small structured book) | Product inventory, ~180 live product types reduced to ~40 semantic families | 4 weeks |
| 2 | Map the internal product model to CDM product objects (economic terms layer) | Mapping specification + regression test set | 8 weeks |
| 3 | Map internal lifecycle events to CDM business events (reset, payment, exercise, amendment, termination, clearing) | Event mapping + gap list | 8 weeks |
| 4 | Populate identifiers: LEI, UTI, UPI, MIC, internal IDs | Identifier standard + validation rules | 4 weeks |
| 5 | Build the translation service (internal model → CDM JSON) | Running service in the test environment | 10 weeks |
| 6 | Integrate DRR for the first regime (start with the regime with the nearest compliance date) | Reports generated and reconciled against current output | 12 weeks |
| 7 | Round-trip reconciliation: DRR output vs incumbent reporting output, trade by trade | Reconciliation report and disposition of every break | 6 weeks |

**Two mapping design decisions to take early:**

- **Extend or translate?** Prefer translation with a **separate custom namespace** for anything Cymbal-specific, per the CDM community's own guidance, so that model upgrades stay cheap. Never patch generated code if you intend to upgrade.
- **Where does enrichment live?** Data the bank holds but the model does not need (internal product codes, desk codes, client segmentation) stays out of the CDM representation and is attached at the reporting layer.

### 12.5 Tooling and vendor decision

| Option | Description | Illustrative cost posture | Our assessment |
|---|---|---|---|
| **Own build on open source** | Rune DSL + open-source generators + open-source Translate 2.0 + own integration | Lowest licence, highest run cost | Viable for the translation service; high risk for DRR execution |
| **REGnosys Rosetta platform (commercial)** | Modelling platform, translation service, integration-as-a-service; also the DRR execution engine per the DRR glossary | Commercial licence, per-use | **Preferred primary.** Single vendor covers mapping, modelling and execution. This is also the concentration risk (§10.2) |
| **Consultancy-led (e.g. TradeHeader model)** | CDM/DRR advisory, implementation services and training by a CDM co-creator | Day-rate services | **Preferred secondary** — training and independent review, and it partially mitigates the single-vendor exposure |
| **Reporting vendor with embedded DRR** | Use LSEG TradeAgent or a reporting utility that has integrated DRR | Bundled in existing licence | **Evaluate in parallel.** Fastest path to a second production-grade option |

**Decision for the pilot:** build the mapping on open source (so the mapping asset is ours and portable), use the commercial Rosetta platform for modelling and translation during the pilot, and use DRR as shipped for report generation. Re-test the build/buy boundary at each stage gate.

### 12.6 Pilot design

| Element | Design |
|---|---|
| **Objective** | Prove that CDM-based reporting logic (DRR) can generate Cymbal Bank's monthly regulatory report for **one** regime, reconciled trade by trade against the incumbent output, at acceptable cost and control |
| **Scope** | One regime; two product families (vanilla interest-rate swaps, vanilla FX forwards); one legal entity; 12 months of historical trades replayed plus 3 months of live trades in parallel |
| **Duration** | 2 quarters (illustrative) |
| **Team** | 6 FTE core (architect, 2 engineers, business analyst, regulatory change analyst, operations SME) plus vendor support |
| **Method** | Parallel run: incumbent output remains the submission of record throughout; the CDM/DRR output is compared, never submitted |
| **Data** | Production-equivalent data, masked where required; no new data pipelines |

**Success criteria — all must be met for stage 2:**

| # | Criterion | Threshold (illustrative) | Measurement |
|---|---|---|---|
| 1 | Trade-level agreement with incumbent output | ≥ 99.5% field-level agreement after disposition of known-good differences | Reconciliation run |
| 2 | Unexplained differences | ≤ 0.25% of trades, each individually dispositioned | Reconciliation report |
| 3 | Regulatory-textable traceability | 100% of generated report fields traceable to a rule provision | DRR rule references |
| 4 | Runtime performance | Full monthly cycle within the reporting window with ≥ 30% headroom | Performance test |
| 6 | Model upgrade rehearsal | A CDM minor-version upgrade executed end to end within 5 working days | Dry run |
| 7 | Cost control | Total pilot cost within the approved envelope, and a defensible steady-state cost per report per regime | Finance sign-off |
| 8 | Business sign-off | Head of Regulatory Reporting would accept the CDM pipeline as submission-of-record for the next regime change | Written acceptance |

### 12.7 Cost and effort, with the numbers labelled

**Illustrative only.** No public, attributable cost benchmark for CDM adoption was found (§14); these figures are constructed to demonstrate the shape of the decision, not to predict it.

| Item | Pilot (2 quarters) | Steady state (annual) |
|---|---|---|
| Internal team (6 FTE avg) | USD 480,000 | USD 560,000 (capability retained) |
| Vendor platform / translation licence | USD 180,000 | USD 220,000 |
| Consultancy (training + independent review) | USD 120,000 | USD 40,000 |
| Infrastructure and environments | USD 60,000 | USD 70,000 |
| **Total** | **USD 840,000** | **USD 890,000** |

**The comparison that actually decides it** (also illustrative): the bank currently spends an estimated USD 2.4m a year of combined compliance-change and technology effort on regime-specific reporting rewrites, of which perhaps USD 1.1m is genuinely *duplicated* work that a shared industry interpretation could remove. If the CDM path costs ~USD 0.9m a year in steady state and removes ~USD 1.1m of duplicated effort while also reducing unexplained reconciliation breaks, the case closes — **narrowly, and only if the duplicated-effort estimate survives validation.** If the CDM path costs USD 1.4m or the duplicated effort is only USD 0.6m, it does not close on cost alone and must be justified on control quality and franchise benefits.

**Break-even is therefore the single most sensitive assumption in the case.** Validate it in the pilot before approving stage 2.

### 12.8 Risk register

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | **Network-effect risk** — counterparties, CCPs and trade repositories do not adopt CDM-shaped data, so benefits stay one-sided | Medium-high | High | Adopt at the reporting boundary only; verify CCP/TR direction per jurisdiction before committing to stage 3 |
| 2 | **Vendor concentration** — mapping, modelling, execution and release management all trace back to one vendor | High | Medium-high | Contract for exit; keep the mapping asset in our own repositories; maintain a second reporting-vendor option; participate in FINOS governance |
| 3 | **DRR licence change** — the reporting logic is proprietary; terms could change | Low-medium | High | Legal review of the DRR licence before pilot exit; document the fallback (own Rune logic for the regime) and its cost |
| 4 | **Non-Java ecosystem friction** — we are primarily a JVM shop for post-trade but not everywhere | Medium | Medium | Confirm Java-first for in-scope systems; treat other languages as out of scope for the pilot |
| 5 | **Skills scarcity** — few engineers globally know Rune/CDM | High | Medium | Buy training early; twin our engineers with the consultancy; budget 6-9 months to productivity |
| 6 | **Model upgrade treadmill** — annual regression and upgrade load | High (certain) | Medium | Build the upgrade rehearsal into the pilot as an exit criterion (criterion 6) |
| 7 | **Scope creep into tokenisation and collateral** while the core is unproven | Medium | Medium | Governance: stage gates with a written decision record; the CDO's tokenisation ambition is explicitly deferred to stage 4 |
| 8 | **Mapping debt** — the internal-model mapping becomes an unmaintained artefact | Medium | High | Treat the mapping as a regulated asset: owned, versioned, tested, with a named accountable owner |
| 9 | **Benefits overstatement** — internal stakeholders book the reconciliation saving before it exists | Medium-high | Medium | Benefits register with measured baselines; no benefit claimed until measured post-implementation |
| 10 | **Standard stalls** — production adoption plateaus at a handful of firms | Medium | High | Boundary adoption limits stranded cost; a CDM wrapper around reporting logic is salvageable even if the ecosystem slows |

### 12.9 Stage gates

| Gate | Decision | Evidence required | Illustrative date |
|---|---|---|---|
| **G0 — Approve pilot** | Fund a two-quarter pilot | Use-case selection (§12.2), cost envelope (§12.7), named team | Q1 |
| **G2 — Pilot exit** | Proceed to regime 1 in production-parallel; approve steady-state budget | All eight success criteria (§12.6) met; DRR licence reviewed; exit plan for the vendor contract | Q3 |
| **G3 — Regime 1 as submission of record** | Make the CDM pipeline the reporting mechanism for one regime | Two consecutive clean parallel cycles; regulator/submission acceptance confirmed by the reporting utility | Q4 |
| **G4 — Extend to regimes 2-3** | Scale the same pipeline to more jurisdictions | Marginal cost per additional regime demonstrably lower than the incumbent pattern | Year 2 |
| **G5 — Post-trade event reconciliation** | Only then consider lifecycle-event reconciliation | Evidence that counterparties/CCPs in scope can exchange CDM-shaped data | Year 2-3, evidence-gated |
| **G6 — Collateral and tokenisation** | Separate business cases | Collateral: vendor or counterparty CDM collateral schedules available. Tokenisation: CDM tokenised-asset model published and stable | Year 3+, not now |

### 12.10 What Cymbal Bank cannot establish before the pilot ends

Stated plainly, because this is what makes the assessment honest:

1. **Whether our internal data can populate the model cleanly at scale.** The historical data cut will not expose every long-tail product or every mis-booked trade. Expect the pilot to understate mapping effort.
2. **The true run cost of the annual model-upgrade cycle.** One rehearsal is not a trend, and the 8.x instruction-composition work will change the shape of future releases.
3. **Whether the reconciliation saving is real.** The 60% attribution of breaks to event representation is an operational judgement, not a measurement. The pilot will not test counterparty-side reconciliation at all.
4. **Whether regulators and repositories will genuinely accept CDM-derived output as business-as-usual.** DRR coverage is one thing; a bank's own regulator accepting a machine-generated report as the submission of record is a separate conversation that depends on that jurisdiction.
7. **Counterparty willingness.** Nothing in a single-firm pilot establishes that our counterparties will ever speak the model. That is the network-effect question, and it can only be answered by the market.

---

## 13. The Claims Audit

Every load-bearing claim, with its source, the quality of that source, and the date it was checked. Governance and adoption claims go stale fastest — re-verify anything older than two quarters.

| # | Claim | Verdict | Source | Source quality | Verified on |
|---|---|---|---|---|---|
| 2 | ISDA, ICMA and ISLA appointed FINOS to host the CDM repository on 8 Sep 2022, with migration targeted by start of 2023 | **Verified** | ISDA press release, 8 Sep 2022 | Primary | 15 Sep 2026 |
| 3 | The CDM is a FINOS-hosted open standard "starting in February 2023" | **Verified** | CDM README; GOVERNANCE.md | Primary | 15 Sep 2026 |
| 4 | The CDM was contributed to FINOS "in 2019" | **REJECTED** | — | No primary source supports this; the 2019 event was CDM 2.0 publication and the UK DRR pilot | 15 Sep 2026 |
| 5 | Post-transfer the CDM is distributed under an Apache licence | **Flagged / qualified** | ISDA press release (8 Sep 2022) says Apache; the actual LICENSE.md is Community Specification License 1.0; Apache-2.0 applies by default to *source code* where no other licence is stated (CSL §4) | Primary but the 2022 statement is superseded | 15 Sep 2026 |
| 6 | CDM versions before 4.0.0 were licensed under the ISDA CDM™ Licence | **Verified** | NOTICE.md | Primary | 15 Sep 2026 |
| 8 | The DRR is not open source; it is proprietary, free to use for ISDA and non-ISDA members | **Verified** | drr-docs.isda.org Governance; TradeHeader DRR FAQ, 10 Feb 2026 | Primary + expert secondary | 15 Sep 2026 |
| 9 | Current CDM Production line is 7.x (7.0.0 GA 17 Jul 2026; 7.3.0 on 9 Sep 2026); 6.x in maintenance (6.28.1 on 14 Sep 2026); 8.0.0-dev.5 in development; 5.x last released 5.40.0 on 9 Jul 2026 | **Verified** | GitHub Releases API for finos/common-domain-model | Primary | 15 Sep 2026 |
| 10 | "The latest CDM version is the one GitHub marks *Latest*" | **Rejected as a method** | GitHub's "Latest" badge sits on **6.28.1**, a maintenance-line tag (14 Sep 2026), while 7.3.0 is the Production line; use SCHEDULE26.md, not the badge | Primary evidence, misleading presentation | 15 Sep 2026 |
| 12 | Older material calling the language "Rosetta DSL" is stale; `github.com/finos/rosetta` returns 404 while `finos/rune-dsl` is live; model files remain `.rosetta` under `rosetta-source/`; "Rosetta" now denotes REGnosys's proprietary platform | **Verified** | GitHub (404 vs live repo); CDM docs; DRR glossary | Primary | 15 Sep 2026 |
| 13 | The CDM repo carries a FINOS "Graduated" badge while the docs site shows an "Incubating" badge | **Verified (both observed)** | CDM README vs cdm.finos.org/docs/cdm-overview, same day | Primary, internally inconsistent | 15 Sep 2026 |
| 17 | The 8.x line introduces instruction composition and stateful reset processing (8-step reset workflow) | **Verified** | GitHub release notes for 8.0.0-dev.4/.5 | Primary | 15 Sep 2026 |
| 19 | DRR coverage dates: CFTC 5 Dec 2022; JFSA 1 Apr 2024; EU EMIR 29 Apr 2024; UK EMIR 30 Sep 2024; ASIC and MAS 21 Oct 2024; CSA 25 Jul 2025; HKMA 29 Sep 2025 | **Verified** | ISDA DRR InfoHub | Primary | 15 Sep 2026 |
| 20 | ISDA is committed to 14 rule sets in 9 jurisdictions; 8 were applied as of Mar 2026 | **Verified** | ISDA DRR InfoHub; ISDA press release 30 Mar 2026 | Primary | 15 Sep 2026 |
| 21 | EU/UK MiFIR, EU/UK SFTR and Switzerland are in development, not live | **Verified** | ISDA DRR InfoHub; TradeHeader FAQ | Primary + expert secondary | 15 Sep 2026 |
| 23 | JSCC is the first CCP and first Japanese entity to adopt DRR/CDM in production | **Verified** | JSCC press release via FINOS, 15 Jan 2025 | Primary (the adopting institution) | 15 Sep 2026 |
| 24 | BNP Paribas, Pictet and JPMorgan are using CDM/DRR in production | **Verified as reported** | TradeHeader DRR FAQ, 10 Feb 2026; ISDA webinar listing; ISDA 2 Nov 2022 | Expert secondary / adopting-source secondary | 15 Sep 2026 |
| 25 | LSEG TradeAgent integrated the ISDA DRR (17 Mar 2026); Natixis CIB adopted it (8 Apr 2026); VERMEG integrated the CDM into COLLINE (10 Jun 2024) | **Verified** | isda.org press releases | Primary | 15 Sep 2026 |
| 28 | The CDM is "the semantic layer for tokenisation" | **Flagged / overstated** | Tokenized Assets WG is *extending* Asset and Settlement structures; atomic settlement and intra-day count fractions are explicitly future work | Primary, contradicts the claim | 15 Sep 2026 |
| 29 | ISDA selected Gentek AI on 30 Mar 2026 to build an AI-based DRR traceability tool (extracting from working-group minutes); announced, in development | **Verified** | ISDA press releases, 30 Oct 2025 and 30 Mar 2026 | Primary | 15 Sep 2026 |
| 30 | LLMs are used in production to interpret reporting rules into CDM/Rune | **REJECTED / unverified** | No primary source shipped such a capability | Absence of evidence | 15 Sep 2026 |
| 31 | AI assistance is already used inside the CDM/Rune toolchain and documentation process (`CLAUDE.md`, `.claude/skills` in rune-dsl; AI co-authored maintainer-list PRs in the CDM repo) | **Verified** | finos/rune-dsl tree; CDM commit metadata | Primary | 15 Sep 2026 |
| 32 | The CDM is a widely adopted industry standard | **Flagged** | 6,915 commits, 278 stars, 134 forks; 4-5 named production adopters; 8 of 14 committed rule sets covered | Primary — supports "significant but not ubiquitous" | 15 Sep 2026 |
| 34 | CDM adoption removes the need for message-format mapping | **REJECTED** | The CDM's own design principles list "Mapping to existing industry messaging formats"; the four documented FpML→CDM routes all involve building or buying mapping | Primary, contradicts the claim | 15 Sep 2026 |

---

## 14. What Could Not Be Verified

Honest gaps, with what was tried. Nothing here should be assumed from memory in a later version of this guide.

**1. The CDM's original version-1.0 publication date and its first published scope.** ISDA's resource listing begins with a November 2017 webinar, "What is the ISDA CDM?"; the earliest *release* artefact referenced on that page is the June 2018 digital iteration and then CDM 2.0 in March 2019. Whether a distinct "CDM 1.0" was released, and when, could not be established from primary sources. The ISDA CDM FAQ PDF linked from the 2019 press release now returns "This file has expired", and ISDA's 2022 CDM milestone page returns a 404.

**2. The precise date of the Rosetta DSL → Rune DSL rename.** The contribution to FINOS is dated 24 June 2024 and the press release says the language was "renamed". Whether the rename happened immediately before contribution or earlier in 2024 is not stated. The `finos/rune-dsl` repository's own history would settle it; that was not attempted beyond confirming the repo exists and is live.

**3. Whether CDM 4.0.0 (June 2023) was designed before or after the FINOS migration.** The FINOS blog calls 6.0 the "second major update since the model found its home at FINOS", which implies 5.0 was the first — but 4.0.0 also post-dates the February 2023 migration. The Fintech Open Source Foundation's project-decision record behind that labelling was not located.

**4. Any independent, methodology-described cost-benefit figure for CDM adoption.** No attributable, non-vendor figure was found. The ISDA/Capgemini study (10 Nov 2025) was identified but its contents were not obtained, so it is cited only for its existence and framing.

**5. A named production CDM/DRR adopter in Singapore.** Singapore has had DRR coverage since 21 October 2024, MAS co-hosted a DRR kick-off webinar in January 2024, and MAS appears in ISDA's jurisdictional commitment list — but no primary source naming a Singapore-based institution in production was found. Given this library's Singapore focus, that is a notable gap. Related searches for a MAS-published machine-readable rule set did not produce a primary source.

**6. Whether the CFTC publishes its own machine-readable reportable-rules code.** Searches surfaced only CFTC rulemaking notices (e.g. the 30 April 2024 final amendments to large-trader reporting rules), not a CFTC-published code set. The DRR's CFTC coverage is ISDA-produced, not CFTC-produced. Claims that a regulator "publishes its rules as code" should be checked jurisdiction by jurisdiction — Strate (South Africa) is the verified example of a rules-as-code publication *ahead* of go-live, and the Ontario Securities Commission's DRR review is a verified cooperation, not a publication.

**7. The number of institutions actually running DRR in production.** No authoritative count exists. The figure derivable from public sources is "several, single digits", based on named firms only.

**8. FCA/Bank of England DRR pilot outcomes.** Phase 1 (2018) and phase 2 (2019) are documented as pilots built on CDM 2.0. What the pilot concluded, and whether it produced any published assessment, was not verified; the FCA's `/digital-regulatory-reporting` page was not successfully retrieved.

**9. Adoption of the CDM as the semantic layer in live tokenisation programmes.** The Tokenized Assets Working Group and a 2026 tokenisation use-case deck are verified. Whether any live tokenised-securities or tokenised-collateral programme has the CDM at its semantic core was not verified; no primary source was found. Do not assume that because a tokenisation project involves derivatives it uses the CDM.

**10. A FINOS-hosted AI workstream with the CDM in scope.** ai.finos.org documents two pillars (Governance As Code; Agentic FSI Specifications) and an AI Fund launched June 2026, but no CDM-specific AI working group was found in either the FINOS AI project or the CDM project's working-group list.

**11. The contents of the two flagship adoption/benefit documents.** ISDA/Capgemini's "Industry Perspectives on the ISDA DRR" (Nov 2025) and ISLA's "CDM: The Road to Adoption" (published ahead of the CDM Showcase, 26 Feb 2025) were identified but not read; both are cited here only as evidence that the documents exist.

**12. The FINOS lifecycle-stage discrepancy.** The CDM repository README displays a "Graduated" badge (linking to the FINOS graduated lifecycle definition) while the CDM documentation site displayed an "Incubating" badge on the same day, and the Rune DSL repository is still "Incubating". Which badge is authoritative for the CDM today was not confirmed with FINOS directly.

**13. Clearing-house and exchange production adoption beyond JSCC.** LCH, CME and Eurex appear in general market commentary about CDM interest; **no primary source confirming production CDM adoption by any of them was found**, so none is named as an adopter anywhere in this guide. Anyone asserting that a named CCP "runs the CDM" should be asked for the primary source.

---

## 15. Glossary

### 15.1 The standard's own terminology

| Term | Meaning |
|---|---|
| **Cardinality** | How many of something a model element may or must contain (e.g. exactly one, zero-or-more) |
| **Choice type** | A modelling construct where one of several options is present; CDM uses "deep path" access to reach a shared attribute across all options |
| **Composability** | Building complex structures by composing and qualifying small primitives from the bottom up (a CDM design principle) |
| **Domain model** | A conceptual model of a business domain that incorporates both data and logic — the sense in which the CDM is "a domain model" |
| **Embedded logic** | Business rules expressed inside the model rather than in implementation code (a CDM design principle) |
| **Editor** | A CDM governance role: experienced contributors authorised to label pull requests and issues, easing the maintainers' load |
| **Maintainer** | A CDM governance role responsible for stewardship, consensus, merging changes and publishing releases; either an OrgRep or a private individual |
| **Namespace** | The organising unit of the model — a group of `.rosetta` files with a shared prefix such as `cdm.base.datetime` |
| **Normalisation** | Abstracting common components so the same concept is defined once (a CDM design principle) |
| **Participant** | Anyone contributing to the CDM under a signed Community Specification CLA |
| **Production release** | The current supported "latest and greatest" CDM version, intended to live about a year |
| **Qualification** | Attaching additional attributes to a composed object to arrive at a specific instrument — the model's native extension idiom |
| **regulatoryReference** | The structure inside DRR logic carrying the verbatim regulatory provision, the segment pointing to the exact rule location, and an optional rationale |
| **Rules as code** | Publishing a regulatory requirement as machine-executable logic rather than prose alone |
| **Steering Working Group (SWG)** | The CDM working group that sets the roadmap, defines major releases and approves working-group formation |

### 15.2 Acronyms a banking reader needs

| Acronym | Expansion |
|---|---|
| **ASIC** | Australian Securities and Investments Commission |
| **BIAN** | Banking Industry Architecture Network |
| **CCP** | Central Counterparty (clearing house) |
| **CCLA / ICLA** | Corporate / Individual Contributor License Agreement (FINOS requirements for contributing) |
| **CDM** | Common Domain Model |
| **CFTC** | Commodity Futures Trading Commission (US derivatives regulator) |
| **CSA** | (i) Canadian Securities Administrators; (ii) Credit Support Annex (a collateral agreement) — context decides |
| **CSL** | Community Specification License 1.0 — the licence covering the CDM specification |
| **DRR** | Digital Regulatory Reporting (ISDA's rules-as-code programme built on the CDM) |
| **DSL** | Domain-Specific Language |
| **EMIR** | European Market Infrastructure Regulation (EU derivatives reporting and clearing rules) |
| **ESMA** | European Securities and Markets Authority |
| **FCA** | Financial Conduct Authority (UK) |
| **FINOS** | Fintech Open Source Foundation (the financial-services arm of the Linux Foundation) |
| **FINMA** | Swiss Financial Market Supervisory Authority |
| **FIX** | Financial Information eXchange protocol (pre-trade/execution messaging) |
| **FpML** | Financial Products Markup Language (OTC derivatives messaging standard) |
| **HKMA** | Hong Kong Monetary Authority |
| **ICMA** | International Capital Market Association (repo and bond markets) |
| **ISDA** | International Swaps and Derivatives Association |
| **ISLA** | International Securities Lending Association |
| **ISO 20022** | International messaging standard for financial data (the DRR's main report output format) |
| **JFSA** | Japan Financial Services Agency |
| **LEI** | Legal Entity Identifier (ISO 17442) |
| **MAS** | Monetary Authority of Singapore |
| **MiFID / MiFIR** | EU markets in financial instruments directive / regulation |
| **SEC** | US Securities and Exchange Commission |
| **TR** | Trade Repository |
| **UPI** | Unique Product Identifier (ISO 4914) |
| **UTI** | Unique Transaction Identifier (assigned per transaction for reporting) |

---

## 16. Cross-References and Further Reading

### 16.1 Primary sources (all accessed 15 September 2026)

| Source | What it is good for |
|---|---|
| `github.com/finos/common-domain-model` | README, governance, licence, working groups, maintainer roster, contribution process |
| `github.com/finos/common-domain-model/releases` and the GitHub Releases API | Definitive version and date evidence (use the API, not the "Latest" badge) |
| `github.com/finos/common-domain-model/blob/master/SCHEDULE26.md` | The release-state policy and 2026/27 version plan |
| `cdm.finos.org/docs/cdm-overview` | Model dimensions, components, scope statement, code-distribution completeness caveat |
| `cdm.finos.org/docs/{maintainers, CDM-Tokenized-Assets-WG, working-groups}` | Governance detail and working-group scope |
| `github.com/finos/rune-dsl` and `rune.finos.org` | The language, toolchain modules, code generators, editor support, licence |
| `regnosys.com/press/...rune-to-finos...` | The 24 June 2024 rename and contribution of the language |
| `isda.org/cdm` and `isda.org/isda-solutions-infohub/isda-digital-regulatory-reporting/` | Canonical CDM and DRR positioning, jurisdiction coverage dates, collateral initiatives |
| `drr-docs.isda.org` | DRR mechanics, glossary, governance and licensing of the DRR |
| `tradeheader.com` DRR FAQ (10 Feb 2026) | Implementation reality: mapping options, what DRR does not cover, who is contributing |

### 16.2 Related guides in this repository

| Guide | Why it matters here |
|---|---|
| `banking/nasdaq_calypso_guide.md` | The post-trade platform landscape the CDM would sit above; collateral and margin workflows |
| `banking/bian_banking_architecture_guide.md` | BIAN contrast in §11; capability-based architecture |
| `technology/architecture/bian_standard_guide.md` | BIAN's own governance and licence model, useful as a governance contrast to FINOS |
| `banking/tokenized_assets_guide.md` | The tokenisation domain the CDM tokenised-asset work targets |
| `banking/data_models_banking_insurance_guide.md` | Contains "Canonical Data Model" — a *different* concept from the CDM; do not conflate |
| `technology/ai_llm/llm_agent_use_cases_finance_guide.md` | The LLM-side mechanics referenced in §9 |
| `technology/ai_llm/ai_governance_framework_guide.md` | The governance frame for AI-assisted rule interpretation |

---

## 17. Closing Summary

The FINOS Common Domain Model is the most serious attempt the derivatives and securities-financing industry has made to give the post-trade stack a shared, executable semantics. It is genuinely open in its model and governance, genuinely used in production by a handful of significant institutions and one CCP, and genuinely immature in everything that depends on counterparties joining in. Its most valuable component — the reporting logic — is free but not open; its most complete code distribution is Java; its maintainer roster is vendor-heavy; and its adoption story is strongest exactly where a single institution acting alone can capture the benefit, which is regulatory reporting. That is not a criticism of the standard; it is a description of where to start with it.

The practical posture for a bank is boundary adoption with evidence gates: use the CDM where a regulator, CCP or trade repository already pulls you toward CDM-shaped data, build the mapping as a governed asset you own, treat the DRR as a free-to-use proprietary dependency rather than an open-source one, and refuse to fund post-trade, collateral or tokenisation ambitions on CDM without measured evidence. Everything else in the model — the composability, the embedded logic, the namespace discipline, the annual release cadence — becomes valuable only after that first boundary is in production.

If the CDM succeeds, it will not be because the model is beautiful. It will be because the industry finally agreed that representing the same trade five different ways is a cost with no return — and that is the case for the common domain.
