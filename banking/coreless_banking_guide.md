# Coreless Banking: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Banking Architecture — the "coreless" banking movement as the companion to this repository's core-banking family: what coreless banking means and why the term is contested, the drivers (cloud economics, API-first architecture, time-to-market), the reference architecture that replaces the monolithic core (ledger-as-a-service, product engines, workflow orchestration, the data layer), the accounting and operational implications (posting, EOD/batch in a distributed world), the verified vendor landscape (Thought Machine, Mambu, 10x Banking, and the legacy vendors' cloud-core offerings), the for-and-against debate with analyst positions and cautionary tales, and a full Cymbal Bank worked example of decomposing a legacy core and migrating to a coreless architecture
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026

---

### Table of Contents

1. [The Concept: What "Coreless" Banking Means](#1-the-concept-what-coreless-banking-means)
2. [The Terminology Debate: Coreless, Composable, and Core-as-a-Service](#2-the-terminology-debate-coreless-composable-and-core-as-a-service)
3. [The Drivers and the Legacy Constraint](#3-the-drivers-and-the-legacy-constraint)
4. [The Reference Architecture: What Replaces the Core](#4-the-reference-architecture-what-replaces-the-core)
5. [The Accounting Layer: Ledger-as-a-Service](#5-the-accounting-layer-ledger-as-a-service)
6. [Product Engines and Workflow Orchestration](#6-product-engines-and-workflow-orchestration)
7. [The Data Layer and Integration Patterns](#7-the-data-layer-and-integration-patterns)
8. [Operational Implications: EOD and Batch in a Distributed World](#8-operational-implications-eod-and-batch-in-a-distributed-world)
9. [The Vendor Landscape](#9-the-vendor-landscape)
10. [The Debate: The Coreless Reality Check](#10-the-debate-the-coreless-reality-check)
11. [Worked Example: Cymbal Bank Goes Coreless](#11-worked-example-cymbal-bank-goes-coreless)
12. [Claims Audit](#12-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [References](#15-references)

---

## 1. The Concept: What "Coreless" Banking Means

**Coreless banking** is an architectural approach to banking technology in which the functions traditionally performed by a single monolithic core banking system are decomposed into a network of independent, API-accessible services that a bank assembles — rather than buying one system that does everything. The name is deliberately provocative and, as Section 2 shows, only partially accurate: "coreless" does not mean *no core functions exist*; it means *no single monolithic core system owns them all*.

The most-cited working definition comes from the industry standards body that formalized the term. The **Banking Industry Architecture Network (BIAN)** — the not-for-profit banking-architecture standards association whose work this repository covers in [bian_banking_architecture_guide.md](bian_banking_architecture_guide.md) — launched its "Coreless Bank" initiative on **23 September 2019** with eleven members (PNC, Citi, Wells Fargo, JPMorgan Chase, Bangkok Bank, CIBC, IBM, Technisys, Infosys, Finxact and Zafin), defining the goal as: banks source and adopt new business services built as **API-based microservices** covering functions such as consumer payments, customer offers and consumer loans, with **plug-and-play interoperability**, **cloud deployability**, and **cross-domain orchestration** (BIAN, "BIAN Launches 'Coreless Bank' Initiative with Six Major Global Banks", 2019). The initiative has since iterated through four concept versions (1.0–4.0), with Coreless Banking 2.0 (announced 12 October 2021) adding message-model translation — each vendor's proprietary message model is translated into BIAN's standard message model so that best-of-breed components from different vendors can interoperate without custom point-to-point integrations.

ThoughtWorks — the global technology consultancy — popularized the same definition for engineers in its November 2024 blog post "Kill your core": *"Coreless in this banking context doesn't mean 'without a core banking system', but rather 'core banking system functions are accessed via an independent banking domain layer using well defined APIs'."* It then characterizes the architecture: coreless banking **decouples traditional monolithic core systems into a network of interconnected microservices grouped according to and managed in their respective business (sub)domains**, enabling banks to choose best-of-breed solutions per function, run them cloud-natively, and innovate without being constrained by legacy infrastructure.

### 1.1 From Monolithic Core to Composed Services

To see what "coreless" replaces, recall what a monolithic core actually is. The mechanics of the legacy core — the product factory, the transaction state machine, the batch/EOD pipeline, the COBOL/Assembler mainframe estate — are the subject of the sibling guides [core_banking_systems_guide.md](core_banking_systems_guide.md) and [core_banking_processes_guide.md](core_banking_processes_guide.md); this guide does not re-derive them. The essential point for the coreless argument is that a classic core (Temenos Transact, Oracle FLEXCUBE, Finastra Essence, TCS BaNCS, or an in-house mainframe system) bundles five distinct capabilities into one deployable, one-database, one-vendor package:

| Capability | What it does in a monolithic core | Coreless replacement |
|---|---|---|
| **Ledger & posting** | The double-entry engine, balances, value dating, GL integration | **Ledger-as-a-service (LaaS)** — a dedicated accounting/ledger service (Section 5) |
| **Product logic** | Loans, deposits, cards, term deposits — parameterized product definitions | **Product engines** — per-domain services (Section 6) |
| **Process/state** | The transaction lifecycle, maker-checker, workflow | **Workflow/orchestration service** (Sections 4.3, 6.4) |
| **Data** | One schema, one database, batch-extracted analytics | **Event-driven data layer** (Section 7) |
| **Integration** | Files, message queues, bespoke adapters | **API gateway + event backbone** (Section 7.2) |

In a coreless architecture, these five capabilities still exist — but as **independently deployable, independently scalable, independently replaceable services** that talk to each other over APIs and events. The bank's "core" becomes an architecture and a set of contracts (the BIAN service landscape, in BIAN's framing) rather than a product.

### 1.2 The Spectrum: Coreless vs Core-as-a-Service vs Composable Banking

"Coreless" sits at one end of a spectrum of modernization strategies that trade-press and vendors often conflate. The useful way to separate them is by *who owns the ledger and the product logic*:

```
  Monolithic core        Cloud core           Core-as-a-service      Coreless
  (on-prem, one           (same product,        (vendor runs the        (no single core
   vendor, one DB)        vendor-hosted/         core for you;           system; services
                          SaaS-ified)            bank consumes it)      assembled per domain)
  Temenos Transact       Temenos Banking       TCS BaNCS Cloud,        BIAN Coreless Banking
  FLEXCUBE on-prem       Cloud, FLEXCUBE       Oracle Banking Cloud    model; composed
                          on OCI, Finacle       Services                Vault/Mambu/Finxact
                          on cloud                                        estates
```

- **Cloud core** — the same monolithic product, re-hosted. The vendor (or a hyperscaler) runs the core in the cloud; the bank's architecture is unchanged. This is the bulk of what "core modernization" meant in the 2018–2023 era (see [ibm_cloud_guide.md](../technology/ibm_cloud_guide.md) and [financial_management_systems_guide.md](../technology/financial_management_systems_guide.md) for the platform context).
- **Core-as-a-service (CaaS)** — the core is consumed as a managed service: the vendor operates the core (SaaS) and exposes APIs; the bank stops running infrastructure but still runs *a* core. TCS BaNCS Cloud (Section 9.5) and Oracle Banking Cloud Services (Section 9.4) are explicit examples; Celent runs a whole "Core Banking in the Cloud" video series around them.
- **Composable banking** — the vendor-side label for selling *components* (deposit engine, lending engine, ledger) that the bank assembles; Mambu's "composable banking platform" positioning (Section 9.2) is the canonical case. Composable is coreless's commercial cousin: a composable *platform* is a set of replaceable parts sold by one vendor, whereas coreless in BIAN's sense is the *interoperability standard* that lets parts from many vendors be assembled.
- **Coreless** — the endpoint: no component is privileged as "the core"; the ledger, product engines, and orchestration are peers, and the bank's architecture — not a vendor's suite — is the system of record for how they fit together.

These are not binary. Section 10.4 covers the hybrid models — headless core, core-plus, coexistence — that dominate in practice.

### 1.3 The Critique: "There Is No Such Thing as a Coreless Bank"

The most important critique of the term is that it is **literally false as advertised and dangerous as a strategy**. Every bank must have a ledger: deposits, loans, fees, interest, and the general ledger are accounting facts, not optional features. Double-entry bookkeeping does not disappear because you decomposed the software that does it — the accounting mechanics (DR = CR, atomicity, no-delete, audit trail) remain, as the posting-engine sibling guide [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) demonstrates in full. The critique runs:

- **A ledger is still a core.** However you slice it, someone owns the system of record for "how much money does this customer have". That system enforces the bank's accounting invariants; calling the architecture "coreless" because that system is now an API service called `ledger` is marketing, not architecture. ThoughtWorks concedes exactly this in its definition — coreless "doesn't mean without a core banking system".
- **A bank is not a software startup.** The BIAN pilot itself was led by incumbent banks precisely because the monolithic core's risk, compliance, and reporting properties (BCBS 239 lineage, MAS notice 644 resilience, audit) must be preserved; the question is where the invariants live, not whether they exist.
- **Distributed ≠ no core; distributed = distributed core.** Replacing one monolith with twenty services moves the complexity into the integration layer: a distributed transaction that used to be one atomic posting in the core becomes an orchestrated saga across the ledger, the product engine, and the payments hub. The failure modes change (Section 10.5); they do not disappear.

For these reasons the honest framing — used throughout this guide — is that **coreless banking is the decomposition of the core, not its abolition**: the ledger and its posting invariants remain the irreducible "coreless core" (Sections 5, 11.5). The exact phrase "there is no such thing as a coreless bank" circulates in analyst and trade commentary as a shorthand for this critique; attribution of the specific quote could not be pinned to a single primary source and is flagged in Section 13.

## 2. The Terminology Debate: Coreless, Composable, and Core-as-a-Service

The vocabulary around coreless banking is genuinely contested, and the contest matters because it drives buying decisions. Three terms — **coreless**, **composable**, **core-as-a-service** — are used almost interchangeably in trade press, yet they carry different origins, different vendors behind them, and different architectural commitments (Section 1.2).

### 2.1 Who Coined "Coreless Banking"?

The verified history is: **BIAN formalized and popularized "coreless banking" as a term of art in 2019** — the "Coreless Bank" initiative launched 23 September 2019 with six marquee banks among its eleven founding supporters (PNC, Citi, Wells Fargo, JPMorgan Chase, Bangkok Bank, CIBC, plus IBM, Technisys, Infosys, Finxact, Zafin). The term was not invented by a single vendor: third-party commentary (e.g., the WAU.com explainer "Coreless Banking: When Your Core Stops Being a Single Platform") states explicitly that the term "wasn't coined by a vendor; it was formalized by BIAN". The exact first use of "coreless" in a banking context before 2019 is not documented in any primary source this guide could verify (Section 13).

What *is* verifiable is that the term entered the vendor and analyst mainstream through BIAN's initiative and its participants:

- **Thought Machine** joined BIAN in August 2020 and participates in the Coreless Banking Initiative — its own announcement ("Shaping the future of banking IT: We've joined BIAN", thoughtmachine.net) says Thought Machine will "help BIAN achieve its vision of creating a universally compatible core banking infrastructure based on microservices technology through its Coreless Banking Initiative", and its COO Gareth Richardson appeared on the BIAN panel presenting the Coreless Banking roadmap.
- **BIAN Coreless Banking 2.0** (12 October 2021) was developed "with experts from DXC, IBM-Redhat, JPMorgan Chase, PNC, Salesforce, Tata Consultancy Services, Thought Machine and Zafin" — the first time Thought Machine's name appears on the initiative's development team.
- **ThoughtWorks** (November 2024, "Kill your core") is the most influential engineering-side popularizer, explicitly anchoring its definition to BIAN's Coreless Banking Model.
- **Trade press and blog adoption** followed: explainers such as "The Critical Shift to Coreless Banking" (Medium) and vendor-adjacent posts (WAU.com) use the term as established by the early 2020s.

**Paul Taylor** — Thought Machine's founder (Section 9.1) — is frequently quoted on the *failure of legacy cores* and on cloud-native core banking, and Thought Machine markets Vault Core as the cloud core that enables modern architectures; however, this guide found **no primary source in which Taylor claims to have coined the term "coreless banking"** (Section 13). The safe, verified attribution is: BIAN (2019) for the term's formalization, Thought Machine and ThoughtWorks for its popularization in the cloud-core vendor and engineering communities respectively.

### 2.2 Gartner's Composable Business and the Analyst Lexicon

"Coreless" entered an analyst vocabulary that was already being reshaped by **Gartner's "composable" concept**. Gartner publicly launched **composable business** as a strategic concept in **October 2020** (Gartner IT Symposium/Xpo keynote, "The Future of Business Is Composable"; press release "Gartner Says Organizations Should Strive for Composability to Be Resilient and Agile During Uncertainty", 19 October 2020), defining **composable digital business** as applying the principles of **modularity, autonomy, orchestration, and discovery** to business architecture. The banking translation — **composable banking** — applies those principles to banking technology: assembling financial services from independent components rather than fixed modules.

Two things follow for the terminology debate:

1. **"Composable" is an analyst concept that vendors imported into banking.** Gartner's composable business (2020) is a *business-architecture* idea; the banking vendors adopted the word. **Mambu** now claims to have "pioneered composable core banking" (mambu.com, "Composability is how modern banking is built" — verified vendor claim, Section 9.2), defining composable banking as "the ability to assemble financial services from independent components, rather than being constrained by fixed systems or predefined modules".
2. **Analyst firms differ in vocabulary.** Celent's public "Core Banking in the Cloud" webinar series (e.g., its 29 May 2024 episode on TCS BaNCS Cloud with principal analyst Daniel Mayo) frames the debate as *core migration to cloud-native platforms*, "intentionally non-evaluative" about the new platform vendors. McKinsey frames it as *core modernization without breaking the bank* — "removing complex customisation in their current core platform... while assembling best-of-class capabilities outside of their core platform" ("Modernizing core technology, without breaking the bank", Banking Matters). Neither uses "coreless" as their primary frame; the term lives mainly in BIAN's standards work, vendor marketing, and the trade press that covers both. (Analyst-report titles using "coreless" could not be verified — Section 13.)

### 2.3 Marketing-Driven Terminology and Vendor Positioning

The term's commercial weight is its problem. "Coreless" is a **negative-space brand**: it defines the competitor (the monolithic core) rather than the product, which makes it ideal marketing and poor specification. Observed positioning, verified from vendor sites:

- **Thought Machine**: sells **Vault Core** — "the clean, cloud-native core banking and integrated payments infrastructure" — and its materials emphasize *replacing* legacy cores and *coexistence* with them (Section 9.1). It does not market "no core"; it markets *one unified ledger with decoupled product logic* — arguably the opposite of multi-ledger coreless, which makes Thought Machine's BIAN membership a careful straddle: standards-compatible components, single-ledger core.
- **Mambu**: markets "composable core banking" explicitly (Section 9.2) — a vendor-branded version of coreless where the assembly happens inside Mambu's ecosystem.
- **BIAN**: uses "coreless" in the standards sense — service definitions as "building blocks that can be assembled and implemented in unique ways" (Coreless Banking 4.0 description, bian.org).
- **10x Banking**: markets a **cloud-native core** (10x SuperCore) that replaces legacy infrastructure "without disrupting live operations" (10xbanking.com) — again a core-replacement story, not a no-core story.

The pattern: **"coreless" is claimed most loudly by standards bodies and challengers, and quietly qualified by every vendor that actually sells a core**. Section 10 treats the resulting hype-vs-reality gap as the central design risk for architects evaluating the approach.

## 3. The Drivers and the Legacy Constraint

Why would a bank abandon the monolithic core that has run its books for decades? The drivers are economic, architectural, and competitive; the constraint is the legacy estate itself. The legacy mechanics (the core's module structure, product parameterization, batch windows, and why they resist change) are covered in [core_banking_systems_guide.md](core_banking_systems_guide.md) and [core_banking_processes_guide.md](core_banking_processes_guide.md) — this section only summarizes them as *drivers*.

### 3.1 Cloud Economics

The first driver is the economics of running banking workloads in the cloud rather than on mainframes and in owned data centers. The platform-side story (infrastructure-as-code, elasticity, managed services, multi-cloud) is the subject of [ibm_cloud_guide.md](../technology/ibm_cloud_guide.md) and [financial_management_systems_guide.md](../technology/financial_management_systems_guide.md); the core-specific economics are:

- **Capacity follows the business, not the forecast.** A monolithic core must be sized for the peak batch window (Section 8) and the worst-case day; a decomposed service estate scales each service independently — the ledger horizontally, the card engine not at all.
- **OpEx replaces CapEx.** Vendors and analysts (Celent's "Core Banking in the Cloud" series; McKinsey's "Modernizing core technology, without breaking the bank") frame cloud cores as shifting the IT model "from in-house, on-premises banking legacy systems to modern cloud-based systems provided by third-party vendors" (Deloitte's phrasing in "Modernizing legacy systems in banking").
- **The legacy cost base is documented and brutal.** The Federal Reserve Bank of Kansas City's research briefing "Core Banking Systems and Options for Modernization" notes many depository institutions still run legacy cores "up to 40 years old" on mainframes in outdated languages — a cost and skills trap the cloud-core vendors and BIAN both cite as the perpetual problem "legacy core infrastructure" poses.
- **Vendor TCO claims are aggressive and contested.** Mambu claims "up to 50% lower TCO than legacy cores" (mambu.com — vendor claim, flagged ⚠ in Section 12); 10x claims its platform "reduces client costs" and runs at "99.99% uptime" processing "over 10,000 transactions per second" (10xbanking.com — vendor claims). These are marketing figures, not audited benchmarks.

### 3.2 API-First Architecture

The second driver is architectural: banking moved from file-based and message-queue integration to an **API-first** world, and the monolithic core is a poor API citizen. The full middleware and integration-platform picture (ESB vs API gateway vs event backbone, canonical data models, contract versioning) is in [enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md) — this guide does not re-derive it. The core-specific consequences:

- **Channels demand APIs; cores expose screens and files.** Digital banking, open banking (MAS's API guidance in [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)), and embedded finance all consume account opening, balance, and payment functions as APIs. A monolithic core exposes these through batch files, terminal screens, or vendor-specific protocols (OFS in Temenos, FLEXCUBE's UBX — see [temenos_data_model_guide.md](temenos_data_model_guide.md) and [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md)); wrapping them in APIs produces the "screen-scraping the core" anti-pattern.
- **API-first is the vendor consensus.** Every vendor verified for Section 9 — Thought Machine, Mambu, 10x, Finxact, Oracle Banking Cloud Services, TCS BaNCS Cloud — markets API-first or API-native architecture; TCS BaNCS's marketplace listing describes "An API-First Platform... TCS BaNCS' Open banking capabilities help unlock the new business models of Banking-as-a-Service and embedded finance".
- **BIAN's coreless model is itself an API standard.** The whole point of the Coreless Banking Initiative is that BIAN's service landscape and message model become the API contract between components (Section 1, and [bian_banking_architecture_guide.md](bian_banking_architecture_guide.md) for the service landscape itself).

### 3.3 Time-to-Market, Product Innovation, and Cost

The third driver is the competitive one, and it is the one incumbents feel most acutely:

- **Product launch speed.** BIAN's launch statement: legacy cores make it "incredibly difficult and costly to innovate at the speeds required to fulfil customers' needs" (Steve Van Wyk, BIAN Board Chairman, 2019). Coreless/composable vendors quantify the delta: Thought Machine's messaging is "launch any product instantly"; Mambu's is "bring new products to market faster by assembling capabilities rather than rebuilding core systems".
- **The digital-bank proof point.** The challengers that went live fastest in the 2019–2024 wave did so on cloud cores: SEB's UNQUO launched on Thought Machine's Vault Core in 2019; Singapore's digital banks (see [gxs_bank_guide.md](gxs_bank_guide.md), [maribank_guide.md](maribank_guide.md), [trust_bank_guide.md](trust_bank_guide.md) — Trust Bank runs on Mambu) went from license to live in months, a timeline a core replacement programme of a decade cannot match.
- **Cost of change.** McKinsey's verified framing is that banks should remove "complex customisation" from their current core and "assemble best-of-class capabilities outside of their core platform" — i.e., the *economics of the coreless approach* (buy components, integrate by API) beat the economics of *customizing the monolith* (each release is a re-test of the whole).

### 3.4 The Legacy Constraint: Why the Monolithic Core Is Hard to Replace

The counterweight to every driver above is the legacy estate itself. The detailed mechanics — product parameterization, the multi-thousand-table schema, COBOL/Assembler code, the EOD batch dependency chain, the skills gap — are in [core_banking_systems_guide.md](core_banking_systems_guide.md) and [core_banking_processes_guide.md](core_banking_processes_guide.md); the architectural summary relevant to coreless is:

- **The core is not one system, it is a constellation.** A typical incumbent runs a primary retail core *plus* a card system *plus* a mortgage system *plus* a payments hub, each with its own ledger — the ThoughtWorks "Kill your core" history describes exactly this evolution: banks "added specialized core ledgers for credit cards and mortgages", creating the fragmented estate that coreless *reassembles* rather than replaces.
- **Replacement risk is existential.** Core replacement failures are among the most expensive events in banking IT — TSB's 2018 migration and RBS's 2012 batch failure are documented in Section 10.5. The asymmetry (a failed big-bang migration is career-ending and regulator-visible; a slow strangler is merely embarrassing) is why every serious coreless programme is an *incremental strangulation*, not a cutover (Section 11.4).
- **The core is entangled with everything.** Regulatory reporting (BCBS 239, MAS notices — see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)), the GL close, interest accruals, and the EOD pipeline all assume the core's single daily state; any decomposition must reproduce those contracts before the core can be switched off (Sections 5, 8).

**Net driver assessment:** the economic and competitive case for decomposing the core is strong and verified across vendors and analysts; the binding constraint is never technology choice but *migration risk* — which is why the rest of this guide is mostly about the architecture that makes migration survivable (Sections 4–8), the vendors that sell it (Section 9), and the cautionary tales that define the failure modes (Section 10.5).

## 4. The Reference Architecture: What Replaces the Core

This section is the architectural heart of the guide: the service decomposition that replaces the monolithic core, drawn from the BIAN coreless model (Section 1), the ThoughtWorks decomposition, and the vendor platforms of Section 9. It is a *reference* architecture — the concrete Cymbal Bank instantiation is Section 11.

### 4.1 The Service Decomposition Map

The coreless decomposition follows **domain boundaries, not product boundaries**. The classic mistake is to decompose by product (a "current account service", a "savings service") and end up with duplicated ledgers and duplicated customer state; the coreless discipline is to decompose by *capability* so that products are assembled from shared capabilities. The canonical map:

```
                    ┌──────────────────────────────────────────────┐
                    │            CHANNELS / EXPERIENCE              │
                    │   (mobile, web, API partners, branches)      │
                    └───────────────────┬──────────────────────────┘
                                        │ APIs
                    ┌───────────────────▼──────────────────────────┐
                    │          API GATEWAY / BFF LAYER             │
                    └───┬───────────┬───────────┬───────────┬──────┘
                        │           │           │           │
        ┌───────────────▼──┐  ┌─────▼──────┐  ┌─▼───────────┴───┐
        │ CUSTOMER & KYC   │  │ PRODUCT    │  │ WORKFLOW /      │
        │ (party, consent) │  │ ENGINES    │  │ ORCHESTRATION   │
        └───────────────┬──┘  │ (loans,    │  │ (process state, │
                        │     │  deposits, │  │  maker-checker) │
                        │     │  cards)    │  └───────┬──────────┘
                        │     └─────┬──────┘          │ events
                        │           │                 │
        ┌───────────────▼───────────▼─────────────────▼──────────┐
        │                 LEDGER / ACCOUNTING LAYER              │
        │   (posting engine, balances, GL, LaaS)  ← Section 5    │
        └───────────────┬────────────────────────────────────────┘
                        │ events (outbox → stream)
        ┌───────────────▼────────────────────────────────────────┐
        │   DATA LAYER: event stream, data lake, reporting       │
        │   (regulatory, analytics, reconciliation) ← Section 7  │
        └────────────────────────────────────────────────────────┘
```

Each box is a **service or service group with its own data store** (or, in the pragmatic variant, its own schema on a shared database — Section 11.3). The two non-negotiable layers are the **ledger layer** (someone must own the money) and the **data layer** (someone must own the truth for reporting).

### 4.2 The Product Engines

Product engines are the services that *calculate* — the interest engine, the fee engine, the limit engine, the loan lifecycle — as distinct from the ledger that *records*. In the monolithic core these are modules inside one process (Temenos's interest/charges applications, FLEXCUBE's product parameterization); in the coreless model they are standalone services that:

- **Hold product state, not money.** A loan engine tracks principal, schedule, and arrears state; every cash movement it produces is handed to the ledger as a posting instruction (Section 5.1). This is the crucial separation: the engine *decides*, the ledger *records*.
- **Are replaceable per domain.** The BIAN 1.0 pilot deliberately chose consumer loans, payments, and offers as the first services — three domains with different change cadences and vendor markets.
- **Communicate by events.** A repayment event from the loan engine triggers a debit in the ledger and a limit update in the limits service — via the event backbone (Section 7), not synchronous calls between every pair.

The detailed mechanics of interest and posting engines are the subjects of [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) and [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md); the coreless-specific point is *which* service owns *which* calculation and how they hand off to the ledger.

### 4.3 Workflow and Orchestration

The workflow layer replaces the core's built-in transaction state machine (the INITIATED → VALIDATED → AUTHORIZED → POSTED → CONFIRMED lifecycle of [core_banking_processes_guide.md](core_banking_processes_guide.md)) with an explicit **orchestration service**:

- **Process state lives outside the engines.** A loan origination is a process spanning KYC, credit decisioning, document management, and disbursement; the orchestration service holds the process instance state and calls each domain service in turn.
- **Sagas, not distributed transactions.** Where a legacy core would post atomically across modules (Section 8.2 of the posting guide), a coreless flow is a **saga**: a sequence of local transactions with compensating actions (reverse the posting, cancel the disbursement) on failure. The outbox pattern (Section 7.1) gives exactly-once semantics for the events between saga steps.
- **Maker-checker and audit move into the workflow layer.** The dual-control and audit-trail obligations that the core enforced internally must be re-implemented as workflow rules, because no single service can see the whole process anymore.

### 4.4 The Data Layer

The data layer is where coreless either succeeds or dies (Section 7). The one-line summary: **the event stream is the new database of record for everything except the ledger itself.** The ledger keeps its own transactional store; everything else — customer 360, regulatory reporting, analytics, reconciliation — is fed by the stream of events the services emit. This is why BIAN's message-model translation (Coreless 2.0) matters: if every vendor speaks a different event schema, the data layer becomes a translation swamp.

### 4.5 The Architecture Sketch

The full reference architecture in one diagram — with the cross-references to where each layer is treated in depth:

```
CHANNELS ──▶ API GATEWAY ──▶ [customer | product engines | workflow]   §6
                                   │  posting instructions (API)
                                   ▼
                              LEDGER (LaaS)                            §5  → posting engine guide
                                   │  events (outbox)
                                   ▼
                    EVENT STREAM ──▶ data lake / reporting / recon     §7  → middleware guide
                                   │  batch orchestration (EOD)
                                   ▼
                         WORKFLOW ORCHESTRATOR (Airflow-class)         §8  → apache_airflow_guide
```

Each arrow is a **contract** (OpenAPI or async event schema); the bank's architecture team owns the contracts, the vendors own the implementations — that is the entire coreless bargain.

## 5. The Accounting Layer: Ledger-as-a-Service

The ledger is the "coreless core": the one component that cannot be decomposed away. This section covers what a ledger service must do, the single-vs-multiple-ledger decision, and the accounting implications of decomposition. The double-entry mechanics themselves — Pacioli, DR = CR, T-accounts, posting lifecycle, balance components, GL integration — are the subject of [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) and are **not** re-derived here; this section references its sections and adds the coreless-specific layer on top.

### 5.1 What a Ledger Service Must Do

A ledger-as-a-service (LaaS) is the posting engine delivered as an API product (the posting guide's Section 10 already names this as the "ledger as a service" trend). Whatever vendor supplies it — Vault's posting engine, Mambu's journaling, a Modern-Treasury-class LaaS (see [full_stack_banking_guide.md](full_stack_banking_guide.md)), or a home-grown one — it must preserve the posting engine's non-negotiable invariants (posting guide Sections 8.2–8.4): **atomicity** (all legs or none), **idempotency** (retries cannot double-post), **no-delete** (reversals, not deletions), **audit trail** (every entry linked to its transaction), and **DR = CR** enforced at the engine, not the caller.

The coreless-specific requirements on top of those invariants:

- **Posting instructions as the unit of work.** Product engines do not post; they submit a **posting instruction** — a balanced set of legs with accounts, amounts, value dates, and a transaction reference — and the ledger executes it atomically and returns the posted entries. This is exactly the Vault model ("posting instructions", atomic units with balance coordinates and phases — see the posting guide §7.4) and the Mambu model (journal entries with ≥1 DR + ≥1 CR via its accounting API — posting guide §7.3).
- **Idempotency keys everywhere.** In a distributed flow, the same instruction can be retried by the orchestrator, the channel, or the network; the ledger must deduplicate on the client-supplied transaction reference.
- **Balance components and phases survive.** Available vs ledger vs cleared balances, holds, and value dating do not disappear because the ledger is an API; the posting guide's balance-component ladder (§5.1) is exactly what the ledger service must still compute and expose.
- **GL integration as a service.** The GL close, suspense handling, and trial balance (posting guide §6) become scheduled jobs *against* the ledger service (Section 8), not internal batch steps.

### 5.2 Single Ledger vs Multiple Ledgers

The most consequential architecture decision in a coreless design is **how many ledgers** the bank runs:

| Option | Description | Verified examples / trade-offs |
|---|---|---|
| **Single unified ledger** | One ledger service holds all customer money positions; product engines are stateless calculators on top | **Vault Core's model** — Thought Machine markets "data unified on one ledger" with product logic decoupled into smart contracts (thoughtmachine.net). Strongest for the 360° customer view and simplest reconciliation; weakest for domain isolation |
| **Multiple product ledgers** | Each domain (cards, mortgages, deposits) keeps its own ledger, reconciled to a consolidated GL | The *legacy reality* — ThoughtWorks's history shows banks already run "specialized core ledgers for credit cards and mortgages". Coreless 1.0's BIAN pilot implicitly accepts this by treating domains as plug-and-play services |
| **Hub-and-spoke (thin universal ledger + specialist ledgers)** | A thin next-gen ledger interoperates with specialist cores for products like cards or asset finance | **ThoughtWorks's recommended pattern** — "coreless banking architecture naturally supports multi-core or co-existence deployments, where universal, thin, next-gen ledger can interoperate with a specialist core banking system" |

The decision rule: **single ledger if the bank's products share one customer money position (current accounts + savings + cards from one balance); multiple ledgers if domains are genuinely separate books that already reconcile today.** Section 11.3 works this through for Cymbal Bank.

### 5.3 Accounting Implications of Decomposition

Decomposition changes *where* accounting happens, not *what* accounting is:

- **The suspense account becomes a service.** In a monolith, an unresolvable posting lands in suspense inside the core (posting guide §6.4); in a coreless world, the ledger service still owns suspense, but the *reason* for suspense is often cross-service — a posting instruction arrived without a valid GL account because the product engine and the chart of accounts disagree. The chart of accounts becomes a shared, versioned contract (see [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) for COA modeling).
- **Inter-service money movement needs clearing accounts.** A transfer from a deposit-engine product to a card-ledger product is now a *cross-ledger* movement; the bank needs explicit inter-ledger clearing accounts and a reconciliation job to prove them (Section 8.3). In a single-ledger design this problem vanishes — the strongest architectural argument for Vault-style unification.
- **Value dating and back-value postings must be first-class API semantics.** When the channel, the product engine, and the ledger are different services, "book today, value yesterday" cannot be a hidden batch adjustment; it must be an explicit parameter of the posting instruction (posting guide §2.5).
- **The trial balance and GL close still run — as orchestrated jobs.** The accounting *calendar* (daily close, monthly close, regulatory submission dates) is unchanged; only the execution engine changes (Section 8).

### 5.4 The LaaS Market

The ledger-service market is real and growing, which is itself evidence that "the ledger is still a core" is the industry's working assumption. Verified categories:

- **Core-vendor ledgers**: Vault Core's posting engine (Thought Machine), Mambu's accounting engine — both expose postings via API (posting guide §§7.3–7.4).
- **Standalone LaaS providers**: the Modern Treasury class of products (the posting guide §8.7 and [full_stack_banking_guide.md](full_stack_banking_guide.md) cover these) — double-entry ledgers as pure API products for embedded finance.
- **Open-source ledgers**: Apache Fineract (see [fineract_database_models_guide.md](fineract_database_models_guide.md) and the [fineract_interest_background_jobs_guide.md](fineract_interest_background_jobs_guide.md)) provides a working open-source reference for the ledger-plus-interest-job pattern that coreless architectures replace or embed.

The LaaS decision for Cymbal Bank is worked in Section 11.3.

## 6. Product Engines and Workflow Orchestration

### 6.1 Deposits and Payments

The deposit domain in a coreless world splits into: the **deposit product engine** (product parameters, interest rates, terms, maturity — see [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) for the interest mechanics), the **ledger** (the money), and the **payments hub** (the rails — FAST/PayNow, MEPS+ in Singapore; see [payments_hub_guide.md](payments_hub_guide.md) and [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)). The BIAN coreless pilot's choice of **consumer payments** as a first service is instructive: payments are the highest-volume, highest-change-cadence domain, and the one where a plug-and-play service (e.g., a specialist payments provider behind BIAN-standard APIs) delivers the fastest visible win.

The architectural rule: **the payments hub never touches balances directly.** It sends posting instructions to the ledger (Section 5.1) and reads available balances from the ledger's balance API — the same separation a monolithic core enforces internally, now enforced across services by contract.

### 6.2 Loans and Credit

The loan engine owns the **contract lifecycle**: origination, disbursement, schedule, accrual, repayment, arrears, write-off. It is the heaviest product engine — which is why BIAN 1.0 also picked **consumer loans** as a pilot service and why the vendor market is deepest here (Mambu's lending engine, Finxact's platform, Thought Machine's smart-contract products, plus specialist LOS vendors).

The coreless-specific complexity is **the handoff to the ledger on every cash event**: disbursement, principal repayment, interest payment, fee, and arrears penalty each produce a posting instruction with a prescribed GL treatment (the loan accounting treatment — accrual vs cash, interest income GLs — is in the posting guide and [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §6). The loan engine must therefore embed the *accounting contract* (which GL accounts each event touches) — product logic and accounting logic are coupled here by design, and the chart of accounts contract (Section 5.3) is what keeps it decoupled from the ledger implementation.

### 6.3 Cards

Cards are the domain that most often *stays* on a specialist system in a coreless estate — ThoughtWorks explicitly cites cards as the canonical specialist ledger that interoperates with a thin universal ledger (Section 5.2). The reasons: card authorization is a real-time, high-availability, network-driven process (Visa/Mastercard switch interfaces) with its own settlement cycle, and card ledgers are usually separate books even in monolithic banks. In a coreless design the card system is wrapped in BIAN-standard APIs (authorization, settlement, statement) and its daily settlement posts into the main ledger through the standard posting-instruction path — the "hub-and-spoke" pattern of Section 5.2.

### 6.4 Workflow Orchestration and Process State

Section 4.3 introduced the orchestration layer; the operational detail is that it is a **state machine as a service**:

- **Process definitions** (loan origination, account opening, KYC remediation) are versioned workflows; each process instance has a state, an owner, and a deadline.
- **Human tasks** (maker-checker approvals, exceptions) are first-class workflow objects with SLAs — replacing the core's internal queue-and-approve screens (see [core_banking_processes_guide.md](core_banking_processes_guide.md) §2 for the maker-checker semantics being re-hosted).
- **Time is an event source.** Schedules (next repayment, term-deposit maturity, fee date) are emitted by the workflow layer as events, triggering the product engines — the distributed replacement for the core's nightly batch triggers (Section 8).

## 7. The Data Layer and Integration Patterns

### 7.1 The Data Layer: Events, Replication, and Analytics

The data layer is the coreless architecture's answer to "where is the system of record for everything that isn't money?" The pattern, in order:

1. **Outbox → stream.** Every service writes its state change and an event to its **outbox** atomically; a relay publishes to the **event backbone**. This is the outbox pattern of the posting guide §10 and [event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) — exactly-once, ordered, replayable.
2. **Stream → lake.** The event backbone feeds the data lake; analytics, regulatory reporting, and reconciliation consume from the lake, never from live service databases (protecting services from reporting queries — the "reporting must not touch the OLTP" rule that monolithic cores violate internally).
3. **Lake → truth for reporting.** BCBS 239-style data lineage (see [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)) is *easier* in a coreless world — every number can be traced to an event — but only if the events carry the full context (transaction ref, GL codes, product, customer, value date).

The **customer 360** and **party/consent** data (KYC, consent management, relationship hierarchy) is a service in its own right (Section 4.1) and the natural owner of the customer identifier every other service references. This guide's sibling [data_models_banking_insurance_guide.md](data_models_banking_insurance_guide.md) covers the canonical party/account/GL data models that the data layer must reconcile to.

### 7.2 Integration Patterns

The integration patterns of a coreless estate are the general middleware patterns of [enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md) applied to banking domains. The pattern choices that matter:

- **Synchronous APIs for reads and commands; events for facts.** Balance enquiry, account open, and payment initiation are API calls; "repayment happened", "limit breached", "account closed" are events. Mixing the two (publishing events for commands, or calling APIs to discover facts) is the classic failure.
- **The API gateway is the only integration point for channels; the event backbone is the only integration point between services.** This is the BIAN coreless discipline — bank-consumable API interface plus cross-domain orchestration (Section 1) — and it is what keeps the estate "plug and play": a vendor swap touches one contract, not N point-to-point adapters.
- **Message-model translation is a first-class component.** BIAN 2.0's contribution (translate each proprietary message model into BIAN's standard message model) is the pattern for onboarding vendors that do not natively speak the bank's canonical events; it belongs in the integration layer, not in every service.
- **Files do not die.** Statements, regulatory submissions, and interbank files (SWIFT, FAST batch) remain file-based; the integration layer terminates files at the edge and converts them to/from the internal event/API model (see [swift_alliance_access_guide.md](swift_alliance_access_guide.md) and [swiftnet_fileact_guide.md](swiftnet_fileact_guide.md) for the file-rails reality).

## 8. Operational Implications: EOD and Batch in a Distributed World

### 8.1 The Death of the Batch Window?

The monolithic core's **end-of-day batch** — the sequenced overnight run that posts accruals, fees, and maturities, closes the GL, and produces reports (the EOD pipeline of [core_banking_processes_guide.md](core_banking_processes_guide.md) §7) — is the single biggest operational artifact the coreless architecture must replace. The coreless claim is not that batch dies; it is that **batch becomes orchestration of distributed jobs, and the daily close becomes a continuous process**. The batch themes — scheduling, dependency graphs, idempotent job reruns, SLA windows — are the general subject of [apache_airflow_guide.md](../technology/apache_airflow_guide.md); this section applies them to the coreless estate.

### 8.2 Distributed EOD: Orchestration, Idempotency, and Ordering

A coreless EOD looks like this:

```
  (all day, continuously)           (scheduled, orchestrated)
  interest accrual events ──┐
  fee events ───────────────┼──▶ ledger posts in real time ──▶ balances always current
  maturity events ──────────┘
                                    22:00  workflow: run product-engine sweeps
                                           (each emits posting instructions)
                                    23:00  workflow: GL close job (vs ledger)
                                    23:30  workflow: reconciliation jobs (Section 8.3)
                                    00:30  workflow: regulatory pack generation
                                    01:00  SLA: all jobs complete, reports published
```

The operational rules that make this survivable:

- **Real-time posting shrinks the batch.** If accruals, fees, and maturities post as events during the day (the "always-on" trend of the posting guide §10), the midnight batch's job is reduced to *closing and proving*, not *computing*.
- **Every job is idempotent and re-runnable.** The orchestration layer (Airflow-class, or the vendor's scheduler — Vault has EOD schedule groups; Mambu runs background jobs, cf. [fineract_interest_background_jobs_guide.md](fineract_interest_background_jobs_guide.md) for the open-source analogue) must support partial rerun without double-posting — which is exactly why the ledger's idempotency keys (Section 5.1) are non-negotiable.
- **Ordering is explicit, not implicit.** The monolith's batch had an implicit order baked into its program structure; a distributed estate needs an explicit **dependency graph** per day (accruals before close, close before reports), versioned and testable — the Airflow DAG model ([apache_airflow_guide.md](../technology/apache_airflow_guide.md)).
- **Time zones and value dating cross services.** A multi-country bank's "end of day" is a rolling set of closes per book; each close is a job that must wait for that book's event stream to quiesce — a distributed-consistency problem the monolith never had.

### 8.3 Reconciliation in a Coreless World

Reconciliation is where coreless estates prove themselves daily. The reconciliation surface grows (every service boundary is a potential drift point) and the tooling is the same as ever: prove that the sum of the parts equals the whole. The mandatory reconciliations:

- **Ledger ↔ GL**: the ledger service's trial balance vs the general ledger control accounts (posting guide §6.6) — the daily DR = CR proof.
- **Ledger ↔ product engines**: the loan engine's aggregate outstanding vs the ledger's loan balances — catching "engine thinks X, ledger thinks Y" drift.
- **Ledger ↔ specialist ledgers** (cards, mortgages): the inter-ledger clearing accounts of Section 5.3.
- **Ledger ↔ payments**: FAST/SWIFT/MEPS+ outbound vs the payments hub's records vs the ledger postings — the classic payments reconciliation, now spanning three systems instead of two.

The pattern: **reconciliation jobs are first-class citizens of the EOD orchestration** (Section 8.2), with agreed tolerances, exception queues, and SLA reporting — the operational layer the BIAN message-model standardization exists to keep manageable.

---

## 9. The Vendor Landscape

The vendor market splits into two camps with different relationships to "coreless": the **cloud-core/composable challengers** (Thought Machine, Mambu, 10x, Finxact, nCino) that sell the components a coreless estate is assembled from, and the **legacy vendors' cloud offerings** (Temenos, Oracle, Finastra, TCS, Fiserv) that sell the same core as a service. All facts below were verified against vendor sites, Wikipedia, or trade press during research for this guide; vendor marketing claims are marked as such.

### 9.1 Thought Machine

**Verified facts** (Wikipedia; thoughtmachine.net; press):

| Fact | Detail |
|---|---|
| **Founded** | 2014, London, by **Paul Taylor** — ex-Google (his speech-technology company was acquired by Google in 2010; he led Google's text-to-speech team; PhD in speech technology) |
| **Locations** | HQ London (Herbrand Street, Bloomsbury); offices in New York, Singapore (APAC office launched 2019), Sydney, Melbourne |
| **Products** | **Vault Core** — cloud-native core where financial products are written as smart contracts with real-time data access; **Vault Payments** |
| **Vault Core architecture** | "Keeps your data unified on one ledger" while decoupling business logic into three layers: *product* (programmable smart contracts), *capability* (ledger workflows), *cloud* (multi-cloud, vendor-independent deployment); adoption models: Coexistence, Replatforming, Greenfield (thoughtmachine.net/vault-core) |
| **Funding** | $25M Series A (2018, Lloyds participated); $125M Series B (2020); $200M Series C (Nov 2021) → $1B valuation; $160M Series D (2022, led by Temasek, with Intesa Sanpaolo and Morgan Stanley) → $2.7B valuation |
| **Clients (reported)** | JPMorgan Chase (retail bank), Lloyds Banking Group, Standard Chartered, Intesa Sanpaolo, Kiwibank, SEB (UNQUO, 2019), M1, Arvest Bank, HD Bank (Vietnam), C6 Bank |
| **BIAN** | Joined August 2020; participant in BIAN's Coreless Banking Initiative (2.0 development team); COO Gareth Richardson on BIAN's coreless panel |
| **Analyst recognition** | Named a Leader in the 2025 Gartner Magic Quadrant for Retail Core Banking (Thought Machine press release) |

Thought Machine is the most important vendor to understand in a coreless discussion because it straddles both worlds: it sells **a** core (Vault Core, one ledger) yet is the loudest vendor voice in BIAN's coreless standards work. Its position is coherent — a BIAN-standard, API-driven core that coexists with other components is exactly what "coreless" estates need as their ledger layer (Section 5.2).

### 9.2 Mambu

**Verified facts** (Wikipedia; mambu.com; WSJ):

| Fact | Detail |
|---|---|
| **Founded** | 2011, Berlin; now **Mambu GmbH** — German-Dutch, HQ **Amsterdam** |
| **Founders** | **Eugene Danilkis, Frederik Pfisterer, Sofia Nunes** — three of the five Carnegie Mellon MHCI students whose microfinance field research in Mozambique became the company (full student team: Josh Coe, Danilkis, Nunes, Pfisterer, Joydeep Sengupta) |
| **Model** | Pure **SaaS** — "infrastructure for banks and financial service providers as a software as a service (SaaS) model" (Wikipedia); multi-tenant cloud platform |
| **Positioning** | "Composable core banking" — "Mambu pioneered composable core banking" (vendor claim, mambu.com); composable banking defined as "the ability to assemble financial services from independent components, rather than being constrained by fixed systems or predefined modules" |
| **Platform** | Lending, deposits, payments engines; open APIs; "open by design", "interoperable by default" |
| **Funding/valuation** | Early funding from Commerzbank and Point Nine; €200M round led by EQT (June 2021); valued at **$5.3B** (December 2021, WSJ); $165M contracted ARR (2022) |
| **Leadership** | Danilkis stepped down as CEO June 2023; CTO **Fernando Zandona** named interim then permanent CEO (August 2023) |
| **Notable deployments** | Trust Bank Singapore (see [trust_bank_guide.md](trust_bank_guide.md)); N26 reported as a customer (Business Insider via Wikipedia) |
| **TCO claim** | "Up to 50% lower TCO than legacy cores" (vendor marketing claim — ⚠) |

Mambu is the canonical **composable-platform** vendor (Section 1.2): it sells the replaceable parts *and* the assembly, which makes it a one-vendor coreless for many digital banks — at the price of being less "coreless" than the BIAN ideal of multi-vendor assembly.

### 9.3 10x Banking

**Verified facts** (10xbanking.com; FinTech Futures; Sky News; Westpac media release):

| Fact | Detail |
|---|---|
| **Founded** | 2016, London, by **Antony Jenkins** — former Group CEO of Barclays (2012–2015) |
| **Product** | **10x SuperCore** — cloud-native, event-driven, real-time core banking platform (Microsoft Azure Marketplace listing) |
| **Positioning** | Replaces legacy core infrastructure "without disrupting live operations" (10xbanking.com) |
| **Vendor claims** | Processes "over 10,000 transactions per second", "99.99% uptime" (10xbanking.com — vendor claims) |
| **Clients (reported)** | **Westpac** (May 2022 media release: institutional banking platform on 10x's "contemporary and cloud-native core technology"), **Chase UK** (JPMorgan), Old Mutual |
| **Funding** | £40M raised from AshGrove (reported by Sky News, 2026; ~$54M per TechTimes, August 2026) |

10x is the "challenger core" — a modern single core rather than a coreless architecture, which is exactly the point: the line between "coreless" and "modern core" is blurry in the vendor market, and 10x sits firmly on the modern-core side while serving as a component in coreless estates.

### 9.4 Other Composable and Platform Vendors

- **Finxact** — cloud-native, event-driven core platform for large financial institutions, fintech, and embedded finance; a founding supporter of BIAN's Coreless Bank initiative (2019); now part of Fiserv (finxact.com: "Explore the combined power of Finxact X Fiserv"); clients include First Horizon and Live Oak Bank. (Acquisition year widely reported as 2021 — ⚠ not re-verified to a primary source here.)
- **nCino** — founded **2011** in **Wilmington, North Carolina** by a team of bankers and entrepreneurs (Pierre Naudé, Chip Mahan; an outgrowth of Live Oak Bank); cloud banking platform for commercial lending, onboarding, and portfolio management; IPO on Nasdaq (NCNO) in 2020; used by 2,700+ financial institutions (2025). nCino is a *platform around* cores rather than a core replacement — the front-office/commercial-lending layer of a coreless estate.
- **Oracle** — **Oracle Banking Cloud Services**: "componentized and composable services that deliver preintegrated, SaaS-based, cloud native versions of Oracle's industry-leading banking solutions on Oracle Cloud Infrastructure" (oracle.com, verified); Oracle FLEXCUBE remains the universal banking core. The microservices architecture is covered in [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md) and the data model in [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md).
- **Technisys** — a founding BIAN coreless pilot member (2019); cloud-native digital banking platform. ⚠ Further facts (including its 2022 acquisition by SoFi, widely reported) were not verified to primary sources in this session.
- **Temenos** — the largest dedicated core-banking vendor by reported deployment count: "Over 950 banks around the world rely on Temenos Core" (temenos.com); all products "delivered on the most advanced cloud-native, cloud-agnostic, AI and API-first platform and deployable on-premise, in the cloud and as SaaS"; named a Leader in the Gartner Magic Quadrant for Retail Core Banking Systems, Europe (Temenos press release, August 2026). Temenos also maintains a "Composable Banking" product page (temenos.com/products/composable-banking/) — the page exists, but its substance is an image-led landing page (⚠ vendor positioning, not a verified architectural commitment).
- **Finastra** — formed **2017** by the combination of **Misys** (London) and **D+H** (Canada) under Vista Equity Partners; HQ London; broadest-portfolio claim across retail banking, transaction banking, lending, and treasury & capital markets; its **Fusion** product family (including the Fusion Essence core) and "open banking platform" are its cloud-era bets (finastra.com).

### 9.5 Legacy Vendors' Cloud-Core Offerings

The legacy vendors' answer to coreless is **core-as-a-service**: run the same core, vendor-operated, API-exposed, on cloud. Verified examples:

- **TCS BaNCS Cloud** — "a SaaS based industry cloud offering" with "more than 65 installations globally" (tcs.com); an API-first platform whose open-banking capabilities target Banking-as-a-Service and embedded finance; per TCS's own description (via Celent's 29 May 2024 webinar), "the world's largest set of components for banking, available as 22 strong business solutions powering banks in 16 countries from our five global and regional cloud command centers".
- **Temenos Banking Cloud / Temenos Core as SaaS** — the same Temenos Transact functionality delivered as SaaS on cloud (temenos.com; the EQ Bank and BIL customer stories on the Temenos site are SaaS deployment testimonials).
- **Oracle Banking Cloud Services** — Section 9.4; FLEXCUBE and the Oracle Banking suite as SaaS on OCI.
- **Finastra FusionCloud** — the Finastra portfolio delivered as managed cloud services (finastra.com; ⚠ specific FusionCloud facts not re-verified this session).

The platform and infrastructure dimensions of these offerings (IBM's banking cloud, hyperscaler partnerships, managed services) are covered in [ibm_cloud_guide.md](../technology/ibm_cloud_guide.md) and [financial_management_systems_guide.md](../technology/financial_management_systems_guide.md) — this guide does not re-derive them. Celent's "Core Banking in the Cloud" webinar series (Section 10.3) is the analyst-side documentation of this entire tier, explicitly framed as "intentionally non-evaluative" interviews with incumbents and challengers alike.

**Vendor landscape summary:**

| Vendor | Founded | HQ | Core offering | Coreless posture | Verified highlights |
|---|---|---|---|---|---|
| Thought Machine | 2014 | London | Vault Core (+ Vault Payments) | BIAN coreless participant; single-ledger modern core | $2.7B valuation (2022); Gartner MQ Leader 2025; JPMorgan, Lloyds, SCB clients |
| Mambu | 2011 | Amsterdam | Composable banking SaaS (lending, deposits, payments) | Markets "composable core banking" | $5.3B valuation (2021); Trust Bank SG; N26 reported |
| 10x Banking | 2016 | London | 10x SuperCore cloud-native core | Modern core for coreless estates | Antony Jenkins; Westpac, Chase UK; £40M (2026) |
| Finxact | 2015 ⚠ | Jacksonville, FL | Cloud-native event-driven core | BIAN founding pilot member | Now part of Fiserv; First Horizon, Live Oak |
| nCino | 2011 | Wilmington, NC | Cloud banking platform (lending/onboarding) | Platform around cores | IPO 2020 (NCNO); 2,700+ FIs |
| Oracle | — | — | FLEXCUBE; Banking Cloud Services | "Componentized and composable" SaaS | Oracle Banking Cloud Services verified |
| Temenos | 1993 ⚠ | Geneva | Temenos Core/Transact | Composable page ⚠; SaaS cloud core | 950+ banks; Gartner MQ Leader Europe 2026 |
| Finastra | 2017 | London | Fusion (Essence core) | Open platform; FusionCloud | Misys + D+H merger |
| TCS BaNCS | — | Mumbai | BaNCS suite; BaNCS Cloud | CaaS — "industry cloud" SaaS | 65+ cloud installs; Celent webinar |

## 10. The Debate: The Coreless Reality Check

### 10.1 The Case For

The verified pro-coreless arguments, in the words of their sources:

- **Modernization speed.** BIAN's Steve Van Wyk at the 2019 launch: legacy cores make it "incredibly difficult and costly to innovate at the speeds required to fulfil customers' needs. Migrating away from legacy technology is a big challenge." The BIAN initiative exists to make core renewal "faster, more cost-effective" and "plug and play".
- **Best-of-breed without integration hell.** BIAN Coreless 2.0 (2021): the platform "will empower banks to select the software vendors needed to obtain the best-of-breed for each application area without worrying about interoperability", via translation of proprietary message models into BIAN's standard model.
- **Agility, cost, innovation, compliance.** ThoughtWorks's "Kill your core" (2024) lists exactly these four benefits, adding the sharpest pro argument: "Commodity core ledgers can be chosen for their proven track records in meeting regulatory requirements" — i.e., the ledger is *more* auditable as a dedicated service than as a buried module.
- **The digital-bank proof point.** SEB's UNQUO (2019, Vault), Chase UK (10x), Trust Bank Singapore (Mambu), and the Singapore digital-bank cohort (see [gxs_bank_guide.md](gxs_bank_guide.md), [maribank_guide.md](maribank_guide.md)) demonstrate months-to-live timelines that monolithic-core programmes cannot match.
- **Analyst support for modularity.** McKinsey's "Modernizing core technology, without breaking the bank" explicitly endorses removing core customization and "assembling best-of-class capabilities outside of their core platform".

### 10.2 The Case Against

The verified counter-arguments:

- **"Coreless" is a misnomer and the ledger is still a core** (Section 1.3). ThoughtWorks's own definition concedes the point. Every coreless estate needs a ledger that enforces DR = CR, atomicity, idempotency, no-delete, and audit (Section 5.1) — the posting guide's §8 invariants — and that ledger is a core by any functional test.
- **Complexity moves, it does not vanish.** One atomic transaction in a monolith becomes a saga across services; the failure modes (partial postings, duplicated events, drift between engines and ledger) are new operational burdens (Sections 8.2–8.3).
- **Integration is the new vendor lock-in.** Replacing one monolith with a BIAN-compliant estate replaces a single vendor contract with N contracts plus an integration layer *you* own. The middleware burden is the subject of [enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md); the coreless-specific version is that the bank, not the vendor, now guarantees the contracts between components.
- **Reporting and compliance do not decompose.** Regulatory reporting (BCBS 239, MAS), the GL close, and audit demand a single reconciled view of the bank; every decomposition must be re-proven daily (Sections 5.3, 8.3) — a permanent cost the monolith paid implicitly.
- **Track record is thin for big incumbents.** The marquee coreless-adjacent wins (Vault at JPMorgan/Lloyds, 10x at Westpac) are *coexistence* deployments, not full core replacement — the vendors' own deployment models say so (Thought Machine's Coexistence model; 10x's "without disrupting live operations"). A full big-bang coreless migration at a tier-1 bank has no verified precedent (Section 13).
- **The cautionary tales are about exactly this risk.** TSB and RBS (Section 10.5) show what happens when a core migration or its batch machinery fails at scale.

### 10.3 Analyst Positions

Verified analyst and research-house positions:

- **Celent** — runs the "Core Banking in the Cloud" webinar series (extended from its "Cloud Kings" series) documenting how core providers support public-cloud deployment; interviews are "intentionally non-evaluative". Episode verified: "TCS BaNCS Cloud, Core Banking in the Cloud" (29 May 2024, principal analyst Daniel Mayo).
- **McKinsey** — "Modernizing core technology, without breaking the bank" (Banking Matters): remove complex customization from the current core, assemble best-of-class capabilities outside it; "Core banking modernization with AI-enabled engines" (TechForward) frames cloud-native core modernization as the route to intelligent, real-time financial engines.
- **Gartner** — composable business concept (October 2020): modularity, autonomy, orchestration, discovery; Magic Quadrant for Retail Core Banking recognizes both modern cores (Thought Machine named a Leader, 2025) and incumbents (Temenos a Leader in Europe, 2026). ⚠ No Gartner report titled "coreless" was verified (Section 13).
- **Deloitte** — "Modernizing legacy systems in banking": the IT model "shifts from in-house, on-premises banking legacy systems to modern cloud-based systems provided by third-party vendors".
- **Federal Reserve Bank of Kansas City** — "Core Banking Systems and Options for Modernization": neutral documentation that many institutions run legacy cores "up to 40 years old" on mainframes — the problem statement the coreless vendors answer.
- ⚠ **Accenture, Forrester, Capgemini** — no report titles or positions on "coreless" specifically were verified during research for this guide (Section 13).

The analyst consensus, as far as it is verifiable: *the direction (decompose, cloud, API) is accepted; the destination (fully coreless) is not yet demonstrated at scale; the migration path matters more than the endpoint.*

### 10.4 Hybrid Models: Headless Core, Core-Plus, and Pragmatic Paths

In practice, almost nobody goes straight to the BIAN ideal. The verified hybrid models:

- **Coexistence / multi-core** — the ThoughtWorks-endorsed pattern: "universal, thin, next-gen ledger can interoperate with a specialist core banking system providing support for some specific products (cards or asset finance)". The modern core runs alongside the legacy core; product domains migrate one at a time. This is also Thought Machine's official "Coexistence" deployment model.
- **Core-plus** — McKinsey's framing: keep the core, strip its customizations, and assemble the differentiating capabilities (origination, pricing, customer journeys) outside it as services. This is "coreless at the edges".
- **Headless core** — using a core's engine capabilities (posting, balances) purely through APIs while replacing its channel/UX layer with modern front-ends (⚠ the term circulates in trade press and vendor materials, but this guide could not verify a canonical primary-source definition).
- **Greenfield coreless** — digital-bank subsidiaries built coreless from day one (the Singapore digital banks, SEB's UNQUO), which avoids the migration problem entirely and is where the coreless model is closest to fully proven.
- **CaaS as a staging step** — run the existing core as a managed cloud service (TCS BaNCS Cloud, Oracle Banking Cloud Services, Temenos SaaS) first, then decompose domain by domain; the cloud-core tier of Section 1.2 as a *phase* rather than a destination.

The architect's rule of thumb, consistent with every verified source: **the endpoint (fully coreless) is a multi-year target; the path is a strangler, and the first services to migrate are the ones with the highest change cadence and the weakest coupling to the ledger** (payments, offers, onboarding — exactly the BIAN 1.0 pilot's choices).

### 10.5 Failure Modes and Cautionary Tales

The cautionary tales are not about "coreless" per se — no large bank has yet failed *at* coreless — they are about **core replacement and core batch failure at scale**, which is the same risk envelope. Verified cases:

- **TSB (UK), April 2018** — migrated customers from the Lloyds Banking Group platform (which had hosted TSB) to Sabadell's Proteo4 platform in a single weekend. Outcome, per BBC reporting: up to **1.9 million customers** locked out of online banking, some for weeks; costs of **£176.4M** pushed TSB into a half-year loss; the FCA/PRA fined TSB **£49M** (December 2022) over the "disastrous IT change". Lessons: big-bang cutover risk, test-data mismatches, and the regulator's willingness to fine operational failures — all directly applicable to a coreless cutover.
- **RBS/NatWest/Ulster Bank, June 2012** — a batch-processing software failure left **6.5 million customers** unable to access accounts for days; regulators fined RBS **£56M** (November 2014: £42M from the FCA and £14M from the PRA per FT/BBC/Guardian reporting). Lessons: the nightly batch is a single point of failure; a distributed coreless EOD (Section 8) must be *more* robust than the batch it replaces, not merely different.
- **Trade-press-only reports (⚠)** — trade press has reported multiple large banks pausing, scaling back, or abandoning core replacement programmes (and the related "banks are quietly extending mainframe life" narrative); none of these specific cases could be verified to primary sources during this guide's research window, so they are listed in Section 13 rather than cited as facts.

**The failure-mode checklist** every coreless programme must design against, drawn from the above and from Section 8: big-bang cutover with no rollback path; parallel-run drift (the new ledger and the old core disagree); data migration errors (balance/interest history); integration spaghetti (N point-to-point adapters instead of BIAN contracts); loss of the daily close (GL never balances); reconciliation gaps at service boundaries; and regulatory reporting breakage during the migration window.

---

## 11. Worked Example: Cymbal Bank Goes Coreless

**Cymbal Bank** is the fictional Singapore-based retail bank used as the worked-example persona across this repository's banking guides — the posting-engine sibling guide [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) uses the same persona for its fund-transfer worked example (S$ accounts, FAST rails, GL codes). This section applies the reference architecture of Sections 4-8 to a concrete Cymbal Bank programme: decompose a legacy monolithic core, stand up the coreless service estate, and migrate without a TSB-style cutover (Section 10.5). All product names, figures, and timelines below are illustrative design choices for the exercise, not vendor commitments.

### 11.1 The Starting State: A Legacy Core at Cymbal Bank

Cymbal Bank runs "Cymbal Core", an in-house mainframe core whose oldest COBOL modules date to the 1980s — squarely inside the "up to 40 years old" legacy population the Federal Reserve Bank of Kansas City documents (Section 3.1). Like most incumbents it is not one system but a constellation (Section 3.4):

| Legacy component | What it does today | Known pain |
|---|---|---|
| **Deposits module** (Cymbal Core) | DDA, savings, term deposits; interest accrual; statements | Product launches take 9-12 months; every release re-tests the whole core |
| **Loans module** (Cymbal Core) | Origination, schedule, arrears, disbursement postings | Heaviest customization; accounting rules embedded in COBOL |
| **GL module** (Cymbal Core) | Chart of accounts, daily close, trial balance | Runs inside the nightly batch; close completes 02:00-04:00 |
| **Batch/EOD pipeline** (Cymbal Core) | Nightly accrual, fee, maturity, and report runs | Single point of failure (the RBS 2012 lesson, Section 10.5) |
| **Card system** (separate specialist ledger) | Visa/Mastercard authorizations, settlement | Separate book; reconciled to the GL by spreadsheet |
| **Payments hub** | FAST/PayNow/MEPS+ connections (see [payments_hub_guide.md](payments_hub_guide.md)) | File-based; no real-time balance API for channels |

The trigger for the programme: open-banking obligations under MAS API guidance ([mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)) and two digital-bank competitors that went from license to live in months on Mambu-class platforms (Section 3.3), while Cymbal's last deposit-product launch took eleven months. The board approved a coreless programme in the shape of this section: **strangler, not big-bang** (Section 11.5).

### 11.2 The Target Service Decomposition

The target estate decomposes by *capability*, not by product (Section 4.1), so that products are assembled from shared services rather than owning duplicate ledgers:

| Legacy core module | Coreless replacement service | Notes |
|---|---|---|
| Deposits module | **Deposits product engine** | Product parameters, rates, terms; interest mechanics per [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md); holds product state, not money (Section 4.2) |
| Loans module | **Loans product engine** | Contract lifecycle: origination, schedule, arrears; embeds the accounting contract for each cash event (Section 6.2) |
| GL module | **Ledger-as-a-service (LaaS)** | The unified posting engine, balances, and GL integration (Section 5) — the "coreless core" |
| Card system | **Card authorizations service** | Specialist ledger retained and wrapped in BIAN-standard APIs (authorization, settlement, statement) — hub-and-spoke (Section 5.2) |
| Payments hub | **Payments service** | FAST/PayNow/MEPS+ rails behind BIAN-standard APIs; never touches balances directly (Section 6.1) |
| Batch/EOD pipeline | **Workflow/orchestration service** | Process state, maker-checker, sagas, time-based triggers (Section 6.4) |
| Customer master | **Customer & KYC service** | Party, consent, relationship hierarchy — owner of the customer identifier (Section 7.1) |
| Reporting extracts | **Data/analytics layer + regulatory reporting** | Event stream → lake → reporting; BCBS 239 lineage (Section 7.1) |

The target architecture in one diagram (each arrow is a contract the bank owns):

```
   CHANNELS (mobile, web, API partners, branch)
        |
        | APIs
        v
   API GATEWAY / BFF
        |
   +----+----------------+----------------+----------------+
   |                     |                |                |
   v                     v                v                v
CUSTOMER & KYC      DEPOSITS          LOANS           CARD
(party, consent)   PRODUCT ENGINE   PRODUCT ENGINE  AUTHORIZATIONS
   |                     |                |        (specialist ledger,
   |                     +-------+--------+         hub-and-spoke)
   |                             |
   |              posting instructions (API)
   v                             v
   +------------> LEDGER-AS-A-SERVICE
                  (unified posting engine,
                   balances, GL, idempotency keys)
                          |
                          | events (outbox)
                          v
              EVENT BACKBONE ----> DATA LAKE ----> ANALYTICS /
                                                    REGULATORY REPORTING
                          |
                          v
              WORKFLOW / ORCHESTRATION
              (EOD DAGs, sagas, maker-checker)
```

### 11.3 The Ledger Strategy: One Unified Ledger, One Specialist

Cymbal Bank's products share a single customer money position — a current account, a savings account, and a card draw on the same balance — which triggers the single-ledger branch of the Section 5.2 decision rule. The design:

- **One unified ledger-as-a-service** holds all retail customer money positions (a Vault-Core-class or Modern-Treasury-class engine; Section 5.4). Product engines are stateless calculators on top: they *decide*, the ledger *records* (Section 4.2).
- **Double-entry mechanics are not re-derived here.** The ledger must preserve the posting engine's invariants — atomicity, idempotency, no-delete, audit trail, DR = CR — exactly as specified in posting guide §§8.2-8.4, and execute balanced **posting instructions** with idempotency keys (posting guide §7.4; Section 5.1 of this guide). Balance components (available vs ledger vs cleared), holds, and value dating remain ledger-service semantics (posting guide §5.1).
- **One specialist ledger survives: cards.** The card system keeps its own book (authorizations settle on the card cycle) and interoperates with the unified ledger in ThoughtWorks's hub-and-spoke pattern (Section 5.2): card settlement posts into the unified ledger through the standard posting-instruction path, with explicit **inter-ledger clearing accounts** and a daily reconciliation job proving them (Sections 5.3, 8.3).
- **The chart of accounts is a versioned shared contract** (Section 5.3): the deposits engine, the loans engine, and the card wrapper all reference the same COA contract, so the ledger never receives an unmapped GL account.

The single-ledger choice costs Cymbal domain isolation but buys the 360° customer view and the simplest possible reconciliation — the strongest architectural argument for Vault-style unification (Section 5.2), and the right trade for a retail bank whose books all feed one customer balance.

### 11.4 The API/Integration Pattern

The integration architecture follows Section 7.2 and the general middleware discipline of [enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md) — this guide does not re-derive it:

- **API gateway for channels, event backbone between services.** Synchronous APIs for reads and commands (balance enquiry, account open, payment initiation); events for facts ("repayment happened", "limit breached"). Mixing the two — publishing events for commands, or calling APIs to discover facts — is the classic failure (Section 7.2).
- **Outbox → stream → lake.** Every service writes its state change and an event atomically to its outbox; a relay publishes to the event backbone; analytics and reporting consume from the lake, never from live service databases (Section 7.1).
- **BIAN message-model translation as a first-class component.** Each vendor's proprietary message model is translated into BIAN's standard model (Coreless 2.0, Section 1), so a vendor swap touches one contract instead of N point-to-point adapters (Section 7.2).
- **Files at the edge.** SWIFT, FAST batch, and MAS regulatory submissions remain file-based; the integration layer terminates files at the edge and converts to/from the internal event/API model (Section 7.2; [swift_alliance_access_guide.md](swift_alliance_access_guide.md) for the file-rails reality).

### 11.5 The Migration Plan: Strangler, Parallel Run, Cutover, Rollback

The programme is an incremental strangulation (Section 3.4), with the TSB and RBS failure modes of Section 10.5 as the explicit design-against list:

| Phase | Scope | Entry/exit criteria |
|---|---|---|
| **0. Contracts first** | Chart-of-accounts contract, canonical event schemas, BIAN message model | Exit: contracts versioned and reviewed before any implementation |
| **1. Payments & onboarding** (the BIAN 1.0 pilot choices, Section 1) | Payments service, Customer & KYC, account opening on the new estate | Highest change cadence, weakest coupling to the ledger; first visible win |
| **2. Deposits read/write split** | Balance reads move to the LaaS; writes follow in phase 3 | New ledger and Cymbal Core run in **parallel run**, reconciled daily (below) |
| **3. Deposits product engine + ledger cutover** | All DDA/savings/term products post to the unified ledger | Cutover criteria met: 30 consecutive days of zero unexplained reconciliation breaks |
| **4. Loans** | Loans product engine live; disbursements and repayments post to the unified ledger | Arrears and accrual accounting re-proven against the old GL |
| **5. Cards wrapped, GL close moved** | Card wrapper live; daily close and regulatory pack run on the new estate | Exit: Cymbal Core's deposits, loans, and GL modules decommissioned |

The operational rules that make it survivable:

- **Parallel run with a daily tie-out.** During phases 2-4 both the old core and the new estate post; a daily reconciliation job (Section 8.3) proves old-core balances equal new-ledger balances plus the card clearing account, within agreed tolerances. Parallel-run *drift* — the new ledger and the old core disagree — is the #1 failure mode to design against (Section 10.5).
- **Cutover is a flag, not a migration.** Each product domain flips a per-product "post to new ledger" switch after its reconciliation runway; the old core stays hot as the rollback target.
- **Rollback is rehearsed.** The trigger list is explicit: the GL never balances at close; reconciliation breaks beyond tolerance for two consecutive days; regulatory pack generation misses its SLA. Rollback means flipping the flag back and letting the next day's batch run on the old core — possible only because the strangler never deletes the legacy path until the final phase's exit criteria are met.
- **Data migration is minimized by design.** Because products keep their account numbers and the new ledger is rebuilt from the event stream during parallel run, Cymbal migrates *state* (balances, interest history) rather than *programs* — the balance/interest-history migration errors of Section 10.5 are the exact trap this avoids.

### 11.6 EOD/Batch Design in the Target State

The nightly batch does not die; it becomes an orchestrated DAG (Section 8). The scheduling, dependency-graph, and idempotent-rerun themes are the general subject of [apache_airflow_guide.md](../technology/apache_airflow_guide.md); Cymbal's instantiation:

- **Real-time posting shrinks the batch.** Accruals, fees, and maturities post as events during the day (Section 8.2), so the 22:00-01:00 window is *closing and proving*, not computing: product-engine sweeps → GL close job against the ledger → reconciliation jobs (ledger ↔ GL, ledger ↔ engines, ledger ↔ cards, ledger ↔ payments; Section 8.3) → regulatory pack generation → SLA report.
- **Every job is idempotent and re-runnable** on the ledger's idempotency keys (Section 5.1) — a partial rerun can never double-post.
- **Ordering is an explicit, versioned dependency graph** per book. Singapore's single time zone keeps Cymbal's close simple; the multi-book rolling-close problem of Section 8.2 is a future multi-country concern, not today's.
- **MAS resilience obligations apply to the orchestration layer.** MAS notice 644 (Section 1.3) covers the EOD DAG and its reconciliation jobs as much as the ledger: they are part of the bank's critical systems inventory, with recovery-time objectives and tested reruns.

### 11.7 What Cymbal Bank Keeps vs What It Retires

| Keeps (re-hosted or wrapped) | Retires (decommissioned) |
|---|---|
| The ledger and its posting invariants — as LaaS, the "coreless core" (Sections 1.3, 5) | Cymbal Core's deposits module (product logic, parameter tables) |
| The card specialist ledger — wrapped, hub-and-spoke | Cymbal Core's loans module (COBOL accounting rules) |
| The payments rails (FAST/PayNow/MEPS+) — behind BIAN-standard APIs | The monolithic GL module and its in-core batch close |
| The chart of accounts — as a versioned contract | The nightly batch program and its implicit ordering |
| Customer data, KYC, and audit history — in the Customer & KYC service | Screen-scraping channel adapters (Section 3.2) |
| Regulatory reporting lineage (BCBS 239) — now event-derived | The single-schema, single-database assumption |

The endpoint is deliberately not "no core": Cymbal Bank still has a ledger, a GL close, and a daily prove-out — but they are services and jobs in an estate the bank owns by contract, not modules inside a vendor's monolith. Section 11.3's unified ledger is the irreducible remainder: the coreless core of Cymbal Bank's new architecture.

---

## 12. Claims Audit

This table audits the guide's key claims against their sources, using the same ✅/⚠/❌ discipline applied throughout: ✅ = verified at a primary or directly cited source (vendor site, standards body, regulator, major outlet, or sibling guide); ⚠ = vendor marketing claim or unverified during this guide's research window; ❌ = disputed by other verified sources in this guide. Everything marked ⚠ is listed in Section 13.

| Claim | Status | Source / note |
|---|---|---|
| Thought Machine founded 2014 in London by Paul Taylor (ex-Google) | ✅ | Wikipedia; thoughtmachine.net; Section 9.1 |
| Vault Core keeps data unified on one ledger with product logic as smart contracts | ✅ | thoughtmachine.net/vault-core; Section 9.1 |
| Thought Machine named a Leader in the 2025 Gartner Magic Quadrant for Retail Core Banking | ✅ | Thought Machine press release; Section 9.1 |
| Thought Machine funding: $200M Series C (Nov 2021) → $1B valuation; $160M Series D (2022) → $2.7B | ✅ | Press reporting; Section 9.1 |
| Thought Machine joined BIAN in August 2020; on the Coreless 2.0 development team | ✅ | thoughtmachine.net; BIAN; Sections 2.1, 9.1 |
| Mambu founded 2011 (Berlin); HQ Amsterdam (Mambu GmbH, German-Dutch) | ✅ | Wikipedia; mambu.com; Section 9.2 |
| Mambu valued at $5.3B (December 2021) | ✅ | WSJ; Section 9.2 |
| Mambu "pioneered composable core banking"; "up to 50% lower TCO than legacy cores" | ⚠ | Vendor claims on mambu.com; TCO figure is marketing, not an audited benchmark (Sections 2.2, 3.1) |
| Trust Bank Singapore runs on Mambu | ✅ | trust_bank_guide.md; Section 9.2 |
| 10x Banking founded 2016 by Antony Jenkins (ex-Barclays Group CEO) | ✅ | 10xbanking.com; FinTech Futures; Section 9.3 |
| 10x SuperCore processes "over 10,000 TPS" at "99.99% uptime" | ⚠ | Vendor claims, 10xbanking.com (Sections 3.1, 9.3) |
| 10x raised £40M from AshGrove (2026; ~$54M per TechTimes) | ✅ | Sky News; TechTimes; Section 9.3 |
| BIAN launched its "Coreless Bank" initiative on 23 September 2019 with eleven members | ✅ | BIAN announcement; Sections 1, 2.1 |
| BIAN Coreless Banking 2.0 (12 October 2021) added message-model translation | ✅ | BIAN; Sections 1, 2.1 |
| ThoughtWorks "Kill your core" (November 2024) popularized the engineering definition | ✅ | thoughtworks.com; Sections 1, 2.1 |
| Gartner launched composable business in October 2020 (modularity, autonomy, orchestration, discovery) | ✅ | Gartner press release, 19 October 2020; Section 2.2 |
| TSB's April 2018 migration locked out up to 1.9M customers; cost £176.4M; fined £49M by FCA/PRA (December 2022) | ✅ | BBC; FCA/PRA; Section 10.5 |
| RBS June 2012 batch failure affected 6.5M customers; £56M in fines (November 2014) | ✅ | FT/BBC/Guardian; Section 10.5 |
| "Coreless banking means a bank has no core system" | ❌ | Disputed: ThoughtWorks's own definition says it "doesn't mean without a core banking system" (Sections 1, 1.3) |
| "The batch dies in a coreless architecture" | ❌ | Disputed: batch becomes orchestrated distributed jobs with an explicit dependency graph (Section 8.1) |
| Paul Taylor coined the term "coreless banking" | ⚠ | No primary source found; BIAN formalized the term in 2019 (Section 2.1) |
| "There is no such thing as a coreless bank" is attributable to one primary source | ⚠ | Quote circulates in analyst/trade commentary; attribution not pinned (Section 1.3) |
| Analyst reports titled "coreless" exist (Gartner, Accenture, Forrester, Capgemini) | ⚠ | None verified during research (Sections 2.2, 10.3) |
| A full big-bang coreless migration at a tier-1 bank has a verified precedent | ⚠ | None found; marquee wins are coexistence deployments (Section 10.2) |
| Headless core has a canonical primary-source definition | ⚠ | Term circulates in trade press/vendor materials only (Section 10.4) |
| Finxact founded 2015; acquired by Fiserv in 2021 | ⚠ | Widely reported; not re-verified to primary sources this session (Section 9.4) |
| Technisys acquired by SoFi (2022) | ⚠ | Widely reported; not verified to primary sources (Section 9.4) |
| Temenos: "Over 950 banks around the world rely on Temenos Core" | ✅ | temenos.com (vendor figure, verified as stated); Section 9.4 |
| TCS BaNCS Cloud has "more than 65 installations globally" | ✅ | tcs.com; Section 9.5 |
| Many institutions still run legacy cores "up to 40 years old" on mainframes | ✅ | Federal Reserve Bank of Kansas City research briefing; Section 3.1 |
| Trade-press reports of large banks pausing or abandoning core replacement programmes | ⚠ | Trade press only; specific cases unverified and listed in Section 13 (Section 10.5) |

---

## 13. What Could Not Be Verified

This section collects every ⚠ item flagged in Sections 1-10 plus anything else the research window could not confirm against a primary source. None of these are asserted as facts in this guide; they are listed so the reader can treat them as open questions.

1. **First use of "coreless" in a banking context before BIAN (2019).** The exact origin is not documented in any primary source this guide could verify (Section 2.1).
2. **Paul Taylor coining "coreless banking."** No primary source found in which Thought Machine's founder claims the term; the verified attribution is BIAN's 2019 formalization with Thought Machine and ThoughtWorks as popularizers (Section 2.1).
3. **The quote "there is no such thing as a coreless bank."** It circulates in analyst and trade commentary as shorthand for the Section 1.3 critique; attribution to a single primary source could not be pinned (Sections 1.3, 12).
4. **Analyst reports titled "coreless."** No Gartner, Accenture, Forrester, or Capgemini report title using "coreless" was verified (Sections 2.2, 10.3).
5. **Gartner "coreless" positioning.** The verified Gartner facts are the composable-business concept (October 2020) and the Retail Core Banking Magic Quadrant placements (Thought Machine a Leader 2025; Temenos a Leader in Europe 2026); no Gartner report on "coreless banking" per se was found (Section 10.3).
6. **Mambu's "up to 50% lower TCO than legacy cores."** Vendor marketing claim; no independent benchmark verified (Sections 3.1, 9.2, 12).
7. **10x's "over 10,000 transactions per second" and "99.99% uptime."** Vendor claims; no audited benchmark verified (Sections 3.1, 9.3, 12).
8. **Finxact founding year (2015) and acquisition by Fiserv (2021).** Widely reported; not re-verified to a primary source this session (Section 9.4).
9. **Technisys's acquisition by SoFi (2022).** Widely reported; not verified to primary sources (Section 9.4).
10. **Temenos's "Composable Banking" product page substance.** The page exists but is an image-led landing page — vendor positioning, not a verified architectural commitment (Section 9.4).
11. **Finastra FusionCloud specifics.** Not re-verified this session (Section 9.5).
12. **Headless core definition.** The term circulates in trade press and vendor materials; no canonical primary-source definition verified (Section 10.4).
13. **A full big-bang coreless migration at a tier-1 bank.** No verified precedent; the marquee coreless-adjacent wins (Vault at JPMorgan/Lloyds, 10x at Westpac) are coexistence deployments, per the vendors' own deployment models (Section 10.2).
14. **Trade-press-only core-replacement-pause reports (promised in Section 10.5).** Trade press has reported multiple large banks pausing, scaling back, or abandoning core replacement programmes, plus the related "banks are quietly extending mainframe life" narrative. None of the specific cases could be verified to primary sources (vendor announcements, regulator filings, or the banks themselves) during this guide's research window — so, as Section 10 states, they are listed here rather than cited as facts.
15. **Vendor TCO/throughput figures generally.** All "lower TCO", "transactions per second", and "uptime" figures quoted in Sections 3 and 9 are marketing claims unless explicitly marked otherwise.

The rule applied throughout: a claim verified at a primary source is a fact (✅); a vendor figure is a claim (⚠); a contested framing is marked ❌. Section 12 tabulates all three.

---

## 14. Glossary

Terms as used in this guide:

- **Coreless banking** — decomposition of the monolithic core into a network of API-accessible services; the term BIAN formalized in 2019. Does not mean "no core": the ledger remains (Sections 1, 1.3).
- **Coreless core** — this guide's term for the irreducible remainder of any decomposition: the ledger and its posting invariants (Sections 1.3, 5).
- **Composable banking** — the vendor-side label for selling components (deposit engine, lending engine, ledger) that the bank assembles; Mambu's canonical positioning (Section 1.2).
- **Core-as-a-service (CaaS)** — the core consumed as a managed, API-exposed SaaS; TCS BaNCS Cloud and Oracle Banking Cloud Services are the verified examples (Section 1.2).
- **Cloud core** — the same monolithic product re-hosted in the cloud; the bank's architecture is unchanged (Section 1.2).
- **Headless core** — using a core's engine capabilities purely through APIs while replacing its channel/UX layer; ⚠ no canonical definition verified (Section 10.4).
- **Ledger-as-a-service (LaaS)** — the posting engine delivered as an API product, preserving atomicity, idempotency, no-delete, audit trail, and DR = CR (Section 5.1).
- **Posting instruction** — a balanced set of legs submitted by a product engine for the ledger to execute atomically; the unit of work between engines and ledger (Sections 5.1, 11.3).
- **Product engine** — a service that calculates (interest, fees, limits, loan lifecycle) and hands cash movements to the ledger; holds product state, not money (Section 4.2).
- **BIAN** — Banking Industry Architecture Network; the standards body behind the Coreless Banking Initiative (2019) and its message-model translation (Coreless 2.0, 2021) (Section 1).
- **Strangler pattern** — incremental migration that grows the new estate around the old until the legacy is retired; the only serious coreless migration path (Sections 3.4, 11.5).
- **Parallel run** — old and new systems both live while a daily tie-out proves they agree; drift here is the #1 failure mode (Sections 10.5, 11.5).
- **Cutover** — the flag-flip that moves a product domain to the new estate; a flag, not a data migration, in Section 11.5's design.
- **Rollback** — returning to the legacy path via the strangler's still-hot old core; rehearsed with explicit triggers (Section 11.5).
- **EOD / batch** — end-of-day processing; in a coreless estate, orchestrated distributed jobs with an explicit dependency graph rather than one implicit program (Section 8).
- **Saga** — a sequence of local transactions with compensating actions, replacing the monolith's atomic cross-module postings (Section 4.3).
- **Outbox pattern** — writing a state change and its event atomically, then publishing via a relay; gives exactly-once, ordered, replayable events (Section 7.1).
- **Event backbone** — the stream infrastructure that carries facts between services; the only inter-service integration point (Section 7.2).
- **API gateway / BFF** — the only integration point for channels; BFF = backend-for-frontend (Sections 4.1, 7.2).
- **Idempotency key** — a client-supplied reference the ledger deduplicates on, so retries cannot double-post (Section 5.1).
- **Chart of accounts (COA)** — the versioned shared contract of GL accounts that product engines reference in posting instructions (Sections 5.3, 11.3).
- **Clearing account** — an explicit inter-ledger account that proves cross-ledger money movement; reconciled daily (Sections 5.3, 8.3).
- **Trial balance / GL close** — the daily DR = CR proof and the accounting-calendar close; orchestrated jobs against the ledger in the target state (Sections 5.3, 8.2).
- **Maker-checker** — dual-control approval; re-hosted in the workflow layer in a coreless estate (Sections 4.3, 6.4).
- **DAG** — directed acyclic graph; the Airflow-class model for the EOD dependency graph (Section 8.2).

## 15. References

URLs below are the primary or directly inferable locations of the sources cited in the prose. Vendor marketing pages are marked where the claim is a vendor claim.

**Standards and industry bodies**

- BIAN — Coreless Banking Initiative (launched 23 September 2019) and Coreless Banking 2.0 (12 October 2021): https://www.bian.org
- BIAN — Coreless Banking 4.0 description ("building blocks that can be assembled and implemented in unique ways"): https://www.bian.org

**Vendors and their claims**

- Thought Machine — BIAN membership announcement ("Shaping the future of banking IT: We've joined BIAN") and product pages: https://thoughtmachine.net ; https://thoughtmachine.net/vault-core
- Mambu — composable core banking positioning and TCO claim (vendor claims ⚠): https://www.mambu.com
- 10x Banking — SuperCore, uptime/TPS claims (vendor claims ⚠), Westpac media release (May 2022): https://10xbanking.com
- Finxact — cloud-native event-driven core, now part of Fiserv: https://www.finxact.com
- nCino — cloud banking platform: https://www.ncino.com
- Oracle — Oracle Banking Cloud Services ("componentized and composable services"): https://www.oracle.com
- Temenos — Temenos Core/Transact and Composable Banking page (⚠ image-led landing page): https://www.temenos.com ; https://www.temenos.com/products/composable-banking/
- Finastra — Fusion product family and FusionCloud: https://www.finastra.com
- TCS — BaNCS Cloud ("more than 65 installations globally"): https://www.tcs.com
- ThoughtWorks — "Kill your core" (November 2024): https://www.thoughtworks.com/insights/blog/kill-your-core

**Analysts and research houses**

- Gartner — "Gartner Says Organizations Should Strive for Composability to Be Resilient and Agile During Uncertainty" (19 October 2020); Magic Quadrant for Retail Core Banking (2025/2026 Leader placements per vendor press releases): https://www.gartner.com
- Celent — "Core Banking in the Cloud" webinar series, including "TCS BaNCS Cloud, Core Banking in the Cloud" (29 May 2024, principal analyst Daniel Mayo): https://www.celent.com
- McKinsey — "Modernizing core technology, without breaking the bank" (Banking Matters): https://www.mckinsey.com
- Deloitte — "Modernizing legacy systems in banking": https://www.deloitte.com
- Federal Reserve Bank of Kansas City — "Core Banking Systems and Options for Modernization": https://www.kansascityfed.org

**News and trade press**

- BBC — TSB April 2018 outage reporting (up to 1.9M customers affected; £176.4M cost): https://www.bbc.com/news
- Financial Times — RBS 2012 batch failure and 2014 fines reporting: https://www.ft.com
- The Guardian — RBS June 2012 batch failure reporting: https://www.theguardian.com
- The Wall Street Journal — Mambu $5.3B valuation (December 2021): https://www.wsj.com
- Sky News — 10x Banking £40M raise from AshGrove (2026): https://news.sky.com
- FinTech Futures — 10x Banking coverage: https://www.fintechfutures.com
- WAU.com — "Coreless Banking: When Your Core Stops Being a Single Platform": https://wau.com
- Medium — "The Critical Shift to Coreless Banking": https://medium.com

**Encyclopedic background**

- Wikipedia — Thought Machine: https://en.wikipedia.org/wiki/Thought_Machine
- Wikipedia — Mambu: https://en.wikipedia.org/wiki/Mambu

The core does not vanish; it dissolves into the platform — the coreless core.
