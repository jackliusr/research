# Migrating from Monolith to Microservices: The Migration Runbook Deep-Dive

*A comprehensive deep-dive on the migration problem and its solutions: the decision framework (should-you-migrate, and the modular-monolith alternative), the decomposition strategies (DDD bounded contexts, business capability), the migration patterns (strangler fig, branch by abstraction, parallel run, anti-corruption layer), the data migration (shared database to database-per-service), the challenges (distributed transactions, eventual consistency, testing, observability), the sequencing (analyze → plan → decompose → migrate → validate), the team topology (two-pizza teams), and a worked example — a Cymbal Bank's monolith migration.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Architecture / Migration (technology/)
> **Audience:** Solution architects, enterprise architects, platform engineers, technical leads, and delivery leads planning or running a monolith-to-microservices migration
> **Last Updated:** August 2026

**Cross-references:** [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the **integration-level** patterns — the strangler fig mechanics and the anti-corruption layer in §6, the EIP catalogue — the *integration* companion to this *migration* guide), [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) (the **target** microservices architecture for banking — the OBMA suite, its coexistence and §14 phased-migration playbook — the *destination* this guide's *journey* serves), [apache_seata_guide.md](apache_seata_guide.md) (distributed **transactions** — AT/TCC/SAGA/XA modes, the saga engine behind §5.1), [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable **orchestration** for sagas and multi-step migrations), [event_stream_processing_guide.md](event_stream_processing_guide.md) and [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the **event-driven** sync backbone for §4), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (messaging **reliability** — delivery semantics and the transactional outbox, the load-bearing pattern for dual-running data), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (CDC/ETL **tooling** for the data migration), [distributed_auth_guide.md](distributed_auth_guide.md) (identity across the new service estate), [ibm_as400_guide.md](ibm_as400_guide.md) (the **legacy platform** reality — the AS/400-class systems migrations must route around), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (the infrastructure the new estate needs), [late_arriving_data_guide.md](late_arriving_data_guide.md) (the data-reconciliation angle — lightly), [finops_guide.md](finops_guide.md) (the **cost** of running N services instead of one — lightly), [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) and [agentic_workflows_guide.md](agentic_workflows_guide.md) (the orchestration ideas migrating into workflow engines — lightly), plus the legacy cores this guide eventually retires: [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/temenos_guide.md](../banking/temenos_guide.md), [../banking/oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md), [../banking/chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md), [../banking/tafj_guide.md](../banking/tafj_guide.md), [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md) (the migration reality — the cores most migrations start from).

---

**How to read this guide.** This is the *migration-runbook* deep-dive of the architecture series: [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) covers *how to integrate* the legacy estate, [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) covers *what the target looks like*, and this guide covers *how to get there* — the decision, the decomposition, the patterns, the data, the challenges, the sequencing, the teams, and a worked example. A fast path through it: read **§1** (should you migrate at all — the monolith cases), **§2** (how to find the seams), **§3** (the patterns that carry the migration), and **§6** (the phase sequencing), then **§8** (the worked example that ties it together); treat **§4** (data) and **§5** (challenges) as the sections to read *before* you commit to a plan, **§7** as the organizational prerequisite, **§9** as the one-page summary. Verification status of every factual claim is in **§10**; terms are in **§11**. Cross-references point at the series' sibling guides throughout.

---

## Table of Contents

1. [The Decision Framework](#1-the-decision-framework)
2. [The Decomposition Strategies](#2-the-decomposition-strategies)
3. [The Migration Patterns](#3-the-migration-patterns)
4. [The Data Migration](#4-the-data-migration)
5. [The Challenges](#5-the-challenges)
6. [The Sequencing](#6-the-sequencing)
7. [The Team Topology](#7-the-team-topology)
8. [The Worked Example: A Cymbal Bank's Monolith Migration](#8-the-worked-example-a-cymbal-banks-monolith-migration)
9. [The Summary: The Strangler's Patient Journey](#9-the-summary-the-stranglers-patient-journey)
10. [Verification and Claims-Status](#10-verification-and-claims-status)
11. [Glossary](#11-glossary)
12. [References](#12-references)

---

## 1. The Decision Framework

### 1.1 The Monolith (verified — the cases where it wins)

Before any migration, the honest question: **is the monolith actually a problem?** The word "monolith" is not a diagnosis. A monolith is a single deployable unit that contains the application's logic and state — and for a large class of systems, that is the *correct* architecture. The industry's microservices enthusiasm has produced a strong counter-literature precisely because so many migrations were started for the wrong reasons. The monolith wins in these cases:

- **The team is small.** Conway's law (§7.1) says the architecture will mirror the organization. A team of 5–15 engineers building one deployable is coherent; splitting it into services would *add* coordination cost, not remove it.
- **The domain is small or tightly coupled.** If the business operations genuinely share data and transactions (a booking engine, a trading blotter, a settlement engine), the monolith's single database and in-process transactions are a *feature* — the distributed version of the same system would pay distributed-transaction costs (§5.1) for no benefit.
- **The scale problem is not a scaling problem.** If the load is modest, or the bottleneck is a single well-understood component (a database, a batch job), the fix is to scale *that* — not to decompose the whole system.
- **Delivery speed is already good.** The strongest argument for microservices is independent deployability: team A ships without waiting for team B. If a monolith's release pipeline already ships daily with one team, the migration would slow delivery for years before it speeds it up.
- **The regulatory and audit context favors fewer moving parts.** In banking (this guide's home context), a single auditable system with one set of controls is operationally simpler than forty services with forty pipelines — simplicity has a compliance value that architecture diagrams rarely price.

The classic symptom list that *does* justify migration: **the deployment is slow and risky** (a release touches everything), **teams block each other** (merge conflicts, shared test environments, release trains), **scaling is all-or-nothing** (the whole monolith scales because one component is hot), **the tech stack is frozen** (nobody can upgrade a library because "it might break the core"), and **the codebase has lost its seams** (no clear boundaries remain — the domain model is a tangle). Notice the pattern: every one of these is a *team-and-process* symptom as much as a technical one. That is why §7 (team topology) comes before §6 (sequencing) in the runbook: the organization must be able to absorb the architecture it is building.

**The monolith symptom table** — the working diagnostic, symptom by symptom:

| Symptom | What it actually means | Does it justify migration? |
|---------|------------------------|----------------------------|
| Deployments are slow, risky, quarterly | The release pipeline couples everything; change lead time is the pain | Yes — if independent deployability is the goal (the core microservices benefit) |
| Teams constantly block each other | Shared code, shared environments, release trains; merge conflicts are routine | Yes — the organizational case; see Conway (§7.1) |
| Scaling is all-or-nothing | One hot component forces the whole monolith to scale (and pay for it) | Yes — if the hot component is separable; otherwise scale the component, not the estate |
| The stack is frozen | No dependency can move without a cross-cutting risk assessment | Partially — refactoring the monolith (§1.3) may unstick this without a migration |
| The domain model is a tangle | No seams remain; nobody can say where one capability ends | Yes — but the *seams* must be recoverable (§2); if they are gone, the migration starts with archaeology |
| The business wants independent scaling/uptime per product | Payments must scale without trade finance, and fail without taking it down | Yes — this is the case no modular monolith answers (§1.3) |
| The team is small and shipping fine | None of the above is true | No — the monolith is working; see the alternatives |

### 1.2 The Should-You-Migrate Decision

The decision framework reduces to **three questions, in order**:

1. **Is the monolith the bottleneck?** Measure it: release cadence, change lead time, deploy failure rate, time-to-market for a new channel, on-call pain. If the monolith is not measurably the constraint, there is nothing to migrate *for*.
2. **Can we get the benefit without decomposing?** The alternatives in §1.3 — especially the modular monolith — capture most of the "independent teams, independent deployability" benefit at a fraction of the cost. A migration is justified only when the modular path is genuinely insufficient (independent *scaling*, independent *failure* domains, heterogeneous *technology* needs).
3. **Can the organization survive the migration?** The migration is a multi-year program that *increases* operational complexity for most of its life (§5, §6). It needs executive sponsorship, a platform team, and a delivery model that tolerates months without user-visible features. If the organization cannot fund the journey, the destination is irrelevant.

The honest answer to "should we migrate?" is often **"not yet"** — and a well-run program treats "modular monolith first, migrate later" as a strategy, not a failure. The reverse error — migrating because microservices are fashionable — is the single most expensive mistake in the catalogue (§10 flags the failure statistics).

**The costs the decision must price.** The migration's benefits are the §1.4 table's right column; its costs are usually under-priced because they are *distributed* — paid by different teams over years:

| Cost | What it is | Who pays |
|------|------------|----------|
| **The distributed-systems tax** | Every in-process call becomes a network call: latency, serialization, partial failure, retries, timeouts — forever | The domain teams, every day, for the life of the estate (§5.1, §5.5) |
| **The data tax** | One transactional database becomes many eventually-consistent stores: sagas, outboxes, reconciliation, drift investigation | The domain teams and operations (§4, §5.2) |
| **The platform cost** | CI/CD, containers, the event backbone, the gateway, observability — N environments × N services | The platform team and the budget ([finops_guide.md](finops_guide.md)) |
| **The operational cost** | N services to monitor, patch, scale, and page on instead of one | SRE/operations, permanently (§5.4) |
| **The organizational cost** | Multi-team coordination, contract governance, the migration program itself (years of double-running) | The whole organization, during the migration (§6, §7) |
| **The migration cost** | The facade, the ACL, the sync machinery, the reconciliation, the cutover rehearsals — all decommissioned at the end | The migration program's budget — the most under-priced line in most business cases |

The pricing rule: **the business case must include the distributed-systems tax, the platform, the operations, and the migration machinery — not just the new services' build cost.** A migration justified on build cost alone is a migration that will be re-litigated in year two.

### 1.3 The Alternatives (verified — the modular monolith)

The migration decision is not binary (monolith vs microservices). The serious alternatives:

- **The modular monolith (verified — the term).** A single deployable unit with *strict internal module boundaries*: modules communicate through well-defined interfaces, own their data within the process, and are kept honest by architecture tests (no cross-module imports, no shared tables). The term was popularized by **Simon Brown's talk "Modular Monoliths" (GOTO Berlin, 2018)** — the *term's* origin is often credited to that talk, though the concept and the phrase predate it (flagged in §10). The modular monolith delivers team autonomy *within* one deployable, keeps transactions in-process, and can be decomposed *later* — the module boundaries become the future service boundaries. Its virtues have made it the default recommendation of the anti-hype literature (Stefan Tilkov's "Don't jump to microservices" talk, 2017, made the same argument in the microservices boom's early years).
- **Keep + refactor.** Sometimes the monolith's problem is internal: extract packages, enforce boundaries, clean the data model, improve the pipeline. No distributed system is involved. This is a *refactoring* program, not a *migration* program.
- **Package/SaaS replacement.** For generic capabilities (CRM, payments hub, regulatory reporting), buying a product and integrating it may beat building a service. This is the *integrate, don't replace* philosophy of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) applied to the new estate.
- **Hybrid: modular monolith core + satellite services.** Keep the transactional core as a modular monolith; extract the *peripheral* capabilities (channels, reporting, notifications, analytics) as services. Most real-world banking estates — including the OBMA-style targets in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) — end up here: a small number of services around a strong core, not a fine-grained mesh.

The decision rule: **modular monolith first unless you can name the specific capability that requires a distributed system** — independent scaling of one hot path, an isolated failure domain for one component, a team boundary that cannot be drawn inside one codebase, or a technology requirement (a different runtime, a different data store) that one process cannot host.

### 1.4 The Decision Table

| Factor | The monolith wins when... | The microservices win when... |
|--------|---------------------------|------------------------------|
| **Team size** | One team (≤ ~15 engineers) owns the whole system | Multiple teams (2+) must ship independently, on their own cadence |
| **Domain coupling** | Operations share transactions and data naturally (booking, settlement, trading) | Capabilities are separable with clear, stable boundaries (party, payments, trade, reporting) |
| **Scaling** | Load is modest or the hot spot is one known component | One component needs independent scaling (or isolation) that the monolith cannot provide |
| **Release cadence** | Daily releases are achievable and low-risk | Teams are blocked by each other's releases; deployment is slow, risky, or frozen |
| **Failure isolation** | A single outage is acceptable; blast radius is already managed | One hot component's failures must not take down the whole system |
| **Technology** | One runtime/stack is fine; upgrades are tractable | Parts need different runtimes, data stores, or upgrade cycles |
| **Organization** | The org is not ready for multi-team ownership | The org has platform engineering, SRE, and product teams that can own services end-to-end |
| **Compliance/audit** | Simplicity and single-control audit outweigh flexibility | Regulatory needs (e.g., per-business-line reporting) map to service boundaries |
| **Cost** | One deployment's infra and ops are cheap; N services would multiply them | The business value of independent delivery exceeds the multiplied platform/ops cost |

The table's lesson: **microservices are an organizational pattern as much as a technical one.** If the rows point at "monolith", the professional move is to say so — the alternatives in §1.3 exist precisely to give that answer a constructive form.

---

## 2. The Decomposition Strategies

Once the decision is made, the first technical question: **where are the seams?** Decomposition is the act of finding the boundaries along which the monolith can be split — and every pattern in §3 and every phase in §6 depends on those boundaries being *right*, because a bad boundary is permanent (it becomes an API contract, a data ownership line, and a team boundary). There are two primary strategies, and a supporting cast.

### 2.1 The DDD Bounded Contexts (verified — Evans, 2003)

**Origin:** *Domain-Driven Design: Tackling Complexity in the Heart of Software*, **Eric Evans, Addison-Wesley, 2003** — the book that gave the migration discipline its single most important concept: the **bounded context**.

**The idea.** A large domain is not one model — it is many models, each valid *inside a boundary*. The **bounded context** is that boundary: the explicit line inside which a particular domain model, its terms, and its rules apply. Within a context, the team speaks a **ubiquitous language** (Evans's term — one vocabulary shared by business and developers, used in code, documents, and meetings). Across contexts, the same word can mean different things: to the *payments* context, a "customer" is a set of routing instructions; to the *CRM* context, the same customer is a relationship history; to the *risk* context, a legal-entity exposure. DDD's answer is not to unify these models — that is the road to the "god object" — but to **keep the contexts separate and define the translations between them** (the context map: the ACL of §3.4, the published language, the shared kernel).

**Why it is the primary decomposition strategy.** The bounded context is the *only* decomposition criterion that is simultaneously: (a) **domain-driven** — derived from the business, not from the code; (b) **data-driven** — each context owns its data, which is exactly what database-per-service (§4.1) requires; and (c) **team-shaped** — a context is roughly the size of one team's cognitive capacity. The canonical migration move: **discover the bounded contexts in the monolith, then make each context a service.** The seams were always there — the monolith blurred them.

**How to find them in a monolith:** start from the *business* (process maps, product definitions, org structure — who owns what vocabulary), then verify against the *code* (aggregate roots, transaction boundaries, table ownership — where does one context's data end and another's begin?). The classic warning sign of a context boundary is **translation**: where the codebase converts one representation of a concept into another (a DTO mapping, a status-code translation, a duplicated entity with a different shape), a bounded context boundary is trying to exist. Evans's own guidance applies: the **aggregate** — the cluster of objects treated as one unit for change — is the natural service/ownership unit, and the **repository** is the natural data-ownership unit.

### 2.2 The Business Capability (verified)

**The idea.** A **business capability** is what the business *does*, expressed independently of how it is done: *onboard customers, originate loans, process payments, manage limits, report to regulators*. The capability map (an enterprise-architecture artifact — see [architecture/enterprise_architecture_guide.md](architecture/enterprise_architecture_guide.md) and the TOGAF coverage in [architecture/togaf_guide.md](architecture/togaf_guide.md) for the framework context) decomposes the business into a stable, technology-free hierarchy. **Decompose along capabilities**: each capability becomes a candidate service, because a capability is stable (the business has always onboarded customers) while the implementation changes (the onboarding *system* gets replaced every decade).

**Why it complements DDD.** Capability mapping works top-down (business → capabilities); DDD works bottom-up (domain → contexts). Where they agree, the boundary is robust — the business *and* the domain model both draw the line there. Where they disagree, the disagreement itself is information: either the capability is not a real seam (DDD says the model is entangled), or the team has drawn artificial boundaries (the capability says the business operates independently). The two strategies are usually combined in practice: **capability map for the portfolio-level picture (what to extract, in what order), bounded contexts for the code-level seams (how to extract it).**

**The three types of subdomain** (also from the DDD tradition — Evans and the DDD literature): **core** (the differentiating capability — for a bank, the pricing, risk, and product logic; never outsource, never neglect), **supporting** (needed but not differentiating — standard, often bought or shared), and **generic** (commodity — authentication, notifications, document storage; buy or use a platform). The migration program should spend its decomposition effort on core subdomains and treat generic ones as off-the-shelf integrations.

### 2.3 The Supporting Strategies

Three further decomposition lenses, each useful in specific situations:

- **The team-first decomposition (Conway's law, §7.1).** Draw the *organizational* boundaries first — the teams the business needs — and let the service boundaries follow. This is the inverse of the common failure mode (services drawn by a committee, then teams awkwardly mapped onto them). Melvin Conway's 1968 observation ("organizations design systems that mirror their own communication structure") is the load-bearing law of the whole discipline.
- **The volatility decomposition.** Split along *change frequency*: the components that change monthly should not be welded to components that change yearly. This is a pragmatic, code-level lens (change-frequency analysis on the monolith's modules) that often exposes seams DDD would also find.
- **The data decomposition.** Split along *data ownership*: which tables does each business operation truly own? Where two operations genuinely share a table with no aggregate boundary, that is a *transactional* coupling — and the honest finding is usually "this part of the monolith is not ready to split" (see §4.1, where the data work is the hardest part of the migration).

### 2.4 The Decomposition Table

| Strategy | Approach | Notes |
|----------|----------|-------|
| **DDD bounded contexts** (Evans, 2003) | Discover the domain models and their boundaries; give each context a ubiquitous language; make each context a service; draw the context map with explicit translations | The primary strategy — domain-driven, data-driven, team-shaped. The translation points between contexts are exactly where the ACL (§3.4) goes |
| **Business capability** | Map the business top-down into stable capabilities; extract one capability at a time as a service | Best for portfolio-level *ordering* (what to extract when); pairs with DDD for the code-level seams. Core subdomains get the investment; generic subdomains get bought |
| **Team-first (Conway's law)** | Design the team topology first; draw service boundaries to match team ownership | The organizational prerequisite (§7). Boundaries that fight the org chart will be redrawn by the org |
| **Volatility** | Split along change frequency | Pragmatic lens; exposes seams DDD would also find; good for quick wins |
| **Data ownership** | Split along table/aggregate ownership; keep transactional couplings together | The honest lens: where the data is entangled, the code is not ready. Feeds §4's database decomposition |

### 2.5 The Context Map: The Relationships Between Contexts (verified — Evans, 2003)

The bounded contexts are not islands: they *relate* to each other, and the migration must design those relationships explicitly. Evans's **context map** — the catalogue of relationship types between contexts — is the blueprint for how the future services will cooperate (and it is where the migration's integration patterns plug in):

| Relationship | What it means | Where it shows up in a migration |
|--------------|---------------|-----------------------------------|
| **Partnership** | Two contexts cooperate on a shared workflow; changes are coordinated | The typical relationship between the *payments* and *accounts* services in a bank's estate — the saga boundary (§5.1) |
| **Shared kernel** | A small, shared model (usually code) both contexts use | The shared reference data (currencies, calendars, product codes) both services consume; keep it *small* or it re-couples them |
| **Customer–supplier** | One context (the supplier) serves the other (the customer); the customer's needs drive the supplier's plan | The natural relationship between *party* (supplier of customer data) and every other service — the event/API contract (§4.3) |
| **Conformist** | The downstream context accepts the upstream's model as-is, with no translation | The pragmatic choice when the upstream (often the legacy core!) cannot change — but it leaks the legacy model, which is exactly what the ACL exists to prevent |
| **Anti-corruption layer** | The downstream context *translates* the upstream's model at the boundary | The default relationship between the new services and the monolith (§3.4) — the migration's single most important context-map entry |
| **Published language** | A shared, formal information model (e.g., ISO 20022) both sides translate to | The banking-standard move: translate the legacy model to ISO 20022 at the ACL, so services never speak copybook |
| **Separate ways** | No relationship at all — the contexts are integrated by nothing but the data they both happen to touch | The honest entry for contexts that only *look* coupled; documenting it prevents fake seams from being drawn |

The context-map rule: **draw the context map before the service boundaries** — it is the migration's dependency diagram, and every line on it (a partnership, a customer–supplier contract, an ACL) becomes an integration pattern in §3 and a data-sync requirement in §4.

### 2.6 The Decomposition Anti-Patterns

The decomposition literature and this series' banking guides are consistent about the ways decomposition goes wrong — worth naming so the runbook can recognize them:

| Anti-pattern | The symptom | The honest alternative |
|--------------|-------------|------------------------|
| **The layer split** | Services carved by technical layer: one "database service", one "business-logic service", one "UI service" | Decompose by *domain* (contexts, capabilities), not by *layer* — a layer split moves the coupling into the network and keeps it there |
| **The nano-service** | Services of one method each; every business operation is a choreography across dozens of calls | One service per bounded context, sized to a team's cognitive capacity (§2.4); fine-grained is not the goal, *independent deployability* is |
| **The god-service** | One context ("customer", "reference data") absorbs everything because it is the easy seam | A context is a boundary, not a dumping ground; the context map (§2.5) keeps the big contexts honest |
| **The fake seam** | A boundary drawn where the code is entangled (shared tables, shared transactions) — "we'll fix it during extraction" | The data-ownership lens (§2.3, §4.1) is the truth-teller: where the data is entangled, the boundary does not exist yet |
| **The committee boundary** | Services designed by an architecture committee nobody owns; the teams that must run them had no say | Team-first decomposition (§2.3): the teams that will own the service draw its boundary |
| **The boundary-by-database** | The split follows the existing database schema (tables → services) instead of the domain | The schema is the *legacy's* shape, not the domain's; decompose by the domain model, then migrate the data to match (§4.1) |

The anti-pattern rule: **if a boundary looks easy but the data underneath it is entangled, it is a fake seam — the decomposition phase exists to find that out before the extraction phase pays for it.**

The decomposition rule: **the boundary must be simultaneously a domain boundary (DDD), a business boundary (capability), and an organizational boundary (Conway) — if it is only one of the three, it will move.**

---
## 3. The Migration Patterns

With the seams found, the migration itself is carried by a small set of patterns. These are the *migration-runbook* patterns — they answer "how do we replace a running system with a new one, without stopping the business?" The integration-level mechanics of the first and last of them are covered in depth in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6; this section treats them as the runbook uses them.

### 3.1 The Strangler Fig (verified — Fowler, 2004)

**Origin:** Martin Fowler, bliki post **"Strangler Fig Application"**, **2004** — the migration pattern, named for the vine that grows around a host tree until the vine *is* the tree. Verified: Fowler's bliki, 2004 (later retitled; page updated as recently as 2024).

**The runbook mechanics (transform → coexist → eliminate), from the migration program's point of view:**

1. **Transform — build the facade.** A routing layer sits in front of the monolith, so that *all* traffic flows through one controllable point. From this moment, the migration has a steering wheel: the facade decides, per function, whether the old system or the new service answers.
2. **Coexist — strangle function by function.** Each extracted capability is routed at the facade from the monolith to its new service. Both run side by side, synchronized (§4), with the facade as the single switch. Each switched function is value delivered and risk retired. This is the *long* phase — in banking it runs for years.
3. **Eliminate — cut the host away.** When the last function is strangled, the monolith is retired. (In banking, "retired" often means *frozen and read-only* for regulatory look-back — the pattern did its job; see §8.)

**Why it is the definitive migration pattern:** it makes the migration **incremental, reversible, and business-visible**. No big-bang cutover (the statistically worst way to replace a core); every slice is small and testable; value lands from the first function; and the facade means the migration can be paused, rolled back per function, or abandoned at any point with the monolith still intact. It is the pattern behind most successful core-banking modernizations — see the phased-migration playbook of the target architecture in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14 (coexistence → phased waves → parallel run → cutover → decommission is the OBMA-shaped strangler), and the core-replacement landscape in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) and [../banking/chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md).

**The facade in practice.** The facade is the migration's control surface, and its design repays the investment: it is an **API gateway / routing layer** in front of the monolith (and, as waves land, in front of the services too), through which *all* channel traffic flows. It carries the routing table (function → old system or new service), the **ACL** (translating between the legacy model and the new domain, §3.4), and the migration's observability (traffic splits, error deltas, routing health — §5.4). It is deliberately *dumb* about business logic — it routes and translates, it does not decide — because the moment the facade grows business logic, it becomes a new monolith (the "facade becomes the bottleneck" risk in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.4). In banking, the facade is also the natural security boundary: the place where authentication, authorization, and rate limiting concentrate ([distributed_auth_guide.md](distributed_auth_guide.md), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md)) — the identity dimension of the migration's first phase.

### 3.2 The Branch by Abstraction (verified — Hammant, 2013; flagged nuance)

**Origin:** **Paul Hammant**, blog post **"Introducing Branch By Abstraction"** (October 2013), who coined the term while arguing for trunk-based development — and who credits **Stacy Curl** with originally coming up with the idea (per Fowler's bliki). Martin Fowler's bliki article **"BranchByAbstraction"** followed in **January 2014**. Verified via Hammant's blog, trunkbaseddevelopment.com, and Fowler's bliki; the Stacy Curl attribution nuance is flagged in §10.

**The pattern.** Replace a component *in place* without a long-lived code branch, in five steps:

1. **Introduce an abstraction** (an interface) at the point where the component is used.
2. **Make the old implementation implement the abstraction** — behavior unchanged.
3. **Write the new implementation** behind the same interface, in parallel.
4. **Switch the callers** to the new implementation (feature-flag style, one caller at a time).
5. **Delete the old implementation** once nothing references it.

**Why the runbook needs it:** the strangler fig operates at the *deployment* level (routing traffic between systems); branch by abstraction operates at the *code* level (replacing a module inside a system). The two compose: before a capability can be routed to a new service, the monolith often needs its internal seams refactored — and branch by abstraction is how that refactoring happens *without* freezing the monolith's development for months on a migration branch. The abstraction is the seam that later becomes the service boundary. The flagged nuance: it is a *refactoring technique* with a *migration flavor* — it does not by itself produce a service; it produces the clean seam the service extraction then uses.

### 3.3 The Parallel Run (verified — industry practice)

**The pattern.** For a period, run the **old and new systems simultaneously for the same workload** — the new system processes real traffic in shadow or dual mode, its outputs are **reconciled** against the old system's outputs, and only when the reconciliation is clean does the facade (or the cutover) switch production traffic. Parallel run is the *validation engine* of the migration: it is how a bank proves that the new payments service posts interest the same way the old core did, before trusting it with real money.

**The mechanics:** shadow-mode (the new system *observes* traffic and compares outputs, with no effect on customers — the safest, cheapest form), dual-write (both systems process the transaction; the old system's result remains authoritative while reconciliation runs — the standard banking form), and full parallel run with cutover (both live; the new system becomes authoritative per book, per the OBMA playbook in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14.4). The reconciliation is the pattern's heart: balance matching, position matching, exception investigation, sign-off per migrated book.

**Why the runbook needs it:** the strangler fig proves *functionality* (the new service answers the request); parallel run proves *correctness* (the new service answers *the same way*). For a bank's ledger, interest accrual, and risk positions, the two are very different claims. The cost is real — dual processing, dual infrastructure, reconciliation teams — which is why parallel run is used *per slice* (each migrated capability runs parallel briefly, then the old path is cut), not for the whole estate for the whole migration.

### 3.4 The Anti-Corruption Layer (verified — Fowler, 2004; see legacy_integration_patterns_guide.md)

**Origin:** Martin Fowler, bliki post **"AntiCorruptionLayer"** (2004, widely cited — the exact publish date is flagged in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §11), arising from DDD (Evans, 2003): each bounded context has its own model, and if one system's model leaks into another's, the receiving model gets *corrupted*.

**The pattern:** a dedicated **translation layer** between the monolith and each new service, so that the legacy model (copybooks, status codes, five address formats) never leaks into the new domain model. The ACL translates at the boundary, isolates the new service from the legacy system's quirks, and owns the mapping in one place — so that when the monolith finally dies, the migration deletes the ACL and the new service is untouched. In banking it is implemented as the facade/API layer in front of the core: modern JSON/ISO 20022 in, COBOL copybook calls out. The full mechanics — and why it is the *structural* realization of the Message Translator pattern — are in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.2; the runbook point here is **the ACL is the strangler's enabler**: the facade (§3.1) routes, the ACL translates, and the new services stay clean while the old system is being strangled. A migration without an ACL builds its new estate on the legacy model — and inherits the corruption it was supposed to escape.

### 3.5 The Supporting Cast

The four patterns above are joined by enabling patterns the runbook uses constantly:

- **The transactional outbox** — the dual-write safety net: when a service writes to its database *and* must publish an event (or the migration must sync old and new systems), the write and the event go into one local transaction (the outbox table), and a relay publishes the events. This is the load-bearing pattern for §4's data sync; full mechanics in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10.
- **The event-driven integration** — the sync backbone between monolith and services (and between services): events, not RPC, carry the state changes across the boundary. See [event_stream_processing_guide.md](event_stream_processing_guide.md) and [kafka_alternatives_guide.md](kafka_alternatives_guide.md).
- **CQRS** (Command Query Responsibility Segregation — credited to Greg Young, circa 2010; Fowler bliki July 2011; date flagged per [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §11) — used selectively where read models must differ from write models (reporting during migration is the classic case).
- **The facade/adapters** — the enabling layer under everything (GoF, 1994), covered in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.3.

### 3.6 The Patterns Table

| Pattern | Mechanism | Use case |
|---------|-----------|----------|
| **Strangler fig** (Fowler, 2004) | Facade routes traffic function-by-function to new implementations; old system retired when empty | The overall migration strategy — replacing a running system incrementally, reversibly, with business value per slice |
| **Branch by abstraction** (Hammant, 2013) | Interface + parallel implementation + per-caller switch + delete old | Replacing a component *inside* the monolith without freezing development on a long-lived branch; creating the seams for extraction |
| **Parallel run** | Old and new process the same workload; reconcile outputs; switch on clean reconciliation | Proving the new service is *correct* (not just functional) before cutover — ledger, interest, positions in banking |
| **Anti-corruption layer** (Fowler, 2004) | Dedicated translation layer at the boundary; mapping owned in one place | Every boundary between the legacy model and new domains; keeps the new estate clean while the old system lives |
| **Transactional outbox** | Local transaction writes DB + event; relay publishes | Dual-write safety: syncing monolith and services, publishing events reliably; see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10 |
| **Event-driven integration** | State changes flow as events on a broker/stream | The sync backbone during coexistence and the service-to-service fabric after; see [event_stream_processing_guide.md](event_stream_processing_guide.md) |

The pattern stack in one sentence: *wrap and translate the monolith (facade + ACL), grow the new estate function by function through the facade (strangler), refactor the seams in place (branch by abstraction), prove correctness against the old system (parallel run), and carry the data with events and outboxes (event-driven + outbox) — until the monolith is empty and can be cut away.*

### 3.7 The Adoption Reality (flagged — the pattern adoption)

An honest footnote on the pattern catalogue, per this series' verification discipline. The *qualitative* claims above — that the strangler fig is the dominant migration strategy for large legacy estates, that the ACL is the standard DDD integration boundary, that parallel run is the norm in banking cutovers — are well supported by practitioner literature and by this repository's own guides ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6, the OBMA §14 playbook in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md), the core-replacement landscape in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)). **Flagged:** no authoritative *adoption statistics* (what share of enterprises use the strangler fig; what share of migrations use parallel run) could be verified in this run — the widely-circulated microservices-adoption and migration-failure numbers are inconsistent across sources and are deliberately not cited here (see §10 for the same flag on the Gartner prediction). The discipline's guidance stands on the pattern mechanics, which are verified, not on the statistics, which are not. The pattern adoption rule: **adopt patterns by their mechanics, not by their marketing** — the strangler fig is used because it is incremental and reversible, not because a survey says so.

---

## 4. The Data Migration

The patterns in §3 move *traffic*; the hardest part of any migration moves *data*. The data layer is where monolith-to-microservices migrations most often stall or silently corrupt the business — and it is the section most runbooks under-plan. In banking, the data *is* the product: the ledger, the customer, the positions.

### 4.1 The Database Decomposition (verified — shared DB to database-per-service)

**The principle.** The microservices architecture's data rule — **database per service**: each service owns its data, and no other service reaches into it; ownership is enforced at the schema and often at the network level. The source of the principle is the canonical microservices literature — **Sam Newman, *Building Microservices*, O'Reilly, 2015** — where data ownership is presented as the defining property of the style (the "shared database" is its anti-pattern: it re-couples everything the services were meant to decouple). The target architecture in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) is built on exactly this rule (per-service databases, no cross-service schema access).

**The migration, step by step.** The monolith has *one* database; the migration must turn it into *many* — and the direction of the work is the reverse of intuition: **you cannot split the database first; you split the data ownership, then the database.** The sequence:

1. **Map table ownership to bounded contexts (§2.1).** Every table (and every column of every joined table) is assigned to exactly one context. The aggregate roots define the ownership units. This is a *discovery* exercise that routinely takes months and exposes how entangled the monolith actually is.
2. **Deal with the shared tables honestly.** Tables genuinely shared across contexts — the classic `CUSTOMER` table that five modules read — are the migration's crux. Options: (a) the table belongs to one context, and the others get its data via *events/APIs* (the clean answer, but it requires the consumers to change); (b) the data is *copied* per context with the owner publishing changes (the pragmatic answer during coexistence — the "five spellings of the customer" problem starts here, so the copy must be governed by a canonical owner); (c) the table is *not* actually shared — the coupling is accidental (two contexts each use a few columns — split the table). What does not work: leaving one shared database "temporarily" — it becomes permanent, and the migration's whole point is defeated.
3. **Split the physical database** as each service goes live: extract the service's schema into its own database, backfill it, wire the service to it, and keep the sync running (§4.3) until the old path is cut.
4. **Enforce the boundary** — schema grants, service-level access only, and (in banking) the change-control that says "no one modifies another service's data" — otherwise the entanglement regrows.

**The dual-write problem** is the technical heart of this phase: during coexistence, the old system and the new service both process the same business event, and both write to their own stores. Writing to two stores from one code path — then the second write fails — is the classic distributed-data failure mode; the transactional outbox (§3.5, and [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10) is the standard fix: one local transaction per store, with events carrying the change across.

### 4.2 The Data Migration Mechanics

Beyond decomposition, the *data itself* must move — historically, in banking, *twice*: once into the new service's database at extraction, and once (the real migration) when the monolith is finally retired. The mechanics:

- **The initial load (backfill).** Extract the service's data from the monolith's database — via ETL/ELT tooling or CDC (change data capture; see [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) for the tooling landscape) — transform it to the service's model (this is where the ACL's mappings do double duty), and load it. The load must be *replayable* (the source keeps changing while you load) and *verifiable* (row counts, control totals, hash comparisons — reconciliation from the first byte).
- **The incremental sync.** Between backfill and cutover, changes flow from the source of record to the new store — via events (§4.3), CDC, or replay. The sync is the data-level twin of parallel run (§3.3): it exists so the new store can be proven equal to the old at any instant.
- **The historical-data decision.** Full history is rarely migrated. The banking standard (documented in the OBMA playbook, [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14.4): migrate a defined window (e.g., 24 months of transactions) into the new system, and keep older history queryable from the legacy system or an archive (read-only, retained for regulatory look-back — the *freeze and archive* pattern of §3.1's eliminate phase).
- **The reference data.** Products, rates, GL mappings, fee schedules — re-created in the new system's product/parameter model, mapped from the old. Reference-data drift between the two systems during coexistence is a chronic reconciliation noise source.
- **The cutover and rollback.** Per book: freeze the legacy writes, migrate the residual, switch the system of record, archive. Cutover windows are weekends, with rollback plans that assume the new system will be switched off and the old one switched back on — which is exactly why the sync must have been running until the last minute, so rollback is lossless.

**The cutover weekend playbook** — the shape of a well-run cutover (the sequence the migration rehearses until boring, §5.3):

1. **Pre-freeze (Friday close-of-business):** the sync is current; the reconciliation for the book is clean; the freeze notice is issued; the rollback plan is re-read by the team that will execute it.
2. **Freeze (Friday night):** legacy writes for the book stop; the residual delta is captured and applied to the new store; the new store is verified against the frozen legacy store (control totals, hash comparisons).
3. **Switch (Saturday):** the facade routes the book's traffic to the service; read-only traffic first, then low-risk writes, then the full book — each step verified, each step reversible.
4. **Verify (Saturday–Sunday):** the book's business operations run against the service; the reconciliation compares the service's store against the frozen legacy snapshot; exceptions investigated *before* the weekend ends.
5. **Unfreeze (Monday open-of-business):** the book is live on the service; the legacy store is archived (read-only, queryable for look-back); the rollback plan is *not* deleted — it is filed with the incident runbooks, because the first weeks on the new system are still the migration's risk window.

The cutover rule: **a cutover that cannot be rolled back is not a cutover, it is a bet — the sync running to the last minute and the rehearsed rollback are what make the weekend a controlled switch instead of a gamble.**

### 4.3 The Event-Driven Patterns (verified)

The sync fabric of the migration — and the permanent fabric of the target estate — is **event-driven**: state changes published as events, consumed by whoever needs the data. The runbook uses the event backbone in three roles:

1. **Coexistence sync** — the monolith (or its ACL facade) publishes domain events for every change; the new services consume and materialize their own views. The new estate is built on the old system's events before it is built on the old system's APIs.
2. **Service-to-service fabric** — after extraction, services publish and subscribe; no service reaches into another's database (the database-per-service rule of §4.1 enforced by construction).
3. **Read-model/analytics** — events feed the reporting and analytics stores, so the migration does not have to keep one giant reporting database alive "just for queries."

The supporting patterns — **event sourcing** (the event log as the system of record; used selectively, it is a *new* data model and should not be adopted *during* a migration), **CQRS** (separate read/write models; useful for migration-era reporting), and the **transactional outbox** (reliable publishing) — are covered in the series: [event_stream_processing_guide.md](event_stream_processing_guide.md) for the stream-processing mechanics and fault tolerance, [kafka_alternatives_guide.md](kafka_alternatives_guide.md) for the broker choice, [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10 for the outbox, and [late_arriving_data_guide.md](late_arriving_data_guide.md) for the reconciliation reality when events arrive late or out of order (they will).

### 4.4 The Data Table

| Pattern | Approach | Risks |
|---------|----------|-------|
| **Database per service** (Newman, 2015) | Each service owns its schema; no cross-service access; ownership enforced | The correct end-state; reached only by the ownership-first sequence of §4.1 — done first, it breaks every consumer |
| **Shared database (during transition)** | One database, ownership governed by convention | Re-couples services; "temporary" sharing becomes permanent; the entanglement the migration was meant to end |
| **Table split** | Split accidentally-shared tables by context ownership | Column-level dependencies missed; joins silently broken; data-quality drift between copies |
| **Backfill + incremental sync** | Initial load (ETL/CDC) + continuous sync via events | Replay/verification failures; sync lag at cutover; reference-data drift |
| **Transactional outbox** | Local transaction writes DB + event; relay publishes | Relay failure = stuck events; ordering; duplicate delivery (idempotent consumers required) |
| **Event-driven sync** | Events carry state changes across the boundary | Late/duplicated/out-of-order events (see [late_arriving_data_guide.md](late_arriving_data_guide.md)); schema evolution of events; the broker becomes critical infrastructure |
| **Historical window + archive** | Migrate N months; freeze and archive the rest | Regulatory look-back requirements; archive queryability; legal holds |
| **Reconciliation** | Compare old and new stores continuously; investigate exceptions | The reconciliation *itself* becomes a project; exception backlog drowns the team; sign-off culture never forms |

### 4.5 The Data Consistency States (verified)

The migration's data design is really a decision about **which consistency state each piece of data lives in** — and the honest program decides this per aggregate, explicitly:

| State | What it means | Where it is acceptable |
|-------|---------------|------------------------|
| **Strongly consistent** | One store, one transaction; reads see the latest write | Inside a single service's database — the *atomic core* of §5.1 (settlement, posting, booking) |
| **Eventually consistent (bounded)** | Multiple stores converge via events/sync; the window is seconds, measured and monitored | Between services and their materialized views; between the monolith and the new estate during coexistence — the normal state of §4.3's sync |
| **Eventually consistent (tolerant)** | Convergence can take hours/days; the business accepts it | Reporting and analytics (overnight data), historical archives, reference-data caches — the data that never claimed to be real-time |
| **Reconciled** | Two stores are *proven* equal by comparison, continuously | The monolith vs service during parallel run (§3.3) — a distinct state: not "eventually equal" but "verified equal, with evidence" |

The consistency rule: **write down, per aggregate, which state it lives in — a data migration that leaves this implicit leaves the business to discover the answer in production.** The reconciliation discipline of §4.4 and the saga machinery of §5.1 are the two mechanisms that make the eventual states safe; see [apache_seata_guide.md](apache_seata_guide.md) and [late_arriving_data_guide.md](late_arriving_data_guide.md) for their production realities.

### 4.6 The Data-Ownership Enforcement

The database-per-service principle (§4.1) is a *rule*, and rules need enforcement — because the entanglement the migration dismantles will quietly regrow. The enforcement stack, in order of strength:

- **The schema boundary.** Each service's schema is owned; cross-service table access is technically impossible (separate databases, separate credentials, no shared grants). This is the *design* enforcement — the strongest, because it cannot be bypassed by convention.
- **The API/data contract.** All cross-service data access goes through the service's API or its events; the contract is versioned and governed (the customer–supplier relationship of §2.5). Contract changes are negotiated, not unilateral.
- **The change-control process.** In banking, the operational layer: schema-change review, data-migration runbooks, sign-off for any change touching another service's data. Slow, but it is the compliance translation of the ownership rule.
- **The architectural guardrails.** Automated checks in CI (no cross-module imports, no foreign-table references, no shared-entity reuse) — the same "architecture tests" that keep a modular monolith honest (§1.3) applied to the distributed estate.

The enforcement rule: **the ownership rule is only as real as the enforcement — a shared database "temporarily" is a shared database permanently, and a service that reads another's tables has already re-created the monolith.**

The data rule: **migrate ownership before data, sync with events, prove equality with reconciliation, and cut over per book with a lossless rollback — the data migration is finished when the old store is empty, not when the new store is full.**

---

## 5. The Challenges

Every benefit of the migration comes with a distributed-systems price. The challenges below are not risks to be *managed around* — they are the permanent operating conditions of the target architecture, and the migration program must build for them from day one.

### 5.1 The Distributed Transactions (verified — the 2PC vs saga reality)

**The problem.** The monolith's killer feature is the ACID transaction: one database, one commit, atomicity guaranteed. Microservices give it up — a business operation that spans services (transfer money: debit *account* service, credit *payments* service, update *limit* service) can no longer be one transaction. The naive fix — the **two-phase commit (2PC)** across databases — exists but is avoided in practice: it requires a transaction coordinator, holds locks across the network, and couples the availability of every participant (one slow database blocks the whole transaction). The industry answer is the **saga**: the business operation is a sequence of local transactions, each with a compensating action, so the overall operation is *eventually* consistent and can always be undone. The saga's two choreography styles — **choreographed** (services react to each other's events) and **orchestrated** (a central coordinator drives the steps, the more controllable form) — and their production implementation are covered in depth in [apache_seata_guide.md](apache_seata_guide.md) (AT/TCC/SAGA/XA modes, Seata as the engine) and [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable orchestration — the workflow engine that makes sagas crash-safe). The migration angle: **which parts of the monolith are truly transactional, and which only look transactional?** Many monolith "transactions" are actually sequential business steps that were never atomic — they just *felt* atomic because they were in one codebase. Those become sagas cheaply. The few that are genuinely atomic (a booking, a settlement entry) are the ones that must either stay together (one service, one database) or be re-engineered deliberately. This is the single most important technical judgment in the migration.

### 5.2 The Eventual Consistency (verified)

**The problem.** With sagas and event-driven sync, the system is no longer *immediately* consistent: between the debit and the credit (or between the monolith's write and the service's materialized view) there is a **consistency window** — and the business must be able to tolerate it. Banking is a hard case: balances, limits, and risk positions are expected to be exact, and the *perception* of inconsistency (a customer sees a different balance on two channels) is a customer-service and regulatory event.

**The mitigations, in order of preference:** (a) **design the consistency window out** — keep genuinely atomic operations in one service (§5.1), so the window only exists where the business can tolerate it; (b) **make the window small and bounded** — synchronous-ish event pipelines, in-process sagas, outbox relays tuned for latency, so "eventually" means milliseconds-to-seconds, not overnight; (c) **make the window visible** — the UI states "updating", the API returns the authoritative source, the reconciliation reports surface the drift; (d) **compensate honestly** — when the saga fails, the compensating actions run, and the customer-facing result is a clean reversal, not a mystery. The reconciliation discipline of §4.4 (compare, investigate, sign off) is the *operational* form of eventual consistency: the system accepts eventual consistency *because* it proves convergence continuously. See [late_arriving_data_guide.md](late_arriving_data_guide.md) for the data-layer reality and [apache_seata_guide.md](apache_seata_guide.md) for the transaction-layer machinery.

### 5.3 The Testing (verified)

**The problem.** The monolith's test strategy — one codebase, one test suite, one deployment — does not transfer. In the distributed estate: a service's unit tests prove little about the system; integration tests against real dependencies are slow and flaky; and end-to-end tests across N services are expensive, non-deterministic, and cannot cover the combinatorics of failure. Testing a microservices system is a *different discipline*, and the migration program must stand it up before the services multiply.

**The practice:** the **test pyramid** still holds (many unit tests, fewer integration tests, few end-to-end) but the middle of the pyramid is replaced by **contract testing** — each service's API contract (request/response schemas, semantics) is tested *against the consumer's expectations* (consumer-driven contracts), so teams can deploy independently *without* breaking each other — the property the migration exists to create. Around it: **consumer-driven contract tests** (the consumer's expectations are the contract), **pact-style contract suites**, **resilience testing** (fault injection: kill a dependency, slow a database — the failure modes of §5.1/§5.2 are tested as behavior, not accidents), **property/characterization tests** on migrated logic (capture the monolith's actual behavior as executable specs *before* rewriting it — the testing twin of the strangler: you cannot rewrite behavior you have not pinned down), and **golden-file/comparison testing** during parallel run (feed both systems the same input, diff the outputs — the automated heart of §3.3's reconciliation). The migration-specific rule: **test the migration itself** — the backfill, the sync, the cutover, and the rollback are rehearsed until they are boring, because the cutover weekend is not the time to discover the sync has a gap.

**The testing table** — what proves what in the distributed estate:

| Test type | What it proves | Where it lives in the runbook |
|-----------|----------------|-------------------------------|
| **Unit tests** | A service's own logic | Every service, continuously — the pyramid's base, unchanged by the migration |
| **Contract tests (consumer-driven)** | Service A's expectations of service B's API hold; teams deploy independently | The estate-wide contract suite, owned per service pair; the *enabler* of independent deployability (§3.6) |
| **Characterization/property tests** | The migrated logic matches the monolith's actual (possibly undocumented) behavior | Written *before* each extraction — the strangler's testing twin (§3.1) |
| **Golden-file / comparison tests** | Old and new produce identical outputs for identical inputs | The automated heart of parallel run (§3.3) — runs in CI against captured production traffic |
| **Resilience/fault-injection tests** | The estate behaves (degrades, compensates, recovers) when dependencies fail | The saga and consistency designs of §5.1–5.2, exercised as behavior; rehearsed per wave |
| **Migration rehearsals** | The backfill, sync, cutover, and rollback work | The cutover gate (§6.4) — rehearsed until boring, on staging with production-shaped data |

### 5.4 The Observability (verified)

**The problem.** In the monolith, one log file, one process, one database: an error is findable. In the distributed estate, a single customer request fans out across services, brokers, and databases — and when something fails, the question "where did it fail?" must be answerable in minutes, or the operations team drowns. Observability is not a dashboard; it is the *property* that the system's internal state can be interrogated from its outputs — and it is a day-one migration requirement, because the migration's own machinery (facade routing, parallel-run reconciliation, sync lag) must be observable before the business relies on it.

**The practice — the three pillars:** **logs** (structured, correlated, centralized — every service logs the same request ID), **metrics** (latency, error rates, saturation per service — and migration-specific metrics: routing splits at the facade, reconciliation deltas, sync lag, outbox backlog), and **distributed traces** (the request's journey across services, via OpenTelemetry-style instrumentation and correlation IDs propagated through every hop, including into the message broker and out of it). The runbook specifics: the **facade is the observability chokepoint** (it sees every routed request — traffic splits, error deltas between old and new, the migration's health in one screen), the **reconciliation is a metric source** (drift between old and new stores is the single most informative number in the whole program), and the **sync pipeline is an observable system in its own right** (outbox backlog, event lag, dead-letter queues — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) and [event_stream_processing_guide.md](event_stream_processing_guide.md)). The identity dimension — tracing and audit across the new estate's services — cross-references [distributed_auth_guide.md](distributed_auth_guide.md).

### 5.5 The Challenges Table

| Challenge | Mitigation |
|-----------|------------|
| **Distributed transactions** | Saga (choreographed or orchestrated) with compensating actions; keep genuinely atomic operations in one service; engines in [apache_seata_guide.md](apache_seata_guide.md) and [temporal_workflow_guide.md](temporal_workflow_guide.md) |
| **Eventual consistency** | Design windows out where possible; make them small, bounded, and visible; compensate honestly; prove convergence with continuous reconciliation ([late_arriving_data_guide.md](late_arriving_data_guide.md)) |
| **Testing** | Contract testing (consumer-driven), characterization tests on migrated logic, resilience/fault-injection testing, golden-file comparison in parallel run; rehearse the cutover and rollback |
| **Observability** | Structured logs + correlation IDs, metrics, distributed tracing (OpenTelemetry-style); facade as the migration chokepoint; reconciliation deltas and sync lag as first-class metrics |
| **Latency** | Design for the network: batch, cache, async where possible; know each operation's latency budget; the monolith's in-process calls became network calls — measure the difference (§5.1) |
| **Security/identity** | Service-to-service auth, mTLS, token propagation across the new estate ([distributed_auth_guide.md](distributed_auth_guide.md)); the ACL facade is also the security boundary |
| **Operational complexity** | Platform engineering + SRE before the services multiply (§7); the deployment, scaling, and rate-limiting platform ([distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md)) |
| **Cost** | N services × environments × pipelines cost more than one monolith; track unit economics per service ([finops_guide.md](finops_guide.md)); retire the old estate's cost as the new estate's grows |
| **The migration itself** | The migration is a project with its own risks: sync gaps, reconciliation backlogs, facade bottlenecks, scope creep — run it with the phase gates of §6 |

### 5.6 The Failure Modes (flagged — the migration failures)

The migration's own failure modes deserve a section of their own — not because the challenges above were not serious, but because these are the ways the *program* dies, and they are organizational before they are technical. The widely-circulated statistics on migration/digital-transformation failure rates are **flagged** (no authoritative figure verified in this run — see §10); the *failure patterns* below, however, are the ones the practitioner literature and this series' banking guides consistently document:

| Failure mode | How it presents | The honest mitigation |
|--------------|-----------------|------------------------|
| **The big-bang rewrite** | The program builds "the new system" for years, then attempts one giant cutover — which fails, is rolled back, and is retried | The strangler fig (§3.1) exists precisely because big-bang replacement of a working core fails; there is no large-scale success story that did not go incremental |
| **The seamless decomposition** | The program starts extracting before the boundaries are known; services are drawn by committee, split by technical layer (one "database service", one "business-logic service") | The decomposition phase (§2, §6 phase 1) is a *phase*, not a formality — the seams come first |
| **The data-first error** | The program splits the database before the ownership is decided; every consumer breaks; the migration is blamed | Ownership before databases (§4.1); the data migration follows the seams, not the other way around |
| **The frozen-host error** | The monolith is frozen "because we're migrating"; business change accumulates; the migration becomes the bottleneck it was meant to fix | Branch by abstraction (§3.2); the host stays healthy until the end (§6.3) |
| **The org-not-ready error** | Services are built without platform, without SRE, without two-pizza owners; the multiplied operational load drowns the teams | The team topology (§7) and the platform (§5.5) are prerequisites, not afterthoughts |
| **The zombie coexistence** | The migration never ends: the facade stays, the dual runs stay, "temporary" sync stays — because nobody signed off the last 10% | Per-wave exit criteria and a decommissioning plan for the scaffolding itself (§6.2, §8.2) |
| **The metric-less program** | Nobody measures whether the migration delivered: release cadence, change lead time, deploy failure rate are not tracked, so the program cannot end | The §1.1 drivers become §6 phase-5 success metrics; the migration is finished when the business metrics say so |

The failure rule: **every failed migration in the catalogue failed in one of these seven ways — and every one of them is prevented by the runbook's structure: seams first, waves with exit criteria, healthy host, ready organization, and measured outcomes.** The failure statistics are flagged because they are unverifiable; the failure *patterns* are not flagged, because they are the documented history of the discipline.

The challenges rule: **none of these are solved by the target architecture; they are solved by the platform, the process, and the team — which is why the next two sections are about sequencing and people.**

---
## 6. The Sequencing

The migration patterns say *how*; the sequencing says *in what order* — and ordering is where migrations are won or lost. A migration that extracts the wrong capability first can stall the whole program; one that extracts the right capability first funds itself. The runbook's canonical phase model is **analyze → plan → decompose → migrate → validate** — five phases with explicit exit criteria, so that no phase is ever "mostly done."

### 6.1 The Phases (verified — the runbook model)

**Phase 1 — Analyze.** The discovery phase: map the monolith's capabilities, code, and data; find the bounded contexts (§2.1) and the data ownership (§4.1); measure the drivers (§1.1) and confirm the decision (§1.2). Deliverables: the capability map, the context map, the table-ownership map, the dependency inventory, and the honest verdict (including "don't migrate" and "modular monolith first"). **Exit criterion:** the seams are documented and agreed — the decomposition is not a guess.

**Phase 2 — Plan.** The design phase: choose the target architecture (the OBMA-style blueprint in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) is the banking reference), sequence the waves (§6.3), design the facade and the ACL (§3), design the data sync (§4.3), stand up the platform (§5.5), and draw the team topology (§7). Deliverables: the target architecture, the wave plan with exit criteria per wave, the platform backlog, the team plan, the risk register. **Exit criterion:** each wave has a definition of done that includes its data migration, its parallel run, and its rollback — not just its code.

**Phase 3 — Decompose.** The extraction phase, wave by wave: build the facade and the ACL, extract the first capability (branch by abstraction at the code level, §3.2), wire the service, sync the data (§4), run it in parallel against the monolith (§3.3), and switch traffic at the facade (§3.1). The first wave is the *pilot* — deliberately small, deliberately end-to-end, it proves the whole machinery (facade, ACL, sync, reconciliation, observability, rollback) before the program scales. **Exit criterion:** the wave's functions are fully routed to the service, reconciliation is clean for the agreed period, and the rollback was rehearsed — then the next wave starts.

**Phase 4 — Migrate.** The *data* migration phase, per book: backfill, sync, freeze, cut over, archive (§4.2). Note that "migrate" is not the same as "decompose": a capability can be *decomposed* (running as a service) long before its historical *data* is migrated — the OBMA playbook's 24-month window + archive pattern is the standard shape ([../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14.4). **Exit criterion:** each book's system of record is the new service, the legacy copy is frozen, and the archive is queryable.

**Phase 5 — Validate.** The acceptance phase, running *throughout* and *after* the migration: the reconciliation continues, the observability proves the new estate, the cost model is tracked, the failure modes (§5) are exercised by fault-injection testing, and the retired monolith's functions are confirmed dead. **Exit criterion:** the migration's success metrics (release cadence, change lead time, deploy failure rate, per-service scaling, the §1.1 drivers that justified the migration) are measured and show the expected improvement — the migration is finished when the *business* says it is, not when the last service is cut over.

### 6.2 The Sequencing Table

| Phase | Activities | Exit criteria |
|-------|-----------|---------------|
| **1. Analyze** | Capability map; bounded-context discovery; table-ownership map; driver measurement; decision confirmation | Seams documented and agreed; "migrate / don't / modular-first" verdict signed off |
| **2. Plan** | Target architecture; wave sequencing; facade/ACL/data-sync design; platform backlog; team topology; risk register | Every wave has definition-of-done covering data migration, parallel run, and rollback |
| **3. Decompose** | Facade + ACL live; pilot wave end-to-end; per-wave extraction (branch-by-abstraction → service → sync → parallel run → route) | Wave functions routed; reconciliation clean for agreed period; rollback rehearsed |
| **4. Migrate** | Per-book backfill + incremental sync; freeze; cutover of system of record; archive | System of record is the service; legacy copy frozen; archive queryable |
| **5. Validate** | Continuous reconciliation; observability of the new estate; fault-injection; cost tracking; success-metric measurement | Business success metrics show the expected improvement; the monolith is empty |

### 6.3 The Ordering Principles

The wave order is a strategic decision, not a convenience. The principles that make it work:

- **Start with the seam, not the symptom.** The first wave must be the capability whose *boundary* is cleanest (DDD context, capability, and team all agree — §2.4) — not the one that hurts most. The pilot exists to prove the machinery; a messy first boundary sinks the pilot and the program's confidence with it.
- **Follow the dependency direction.** Migrate *upstream of* the dependencies: the party/customer data first (everyone references it), then accounts, then payments, then the products that consume them. The OBMA waves (§14 of [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md)) are the canonical banking order: party + accounts → payments → loans/deposits → cards/limits/trade — each wave gives the next one a cleaner dependency base.
- **Sequence for value and risk together.** Each wave should deliver *some* business value (a new channel, a faster process, a regulatory win) and retire *some* risk (the scariest dependency, the most-coupled table). A wave with neither is waste; a wave with only risk is unfundable.
- **The facade order is the data order.** Traffic can only be routed to a service whose data is synced; so the §3 routing order and the §4 data order must be the same wave plan — a classic failure is the "API-first" program that routes traffic to a service still reading the monolith's database.
- **Keep the monolith healthy until the end.** The strangler assumes the host stays alive and *changeable* (new functions still land in the monolith while old ones are extracted). Freezing monolith development "because we're migrating" turns the migration into a hostage situation — branch by abstraction (§3.2) exists precisely so the monolith can keep shipping.

The sequencing rule: **the migration is a sequence of small, reversible, value-carrying slices — each slice is a complete mini-migration (decompose → sync → parallel run → route → validate) — and the order is decided by seams, dependencies, value, and risk, in that priority.**

### 6.4 The Governance Gates

A migration without gates is a migration without a stop mechanism — and the runbook's phases are only real if each has a **sign-off** that the program actually honors. The gate structure, mapped to the phases:

| Gate | When it fires | What it decides | Who signs |
|------|---------------|-----------------|-----------|
| **The decision gate** | End of Phase 1 (Analyze) | Migrate / don't / modular-first — the §1 verdict, with the evidence | Architecture + business sponsors |
| **The wave-approval gate** | Before each wave's extraction starts | The wave's scope, its definition-of-done (data migration + parallel run + rollback), its value and risk (§6.3) | The architecture enablement team + the owning domain team |
| **The reconciliation gate** | End of each wave's parallel run | The old and new systems agree for the agreed period — reconciliation is clean, exceptions are investigated and closed | Operations/SRE + the domain team |
| **The cutover gate** | Before each book's cutover weekend | The cutover and rollback were rehearsed; the sync is current; the freeze is agreed; the rollback plan is current | Operations + the business owner of the book |
| **The phase-completion gate** | End of each phase | The phase's exit criteria (§6.2) are met — including the *organizational* ones (§7) | The program board |
| **The decommission gate** | After the last wave | The monolith is empty, the facade and ACL are decommissioned, the archive is queryable — the migration is over (§9) | The program board + the business |

The gate rule: **a gate that can be waived by enthusiasm is not a gate — the reconciliation gate in particular must be able to stop a wave, because it is the only gate with direct evidence about correctness.** The gates are the runbook's institutional memory: they are what turns the five phases from a diagram into a program.

### 6.5 The Duration Reality (flagged — how long this takes)

An honest paragraph about time, because the runbook's biggest enemy is the expectation of a deadline. **Flagged:** the widely-circulated figures for migration duration (e.g., "2–4 years for a core-banking modernization", "two years to decompose a large monolith") could not be verified against an authoritative source in this run — durations are inherently case-specific, and §10 records the flag. What the *qualitative* evidence supports, consistently: (a) a large legacy-core replacement is measured in **years**, not quarters — the OBMA playbook's five phases in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14 are explicitly a multi-year journey; (b) the decomposition and platform phases *feel* unproductive (no user-visible features) and are where sponsors get impatient — the governance gates (§6.4) exist to make their progress visible; (c) the pilot wave (first value) typically lands in **6–12 months** if the seams were honestly drawn; and (d) the migration's end is defined by *business metrics* (§6.2 phase 5), not by a calendar. The duration rule: **plan for years, prove value in months, and let the gates — not the Gantt chart — decide when each phase is done.** The strangler's patient journey is patient for a reason (§9).

---

## 7. The Team Topology

The migration's success is decided by the organization before it is decided by the architecture. This section is deliberately placed after the sequencing *in the runbook* but before the worked example — because the worked example assumes the organization that can run it.

### 7.1 The Teams (verified — the two-pizza, Bezos; Conway, 1968)

**The two-pizza team (verified — Bezos; date flagged).** The rule attributed to **Jeff Bezos** at Amazon: **a team should be small enough to be fed with two pizzas** — small enough that coordination is informal and ownership is total. The two-pizza rule dates from Amazon's early years (commonly cited as circa **2002**, but the exact origin date is anecdotal — flagged in §10); it was codified in the wider Amazon operating model (single-threaded owners, the "fitness" of owning a service end-to-end) and is the canonical statement of the *service-sized team*. In migration terms: **each extracted service (or small group of related services) is owned by one two-pizza-sized team** — build, run, and evolve it — and the service boundary is drawn to match the team's cognitive capacity (§2.3's team-first decomposition). A service too big for its team becomes a monolith-in-miniature; a service too small for a team becomes a coordination tax.

**Conway's law (verified — Conway, 1968).** Melvin Conway's observation, "organizations design systems that mirror their own communication structure" (from his 1968 paper *"How Do Committees Invent?"*, Datamation, April 1968 — verified): the architecture will *end up* matching the org chart whether or not anyone designs it that way. The migration corollary — the **inverse Conway maneuver**: design the team topology first, then draw the service boundaries to match — is the organizational form of §2.3. A migration that draws services by architecture committee and then maps teams onto them fights Conway's law; one that draws the teams first harnesses it.

### 7.2 The Topology Table

| Team | Responsibility | Services/scope owned |
|------|----------------|----------------------|
| **The product/domain teams** (two-pizza sized, per bounded context) | Build, run, and evolve their services end-to-end; own the domain model, the API contract, the data | Party & customer, accounts, payments, trade finance, limits, reporting (each a service or small group) |
| **The platform team** | The shared substrate: CI/CD, containers/K8s, the event backbone, the API gateway, the observability stack, the deployment platform | Owns *infrastructure*, not business domains — the enabler of every domain team |
| **The SRE/operations team** | Reliability, incident response, capacity, the reconciliation and cutover operations | Cross-cutting: the parallel-run reconciliation, the facade's health, the sync pipelines |
| **The migration/transformation team** | The facade, the ACL, the data-sync machinery, the wave plan, the cutover rehearsals | The migration *itself* is a product: the strangler's scaffolding until the last wave |
| **The architecture enablement team** | The target architecture, the standards, the guardrails, the context-map governance | The design authority; advises, does not command-delegate (or Conway reasserts itself) |
| **The legacy caretakers** | Keep the monolith healthy and changeable until it is empty (§6.3) | The monolith's remaining functions; the freeze-and-archive exit |

### 7.3 The Topology Principles

- **The platform team is not optional.** Without it, every domain team reinvents deployment, messaging, and observability — N teams × N platforms, the multiplied-cost failure of §5.5. The platform is the *economic* precondition for microservices (and the [finops_guide.md](finops_guide.md) angle: shared platform cost is how the multiplied estate stays affordable).
- **The domain teams own the contract.** The API contract and the data ownership live with the domain team; the integration patterns ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md)) govern how they connect. The ACL is owned jointly: the migration team builds it, the domain team owns its mappings.
- **The migration team is temporary; the domain teams are permanent.** The strangler's scaffolding (facade, ACL, sync, reconciliation) is decommissioned at the end; the domain teams and the platform persist. Staff the permanent teams from the start — the migration team is the seed, not the destination (the people who build the facade should end up owning the services).
- **One team per context, one context per team.** The bounded context (§2.1) and the two-pizza team (§7.1) are the same boundary drawn twice — once in the domain, once in the org. Where they diverge, the org wins (Conway), and the architecture drifts.

The topology rule: **draw the teams first (inverse Conway), size them two-pizza (Bezos), align one context per team (Evans), and build the platform before the services multiply — the architecture will then mirror the organization you actually want.**

### 7.4 The Topology Transition and the Anti-Patterns

The topology is not static — it *moves* with the migration, and the transition is where organizations most often stumble. The healthy trajectory:

- **Before the migration:** one monolith team (or a few), a small ops function, no platform team. The migration team is seeded from the monolith's own engineers — the people who know where the bodies are buried.
- **Phase 2–3 (plan/decompose):** the domain teams form around the first contexts (party, payments), the platform team is stood up *before* Wave 2 needs it, and the migration team owns the scaffolding (facade, ACL, sync). The monolith caretakers remain — the host must stay healthy (§6.3).
- **Phase 4–5 (migrate/validate):** the domain teams absorb the services fully; the migration team shrinks as the scaffolding is decommissioned; the monolith caretakers wind down as the last books freeze. The permanent estate is the domain teams + platform + SRE.

And the anti-patterns — the topologies that reliably sink migrations:

| Anti-pattern | The symptom | The fix |
|--------------|-------------|---------|
| **Services without owners** | Nobody is responsible end-to-end; "the platform team" or "the integration team" absorbs everything; incidents ping-pong | Two-pizza ownership per service/context (§7.1); a service with no named owner is not a service, it is a liability |
| **Teams without services** | The org chart keeps its old shape (one big delivery team, one ops team) while the architecture is carved into services; Conway reasserts itself and the services re-couple | Inverse Conway: redraw the teams to the target boundaries *before* the services land (§7.1) |
| **The matrix paralysis** | Every service has three "owners" (a product manager, a tech lead, an architect) and no single accountable team | One accountable two-pizza team per context; advisory roles advise |
| **No platform team** | Each domain team builds its own deployment, messaging, and observability; N teams × N platforms | The platform is the economic precondition (§7.3, [finops_guide.md](finops_guide.md)); stand it up in Phase 2, not after the services multiply |
| **The migration team that never dissolves** | The scaffolding team becomes permanent; the facade and ACL are staffed forever; the "temporary" layers become a second legacy | The migration team's job description includes its own dissolution (§6.4's decommission gate) |

The transition rule: **the org chart and the architecture must arrive at the target together — the teams are drawn for the destination in Phase 2, the services land on them wave by wave, and the scaffolding teams are designed to dissolve.** A migration whose org chart is unchanged at the end has not migrated; it has renamed a monolith.

---

## 8. The Worked Example: A Cymbal Bank's Monolith Migration

### 8.1 The Scenario (the familiar context)

The subject: a **Cymbal Bank-style wholesale bank** — the familiar context of this series (see the trade-finance and software-systems guides: [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md), [../banking/dbs_software_systems_guide.md](../banking/dbs_software_systems_guide.md)) — global corporate-and-investment banking with a Singapore hub: trade finance, payments and cash management, FX, loans, and regulatory reporting. Its core estate is a **monolith**: a packaged core banking platform (FLEXCUBE-class — the migration playbook of [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14 is the reference for exactly this platform) or a mainframe/AS/400-era core ([ibm_as400_guide.md](ibm_as400_guide.md) for the platform reality), surrounded by satellite channel apps. The symptoms are the classic ones from §1.1: releases are quarterly and risky, the channel teams block each other, the core's batch window limits intraday integration, and the digital ambitions (real-time trade tracking, API banking, instant payments) are impossible on the monolith's integration model.

The decision (§1) is made honestly: **the monolith is the bottleneck for the digital agenda** (drivers measured: release cadence, time-to-market for a new API product), the **modular-monolith alternative is insufficient** (the digital products need independent scaling and isolated failure domains — a payments outage must not take down trade finance), and the **organization can absorb the journey** (executive sponsorship, a platform team being formed, a delivery model that funds a multi-year program). The target architecture is the OBMA-style blueprint — the series' reference architecture in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md).

### 8.2 The Strangler Plan (the design)

**Phase 0 — the seams (§2).** The bounded-context discovery finds the bank's contexts: **Party & Customer** (onboarding, KYC, legal entities), **Accounts** (the ledgers), **Payments** (initiation, clearing, SWIFT), **Trade Finance** (LCs, guarantees, collections), **Limits & Collateral**, **Risk & Reporting**. The capability map agrees. The table-ownership map (§4.1) reveals the honest truth: the `CUSTOMER` table is genuinely shared (every context reads it) — it belongs to Party & Customer, and every other context will receive customer changes as **events**, not by table access. One area — the **settlement engine** — is found to be genuinely atomic and transactionally coupled to the ledgers; it stays inside Accounts (the §5.1 judgment applied: not everything is separable).

**Phase 1–2 — the facade and the plan (§3, §6).** A **facade/API gateway** goes in front of the core: all channel traffic now flows through one routing point, wrapped in an **anti-corruption layer** that translates the core's model (copybooks, status codes, five customer-address formats) into the new domain model ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.2 — the ACL mechanics). The waves are sequenced by dependency (§6.3):

- **Wave 1 — Party & Customer (the pilot).** The cleanest seam, everyone's dependency. The service is built (branch by abstraction creates the seam inside the monolith's customer module), its data is backfilled and synced (**outbox + events**, §4.3 — the core publishes customer events; the service materializes its own store), it runs in **parallel** against the core (golden-file comparison: the same onboarding inputs, diffed outputs), and the facade routes onboarding traffic to it. The pilot proves the whole machinery — facade, ACL, sync, reconciliation, observability, rollback — in six months.
- **Wave 2 — Payments.** The digital agenda's crown jewel. Payment initiation moves to the payments service; posting stays in whichever system owns the account (the OBMA coexistence pattern, §14.2 of the target guide); the **saga** handles cross-service operations (debit account → initiate payment → update limit, with compensating reversals — orchestrated via the workflow engine, [temporal_workflow_guide.md](temporal_workflow_guide.md), with the transaction machinery of [apache_seata_guide.md](apache_seata_guide.md)). Parallel run with daily balance/position reconciliation before the facade switches payment traffic.
- **Wave 3 — Trade Finance.** The differentiating book: LCs, guarantees, collections. Slowest to migrate (documentary processes, the paper-based lifecycle of [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md)); the ACL carries the heaviest translation here; the events keep the core's trade positions and the service's synchronized through the parallel run.
- **Wave 4 — Limits, Risk & Reporting.** Limits join the service estate; reporting moves to an **event-fed** analytics store (CQRS-shaped read models, §4.3) — the old reporting database is retired rather than kept alive "for queries."

**The estate before and after** — the migration's shape in one table:

| Layer | Before (the monolith era) | After (the service estate) |
|-------|---------------------------|----------------------------|
| **Channels** | Mobile, internet, branch — all calling the monolith's one interface | Channels call the **facade**, which routes to services; new API products are built on the services directly |
| **Business capabilities** | One codebase, one deployable | Party & Customer, Accounts, Payments, Trade Finance, Limits & Collateral, Risk & Reporting — one service (or small group) each |
| **Data** | One shared database; `CUSTOMER` read by everyone | **Database per service** (§4.1); customer data owned by Party, distributed by **events**; the ledger atomic within Accounts (§5.1) |
| **Integration** | Point-to-point, batch files, screen scraping | **ACL + facade** at the legacy boundary; **event backbone** inside the estate ([event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md)) |
| **The core** | System of record for everything | System of record for the *frozen books only*; read-only archive for look-back; eventually decommissioned (§3.1 eliminate) |
| **Operations** | One batch window, one team, one release train | Per-service pipelines, SRE, reconciliation and observability as first-class systems (§5.3–5.4) |

**The wave timeline** — the sequencing with its evidence:

| Wave | Capability | Duration (indicative) | Exit evidence (the gate) |
|------|-----------|----------------------|---------------------------|
| 0–1 | Seams + facade + ACL | 4–6 months | Context map, ownership map, facade routing, ACL translations agreed and signed (§6.4's decision gate) |
| 1 | Party & Customer (the pilot) | ~6 months | Onboarding routed to the service; reconciliation clean 4 consecutive weeks; rollback rehearsed |
| 2 | Payments | ~9–12 months | Payment initiation routed; saga + compensation exercised in fault-injection; balance/position reconciliation clean |
| 3 | Trade Finance | ~9–12 months | LC/guarantee journeys routed; the ACL's heaviest mappings signed; trade-position reconciliation clean |
| 4 | Limits, Risk & Reporting | ~6–9 months | Limits routed; reporting on the event-fed store; the old reporting DB retired |
| 5 | Data migration + cutover | ~6 months, per book | System of record per book is the service; legacy frozen; archive queryable (§6.2 phase 4) |
| 6 | Decommission | ~3 months | Monolith empty; facade + ACL deleted; the decommission gate fires (§6.4) |

**Phase 3–4 — data migration and cutover (§4.2).** Per book: backfill → sync → freeze → cut over → archive. **24 months** of transaction history migrates; older history is archived and kept queryable (the regulatory look-back). Cutovers happen on weekends, each with a rehearsed rollback (the sync has been running until the last minute, so rollback is lossless). The core is *frozen per book* — not all at once.

**Phase 5 — validate and eliminate (§3.1).** Wave by wave, the facade routes more, the core does less. The reconciliation continues through the whole program. The success metrics from §1.1 are measured: release cadence goes quarterly → weekly on the services; the payments service scales independently during the instant-payments push; a payments incident no longer takes down trade finance. When the last function is strangled, the core is **retired — frozen read-only and archived** for the books that require look-back, per the pattern's eliminate phase. The facade, ACL, and sync scaffolding are decommissioned: the ACL is deleted, and the domain services are untouched (§3.4's payoff).

### 8.3 The Lessons

- **The seam came first.** The decomposition (contexts, capability, teams agreeing) took as long as any single wave — and every wave after it was cheap by comparison. Skipping it would have made every wave expensive.
- **The pilot funded the program.** Wave 1 delivered a real business outcome (digital onboarding with instant KYC status) and proved the machinery; the program's skeptics became its sponsors. A pilot with no business value is a science project.
- **The data was the plan.** The wave order was effectively decided by the table-ownership map: you can only route what you can sync. The "API-first" temptation (route traffic before data) was the program's biggest avoided mistake.
- **The monolith stayed alive.** The core kept shipping (new products, regulatory changes) throughout — branch by abstraction kept the seams clean without freezing development. The strangler's host must stay healthy or the vine dies with it.
- **The organization was designed, not hoped for.** The two-pizza domain teams, the platform team (built before Wave 2), the temporary migration team that seeded the permanent ones — the org chart was drawn before the service boundaries, and Conway's law did the rest.
- **Eventually consistent, provably convergent.** The bank accepted consistency windows because the reconciliation *proved* convergence continuously — the sync lag and reconciliation deltas were the program's most-watched metrics. "Eventually" was a measured number, not a hope.

### 8.4 The Migration in Numbers

The §1.1 drivers become the §6 phase-5 success metrics — and a worked example is only honest with its numbers visible. The indicative before/after for the Cymbal Bank (illustrative, not a real program's data — flagged in §10):

| Metric | The monolith era | After the migration (measured per §6 phase 5) |
|--------|------------------|------------------------------------------------|
| **Release cadence** | Quarterly, one train, everything coupled | Weekly per service, independent pipelines |
| **Change lead time** | Weeks (change advisory + the one train) | Hours-to-days per service, contract-tested (§5.3) |
| **Deploy failure rate** | High — one bad change blocks everything | Lower per service — blast radius is one service, not the bank |
| **Time-to-market for a new API product** | Quarters (touch the monolith, the core, the batch window) | Weeks — built on the services behind the facade |
| **Scaling** | All-or-nothing monolith scale | Payments scales independently during instant-payments peaks |
| **Failure isolation** | A payments incident takes down trade finance | One service degrades; the estate and the facade route around it |
| **Reconciliation status** | End-of-day batch reconciliation, silent unless broken | Continuous, visible, with signed-off deltas — the §4.4 discipline |

The numbers rule: **if the migration's before/after table cannot be filled in, the program has no definition of done — measure the §1.1 drivers at the start (Phase 1), and the migration ends when the table's right column is true, not when the last service deploys.**

---

## 9. The Summary: The Strangler's Patient Journey

The migration from monolith to microservices is not a project with a deadline; it is a **patient journey** — the strangler fig growing around the host tree, ring by ring, until the vine *is* the tree. The one-page version of everything above:

- **Decide first (§1).** Measure the drivers; if the monolith is not the bottleneck, keep it or make it modular (the modular monolith is a destination, not a detour). Microservices are an organizational pattern — adopt them for teams, scaling, and isolation, never for fashion.
- **Find the seams (§2).** DDD bounded contexts (Evans, 2003) and business capabilities, agreed with the team topology (Conway, 1968; two-pizza, Bezos). A boundary that is only technical will move; a boundary that is domain + business + organizational holds.
- **Strangle with patterns (§3).** The facade routes, the ACL translates, branch by abstraction refactors in place, parallel run proves correctness, events and outboxes carry the data (Fowler, 2004; Hammant, 2013; the integration catalogue in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md)).
- **Migrate the data honestly (§4).** Ownership before databases; sync with events; prove equality with reconciliation; migrate the window, archive the rest; cut over per book with lossless rollback.
- **Pay the distributed price knowingly (§5).** Sagas instead of 2PC, bounded and visible consistency windows, contract testing, and observability from day one — the platform and the process carry these, not the architecture.
- **Sequence in slices (§6).** Analyze → plan → decompose → migrate → validate, wave by wave, each wave a complete mini-migration with value, risk-retirement, and exit criteria. The pilot first, dependencies next, the monolith healthy until the end.
- **Build the organization (§7).** One context per team, teams before boundaries, the platform before the services multiply, the migration team as the seed of the permanent ones.

**The runbook at a glance** — the one-page quick reference:

| The question | The answer | The section |
|--------------|------------|-------------|
| Should we migrate? | Only if the monolith is measurably the bottleneck — otherwise modular monolith first | §1 |
| Where are the seams? | DDD bounded contexts + business capabilities, agreed with the teams | §2 |
| How do we replace it? | Strangler fig: facade → route → retire; ACL at the boundary; branch by abstraction inside; parallel run for proof | §3 |
| How does the data move? | Ownership before databases; sync with events and outboxes; reconcile; migrate the window, archive the rest; cut over per book | §4 |
| What will hurt? | Distributed transactions (sagas), eventual consistency (bounded + visible), testing (contracts + characterization), observability (traces + reconciliation metrics) | §5 |
| In what order? | Analyze → plan → decompose → migrate → validate; waves with exit criteria; gates that can stop the program | §6 |
| Who runs it? | Two-pizza domain teams (one per context), a platform team, SRE, a temporary migration team that dissolves | §7 |
| How do we know it worked? | The §1.1 drivers, measured: release cadence, change lead time, deploy failure rate, per-service scaling | §6 phase 5 |

**The final word — the strangler's patient journey:** the migration succeeds when it stops being a migration — when the facade is gone, the ACL is deleted, the core is archived, and the teams simply *run* the estate they built. The patient journey is slow by design: every ring of the vine is small enough to test, reversible enough to unwind, and valuable enough to fund the next. That is not slowness; that is how you replace a bank's core without stopping the bank. The host tree is not the enemy — it is the map of the journey, the proof that the new system works, and eventually the archive that lets the new one stand alone. *Migrate like the strangler: patiently, ring by ring, until the old is gone and the new has always been there.*

---

## 10. Verification and Claims-Status

Per the research discipline of this series, each key factual claim was checked against sources during the writing of this guide. Items that could not be verified are **flagged** rather than silently asserted.

**Method.** Verification used targeted web searches against primary sources (Fowler's bliki, Paul Hammant's blog, trunkbaseddevelopment.com, publisher records for Evans 2003 and Newman 2015, the Wikipedia/canonical citations for the strangler fig and two-pizza rule) and cross-checks against this repository's sibling guides ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [apache_seata_guide.md](apache_seata_guide.md), [event_stream_processing_guide.md](event_stream_processing_guide.md)), which were written to the same citation discipline.

**Verified:**

- **Strangler fig** — Martin Fowler, bliki post *"StranglerFigApplication"*, **2004** (later retitled; the page has been updated as recently as 2024). Confirmed via Fowler's bliki citation and secondary sources (Wikipedia; multiple architecture references). The Queensland-tree metaphor and the transform → coexist → eliminate mechanics are Fowler's.
- **DDD / bounded context** — *Domain-Driven Design: Tackling Complexity in the Heart of Software*, **Eric Evans, Addison-Wesley, 2003**; the **bounded context**, **ubiquitous language**, **aggregate**, and **subdomain** (core/supporting/generic) concepts are Evans's. Confirmed via publisher records and the canonical DDD literature.
- **Branch by abstraction** — **Paul Hammant**, blog post *"Introducing Branch By Abstraction"* (October 2013), who introduced the term and **credits Stacy Curl** with originally coming up with the idea (per Fowler's bliki); Fowler's bliki article *"BranchByAbstraction"* is dated **7 January 2014**. Confirmed via Hammant's blog, trunkbaseddevelopment.com, and Fowler's bliki. The Stacy Curl attribution is retained as flagged nuance (the precise division of credit is as Hammant/Fowler describe it).
- **Modular monolith** — the term was popularized by **Simon Brown, "Modular Monoliths", GOTO Berlin 2018**; the concept and phrase predate the talk (e.g., the microservices-anti-hype literature of the mid-2010s, Stefan Tilkov's "Don't jump to microservices", 2017). **Flagged nuance:** "popularized by Brown 2018" is the verified claim; "coined by Brown 2018" would be an overstatement.
- **Two-pizza team** — the rule is attributed to **Jeff Bezos** at Amazon; the commonly cited origin is Amazon's early years, **circa 2002**, but the exact date is anecdotal (secondary sources; Brad Stone's *The Everything Store* documents the Amazon operating model). **Flagged:** "Bezos, early Amazon years" is well established; the precise year is not independently verifiable.
- **Conway's law** — **Melvin Conway**, *"How Do Committees Invent?"*, Datamation, **April 1968**. Confirmed via the canonical citation.
- **Anti-corruption layer** — Martin Fowler, bliki *"AntiCorruptionLayer"*, **2004 (widely cited)**; the exact publish date could not be re-confirmed directly in this run — the same flag as [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §11. The DDD lineage (Evans, 2003) is verified.
- **Database per service / shared database anti-pattern** — **Sam Newman, *Building Microservices*, O'Reilly, 2015** — the canonical statement of data ownership per service. Confirmed via publisher records and the canonical literature.
- **Parallel run** — industry-standard practice (no single originator); the banking form is documented in the series' own target-architecture guide ([../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) §14: parallel run, dual processing, reconciliation, cutover) and the legacy-integration guide.
- **CQRS** — credited to **Greg Young (circa 2010)**; Fowler's bliki *CQRS.html* dated **July 2011** — per the flag already recorded in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §11.
- **Saga** — the pattern is long-established distributed-systems practice (the term originates with García-Molina & Salem, 1987, "Sagas", ACM SIGMOD); the production engines referenced ([apache_seata_guide.md](apache_seata_guide.md), [temporal_workflow_guide.md](temporal_workflow_guide.md)) are documented in this repository.

**Flagged / not independently verified in this run:**

- **The Gartner decomposition prediction** — the widely-circulated Gartner claim that "by 2018, less than one-third of enterprises will have successfully decomposed their application portfolios into microservices" could **not be re-verified** against a primary Gartner source in this run (search results surfaced a different, cloud-strategy Gartner statistic). Treat it as indicative, not citable. The *qualitative* claim — that monolith-to-microservices migrations frequently fail or stall — is well supported by practitioner literature (the anti-hype corpus: Brown, Tilkov, the "microservices are a great way to make a mess" refrain) but **no authoritative failure-rate statistic was verified**.
- **The two-pizza origin date** — see above; "circa 2002" is flagged.
- **Microservices adoption numbers** — no authoritative adoption statistic verified; the *qualitative* adoption claim (microservices as the dominant target architecture for digital-platform modernization) is directly evidenced by the target-architecture and vendor guides in this repository.
- **The worked example** — the Cymbal Bank scenario is a *synthetic* worked example in the series' established style (the same convention as the trade-finance guides' worked examples); it is not a description of any actual Cymbal Bank migration program.

---

## 11. Glossary

- **Monolith** — a software application built and deployed as a single unit: one codebase, one process, one database, one release pipeline. Not inherently bad; the *deployment* and *ownership* coupling is the problem the migration addresses (§1.1).
- **Microservices** — an architectural style that structures an application as a set of small, independently deployable services, each owning its own data (database per service) and communicating over the network. The target of the migration (§1.4).
- **Migration** — the program of work that replaces a running system (the monolith) with a new architecture (microservices) without stopping the business — the subject of this guide.
- **Strangler fig** — the migration pattern named for the vine: grow the new system around the old, function by function, until the old can be cut away. Coined by Martin Fowler, 2004 (§3.1).
- **Strangler pattern** — the common name for the strangler fig pattern; also used for its *mechanism* (facade routing + incremental replacement) in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.1.
- **Branch by abstraction** — the refactoring technique of replacing a component behind an interface, in parallel, without a long-lived branch; coined by Paul Hammant (2013, crediting Stacy Curl) (§3.2).
- **Parallel run** — running the old and new systems on the same workload and reconciling their outputs before cutover; the validation engine of the migration (§3.3).
- **Anti-corruption layer (ACL)** — the dedicated translation layer that protects one system's domain model from another's; Fowler, 2004; from DDD (§3.4).
- **ACL** — abbreviation of anti-corruption layer.
- **DDD** — domain-driven design; the modeling discipline of Evans, 2003 (§2.1).
- **Domain-driven design** — the approach that models software on the business domain, with bounded contexts and ubiquitous language as its core ideas (Evans, 2003).
- **Bounded context** — the explicit boundary inside which a domain model, its terms, and its rules apply; the primary decomposition unit of the migration (§2.1).
- **Evans** — Eric Evans, author of *Domain-Driven Design* (2003), the origin of bounded contexts and the DDD tradition.
- **Modular monolith** — a single deployable unit with strict internal module boundaries; the leading alternative to migrating, and a valid destination (§1.3).
- **Decomposition** — the act of finding and drawing the boundaries (contexts, capabilities, teams, data) along which the monolith is split (§2).
- **Database per service** — the microservices data rule: each service owns its data, and no other service reaches into it (Newman, 2015) (§4.1).
- **Shared database** — multiple applications reading and writing the same schema; the monolith's data model, and the microservices anti-pattern the migration must dismantle (§4.1).
- **Event-driven** — an architecture where state changes flow as events on a broker/stream; the sync fabric of the migration and the target estate (§4.3).
- **Distributed transaction** — a transaction spanning multiple services/databases; solved by sagas, not 2PC, in practice (§5.1).
- **Eventual consistency** — the guarantee that, absent new changes, all copies converge; the operating condition of the distributed estate, made tolerable by bounded windows and reconciliation (§5.2).
- **Saga** — a sequence of local transactions with compensating actions that implements a business operation across services (§5.1; [apache_seata_guide.md](apache_seata_guide.md)).
- **Testing** — the distributed-estate discipline: contract testing, characterization tests, resilience testing, and comparison testing during parallel run (§5.3).
- **Observability** — the property that the system's internal state can be interrogated from its outputs: logs, metrics, and traces (§5.4).
- **Two-pizza team** — a team small enough to be fed with two pizzas; the Bezos/Amazon rule for service-sized teams (§7.1).
- **Bezos** — Jeff Bezos, Amazon founder, the two-pizza rule's originator (§7.1).
- **Sequencing** — the ordering of the migration's slices: analyze → plan → decompose → migrate → validate (§6).
- **Phase** — a stage of the migration with explicit activities and exit criteria (§6.2).
- **Legacy** — the old systems the migration wraps, routes around, and eventually retires; see [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) and [ibm_as400_guide.md](ibm_as400_guide.md).
- **Refactoring** — changing the internal structure of code without changing its behavior; the monolith-internal work (branch by abstraction) that precedes extraction (§3.2).

---

## 12. References

1. **Fowler, M.** — *"Strangler Fig Application"*, martinfowler.com bliki, 2004 (updated through 2024). The origin of the strangler fig pattern.
2. **Fowler, M.** — *"BranchByAbstraction"*, martinfowler.com bliki, January 2014; **Hammant, P.** — *"Introducing Branch By Abstraction"*, paulhammant.com, October 2013; trunkbaseddevelopment.com (branch-by-abstraction).
3. **Evans, E.** — *Domain-Driven Design: Tackling Complexity in the Heart of Software*, Addison-Wesley, 2003. Bounded contexts, ubiquitous language, aggregates, subdomains.
4. **Newman, S.** — *Building Microservices*, O'Reilly, 2015. Database per service; the canonical microservices book.
5. **Conway, M.** — *"How Do Committees Invent?"*, Datamation, April 1968. Conway's law.
6. **Stone, B.** — *The Everything Store*, 2013 (the Amazon operating model context for the two-pizza rule).
7. **Brown, S.** — *"Modular Monoliths"*, GOTO Berlin 2018; **Tilkov, S.** — *"Don't jump to microservices"*, 2017 (the anti-hype corpus).
8. **García-Molina, H. & Salem, K.** — *"Sagas"*, ACM SIGMOD, 1987 (the saga pattern's origin).
9. **Hohpe, G. & Woolf, B.** — *Enterprise Integration Patterns*, Addison-Wesley, 2003 (the integration-pattern foundation; see [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md)).
10. **This repository's series** — [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (strangler + ACL mechanics, EIP catalogue), [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) (the target architecture, §14 phased migration), [apache_seata_guide.md](apache_seata_guide.md) (saga engines), [temporal_workflow_guide.md](temporal_workflow_guide.md) (orchestration), [event_stream_processing_guide.md](event_stream_processing_guide.md), [kafka_alternatives_guide.md](kafka_alternatives_guide.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (outbox, delivery semantics), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (CDC/ETL tooling), [distributed_auth_guide.md](distributed_auth_guide.md) (identity), [ibm_as400_guide.md](ibm_as400_guide.md) (legacy platform), [late_arriving_data_guide.md](late_arriving_data_guide.md) (reconciliation reality), [finops_guide.md](finops_guide.md) (cost), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (platform infra), and the banking core guides ([../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/temenos_guide.md](../banking/temenos_guide.md), [../banking/oracle_flexcube_data_model_guide.md](../banking/oracle_flexcube_data_model_guide.md), [../banking/chinese_bank_core_systems_guide.md](../banking/chinese_bank_core_systems_guide.md), [../banking/apache_fineract_guide.md](../banking/apache_fineract_guide.md)) — the migration reality this guide navigates.
