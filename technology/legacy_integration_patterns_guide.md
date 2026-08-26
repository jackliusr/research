# Integration Patterns Across Legacy Enterprise Systems: The Patterns Deep-Dive

*A comprehensive deep-dive on the integration problem and its solutions: the silos and the mainframes that make integration hard, the four integration styles (file transfer, shared database, RPC, messaging), the Enterprise Integration Patterns (Hohpe & Woolf, 2003) and their categories, the six key patterns, the full pattern catalogue with Apache Camel support, the legacy patterns (strangler fig, anti-corruption layer), the modernization patterns (event-driven, CQRS, outbox), the middleware landscape (ESB vs message broker vs iPaaS), and a worked example — a bank's legacy core integration.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Integration / Enterprise-Architecture (technology/)
> **Audience:** Solution architects, integration architects, platform engineers, technical leads modernizing legacy estates
> **Last Updated:** August 2026

**Cross-references:** [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (the integration **frameworks** — CDC/ETL/API tooling, 0 EIP mentions — the *tools* companion to this *patterns* guide), [camel_camelk_research.md](camel_camelk_research.md) (Apache Camel — the reference **EIP implementation**, 80+ patterns, 350+ connectors), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) (the **file-transfer** style in production — MFT, PeSIT/SFTP, checkpoint/restart), [event_stream_processing_guide.md](event_stream_processing_guide.md) (the **event-driven** modernization pattern — stream processing, fault tolerance), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (messaging **reliability** — delivery semantics, the transactional outbox in §10), [apache_seata_guide.md](apache_seata_guide.md) (distributed **transactions** and sagas — the transactionality that messaging alone cannot provide), [distributed_auth_guide.md](distributed_auth_guide.md) (identity across integrated systems — lightly), [temporal_workflow_guide.md](temporal_workflow_guide.md) (durable **orchestration** — the workflow angle on multi-step integration), [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md), [banking/tafj_guide.md](banking/tafj_guide.md), [banking/jbase_vs_infobasic_guide.md](banking/jbase_vs_infobasic_guide.md), [banking/temenos_guide.md](banking/temenos_guide.md), [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md), [banking/oracle_flexcube_data_model_guide.md](banking/oracle_flexcube_data_model_guide.md), [banking/apache_fineract_guide.md](banking/apache_fineract_guide.md) (the **legacy cores** — the systems being integrated), [banking/dbs_software_systems_guide.md](banking/dbs_software_systems_guide.md) and [banking/standard_chartered_guide.md](banking/standard_chartered_guide.md) (real bank **integration landscapes**).

---

**How to read this guide.** This is the *patterns* deep-dive of the integration series: [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) covers the *tools* (CDC/ETL/API frameworks), this guide covers the *design* (the patterns themselves). A fast path through it: read **§1** (the problem — the silos and the mainframes), **§2** (the four styles — the decision space), **§4** (the six patterns that carry most real work) and **§6** (strangler + ACL — the legacy playbook), then **§9** (the worked example that ties everything together); treat **§5** (the catalogue) as the reference table, **§3** as the theory, **§7** as the target-architecture playbook, and **§8** as the middleware buying decision. Verification status of every factual claim is in **§11**; terms are in **§12**. Cross-references point at the series' sibling guides throughout.

---

## Table of Contents

1. [The Integration Problem](#1-the-integration-problem)
2. [The Integration Styles](#2-the-integration-styles)
3. [The Enterprise Integration Patterns](#3-the-enterprise-integration-patterns)
4. [The Key Patterns](#4-the-key-patterns)
5. [The Pattern Catalogue](#5-the-pattern-catalogue)
6. [The Legacy Patterns](#6-the-legacy-patterns)
7. [The Modernization Patterns](#7-the-modernization-patterns)
8. [The Middleware](#8-the-middleware)
9. [The Worked Example: A Bank's Legacy Core Integration](#9-the-worked-example-a-banks-legacy-core-integration)
10. [The Summary: Integrate, Don't Replace](#10-the-summary-integrate-dont-replace)
11. [Verification and Claims-Status](#11-verification-and-claims-status)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## 1. The Integration Problem

### 1.1 The Silos: The Legacy Systems

Every large enterprise — and no enterprise more so than a bank — runs on a **patchwork of systems that predate its current architecture**. These are the *legacy systems*: the decades-old applications that were never designed to talk to anything else, that own the crown-jewel data (the ledger, the customer, the risk position), and that cannot be switched off because they *are* the business. In banking these are the **core banking systems**: the mainframe COBOL cores, and the packaged platforms built on them — Temenos T24, Oracle Flexcube, the jBASE/TAFJ-derived cores, open-source Fineract in digital banks (see [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md), [banking/temenos_guide.md](banking/temenos_guide.md), [banking/oracle_flexcube_data_model_guide.md](banking/oracle_flexcube_data_model_guide.md), [banking/jbase_vs_infobasic_guide.md](banking/jbase_vs_infobasic_guide.md), [banking/tafj_guide.md](banking/tafj_guide.md), [banking/apache_fineract_guide.md](banking/apache_fineract_guide.md)). The real-world landscapes — DBS and Standard Chartered — are documented in [banking/dbs_software_systems_guide.md](banking/dbs_software_systems_guide.md) and [banking/standard_chartered_guide.md](banking/standard_chartered_guide.md).

Around the core cluster the rest of the estate: payments hubs, CRM, risk engines, regulatory reporting, channels (mobile, internet, branches, ATMs). Each of these systems was built independently, by different teams, in different eras, and each **owns its own copy of the data**. This is the **silo**: a system whose data is trapped inside it, duplicated everywhere else, and reconcilable only by batch jobs and human effort.

The defining symptom of the siloed estate is **point-to-point integration gone exponential**. With *N* systems and *N(N−1)/2* potential pairs, direct connections become an unmaintainable tangle — the "spaghetti" diagram that every integration architect has been shown by a nervous CIO. Every new system (a new mobile channel, a new fraud engine) means touching a dozen old ones.

### 1.2 Why It's Hard: The Mainframes, The COBOL

Integration across legacy systems is hard for reasons that are structural, not accidental:

- **The mainframe reality.** The core runs on mainframes (z/OS class) or legacy midrange platforms, optimized for **batch processing** — the end-of-day (EOD) cycle that posts interest, generates statements, and reconciles. The machine's whole operating model is *collect, batch, post, report*. Real-time integration is alien to it. Transactions sit behind CICS/IMS-style transaction monitors with screen-oriented or record-oriented interfaces, not APIs. There is no REST endpoint on a 1985 COBOL program; there is a **copybook** and a file layout.
- **COBOL, the language that will not die.** COBOL (COmmon Business-Oriented Language, 1959) was designed for exactly what banks do: business data processing with decimal arithmetic, record structures, and report generation. It is still the language of core banking in most of the world's large banks. The code is **correct, battle-tested, and opaque** — the system of record, written in a language few engineers under 40 can read. Widely-circulated figures (e.g., "billions of lines of COBOL still in production", "the majority of banking transactions touch COBOL") are consistent with every practitioner's experience, but exact statistics are hard to pin down and are flagged as unverified in §11.
- **The skills gap.** The engineers who wrote and understand the core are retiring. The engineers who must integrate it (and who will eventually replace it) have never seen a copybook, a JCL job stream, or a VSAM dataset. This asymmetry — *high business criticality, low change capacity* — is the fundamental reason legacy integration is a discipline of its own.
- **The batch window.** The core's availability for integration work is bounded by its batch cycle. Nightly file exchanges fit the batch model; intraday events do not. Any integration design that demands more than the batch window allows must either stretch the window (risky) or find a side door (messaging, CDC — see [event_stream_processing_guide.md](event_stream_processing_guide.md)).
- **Duplicated, drifting data.** Because every silo keeps its own copy, the same customer exists in five systems with five spellings and three addresses. Integration is as much about **reconciling and canonicalizing** as about moving bytes.

The result is the classic **integration paradox**: the systems that are the most valuable to integrate (they hold the truth) are the hardest to integrate (they predate every modern interface convention). This paradox — not technology choice — is why integration patterns exist at all.

### 1.3 The Problem Table

| # | The problem | The symptom | Why it hurts |
|---|-------------|-------------|--------------|
| 1 | **Siloed systems** — each app owns its data | Same customer/position duplicated across systems; no single view | "Customer 360" is a project, not a fact; risk and revenue are invisible |
| 2 | **Batch-only integration** — EOD file drops (the file-transfer style, §2.2) | Data is yesterday's by definition | Channels can't show real-time balances; fraud and payments run blind intraday |
| 3 | **The mainframe black box** — no APIs, record-oriented interfaces | Screen scraping, file extracts, or nothing | Every new consumer needs a bespoke, fragile bridge |
| 4 | **COBOL opacity + skills gap** | Changes to the core are slow, feared, and expensive | The core's behavior is a spec written in an unreadable language |
| 5 | **Dual writes** — app updates DB and calls another system | Inconsistent state when one write fails | The root cause of a whole reliability discipline; see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) |
| 6 | **Point-to-point spaghetti** — *N(N−1)/2* direct links | A web of ungovernable, undocumented connections | Every change cascades; nobody knows what talks to what |
| 7 | **Semantic drift** — same term, five meanings | Translation errors at every boundary | Data quality decays; reconciliation breaks |

### 1.4 The Integration Imperative: Integrate, Don't Replace

The conclusion the entire industry has converged on — and the thesis of this guide — is that **you do not fix a legacy estate by replacing it, and you do not fix it by ignoring it. You fix it by integrating it**: systematically, pattern by pattern, until the legacy systems become services in a larger architecture, and eventually — gently — most of them can be retired. That is the *integrate, don't replace* philosophy, and every pattern in this guide serves it.

### 1.5 The Legacy-Estate Taxonomy

Before patterns, it helps to classify the estate, because each class of legacy system integrates differently:

| Class | What it is | Examples (this repo) | Integration profile |
|-------|------------|----------------------|---------------------|
| **The systems of record** | The cores — the oldest, most valuable, least changeable layer | Mainframe COBOL cores; T24, Flexcube, jBASE/TAFJ, Fineract ([banking/temenos_guide.md](banking/temenos_guide.md), [banking/oracle_flexcube_data_model_guide.md](banking/oracle_flexcube_data_model_guide.md), [banking/jbase_vs_infobasic_guide.md](banking/jbase_vs_infobasic_guide.md), [banking/tafj_guide.md](banking/tafj_guide.md), [banking/apache_fineract_guide.md](banking/apache_fineract_guide.md)) | Record-oriented interfaces, batch cycles, no APIs — the target of the ACL + outbox + strangler |
| **The glue systems** | The accidental integration layer built over decades | MFT fleets (Axway Transfer CFT), scheduled batch jobs (Control-M), home-grown adapters, screen scrapers | They *are* the file-transfer style (§2.2); brittle, undocumented, but already "integrated" — replace their logic with patterns, not their traffic with nothing |
| **The satellites** | Younger, semi-modern apps that grew up around the cores | Channel apps, CRM, reporting, risk engines | Modern-ish individually, entangled point-to-point collectively — the N×M problem lives here |

The **age pyramid** insight: the older the layer, the higher its business criticality and the lower its change capacity — and the more the integration architecture must *route around* it rather than *through* it. Every pattern in this guide maps onto this taxonomy: the ACL wraps the systems of record, the EIPs replace the glue, and the strangler consumes the satellites' functions from the top down.

---

## 2. The Integration Styles

### 2.1 The Four Styles (verified — Hohpe & Woolf, Chapter 1)

The canonical framing of *how* applications can exchange information comes from the opening chapter of **Enterprise Integration Patterns** (Hohpe & Woolf, 2003 — see §3.1). The book identifies exactly **four integration styles**, each a different answer to the question *"how do I get data from application A to application B?"*:

1. **File Transfer** — A writes a file; B reads it (later, in batch).
2. **Shared Database** — A and B read and write the *same* database.
3. **Remote Procedure Invocation (RPC)** — A calls a procedure/function that executes on B's machine, synchronously.
4. **Messaging** — A sends a message to a channel; B receives it asynchronously.

The styles are not competing products — they are **decision points**. Real enterprises run all four simultaneously (a bank's estate typically does: SWIFT file exchanges, a shared data warehouse, API/RPC calls to the payments hub, and a Kafka event backbone — all at once). The styles table in §2.6 is the architect's cheat-sheet for choosing.

### 2.2 File Transfer

**Mechanism:** Application A exports data to a file (CSV, fixed-width, EBCDIC records); the file is moved (FTP/SFTP/PeSIT/AS2 — or even tape); Application B imports it. The two applications never touch each other; they only touch the file. Timestamps and file names are the only coordination.

**Where it lives:** this is the **default integration style of the legacy era** — it matches the batch model perfectly (EOD extract, overnight load, morning report). In banking it is still the dominant style for interbank and regulatory traffic, industrialized as **managed file transfer (MFT)** — Axway Transfer CFT, with its PeSIT protocol, checkpoint/restart, and exactly-once transfer semantics, is the European banking standard (see [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) for the production reality, including workload-automation scheduling).

**Pros:** simple, decoupled, batch-friendly, natural fit for mainframes, auditable (a file on disk is evidence), no real-time dependency. **Cons:** data is stale by definition, no transactional guarantees inside the file, format coupling (both sides must agree the layout), error handling is "the file was wrong", and it cannot carry a request/response conversation. File transfer answers *"how do we exchange bulk data on a schedule?"* — not *"how do we integrate in real time?"*.

### 2.3 Shared Database

**Mechanism:** Multiple applications read and write the **same database**. No file, no network call — just tables.

**Pros:** real-time visibility, simple for the applications (they already know SQL), no transport layer, transactional integrity inside the DB. **Cons:** tight coupling — every application depends on the schema, and changing the schema changes every application; no encapsulation (the DB schema *is* the interface, and it leaks); locking/contention between applications; and it silently merges the applications' data models, which is exactly how the "five spellings of the customer" problem begins. A shared database is how many legacy estates *accidentally* integrate today — through the warehouse, the operational data store, or a direct schema grant that somebody granted in 1998 and nobody revoked.

### 2.4 Remote Procedure Invocation (RPC)

**Mechanism:** Application A invokes a procedure that runs on B's machine and returns a result — synchronous, request/reply. The historical stack: CORBA, DCOM, Java RMI, then web services (SOAP/WS-*) and now REST/HTTP and gRPC.

**Pros:** natural programming model (a call looks like a local function call), immediate response, great for request/reply business operations (check balance, transfer funds). **Cons:** **coupling** — the caller must know the callee's interface and availability; **synchronous fragility** — if B is down, A fails (the EIP book's own critique: RPC is the style that *feels* simple and *is* the most tightly coupled, because the caller blocks on a remote machine it cannot control); poor fit for fan-out (one event to many consumers) and for long-running work. Modern REST mitigates some of this but inherits the same synchronous assumption. The service-oriented style (SOA/ESB, §8.1) is RPC's industrialized descendant.

### 2.5 Messaging

**Mechanism:** Applications communicate by exchanging **messages** via a **message channel** managed by a message broker / message-oriented middleware (MOM) — the messaging system's job is to move the message reliably from sender to receiver (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the full reliability treatment). The sender does not wait; the receiver does not have to be up. Store-and-forward, decoupling, and **asynchrony** are the essence.

**Pros:** the **decoupled style** — sender and receiver know only the channel, not each other; reliable delivery (at-least-once, retries, dead-letter channels); natural fit for events and fan-out (publish/subscribe); throttling and load leveling; enables the whole EIP catalogue (§3) because routers and transformers sit *between* sender and receiver. **Cons:** asynchronous programming model (no natural "return value" — request/reply must be built from two messages), eventual consistency (the receiver is not updated at the instant the sender acts), and operational complexity (the broker is now a critical system; monitoring, ordering, dedupe, and exactly-once all become your problem — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)).

Messaging is the style the **Enterprise Integration Patterns are built on** — the EIP book is, precisely, *"Designing, Building, and Deploying Messaging Solutions"*. It is also the style of the modern event-driven estate: Kafka, RabbitMQ, Pulsar, JMS/IBM MQ (see [event_stream_processing_guide.md](event_stream_processing_guide.md)).

### 2.6 The Styles Table

| Style | Mechanism | Pros | Cons |
|-------|-----------|------|------|
| **File Transfer** | A exports a file; the file moves; B imports it (batch) | Simplest; decoupled; batch-friendly; auditable; mainframe-native | Stale data; format coupling; no transactions; no request/reply; error handling is manual |
| **Shared Database** | A and B read/write the same DB | Real-time; simple (SQL); transactional; no transport | Schema coupling; no encapsulation; contention; model drift; the schema becomes the interface |
| **RPC** | A synchronously calls a procedure on B (CORBA/DCOM/SOAP/REST/gRPC) | Natural call model; immediate response; good for request/reply | Tight coupling; caller blocks on remote availability; poor fan-out; fragile |
| **Messaging** | A sends a message to a channel; B consumes it asynchronously (MOM/broker) | Maximal decoupling; reliable; event-friendly; fan-out; load leveling | Async model; eventual consistency; broker ops; ordering/dedupe/exactly-once are your problem |

**The architect's rule of thumb:** batch bulk → file transfer; shared operational truth with transactional needs → shared database (with discipline); immediate request/reply → RPC; decoupling, events, reliability, fan-out → messaging. And when in doubt, **messaging** — because it is the only style that keeps the door open to the patterns in the rest of this guide.

### 2.7 Choosing a Style: The Decision Guidance

The styles are not a ladder (newer ≠ better) — they are a **decision space with different failure modes**, and mature estates run all four at once. The practical guidance, legacy-flavored:

| You need... | Choose... | Because... |
|-------------|-----------|------------|
| Bulk data on a schedule (regulatory reporting, interbank files, EOD extracts) | **File transfer** (MFT) | The core's batch model, the auditors' evidence model, and the regulators' file formats all agree |
| A synchronous business operation (check balance, post a payment) with an immediate answer | **RPC** (API/ACL facade) | Request/reply semantics; keep it behind the ACL so the core's protocol stays a local concern |
| Events, fan-out, decoupling, resilience (the modern backbone) | **Messaging** | The only style that decouples sender from receiver and lets routers/transformers live between them |
| One genuinely shared operational dataset with transactional integrity | **Shared database** | Only when the data truly belongs to one domain — otherwise you are *building* the next silo |

**The composite reality:** a bank's estate is not "file transfer OR messaging" — it is **file transfer for the regulatory/interbank bulk, RPC through the ACL for channel operations, messaging (Kafka/IBM MQ) for the event backbone, and a governed shared layer only where the warehouse truly needs it**. The patterns that follow are mostly about the messaging layer, because that is where the decoupling and the intelligence live — but a complete design knows which style each flow uses and why.

---

## 3. The Enterprise Integration Patterns

### 3.1 The EIP: Hohpe and Woolf, The 2003 Book (verified)

**Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions** — Gregor Hohpe and Bobby Woolf, Addison-Wesley, **November 2003**, ~683 pages, in the **Addison-Wesley Signature Series** (Martin Fowler's series; foreword by John Crupi). The book is the field's foundational text: it catalogues the recurring integration problems and their solutions as a formal **pattern language**, each pattern with a name, a problem statement, a solution, and — critically — a **visual notation** (the iconic EIP diagram shapes: the channel as a line, the router as a circle with a "?"). The book is deliberately technology-agnostic: the examples use JMS, MSMQ, TIBCO, BizTalk, and XSL, but the patterns outlive all of them.

Why a *pattern language* matters: before EIP, integration conversations were "we'll use JMS and write an adapter" — implementation details with no shared vocabulary. After EIP, an architect can sketch a solution as **"a content-based router on the orders channel, transforming to the canonical model, with a dead-letter channel for poison messages"** — and every engineer in the room knows precisely what is meant. The patterns give the team a **common language**, which is the first integration win.

The EIP's enduring proof is that the modern integration toolchain is built on it: **Apache Camel** implements 80+ EIPs as first-class DSL constructs (see [camel_camelk_research.md](camel_camelk_research.md)); Spring Integration, Mule, and the cloud integration platforms (Azure Logic Apps, AWS Step Functions/EventBridge) all expose EIP-shaped primitives. What Hohpe & Woolf described on paper in 2003, Camel makes executable — the reference implementation is covered in §5.

### 3.2 Why Patterns for Integration

The book's own framing: applications were built independently and **"the only constant is that they must exchange information."** The four styles (§2) answer *how*; the patterns answer *what to build when messaging is the answer*. Patterns capture hard-won knowledge in a reusable, testable form — the same argument the GoF made for OO design in 1994, applied to the integration layer. For the legacy-integration architect this is doubly valuable: every pattern in the catalogue has been proven on estates like yours, and the vocabulary lets you describe a strangler-fig migration or an ACL in one sentence.

### 3.3 The Categories (verified — the EIP's structure)

The EIP book groups its patterns into **parts**, and the four that matter most are the ones this guide's catalogue (§5) is organized around:

1. **Message Channels** (book Part II) — the *plumbing*: how messages travel (point-to-point, publish-subscribe, dead letter, guaranteed delivery, message bus). *The answer to "where do messages go?"*
2. **Message Routers** (Part III) — the *decisions*: how messages are directed (content-based router, recipient list, splitter, aggregator, resequencer, filter). *The answer to "which way does this message go, and how do parts recombine?"*
3. **Message Transformers** (Part IV) — the *translation*: how messages change shape (message translator, normalizer, content enricher, claim check, canonical data model). *The answer to "how do we speak each system's language?"*
4. **Message Endpoints** (Part V) — the *plugs*: how applications connect to channels (messaging gateway, transactional client, polling/event-driven consumers, competing consumers, idempotent consumer, service activator). *The answer to "how does an application attach to the bus?"*

Two further parts complete the book: **Message Construction** (Part IV in the book's numbering — message, command message, event message, request-reply, correlation) and **System Management** (Part VI — control bus, wire tap, message history, detour, throttler — the *observability* patterns). This guide folds message construction into §4/§5 where relevant and treats system management as the ops layer.

### 3.4 The EIP Overview Table

| Category | The question it answers | Example patterns | Camel support (see [camel_camelk_research.md](camel_camelk_research.md)) |
|----------|------------------------|------------------|------|
| **Message Channels** | Where do messages travel? | Point-to-Point Channel, Publish-Subscribe Channel, Dead Letter Channel, Guaranteed Delivery, Message Bus | Yes — endpoints/components (`jms:`, `kafka:`, `activemq:`), error handler → dead letter |
| **Message Routers** | Which way does each message go? | Content-Based Router, Recipient List, Splitter, Aggregator, Resequencer, Message Filter, Dynamic Router | Yes — `.choice()`, `.split()`, `.aggregate()`, `.filter()`, `.recipientList()` |
| **Message Transformers** | How do we translate shapes and meanings? | Message Translator, Normalizer, Content Enricher, Claim Check, Canonical Data Model | Yes — `.transform()`, `.convertBodyTo()`, `.enrich()`, `.marshal()/.unmarshal()` |
| **Message Endpoints** | How do applications plug in? | Messaging Gateway, Transactional Client, Polling/Event-Driven Consumer, Competing Consumers, Idempotent Consumer, Service Activator | Yes — consumer/producer templates, `.pollEnrich()`, idempotent repository |
| **Message Construction** | What is a message, and how do conversations pair up? | Message, Command Message, Event Message, Request-Reply, Correlation Identifier | Yes — exchange/body/headers, `correlationId`, request-reply EIP |
| **System Management** | How do we observe and control the flow? | Control Bus, Wire Tap, Message History, Throttler, Delayer, Load Balancer | Yes — `.wireTap()`, `.throttle()`, `.loadBalance()`, message history |

### 3.5 The Anatomy of an EIP Pattern

To read the catalogue (§5) like an EIP native, know what each pattern entry is made of. Following the book's format (itself the GoF pattern format, adapted to integration), every pattern has:

- **Name** — the vocabulary unit. "Aggregator" says more than any paragraph.
- **Problem** — the recurring integration situation ("we have a batch file but downstream wants individual events").
- **Forces** — the constraints that pull the solution in different directions (decoupling vs. control, immediacy vs. reliability, simplicity vs. capability).
- **Solution** — the named structure that resolves the forces (the splitter's one-to-many fan-out with correlation).
- **Visual icon** — the EIP notation: channels as lines, endpoints as squares/arrows, routers as circles with a "?" — the *diagram is the design*, and the icon language is why EIP diagrams are readable by any integrator.
- **Implementation notes** — the book gives each pattern in JMS/MSMQ/TIBCO terms; today the same notes read "in Camel: `.split()`" (see [camel_camelk_research.md](camel_camelk_research.md)).
- **Related patterns** — how patterns compose (splitter ↔ aggregator; translator ↔ normalizer; enricher ↔ claim check).

**Worked anatomy — the Content Enricher (§4.6):** problem: the message lacks data the receiver needs; forces: the sender cannot (or must not) supply it, and the receiver must not go looking itself; solution: an intermediate component consults a reference source and augments the message in transit; icon: a channel with a small circle tapping a data source; implementation: `.enrich()`; related: claim check (the inverse — store, don't add).

### 3.6 The EIP in the Wild: 2003 → Today

The pattern language did not stay in the book. The lineage is direct and observable: **Camel** made EIPs executable DSL constructs (2007, Apache; 80+ patterns — see [camel_camelk_research.md](camel_camelk_research.md)); Spring Integration and Mule exposed the same primitives; the cloud integration platforms (Azure Logic Apps, AWS EventBridge/Step Functions) and the iPaaS vendors (§8.3) rebuilt EIP concepts as managed services; and event-streaming platforms (Kafka's ecosystem) supply the channels the patterns route over (see [event_stream_processing_guide.md](event_stream_processing_guide.md)). **A pattern from 2003 — the content-based router — is literally the routing logic inside a Kafka Streams topology or a Logic App switch today.** That longevity is the strongest evidence that the EIP catalogue names real, durable integration forces rather than vendor fashions.

---

## 4. The Key Patterns

Six patterns carry the bulk of real-world integration work. Master these and you can design — and *describe* — most legacy-integration solutions. Each is presented with its intent, its solution, a Camel DSL sketch (see [camel_camelk_research.md](camel_camelk_research.md) for the framework), and its typical legacy use.

### 4.1 Message Channel (verified — EIP)

**Intent:** *Connect applications by placing messages on a channel; applications never communicate directly.* The channel is the logical pipe between a sender and one or more receivers — in practice a queue or topic on a broker (JMS queue, Kafka topic, RabbitMQ exchange+queue). The two channel types are **Point-to-Point** (one message, one consumer — work queues) and **Publish-Subscribe** (one message, all subscribers — event fan-out).

**Why it matters for legacy:** the channel is the *decoupling device* — the mainframe core can publish account events to a topic without knowing or caring who consumes them, and ten downstream systems can subscribe without the core changing. The channel is also where reliability lives (delivery semantics, retries, dead-lettering — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)).

```java
// Point-to-point: orders in, one worker consumes
from("jms:queue:orders").to("bean:orderProcessor");
// Publish-subscribe: every subscriber gets the event
from("jms:topic:customerEvents").to("bean:customerViewUpdater");
```

### 4.2 Message Router (verified — EIP)

**Intent:** *Decouple the source of a message from its destination by routing it to the right receiver based on message content or state.* The canonical form is the **Content-Based Router**: inspect the message, send it down the matching branch. Related forms: Recipient List (send to *all* matching receivers), Message Filter (drop non-matching), Dynamic Router (decision made by the receivers).

**Why it matters for legacy:** the router is how you **insert policy without touching the systems**. A payment instruction from the core can be routed to RTGS, SWIFT, or the domestic ACH rail by header or amount — a decision the mainframe never had to make and never needs to learn.

```java
from("jms:queue:payments")
    .choice()
        .when(header("rail").isEqualTo("RTGS")).to("jms:queue:rtgs")
        .when(header("rail").isEqualTo("SWIFT")).to("jms:queue:swift")
        .otherwise().to("jms:queue:ach");
```

### 4.3 Message Translator (verified — EIP)

**Intent:** *Translate a message from one system's data format into another's, so that systems that speak different languages can communicate.* The translator is the **adapter at the message level**: COBOL copybook record ↔ JSON ↔ ISO 20022 ↔ fixed-width — each system talks to its own side, the translator mediates.

**Why it matters for legacy:** the *entire* problem of integrating a mainframe is translation — EBCDIC vs ASCII, packed decimal vs string, copybook layouts vs REST schemas, bank-internal codes vs SWIFT codes. Every legacy integration has a translation layer; the pattern's discipline is to **centralize it** (in the ACL, §6.2) instead of scattering conversion logic through every consumer.

```java
from("file:data/incoming")                    // fixed-width legacy extract
    .unmarshal().fixedLength()                // parse copybook-style layout
    .marshal().json()                         // emit modern JSON
    .to("kafka:topic:accounts");
```

### 4.4 Aggregator (verified — EIP)

**Intent:** *Combine related messages into a single message, using a correlation identifier to group them and a completion condition to know when the group is done.* The aggregator is the pattern that **rebuilds a whole from parts**.

**Why it matters for legacy:** classic uses — assemble a daily statement from hundreds of transaction messages; consolidate a customer's positions across multiple product systems into one balance snapshot; collect all confirmations of a multi-leg trade before releasing the settlement instruction. The correlation identifier is usually the account/order/trade ID.

```java
from("kafka:topic:txnEvents")
    .aggregate(header("accountId"), new StatementAssembler())
    .completionSize(500)                       // or completionTimeout
    .to("jms:queue:statements");
```

### 4.5 Splitter (verified — EIP)

**Intent:** *Split one message into many — a batch into individual messages, a composite order into line items — so each part can be processed independently.* The mirror image of the aggregator.

**Why it matters for legacy:** legacy systems love **bulk** (a file of 10,000 records is one transfer); modern systems love **granular** (10,000 events, each routable, retryable, observable). The splitter is the pattern that converts the legacy batch world into the event world, one record at a time — the bridge between the file-transfer style (§2.2) and messaging (§2.5).

```java
from("file:data/payments")                    // one big legacy file
    .split(body().tokenize("\n"))             // into individual lines
    .convertBodyTo(Payment.class)             // each becomes a message
    .to("jms:queue:paymentEvents");
```

### 4.6 Content Enricher (verified — EIP)

**Intent:** *Add missing data to a message by looking it up from another source — without the original sender having to know about it.* Enrichment happens *in transit*: the enricher consults a reference source (another system, a database, an API) and augments the message before forwarding.

**Why it matters for legacy:** the core knows the transaction but not the customer's risk score; the payments hub knows the instruction but not the counterparty name; the CRM knows the customer but not the position. Enrichment **lets thin legacy messages become rich events downstream** without asking the legacy system to do anything new.

```java
from("jms:queue:transactions")
    .enrich("jms:queue:customerRef", new CustomerEnricher())  // look up & merge
    .to("kafka:topic:enrichedTransactions");
```

### 4.7 The Key-Pattern Table

| Pattern | Category | Use case (legacy-flavored) |
|---------|----------|---------------------------|
| **Message Channel** | Channels | Decouple the mainframe core from every consumer; publish account events once, subscribe many |
| **Message Router** | Routers | Route payments/instructions to the right rail (RTGS/SWIFT/ACH) by content, without touching the core |
| **Message Translator** | Transformers | Copybook ↔ JSON ↔ ISO 20022; make the legacy format a *local* concern |
| **Aggregator** | Routers | Assemble statements, consolidate positions, correlate multi-part conversations |
| **Splitter** | Routers | Turn batch files into individual events; bridge file-transfer to messaging |
| **Content Enricher** | Transformers | Add counterparty, risk, or customer context to thin legacy messages in transit |

### 4.8 Caveats and Anti-Patterns (the honest footnotes)

Every pattern has a failure mode when over-applied. The practitioner's footnotes:

- **Message Channel** — *ordering is not free*: topic ordering and queue ordering are per-partition/per-queue guarantees, not global; an unbounded queue is a hidden backlog. Watch: queue depth, consumer lag (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §11).
- **Message Router** — *routing on fragile fields* (a header that half the senders omit) and *rule sprawl* (a router that grows into a god-object holding every business rule). The router should know *where*, not *why* — policy should live with the rules engine/workflow (see [temporal_workflow_guide.md](temporal_workflow_guide.md) for the orchestration angle).
- **Message Translator** — *translator sprawl*: N systems × M formats = N×M translators, unless a **canonical data model** breaks the mesh. And *silent field drift*: the legacy copybook changed and nobody updated the mapping — schema/contract tests are the discipline.
- **Aggregator** — *memory* (a long-running incomplete group holds state forever — always set a completion timeout), *partial groups* (what happens to the 499 of 500 messages when the 500th never arrives?), and *correlation-key collisions* (reused IDs mixing groups).
- **Splitter** — *one bad item fails the whole batch* (decide: poison-item routing to the DLQ vs. whole-batch rollback — in legacy batch worlds the latter is often the regulatory requirement), and *lost correlation* when the parts need to rejoin downstream (pair the splitter with an aggregator and a correlation identifier).
- **Content Enricher** — the *N+1 lookup* (enriching 10,000 events with 10,000 synchronous calls — batch the enrichment source), *enrichment staleness* (enriched data is a snapshot, not a join — label it), and *source availability* (the enricher's reference system going down takes the pipeline down — enrich from a local cache/projection when possible).

The unifying anti-pattern across all six: **building the patterns into the applications instead of between them**. A router inside a service is business logic; a router on the bus is an integration pattern. Keep the patterns in the integration layer (the ACL, the Camel routes, the backbone) — that is what makes them replaceable when the legacy estate finally shrinks.

---

## 5. The Pattern Catalogue

The full catalogue — the patterns this guide's author reaches for on legacy estates, organized by EIP category, with the use case and whether Apache Camel ships first-class support. **Camel support** = the pattern exists as a direct DSL construct or endpoint semantic in Camel (see [camel_camelk_research.md](camel_camelk_research.md) for the implementation details; Camel implements 80+ EIPs across Routing, Transformation, Mediation, Messaging, and System Management categories).

### 5.1 The Catalogue Table

| Pattern | Category | Use case | Camel support |
|---------|----------|----------|---------------|
| Message Channel | Channels | Logical pipe between sender and receiver(s); queue or topic | Yes — endpoints (`jms:`, `kafka:`, `activemq:`, `seda:`) |
| Point-to-Point Channel | Channels | Exactly one consumer takes each message (work queue) | Yes — queue endpoints |
| Publish-Subscribe Channel | Channels | Every subscriber receives each message (event fan-out) | Yes — topic endpoints |
| Dead Letter Channel | Channels | Park undeliverable/poison messages for investigation instead of losing them | Yes — `.errorHandler(deadLetterChannel("..."))` |
| Guaranteed Delivery | Channels | Broker persists messages so delivery survives crashes | Yes — broker components with persistence (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)) |
| Message Bus | Channels | One shared infrastructure with a common command set for all apps | Yes — the CamelContext *is* the bus |
| Content-Based Router | Routers | Route each message by its content/headers | Yes — `.choice()/.when()/.otherwise()` |
| Recipient List | Routers | Send each message to *all* matching destinations | Yes — `.recipientList()` |
| Message Filter | Routers | Drop messages that don't match a predicate | Yes — `.filter()` |
| Splitter | Routers | One message → many (batch file → individual events) | Yes — `.split()` |
| Aggregator | Routers | Many related messages → one (statements, consolidations) | Yes — `.aggregate()` + correlation/completion |
| Resequencer | Routers | Re-order out-of-sequence messages before delivery | Yes — `.resequence()` |
| Dynamic Router | Routers | Route decisions made by participants at runtime | Yes — `.dynamicRouter()` |
| Message Translator | Transformers | Convert formats between systems (copybook ↔ JSON) | Yes — `.transform()`, `.convertBodyTo()`, `.marshal()/.unmarshal()` |
| Normalizer | Transformers | Translate many source formats into one canonical format | Yes — `.choice()` + `.convertBodyTo()` per format |
| Content Enricher | Transformers | Add data from another source in transit | Yes — `.enrich()`, `.pollEnrich()` |
| Claim Check | Transformers | Store large payload aside, pass a token, restore later | Yes — `.claimCheck()` |
| Canonical Data Model | Transformers | A shared, standard data model all systems translate to/from | Yes — via data formats & type converters |
| Message | Construction | The atomic packet: body + headers + correlation | Yes — Exchange/Message model |
| Command Message | Construction | A message that asks the receiver to do something | Yes — any routed message |
| Event Message | Construction | A message announcing that something *happened* | Yes — event-driven routes |
| Request-Reply | Construction | Two messages pair a request with its reply | Yes — request/reply EIP, `.request()` |
| Correlation Identifier | Construction | The field that ties related messages together | Yes — `correlationId` header |
| Messaging Gateway | Endpoints | Hide messaging behind a method call in the app | Yes — producer/consumer templates |
| Transactional Client | Endpoints | Send/receive atomically with the app's own transaction | Yes — transacted routes |
| Polling Consumer | Endpoints | App pulls messages when it is ready (fits batch cores) | Yes — `from("jms:queue:...?consumerType=..." )`, file polling |
| Event-Driven Consumer | Endpoints | Broker pushes messages to the app | Yes — default consumer model |
| Competing Consumers | Endpoints | Multiple workers share a queue for scale | Yes — concurrent consumers |
| Idempotent Consumer | Endpoints | Dedupe repeated deliveries (at-least-once + duplicates) | Yes — `.idempotentConsumer()` with repository |
| Service Activator | Endpoints | Expose an app's function as a message consumer | Yes — `.bean()`, `.to("bean:...")` |
| Wire Tap | System Mgmt | Copy messages to a side channel for monitoring | Yes — `.wireTap()` |
| Message History | System Mgmt | Track the path a message has taken | Yes — message history EIP |
| Control Bus | System Mgmt | Manage integration infrastructure via messages | Yes — control bus component |
| Throttler / Delayer | System Mgmt | Rate-limit and pace message flow (protect legacy cores!) | Yes — `.throttle()`, `.delay()` |
| Load Balancer | System Mgmt | Distribute messages across equivalent endpoints | Yes — `.loadBalance()` |
| Circuit Breaker | System Mgmt | Fail fast when a downstream system is down | Yes — `.circuitBreaker()` |

**Reading the table:** the pattern names *are* the vocabulary of a modern integration architecture. "We run an event-driven backbone (publish-subscribe channels) with a canonical data model, translate at the edges (message translators inside the ACL), split legacy batch files (splitter), enrich with customer context (content enricher), aggregate statements (aggregator), and park failures in a dead-letter channel — with throttlers protecting the mainframe and idempotent consumers on every core-facing intake." Every clause of that sentence is one row of this table, and every row is implemented (or implementable) directly in Camel — which is why [camel_camelk_research.md](camel_camelk_research.md) is the natural companion to this guide.

### 5.2 The Classic Pattern Combinations

Patterns rarely appear alone; they compose into **recognizable recipes** that recur on every legacy estate:

1. **The batch-to-event bridge** — *file → splitter → translator → channel*. The legacy EOD extract becomes a stream of individual, routable, retryable events. The single most common modernization move in banking (see §9).
2. **The reliability trio** — *dead-letter channel + idempotent consumer + retry*. Every core-facing intake gets it: at-least-once delivery in, exactly-once effects out, poison messages parked for inspection (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)).
3. **The enrich-and-route pipeline** — *enricher → content-based router → target channels*. Thin core events become rich, context-tagged events, then route to the consumers that need them (risk, CRM, analytics).
4. **The request-reply pair** — *two channels + correlation identifier*. The asynchronous answer to "give me a synchronous answer": request channel out, reply channel back, correlated by ID — how the ACL offers RPC-style operations to channels over a messaging backbone.
5. **The strangler's routing front** — *facade/ACL → content-based router → (legacy | new implementation)*. The single decision point that makes incremental migration possible (§6.1): each request routed to the old core or the new service, function by function.

In Camel terms, combinations 1–3 are a few lines of DSL chained on one route; combination 5 is the architectural pattern the whole route topology serves.

---

## 6. The Legacy Patterns

The EIP catalogue assumes you *can* build messaging around your systems. The legacy patterns answer a harder question: **how do you integrate with — and eventually retire — systems you can barely change at all?**

### 6.1 The Strangler Fig Pattern (verified — Fowler, 2004)

**Origin:** Martin Fowler, bliki post **"Strangler Fig Application"** (2004; later retitled, and updated as recently as 2024). The metaphor: the strangler fig vine grows around a host tree, gradually enveloping it, until the host dies and the vine *is* the tree. Fowler observed the trees in Queensland and realized it is the perfect image for application migration: **don't rewrite the monolith — grow the new system around it, function by function, until the old one can be cut away.**

**The mechanics (transform → coexist → eliminate):**
1. **Transform:** put a **facade** (a routing layer) in front of the legacy system so all traffic flows through one controllable point.
2. **Coexist:** incrementally route individual functions to new implementations — balances first, then payments, then statements. Each replaced function is one less reason the legacy system exists; both systems run side by side, the facade deciding which serves which request.
3. **Eliminate:** when the last function has been strangled, retire the legacy system. (In banking, "retire" often means "freeze and keep read-only for regulatory look-back" — the pattern still did its job.)

**Why it is the definitive legacy pattern:** it **de-risks migration by making it incremental and reversible**. No big-bang rewrite (which statistically fails); every step is a small, testable slice; business value lands from the first function; and the facade gives you a single place to control the whole migration. It is the pattern behind most successful core-banking modernizations (see [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) and [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md) for the core-replacement landscape the strangler navigates).

### 6.2 The Anti-Corruption Layer (verified — Fowler, 2004)

**Origin:** Martin Fowler, bliki post **"AntiCorruptionLayer"** (2004) — arising from Domain-Driven Design (Eric Evans, 2003/2004) and the **bounded context**: each system has its own model of the world, and if you let one system's model leak into another's, the receiving system's model gets *corrupted*.

**The pattern:** when integrating with a system whose model you cannot change (the legacy core!), build a **dedicated translation layer — the anti-corruption layer (ACL)** — between it and your domain. The ACL:
- **Translates** the legacy model into your model (copybook → domain objects, "account status 3" → `ACCOUNT_FROZEN`, five address formats → one);
- **Isolates** your system from the legacy system's quirks, so legacy concepts never leak into your code;
- **Owns** the mapping logic in one place, so when the legacy system finally dies, you delete the ACL and your domain is untouched.

The ACL is the *structural* realization of the **Message Translator** pattern (§4.3) and the standard integration pattern in DDD. In legacy-integration practice it is usually implemented as a **facade service/API** in front of the core: modern JSON/ISO 20022 in, COBOL copybook calls out. It is also the natural place to put the EIP translators, throttlers, and idempotency that protect the core.

### 6.3 The Supporting Cast

The strangler's facade and the ACL's translation are often joined by three older, simpler patterns: **Facade** (a single simplified interface in front of a complex subsystem), **Adapter** (convert one interface to another — the object-level cousin of the translator), and **Legacy Wrapper** (wrap the legacy system's interface so new code talks to a clean API; the wrapper can be a service activator or gateway on the messaging side, §4.1/§5). All three are *enabling* patterns: they exist to make the strangler and the ACL possible.

### 6.4 The Legacy-Pattern Table

| Pattern | Origin | Intent | Mechanism | When to use | Risks |
|---------|--------|--------|-----------|-------------|-------|
| **Strangler Fig** | Fowler, 2004 (bliki) | Replace a monolith/legacy system incrementally, without a big-bang rewrite | Facade → route function-by-function to new implementation → retire old | Rewriting is too risky; system is too big to replace at once | Long migration; dual-running costs; facade becomes a bottleneck; scope creep |
| **Anti-Corruption Layer** | Fowler, 2004 (bliki; DDD) | Protect your domain model from a system whose model you can't change | Dedicated translation layer (facade service) between systems; mapping owned in one place | Integrating with cores/third parties; DDD bounded contexts; anytime a model would leak | Mapping drift; ACL grows fat ("god translator"); performance overhead |
| **Facade / Adapter / Legacy Wrapper** | GoF (1994) / integration practice | Simplify or translate an interface so the legacy system can be consumed cleanly | Single interface in front of the legacy surface; wrap calls | Every legacy integration, as the *enabling* layer under the ACL/strangler | Layers multiply; one more hop to debug |

**The legacy-pattern stack in one sentence:** *wrap the legacy system so it is consumable (facade/wrapper), translate its model at the boundary (ACL), and migrate function by function through a single routing point (strangler) — the ACL keeps you clean today, the strangler retires the beast tomorrow.*

### 6.5 Choosing Between the Legacy Patterns

The three legacy patterns are complementary, but each leads in a different situation:

| Situation | Lead with... | Because... |
|-----------|--------------|------------|
| "We must integrate with the core now, and the core will live for years" | **ACL** | You need a durable, well-governed boundary; the ACL *is* the integration contract |
| "We must replace the core's functions, but a big-bang rewrite is unaffordable/unsafe" | **Strangler fig** | Incremental, reversible, business-value-per-phase — the only migration strategy with a track record in core banking |
| "One legacy system's interface is awkward but we just need to consume it today" | **Wrapper/Adapter** | Cheapest option; use it as the seed of the ACL if the relationship will outlive this quarter |
| "We're not sure yet" | **Facade + ACL-first** | The facade gives you a single point of control now and becomes the strangler's routing front later — the safest default |

The decision rule: **the ACL is the investment, the strangler is the exit strategy, and the wrapper is the expedient** — and a well-run program builds all three in that order.

---

## 7. The Modernization Patterns

Where the legacy patterns govern *how to touch old systems*, the modernization patterns govern *what the target architecture looks like*. Three patterns define the modern shape: **event-driven** (the backbone), **CQRS** (the read-side), and the **transactional outbox** (the write-side glue that makes events trustworthy).

### 7.1 Event-Driven (verified — the modern default)

**The pattern:** systems communicate by **publishing events** (facts that happened: `AccountOpened`, `PaymentSettled`, `BalanceChanged`) to an event backbone; consumers subscribe and react. No system asks another for data; each system publishes what it knows and consumes what it needs. The event stream becomes the **system of record for what happened**, and downstream systems build their own views (projections) from it.

**Why it matters for legacy:** the event backbone is the *strangler's best friend* — legacy batch outputs can be split into events (§4.5), the core's changes can be captured via **CDC** (change data capture) and published without touching the core, and every new system subscribes instead of point-to-point-calling the core. The full stream-processing treatment — fault tolerance, at-least-once, reprocessing — is in [event_stream_processing_guide.md](event_stream_processing_guide.md); the reliability contract for the backbone itself is in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md). The banking reality — posting engines and core processes emitting events — is in [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) and [banking/core_banking_processes_guide.md](banking/core_banking_processes_guide.md).

### 7.2 CQRS — Command Query Responsibility Segregation (Fowler bliki; **date flagged — see §11**)

**The pattern (Fowler's definition):** *"use a different model to update information than the model you use to read information."* Commands (writes) go to a write model; queries (reads) go to one or more **read models** — projections of the write side, optimized for how data is actually consumed (denormalized, aggregated, read-optimized). The write side emits events; the read side subscribes and builds its projections (§7.1 is CQRS's natural carrier).

**Origin note (flag):** the task brief for this guide said "Fowler, 2010". Verification found Fowler's bliki page **CQRS.html is dated 14 July 2011**; Greg Young (who coined the term and developed the pattern, ~2010) is credited in Fowler's post as the source. So: *pattern ~2010 (Young), Fowler's canonical write-up 2011*. Fowler also warns: **"for most systems CQRS adds risky complexity"** — use it where the read/write asymmetry is real (reporting-heavy, high-read-scale domains), not as decoration.

**Why it matters for legacy:** the legacy core is a write-optimized batch machine that is terrible at serving modern reads (real-time balances, analytics, customer 360). CQRS lets you **leave the core as the write model** and build read models from its events — the core keeps doing what it does, and the read side becomes fast, modern, and independently scalable. This is precisely how banks deliver real-time balance APIs on top of batch cores.

### 7.3 The Transactional Outbox (verified — see message_queue_data_loss_guide.md)

**The problem:** the **dual-write problem** — an application updates its database *and* publishes an event; if the DB write succeeds but the publish fails (or vice versa), the event stream and the database disagree, and every downstream consumer is built on a lie. (The three loss points of messaging — producer→broker, broker→consumer, consumer→downstream — are dissected in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §1.)

**The pattern:** in the *same* database transaction that changes business state, also insert an **outbox record** (the event, serialized, with an id and status). Then a **relay** (a poller, or CDC — e.g., Debezium reading the outbox table's binlog) publishes outbox rows to the broker and marks them sent. Because the event is written *atomically* with the state change, the stream can never disagree with the database — and the relay can retry forever without losing or duplicating events (the consumer stays idempotent, §5).

**Why it matters for legacy:** the outbox is the pattern that makes **the legacy core a trustworthy event source**. The core already has a database transaction for every business change; adding an outbox table to that transaction means the core can emit events *without any reliable-messaging capability of its own* — the mainframe doesn't need to know Kafka exists. The full treatment (dedupe, exactly-once, broker integration) is in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10.

### 7.4 The Modernization Table

| Pattern | Problem it solves | Key mechanism | Modern role | Cross-ref |
|---------|-------------------|---------------|-------------|-----------|
| **Event-Driven** | Point-to-point coupling; no system-of-record for facts | Event backbone; publish/subscribe; projections from events | The target architecture's nervous system; strangler fuel | [event_stream_processing_guide.md](event_stream_processing_guide.md) |
| **CQRS** | Read/write model mismatch; cores can't serve modern reads | Separate write model (the core) from read models built from events | Real-time APIs and analytics without touching the core | §7.2; Fowler CQRS bliki (2011) |
| **Transactional Outbox** | Dual-write inconsistency (DB vs broker) | Event written atomically with state; relay (poller/CDC) publishes | Makes the legacy core a trustworthy event source | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10 |

**The modernization stack in one sentence:** *capture the legacy core's changes as events (outbox + CDC), carry them on an event backbone (event-driven), and serve modern reads from projections (CQRS) — while the strangler (with its ACL) gradually absorbs the core's functions.*

### 7.5 The Modernization Realism Check

The modernization patterns are powerful and fashionable — which is exactly why they deserve a counterweight. They are the right answer when the estate is large, the data is shared, and the change rate is high; they are the wrong answer when:

- **The estate is small or the change rate is low** — a handful of systems with stable interfaces do not need an event backbone; the overhead of brokers, outbox relays, and projections will exceed the benefit. A clean ACL and a few file/API flows are the proportionate answer.
- **Regulatory or audit reality forbids distributed state** — some jurisdictions and some product lines (certain ledger/position functions) require transactional, reconcilable state that eventual consistency cannot offer; there the outbox still applies (it preserves atomicity *at the source*), but event-driven fan-out of the ledger itself may not. The distributed-transaction trade-offs are covered in [apache_seata_guide.md](apache_seata_guide.md).
- **The team cannot operate the machinery** — an event backbone without operators who understand lag, offsets, and replay (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)) is a liability, not an architecture. Fowler's warning on CQRS applies to the whole stack: **these patterns add risky complexity; use them where the asymmetry is real, not as decoration.**

The realism test in one question per pattern: *does this pattern remove a failure mode the business actually experiences?* Live-balance demands → yes, event-driven + CQRS. Regulatory file formats → no, file transfer. Duplicate payment risk → yes, outbox + idempotent consumer. Everything else → start with the simple thing that works and let the patterns earn their place.

---

## 8. The Middleware

Patterns describe *what* to build; middleware is *what you build it on*. Three middleware families dominate the legacy-integration landscape, and choosing between them is one of the first decisions an integration architect makes. The honest framing: **they overlap, they compose, and the boundaries have blurred** — but each has a center of gravity.

### 8.1 The ESB — Enterprise Service Bus (verified)

**What it is:** a middleware layer that **sits between applications and mediates their interactions** — routing, transforming, orchestrating, and protocol-converting messages between services in a service-oriented architecture (SOA). The term was canonized by David A. Chappell's *Enterprise Service Bus: Theory in Practice* (O'Reilly, **2004**) and the SOA wave that followed: the ESB as the **"bus"** that every service plugs into, replacing point-to-point links with hub-and-spoke mediation.

**Capabilities:** routing/transformation (the EIPs — many ESBs *are* EIP engines), protocol conversion (SOAP↔JMS↔HTTP↔file), orchestration, security, and monitoring. Vendors: Mule, IBM Integration Bus (ex-Message Broker), TIBCO, Oracle Service Bus, WSO2, and open-source Camel (which is routinely used *as* a lightweight embedded ESB — see [camel_camelk_research.md](camel_camelk_research.md)).

**The critique:** ESBs became the poster child of over-centralization — a **single point of failure, a single point of bottleneck, and a single point of vendor lock-in**, with "ESB as kitchen sink" architectures that routed everything through one hub. The industry's later lesson (and Camel's design philosophy) is: *keep the mediation logic, drop the heavy hub* — embed the ESB capability (routes, transformers, patterns) inside the services or at the edges (which is exactly what an ACL built on Camel is).

### 8.2 The Message Broker (verified)

**What it is:** the **transport** — message-oriented middleware (MOM) that owns channels, queues, topics, and delivery semantics: JMS brokers (IBM MQ, ActiveMQ), AMQP (RabbitMQ), and log-based streaming brokers (Kafka, Pulsar, Redpanda). The broker's job is narrower than the ESB's: **reliably move messages** — persist, replicate, deliver, retry, dead-letter. It is not (natively) a transformation or orchestration engine; you put the EIPs around it (in Camel, in your services) and the broker underneath it.

**The relationship to the ESB:** historically, ESBs sat *on top of* brokers (the bus mediated; the MOM transported). Today the broker is the backbone of the event-driven estate — Kafka's log, RabbitMQ's queues, IBM MQ's transactional point-to-point — and the reliability contract (acks, replication, offsets, dedupe, exactly-once, DLQs) is documented in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md). For legacy estates, the broker is also the *least invasive* middleware: the core doesn't integrate with the bus; it just gets a producer (often the outbox relay, §7.3).

### 8.3 The iPaaS — Integration Platform as a Service (verified)

**What it is:** **integration-platform-as-a-service** — a *cloud-delivered* integration suite: pre-built connectors (SaaS APIs, databases, files, messaging), low-code flow design, transformation, orchestration, monitoring, and governance, all managed. The term was coined by **Gartner in 2011** — Massimo Pezzini and Benoit J. Lheureux, research note G00210747, *"Integration Platform as a Service: Moving Integration to the Cloud"* (7 March 2011). Vendors: MuleSoft Anypoint, Boomi, Azure Logic Apps, Workato, Informatica, SAP BTP IS.

**Why it matters:** iPaaS moves integration effort **out of the data center and out of the codebase** — business teams compose flows against a catalog of connectors. For legacy estates it is the pragmatic layer for **SaaS/cloud-to-core** integration (CRM ↔ core, HR ↔ core, cloud analytics ↔ core), where the legacy side is reached through the ACL/facade (§6.2) and the cloud side through connectors. The trade-offs: it is a *platform* (cost, lock-in, governance of citizen integrators), and heavy real-time/event workloads still belong on a broker with code-level control.

### 8.4 The Middleware Comparison Table

| Dimension | **ESB** | **Message Broker** | **iPaaS** |
|-----------|---------|--------------------|-----------|
| **Primary job** | Mediate: route, transform, orchestrate between services | Transport: reliably move messages (queues/topics, delivery semantics) | Integrate: connect apps with connectors + low-code flows |
| **Paradigm** | SOA hub-and-spoke (or embedded mediation) | MOM / streaming: publish-subscribe, point-to-point | Cloud integration platform; API-first |
| **Strength** | Central control of integration logic; protocol conversion | Decoupling + reliability + scale; event backbone | Speed to integrate SaaS/cloud; low-code; managed |
| **Weakness** | Central bottleneck; lock-in; "kitchen sink" risk | No built-in mediation (transform/route is your job) | Platform cost; less control; not for heavy real-time event work |
| **Legacy-fit** | The classic layer for core mediation — today better done as embedded/edge (Camel-based ACL) | The least-invasive option: core just needs a producer/relay (outbox) | Best for SaaS↔core flows through the ACL/facade |
| **Representatives** | Mule, IBM Integration Bus, TIBCO, WSO2, **Camel-as-ESB** | IBM MQ, RabbitMQ, Kafka, Pulsar, ActiveMQ | MuleSoft Anypoint, Boomi, Azure Logic Apps, Workato |
| **Cross-ref** | [camel_camelk_research.md](camel_camelk_research.md) | [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | §8.3 (Gartner G00210747) |

**The middleware one-liner:** *the broker carries the messages, the ESB (or its lean modern form — Camel-style mediation embedded at the edges) decides where they go and how they change, and the iPaaS plugs the cloud/SaaS world in — and a modern legacy estate usually runs all three, layered, with the ACL at the core boundary.*

### 8.5 The Middleware Selection Checklist

When the choice between ESB, broker, and iPaaS comes up, run the estate through these questions:

1. **What is the dominant flow shape?** Bulk/scheduled → file transfer + MFT (§2.2); events/fan-out → broker; request/reply operations → API/ACL layer (RPC, §2.4) — middleware follows the flows, not the reverse.
2. **Where does the intelligence live?** If routing/transformation is needed *between* systems, that is mediation — embedded EIP engines (Camel) or an ESB; if the transport is the hard problem (reliability, scale), that is a broker.
3. **Who consumes?** SaaS/cloud consumers → iPaaS connectors earn their keep; internal systems → broker/mediation; regulatory/bank counterparties → MFT files (their formats are files).
4. **What is the team's operating capacity?** Every middleware is a system you must operate — a broker's lag/offsets/DLQs, an ESB's flows, an iPaaS's governance. Choose the one the team can actually run (see §7.5's realism check).
5. **What is the lock-in tolerance?** Open-source embedded (Camel, Kafka, RabbitMQ) maximizes portability; iPaaS and commercial ESBs trade it for speed/managed ops.
6. **What does the core allow?** The core's interface reality (batch files only? CICS transactions? DB2 access?) constrains everything: the middleware must be able to *reach* the core through whatever the ACL exposes (see §6.2 and §9).

The one-sentence outcome: **default to a broker + embedded mediation (Camel) for the event/real-time layer, MFT for the bulk/regulatory layer, an API/ACL facade for operations, and iPaaS only where cloud/SaaS integration speed justifies the platform.**

---

## 9. The Worked Example: A Bank's Legacy Core Integration

### 9.1 The Scenario: The Bank's Core (the familiar context)

Consider a mid-sized bank — the profile that appears throughout this repository's banking guides ([banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md), [banking/dbs_software_systems_guide.md](banking/dbs_software_systems_guide.md), [banking/standard_chartered_guide.md](banking/standard_chartered_guide.md)): a **core banking system** — mainframe-era COBOL, or a packaged core in the T24/Flexcube/jBASE family (see [banking/temenos_guide.md](banking/temenos_guide.md), [banking/oracle_flexcube_data_model_guide.md](banking/oracle_flexcube_data_model_guide.md), [banking/jbase_vs_infobasic_guide.md](banking/jbase_vs_infobasic_guide.md), [banking/tafj_guide.md](banking/tafj_guide.md)) — surrounded by satellites: the payments hub, the CRM, the risk/AML engines, the channels (mobile, internet, branch, ATM). The core is the **system of record**: accounts, balances, interest, statements (see [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md) and [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md) for what happens inside it).

**The as-is state — the silos in full flower:** the channels get balance information from **end-of-day file extracts** (the file-transfer style, §2.2 — the bank runs Axway Transfer CFT-class MFT, cross-ref [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md)); the CRM loads customers overnight; risk gets flat files on a schedule; and a tangle of point-to-point links carries everything else. Symptom: the mobile app tells a customer her balance is "as of yesterday" — and after a card payment the balance is wrong for up to 24 hours. Fraud detection works on yesterday's data. The regulators are asking hard questions about real-time risk exposure. Meanwhile the core **cannot change**: the COBOL is undocumented, the skills are gone, and a core replacement is a 5–10-year, hundreds-of-millions program nobody can justify.

**The mandate:** *don't replace the core; make it behave like a modern, real-time service.* That mandate — and only that — is what the patterns in this guide are for.

### 9.2 The Patterns Applied: The Design

**Step 1 — The ACL in front of the core (anti-corruption layer, §6.2).** Build a dedicated **core services facade**: a stateless API layer (REST/ISO 20022-style JSON outward; COBOL copybook calls / CICS transactions inward) that owns *all* translation between the bank's modern domain model and the core's record model. Every consumer (channels, payments hub, CRM, risk) talks to the ACL — never to the core directly. The ACL is built on **Camel** (see [camel_camelk_research.md](camel_camelk_research.md)): each facade operation is a route whose tail is a **message translator** (§4.3) into the core's format, protected by a **throttler** (the mainframe can only take so many MIPS of CICS calls per second) and an **idempotent consumer** (at-least-once calls in, exactly-once effects out — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)).

**Step 2 — The outbox makes the core an event source (transactional outbox, §7.3).** The core's own transaction already records every account movement. Add an **outbox table** written in that same transaction (a small COBOL/DB2 change — one table, one insert — the cheapest possible change to a mainframe), and stand up a **relay** (a poller or CDC reader, e.g., Debezium on the DB2 log) that publishes `BalanceChanged`, `PaymentSettled`, `AccountOpened` events to a **Kafka backbone** and marks them sent. The core never learns what Kafka is. The events are trustworthy because they were written atomically with the state they describe.

**Step 3 — The strangler absorbs the core's functions (strangler fig, §6.1).** Now the roadmap: every core function is a candidate to be *strangled*. **Phase 1 (coexist):** balances and transaction history are served to the channels from **read models built on the event stream** (CQRS, §7.2) instead of from EOD files — the mobile app finally shows a live balance while the core continues as the write model. **Phase 2:** payments initiation moves to a new payments domain service that writes through the ACL for posting but owns the workflow itself (see [temporal_workflow_guide.md](temporal_workflow_guide.md) for the orchestration angle). **Phase 3:** statement generation, interest, and finally account opening move out. Each phase ends with the core serving one fewer function; the facade (the ACL) is the single routing point that decides, request by request, whether the old core or the new implementation answers. The core shrinks toward retirement without a single big-bang weekend.

**Supporting cast (the EIPs at work, §4/§5):** the event backbone is a set of **publish-subscribe channels**; incoming payment files from the MFT layer are **split** into individual payment events (§4.5); a **content-based router** sends each payment to RTGS/SWIFT/ACH by amount and currency (§4.2); a **content enricher** adds counterparty and risk context to core events before risk consumes them (§4.6); an **aggregator** assembles daily statements from the day's events (§4.4); and a **dead-letter channel** parks anything unprocessable, with a **control bus** (wire tap, message history) giving operations full observability of every flow.

### 9.3 The Phase Plan at a Glance

The strangler turns the program into **phases, each with its own patterns and its own visible business win**:

| Phase | What gets strangled / built | Patterns at work | Business outcome |
|-------|-----------------------------|------------------|------------------|
| 0 | The pipes: ACL facade, outbox table + relay, event backbone | ACL, transactional outbox, publish-subscribe channels, throttlers, idempotent consumers | The core becomes consumable and event-emitting — with one safe table change and zero core re-platforming |
| 1 | Balances & transaction history (read side) | CQRS read models, splitter, content enricher, aggregator | **Live balances in the mobile app** — the first customer-visible win; EOD file dependency for channels ends |
| 2 | Payments initiation & lifecycle | Content-based router (RTGS/SWIFT/ACH), request-reply, orchestration ([temporal_workflow_guide.md](temporal_workflow_guide.md)) | New payment products and rails without core changes; fraud sees payments in real time |
| 3 | Statements & reporting | Aggregator, normalizer, canonical data model | Same-day statements; the nightly statement batch shrinks |
| 4 | Account opening, interest, remaining functions | Strangler completes; ACL shrinks function by function | The core is down to read-only/look-back; retirement (or freeze) becomes a routine decision, not a program |

Each phase is independently valuable, independently reversible, and — critically — **the ACL is the only thing that knows which implementation answered**, so a phase can be rolled back by flipping the router without a system-wide change.

### 9.4 The Target Architecture Sketch

```
                    ┌─────────────────────────────────────────────┐
   Mobile / Web ───▶│                                             │
   Branch / ATM ───▶│   Core Services Facade = THE ACL (Camel)   │
   Payments Hub ───▶│   • message translators (copybook ↔ JSON)   │
   CRM / Risk ─────▶│   • throttlers protecting the core          │
                    │   • idempotent consumers                    │
                    └───────────────┬─────────────────────────────┘
                                    │ CICS / copybook calls
                    ┌───────────────▼─────────────────────────────┐
                    │   LEGACY CORE (COBOL mainframe)             │
                    │   + outbox table (written atomically)       │
                    └───────────────┬─────────────────────────────┘
                                    │ relay (poller / CDC)
                    ┌───────────────▼─────────────────────────────┐
                    │   EVENT BACKBONE (Kafka)                    │
                    │   balance/ payment/ account events          │
                    └───────┬──────────────┬──────────────┬───────┘
                            │              │              │
                 ┌──────────▼───┐  ┌───────▼──────┐  ┌────▼───────────┐
                 │ Read models  │  │ New payments │  │ Risk / AML    │
                 │ (CQRS) for   │  │ domain svc   │  │ (real-time)   │
                 │ channels     │  │ (strangled)  │  │ (enriched)    │
                 └──────────────┘  └──────────────┘  └────────────────┘
```

### 9.5 The Lessons: Integrate, Don't Replace

1. **The core doesn't need to change for the estate to modernize** — the ACL, the outbox, and the event backbone give the core a modern *skin* with one tiny, safe change (an outbox table). Everything else happens *around* it.
2. **Translation is a layer, not a habit** — all mapping lives in the ACL, in one place, so the day the core dies you delete one service and the domain is untouched.
3. **Events are the truth** — once the outbox makes the event stream consistent with the ledger, every downstream system can rebuild its own view (CQRS) and stop asking the core.
4. **Modernize in slices, not in a big bang** — the strangler turns a 10-year core replacement program into a sequence of quarterly, independently valuable slices (first visible win: live balances in the app).
5. **Protect the old, not just the new** — throttlers and idempotency at the ACL are not optional; the mainframe's capacity and the reliability contract (see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)) are what keep the migration from breaking production.
6. **The patterns are the vocabulary** — "strangler with an ACL, outbox-fed events, CQRS read models" is a complete architecture description that any architect on the program understands in one sentence.
7. **Integration is the strategy; replacement is the exit** — you integrate first so you *can* replace later, cheaply and safely. Integrate, don't replace — and the replacing takes care of itself.

---

## 10. The Summary: Integrate, Don't Replace

**The problem (one paragraph).** Enterprises — banks above all — run on legacy systems: mainframe COBOL cores and packaged platforms that hold the crown-jewel data, speak record-oriented languages, run on batch cycles, and cannot be changed. These silos duplicate data, drift semantically, and force point-to-point spaghetti or stale EOD file exchanges. Integration across them is hard because the most valuable systems are the hardest to touch — and the people who understood them are retiring.

**The styles (one paragraph).** Four integration styles — **file transfer** (batch, simple, stale), **shared database** (real-time, schema-coupled), **RPC** (synchronous, tightly coupled), and **messaging** (asynchronous, decoupled, reliable) — are the decision space. Messaging is the style the patterns are built on, because it is the only one that decouples sender from receiver and lets intelligence live *between* systems.

**The patterns (one paragraph).** The **Enterprise Integration Patterns** (Hohpe & Woolf, Addison-Wesley, 2003) named the game: **channels** carry messages, **routers** decide where they go (splitter, aggregator, content-based router), **transformers** translate shapes and meanings (message translator, content enricher), **endpoints** plug applications in. Six patterns — message channel, message router, message translator, aggregator, splitter, content enricher — carry most real work, and the full catalogue is implemented by **Apache Camel** (see [camel_camelk_research.md](camel_camelk_research.md)).

**The legacy patterns (one paragraph).** To integrate systems you can't change: the **strangler fig** (Fowler, 2004) grows the new system around the old, function by function, through a facade; the **anti-corruption layer** (Fowler, 2004) translates the legacy model at the boundary so it never corrupts yours. Together they make modernization incremental and reversible.

**The modernization patterns (one paragraph).** The target shape: **event-driven** (an event backbone as the system of record for what happened), **CQRS** (read models built from events so the batch core never has to serve real-time reads), and the **transactional outbox** (events written atomically with state, so the stream can't disagree with the ledger — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10). Middleware — **ESB**, **message broker**, **iPaaS** — carries it: broker transports, mediation lives at the edges (Camel), iPaaS connects the cloud.

**The worked example (one paragraph).** A bank's core: the **ACL** gives the core a modern face, the **outbox** makes it an event source with one safe table change, the **strangler** migrates balances → payments → statements in quarterly slices, and the EIPs (splitter, router, enricher, aggregator, dead-letter channel) do the daily work on the backbone. Live balances on day one; a retirement-ready core in years, not decades.

**The final word.** The legacy estate is not the enemy and it is not going anywhere soon. **Integrate, don't replace** — wrap it (facade, ACL), translate it (translators, canonical models), listen to it (outbox, events), and let it shrink (strangler) on its own schedule. The patterns in this guide are that strategy, named, verified, and executable.

---

## 11. Verification and Claims-Status

Per the research discipline of this series, each key factual claim was checked against sources during the writing of this guide. Items that could not be verified are **flagged** rather than silently asserted.

**Method.** Verification used (a) targeted web searches against publisher, vendor, and author primary sources (Amazon/Google Books publisher records, Pearson sample pages, O'Reilly records, Gartner's public research-note PDF, Apache Camel documentation, Fowler's bliki), and (b) cross-checks against this repository's sibling guides ([camel_camelk_research.md](camel_camelk_research.md), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), the banking guides), which were written to the same citation discipline. Where a claim rests only on widely-circulated but unconfirmed statistics, it is marked **flagged** below.

**Verified:**

- **The EIP book** — *Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions*, Gregor Hohpe & Bobby Woolf, Addison-Wesley, **November 2003**, ~683 pages, Addison-Wesley Signature Series (Fowler). Confirmed via publisher/bookseller records (Amazon, Google Books, Pearson sample pages). The book's Chapter 1 introduces exactly the four integration styles (File Transfer — first pattern at p.43, Shared Database, RPC, Messaging) confirmed in the published excerpt.
- **Strangler Fig** — Martin Fowler, bliki post "StranglerFigApplication", **2004** (later retitled; page updated 2024). Confirmed via Fowler's site and secondary citations.
- **CQRS (flagged date)** — the brief said "Fowler, 2010". Verification found Fowler's bliki *CQRS.html* is dated **14 July 2011**; the pattern is credited to **Greg Young** (circa 2010) in Fowler's own post. **Flagged: the correct citation is Young ~2010 / Fowler bliki 2011, not "Fowler 2010".** Fowler's bliki also carries his warning that CQRS adds risky complexity for most systems — quoted in §7.2.
- **ESB** — *Enterprise Service Bus: Theory in Practice*, David A. Chappell, O'Reilly, **2004** (June 2004). Confirmed via publisher records.
- **iPaaS** — term coined by **Gartner, 2011**: Massimo Pezzini & Benoit J. Lheureux, *"Integration Platform as a Service: Moving Integration to the Cloud"*, Gartner RAS Core Research Note **G00210747**, **7 March 2011**. Confirmed via the note's public PDF and industry citations.
- **Apache Camel as the EIP implementation** — Camel's own docs state it "supports most of the Enterprise Integration Patterns from the excellent book by Gregor Hohpe and Bobby Woolf"; the repo's [camel_camelk_research.md](camel_camelk_research.md) documents 80+ EIPs, 350+ connectors, six DSLs, created by James Strachan 2007, Apache 2.0. Confirmed.
- **Transactional outbox** — cross-referenced to [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10 (the dual-write problem, outbox mechanics, relay via poller/CDC). Consistent with the messaging-reliability literature.

**Flagged / not independently verified in this run:**

- **Anti-Corruption Layer date** — the bliki *AntiCorruptionLayer.html* (martinfowler.com) is **widely cited as 2004** alongside StranglerFigApplication; the exact publish date could not be re-confirmed directly in this run (page extraction unavailable in the search backend). Treat as "Fowler, 2004 (widely cited)".
- **Mainframe/COBOL statistics** — widely-circulated figures ("billions of lines of COBOL in production", "the majority of banking transactions touch COBOL/mainframes") are consistent with practitioner experience but **no authoritative statistic could be verified in this run**. The *qualitative* claim — mainframe COBOL cores still run the majority of large banks' systems of record — is well established (IBM COBOL documentation; the banking guides in this repository).
- **EIP adoption numbers** — no authoritative adoption statistic verified; the *qualitative* adoption claim (EIPs implemented by Camel, Spring Integration, Mule, and cloud integration platforms) is directly evidenced by the vendors' own documentation.

---

## 12. Glossary

- **Integration** — making independently built applications exchange information and work together as one system; the discipline of connecting systems, not merging them.
- **Legacy system** — an application that predates the current architecture and technology, is mission-critical, and is hard to change (aging languages, undocumented behavior, no modern interfaces).
- **Mainframe** — the large, centralized, batch-optimized computer class (z/OS, CICS/IMS) that runs most banks' core systems; the archetypal legacy platform.
- **COBOL** — COmmon Business-Oriented Language (1959); the English-like business data-processing language of the mainframe era, still the language of most core banking systems.
- **Silo** — a system whose data is trapped inside it, duplicated elsewhere, and reconcilable only by batch jobs and manual effort; the unit of the integration problem.
- **File transfer** — integration style: A exports a file, the file moves, B imports it (batch). See §2.2; production form: MFT (e.g., Axway Transfer CFT).
- **Shared database** — integration style: multiple applications read/write the same database. See §2.3.
- **RPC** — Remote Procedure Invocation; integration style where A synchronously calls a procedure on B (CORBA/DCOM/SOAP/REST/gRPC). See §2.4.
- **Messaging** — integration style where applications exchange messages via channels/brokers, asynchronously and reliably. See §2.5.
- **EIP / Enterprise Integration Patterns** — the catalogue of integration solutions (and their visual notation) from Hohpe & Woolf's 2003 book; the shared vocabulary of integration architecture.
- **Hohpe** — Gregor Hohpe, co-author of *Enterprise Integration Patterns* (2003); enterprise architect and integration thought leader.
- **Woolf** — Bobby Woolf, co-author of *Enterprise Integration Patterns* (2003); integration patterns authority.
- **Message channel** — the logical pipe between sender and receiver(s); queue (point-to-point) or topic (publish-subscribe). Key pattern §4.1.
- **Message router** — a component that directs each message to the right destination based on content or state (content-based router and relatives). Key pattern §4.2.
- **Message translator** — a component that converts a message from one system's format to another's. Key pattern §4.3.
- **Aggregator** — the pattern that combines related messages (by correlation identifier) into one, on a completion condition. Key pattern §4.4.
- **Splitter** — the pattern that splits one message into many for independent processing. Key pattern §4.5.
- **Content enricher** — the pattern that adds data to a message in transit from another source. Key pattern §4.6.
- **Strangler fig / strangler pattern** — Fowler's 2004 pattern for incremental replacement: wrap (facade), migrate function by function, retire the old system. §6.1.
- **ACL / anti-corruption layer** — the dedicated translation layer (Fowler, 2004; DDD) that isolates your domain model from a system whose model you can't change. §6.2.
- **CQRS** — Command Query Responsibility Segregation; separate models for updates (commands) and reads (queries), read models built from events. Young ~2010; Fowler bliki 2011 (see §11 flag). §7.2.
- **Outbox (transactional outbox)** — pattern solving the dual-write problem: write the event atomically with the state change; a relay publishes it to the broker. §7.3; [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10.
- **ESB / enterprise service bus** — the SOA-era middleware layer mediating between services: routing, transformation, orchestration, protocol conversion (Chappell, 2004). §8.1.
- **Message broker** — message-oriented middleware owning channels and delivery: queues, topics, persistence, retries (IBM MQ, RabbitMQ, Kafka, Pulsar). §8.2.
- **iPaaS** — integration-platform-as-a-service; cloud-delivered integration suites with connectors and low-code flows (Gartner, 2011). §8.3.
- **Camel** — Apache Camel, the open-source integration framework implementing 80+ EIPs as DSL constructs with 350+ connectors; the reference EIP implementation (see [camel_camelk_research.md](camel_camelk_research.md)).
- **Modernization** — evolving a legacy estate toward a modern architecture (event-driven, services, real-time) by integrating and incrementally replacing — not big-bang rewriting.

---

## 13. References

**Primary sources**

1. Gregor Hohpe & Bobby Woolf, *Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions*, Addison-Wesley, November 2003 (Addison-Wesley Signature Series).
2. Martin Fowler, "Strangler Fig Application", bliki, 2004 (martinfowler.com/bliki/StranglerFigApplication.html; updated 2024).
3. Martin Fowler, "AntiCorruptionLayer", bliki, 2004 (martinfowler.com/bliki/AntiCorruptionLayer.html) — date widely cited; see §11 flag.
4. Martin Fowler, "CQRS", bliki, 14 July 2011 (martinfowler.com/bliki/CQRS.html); pattern credited to Greg Young (~2010). See §11 flag.
5. David A. Chappell, *Enterprise Service Bus: Theory in Practice*, O'Reilly, 2004.
6. Massimo Pezzini & Benoit J. Lheureux, *Integration Platform as a Service: Moving Integration to the Cloud*, Gartner RAS Core Research Note G00210747, 7 March 2011.
7. Apache Camel documentation, "Enterprise Integration Patterns" (camel.apache.org).
8. Eric Evans, *Domain-Driven Design: Tackling Complexity in the Heart of Software*, Addison-Wesley, 2003 (bounded contexts; source of the ACL's DDD framing).

**Repository cross-references (this series)**

- [camel_camelk_research.md](camel_camelk_research.md) — Apache Camel & Camel-K, the EIP implementation (80+ EIPs, 350+ connectors, DSLs).
- [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) — the integration *frameworks* companion (CDC/ETL/API tooling; 0 EIP mentions).
- [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) — messaging reliability; delivery semantics; the transactional outbox (§10).
- [event_stream_processing_guide.md](event_stream_processing_guide.md) — event-driven processing; fault tolerance; at-least-once discipline.
- [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) — managed file transfer in production (the file-transfer style).
- [apache_seata_guide.md](apache_seata_guide.md) — distributed transactions and sagas; [temporal_workflow_guide.md](temporal_workflow_guide.md) — durable orchestration; [distributed_auth_guide.md](distributed_auth_guide.md) — identity across systems.
- [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md), [banking/tafj_guide.md](banking/tafj_guide.md), [banking/jbase_vs_infobasic_guide.md](banking/jbase_vs_infobasic_guide.md), [banking/temenos_guide.md](banking/temenos_guide.md), [banking/temenos_data_model_guide.md](banking/temenos_data_model_guide.md), [banking/oracle_flexcube_data_model_guide.md](banking/oracle_flexcube_data_model_guide.md), [banking/chinese_bank_core_systems_guide.md](banking/chinese_bank_core_systems_guide.md), [banking/apache_fineract_guide.md](banking/apache_fineract_guide.md), [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md), [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md) — the legacy cores being integrated.
- [banking/dbs_software_systems_guide.md](banking/dbs_software_systems_guide.md), [banking/standard_chartered_guide.md](banking/standard_chartered_guide.md) — real bank integration landscapes.
- [banking/posting_engine_core_banking_guide.md](banking/posting_engine_core_banking_guide.md), [banking/interest_engines_core_banking_guide.md](banking/interest_engines_core_banking_guide.md) — the core's internal mechanics that integration must wrap around (posting, interest, batch).

**Online references.** The Enterprise Integration Patterns website (eaipatterns.com — the patterns' canonical online home), Fowler's bliki (martinfowler.com/bliki — StranglerFigApplication, AntiCorruptionLayer, CQRS), Apache Camel EIP documentation (camel.apache.org), and the microservices.io pattern catalog (strangler fig, transactional outbox — cross-validates §6.1 and §7.3).

---

*End of guide — the final word: **integrate, don't replace.***
