# Why Tech Companies Bypass TOGAF (And What They Do Instead)

> **A critical analysis examining why FAANG-tier and modern tech companies reject formal Enterprise Architecture frameworks in favour of lightweight, engineering-driven approaches — and what bank architects can learn.**

> **Author:** Jack Liu Shurui  
> **Context:** Solution Architect, Crédit Agricole CIB, Singapore  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) — `technology/`  
> **Companion guides:** [`enterprise_architecture_guide.md`](../enterprise_architecture_guide.md) · [`togaf_guide.md`](../togaf_guide.md)

---

## Table of Contents

1. [The Premise](#1-the-premise)
2. [Why TOGAF Is Often Bypassed](#2-why-togaf-is-often-bypassed)
3. [What FAANG Companies Actually Do](#3-what-faang-companies-actually-do)
   - 3.1 Meta / Facebook
   - 3.2 Apple
   - 3.3 Amazon
   - 3.4 Netflix
   - 3.5 Google / Alphabet
4. [Common Patterns at Modern Tech Companies](#4-common-patterns-at-modern-tech-companies)
5. [When Formal EA Frameworks Do Make Sense](#5-when-formal-ea-frameworks-do-make-sense)
6. [Lessons for Bank Architects](#6-lessons-for-bank-architects)
7. [Conclusions](#7-conclusions)

---

## 1. The Premise

Many of the world's most valuable and influential technology companies — Meta, Apple, Amazon, Netflix, Google (the FAANG cohort), and fast-growing modern enterprises — do **not** use TOGAF or any other formal Enterprise Architecture framework as their primary approach to architectural governance. Yet these same companies consistently ship coherent, evolvable, highly scalable systems.

This apparent paradox reveals a fundamental truth: **TOGAF is one approach to achieving architectural coherence, but it is not the only one, and for many organisations it is not the best one.**

Tech companies achieve architectural consistency through a different set of mechanisms:

| Mechanism | What It Provides |
|---|---|
| **Strong engineering culture** | Shared values and norms that guide technical decisions without explicit governance |
| **Technical excellence standards** | Hiring bars, code review rigour, and engineering-level quality criteria |
| **Architectural principles** | A small set of strong, memorable rules that constrain choices |
| **Lightweight governance** | Code review, design docs, and lightweight RFC processes instead of architecture boards |
| **Platform teams** | Internal product teams that build paved-road platforms consumed as self-service |
| **API mandates** | Every service communicates only through well-defined, versioned APIs |
| **Internal developer platforms (IDPs)** | Abstraction layers that enforce architectural patterns in tooling and infrastructure |

The core argument of this guide is not that TOGAF is wrong — it is that TOGAF was designed for a different context (government, regulated industry, large enterprise IT) and that tech companies solve the **same problem** (architectural coherence at scale) through fundamentally different means rooted in engineering culture, platform thinking, and lightweight governance.

---

## 2. Why TOGAF Is Often Bypassed

Seven structural reasons explain why TOGAF rarely gains traction in technology-first organisations:

### 2.1 TOGAF Is Heavyweight

The TOGAF Architecture Development Method (ADM) comprises **10 phases** (Preliminary, A–H, Requirements Management), each with detailed inputs, steps, outputs, and deliverables. A full ADM cycle can take months. The documentation expectations — Architecture Vision, Business Architecture, Information Systems Architectures (Data + Application), Technology Architecture, Opportunities & Solutions, Migration Planning, Implementation Governance, Architecture Change Management — produce extensive artifacts that require dedicated architects to create and maintain.

For a tech company shipping code multiple times per day, a process that requires weeks or months to produce an architecture deliverable is simply incompatible with the pace of delivery.

### 2.2 TOGAF Assumes Centralized Control

TOGAF's governance model envisions architecture boards, architecture review committees, and centralised decision-making bodies that approve deviations from standards. This assumes a **top-down** control model.

Tech companies overwhelmingly favour **federated ownership** and **team autonomy**. Amazon's two-pizza teams, Spotify's squads, and Google's engineering teams all operate with substantial autonomy. A central architecture board that must approve every significant decision would be a bottleneck that destroys the velocity these organisations depend on.

### 2.3 TOGAF Is Business-IT Alignment Focused

TOGAF's foundational premise is aligning business and IT — treating them as separate domains that must be bridged. The ADM phases explicitly move from Business Architecture to Information Systems Architecture to Technology Architecture.

Technology companies **are** technology companies. There is no separation between "business" and "IT" — the business is the technology. Engineering decisions **are** business decisions. The entire premise of business-IT alignment collapses when the product is the code.

### 2.4 TOGAF's Pace Doesn't Match Modern Development

Modern software delivery operates on continuous integration / continuous deployment (CI/CD) cycles measured in minutes or hours. ADM phases are measured in weeks or months. Even a lightweight ADM iteration is too slow to inform the hundreds of architectural decisions embedded in a day's worth of production changes.

In a tech company, architecture decisions are made constantly — in every pull request, every API contract change, every dependency upgrade. TOGAF's periodic, batch-oriented approach cannot keep pace with continuous delivery.

### 2.5 TOGAF Artifacts Are Often Shelf-Ware

The detailed architecture documentation that TOGAF prescribes — catalogs, matrices, and diagrams — quickly becomes obsolete in a fast-moving environment. A system architecture diagram drawn six months ago may bear little resemblance to the production system today.

Tech companies solve this by making architecture **observable** (you can see what the system is doing through monitoring, tracing, and metrics) and **executable** (architecture is encoded in infrastructure-as-code, service contracts, and CI/CD pipelines). Documents that describe what should exist are inferior to code that **is** what exists.

### 2.6 TOGAF Certification Doesn't Guarantee Practical Skills

The TOGAF certification process tests knowledge of the TOGAF framework itself — its concepts, terminology, and process. It does not test the ability to design distributed systems, evaluate trade-offs, or ship production software.

Many practicing architects with TOGAF certifications have never built a production system or written code professionally. In engineering-driven organisations, credibility comes from having **built things that work at scale**, not from having passed a certification exam.

### 2.7 TOGAF's Origin in Government and Enterprise IT

TOGAF originated in the US Department of Defense (DoD TAFIM in the 1990s) and was developed for a context of government procurement, large-scale system integration, and compliance-heavy enterprise IT environments. The framework reflects the values of that world:

- **Documentation as evidence** — for audits, procurement, and compliance
- **Process as control** — to manage risk in large, slow-moving organisations
- **Standardisation as goal** — for interoperability across diverse systems and vendors
- **Separation of concerns** — business, data, application, and technology as distinct domains managed by distinct groups

These assumptions do not hold in a product engineering organisation where small teams own services end-to-end, deploy independently, and evolve their systems continuously.

---

## 3. What FAANG Companies Actually Do

Each FAANG company has developed its own approach to architectural coherence. While the specifics differ, common threads emerge: lightweight governance, strong engineering culture, platform thinking, and architecture encoded in code rather than documents.

### 3.1 Meta / Facebook

**Architecture through lightweight RFCs.** Meta's engineering culture has long used an internal Request for Comments (RFC) process for significant architecture decisions. Any engineer can write an RFC proposing an architectural change. The document is shared broadly, discussed in comments, and either accepted, rejected, or revised. This is strikingly similar to Architecture Decision Records (ADRs) in spirit, but predates the ADR movement.

**Engineering-driven architecture review.** Architecture review happens as part of code review. Meta's code review culture is legendary — every change is reviewed by peers who understand the system. Architectural concerns (does this break layering? does this introduce the wrong dependency?) are caught in the review process, not in a separate architecture board.

**Strong API and service ownership culture.** Every service has an identified owner (often an individual, sometimes a small team). The owner is responsible for the service's API, its contract, its evolution, and its deprecation schedule. This creates strong, decentralised ownership without centralised architecture governance.

**Internal developer platform.** Meta built extensive internal tooling — a monorepo with custom version control (Mercurial-based), the Buck build system, a continuous integration platform, and an internal service discovery framework. These tools **encode** architectural decisions (how code is structured, how dependencies are managed, how services find each other) so engineers don't need to think about them consciously.

**Architecture evolution through incremental refactoring.** Rather than designing a target architecture upfront, Meta evolves its architecture incrementally. Systems are continuously refactored, split, merged, and replaced. The famous "Move Fast and Fix Things" ethos means architectural changes are made as small, incremental steps rather than large transformation initiatives.

**Minimal centralized architecture board.** Meta does have senior engineers who think about architecture at scale, but there is no formal architecture board with approval authority. Architecture decisions that cross team boundaries are resolved through engineering debate, escalation to senior engineers, or the RFC process — not through a governance gate.

### 3.2 Apple

**Strong design culture as architectural force.** Apple's approach to architecture is inseparable from its approach to design. The same sensibility that produces beautiful, coherent products also produces coherent architecture. Design principles — simplicity, elegance, minimalism — apply equally to user interfaces and system architecture.

**Architecture driven by product vision.** Architectural decisions at Apple are driven by product goals, not by abstract architectural concerns. The decision to create Swift came from the need for a modern language for iOS/macOS development. Metal was born from the need for low-overhead graphics. ARKit exists because Apple saw augmented reality as strategically important. Architecture serves product strategy, not the other way around.

**Tight integration across hardware, software, and services.** Apple's vertical integration is a deliberate architectural strategy. By controlling the hardware (A-series and M-series chips), the operating system (iOS, macOS), the frameworks (Swift, Metal, Core ML), and the services (iCloud, Apple Pay), Apple can enforce architectural coherence at every layer. This is architecture as competitive moat.

**Centralized platform decisions.** While Apple gives teams substantial autonomy in implementation, key platform decisions — programming languages, rendering frameworks, machine learning APIs — are made centrally by platform leadership. Sheryl Sandberg-era Facebook and Tim Cook-era Apple both demonstrate that the pendulum between centralization and decentralization swings with organisational context.

**Architecture decisions made by product leadership.** At Apple, the most important architectural decisions are made by product leaders with deep technical backgrounds (and often, engineering co-founders or long-tenured executives). There is no separate architecture function that produces documents for business leaders to approve — the people making architecture decisions are the people shipping products.

### 3.3 Amazon

**Two-pizza teams (autonomous, decentralized).** Amazon's foundational organisational unit is the two-pizza team — a small team (6–10 people) that owns a service end-to-end and has complete autonomy over its design, implementation, deployment, and operation. This organisational structure is an architectural decision: small, autonomous teams naturally produce small, loosely coupled services.

**API-first culture.** At Amazon, all communication between teams happens through well-defined APIs. There is no shared database access, no direct calls to another team's code, no shared libraries that evolve out of sync. The API is the contract, and the contract must be respected. This was codified in Jeff Bezos's famous API mandate (circa 2002), which required all teams to expose their data and functionality through service interfaces and communicate only through those interfaces.

**"You build it, you run it" ownership model.** Amazon pioneered the model where the team that builds a service also operates it. This creates powerful incentives for architectural quality: teams that own their services in production make different decisions about reliability, observability, and operability than teams that throw designs over the wall to an operations group.

**Six-page narrative memos instead of architecture documents.** At Amazon, significant proposals are communicated as six-page narrative memos, not PowerPoint decks or templated architecture documents. The memo format forces clear thinking about context, problem, proposed solution, alternatives considered, and rationale. This is a lightweight alternative to TOGAF's extensive artifact suite.

**AWS services built with strict API contracts.** Every AWS service has a well-defined API — initially REST/JSON, later also supporting event-driven patterns. The API is the contract. Internal AWS teams follow the same discipline: internal APIs are designed with the same rigor as external customer-facing APIs.

**Architecture evolves through service decomposition.** Amazon's architecture has evolved over decades through continuous service decomposition. Monolithic systems are split into services, services are split into finer-grained services, and new capabilities are added as new services. This is evolutionary architecture in practice — the system's structure changes incrementally over time.

**CTO-level architecture principles.** Beyond the API mandate, Amazon has a set of strong architectural principles enforced from the top:
- All teams must expose data through service APIs (the API mandate).
- No direct database access across teams.
- Systems should be stateless where possible.
- Every service must be designed for failure (availability zone independence is the most visible expression).

These principles are few, strong, and enforced — a sharp contrast to TOGAF's hundreds of pages of standards and guidelines.

### 3.4 Netflix

**Freedom and responsibility culture.** Netflix's culture famously values "freedom and responsibility" over process and controls. This applies to architecture: teams have substantial freedom to make architectural decisions, but are expected to own the consequences. If a team's architectural choices cause production issues, the team fixes them — there is no central architecture team to blame or rescue them.

**Architecture guided by principles.** Netflix publishes its engineering culture and architectural principles publicly. The guiding principles include high availability, fault tolerance, resilience, and the ability to continuously deploy without downtime. These principles guide thousands of local decisions without requiring a central architecture authority.

**Chaos engineering as an architectural tool.** Netflix pioneered chaos engineering — intentionally injecting failures into production systems to test their resilience. Chaos Monkey randomly terminates production instances. Chaos Gorilla simulates an entire AWS region failure. These practices don't verify architecture against a document; they verify it against reality. This is a fundamentally different approach to architecture validation than TOGAF's compliance reviews.

**API gateway and microservices evolved organically.** Netflix's API gateway (Zuul) and microservices architecture were not designed upfront in an architecture workshop. They evolved organically in response to real scaling challenges. The architecture emerged from engineering decisions made under pressure, guided by principles, not from a phase of the ADM.

**Architecture documentation in runbooks and operational playbooks.** Netflix's most important architecture documentation isn't in architecture catalogs — it's in runbooks for incident response, playbooks for deployment, and operational dashboards. If you want to understand how a Netflix service works, you look at its monitoring dashboard, its deployment pipeline, its incident postmortems, and the runbook for operating it.

**Platform teams provide building blocks, not blueprints.** Netflix's platform teams build internal tools and platforms (the Netflix engineering tooling ecosystem for CI/CD, chaos testing, monitoring, etc.) but do not produce architecture blueprints for application teams to follow. The platform enforces certain architectural patterns (how services are packaged, deployed, and monitored) through tooling, not through documents or governance gates.

### 3.5 Google / Alphabet

**SRE culture for operational architecture.** Google created the Site Reliability Engineering discipline, which applies software engineering principles to operations. SRE enforces operational architecture — SLIs, SLOs, error budgets, incident response, capacity planning — through engineering practices, not through architecture documents. The SRE model ensures that reliability, scalability, and operational concerns are embedded in the architecture from the start.

**Monorepo with strict tooling.** Google famously operates a single monorepo containing most of its codebase. This is supported by custom tooling (Piper for version control, Blaze/Bazel for building) that enforces dependency management, visibility controls, and build correctness at scale. The monorepo is an architectural choice — it enables massive code reuse, consistent tooling, and system-wide refactoring. The tooling enforces architectural discipline (who can depend on what, how code is structured) without requiring an architecture governance board.

**Code review culture ensures architectural consistency.** Google's code review process is legendary. Every change must be reviewed by at least one qualified reviewer. Architectural concerns — is this the right approach? does this fit the system's design? are dependencies correct? — are raised and resolved during code review. This is engineering-driven governance in its purest form.

**Design docs as lightweight architecture decision records.** Google engineers write design docs for significant changes — typically 1–5 page documents describing the problem, proposed solution, alternatives, and trade-offs. These are shared, reviewed, revised, and ultimately approved or rejected by experienced engineers. Design docs are strikingly similar to ADRs but predate the formal ADR pattern.

**Architecture driven by platform teams.** Google's platform teams build the foundational infrastructure that encodes architectural decisions:
- **Borg / Kubernetes** — the cluster management system that defines how services are deployed, scaled, and managed
- **Spanner** — the globally distributed database that defines data consistency and distribution patterns
- **Bigtable** — the scalable NoSQL store for high-throughput workloads
- **gRPC** — the RPC framework that defines service communication patterns
- **Istio / service mesh** — the infrastructure layer for service-to-service communication, security, and observability

Each platform product encodes architectural decisions that application teams inherit automatically.

**Architecture review through senior engineer design reviews.** Significant architecture decisions at Google are reviewed by senior engineers (Staff+, Principal, Distinguished) who have deep technical expertise and broad system knowledge. This is not a governance board with formal voting rights — it's a technical review by peers with relevant expertise. The authority comes from technical credibility, not organisational position.

## 4. Common Patterns at Modern Tech Companies

Beyond the FAANG cohort, a set of common patterns has emerged across modern tech companies. These represent the de facto approach to architectural coherence in technology-driven organisations.

### 4.1 Architecture as Code

The most powerful shift in modern architecture practice is encoding architectural decisions in executable artifacts rather than documents:

- **Infrastructure-as-Code (IaC)** — Terraform, Pulumi, CDK, and CloudFormation define infrastructure architecture in code. Architectural decisions about networking, security, data storage, and service topology are expressed as code, versioned, reviewed, and deployed through CI/CD pipelines.
- **Service contracts** — Protobuf (gRPC), OpenAPI (REST), and AsyncAPI (event-driven) define service interfaces in machine-readable formats. These contracts are the source of truth for service architecture — they are versioned, validated, and can generate client libraries, documentation, and test harnesses automatically.
- **CI/CD pipelines** — Build, test, and deployment pipelines encode architectural standards (must pass security scans, must meet performance benchmarks, must deploy to staging before production) in automated gates rather than manual reviews.
- **Policy-as-Code** — Tools like Open Policy Agent (OPA), Kyverno, and Cedar enforce architectural policies (which services can talk to which databases, what cloud resources are allowed, what security configurations are required) programmatically.

When architecture is code, it is always up to date, always executable, and always verifiable. This is the polar opposite of shelf-ware architecture documents.

### 4.2 Architecture Decision Records (ADRs)

ADRs have become the de facto standard for capturing architecture decisions in tech companies. An ADR is a short document (typically 1–2 pages) capturing:

- **Context** — Why a decision is needed
- **Decision** — What was decided
- **Rationale** — Why this option was chosen
- **Consequences** — What trade-offs this creates
- **Status** — Proposed, accepted, deprecated, superseded

ADRs are **lightweight** (no template bureaucracy), **version-controlled** (stored alongside code in the repository), **context-capturing** (preserve the reasoning for future engineers), and **mutable** (status changes as the system evolves).

ADRs replace most of what TOGAF's architecture documentation suite provides (architecture principles, standards, guidelines, decisions) with an order of magnitude less overhead. They have been widely adopted across companies including ThoughtWorks, Zalando, AWS, and thousands of others.

### 4.3 Internal Developer Platforms (IDPs)

IDPs provide "paved roads" — well-supported, self-service platforms that encode architectural best practices and reduce cognitive load for application teams. The platform team builds and maintains the platform; application teams consume it as a product.

Characteristics of effective IDPs:
- **Self-service provisioning** — teams spin up services, databases, and infrastructure without waiting for a central team
- **Architecture guardrails** — the platform enforces architectural standards (which services can be created, how they connect, what observability is required) through tooling, not through manual review
- **Abstraction of complexity** — teams don't need to understand the underlying infrastructure (Kubernetes, networking, security policies) to build and deploy services
- **Product orientation** — the platform team treats application teams as customers, with roadmaps, SLAs, feedback cycles, and product management

Spotify's Backstage (now a CNCF project) has become the most widely adopted IDP framework, but every major tech company has built internal platforms of varying sophistication.

### 4.4 Engineering-Driven Governance

Instead of separate architecture review boards and governance committees, modern tech companies integrate architecture governance into existing engineering workflows:

- **Code review** is the primary mechanism for architecture review. Every change is reviewed by peers who understand architectural implications.
- **Design reviews** are conducted by senior engineers for significant changes, not by a separate architecture function.
- **Architecture standards** are enforced through tooling (linters, static analysis, CI/CD gates), not through manual compliance checks.
- **Post-incident reviews** drive architectural improvements through real-world failure analysis, not through periodic architecture health checks.

This approach scales because governance cost is proportional to change frequency — every change gets appropriate scrutiny, and the scrutiny is provided by people with relevant technical expertise.

### 4.5 Principles Over Processes

Tech companies tend to rely on a small set of strong, memorable architectural principles rather than extensive standards documents. Effective principles share characteristics:

- **Few in number** — typically 5–12 principles, not 50+
- **Strong and opinionated** — they actually constrain choices (e.g., "all communication via APIs" is a strong principle; "we value interoperability" is weak)
- **Memorable** — engineers can recite them without referring to a document
- **Enforceable** — it's clear whether a decision conforms to the principle
- **Stable but not immutable** — principles change, but infrequently

Examples of effective principles from tech companies:
- Amazon: "All teams must expose data through service APIs"
- Netflix: "Architect for failure"
- Google: "Every service must have SLOs and error budgets"
- ThoughtWorks: "Lift and shift is not a strategy"

### 4.6 Evolutionary Architecture

Evolutionary architecture treats architecture as something that evolves incrementally over time, not something designed upfront. Key characteristics:

- **Architecture decisions are made in context** — the right architecture for a system depends on what the team has learned so far
- **Incremental change is the norm** — systems are refactored, split, and evolved continuously
- **Reversibility is valued** — architectural choices that are hard to undo (database selection, service boundaries) are approached carefully; choices that are easy to undo (framework selection, library choice) are made more freely
- **Fitness functions** — automated tests and verification mechanisms that measure architectural characteristics (performance, scalability, resilience) and alert when they degrade

This contrasts sharply with TOGAF's target architecture / transition plan model, which envisions architecture as a series of well-defined states with planned transitions between them.

### 4.7 Team Topologies

The Team Topologies framework (Matthew Skelton & Manuel Pais) has become influential in modern tech companies as an alternative approach to architectural governance:

- **Stream-aligned teams** — teams aligned to a single flow of work (a product, a service, a feature area); they own their systems end-to-end and have the autonomy to evolve them
- **Enabling teams** — teams that help stream-aligned teams build capabilities (specialised skills, new tools, new practices)
- **Complicated-subsystem teams** — teams that own a particularly complex domain (recommendation engines, search indexing, payment systems) that requires deep expertise
- **Platform teams** — teams that build internal platforms consumed by stream-aligned teams

The Team Topologies model matches how FAANG companies actually organise — a far cry from the centralised architecture team model that TOGAF assumes.

### 4.8 API-First and Contract-First Development

Modern tech companies treat API contracts as a primary design artifact, not a documentation afterthought:

- **OpenAPI / Swagger** — for REST APIs, the specification is the source of truth, driving documentation, client generation, testing, and mock servers
- **gRPC / Protobuf** — for RPC-based services, the `.proto` file defines the contract and generates code for multiple languages
- **AsyncAPI** — for event-driven architectures, the specification defines channels, message schemas, and bindings
- **GraphQL schemas** — for API layers that aggregate data from multiple services

Contracts are versioned, stored in source control, reviewed as part of the change process, and used to validate implementations. This approach replaces TOGAF's Information Systems Architecture deliverables (data models, interface catalogs, system interaction matrices) with executable artifacts that are always current.

### 4.9 Observable Infrastructure

Modern tech companies invest heavily in observability — logging, metrics, tracing, and monitoring — to understand how systems actually behave in production. This observability replaces much of what architecture documentation was traditionally used for:

- **Service topology** is visible through distributed tracing (Jaeger, Zipkin, Honeycomb)
- **Dependencies** are visible through service maps (Datadog, New Relic, Lightstep)
- **Architecture changes** are visible through deployment dashboards and change logs
- **System behaviour** is visible through metrics dashboards and alerting
- **Capacity and scaling** are visible through monitoring data, not capacity planning documents

When you can see what the system is actually doing in real time, the value of a static architecture diagram depreciates dramatically.

### 4.10 Post-Incident Reviews as Architecture Improvement Mechanism

Instead of periodic architecture reviews (quarterly architecture board reviews, annual architecture health checks), tech companies drive architectural improvements through post-incident reviews:

- Every significant incident produces a postmortem (exceptionally, a blameless one)
- Root causes often reveal architectural issues — missing circuit breakers, single points of failure, insufficient test coverage, tight coupling between services
- Corrective actions address the architectural root cause directly
- This creates a continuous feedback loop from production reality to architectural improvement

This pattern is far more effective than periodic architecture reviews because the improvements are driven by actual production failures that have real business impact.

---

## 5. When Formal EA Frameworks Do Make Sense

Despite the strong critique above, formal EA frameworks like TOGAF remain valuable — and sometimes essential — in specific contexts.

### 5.1 Regulated Industries

Banking, healthcare, insurance, and other heavily regulated industries need documented architecture for regulatory compliance:

- **BCBS 239** (Basel Committee on Banking Supervision) — requires banks to demonstrate data lineage, risk data aggregation, and risk reporting architecture
- **SOX** (Sarbanes-Oxley Act) — requires documented IT controls and audit trails
- **PCI-DSS** — requires documented security architecture for payment card data
- **HIPAA** — requires documented privacy and security architecture for healthcare data
- **MAS guidelines** (Monetary Authority of Singapore) — require documented technology risk management, business continuity, and outsourcing architecture

In these contexts, architecture documentation is not shelf-ware — it is a regulatory deliverable that must exist, be maintained, and be producible for audits and examinations.

### 5.2 Large-Scale Transformation and M&A Integration

When an organisation is undergoing fundamental transformation (legacy modernisation, platform migration, operating model redesign) or integrating a major acquisition, a structured EA approach can provide:

- A current-state baseline for understanding what exists
- A target-state architecture for aligning efforts toward a common goal
- A transition plan for sequencing work and managing dependencies
- A governance mechanism for managing scope, risk, and investment

The structured approach of the ADM can bring coherence to situations where the absence of architecture governance would lead to chaos and wasted investment.

### 5.3 Government and Public Sector

Government procurement, compliance, and risk management requirements often mandate formal EA approaches. Many government agencies require EA artifacts as part of procurement, and TOGAF is frequently the framework of record for public sector architecture.

The culture of government IT — hierarchical, process-oriented, risk-averse, budget-constrained — aligns more naturally with TOGAF's structured approach than with tech company patterns.

### 5.4 Organisations with Low Engineering Maturity

Not every organisation has the engineering culture, talent, or practices to operate like a tech company. For organisations with:

- Low engineering maturity (weak testing practices, poor deployment discipline, inconsistent coding standards)
- Heavy reliance on vendors and system integrators
- Limited in-house technical leadership
- High turnover in technical roles

...a formal EA framework provides needed structure, standards, and governance that would otherwise be absent. The scaffolding of TOGAF's ADM can guide organisations toward better practices even if it never achieves the ideal of full EA maturity.

### 5.5 Using TOGAF Well (When You Must Use It)

When TOGAF is used in technology companies or in regulated industries with tech company patterns, the most effective approach is a **tailored** one:

- **Not full ADM** — use the phases and artifacts that add value; skip the ones that don't
- **Strategic, not operational** — use TOGAF for strategic planning (which markets to enter, what capabilities to build, what platforms to invest in) not for day-to-day service architecture decisions
- **Compliance-focused** — use TOGAF to produce the architecture documentation that regulators require; use tech company patterns for everything else
- **Reference model, not prescriptive process** — use TOGAF as a reference for the kinds of things you might want to think about, not as a step-by-step process to follow
- **Combined with lightweight practices** — use ADRs for decision capture, principles-based governance for speed, and platform teams for architecture guardrails; overlay TOGAF governance only where regulatory or strategic requirements demand it

### 5.6 The Hybrid Approach

The most pragmatic approach for forward-thinking architecture organisations is a **hybrid**:

| Domain | Approach |
|---|---|
| **Strategic architecture** | TOGAF ADM (simplified) — capability planning, investment decisions, target-state vision |
| **Compliance architecture** | TOGAF artifacts — documentation for regulatory requirements, audit trails, procurement |
| **Delivery architecture** | Tech company patterns — ADRs, code review, platform teams, principles-based governance |
| **Decision capture** | ADRs — lightweight, version-controlled, context-capturing |
| **Architecture guardrails** | Platform teams — IDPs, service contracts, IaC, CI/CD policy-as-code |
| **Governance** | Principles-based — small set of strong principles enforced through tooling and code review |
| **Operations** | SRE practices — SLIs/SLOs, error budgets, incidents, postmortems |

This hybrid approach recognises that different architectural activities require different levels of ceremony, formality, and governance, and applies each approach where it fits best.

---

## 6. Lessons for Bank Architects

For a bank architect at an institution like Crédit Agricole CIB, the question is not "should we use TOGAF or not?" but rather "what can we learn from tech companies while meeting our regulatory and operational obligations?"

### 6.1 Banking Adoption of Tech Company Patterns

Many major banks are already adopting tech company patterns, often in their digital and cloud-native domains:

- **API-first architecture** — Open Banking / PSD2, BIAN-compliant banking APIs, internal API marketplaces
- **Platform teams** — cloud platforms (AWS Landing Zone, Azure CAF), API management platforms, data platforms (data lakehouses, event streaming), developer platforms (Backstage-based IDPs)
- **ADRs for decision capture** — growing adoption of ADRs alongside or instead of traditional architecture templates
- **SRE practices** — SLIs and SLOs for critical banking systems, error budgets for release velocity, blameless postmortems
- **Internal developer platforms** — banks including Goldman Sachs (Legend), JPMorgan Chase, ING, and Deutsche Bank are investing in IDP-like capabilities

### 6.2 What Banks Can Learn

**Architecture as code over documents.** Banks are document-heavy by nature (policy documents, risk frameworks, compliance artifacts). Adopting architecture-as-code practices (IaC, service contracts, policy-as-code) in parallel with traditional documentation reduces the gap between architecture intent and production reality.

**Lightweight governance over heavy boards.** Banks rely heavily on architecture review boards, design authorities, and governance forums. These can be supplemented — and in some areas replaced — by code review, design reviews, and CI/CD gates that enforce architectural standards without slowing down delivery.

**Evolutionary architecture over big design upfront.** Bank architecture has traditionally been driven by multi-year programmes with detailed target architectures. Adopting evolutionary architecture patterns — incremental change, fitness functions, reversible decisions — can accelerate delivery while maintaining architectural quality.

**Platform teams over centralised architecture teams.** Instead of a central architecture team producing blueprints that delivery teams implement (poorly), platform teams can build infrastructure and development platforms that encode architectural standards and provide self-service capabilities to delivery teams.

**Principles over prescriptive standards.** Instead of hundreds of pages of architecture standards that few have read and fewer follow, a small set of strong architectural principles — aligned with both business strategy and regulatory requirements — can guide thousands of local decisions more effectively.

### 6.3 What Banks Can't Ignore

At the same time, banks face constraints that pure tech companies don't:

**Regulatory compliance documentation is non-negotiable.** BCBS 239, MAS Technology Risk Management (TRM), HKMA Supervisory Policy Manual, and other regulations require specific architecture documentation. These cannot be replaced by ADRs and IaC alone — but they can be generated from code artifacts where possible and supplemented with lightweight documents.

**Vendor management and procurement processes are real.** Banks must manage multiple vendors with standardised processes, contracts, and compliance requirements. This creates inherent complexity that tech companies (which tend to build rather than buy) don't face to the same degree.

**Legacy system integration complexity is daunting.** Banks run decades-old mainframes, custom core banking platforms, and deeply embedded legacy systems. The architecture challenge is not designing greenfield systems but evolving existing ones — a fundamentally harder problem.

**Organisational change management is slower in banking.** Banks have more layers, more committees, more risk functions, and more stakeholders. Culture change happens at a different pace, and initiatives that would take months in a tech company can take years in a bank.

**Risk-averse culture limits experimentation.** Tech companies embrace failure as a learning mechanism. Banks cannot afford to fail in the same way — a production outage, a data breach, or a compliance failure has existential consequences. This is not a flaw to be fixed; it is a constraint to be managed. Architecture approaches must respect risk appetite, not fight it.

### 6.4 Practical Guidance for Bank Architects

For a bank architect navigating these tensions:

1. **Use TOGAF for what it's good at** — regulatory compliance, strategic planning, architecture documentation that auditors and regulators need. Don't throw it out; use it where it adds value.

2. **Adopt ADRs for internal decision capture** — replace heavy architecture templates with ADRs for day-to-day decision-making. Store them in version control alongside your architecture documentation.

3. **Build platform teams** — start with cloud platform and API platform teams. Let them build paved roads that encode architectural standards in tooling, not just in documents.

4. **Invest in architecture-as-code** — Terraform, OpenAPI, CI/CD pipelines, and policy-as-code are investments that pay dividends in both speed and compliance.

5. **Adopt principles-based governance** — define 5–10 strong architectural principles for your domain, get them endorsed by senior leadership, and enforce them through tooling and review practices.

6. **Integrate architecture into engineering workflows** — architecture review happens in code review and design review, not in separate governance forums. Architecture decisions are documented as ADRs alongside code changes.

7. **Use observability to validate architecture** — monitoring, tracing, and logging provide real-time feedback on whether your architecture is working as intended, complementing (not replacing) compliance documentation.

8. **Drive architecture improvement through incidents** — invest in blameless postmortem culture. Every significant incident is an opportunity to improve architecture, not just to fix a bug.

---

## 7. Conclusions

### 7.1 TOGAF Is Not Wrong — It's Designed for a Different Context

TOGAF is a product of its origins: government and large enterprise IT in the 1990s and early 2000s. In that context — slow-moving, hierarchical, compliance-focused, procurement-heavy — the framework's structured phases, extensive documentation, and centralised governance model make sense.

The mistake is applying it uncritically to contexts for which it was never designed — fast-moving product engineering organisations where teams are autonomous, deployment is continuous, and the business is the technology.

### 7.2 Tech Companies Solve the Same Problem Through Different Means

Architectural coherence at scale is a real problem, and tech companies solve it — just not through TOGAF. They use:

- **Culture** — engineering norms, design values, shared principles
- **Platforms** — internal developer platforms, infrastructure-as-code, service meshes
- **Code** — service contracts, CI/CD pipelines, policy-as-code, ADRs
- **Lightweight governance** — code review, design docs, post-incident reviews, engineering-driven decision-making

These mechanisms achieve the same goals (consistency, standardisation, quality, evolvability) without the overhead and rigidity of formal frameworks.

### 7.3 The Best Approach Is Context-Dependent

There is no single right approach to enterprise architecture. The best approach depends on:

- **Organisational culture** — are teams autonomous or centrally controlled? Is risk embraced or avoided?
- **Engineering maturity** — do teams have strong testing, CI/CD, and operational practices?
- **Regulatory environment** — what documentation and compliance obligations exist?
- **Organisational scale** — how many teams, systems, and dependencies exist?
- **Business context** — is technology the product or an enabler of the product?

The answer for a startup (no regulation, small teams, greenfield systems) will be different from the answer for a global bank (heavy regulation, large teams, legacy systems), which will be different from the answer for FAANG (technology product, massive scale, strong engineering culture).

### 7.4 For Bank Architects: Combine TOGAF for Compliance with Tech Patterns for Delivery

The hybrid approach is not a compromise — it is a strategically superior model:

- **TOGAF for compliance and strategy** — regulatory documentation, target-state architecture, capability planning, investment governance
- **Tech company patterns for delivery and platforms** — ADRs, platform teams, architecture-as-code, principles-based governance, code review as architecture review, evolutionary architecture, observability

This gives banks the regulatory compliance they need and the engineering velocity they want.

### 7.5 The Future of EA Is Hybrid

The future of Enterprise Architecture will be a synthesis of multiple traditions:

| Source | Contribution |
|---|---|
| **EA frameworks (TOGAF, Zachman)** | Strategic planning, compliance governance, structured viewpoints, capability-based planning |
| **Tech company engineering culture** | Code review governance, lightweight decision capture (ADRs), evolutionary architecture, platform thinking |
| **Product engineering** | Platform teams as product teams, developer experience as a first-class concern, self-service as a design goal |
| **SRE and operations** | Observability-driven architecture, error budgets, chaos engineering, post-incident improvement |
| **Team Topologies** | Organisational design as an architectural concern, team interaction patterns as governance mechanisms |

Enterprise Architecture as a discipline is evolving from a document-centric, process-heavy, board-governed practice toward a more integrated, engineering-driven, platform-enabled discipline. The architects who thrive in this environment will be those who can operate in both worlds — comfortable with formal frameworks when regulators require them and with engineering practices when delivery demands them.

---

## References and Further Reading

- **Enterprise Architecture as Strategy** — Ross, Weill & Robertson (Harvard Business Review Press)
- **Building Evolutionary Architectures** — Ford, Parsons & Kua (O'Reilly)
- **Team Topologies** — Skelton & Pais (IT Revolution Press)
- **Accelerate** — Forsgren, Humble & Kim (IT Revolution Press)
- **Site Reliability Engineering** — Beyer, Jones, Petoff & Murphy (O'Reilly)
- **The Culture Map** — Erin Meyer (PublicAffairs) — for understanding cross-cultural organisational differences relevant to EA adoption
- **ThoughtWorks Technology Radar** — [thoughtworks.com/radar](https://www.thoughtworks.com/radar) — for tracking ADR, platform, and evolutionary architecture adoption
- **Backstage (CNCF)** — [backstage.io](https://backstage.io) — the open-source IDP framework

---

> **About the Author**  
> Jack Liu Shurui is a Solution Architect at Crédit Agricole CIB in Singapore, working at the intersection of enterprise architecture, regulatory compliance, and modern engineering practices. This guide is part of a series examining architecture frameworks and their practical application in banking contexts.  
>  
> *"The best architecture framework is the one that produces good decisions, not the one that produces the most documents."*
