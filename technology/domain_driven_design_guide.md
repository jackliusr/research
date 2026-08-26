# Domain-Driven Design: The DDD Discipline Deep-Dive

*A comprehensive deep-dive on the DDD discipline: the Evans 2003 premise (complexity at the heart of software), the tactical patterns (entities, value objects, aggregates, domain events, repositories, domain services, factories), the strategic design (bounded contexts, context maps, ubiquitous language, subdomains), the integration patterns (anti-corruption layer, shared kernel, customer-supplier, conformist, published language, separate ways, open-host service), the event storming technique, the anemic domain model critique, DDD in banking (lending, payments, trade finance), and a worked example — a Cymbal Bank's context map.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Architecture / Domain-Modeling (technology/)
> **Audience:** Solution architects, enterprise architects, domain modelers, technical leads, and software engineers designing or modernizing complex business systems
> **Last Updated:** August 2026

**Cross-references:** [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) (the **decomposition** strategy built on DDD bounded contexts — Evans 2003 verified there in §2.1 — this guide is the *discipline* behind that strategy, cross-ref heavily), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the **strangler fig** and the **anti-corruption layer** in §6 — the ACL's integration mechanics live there, cross-ref heavily), [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) (the **target microservices architecture** for banking — 14 migration mentions — the DDD-in-banking application of §7–§8, cross-ref heavily), [apache_seata_guide.md](apache_seata_guide.md) (distributed **transactions** and sagas — the consistency mechanics behind multi-context flows), [event_stream_processing_guide.md](event_stream_processing_guide.md) (the **domain events** angle — events as the integration backbone between contexts), [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable **orchestration** for long-running banking workflows), [late_arriving_data_guide.md](late_arriving_data_guide.md) (the data-reconciliation angle — lightly), [distributed_auth_guide.md](distributed_auth_guide.md) (identity across contexts — lightly), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the **legacy cores** the bank's contexts must wrap), [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) and [../banking/hsbc_software_systems_guide.md](../banking/hsbc_software_systems_guide.md) (the **bank-IT context** — Cymbal Bank's real estate as the backdrop for the worked example), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) (the **trading-domain** angle), [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) and [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md) (the **trade-finance** subdomain in practice), [../management/business_case_development_guide.md](../management/business_case_development_guide.md) (the **investment** angle — when DDD is worth funding — lightly).

---

**How to read this guide.** This is the *dedicated* deep-dive on Domain-Driven Design: the discipline itself, not its application in one context. The sibling guides apply DDD — [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) uses bounded contexts as its decomposition strategy, [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) uses the anti-corruption layer as its legacy-integration shield, [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) uses it as the target architecture for banking. This guide is the theory those applications draw on. A fast path through it: read **§1** (the premise), **§2** (the tactical building blocks), **§3** (the strategic boundaries), and **§4** (how contexts integrate) — that is the whole canon — then **§8** (the worked example that ties it to a bank) and **§9** (the one-page summary). Treat **§5** (event storming) as the workshop technique you run *before* designing, **§6** (the anemic model) as the anti-pattern to avoid, **§7** as the banking mapping, **§10** for verification status of every factual claim, and **§11** as the glossary. Cross-references point at the series' sibling guides throughout.

---

## Table of Contents

1. [The DDD Overview](#1-the-ddd-overview)
2. [The Tactical Patterns](#2-the-tactical-patterns)
3. [The Strategic Design](#3-the-strategic-design)
4. [The Integration Patterns](#4-the-integration-patterns)
5. [The Event Storming](#5-the-event-storming)
6. [The Anemic Model](#6-the-anemic-model)
7. [The DDD in Banking](#7-the-ddd-in-banking)
8. [The Worked Example: A Cymbal Bank's Context Map](#8-the-worked-example-a-cymbal-banks-context-map)
9. [The Summary: The Language First](#9-the-summary-the-language-first)
10. [Verification and Claims-Status](#10-verification-and-claims-status)
11. [Glossary](#11-glossary)
12. [References](#12-references)

---

## 1. The DDD Overview

### 1.1 The Evans 2003 Premise (verified)

**Origin.** Domain-Driven Design was coined by **Eric Evans** in his book *Domain-Driven Design: Tackling Complexity in the Heart of Software*, published by **Addison-Wesley in 2003** (ISBN 978-032112521-7). The subtitle is the thesis: the *complexity* that matters is not the technical complexity of frameworks, databases, or infrastructure — it is the complexity **in the heart of the software**, which is the business domain itself. Evans' premise, as restated in the official DDD glossary, is that (1) *for most software projects, the primary focus should be on the domain and domain logic*, and (2) *complex domain designs should be based on a model*. Everything in this guide follows from those two sentences.

**The premise, unpacked.** The software industry has spent decades getting better at the *technical* parts of systems — networking, storage, GUIs, deployment — while the part that actually distinguishes one bank's system from another's, the part that the business pays for, remains the hardest and the least well-understood: **the domain logic**. Evans' insight was that the bottleneck is not a shortage of technology but a shortage of *understanding* — and that understanding must be captured in a **model**: "a system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain" (official glossary definition). The model is not a diagram you draw and discard; it is the conceptual backbone that the design, the code, and the team's language all share. Where the model is absent, code drifts into a procedural tangle that no amount of architecture can rescue — the fate documented in §6 as the anemic model.

**The three pillars.** DDD as a discipline rests on three commitments, each answered by a section of this guide:

- **The model and the language.** A model is only useful if it is *shared* — the **ubiquitous language** (§3.3) binds the team and the business to the model. This is the discipline's first move, and its final word (§9).
- **Strategic design.** Large systems cannot have one unified model; the domain must be **partitioned into bounded contexts** with explicit relationships between them (§3–§4). This is DDD's answer to enterprise scale.
- **Tactical design.** Inside each bounded context, the model is built from a small, sharp set of building blocks — entities, value objects, aggregates, domain events, repositories, domain services, factories (§2).

**The book's structure.** *Domain-Driven Design* is organized in four parts: Part I "Putting the Domain Model to Work" (the model, the ubiquitous language, the layered architecture); Part II "The Building Blocks of a Model-Driven Design" (the tactical patterns — entities, value objects, services, factories, repositories, aggregates); Part III "Refactoring Toward Deeper Insight" (supple design, distillation, deep models); and Part IV "Strategic Design" (contexts, distillation at scale, the large-scale structure — this is the part that **bounded context opens**, per Martin Fowler's reading). Part IV is where DDD grows from a design technique into an enterprise strategy, and it is the part the microservices movement rediscovered: each bounded context is a microservice candidate ([monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §2.1 builds its decomposition strategy on exactly this).

**The follow-on canon.** The second canonical text is Vaughn Vernon's *Implementing Domain-Driven Design* (Addison-Wesley, 2013 — "IDDD"), which opens with strategic design rather than closing with it: Vernon's Chapter 2 details how a domain divides into bounded contexts and Chapter 3 is, per Fowler, "the best source on drawing context maps." Vernon's *Effective Aggregate Design* series (DDD Community, 2011–2012) remains the standard treatment of aggregates (§2.3). Evans himself published a *Domain-Driven Design Reference: Definitions and Pattern Summaries* (2015) — the definitions quoted throughout this guide come from the same lineage as that reference.

### 1.2 The Complexity at the Heart of Software

**Why the domain is where complexity lives.** A bank's payment engine, a trader's risk system, a trade-finance workflow — the hard parts of these systems are not the HTTP calls or the database indexes. The hard parts are the rules: *when can a payment be recalled? what happens to a letter of credit when the shipping documents arrive three days late? how is a syndicated facility's margin re-priced when the reference rate changes?* These rules are the domain, and they are genuinely, stubbornly complex: they are full of exceptions, edge cases, regulatory constraints, and silent assumptions that only domain experts know. Evans' core claim is that **this complexity cannot be managed away with better technology — it must be modeled, deliberately**, with the model built *with* the experts who hold the knowledge.

**Modeling as simplification.** A model is always a simplification — it selects the aspects of the domain that matter for the problem at hand and ignores the rest. The art is choosing the right simplifications: the model must be **strictly useful** (it must solve the problems the software exists to solve), **deep** (it must express the underlying structure, not the surface), and **faithful** (it must mean the same thing to the business and to the code). A good model is a *shared fiction* that lets the business and the software talk about the same thing in the same words — which is precisely what makes the ubiquitous language possible.

**The cost of getting it wrong.** When the model is missing or wrong, the symptoms are familiar to every architect in a bank: business rules scattered across services and stored procedures, the same concept ("customer", "balance", "position") meaning different things in different systems, and a business that cannot explain *its own* system because the software's vocabulary drifted from the business's. DDD's wager is that the up-front cost of modeling — the workshops, the language-building, the aggregate design — is repaid many times over in the only currency that matters at enterprise scale: the ability to change the software when the business changes, without the change cascading across a monolith of incoherent models. (For the cost side of that wager, see [../management/business_case_development_guide.md](../management/business_case_development_guide.md) — the investment logic for when the modeling effort is justified.)

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **The origin** | Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software*, Addison-Wesley, 2003 — the book that coined the term and defined the discipline |
| **The premise** | For most software projects, the primary focus should be the domain and domain logic; complex domain designs should be based on a model (official DDD glossary definition) |
| **The target** | The complexity *in the heart of the software* — the business domain, not the technical infrastructure — which no technology can eliminate, only model |
| **The model** | "A system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain" (Evans) — the shared conceptual backbone of design and code |
| **The language** | The ubiquitous language — one rigorous vocabulary, structured around the model, spoken by business and developers alike (§3.3) |
| **The strategy** | Partition the domain into bounded contexts; map the relationships between them explicitly (§3–§4) — the answer to enterprise scale |
| **The tactics** | The building blocks inside a context: entities, value objects, aggregates, domain events, repositories, domain services, factories (§2) |
| **The technique** | Event storming — the collaborative workshop that surfaces the model and the boundaries before any code is written (§5) |
| **The anti-pattern** | The anemic domain model — domain objects as bags of getters and setters with all logic in services (Fowler, 2003; §6) |
| **The canon** | Evans 2003 (the book); Vernon, *Implementing Domain-Driven Design*, 2013 (the implementation guide); the DDD Community glossary (the definitions); Evans, *DDD Reference*, 2015 (the summaries) |
| **The adoption reality** | DDD is recommended by Microsoft *only* for complex domains where the model pays for itself; it is a discipline for the core domain, not a blanket methodology — and it fails when applied as one (Greg Young, "7 Reasons DDD Projects #FAIL") |

### 1.4 The Layered Architecture (verified)

**The model needs a home.** A domain model cannot live in a tangle of controllers, database mappings, and UI code — it needs a protected layer of its own. Evans prescribes the **layered architecture**, and his description of the layers is precise (quoted in Fowler's anemic-model essay): the **Application Layer** "defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems... kept thin. It does not contain business rules or knowledge, but only coordinates tasks and delegates work to collaborations of domain objects in the next layer down"; the **Domain Layer** (the Model Layer) "is responsible for representing concepts of the business... This layer is the heart of business software." The official glossary defines the **Domain Layer** as "that portion of the design and implementation responsible for domain logic within a LAYERED ARCHITECTURE... where the software expression of the domain model lives."

| Layer | Responsibility (Evans) | What lives there |
|---|---|---|
| **User Interface** | Presenting information to the user and interpreting their commands | Screens, APIs, message handlers, channel adapters |
| **Application** | "Defines the jobs the software is supposed to do"; thin; "does not contain business rules or knowledge, but only coordinates tasks" | Use-case orchestration, transactions, security — delegating to the domain |
| **Domain** | "The heart of business software" — the business concepts, information, and rules | Entities, value objects, aggregates, domain services, domain events, repositories' interfaces |
| **Infrastructure** | Technical capabilities supporting the upper layers | Persistence implementations, messaging, file I/O — *behind* the domain's interfaces |

**The dependency rule.** Each layer may depend only on the layers below it, and the domain layer depends on *nothing* technical — the repository *interface* lives in the domain, its implementation (JPA, JDBC, a legacy file interface) lives in infrastructure. This is the same discipline the post-2003 ecosystem formalized as **hexagonal architecture / ports-and-adapters** (Alistair Cockburn, 2005): the domain model sits at the center, and every external concern — database, message bus, UI — plugs in through an adapter. The rule's payoff is exactly the seam DDD's integration patterns need: if the domain layer does not know what database or core system sits behind the repository, then the database or core system can be *replaced* — which is the DDD half of the strangler-fig migration in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.

### 1.5 The DDD Ecosystem: What Came After 2003 (verified — flagged)

The 2003 book defined the discipline; the ecosystem since then extended it. This guide's focus stays on the canon, but a map of the neighborhood is necessary to place the sibling guides:

- **Vernon's *Implementing Domain-Driven Design* (2013)** — the implementation-focused companion; bounded contexts and context maps from the start; aggregate design rules (§2.3) that became the community standard.
- **CQRS and event sourcing** — Fowler's Command Query Responsibility Segregation bliki (2010s) and the event-sourcing community (Fowler's *Event Sourcing* bliki; Greg Young's work) built on DDD's domain events (§2.4). CQRS splits the read side from the write side; event sourcing stores the events instead of the state. Both are *compatible extensions*, not Evans-2003 core — DDD works without them.
- **Event storming** — Brandolini's workshop technique (§5), the discovery method of the DDD community.
- **Hexagonal architecture** — Cockburn's ports-and-adapters, the structural expression of DDD's layered rule (§1.4).
- **The DDD Crew and modeling tooling** — the open-source DDD Crew (GitHub) publishes the *Bounded Context Canvas*, the *Aggregate Canvas*, and modeling-process guidance; Context Mapper is a modeling framework for strategic DDD. These formalize and tool the canvases of §3–§4.
- **DDD and microservices** — the movement that made DDD mainstream: Sam Newman's *Building Microservices* and Microsoft's architecture guidance both treat bounded contexts as the primary microservice-decomposition criterion — the application documented in [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §2.1 and [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md).
- **The criticisms** (flagged — verified as existing, not adjudicated): DDD is *hard* — it demands sustained collaboration with domain experts, disciplined modeling, and teams with the authority to draw boundaries. Greg Young's "7 Reasons DDD Projects #FAIL" (NDC 2010, referenced on dddcommunity.org) catalogs the failure modes: starting with DDD where there is no complex domain, treating the patterns as a checklist, and skipping the strategic half. Microsoft's application-architecture guidance recommends DDD *only for complex domains* where the model pays for itself. The honest reading: DDD is a discipline for the **core domain** (§3.4), not a blanket methodology — and the investment logic is exactly that of [../management/business_case_development_guide.md](../management/business_case_development_guide.md).

---

## 2. The Tactical Patterns

**The role of the tactical patterns.** Where strategic design (§3) decides *where the boundaries are*, the tactical patterns decide *what lives inside them*. They are the building blocks of a model-driven design — the small, sharp vocabulary with which the domain model is expressed in code. Evans calls these the *building blocks* of the domain layer (Part II of the 2003 book); Vernon's IDDD and the modern DDD community treat them as the code-level toolkit of a bounded context. A team that masters these seven patterns can express almost any business rule in the model itself — and a team that skips them ends up with the anemic model of §6. The tactical patterns are:

1. **Entities** — objects with identity
2. **Value objects** — objects with attributes but no identity
3. **Aggregates** (with their **roots**) — clusters of objects treated as a unit
4. **Domain events** — things that happened, expressed as model objects
5. **Repositories** — the retrieval interface over persistence
6. **Domain services** — operations that belong to no object
7. **Factories** — encapsulated creation logic

### 2.1 The Entities (verified)

**Definition (Evans' glossary):** "An object fundamentally defined not by its attributes, but by a thread of continuity and identity." An entity is a domain object whose *identity* matters more than its properties. Two Customer objects with the same name, address, and phone number are *not* the same customer — what makes them distinct is a stable identity (a customer ID, a passport number, an account number) that survives changes to every attribute.

**The practical rules.** (1) Give each entity a stable identity — a natural key (account number, ISIN, vehicle VIN) or a surrogate ID — and define equality on identity, never on attributes. (2) Entities may change state over their lifecycle: a Loan is the same loan when its outstanding balance changes, because its identity is unchanged. (3) Protect the entity's invariants: an entity is not a passive data holder — it exposes behavior that enforces its own rules (`loan.approve()` enforces the approval invariants; it does not let a service set `loan.status = "APPROVED"` from outside). (4) Model *state transitions* explicitly where the domain cares about them — a trade, a facility, a payment each have lifecycles with legal meaning. In banking the entity is the workhorse of every context: the Customer, the Account, the Trade, the Facility, the LetterOfCredit are all entities because a bank must track them *over time*, through state changes, by identity.

### 2.2 The Value Objects (verified)

**Definition (Evans' glossary):** "An object that describes some characteristic or attribute but carries no concept of identity." A value object is defined entirely by its attributes; two value objects with the same attributes are interchangeable. The canonical example from the glossary tradition: two identical business cards — you only care about the information on the card, never about "which" card it is. **Immutability is the defining discipline**: a value object is created whole and never changes; any "change" produces a new value object. This makes value objects naturally safe to share, to compare, and to use as keys — and it is why they compose so well with entities: an entity holds value objects as its attributes (a Customer entity holds an Address value object).

**The practical rules.** (1) Model as a value object anything that is conceptually a *measurement, description, or amount* — Money, Currency, DateRange, Address, AccountNumber, IBAN, InterestRate. (2) Make them immutable; provide `equals()` on full value equality. (3) Give them behavior: a Money value object should know how to add, subtract, and convert — arithmetic on money is *domain logic* and belongs in the Money object, not scattered across services. (4) Prefer value objects over primitives: a `LoanAmount` of type Money with a Currency is a whole concept; two primitives (`BigDecimal amount`, `String currency`) are a bug waiting to happen. The *whole value* principle — model a single, complete concept as one object — is an Evans staple of supple design. **Money deserves special attention in banking**: currency-aware arithmetic, rounding rules, and decimal precision are exactly the kind of rule that an anemic design scatters and a value object encapsulates.

### 2.3 The Aggregates and the Roots (verified)

**Definition (Evans' glossary):** "A cluster of associated objects that are treated as a unit for the purpose of data changes. External references are restricted to one member of the AGGREGATE, designated as the root. A set of consistency rules applies within the AGGREGATE'S boundaries."

**The problem aggregates solve.** Entities rarely stand alone: a Loan aggregates its Schedules, its Collateral, its Guarantors; an Order aggregates its LineItems. If every object is independently modifiable, the consistency rules that span them (the loan's total exposure, the order's total amount) have no home — every caller must enforce them, and eventually nobody does. The aggregate draws a **consistency boundary**: the cluster of objects inside must change *as a unit*, and all change flows through the **aggregate root** — the single entity that is the cluster's representative to the outside world.

**The practical rules.** (1) Choose the root: one entity inside the cluster is the root — the only member external objects may reference. (2) Access *through* the root only: you cannot reach a LineItem without going through its Order, and you cannot mutate a LineItem except by a method on Order. (3) The root enforces the cluster's invariants: every operation on the aggregate starts at the root, which validates the rules that hold the cluster together. (4) **Reference other aggregates by identity only**, not by object reference — an aggregate holds another aggregate's ID, not a pointer, so that aggregates can live in different stores or services. (5) **Keep aggregates small** — Vernon's central guidance from *Effective Aggregate Design* (2011–2012): large aggregates are the root cause of transaction contention, lost updates, and scaling pain. Rule of thumb: one aggregate = one transaction; if two aggregates must change together in one transaction, they were probably one aggregate. (6) In a microservices estate, the aggregate boundary is the natural service-boundary candidate — this is the link to [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §2 and the reason the oracle banking architecture ([../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md)) is built around them.

**The banking consequence.** Consider a payment: the Payment aggregate (root: Payment) with its Legs, Fees, and StatusHistory as members. If a naive design lets any service mutate `PaymentLeg.amount` directly, the fee recomputation and the audit trail silently rot. The aggregate keeps the payment's parts honest — and in a distributed world, the aggregate boundary is exactly where the "one transaction" of [apache_seata_guide.md](apache_seata_guide.md) and the saga fallback live.

### 2.4 The Domain Events (verified)

**Definition.** A domain event is a model object that records something *that happened* in the domain, expressed in the past tense: `PaymentExecuted`, `LoanDisbursed`, `LimitBreached`, `DocumentReceived`. The term entered the canon through Evans (the book's later editions and the DDD Reference discuss events; Part IV's model of integration leans on them) and was sharpened by Vernon (IDDD treats domain events as a first-class building block) and the CQRS/event-sourcing community. A domain event carries the facts the past tense implies: the identity of the aggregate it happened to, the timestamp, and the payload the domain cares about (but *not* internal implementation detail).

**The practical rules.** (1) Publish a domain event when the aggregate's state changes in a way that *other parts of the domain* care about — the event is the domain's way of saying "something important happened, here are the facts." (2) Name it in the ubiquitous language, past tense: the business says "the payment was executed", so the event is `PaymentExecuted`. (3) Keep the event small and meaningful — a domain event is not a full state snapshot. (4) Persist/publish reliably — in practice this means the **transactional outbox**: write the event in the same transaction as the state change, then publish asynchronously (the mechanics are covered in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §9 and [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)). (5) Distinguish **domain events** (inside a bounded context, driving that context's own logic — light payloads) from **integration events** (across contexts — richer payloads, versioned, published as the *published language* of §4.5); the distinction is standard in the modern canon.

**The strategic role.** Domain events are the natural *glue* between bounded contexts: when the Lending context publishes `FacilityDrawn`, the Risk context subscribes. This is the event-driven angle of [event_stream_processing_guide.md](event_stream_processing_guide.md) applied to DDD — and it is why the modern reading of DDD pairs so naturally with event-driven architectures, CQRS, and event sourcing (all flagged in §10 as ecosystem extensions rather than Evans-2003 core).

### 2.5 The Repositories (verified)

**Definition (Evans' glossary):** "A mechanism for encapsulating storage, retrieval, and search behavior which emulates a collection of objects." A repository is the domain's interface to persistence: it lets the model *retrieve and store aggregates* without knowing anything about SQL, ORMs, or document stores. The repository hands back a fully reconstructed aggregate root and accepts a whole aggregate for saving — the client never issues queries or statements against the model's internals.

**The practical rules.** (1) One repository per **aggregate root** — never per table, never per entity. `LoanRepository.findByAccountNumber()` returns a `Loan` aggregate, schedules and all. (2) The repository interface is written in the domain's language (`findByCustomerId`, `save`) — the *implementation* (JPA, JDBC, Mongo, a legacy file interface) lives behind it. (3) The repository makes the persistence technology replaceable — which is precisely why DDD and the strangler-fig migration of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) work so well together: the repository is the seam where a legacy core can sit behind a modern domain model while the migration proceeds. (4) Do not put query logic in the repository that belongs to the domain; do not leak the ORM into the domain layer.

### 2.6 The Domain Services (verified)

**Definition (Evans' glossary):** "An operation offered as an interface that stands alone in the model, with no encapsulated state." A domain service holds **domain logic that does not naturally belong to any entity or value object** — operations that span multiple aggregates, or that involve external parties: a funds-transfer operation touching two accounts, a limit-checking operation consulting positions and limits, an FX-conversion operation across currencies. The service is stateless; it *orchestrates* the entities and value objects, letting them enforce their own invariants.

**The practical rules.** (1) Name it after the *operation* in the ubiquitous language (`FundsTransferService`, `LimitCheckService`, `CollateralValuationService`). (2) Keep it thin — if a service is doing the *entity's* work, the entity is anemic (§6); the service exists only for logic with no natural object home. (3) Contrast with the **application service** (Fowler's *Service Layer*): the application layer "defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems... kept thin... does not contain business rules or knowledge, but only coordinates tasks" (Evans, quoted in Fowler's anemic-model essay). Domain services live in the domain layer and contain domain logic; application services live above and contain use-case coordination. The anemic model collapses this distinction — all logic migrates up into services, and the domain layer dies (§6).

### 2.7 The Factories (verified)

**Definition (Evans' glossary):** "A mechanism for encapsulating complex creation logic and abstracting the type of a created object for the sake of a client." When constructing an aggregate is complicated — invariants to establish, children to build, defaults to apply, IDs to assign — the construction logic should not sit in the client and should not sit in the constructor. A factory encapsulates it: `LoanFactory.create(application, pricing, limit)` returns a fully formed, valid `Loan` aggregate, root and all.

**The practical rules.** (1) Use a factory when creation is more than trivial assignment — when building the object requires enforcing invariants or assembling a cluster. (2) The factory's job is to hand back a *valid* aggregate: either the aggregate is fully constructed and consistent, or creation fails — no half-built objects. (3) Distinguish the factory from the repository: the factory *creates new* objects; the repository *retrieves existing* ones. (4) Keep factory methods named in the ubiquitous language — `open`, `draw`, `disburse`, not `construct` — so the code reads like the business story.

### 2.8 The Tactical Patterns in Code (a worked mini-example)

The patterns are easiest to hold when seen together in one small model. Consider the **Facility** aggregate of §8, expressed in minimal pseudo-code — the point is the *shape*, not the language:

```
// VALUE OBJECT: money with currency, immutable, knows its own arithmetic
class Money {
  final BigDecimal amount; final Currency currency;
  Money add(Money other) { /* currency-checked addition */ }
  Money subtract(Money other) { /* currency-checked subtraction */ }
  // equals() on amount AND currency
}

// ENTITY: the aggregate root — identity, behavior, invariants
class Facility {
  final FacilityId id;                       // identity: survives state change
  final CustomerId borrower;                  // reference to OTHER aggregate: by ID only
  Money committedAmount;                      // value objects as attributes
  List<Drawdown> drawdowns;                   // members of the aggregate
  List<Covenant> covenants;
  Collateral collateral;

  Drawdown draw(Money amount, ValueDate valueDate) {
    if (amount.exceeds(availableCommitment()))  // the invariant, enforced HERE
      throw new CovenantBreach(...);
    Drawdown d = DrawdownFactory.create(this, amount, valueDate); // factory
    drawdowns.add(d);
    DomainEvents.raise(new FacilityDrawn(id, amount, valueDate)); // domain event
    return d;
  }
  Money availableCommitment() { /* utilization math over drawdowns + collateral */ }
}

// DOMAIN SERVICE: logic with no natural object home (spans aggregates / external)
class LimitCheckService {
  boolean check(Facility facility, Customer customer, RiskModel risk) { ... }
}

// REPOSITORY: interface in the domain layer, implementation in infrastructure
interface FacilityRepository {
  Facility findById(FacilityId id);
  void save(Facility facility);
}
// (implementation: JPA, or an ACL over the legacy core — §4.1)
```

**Reading the example.** The aggregate root `Facility` enforces its own invariant (no drawdown beyond available commitment) — no service, controller, or client can bypass it, because the only way in is through `draw()`. `Money` is a value object: immutable, self-arithmetic, no identity — the exact rules of §2.2. The `DrawdownFactory` encapsulates creation; `LimitCheckService` holds the logic that spans aggregates; `FacilityRepository` hides persistence behind a domain-language interface; `FacilityDrawn` is the domain event announcing the change to anyone who cares. Contrast this with the anemic version of §6: `Facility` as getters and setters, a `FacilityService` doing the arithmetic, and the invariant enforced — if at all — in some service nobody can find. The DDD version is not more code; it is *the same rules, placed where they can be found, tested, and trusted*.

### 2.9 The Tactical Table

| Pattern | Definition (Evans' glossary) | Notes |
|---|---|---|
| **Entity** | "An object fundamentally defined not by its attributes, but by a thread of continuity and identity" | Identity survives state change; equality on identity; enforces its own invariants; the workhorse of banking (Customer, Account, Trade, Facility) |
| **Value Object** | "An object that describes some characteristic or attribute but carries no concept of identity" | Immutable; equality on attributes; whole-value modeling (Money, IBAN, DateRange); safer and more expressive than primitives |
| **Aggregate + Root** | "A cluster of associated objects that are treated as a unit for the purpose of data changes"; external references restricted to the designated root | Consistency boundary; access only through the root; reference other aggregates by ID; **keep aggregates small** (Vernon) — one aggregate ≈ one transaction; the microservice-boundary candidate |
| **Domain Event** | An object recording something that happened in the domain, past tense, with the facts the domain cares about | Published when state changes that others care about; distinguish domain events (intra-context, light) from integration events (inter-context, versioned); needs the transactional outbox for reliability |
| **Repository** | "A mechanism for encapsulating storage, retrieval, and search behavior which emulates a collection of objects" | One per aggregate root; interface in domain language, implementation behind it; the seam that makes legacy cores swappable |
| **Domain Service** | "An operation offered as an interface that stands alone in the model, with no encapsulated state" | Stateless; hosts logic spanning aggregates; must stay thin or the model turns anemic; distinct from the thin application/service layer above it |
| **Factory** | "A mechanism for encapsulating complex creation logic and abstracting the type of a created object for the sake of a client" | Encapsulates non-trivial creation; returns fully valid aggregates; creates new objects (vs repository retrieving existing) |

### 2.10 When the Tactical Patterns Don't Apply (verified — the honest scope)

The tactical patterns are powerful and *not* universally appropriate. The canon and the community are explicit about the boundary:

- **Simple domains.** Where the logic is thin — CRUD over reference data, a record-keeping screen, a commodity workflow — the model is not the problem and the patterns add ceremony without payoff. Evans' layered model and Fowler's *Patterns of Enterprise Application Architecture* both acknowledge that **transaction scripts** are the right tool for simple domains; the domain model earns its keep only where the logic is complex enough to justify it.
- **The generic subdomains of §3.4.** Identity, notifications, document storage: buy the product, wrap it with an ACL, and move on — modeling a commodity as an aggregate is how the generic domain starves the core.
- **The greenfield trap.** DDD is most valuable *in a complex domain*; adopting it where the domain is a thin CRUD shell over a database produces the opposite of the intended effect — a model that models nothing. Greg Young's failure catalogue (NDC 2010) starts with exactly this: "DDD where there is no domain."
- **The discipline requirement.** The patterns fail silently when the organization cannot supply the prerequisites: access to domain experts, teams with boundary authority, and the willingness to evolve the model. Without the experts, the "ubiquitous language" is invented jargon; without boundary authority, the context map is fiction; without evolution, the model ossifies into the very monolith DDD exists to avoid.

The honest scope, in one line: **use the tactical patterns where the rules are complex and the business differentiates on them — the core domain — and use simpler tools everywhere else.** This is the investment discipline of [../management/business_case_development_guide.md](../management/business_case_development_guide.md) applied to modeling effort.

---

## 3. The Strategic Design

**The role of the strategic patterns.** The tactical patterns answer "how do we build the model?"; the strategic patterns answer the prior question: **"how many models are there, where are their boundaries, and how do they relate?"** DDD's strategic answer — developed in Part IV of Evans' 2003 book and brought to the fore by Vernon's IDDD — is that a large system must *not* have one unified model. Evans' own words, quoted by Fowler: "total unification of the domain model for a large system will not be feasible or cost-effective." The strategic design is the discipline of drawing the boundaries well and making the relationships between the resulting contexts explicit.

### 3.1 The Bounded Contexts (verified)

**Definition (Evans' glossary):** "The delimited applicability of a particular model. BOUNDING CONTEXTS gives team members a clear and shared understanding of what has to be consistent and what can develop independently." A **bounded context** is the boundary inside which one domain model — its terms, its rules, its ubiquitous language — applies with full force and internal consistency. Outside the boundary, the same words may mean different things.

**Why boundaries exist.** The same word means subtly different things to different parts of a large organization — Fowler's electricity-utility example: "meter" meant the grid-to-location connection to one group, the grid-to-customer connection to another, the physical device to a third. "Customer" and "Product" are the perennial polysemes. In conversation, humans smooth these over; in software, they are contradictions that corrupt the model. The bounded context makes the polysemy *explicit*: the Payments context's "customer" (an account holder, a beneficiary) is a different concept from the Onboarding context's "customer" (a KYC-verified legal entity) — and the mapping between them is a first-class design artifact, not an accident.

**The practical rules.** (1) Draw the boundary where the *language* changes — a new ubiquitous language means a new context (the dominant factor is human culture and team organization, per Fowler). (2) Each context owns its model, its data, and (in microservices) its service — this is the decomposition criterion of [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §2.1. (3) A context is roughly the size of one team's cognitive capacity — Conway's law made explicit. (4) Do not try to unify the models across contexts: the point is that they *differ*, deliberately, with explicit translation at the seams (§4). (5) Contexts may overlap in the *words* they use but never in the *ownership* of a concept: each concept in each context has exactly one owner. The "bubble context" (Evans' technique for grafting DDD onto a legacy system from the inside) is the strategic version of the strangler fig — see [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.

### 3.2 The Context Maps (verified)

**Definition (Evans' glossary):** "A representation of the BOUNDED CONTEXTs involved in a project and the actual relationships between them and their models." A **context map** is the drawing — and the discipline of drawing it — that makes the system's boundaries and their relationships visible: which contexts exist, who talks to whom, and *through which pattern* (§4). Where the bounded context answers "where does this model end?", the context map answers "and what happens at the seam?"

**The practical rules.** (1) Draw the map *honestly* — the map shows the actual relationships (including the Big Ball of Mud you wish you didn't have), not the idealized ones. Evans is explicit that the context map should reflect reality, warts and all, because you can only improve what you have named. (2) Name every relationship with its pattern: the arrow between two contexts is an ACL, a conformist, a customer-supplier, a shared kernel — never a bare "integrates with." (3) Keep the map small enough to be a conversation piece: one diagram per project, updated as the system evolves. Vernon's IDDD Chapter 3 is the standard treatment of drawing context maps. (4) In a microservices estate the context map *is* the architecture diagram — the relationship arrows become API contracts, event channels, and data flows; this is the bridge to [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) and the working example of §8.

### 3.3 The Ubiquitous Language (verified)

**Definition (Evans' glossary):** "A language structured around the domain model and used by all team members to connect all the activities of the team with the software." The **ubiquitous language** is the discipline's keystone: one rigorous vocabulary, built from the model, used *everywhere* — in code, in meetings, in tickets, in documents, in tests — by *everyone* — business experts, developers, testers, product owners. Fowler's summary: "a common, rigorous language between developers and users... based on the Domain Model... hence the need for it to be rigorous, since software doesn't cope well with ambiguity."

**The practical rules.** (1) Build it with the domain experts — the language is discovered, not invented; if a term is awkward or wrong, the experts say so, and the model changes (Evans: "Domain experts should object to terms or structures that are awkward or inadequate to convey domain understanding; developers should watch for ambiguity or inconsistency that will trip up design"). (2) Use it pervasively — "by using the model-based language pervasively and not being satisfied until it flows, we approach a model that is complete and comprehensible, made up of simple elements that combine to express complex ideas" (Evans). (3) Let it evolve — the language and the model change together as understanding deepens; a frozen language is a frozen (wrong) model. (4) Enforce it in code — the class names, method names, and event names *are* the language; when the code says `approveLoan` and the business says "disburse", the language has broken and the code is wrong. (5) It is scoped per bounded context — each context has its own dialect; the map (§3.2) records the translations.

### 3.4 The Subdomains: Core, Supporting, Generic (verified)

**Definition (Evans' glossary — Core Domain):** "The distinctive part of the model, central to the user's goals, that differentiates the application and makes it valuable." The **subdomain** framing (formalized in the DDD Reference and the post-2003 canon; Vernon and the DDD community use it as the problem-space counterpart to the bounded context's solution space) classifies every part of the business domain by *strategic value*:

- **The core domain** — where the business *differentiates itself*: the thing the company does that competitors can't easily copy, and the thing the software must therefore get *right*. For a bank this is the pricing engine, the credit decision, the risk model, the treasury position — not the CRM, not the invoice system. **The core domain is where DDD effort belongs**: the best modeling, the best engineers, the most careful design. Evans' *distillation* is the act of separating the core from the rest so it can be seen and invested in clearly.
- **The supporting domain** — necessary but not differentiating: the business *needs* it to operate (settlement messaging, collateral management, regulatory reporting), but it is not where the company wins. Build it well, but with standard effort — and consider off-the-shelf software.
- **The generic domain** — needed, but entirely undifferentiated: identity management, notifications, document storage, authentication. Buy it, use a standard package, or build it with commodity effort — *never* let the generic domain consume the attention the core deserves.

**The practical rules.** (1) Classify every subdomain explicitly — the classification is a strategic decision, made with the business, not a technical one. (2) Align investment with classification: the core gets the modeling workshops, the aggregate design, the best people; the generic gets a vendor product behind a repository (§2.5) or an ACL (§4.1). (3) Do not confuse subdomain and bounded context: subdomains are *problem space* (the business's structure); bounded contexts are *solution space* (the model's boundaries). Ideally they align — one context per subdomain — but a subdomain can span contexts and a context can serve several subdomains; the map (§3.2) records the truth. (4) This classification is the DDD lens on the investment question of [../management/business_case_development_guide.md](../management/business_case_development_guide.md): DDD's expensive modeling effort is justified precisely where the subdomain is core.

### 3.5 Distillation and the Large-Scale Structure (verified)

**Distillation.** Evans' *distillation* is the act of concentrating the model on what matters: "the abstraction of key aspects in a model, or the partitioning of a larger system to bring the CORE DOMAIN to the fore" (glossary). The distillation techniques from Part III/IV of the 2003 book:

- **The domain vision statement** — one paragraph describing what the core domain does and how it creates value; the anchor that keeps every modeling decision pointed at the core.
- **The highlighted core** — a written description of the core domain's most essential concepts and their relationships, the first thing a new team member reads.
- **The segregated core** — *in code*: refactor the core into its own module/package/service so it can be developed and reasoned about without the supporting noise. In a microservice estate this is the argument for the core's services being the best-engineered ones.
- **The abstract core** — identify the deep, general concepts (the *interfaces* the whole model rests on) and express them as abstract interfaces and superclasses, so the model's conceptual skeleton is visible.
- **Cohesive mechanisms** — extract generic, mechanical computations (formula engines, rate curves, date arithmetic) out of the core into separate mechanisms with clear interfaces — the DDD name for "don't let the plumbing hide the business."
- **The generic subdomain** — the §3.4 classification applied: buy, borrow, or build generically everything that is not the core.

**The large-scale structure.** For very large systems, Evans adds the **large-scale structure**: "a set of high-level concepts, rules, or both that establishes a pattern of design for an entire system... a language that allows the system to be discussed and understood in broad strokes" (glossary). His structures include the **system metaphor** (a shared analogy that orients the whole design), **responsibility layers** (a vertical layering of the system's responsibilities so dependencies flow one way — the strategic cousin of §1.4's layers), the **knowledge level** (a structure where the rules themselves are data), and **evolving order** (the discipline of applying structure only where it earns its keep, and letting the structure evolve with the system). The warning attached: a large-scale structure that does not fit the domain is worse than none — "evolving order" is the anti-dogma principle.

### 3.6 The Strategic Table

| Pattern | Definition | Notes |
|---|---|---|
| **Bounded Context** | "The delimited applicability of a particular model" (Evans) | The boundary where one model, one ubiquitous language applies; drawn where language/team/culture changes; owns its model and data; the microservice candidate |
| **Context Map** | "A representation of the Bounded Contexts involved in a project and the actual relationships between them and their models" (Evans) | Drawn honestly, naming every seam with its integration pattern; Vernon's IDDD ch. 3 is the reference treatment; the architecture diagram of a service estate |
| **Ubiquitous Language** | "A language structured around the domain model and used by all team members to connect all the activities of the team with the software" (Evans) | Built with experts, used everywhere, evolves with the model, enforced in code; scoped per context |
| **Core Domain** | "The distinctive part of the model, central to the user's goals, that differentiates the application and makes it valuable" (Evans) | Where DDD effort belongs; the pricing, credit, risk, treasury of a bank; protected by distillation |
| **Supporting Domain** | A subdomain the business needs but that does not differentiate it | Standard effort, standard design; often off-the-shelf software behind a seam |
| **Generic Domain** | A needed but undifferentiated subdomain | Commodity effort: buy or build lightly; identity, notifications, storage — never allowed to starve the core |

---

## 4. The Integration Patterns

**The seam language.** Evans' Part IV defines a small set of **context-mapping patterns** — the vocabulary for the relationships between bounded contexts (the arrows on the context map). These are the DDD-specific half of the integration story; the mechanical half (messaging, file transfer, the four integration styles) lives in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md), and the EIP catalogue there is the transport-level complement to the model-level patterns here. Each pattern is a *deliberate choice* about how much translation, coupling, and independence two contexts will have. The full Evans set is nine patterns (Partnership, Shared Kernel, Customer-Supplier, Conformist, Anticorruption Layer, Open Host Service, Published Language, Separate Ways, and the Big Ball of Mud); the seven below are the ones that do the real work, with the other two noted where relevant.

### 4.1 The ACL — Anti-Corruption Layer (verified — cross-ref [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md))

**Evans' definition:** "create an isolating layer to provide your system with functionality of the upstream system in terms of your own domain model." The **anti-corruption layer** (ACL) is the translation shield: it sits between your context and another (typically a legacy or foreign) system, and it *translates* — the foreign system's model, terms, and data structures are converted into *your* context's model at the boundary, so that the foreign system's ugliness never leaks into your domain. Your model stays pure; the ACL absorbs the corruption.

**When to use it.** Whenever you must integrate with a system whose model you do not control and do not want to adopt — the legacy core in COBOL, a vendor platform, another bank's system. In banking this is *the* pattern of modernization: the new Lending context does not speak the core's record layouts; the ACL between them does. Its migration mechanics — where it sits in the strangler fig, how it exposes the legacy behind a clean interface — are detailed in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6, and the repository seam of §2.5 is the classic place an ACL is implemented. The ACL is the *defensive* pattern: it assumes the upstream model is wrong for you and refuses to let it in.

### 4.2 The Shared Kernel (verified)

**Evans' definition:** "Designate with an explicit boundary some subset of the domain model that the teams agree to share. Keep this kernel small." The **shared kernel** is a deliberate exception to context independence: two (or more) contexts agree to *share a small part of their model* — the concepts so fundamental that duplicating them would be worse than sharing them — under joint ownership and with explicit change control. Both teams must pass CI before either can change it.

**When to use it.** Only between teams that collaborate closely and succeed or fail together (it pairs with Partnership). In a bank, a shared kernel might hold the `Money` value object or the `Counterparty` identity concept shared by the Trading and Risk contexts. The rule is always the same: **keep it small** — a shared kernel that grows is a shared monolith in the making, and the boundary that was supposed to give independence silently disappears.

### 4.3 The Customer-Supplier (verified)

**Evans' definition:** "Establish a clear customer/supplier relationship between the two teams," when "two teams are in [an] upstream-downstream relationship." When context A (upstream, the *supplier*) provides data or behavior that context B (downstream, the *customer*) depends on, the relationship is made explicit and contractual: the supplier's team is accountable to the customer's needs, the customer's team negotiates as a customer would — with requirements, priorities, and acceptance criteria. The supplier's roadmap serves the customer's needs because the customer's success is, in part, the supplier's obligation.

**When to use it.** The classic bank case: the Payments context (supplier) provides transaction data to the Reconciliation and Regulatory Reporting contexts (customers). Making it a customer-supplier relationship converts an informal "we'll get to your requirements when we can" into a negotiated, prioritized commitment — without the full rigidity of a shared kernel and without the defensive cost of an ACL. The DDD glossary and Evans' canon place it as the *cooperative* middle ground between Partnership (tight) and Conformist (submissive).

### 4.4 The Conformist (verified)

**Evans' definition:** "Eliminate the complexity of translation [...] choosing conformity enormously simplifies integration," when a custom interface for a downstream subsystem isn't likely to happen. The **conformist** is the pragmatic surrender: when the upstream team has no interest in accommodating you (and you cannot make them), you *conform* — you adopt their model, their terms, their interface, and you do the translation in your head (or not at all). It is the cheapest integration and the most coupling: your model is constrained by theirs.

**When to use it.** When the upstream is a vendor, a regulator's schema, or a system so entrenched that negotiation is futile — and the downstream's needs are simple enough that the foreign model is tolerable. A bank's regulatory-reporting context conforming to the regulator's message formats is a legitimate conformist; the same context conforming to the legacy core's internal record layout is usually a *symptom* — the place where an ACL (§4.1) would have been the honest choice. The decision between conformist and ACL is exactly the decision of how much the foreign model is allowed to cost you.

### 4.5 The Published Language (verified)

**Evans' definition:** "a well-documented shared language that can express the necessary domain information as a common medium of communication." The **published language** is a *documented, versioned* interchange format — the common medium that multiple contexts (or external parties) can speak without sharing a model or a team: ISO 20022 for payments, FIX for trading, SWIFT MT/MX messages, a well-versioned REST/event contract. It is the *what* of integration; the open-host service (§4.7) is the *how*.

**When to use it.** Whenever integration spans contexts that cannot or should not share a model — which in banking is *most* interbank and cross-context integration: SWIFT messages are the published language of the payments industry; ISO 20022 is its successor; FIX is the published language of the trading front office; ISO 15022 for securities. The published language is what makes the conformist bearable (you conform to a *standard*, which is different from conforming to a stranger's internal model) and what makes the open-host service worth publishing. Note the deliberate ambiguity-tolerance the glossary demands: a published language "can express the necessary domain information" — it is not the full model, just the message that travels.

### 4.6 The Separate Ways (verified)

**Evans' definition:** "a bounded context [with] no connection to the others at all, allowing developers to find simple, specialized solutions within this small scope." **Separate ways** is the honest acknowledgement that some contexts have no integration worth paying for: two teams with no shared flow, no shared data, no shared success should not be forced to integrate. The cost of a seam — a translation layer, a contract, a shared kernel to maintain — is real, and when the value is zero, the seam is waste.

**When to use it.** More often than architects admit. In a bank, the internal reporting tool and the marketing-site context share almost nothing; a hand-built integration between them would be pure overhead. The pattern is also the strategic justification for *not* building a single canonical model of everything — the enterprise-data-model dream that Evans explicitly rejects ("total unification of the domain model for a large system will not be feasible or cost-effective"). The discipline is to name the separation deliberately on the context map, rather than leaving it to chance.

### 4.7 The Open-Host Service (verified)

**Evans' definition:** "a protocol that gives access to your subsystem as a set of services," when integrating one subsystem with many others makes custom translations between subsystems infeasible. The **open-host service** is the *generous* pattern: instead of letting every consumer translate your model into theirs, you publish *your* model as a set of services — a stable, versioned, documented API — and each consumer uses it directly. Combined with a published language (§4.5), it becomes the pattern of API-first design: the context publishes its operations in a shared, documented medium; consumers integrate at commodity cost.

**When to use it.** For contexts that many others consume: the Customer context serving a stable customer API to twenty other contexts; the Payments context publishing payment-status services. The open-host service is the DDD name for what the API-first movement calls a productized API — and it is the model-level half of the integration styles in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §2 (RPC/API style) and the service contracts of [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md). The two remaining Evans patterns round out the set: **Partnership** (two teams coordinating planning and integration jointly — the organizational twin of the shared kernel) and the **Big Ball of Mud** (the honest name for a legacy context with no discernible boundaries — the pattern you map first and fix last, per Foote & Yoder's 1999 paper, and the reality of [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)'s legacy cores).

### 4.8 Partnership, the Big Ball of Mud, and Choosing the Seam

**Partnership (verified).** Evans' definition: "forge a partnership between the teams in charge of the two contexts. Institute a process for coordinated planning of development and joint management of integration," when "teams in two contexts will succeed or fail together." Partnership is the organizational pattern: two teams that share success coordinate their plans and manage the integration jointly — no formal contracts, just a shared fate and the processes that honor it. It pairs naturally with the **shared kernel** (§4.2); the risk is the same one — partnership that outlives its usefulness keeps two teams chained together after their contexts have diverged.

**The Big Ball of Mud (verified).** Foote and Yoder's 1999 paper gave DDD its honest name for the legacy reality: "a boundary around the entire mess" when no real boundaries can be found surveying an existing system. Evans' counsel is that the Big Ball of Mud *is* a legitimate entry on the context map — naming it is the first step to containing it — and the ACL (§4.1) is the instrument of containment. This is the mapped reality of every bank's legacy core ([../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)) and the reason the strangler fig of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6 starts with a map that dares to draw the mud.

**Choosing the seam — the decision frame.** With nine patterns, the map's arrows are a decision problem. The questions that decide the pattern:

- *Do the contexts share a fate?* → **Partnership** (and maybe a small **shared kernel**).
- *Is there a clear upstream/downstream dependency?* → **Customer-Supplier** if the upstream will negotiate; **Conformist** if it will not and the cost of translation exceeds the cost of conformity; **ACL** if you must keep your model pure regardless.
- *Do many contexts consume one?* → **Open-Host Service**, ideally over a **published language** (a standard, or a contract you publish).
- *Does a standard exist?* → **Published Language** — ISO 20022, FIX, SWIFT, a regulator's schema — and conform to *it*, which is different from conforming to a stranger's internal model.
- *Is the value of the seam genuinely zero?* → **Separate Ways** — and say so on the map.
- *Is the other side an unmappable legacy mess?* → **ACL** around it, mapped as the **Big Ball of Mud**.

The table that follows is the summary; the worked example (§8) applies the frame to a whole bank.

### 4.9 The Integration Table

| Pattern | Mechanism | Use |
|---|---|---|
| **Anti-Corruption Layer (ACL)** | A translation shield between contexts; foreign model converted to yours at the boundary | Integrating with legacy cores, vendors, or any system whose model you refuse to adopt; the modernization default — cross-ref [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6 |
| **Shared Kernel** | A small, explicitly bounded, jointly owned subset of the model, with joint change control | Close teams sharing fundamental concepts (Money, Counterparty identity); keep it small or it becomes a monolith |
| **Customer-Supplier** | Explicit upstream/downstream relationship with negotiated, prioritized commitments | Supplier contexts serving dependent customer contexts (Payments → Reconciliation); cooperative middle ground |
| **Conformist** | Adopt the upstream's model wholesale; no translation | Un-negotiable upstream (regulator schemas, vendors) with simple downstream needs; the cheap-but-coupled option |
| **Published Language** | A documented, versioned interchange format as the common medium (ISO 20022, FIX, SWIFT) | Cross-context and interbank messaging; the standard for payments, trading, securities |
| **Separate Ways** | No integration; contexts deliberately disconnected | Zero-value seams; the strategic refusal of the single canonical model |
| **Open-Host Service** | Publish your model as a stable, documented set of services | Contexts consumed by many (Customer API, Payments status); the DDD name for API-first design; pairs with Published Language |

### 4.10 The Integration Patterns in the Estate (the transport reality)

The model-level patterns of §4 answer *what the seam is*; the transport-level reality answers *how the bytes move*. The two layers must be designed together — a context map whose arrows have no transport is a drawing, and a transport without a seam pattern is a point-to-point tangle:

- **ACL and Customer-Supplier over APIs.** The modern default transport for the translation and negotiation patterns is the service API — REST/gRPC contracts, versioned, behind the open-host service. The mechanics (contract-first design, versioning, the four integration styles) are the subject of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §2–§3.
- **Published Language over messaging.** The interchange formats (ISO 20022, FIX, SWIFT) travel on message infrastructure; the **integration events** of §2.4 are published over the event backbone with the **transactional outbox** guaranteeing exactly-once-ish delivery (the outbox is load-bearing and documented in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §9 and [event_stream_processing_guide.md](event_stream_processing_guide.md)).
- **Sagas at the seam.** When a flow spans contexts (drawdown → risk update → payment), the seam cannot be a distributed transaction — the bank's multi-context flows are **sagas**: sequences of local transactions linked by events, with compensating actions on failure. The mechanics are in [apache_seata_guide.md](apache_seata_guide.md); the durable orchestration that keeps long banking workflows alive is in [temporal_workflow_guide.md](temporal_workflow_guide.md).
- **Identity across the seams.** The same legal entity crossing five contexts needs a consistent identity story — the shared kernel on the Counterparty ID (§4.2) and the identity infrastructure of [distributed_auth_guide.md](distributed_auth_guide.md) are two halves of one problem.
- **The drift problem.** Every context copies reference data into its own store ([late_arriving_data_guide.md](late_arriving_data_guide.md)); the context map's job is to make the *ownership* of each piece of data explicit so the copies are recognized as copies, with one source of truth per concept.

The rule that ties it together: **the map names the pattern; the transport serves the pattern; the outbox, the saga, and the ACL carry the load.** An integration architecture is the context map made executable.

---

## 5. The Event Storming

### 5.1 The Technique (verified)

**Origin.** **EventStorming** is a collaborative workshop format invented by **Alberto Brandolini** (who remains its principal evangelist; the official site, eventstorming.com, defines it as "a flexible workshop format for collaborative exploration of complex business domains" and his book *Introducing EventStorming* is the reference text, published via Leanpub). It grew out of Brandolini's DDD facilitation practice in the early 2010s and has become the standard *discovery* technique of the DDD community — the way a team surfaces the domain, the events, the aggregates, and the boundaries before (or while) designing the model.

**The mechanics.** A room (or a virtual board), a wall covered in paper, **sticky notes of fixed colors with fixed meanings**, and everyone who knows anything about the business — domain experts, operations, developers, product owners, architects — standing together and building a timeline. The core notation:

- **Orange sticky — a domain event** (past tense): "Payment Executed", "Limit Breached". The group starts by flooding the timeline with events: *what happens?*
- **Blue sticky — a command** (imperative): the trigger that causes events — "Approve Loan", "Submit Documents". Commands are placed on the left of the events they cause.
- **Yellow sticky — an actor** (or role): who issues the commands — the relationship manager, the risk officer, the system.
- **Green sticky — a read model / query**: the information the actors need to decide — "Outstanding Exposure", "Customer Status".
- **Purple/lilac sticky — a policy or business rule**: the automated "when X happens, do Y" logic that connects events to commands.
- **Pink/red sticky — a hot spot / open question**: the things nobody knows, the disagreements, the risks — left on the wall until resolved.
- **Big sticky / boundary marker — the aggregate and the bounded context**: after the storm settles, the group draws the aggregate boundaries (which events belong to which aggregate's lifecycle) and the context boundaries (which clusters of events belong to which part of the business).

**The four flavours.** The official site distinguishes four workshop styles: **Improve** (dig into the business flow of an existing line of business, find the inconsistencies and impediments), **Envision** (explore a whole new business ecosystem and its architectural boundaries), **Explore** (design new services' dynamics with stakeholders), and **Design** (design critical software behaviour on an event-driven model — "be ready to embrace the full power of Microservices and Domain-Driven Design"). A full DDD engagement typically runs them in sequence: improve/envision to understand, explore/design to build.

**Why it works.** The technique's power is that it *forces the ubiquitous language into existence*: the events are named in the business's words, the disagreements surface as hot spots, and the model emerges from the group rather than being dictated by an architect. The timeline structure reveals the aggregate boundaries naturally — an aggregate is the cluster of events that share a lifecycle — and the context boundaries appear where the story breaks into stories told in different vocabularies. For a bank, an event-storming session on the lending lifecycle routinely surfaces the seams between origination, credit, booking, and servicing in an afternoon — seams that months of design documents had left fuzzy. This is the discovery phase that feeds the strategic design of §3 and the tactical design of §2 — and the natural kick-off for the context-map work of §8.

### 5.2 Running an Event-Storming Session (verified — the standard mechanics)

The official method, in practice, runs like this (the steps are the standard facilitation sequence; the colors are the published notation):

1. **Assemble the room.** Domain experts, operations people, developers, product owners, architects — "everyone who knows something" (Brandolini's principle). No observers: everyone is a participant.
2. **Flood the timeline with events (orange).** Unconstrained, chaotic, fast: *what happens?* "Payment executed", "Limit breached", "Documents received", "Customer complained". Ten minutes, hundreds of stickies. Chaos is the point — it is the business's raw memory.
3. **Add commands (blue) and actors (yellow).** For each event, who caused it and through what command? The timeline starts to take shape: events on the right of their triggering commands.
4. **Add read models (green).** What information did the actor need to issue that command? "Outstanding exposure", "Customer status". The green stickies are where the *queries* of the system reveal themselves.
5. **Add policies (purple).** The automated rules: "when X happens, do Y". The purple stickies connect events to commands without human actors — the system's own reactions.
6. **Mark the hot spots (pink).** Disagreements, unknowns, risks, missing knowledge — the pink stickies stay on the wall, visible, until resolved. *The hot spots are the most valuable output of the session*: they are the list of things the business itself does not yet agree on.
7. **Cluster into aggregates.** Walk the timeline: which events share a lifecycle? The clusters of events that belong to one thing's story are the aggregates — the consistency boundaries of §2.3 emerge from the wall.
8. **Draw the bounded contexts.** Where does the story stop being one story? The seams between clusters of clusters are the context boundaries — the strategic design of §3.1 appears.
9. **Capture the model.** Photograph the wall; transcribe the events, the aggregates, and the boundaries; resolve the hot spots in follow-up sessions.
10. **Repeat at the next scale.** The same technique scales: event-storm the whole bank's flow (§7), then storm a single context's design (§8).

The output of a good session is not a document — it is a *shared model*: the ubiquitous language made visible, the aggregates named, the boundaries drawn, and the disagreements listed. Everything else in this guide is what you do with that wall.

### 5.3 The Storming Table

| Element | Sticky color | Meaning | Banking example |
|---|---|---|---|
| **Domain Event** | Orange | Something that happened, past tense — the backbone of the timeline | "Loan Disbursed", "Payment Rejected", "Limit Breached" |
| **Command** | Blue | An imperative trigger that causes events, issued by an actor or a policy | "Approve Facility", "Execute Payment", "Freeze Account" |
| **Actor** | Yellow | The person, role, or system issuing commands | Relationship Manager, Risk Officer, SWIFT Gateway, Batch EOD |
| **Read Model / Query** | Green | The information an actor needs to decide | "Outstanding Exposure", "Customer KYC Status", "Available Limit" |
| **Policy / Business Rule** | Purple | Automated "when X, then Y" logic linking events to commands | "When Limit Breached, then Freeze New Drawings" |
| **Hot Spot / Question** | Pink/Red | Unknown, disagreement, or risk — left visible until resolved | "Who owns the exposure figure at month-end?" |
| **Aggregate / Context Boundary** | Big sticky / marker | Where the model's consistency boundaries and context seams fall | The Lending aggregate; the boundary between Lending and Risk contexts |
| **Workshop style** | — | Improve / Envision / Explore / Design (the four flavours) | Improve the lending flow → Design the event-driven servicing model |

### 5.4 Event Storming and the Other Workshops (the modeling-family context)

Event storming is the most famous of the collaborative modeling formats, but it is not the only one, and the DDD community uses them as a family:

- **Event storming** — the big-timeline discovery format of §5.1–§5.2: events, commands, aggregates, contexts. Best for *understanding the whole*: a line of business, a new ecosystem, a complex flow.
- **Example mapping** — a smaller, faster format (stories, rules, examples, questions — four colors of sticky notes) for *refining one user story* into rules and acceptance examples; the bridge from the wall to the backlog. It was popularized by Matt Wynne and the Cucumber community as the discovery half of BDD.
- **Domain storytelling** — a narrative format (Stefan Hofer and Henning Schwentner's book, *Domain Storytelling*) that models business processes as stories of actors and work objects, which translates directly into bounded contexts and aggregates.
- **The DDD Crew's modeling process** — the open-source DDD Crew publishes a structured process: start with an **Event Storming** to discover, use the **Bounded Context Canvas** to capture each context's essentials (name, responsibilities, ubiquitous language, relationships), and the **Aggregate Canvas** to design each aggregate's invariants and commands.

The family rule: **discover with event storming, refine with example mapping, capture with the canvases, design with the tactical patterns.** They are all the same discipline — the ubiquitous language made visible — at different zoom levels.

---

## 6. The Anemic Model

### 6.1 The Fowler 2003 Critique (verified)

**Origin.** The **anemic domain model** is the anti-pattern diagnosed by **Martin Fowler on his bliki, 25 November 2003** — "Anemic Domain Model," still the canonical critique. Fowler opens by noting he had discussed the trend with Evans himself, and that both were alarmed by its spread. The diagnosis: the anemic model "looks like the real thing" — objects named after the domain's nouns, with rich relationships — but "there is hardly any behavior on these objects, making them little more than bags of getters and setters." All the domain logic lives in "a set of service objects" on top, which "capture all the domain logic, carrying out all the computation and updating the model objects with the results."

**The critique, in Fowler's own terms.** (1) It is *contrary to the basic idea of object-oriented design*, which is "to combine data and process together" — the anemic model is "really just a procedural style design." (2) Worse, it *looks* object-oriented, so "many people think that anemic objects are real objects, and thus completely miss the point." (3) The economic argument: "the problem with anemic domain models is that they incur all of the costs of a domain model, without yielding any of the benefits" — you pay the O/R-mapping cost, the domain-object scaffolding cost, but because the behavior is in services you "essentially end up with Transaction Scripts, and thus lose the advantages that the domain model can bring." (4) The layer confusion: a **service layer** above a *behaviorally rich* domain model is legitimate (Fowler explicitly defends it), but a service layer *instead of* behavior is anemia. The rule of thumb Fowler leaves us with: "the more behavior you find in the services, the more likely you are to be robbing yourself of the benefits of a domain model. If all your logic is in services, you've robbed yourself blind."

**Why it happens.** Fowler's suspects: teams that "haven't really worked with a proper domain model," especially those "from a data background," and technologies that encourage it (he cites J2EE's Entity Beans as a historical culprit). The structural causes are familiar in every bank: the persistence-first mindset (the database schema *is* the model, so objects become table rows with getters/setters), the framework mindset (JPA entities, DTOs, and controllers as the whole design vocabulary), and the speed trap (services are faster to write today, and the debt is deferred until the business rules have nowhere to live and every change scatters across twenty services). The DDD answer is the discipline of §2: behavior belongs in entities and value objects, aggregates enforce invariants, and services stay thin.

**The healthy counter-picture.** The layered architecture Evans prescribes (quoted in Fowler's essay): the **Application Layer** "defines the jobs the software is supposed to do and directs the expressive domain objects to work out problems... kept thin. It does not contain business rules or knowledge, but only coordinates tasks"; the **Domain Layer** "is responsible for representing concepts of the business... This layer is the heart of business software." Evans' warning, quoted there: "the more common mistake is to give up too easily on fitting the behavior into an appropriate object, gradually slipping toward procedural programming." The anemic model is that mistake, systematized.

### 6.2 The Anemic Model in Banking

The anemic model is not an academic risk — it is the *default state* of much banking software, for structural reasons Fowler identified in 2003 and the industry has since industrialized:

- **The persistence-first reflex.** The database schema is treated as the model: JPA entities mirror tables, getters and setters mirror columns, and the "domain model" is a DTO graph. The business rules then have nowhere to live except services — and the services grow until they are transaction scripts with a JPA facade. The fix is the §2 discipline: entities and value objects with behavior, the repository hiding the schema behind the domain (§2.5).
- **The framework vocabulary.** In a stack of controllers, services, and repositories generated from a schema, the team *stops hearing the business*: the code speaks "DTO", "mapping", "save" — not "drawdown", "covenant", "dishonor". The ubiquitous language (§3.3) is precisely the antidote: if the code cannot say what the business says, the model has been lost.
- **The compliance illusion.** Anemic code looks safe — plain data objects, simple services, no cleverness — which appeals to governance reviews. The hidden cost is that the *rules* (the loan covenant logic, the payment validation, the LC examination steps) are scattered across services where nobody can see them whole, and every regulatory change becomes a hunt. Fowler's 2003 arithmetic still applies: all the costs of a domain model, none of the benefits.
- **The legacy inheritance.** The core banking systems of [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) are, structurally, procedural data-processing machines — COBOL record layouts and batch jobs. When a new context wraps the core through an ACL (§4.1) and the ACL is built as services mutating DTOs, the anemic model is *imported* along with the data. The remedy is the same one: the ACL must translate *into the domain model*, not into another data bag.

The diagnostic is Fowler's, and it is cheap to run: count the behavior in the domain objects. If the services hold the logic and the objects hold only state, the model is anemic — and every §2 pattern in this guide is the treatment.

### 6.3 The Anemic Table

| Aspect | The anemic domain model | The DDD domain model |
|---|---|---|
| **Objects** | "Bags of getters and setters" (Fowler) — nouns with no behavior | Entities and value objects carrying their own behavior and invariants |
| **Logic** | All domain logic in service objects on top of the model | Logic in the domain layer; services thin and stateless, hosting only logic with no object home |
| **Paradigm** | "Really just a procedural style design" (Fowler) in OO clothing | Data and process combined — the OO idea, applied to the domain |
| **Cost/benefit** | "All of the costs of a domain model, without yielding any of the benefits" (Fowler) — pays O/R mapping, gets Transaction Scripts | The modeling cost is repaid by rules that live where they belong and change without scattering |
| **Layer confusion** | Service layer *instead of* domain behavior | Service layer *above* a behaviorally rich domain layer ("kept thin", Evans) |
| **Why it happens** | Data-background teams, persistence-first thinking, frameworks that encourage passive objects (Fowler: J2EE Entity Beans) | Averted by deliberate model design, ubiquitous language, aggregate boundaries |
| **The verdict** | "If all your logic is in services, you've robbed yourself blind" (Fowler, 2003) | "The more behavior you find in the services, the more likely you are to be robbing yourself" — the inversion is the cure |

---

## 7. The DDD in Banking

### 7.1 The Lending, Payments, Trade-Finance Domains (verified — the Cymbal Bank context)

**Why banking is DDD's natural habitat.** A bank is a machine for processing domain complexity: every product has a legal lifecycle, every transaction has regulatory consequences, every term means different things to different desks. The polysemy problem that motivates bounded contexts is *endemic* to banking — "customer", "balance", "position", "value date" all shift meaning across the bank — and the legacy reality (decades-old cores that *are* the system of record, documented in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)) means the integration patterns of §4 are not optional. The bank's context map is not an abstraction exercise: it is the architecture.

**The lending domain.** The lending subdomain spans origination (application, pricing, limit), credit (assessment, approval, collateral), booking (facility, drawdown, disbursement), and servicing (repayment, interest, restructuring, recovery). Its core entities are the Customer, the Facility, the Loan, the Collateral, the Guarantor; its core value objects are Money, InterestRate, Tenor, RepaymentSchedule; its aggregates cluster around the Facility (root) with its Drawdowns, Schedules, and Collateral as members. The rules are heavy and legal: covenants, cross-defaults, margin re-pricing, limit utilization. This is a *core domain* — a bank's lending capability is differentiating — and it is where DDD's modeling investment belongs. In the Cymbal Bank context, lending spans corporate and investment-banking credit — the syndicated facilities and structured finance where the reference-rate mechanics, the commitment and utilization math, and the multi-bank agent roles make the domain genuinely hard ([../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) §3 documents the real Cymbal Bank systems estate this maps onto).

**The payments domain.** The payments subdomain is the flow machine: instruction capture, validation, routing, clearing (domestic RTGS, SWIFT, cards), settlement, and reconciliation. Its context map is *the* classic DDD teaching case because every bank has at least: a Payments context (the payment itself — the Payment aggregate with its legs, fees, and status), a Channels context (how instructions enter), a Counterparty/Reference-Data context (who the parties are), a Liquidity/Funding context (which accounts fund the flow), and a Regulatory Reporting context (the conformist to the regulator's schemas). The payments world is where **published language** is king: SWIFT MT/MX messages and ISO 20022 *are* the industry's shared interchange format, and the modern payments hub is an open-host service publishing payment status to every consumer. The domain events — `PaymentExecuted`, `PaymentRejected`, `RecallRequested` — are the natural backbone of the event-driven integration in [event_stream_processing_guide.md](event_stream_processing_guide.md) and the reason the outbox pattern ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §9) is load-bearing in every payment service.

**The trade-finance domain.** Trade finance is the *least* digitized and the *most* document-heavy of the three — letters of credit, guarantees, collections, and the shipping documents behind them (bills of lading, invoices, certificates of origin). The domain's complexity is procedural and temporal: the letter of credit lifecycle (issue → advise → present → examine → pay/accept → reimburse) is a state machine with strict timing rules, UCP 600 governing the rules of documentary credits, and the examination of documents as a genuinely hard domain skill. DDD fits it exceptionally well: the LetterOfCredit aggregate (root) with its Documents, Amendments, and Presentations; the TradeFinance context speaking its own ubiquitous language (the "presenting bank", the "issuing bank", "dishonor"); and the integration story dominated by SWIFT's trade messages (MT700 series) as published language. The working guides [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md) and [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) document the real systems landscape this models.

**The Cymbal Bank context.** Cymbal Bank — the corporate and investment bank of the Crédit Agricole group — runs all three: global markets and treasury, corporate banking with syndicated lending, trade finance and structured finance, and the payments rails behind them ([../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) §3). For a solution architect there, DDD is not an academic exercise: the *same* "customer" is a legal entity in Onboarding, a counterparty in Markets, a borrower in Lending, a beneficiary in Trade Finance, and a payer in Payments — five bounded contexts, five models, one map connecting them. The trading-domain angle (positions, books, risk) is covered in [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md); the migration reality of the legacy cores beneath all of it is in [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) and the target architecture in [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md). What DDD adds is the *language*: the ability to say precisely which model owns which word, and what happens at every seam.

### 7.2 The Domain Lifecycles

**The lending lifecycle.** The lending subdomain (§7.1) is a state machine with money: **origination** (application, pricing, limit check) → **credit assessment** (analysis, approval, collateral) → **booking** (facility agreement, margin, covenants) → **servicing** (drawdowns, repayments, interest, covenant monitoring, restructuring) → **recovery** (default, workout, write-off). Each transition is an event (`ApplicationSubmitted`, `FacilityApproved`, `FacilityDrawn`, `CovenantBreached`, `LoanRestructured`); each event is a candidate for the integration backbone (§2.4). The aggregate design follows the lifecycle: the Facility aggregate owns booking through servicing; the Application and the Workout are *separate* aggregates that hand off at the boundaries — they are different lifecycles with different rules, and forcing them into one cluster violates §2.3's smallness rule.

**The payments flow.** The payments subdomain is a pipeline: **capture** (instruction from a channel) → **validate** (syntax, funds, limits) → **route** (clearing path: RTGS, SWIFT, cards, internal) → **settle** (funds movement, value date) → **reconcile** (the position against the flows). The statuses are the domain events (`PaymentReceived`, `PaymentValidated`, `PaymentSent`, `PaymentSettled`, `PaymentRejected`, `RecallRequested`); the Payment aggregate's root enforces the transition rules (a settled payment cannot be recalled except through the recall workflow — a rule with legal weight). The reconciliation leg is where [late_arriving_data_guide.md](late_arriving_data_guide.md) bites: the counterparty's confirmation arrives after the settlement, and the domain must model "settled but unconfirmed" as a first-class state, not an error.

**The letter-of-credit state machine.** Trade finance's core object, the letter of credit, is the cleanest state machine in banking — its states and transitions are governed by UCP 600 and the standard lifecycle:

| State | Meaning | Events in / out |
|---|---|---|
| **Issued** | The LC is opened (MT700 sent) | `LCIssued` → |
| **Advised** | The beneficiary's bank has notified the beneficiary | `LCAdvised` → |
| **Amended** | Terms changed (MT707) | `LCAmended` (loop) → |
| **Presented** | Documents received for examination | `DocumentsPresented` → |
| **Examined** | Documents checked against the LC terms | `DocumentsAccepted` / `DocumentsRejected` → |
| **Paid / Accepted** | Settlement made or a time draft accepted | `LCPaid`, `LCAccepted` → |
| **Reimbursed** | The issuing bank has reimbursed the presenting bank | `LCReimbursed` (terminal) |
| **Closed / Expired** | The LC ends unused or by expiry | `LCClosed` (terminal) |

Every transition is a domain event; every state is enforced by the LetterOfCredit aggregate root; every *document* examination is domain logic that belongs in the model (§2.2 — the examined DocumentSet as a value object). This is DDD's natural habitat: a state machine with legal consequences, owned by experts, expressible in one ubiquitous language.

### 7.3 The Banking Table

| Banking subdomain | Core entities & aggregates | Key value objects | The ubiquitous language in play | The integration reality |
|---|---|---|---|---|
| **Lending** (origination → credit → booking → servicing) | Facility (root), Drawdown, Collateral, Guarantor, Customer | Money, InterestRate, Tenor, RepaymentSchedule, LimitUtilization | "facility", "drawdown", "covenant", "default", "margin" | Customer-Supplier with credit-risk and reference-data; ACL around the legacy core; Customer-Supplier to reporting |
| **Payments** (instruction → clearing → settlement) | Payment (root), PaymentLeg, Fee, StatusHistory | Amount, Currency, ValueDate, Fee, IBAN | "payment", "recall", "reject", "settle", "value date" | Published Language (SWIFT/ISO 20022); Open-Host Service for status; domain events as the backbone; outbox for reliability |
| **Trade Finance** (LC → documents → reimbursement) | LetterOfCredit (root), Document, Amendment, Presentation | DocumentSet, Amount, Tenor, PortPair | "issuing bank", "presenting bank", "dishonor", "documents" | Published Language (MT700 series); Conformist to UCP rules; the state machine as the aggregate |
| **Across the bank** | Customer/LegalEntity (per-context), Counterparty, Account | Money everywhere, identity everywhere | "customer" means five different things — that is the point | Context map + ACLs at every legacy seam; Separate Ways where integration is waste |

### 7.4 The Banking Adoption Reality (flagged — the honest picture)

The theory of §7.1–§7.3 is how DDD *maps* a bank. How banks actually *adopt* it is a different, more sober story, and the architect should hold both:

- **The legacy core sets the pace.** The system of record is the mainframe COBOL core of [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md): its model is record layouts and batch jobs, its "language" is decades old, and it will not be replaced — it will be *wrapped and strangled*. DDD's role in a bank is therefore mostly played at the *edges*: new contexts (channels, digital lending, trade digitization) built DDD-clean, connected to the core through ACLs, per [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6. The context map of §8 is as much a *migration plan* as an architecture.
- **Vendor packages resist the map.** Much of a bank runs on vendor platforms (the core, the payments hub, the trade system) whose internal models are opaque and non-negotiable. DDD does not vanish here — it relocates: the vendor's model is the *foreign model* behind an ACL (§4.1), the regulator's formats are the *published language* you conform to (§4.4–§4.5), and the *integration context* (the bank's own translation layer) becomes the place where the bank's domain model actually lives. The [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) target architecture is exactly this: DDD-defined service contexts around and between the packaged systems.
- **The organization is the constraint.** Conway's law and the product-line structure of a bank like Cymbal Bank ([../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md)) mean the contexts that *should* exist (per the domain) and the teams that *do* exist (per the org) rarely align on day one. DDD's strategic answer is patient: draw the map honestly, name the misalignments, and let the context boundaries migrate toward the domain boundaries as teams reorganize — the same patience the migration playbooks of [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §7 demand.
- **The success pattern is narrow and real.** Where banking DDD succeeds, it is usually the same shape: a *core-domain context* (lending origination, trade digitization, a payments hub) with real business complexity, a genuine expert partnership, event-stormed boundaries, and a clean event backbone — funded like the investment it is ([../management/business_case_development_guide.md](../management/business_case_development_guide.md)), measured on the change-velocity and risk-control it buys, and extended context by context, never all at once.

---

## 8. The Worked Example: A Cymbal Bank's Context Map

### 8.1 The Scenario (the familiar context)

The scene is a Cymbal Bank — the corporate and investment bank of a large European cooperative group, with global markets, corporate lending, trade finance, and payments, running on a legacy core of mainframe and midrange systems ([../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) documents the family; [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) documents the actual estate). The bank is mid-modernization: the channels are new, the front-office systems are modern, but the *system of record* — the accounts, the ledgers, the positions — still lives in the legacy core, surrounded by the strangler fig of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6. The mandate: **map the context landscape, design the bounded contexts, and specify the seams** — the strategic design that will drive the microservice target architecture ([../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md)) and the decomposition ([monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §2).

The method is event storming (§5): two days, the wall covered in orange (events) and blue (commands), the relationship managers and the risk officers arguing with the developers about what "exposure" means — until the hot spots resolve and the boundaries appear. What follows is the map that comes out of it.

### 8.2 The Bounded-Context Design

**The context map.** Ten bounded contexts, drawn honestly:

1. **Onboarding & KYC** — owns the *legal entity*: identity, KYC/AML documentation, sanctions screening. Its "customer" is the verified legal entity with a documented risk profile. It is the **open-host service** of the bank: every other context consumes its customer API.
2. **Reference Data** — owns the static truth: products, rates, calendars, counterparties-as-identifiers. A **shared kernel** (small!) with Onboarding on the Counterparty identifier, and an **open-host service** to everyone else.
3. **Lending** — owns the *borrower relationship*: facilities, limits, drawdowns, covenants, collateral. The **core domain** of the corporate bank. Its "customer" is the borrower; its "exposure" is *utilization against committed facilities*.
4. **Trade Finance** — owns the *documentary* business: letters of credit, guarantees, collections, the UCP 600 state machine. Its "customer" is the applicant/beneficiary; its "documents" are the examined goods of the trade. **Conformist** to the SWIFT MT700 published language; a **customer-supplier** relationship with Lending (trade limits draw on facilities).
5. **Payments** — owns the *flow*: instructions, validation, routing, clearing, settlement. Its "customer" is the account holder/beneficiary; its "account" is the funding/debit target. **Published language** out (SWIFT, ISO 20022); **open-host service** of payment status to every consumer; domain events (`PaymentExecuted`) feeding Reconciliation and Reporting.
6. **Global Markets** — owns the *positions*: trading books, derivatives, treasury. Its "counterparty" is a *trading* concept (with credit-limit hooks into Risk); its language is the front office's (see [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md)). **Published language** (FIX) inbound; **ACL** against the risk engine's model.
7. **Risk & Limits** — owns the *aggregate exposure*: counterparty credit risk across all businesses, the limit framework. The consumer of everyone's exposure events; the **customer** in a **customer-supplier** relationship with Lending, Trade Finance, and Markets (they are the suppliers of exposure data).
8. **Finance / Regulatory Reporting** — owns the *books and the returns*: the ledger, IFRS/CRR reporting, the regulator's schemas. **Conformist** to the regulator's published language; **ACL** around the legacy general ledger.
9. **The Legacy Core** — the system of record beneath: accounts, ledgers, the batch engine. Deliberately mapped as the **Big Ball of Mud** at the center, wrapped by **ACLs** at every seam — the strangler fig of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6 in full operation.
10. **Channels & Servicing** — the front doors: portal, API for corporate clients, statements, alerts. **Separate ways** with most of the bank; **customer-supplier** with Payments and Lending where it needs their status.

**The seams, named.** The map's arrows are the interesting part: Onboarding → Reference Data via a **small shared kernel** (the Counterparty ID) and open-host customer API; Lending → Legacy Core through an **ACL** that translates facility bookings into the core's record layouts; Trade Finance → Payments through **published language** (the settlement instruction is an ISO 20022 message, not a call); Risk & Limits ← Lending/Trade/Markets through **integration events** (`FacilityDrawn`, `LimitUtilized`) consumed asynchronously — the saga and outbox mechanics of [apache_seata_guide.md](apache_seata_guide.md) and [event_stream_processing_guide.md](event_stream_processing_guide.md); Reporting → everything as **conformist** to regulatory formats. The seams that *don't* exist are named too: Channels has **separate ways** with Markets — no integration, deliberately.

**One aggregate, designed.** Inside Lending, the event storm surfaced the **Facility** aggregate: root Facility, members Drawdown, RepaymentSchedule, Covenant, Collateral. External contexts hold the Facility *ID*, never the object. The root enforces the invariant that made the relationship managers angry on day one: *no drawdown may exceed the facility's available commitment after applying the utilization rules and the outstanding collateral haircut*. That single rule — enforced in one place, in the ubiquitous language, with the business in the room — is the entire point of the exercise: in the legacy core it lived, unreachable, inside a COBOL batch program.

### 8.3 The Context Map, Drawn

The ten contexts of §8.2, as the map the team pins on the wall (arrows are the named seams of §4):

```
                            ┌──────────────────────────────┐
                            │  Onboarding & KYC            │  open-host (customer API)
                            │  (legal entity, AML)        │◄──────────────┐
                            └──────────────┬───────────────┘               │
                                 shared kernel (Counterparty ID)           │
                            ┌──────────────▼───────────────┐               │
                            │  Reference Data              │               │
                            │  (products, rates, calendars)│  open-host    │
                            └──────────────┬───────────────┘               │
         ┌──────────────────────────────────┼──────────────────────────────┘
         │  customer-supplier (limits)      │  published language (ISO 20022)
   ┌─────▼──────────┐   events:   ┌─────────▼──────────┐   events:   ┌──────▼──────────┐
   │  Lending       │◄───────────►│  Risk & Limits    │◄───────────►│  Payments       │
   │  (core domain) │  FacilityDrawn│  (aggregate      │ LimitBreached│  (flow machine) │
   └─────┬──────────┘  LimitUtilized└─────────┬──────────┘             └──────┬──────────┘
         │ ACL                                │ events                        │ open-host (status)
   ┌─────▼──────────┐                  ┌──────▼──────────┐            ┌──────▼──────────┐
   │ Trade Finance  │  conformist      │ Finance/Reporting│  conformist │ Channels &     │
   │ (LC state      │◄────────────────►│ (ledger, CRR)   │◄───────────►│ Servicing      │
   │  machine)      │   MT700 (SWIFT)  └──────┬──────────┘   MT/MX      └────────────────┘
   └─────┬──────────┘                         │
         └──────────────┬─────────────────────┘
                   ┌────▼─────────────────────────────┐
                   │  THE LEGACY CORE (Big Ball of Mud)│◄── ACL at every seam
                   │  accounts · ledgers · batch EOD   │    (strangler fig in progress)
                   └──────────────────────────────────┘
```

The drawing is deliberately ugly: the legacy core sits at the bottom as the mud it is, and every arrow *leaving* it is an ACL. The modern contexts speak to each other in events and standards; they speak to the core only through translators — which is the honest picture of a bank mid-migration ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6, [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) §8).

### 8.4 The Tactical Design per Context

The map is the strategy; each context's interior is the tactics. The same storm that produced the boundaries also produced the aggregates:

| Context | Key aggregate (root) | Core value objects | Domain events it publishes | The seam out |
|---|---|---|---|---|
| **Onboarding & KYC** | LegalEntity (KYC dossier) | EntityId, RiskRating, SanctionsStatus | `EntityVerified`, `SanctionsHit` | Open-host customer API; shared kernel on the ID |
| **Reference Data** | Product, RateCurve | CurrencyPair, Calendar, Tenor | `RateCurveUpdated` | Open-host; shared kernel |
| **Lending** | Facility (drawdowns, schedules, covenants, collateral) | Money, InterestRate, RepaymentSchedule | `FacilityDrawn`, `CovenantBreached`, `FacilityApproved` | Integration events → Risk; ACL → core |
| **Trade Finance** | LetterOfCredit (documents, amendments, presentations) | DocumentSet, Amount, Tenor | `DocumentsPresented`, `LCPaid` | Published language (MT700); customer-supplier with Lending |
| **Payments** | Payment (legs, fees, status history) | Amount, Currency, ValueDate, IBAN | `PaymentExecuted`, `PaymentRejected` | Published language (ISO 20022); open-host status; outbox |
| **Global Markets** | Trade, Position (book, risk hooks) | Price, Notional, Maturity | `TradeExecuted`, `PositionChanged` | Published language (FIX); ACL to risk engine |
| **Risk & Limits** | Limit (utilization across businesses) | Exposure, Utilization, Haircut | `LimitBreached`, `LimitUtilized` | Consumes everyone's events; customer of the suppliers |
| **Finance / Reporting** | LedgerEntry (the accounting truth) | Balance, GLAccount, ValueDate | `LedgerPosted` | Conformist to regulatory schemas; ACL to the GL |
| **Channels & Servicing** | ServicingRequest (statements, alerts) | StatementPeriod, AlertPreference | `StatementGenerated` | Customer-supplier with Payments/Lending; separate ways elsewhere |
| **Legacy Core** | (none — the mud) | (none) | (none — batch only) | ACLs only, until strangled |

### 8.5 The Lessons

- **The language came first, and it found the bugs.** The storming session that produced this map was, first and foremost, a conversation in which "exposure" was forced to be *defined* — and the definitions disagreed. Naming the disagreement was the first artifact; the map was the second. This is the pattern of every successful DDD engagement: **the language is not documentation, it is discovery** (§9).
- **The core domain got the investment; the generic domain got a vendor.** Lending and Risk got the modeling workshops, the aggregate design, the careful events. Reference Data was bought where possible, wrapped where not. Nobody spent a week modeling the notification service — and *that* allocation of attention is itself a strategic decision (§3.4).
- **The legacy core is not a context — it is a *place*.** The Big Ball of Mud has no model of its own worth adopting; it is wrapped by ACLs and strangled context by context ([legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6). The DDD map makes the migration *sequential*: each context becomes a microservice as its ACL and data ownership are established — the 14-migration playbook of [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) is the execution plan for exactly this map.
- **Events are the glue; transactions are the exception.** Across contexts the bank runs on integration events and published language — `PaymentExecuted`, `FacilityDrawn`, `DocumentPresented` — with the outbox guaranteeing delivery and the saga (not the distributed transaction) handling multi-context consistency ([apache_seata_guide.md](apache_seata_guide.md)). The *aggregate* is where "one transaction" still means one transaction.
- **The anemic model is the default failure mode.** Every team that skipped the modeling and "just built services" produced bags of getters and setters with all logic in the service layer — Fowler's 2003 diagnosis, replayed every year (§6). The map and the aggregates are the guard.
- **The map must be kept honest and current.** A context map that describes last year's architecture is a lie. The map lives with the architecture reviews, updated as contexts split (a Markets context that outgrows one team becomes two contexts, with their own seam) — the same discipline as [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md)'s boundary honesty.

---

## 9. The Summary: The Language First

**The one-page.** Domain-Driven Design is Evans' 2003 answer to a single question: *how do you build software in a domain complex enough that no one person holds the whole truth?* The answer has four moves. **First, model**: capture the domain in a shared system of abstractions, built *with* the domain experts, deep enough to express the rules and simple enough to be useful. **Second, speak**: bind the whole team to a ubiquitous language structured around that model — used in code, meetings, and documents, and allowed to evolve as understanding grows. **Third, bound**: never try to unify the whole enterprise under one model; partition the domain into bounded contexts, classify the subdomains by strategic value (core, supporting, generic), and draw the context map honestly — every seam named with its pattern. **Fourth, build**: inside each context, express the model with the tactical patterns — entities and value objects that carry behavior, aggregates that enforce invariants through their roots, domain events that announce what happened, repositories that hide persistence, thin domain services, and factories that construct valid objects — and refuse the anemic model that would drain the behavior into a service layer. Where contexts meet, choose the seam deliberately: the ACL where you must be protected from a foreign model, the shared kernel and customer-supplier where you cooperate, the conformist where you cannot negotiate, the published language and open-host service where many must speak, separate ways where integration is waste. Discover the whole thing with event storming, where the business tells the story in events and the boundaries fall out of the conversation.

**The final word — the language first.** Every pattern in this guide is downstream of the language. The bounded context is where a language is coherent; the context map is where languages translate; the aggregate is a language fragment made consistent; the ACL is a translation layer; the anemic model is what happens when the language was never built and the code silently stopped speaking the business's words. DDD is not a set of diagrams or classes — it is the discipline of *talking about the business until the software and the business mean the same thing*. The model is the language made rigorous; the code is the language made executable. Start there. If the team and the business cannot yet say the same sentence about "exposure", no aggregate, no context map, and no microservice will save the project — build the language first, and everything else in DDD follows from it.

---

## 10. Verification and Claims-Status

Every factual claim in this guide was checked against primary sources during this pass (web access live; `web_extract` used against primary pages — note the `web_search` backend returned empty results for several queries, so verification was done by extracting primary sources directly rather than via search snippets):

- ✅ **The Evans book.** *Domain-Driven Design: Tackling Complexity in the Heart of Software*, Eric Evans, Addison-Wesley, 2003, ISBN 978-032112521-7 — verified against Amazon/O'Reilly/Google Books/ACM listings. The subtitle is exactly as quoted. The book's four-part structure (Parts I–IV, strategic design as Part IV) is confirmed by Martin Fowler's Bounded Context bliki ("Bounded Context opens part IV (Strategic Design)").
- ✅ **The official glossary definitions.** Entity, Value Object, Aggregate, Factory, Repository, Service, Bounded Context, Context Map, Core Domain, Ubiquitous Language, Strategic Design, DDD itself — all quoted from the DDD Community's "Glossary of Domain-Driven Design Terms" (dddcommunity.org/resources/ddd_terms/, excerpted from Evans' book and consistent with Evans' *DDD Reference* 2015 PDF).
- ✅ **Fowler, "Anemic Domain Model", 25 November 2003** — extracted verbatim from martinfowler.com; all quotes ("bags of getters and setters", "incur all of the costs of a domain model, without yielding any of the benefits", "If all your logic is in services, you've robbed yourself blind", the Evans application-layer/domain-layer quotes) are direct from that page.
- ✅ **Fowler, "Bounded Context", 15 January 2014** and **"Ubiquitous Language", 31 October 2006** — extracted verbatim; the "total unification of the domain model for a large system will not be feasible or cost-effective" quote is Evans as cited by Fowler.
- ✅ **The context-mapping patterns.** Anti-corruption layer, shared kernel, customer-supplier, conformist, published language, separate ways, open-host service, partnership, big ball of mud — verified against Evans' own definitions as reproduced (with quotes from the *DDD Reference*) in Wikipedia's DDD article, and consistent with the ACL treatment in [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md).
- ✅ **Vernon.** *Implementing Domain-Driven Design*, Addison-Wesley, 2013 (ISBN 978-0321834577, confirmed via Fowler's Bounded Context page, which also confirms IDDD ch. 2 = bounded contexts, ch. 3 = context maps); *Effective Aggregate Design* series on dddcommunity.org (2011–2012).
- ✅ **Event storming.** eventstorming.com (Alberto Brandolini's official site) — the definition ("a flexible workshop format for collaborative exploration of complex business domains"), the four flavours (Improve/Envision/Explore/Design), the Leanpub book *Introducing EventStorming*, and the microservices/DDD link. The sticky-note color conventions follow the standard published notation (orange events, blue commands, etc.).
- ✅ **Subdomains (core/supporting/generic).** Core Domain definition quoted from the official glossary; the three-way classification and problem-space/solution-space framing follow the DDD Reference and Vernon — consistent across the canon, though the "supporting" vs "generic" split is a post-2003 formalization rather than Evans-2003 verbatim; flagged as such.
- ⚠ **Adoption statistics** — none cited; the claim that DDD is recommended by Microsoft only for complex domains is verified (Wikipedia cites the Microsoft Application Architecture Guide). Greg Young's "7 Reasons DDD Projects #FAIL" talk is real and referenced on dddcommunity.org.
- ⚠ **Banking mappings (lending/payments/trade finance, Cymbal Bank)** — these are the author's architectural analysis applied to the Cymbal Bank context, grounded in the sibling banking guides ([credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md) etc.); they are interpretation, not sourced facts, and are flagged as such. The worked example (§8) is a constructed illustration, not a documented Cymbal Bank initiative.
- ⚠ **Event storming as "the standard discovery technique" and its dating** ("early 2010s") — consistent with the official site and community literature but not pinned to a specific dated source this pass; flagged lightly.

---

## 11. Glossary

- **DDD / Domain-Driven Design** — An approach to software development holding that (1) for most software projects the primary focus should be the domain and domain logic, and (2) complex domain designs should be based on a model (official glossary definition). Coined by Eric Evans, 2003.
- **Evans, Eric** — Software designer, founder of Domain Language Inc.; author of *Domain-Driven Design: Tackling Complexity in the Heart of Software* (Addison-Wesley, 2003), the book that defined the discipline; author of the *DDD Reference* (2015).
- **Ubiquitous Language** — "A language structured around the domain model and used by all team members to connect all the activities of the team with the software" (Evans) — the shared vocabulary of business and developers, per bounded context.
- **Bounded Context** — "The delimited applicability of a particular model" (Evans); the boundary inside which one model and one ubiquitous language apply with full consistency; the microservice-boundary candidate.
- **Context Map** — "A representation of the Bounded Contexts involved in a project and the actual relationships between them and their models" (Evans); the honest diagram of the seams, each named with its integration pattern.
- **Subdomain** — A part of the business's problem space, classified by strategic value: core, supporting, or generic.
- **Core Domain** — "The distinctive part of the model, central to the user's goals, that differentiates the application and makes it valuable" (Evans); where DDD's modeling investment belongs.
- **Supporting Domain** — A subdomain the business needs but that does not differentiate it; standard effort and often off-the-shelf software.
- **Generic Domain** — A needed but undifferentiated subdomain (identity, notifications, storage); commodity effort.
- **Entity** — "An object fundamentally defined not by its attributes, but by a thread of continuity and identity" (Evans).
- **Value Object** — "An object that describes some characteristic or attribute but carries no concept of identity" (Evans); immutable, equal on attributes.
- **Aggregate** — "A cluster of associated objects that are treated as a unit for the purpose of data changes" (Evans); the consistency boundary, accessed through its root.
- **Aggregate Root** — The single member of an aggregate that external references are restricted to; the enforcer of the cluster's invariants.
- **Domain Event** — A model object recording something that happened in the domain, past tense, with the facts the domain cares about; distinct from integration events across contexts.
- **Repository** — "A mechanism for encapsulating storage, retrieval, and search behavior which emulates a collection of objects" (Evans); one per aggregate root.
- **Domain Service** — "An operation offered as an interface that stands alone in the model, with no encapsulated state" (Evans); stateless logic with no natural object home.
- **Factory** — "A mechanism for encapsulating complex creation logic and abstracting the type of a created object for the sake of a client" (Evans).
- **Anti-Corruption Layer (ACL)** — "An isolating layer to provide your system with functionality of the upstream system in terms of your own domain model" (Evans); the translation shield around foreign models.
- **Shared Kernel** — A small, explicitly bounded, jointly owned subset of the domain model shared between teams under joint change control (Evans).
- **Customer-Supplier** — The explicit upstream-supplier/downstream-customer relationship between two contexts' teams, with negotiated commitments (Evans).
- **Conformist** — The pragmatic adoption of the upstream's model with no translation, when negotiation is futile (Evans).
- **Published Language** — "A well-documented shared language that can express the necessary domain information as a common medium of communication" (Evans); ISO 20022, FIX, SWIFT.
- **Separate Ways** — The deliberate absence of integration between contexts whose seam has no value (Evans).
- **Open-Host Service** — "A protocol that gives access to your subsystem as a set of services" (Evans); the DDD name for API-first publishing, often with a published language.
- **Event Storming** — Alberto Brandolini's collaborative workshop format (eventstorming.com; *Introducing EventStorming*) for exploring complex domains with sticky-note timelines of events, commands, actors, read models, policies, and hot spots.
- **Anemic Domain Model** — The anti-pattern (Fowler, 2003) of domain objects as "bags of getters and setters" with all behavior in services; "all of the costs of a domain model, without yielding any of the benefits."
- **Fowler, Martin** — Software thinker and author; wrote the "Anemic Domain Model" bliki (25 Nov 2003) and the "Bounded Context" (15 Jan 2014) and "Ubiquitous Language" (31 Oct 2006) bliki entries; the accessible expositor of DDD's key ideas.
- **Vernon, Vaughn** — Author of *Implementing Domain-Driven Design* (Addison-Wesley, 2013), the implementation-focused companion to Evans, and the *Effective Aggregate Design* series.
- **Tactical Patterns** — The code-level building blocks inside a bounded context: entities, value objects, aggregates, domain events, repositories, domain services, factories.
- **Strategic Design** — "Modeling and design decisions that apply to large parts of the system... decided at team level" (glossary); bounded contexts, context maps, ubiquitous language, subdomains, integration patterns.
- **Complexity** — The subject of the discipline: the intricacy of the business domain itself, which Evans argues lives "in the heart of software" and must be modeled, not engineered around.

---

## 12. References

1. Evans, Eric. *Domain-Driven Design: Tackling Complexity in the Heart of Software*. Addison-Wesley, 2003. ISBN 978-032112521-7.
2. Evans, Eric. *Domain-Driven Design Reference: Definitions and Pattern Summaries*. 2015 (domainlanguage.com PDF).
3. Vernon, Vaughn. *Implementing Domain-Driven Design*. Addison-Wesley, 2013. ISBN 978-0321834577.
4. Vernon, Vaughn. *Effective Aggregate Design* (Parts I–III). DDD Community, 2011–2012 (dddcommunity.org/library/vernon_2011/, /vernon_2012/).
5. DDD Community. *Glossary of Domain-Driven Design Terms* — dddcommunity.org/resources/ddd_terms/.
6. Fowler, Martin. *Anemic Domain Model*. martinfowler.com/bliki, 25 November 2003.
7. Fowler, Martin. *Bounded Context*. martinfowler.com/bliki, 15 January 2014.
8. Fowler, Martin. *Ubiquitous Language*. martinfowler.com/bliki, 31 October 2006.
9. Brandolini, Alberto. *Introducing EventStorming*. Leanpub; eventstorming.com (official site, four workshop flavours).
10. Young, Greg. *7 Reasons DDD Projects #FAIL*. NDC 2010 (referenced on dddcommunity.org).
11. Foote, Brian and Yoder, Joseph. *Big Ball of Mud*. 1999 (laputan.org).
12. Wikipedia. *Domain-driven design* (context-mapping pattern definitions quoting Evans; Microsoft's "complex domains only" recommendation citing the Microsoft Application Architecture Guide).
13. Series siblings: [monolith_to_microservices_guide.md](monolith_to_microservices_guide.md) (Evans 2003 verified in §2.1), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (ACL + strangler fig in §6), [../banking/oracle_banking_microservices_architecture_guide.md](../banking/oracle_banking_microservices_architecture_guide.md) (the banking target architecture), [apache_seata_guide.md](apache_seata_guide.md), [event_stream_processing_guide.md](event_stream_processing_guide.md), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md), [../banking/credit_agricole_software_systems_guide.md](../banking/credit_agricole_software_systems_guide.md), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md), [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md), [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md).
