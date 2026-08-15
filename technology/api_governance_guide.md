# API Governance: The Enterprise Discipline of Governing APIs — A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Crédit Agricole CIB, Singapore
> **Context:** Technology / Enterprise Architecture — Platform Engineering Series — API Governance, API Standards, API Lifecycle, API Registry, API Gateway, Open Banking
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Path:** `technology/api_governance_guide.md` | **Version:** 1.0 — August 2026
> **Reading time:** ~45 minutes

> **What this guide is.** This is the dedicated deep-dive on **API governance** — the enterprise discipline of governing APIs across the organization: the definition and framework, the standards (REST, OpenAPI, HTTP, error handling, versioning, pagination, idempotency), the design guidelines (Microsoft, Google, Zalando), the lifecycle and its gates, the governance processes (design review, contract-first, change management, compliance), the API registry/catalog, the gateway and runtime governance, security and observability, the roles and operating model, and the banking context (PSD2, CMA, MAS) with a worked governance operating model for a bank.

> **Companion guides in this series (cross-references):** [Enterprise Architecture](architecture/enterprise_architecture_guide.md) · [TOGAF](architecture/togaf_guide.md) · [BIAN Standard](architecture/bian_standard_guide.md) · [Data Integration Frameworks](data_integration_frameworks_guide.md) · [Event Stream Processing](event_stream_processing_guide.md) · [Cloud Providers](cloud_providers_guide.md) · [Spec-Driven Development](spec_driven_development_frameworks_guide.md) · [Mock Servers](mock_server_guide.md) · [Enterprise AI Gateway](enterprise_ai_gateway_guide.md) · [Kargo GitOps](kargo_gitops_guide.md) · [Data Governance](data_governance_guide.md) · [MCP Framework & Tools](ai_llm/mcp_framework_tools_guide.md) · [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) · Banking: [DBS Software Systems](../banking/dbs_software_systems_guide.md) · [Oracle Banking Microservices Architecture](../banking/oracle_banking_microservices_architecture_guide.md) · [Oracle FLEXCUBE Data Model](../banking/oracle_flexcube_data_model_guide.md)

> **Verification convention used throughout:** ✅ = verified in this research pass (primary/secondary sources); ⚠ = flagged (inferred, approximate, single-source, or no single industry standard exists); unmarked = structural/industry knowledge presented as such. The consolidated [Claims-Status table is in §11](#11-verification-and-claims-status).

---

## Table of Contents

1. [API Governance: Definition and Framework](#1-api-governance-definition-and-framework)
2. [API Standards](#2-api-standards)
3. [API Design Guidelines](#3-api-design-guidelines)
4. [API Lifecycle](#4-api-lifecycle)
5. [API Governance Processes](#5-api-governance-processes)
6. [API Registry and Catalog](#6-api-registry-and-catalog)
7. [API Gateway and Runtime Governance](#7-api-gateway-and-runtime-governance)
8. [API Security and Observability](#8-api-security-and-observability)
9. [Governance Roles and Operating Model](#9-governance-roles-and-operating-model)
10. [Banking Context and Worked Example](#10-banking-context-and-worked-example)
11. [Verification and Claims-Status](#11-verification-and-claims-status)
12. [Glossary](#12-glossary)

---

## 1. API Governance: Definition and Framework

### 1.1 What API Governance Is

**API governance** is the discipline of applying **standards, processes, and controls** across the entire API lifecycle — from design and development through deployment, operation, consumption, and retirement — so that an organization's APIs are consistent, secure, discoverable, reusable, and compliant. It is the answer to a specific failure mode: the "API wild west," where every team designs its own URL scheme, error format, authentication approach, and versioning strategy, producing an unmanageable sprawl of thousands of inconsistent endpoints.

Governance is *decision-making about APIs* made explicit and repeatable:

- **Standards** — the technical norms every API must conform to (REST conventions, OpenAPI contracts, error formats, versioning, security requirements).
- **Processes** — the workflow gates an API passes through (design review, security review, breaking-change approval, compliance checks).
- **Controls** — the enforcement mechanisms (linting in CI, gateway policies, registry metadata, audits) that make standards binding rather than aspirational.

A governed organization does not merely *have* APIs; it treats each API as a **product with an owner, a contract, a lifecycle, and an audience** — and it operates a machinery (review boards, catalogs, gateways, linting pipelines) that keeps hundreds of APIs coherent at scale.

### 1.2 Why Govern: The Rationale

| Driver | What governance delivers |
|---|---|
| **Consistency** | One URL style, one error format, one versioning scheme — lower cognitive load for consumers, cheaper onboarding, fewer integration defects |
| **Security** | Mandatory authentication/authorization patterns, OWASP-aligned design checks, gateway-enforced policies, fewer "custom security" surprises |
| **Discoverability** | A registry/catalog where consumers find APIs, owners, versions, and docs — prevents duplicate APIs being built for the same capability |
| **Reuse** | Standardized, findable APIs get consumed instead of re-implemented — reduces duplication and integration cost across domains |
| **Compliance** | Auditability of who changed what, when; demonstrable conformance for regulators (banking: PSD2/CMA obligations, MAS guidance, internal risk frameworks) |
| **Efficiency & DX** | Fewer breaking changes, clearer deprecation timelines, better developer experience for both internal and external consumers |

Without governance, the five drivers above degrade in a predictable order: inconsistency first (every API looks different), then security drift (each team re-invents authn), then duplication (nobody can find the existing API), and finally compliance exposure (no audit trail, no conformance evidence). ✅ The failure mode is well documented in industry practitioner literature; the *specific* sequence above is a structural inference (⚠) rather than a measured law.

### 1.3 API Governance vs. API Management

The two terms are frequently conflated; the distinction matters for organization design.

| | **API Management** | **API Governance** |
|---|---|---|
| Layer | Runtime / operational | Decision / policy |
| Concern | Gateway, rate limiting, quotas, auth enforcement, routing, transformations, analytics, developer portal | Standards, policies, design review, contract compliance, lifecycle policy, registry quality |
| Question it answers | *"How is this API running?"* | *"Should this API exist, and does it conform?"* |
| Horizon | Minutes to months (runtime operations) | Months to years (portfolio and lifecycle) |
| Typical owners | Platform/API platform team | API guild, stewards, review board, CoE |
| Tools | Apigee, Kong, AWS API Gateway, Azure APIM, MuleSoft | Spectral/Zally linting, OpenAPI contracts, Backstage catalog, review boards, ADRs |

The practical relationship: **management enforces governance at runtime.** The gateway is where governance policies (authn requirements, rate limits, quota tiers) become mechanical controls; the CI linting pipeline is where design standards become mechanical checks. Governance decides the policy; management executes it. A common failure is buying an API management platform and assuming governance follows — it does not; the standards, processes, and roles must be deliberately built (⚠ structural observation, widely repeated in practitioner guidance).

### 1.4 The Governance Scope

Governance spans the full API continuum:

- **Design** — contract style, naming, error handling, versioning strategy (governed pre-code).
- **Lifecycle** — stage gates from proposal to sunset; who approves what at each stage.
- **Security** — authn/authz standards, data classification, OWASP alignment, secrets handling.
- **Registry** — what gets registered, what metadata is mandatory (owner, status, SLA).
- **Consumption** — who may consume (internal/partner/external), onboarding, quotas, support tiers.
- **Retirement** — deprecation policy, sunset notices, consumer migration.

All six are in scope of a governance framework; many organizations govern only design and security, leaving registry and retirement ad hoc — the two areas that most often collapse into API sprawl (⚠ inference).

### 1.5 The Governance Framework: Five Pillars

A practical framework organizes governance into **five pillars** (a practitioner synthesis — ⚠ there is no single ISO-style standard named "API governance"; the pillars are the recurring elements across practitioner frameworks and vendor methodologies):

1. **Principles** — the enduring values: API-first, consistency over cleverness, additive over breaking change, secure by default, products not projects.
2. **Standards** — the binding technical norms (Section 2) and the style guide (Section 3).
3. **Processes** — the workflow gates: design review, security review, breaking-change review, compliance (Section 5).
4. **Roles** — ownership and accountability: API owner, API steward, API guild, Center of Enablement (Section 9).
5. **Tools** — the enforcement machinery: contract tooling, linters, registry, gateway policies, observability (Sections 6–8).

### 1.6 The Framework Table

| Pillar | Scope | Key artifacts |
|---|---|---|
| **Principles** | Values that guide every decision | Governance charter, API-first statement, decision records (ADRs) |
| **Standards** | Technical norms every API must meet | OpenAPI 3.1 contracts, style guide, error/versioning/pagination/idempotency standards, security standard |
| **Processes** | Gates and workflows | Design-review process, security review checklist, breaking-change approval flow, deprecation/sunset process, CI compliance pipeline |
| **Roles** | Ownership and accountability | API owner, API steward, API guild charter, Center of Enablement mandate, RACI matrix |
| **Tools** | Enforcement and transparency | Spectral rulesets, CI linting, Backstage catalog, gateway policy templates, observability dashboards, audit logs |

The pillars reinforce each other: principles justify standards, standards feed processes, processes need roles to run them, and tools mechanize everything. A governance framework missing any pillar leaks — e.g., standards without tools are unenforced; processes without roles are nobody's job.

---

## 2. API Standards

Standards are the backbone of governance: they are the *measurable* part. This section covers the technical standards an enterprise API program should adopt and enforce.

### 2.1 REST and the Richardson Maturity Model

**REST** (Representational State Transfer, Roy Fielding's 2000 dissertation) is the dominant architectural style for enterprise APIs: resources identified by URIs, manipulated via HTTP methods, with stateless interactions and self-descriptive messages. RESTful design is the default starting point for API governance standards because it is widely understood, toolable, and HTTP-native.

The **Richardson Maturity Model** (RMM) — suggested by **Leonard Richardson in 2008** and popularized by **Martin Fowler's 2010 article** ("Richardson Maturity Model: steps toward the glory of REST") ✅ — classifies APIs into four levels of REST maturity:

| Level | Name | What it means | Example |
|---|---|---|---|
| **0** | The Swamp of POX | HTTP as a tunnel; single endpoint, RPC-style; everything is POST | `POST /api` with `{action: "getCustomer", id: 42}` |
| **1** | Resources | Individual resources identified by URI; verbs still tunnelled | `POST /customer/42` with `{action: "get"}` |
| **2** | HTTP Verbs | Proper use of methods (GET/POST/PUT/PATCH/DELETE) and status codes | `GET /customers/42`, `DELETE /customers/42`, `404` when missing |
| **3** | Hypermedia (HATEOAS) | Responses carry links to related actions; client navigates | `GET /orders/7` returns links to `cancel`, `invoice`, etc. |

Most enterprise APIs in production sit at **Level 2** — resource-oriented, method-correct, status-code-correct. Level 3 (hypermedia) is rare in practice outside specialized domains; governance standards should *target* Level 2 and treat Level 3 as optional rather than mandatory (⚠ structural observation: Fowler's own article notes most APIs reasonably stop at Level 2, and few ecosystems are built around hypermedia).

### 2.2 OpenAPI (OAS) — The Contract Standard

**OpenAPI** is the de-facto industry standard for describing HTTP APIs as machine-readable contracts (JSON/YAML). History: Swagger 2.0 → donated to the OpenAPI Initiative (Linux Foundation) → **OAS 3.0 (2017)** → **OAS 3.1.0 (August 2021)** → **OAS 3.1.1 (February 2024, the current patch release)** ✅. OpenAPI 3.1 is the version to adopt for new programs because:

- **JSON Schema 2020-12 alignment** — full JSON Schema support (nullable, exclusiveMinimum as number, `$defs`, type unions), eliminating the 3.0 awkwardness (`nullable: true`, `type` arrays).
- **Webhooks** — first-class `webhooks` section for event-driven APIs (complements [event_stream_processing_guide.md](event_stream_processing_guide.md)).
- **`license.identifier`**, **`summary`** fields, and cleaner component reuse.

The OpenAPI document is the **single source of truth** in a governed program: it drives mock servers, code generation, tests, docs, registry metadata, and gateway configuration. Governance rule of thumb: *no contract, no API* — a service that cannot produce an OpenAPI document is not an API yet (⚠ practitioner norm, not an official standard).

### 2.3 HTTP Standards

Governance standards should pin down the HTTP layer so all APIs behave alike:

- **Methods**: GET (read, safe, idempotent), POST (create/action, non-idempotent), PUT (replace, idempotent), PATCH (partial update), DELETE (remove, idempotent). Prohibit method overloading and GET-with-body.
- **Status codes**: use the standard families correctly — 200/201/202/204 for success (201 + Location for creation, 202 for async acceptance), 400/401/403/404/405/409/410/422/429 for client errors, 5xx for server errors. Define exactly which codes each operation may return.
- **Headers**: `Content-Type`/`Accept` (with media types), `Cache-Control`/`ETag` (caching and optimistic concurrency), `Idempotency-Key` (Section 2.7), `Link` (pagination, Section 2.6), `Retry-After` (rate limiting), `Deprecation`/`Sunset` (Section 4.4).

### 2.4 Error Handling: RFC 7807 → RFC 9457 (Problem Details)

Consistent error payloads are one of the highest-value governance rules: they let consumers write one error-handling path for every API.

**RFC 7807** ("Problem Details for HTTP APIs", March 2016) defined a standard JSON error format; it was **obsoleted by RFC 9457 (April 2023)**, which carries the format forward with clarifications ✅. The standard members:

| Member | Meaning |
|---|---|
| `type` | URI identifying the problem type (may be a docs page) |
| `title` | Short, human-readable summary |
| `status` | HTTP status code |
| `detail` | Human-readable, situation-specific explanation |
| `instance` | URI identifying the specific occurrence (trace/log link) |

Served as `application/problem+json`. Example:

```json
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "type": "https://api.example.com/problems/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 403,
  "detail": "Account 1234-5678 has balance 12.50; transfer requires 100.00.",
  "instance": "/transfers/8a1f...",
  "currency": "SGD",
  "required_amount": 100.00
}
```

Governance rule: all error responses must be RFC 9457-shaped; extension members (like `currency`) are allowed under the problem's `type` namespace.

### 2.5 Versioning

Versioning is the governance area with the most religious debate. The standards options:

| Strategy | Mechanism | Strengths | Weaknesses |
|---|---|---|---|
| **URI versioning** | `/v1/customers` | Simple, human-visible, cache-friendly, tool-friendly | Clutters URIs; version becomes part of resource identity; encourages parallel versions forever |
| **Header versioning** | `X-API-Version: 1` or custom header | Clean URIs | Invisible in URLs (harder to debug/share); requires header propagation |
| **Media-type versioning** | `Accept: application/vnd.bank.customers.v1+json` | RESTful purist; content negotiation | Complex; poor tooling support; friction for consumers |

Industry practice: **URI versioning for major versions** (`/v1/`, `/v2/`) is the most common enterprise choice (⚠ — observed pattern across Zalando/Microsoft/Google-adjacent practice; each has nuances — Google's AIPs, for example, discourage versioning where avoidable), with **semantic versioning of the contract itself**. Versioning rules to standardize:

- **SemVer (semantic versioning, v2.0.0, 2013)** ✅ — `MAJOR.MINOR.PATCH`; for APIs: MAJOR = breaking change, MINOR = additive change, PATCH = non-visible fix. Note SemVer was designed for libraries; its API usage is an adaptation (⚠ semantics vary between organizations — some treat MINOR as "backwards-compatible additions" and never release a non-breaking MAJOR).
- **Additive changes** (new endpoint, new optional field, new enum value) must never break existing consumers — allowed without a new major version, *provided* consumers follow "ignore unknown fields" and servers follow "default missing optional fields".
- **Breaking changes** (rename/remove field, change type, change semantics, tighten validation, change auth requirements) require a new major version *and* a breaking-change review (Section 5.3).
- **Deprecation policy**: announce deprecation (Deprecation header, RFC 9745, November 2024 ✅), set a sunset date (Sunset header, RFC 8594, May 2019 ✅), publish a migration guide, and keep the old version alive for a minimum notice period (commonly 6–24 months; bank-specific policy in Section 10).

### 2.6 Pagination

Two standard approaches:

| Approach | Mechanism | Strengths | Weaknesses |
|---|---|---|---|
| **Offset/limit** | `?offset=40&limit=20` | Simple, jump-to-page, stable ordering OK for small sets | Skips/duplicates on concurrent writes; deep offsets are slow; page size unbounded without limit caps |
| **Cursor (keyset)** | `?cursor=<opaque-token>` | Consistent under writes; fast at any depth; opaque token hides internals | No random page access; token expiry; slightly more complex |

Governance rule: **cursor-based pagination for large or volatile collections; offset/limit acceptable for small stable ones**; always return `limit` caps (e.g., max 100–1000), always include next-page linkage. The `Link` header (`rel="next"`, `rel="prev"`) is the standard way to expose cursors without polluting the payload:

```
Link: <https://api.example.com/v1/transactions?cursor=abc123>; rel="next"
```

### 2.7 Idempotency

Network retries are inevitable (client timeouts, gateway retries, agent-driven traffic — see [mcp_framework_tools_guide.md](ai_llm/mcp_framework_tools_guide.md) for the AI-agent consumer angle). **Idempotency** guarantees that retrying a request produces the same result as the first attempt. The practical enterprise convention (popularized by Stripe, documented in an IETF draft; ⚠ not an RFC):

- Client sends `Idempotency-Key: <uuid>` header on mutating operations.
- Server stores the key + request hash + response; replays the stored response for retries with the same key (within a window).
- Rule: **all POST/PATCH operations that create or mutate state must accept and honor Idempotency-Key**; GET/PUT/DELETE are idempotent by HTTP semantics.

This is a governance standard, not just a nicety: without it, payment and transfer APIs double-execute under retries — a direct operational risk in banking (cross-ref [payments_hub_guide.md](../banking/payments_hub_guide.md) for the payments context).

### 2.8 The Standards Table

| Standard | Purpose | Current version | Notes |
|---|---|---|---|
| REST / Richardson Maturity Model | Architectural style classification | RMM: 2008 (Richardson); popularized 2010 (Fowler) ✅ | Target Level 2; Level 3 optional |
| OpenAPI (OAS) | Machine-readable API contract | **3.1.1** (Feb 2024) ✅ | 3.1.0 Aug 2021; JSON Schema 2020-12; webhooks |
| HTTP | Methods, status codes, headers | HTTP/1.1 (RFC 9110–9112, 2022); HTTP/3 ✅ | Standardize method/status/header usage per API |
| Problem Details (errors) | Standard error payload | **RFC 9457** (Apr 2023), obsoletes RFC 7807 ✅ | `application/problem+json`; type/title/status/detail/instance |
| Semantic Versioning | Version numbering | SemVer 2.0.0 (2013) ✅ | Adapted for APIs; MAJOR = breaking |
| Sunset header | Retirement signaling | RFC 8594 (May 2019) ✅ | Companion: Deprecation header RFC 9745 (Nov 2024) ✅ |
| Idempotency-Key | Retry-safe mutations | Stripe convention / IETF draft (⚠) | All mutating operations |
| Pagination | Large collection access | Cursor vs offset (practice) | Link header for next/prev |

---

## 3. API Design Guidelines

Standards say *what* must hold; design guidelines say *how to design well*. The most effective way to build an enterprise design guideline is to **adopt and adapt** the public style guides from organizations that have already solved this at scale.

### 3.1 The Major Public API Style Guides

| Source | What it is | Key characteristics |
|---|---|---|
| **Microsoft API Guidelines** ✅ | GitHub `microsoft/api-guidelines` — the Azure API Guidelines | Resource-oriented REST, consistent naming, versioning guidance (URI + date-based versioning for Azure), error model, long-running operations, pagination |
| **Google API Design Guide** ✅ | `googleapis/api-design-guide`, published via AIPs (google.aip.dev) | Resource-oriented design (methods map to standard verbs), AIP-1xx/2xx numbering, proto-based thinking, strong conventions on naming, errors, pagination, versioning |
| **Zalando RESTful API Guidelines** ✅ | GitHub `zalando/restful-api-guidelines` — **open source**, the most widely reused enterprise-style guide | ~200 must/should/may rules (numbered Gxxx), covers REST + events, ships with the **Zally linter** and a spectral ruleset; explicitly designed to be adopted and adapted by other companies |

The Zalando guide is notable for governance purposes because it was built *as* a governance artifact: numbered, categorized rules with must/should/may priority levels, machine-checkable via Zally, and explicitly licensed for reuse. ⚠ Zalando discontinued active maintenance of the guidelines repository around 2024 (the repo remains the reference; the model is now maintained in adapted forks) — flag for currency when adopting.

### 3.2 Design Principles

Whatever the source guide, the governing principles are stable:

- **Resource-oriented**: model the domain as nouns/resources, not function calls — `POST /transfers`, not `POST /executeTransfer`.
- **Noun-based URIs, verbs via HTTP methods**: `GET /accounts/42/transactions`, `POST /accounts/42/transactions`; use verb-like sub-resources (`/transfers/42/cancel`) only for actions that don't fit methods.
- **Consistent naming**: pick one case per layer and enforce it — URIs lowercase kebab-case (`/payment-accounts`), JSON fields camelCase or snake_case *but consistently* (⚠ the public guides disagree: Zalando leans snake_case per popular platforms; Microsoft/Google lean camelCase — the governance point is to pick one and lint it).
- **Plural nouns for collections** (`/customers`, not `/customer`), singular for items (`/customers/42`).
- **Consistent errors, pagination, idempotency, versioning** per Section 2.
- **Backwards compatibility as a design goal**, not an afterthought: field additions, enum extensions, optionality by default.
- **API-first mindset**: the API is a product designed before the implementation (Section 3.3).

### 3.3 API-First (Design-First)

**API-first** is both a mindset and a movement (✅ well-established terminology — popularized by the Swagger/OpenAPI ecosystem and platform vendors such as Google's API-first strategy; not an official standard). It means: the API contract is **the first artifact** — created, reviewed, and versioned before (or without) implementation. Consequences:

- Business/architecture decide the API; implementation conforms to it — not the reverse.
- Multiple consumers (internal apps, partners, agents — see [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md) and [mcp_framework_tools_guide.md](ai_llm/mcp_framework_tools_guide.md)) are designed for from day one.
- Governance has a single artifact to review: the contract.

### 3.4 Contract-First Development

Contract-first is API-first made operational — the **OpenAPI-first workflow**:

```
Design contract → Review (design review board) → Mock (Prism/mock server) →
Generate (codegen, SDKs, docs) → Develop against contract → Test contract compliance →
Publish to registry → Deploy behind gateway
```

- **Mocking**: consumers and providers develop in parallel against a contract-driven mock ([mock_server_guide.md](mock_server_guide.md) covers the tooling).
- **Code generation**: OpenAPI Generator / vendor generators produce server stubs, client SDKs, and docs from the contract — eliminating drift between contract and code.
- **Test generation**: contract tests (e.g., via Dredd/Schemathesis-style tools) verify the running API against its OpenAPI document.
- **Spec-driven development**: see [spec_driven_development_frameworks_guide.md](spec_driven_development_frameworks_guide.md) for the broader family of specification-first methodologies.

### 3.5 Linting: Spectral, Zally, and CI Enforcement

Standards only bind when they are machine-checked. The standard tooling:

- **Spectral** ✅ — open-source JSON/YAML linter by Stoplight; the de-facto standard for OpenAPI linting. Runs rulesets (built-in OpenAPI rules + custom rules) against contracts; supports OAS 3.0/3.1; integrates into CI and editor workflows; YAML/JSON rulesets committed to the repo.
- **Zally** ✅ — Zalando's open-source API linter (Java), built to enforce the Zalando guidelines; exposes a REST API and a CLI.
- **Vacuum** (open-source, newer), **Stoplight** (commercial platform: Studio for design, Spectral engine, Prism mocking, documentation) ✅.

Governance practice: a **custom Spectral ruleset encoding the enterprise style guide** runs on every contract change in CI (a "contract build"): severity levels must/should/may mapped to build pass/warn. Linting failures block merge — that is the enforcement mechanism that makes the style guide real (Section 5.4).

### 3.6 Guideline Adoption: "Adopt, Adapt"

The proven adoption path for enterprise style guides:

1. **Adopt** a battle-tested public guide (Zalando or Microsoft as the base; Google's AIPs for resource-design specifics).
2. **Adapt**: trim to ~30–60 rules that matter to your domain; add bank-specific rules (data protection fields, audit headers, idempotency mandatory on money movement); assign must/should/may priorities.
3. **Publish** the custom style guide with examples (good/bad pairs) — as a versioned, reviewed document *and* as a Spectral ruleset (single source: ruleset generated from the guide or vice versa).
4. **Enforce** via CI linting and design review (Sections 5.1, 5.4).
5. **Evolve**: the guide itself is governed — rule changes go through the API guild, are versioned, and deprecate old rules with migration notices.

⚠ The "adopt, adapt" method is universal practitioner advice, not a formal standard.

### 3.7 The Guidelines Table

| Source | Focus | Adoption in an enterprise program |
|---|---|---|
| Microsoft API Guidelines (Azure) | Resource-oriented REST, Azure-scale operations, versioning, pagination | Good base for large enterprises; cloud-native bias |
| Google API Design Guide / AIPs | Resource design, standard methods, naming, errors | Best for resource-model rigor; proto/RPC heritage |
| Zalando RESTful API Guidelines | ~200 numbered REST+event rules, must/should/may, Zally linter | **Most common starting point** for custom enterprise guides (open source, built to be forked) |
| Custom enterprise style guide | Your standards + domain rules | The actual governance artifact; expressed as doc + Spectral ruleset |

---

## 4. API Lifecycle

### 4.1 The Lifecycle Stages

A governed API passes through defined stages; each stage has activities, gates, and artifacts. The canonical lifecycle (⚠ practitioner-standard sequence; variants exist but the shape is universal):

```
Design → Develop → Test → Deploy → Operate → Retire
  ↑___________________________________________|
        (version N+1 loops back into Design)
```

| Stage | Activities |
|---|---|
| **Design** | Business need → capability analysis → contract authored (OpenAPI) → style-guide linting → design review |
| **Develop** | Contract-first implementation, codegen, mock-first parallel consumer development |
| **Test** | Contract tests, functional tests, security tests (OWASP-aligned), performance/load tests, chaos for resilience |
| **Deploy** | CI/CD pipeline, gateway configuration, registry publication, docs/SDK publication (see [kargo_gitops_guide.md](kargo_gitops_guide.md) for GitOps delivery of the surrounding platform) |
| **Operate** | Monitoring, alerting, SLOs, consumer support, security monitoring, capacity management |
| **Retire** | Deprecation notice, sunset date, consumer migration, shutdown, registry removal |

### 4.2 Lifecycle Gates

Gates are the governance checkpoints that decide whether an API may advance:

| Gate | When | Who | What it checks |
|---|---|---|---|
| **Design review** | Contract complete, pre-development | API design review board | Style-guide conformance, resource design, error/versioning/pagination/idempotency, naming, security design |
| **Security review** | Design + pre-production | Security team / AppSec | Authn/authz model, data classification, OWASP API Top 10 checklist, secrets handling, pen-test readiness |
| **Breaking-change review** | Any change to a published API | Review board + affected consumers | Is the change truly breaking? Is a new major justified? Migration plan? Deprecation communication? |
| **Release/launch review** | Pre-production | Platform team + owner | Gateway policies configured, quotas set, registry metadata complete, SLOs defined, runbooks exist |

Gates must be **lightweight by default** (self-service checklists + automated linting cover 80%) and escalate only exceptions to human review — otherwise the review board becomes a bottleneck and teams route around it (⚠ structural observation).

### 4.3 Versioning Strategy

The governed versioning strategy (details in Section 2.5):

- **Additive changes** (new endpoint, new optional field, new enum value, relaxed validation): no new major version; must be non-breaking and documented in the contract changelog.
- **Breaking changes** (removal/rename/type change/semantic change): require a **new MAJOR version** + breaking-change review + consumer impact assessment + migration guide.
- **Deprecation policy**: when a version is superseded, mark it deprecated (Deprecation header, RFC 9745), announce publicly, publish the migration path, and keep it running at least the mandated notice period (commonly 6–24 months; regulated contexts may mandate longer).
- **Never silently change** published behavior — contract diffs in CI detect drift between the deployed API and its published contract (Section 5.4).

### 4.4 Retirement: Sunset Policy

Retirement is the most-often-neglected stage — and the one that determines whether the estate stays governable. Governed retirement:

1. **Deprecation**: mark deprecated in the registry, return the `Deprecation` header (RFC 9745) on responses, update docs.
2. **Sunset announcement**: return the `Sunset` header (RFC 8594) with the removal date — machine-readable, so client tooling can warn consumers automatically:

```
Deprecation: true
Sunset: Wed, 31 Dec 2026 23:59:59 GMT
Link: <https://api.example.com/v2/customers>; rel="successor-version"
```

3. **Migration period**: publish migration guide; work with known consumers; track traffic on the deprecated version.
4. **Shutdown**: return `410 Gone` after the sunset date; remove from gateway and registry; archive the contract.

Governance rule: **no API retires silently**. Every retirement has a record (who, when, why, migration path) — this is also the compliance story regulators ask for (Section 10).

### 4.5 The Lifecycle Table

| Stage | Activities | Gates | Artifacts |
|---|---|---|---|
| Design | Needs analysis, contract authoring, linting | Design review, security review (design phase) | OpenAPI contract, style-guide conformance report, ADR |
| Develop | Contract-first implementation, mocks, codegen | CI linting (contract build) | Source, generated SDKs, mock server |
| Test | Contract/functional/security/performance tests | Test sign-off, security review (pre-prod) | Test reports, pen-test findings (see [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md)) |
| Deploy | CI/CD, gateway config, registry publish | Release/launch review | Gateway policies, registry entry, docs/SDK publication |
| Operate | Monitoring, SLOs, support, capacity | Operational reviews | Dashboards, SLO reports, runbooks, incident records |
| Retire | Deprecation, sunset, migration, shutdown | Breaking-change/sunset approval | Sunset notice, migration guide, retirement record |

---

## 5. API Governance Processes

Processes are the workflow spine of governance: they convert standards into decisions, and decisions into records.

### 5.1 Design Review (The Review Board)

An **API design review board** (naming varies: API council, API review committee) is the human gate for API design decisions. Operating principles:

- **Trigger**: every new API, every new major version, every contract-level change to a published API.
- **Composition**: API stewards, domain architects, security representative, platform/API platform team lead; consumers invited for high-impact APIs.
- **Cadence**: fixed weekly/bi-weekly slots (batched reviews) so it never blocks delivery.
- **What is reviewed**: contract conformance to the style guide (lint report attached), resource design, error/versioning/idempotency choices, security design, data protection implications.
- **Outcome**: approve / approve-with-conditions / reject, recorded as a decision record; conditions are tracked to closure.
- **Efficiency rule**: automated linting decides the mechanical 80%; the board spends its time on design judgment (resource modeling, breaking-change justification, domain semantics).

### 5.2 The API-First Process (Contract-First Workflow)

The API-first process operationalized (Section 3.4) as a repeatable workflow:

```
1. Need identified → 2. Contract drafted (OpenAPI 3.1, style guide applied)
→ 3. Linted in CI (Spectral ruleset) → 4. Design review board
→ 5. Mock published (Prism/mock server) → 6. Provider develops; consumers build against mock
→ 7. Contract tests green → 8. Deploy behind gateway → 9. Registered in catalog
```

Key governance properties: the contract is versioned in source control like code; the mock is a governance artifact (consumers can validate before the real API exists); nothing reaches the gateway without a linted, reviewed contract.

### 5.3 Change Management: Breaking Changes and Approval

- **Change types**: additive (no approval — must pass CI contract checks), breaking (requires breaking-change review + new major + deprecation plan), behavioral (subtle semantic changes — often sneakier than breaking ones; require review too).
- **Approval flow**: change proposal (what/why/impact) → consumer impact assessment → review board decision → communication (changelog, deprecation notices) → implementation.
- **Records**: every breaking change is an ADR (architecture decision record) — the audit trail regulators and internal risk ask for.
- **No backdoor changes**: the deployed contract must match the published contract; CI contract-diff detects drift (a "contract compliance" pipeline, Section 5.4).

### 5.4 Compliance: Standards Conformance in CI

Compliance is governance made automatic:

- **Contract linting in CI** (Spectral/Zally) — every contract change must pass the enterprise ruleset; severity must/should/may mapped to pass/warn. ✅ This pattern is standard practice in the OpenAPI tooling ecosystem.
- **Contract-drift detection** — scheduled jobs diff the running API against its published contract (contract tests) and alert on divergence.
- **Gateway policy conformance** — the platform verifies every API is behind the mandated policy set (authn required, rate limits attached, quotas configured).
- **Registry completeness** — metadata (owner, status, SLA, docs) is machine-checked; incomplete entries block "GA" status.
- **Audit logging** — review decisions, approvals, and policy changes are logged with timestamps and identities.

### 5.5 The Processes Table

| Process | Trigger | Owner | Outcome |
|---|---|---|---|
| Design review | New API / new major / contract change | Design review board (stewards + architects + security) | Approved / conditional / rejected decision record |
| API-first / contract-first flow | New API need | API owner + platform team | Linted, reviewed contract + mock in source control |
| Breaking-change review | Breaking change to published API | Review board + affected consumers | Approval + new major version + migration plan |
| Deprecation & sunset | Version superseded | API owner | Sunset notice + migration guide + retirement record |
| CI compliance (linting/drift) | Every contract change; scheduled drift scans | Platform team (automation) | Green/red build; drift alert |
| Registry publication | Design approval → GA | API owner | Catalog entry with complete metadata |

---

## 6. API Registry and Catalog

### 6.1 The API Registry: Discoverability as a Governance Control

An **API registry/catalog** is the system of record for *what APIs exist, who owns them, and what state they are in*. It is a governance control, not just a convenience: without it, discoverability fails and teams build duplicate APIs for the same capability.

Registry functions:

- **Discoverability**: consumers find APIs by capability, domain, owner.
- **Quality/state**: each API carries a status (proposed → in design → in development → GA → deprecated → sunset), version, and quality scorecard.
- **Ownership**: every entry names an accountable owner and support contact.
- **Lifecycle visibility**: deprecations and sunsets are visible to consumers before they break.
- **Portfolio analysis**: the estate (count, duplication, maturity, coverage) becomes measurable — the raw material for API strategy.

### 6.2 Catalog Platforms

| Platform | Type | Governance-relevant features |
|---|---|---|
| **Backstage** ✅ | Open-source developer portal / IDP framework (created by Spotify, open-sourced 2020, CNCF project) | Software catalog with first-class **API entities** (`kind: API`, with `spec.type: openapi`), ownership metadata, scorecards/checks, docs, tech-radar style views; extensible via plugins |
| **Apigee** ✅ | Commercial API management (Google) | API hub/catalog, developer portal, analytics, policies, monetization; catalog coupled to the runtime |
| **Kong** ✅ | API gateway + Kong Konnect portal (open-source gateway, commercial control plane) | Portal with API listings, docs from OpenAPI, RBAC, analytics; catalog tied to gateway config |
| **Azure APIM** ✅ | Cloud API management (Microsoft) | Developer portal, product/API/subscription model, policy enforcement, analytics |
| **MuleSoft Anypoint** ✅ | Integration + API management (Salesforce) | Anypoint Exchange as the catalog/registry across APIs and integrations |
| Others | WSO2, Red Hat 3scale, Gravitee, Tyk | Each ships a portal/catalog with varying registry depth |

**Backstage** is the notable *governance-first* choice: as a developer portal it registers APIs independently of any vendor runtime, making it a neutral system of record (catalog-as-cache pattern: entity definitions live in YAML in source control, ingested into Backstage — ✅ per Backstage docs, the catalog is a caching layer fed by processors, not the source of truth).

### 6.3 Catalog Metadata

Governance requires a minimum metadata contract per API entry (machine-checked):

| Field | Meaning | Required? |
|---|---|---|
| `owner` | Team/individual accountable (API owner) | ✅ |
| `status` | proposed / in-development / GA / deprecated / sunset | ✅ |
| `version` | Current contract version (semver) | ✅ |
| `contract` | Link to OpenAPI document | ✅ |
| `docs` | Developer documentation URL | ✅ |
| `SLA` | Availability target, support hours, severity response times | ✅ (GA) |
| `consumers` | Known consuming applications | Recommended |
| `tags` / `domain` | Capability classification | Recommended |
| `scorecard` | Lint pass, security review date, test coverage | ✅ (GA) |

### 6.4 The Registry Table

| Platform | Type | Features |
|---|---|---|
| Backstage | Open-source IDP/catalog (CNCF) | API entities, ownership, scorecards, vendor-neutral system of record |
| Apigee | Commercial API management (Google) | Catalog + developer portal + analytics + monetization, coupled to runtime |
| Kong (Konnect) | Open-source gateway + commercial platform | Portal, OpenAPI docs, RBAC, analytics |
| Azure APIM | Cloud API management (Microsoft) | Developer portal, products/subscriptions, policies |
| MuleSoft Anypoint Exchange | Integration/API platform (Salesforce) | Registry across APIs and integrations |
| WSO2 / 3scale / Gravitee / Tyk | Open-source/commercial API platforms | Portals with varying registry depth |

---

## 7. API Gateway and Runtime Governance

### 7.1 Runtime Controls

The gateway is where governance becomes mechanical at runtime — every request passes through the policy engine. Standard runtime controls:

| Control | What it does | Governance role |
|---|---|---|
| **Authentication** | Validates tokens/keys/mTLS at the edge | Enforces "no anonymous APIs" standard |
| **Authorization** | Scope/OAuth claims checks, policy-based access | Enforces least-privilege consumption |
| **Rate limiting** | Per-consumer request rate caps | Protects capacity; enforces fair-use |
| **Quotas** | Periodic (daily/monthly) call allowances | Enforces consumption contracts/SLA tiers |
| **Routing** | Path/host-based routing to backends, version routing | Implements versioning strategy at runtime |
| **Transformations** | Request/response mapping, header injection/stripping | Enforces wire conventions (media types, headers) |
| **Caching** | Response caching for idempotent reads | Performance + cost control |
| **Circuit breaking / retries** | Failure isolation, bounded retries | Resilience; pairs with Idempotency-Key (Section 2.7) |
| **Observability hooks** | Metrics/logs/trace propagation | Feeds the observability stack (Section 8.3) |

### 7.2 The Gateway Landscape

| Gateway | Vendor | Notes |
|---|---|---|
| **Apigee** ✅ | Google | Full lifecycle platform: gateway, analytics, developer portal, monetization; heavy enterprise adoption |
| **Kong Gateway** ✅ | Kong Inc. | Open-source gateway (NGINX/OpenResty heritage), plugin ecosystem, Konnect control plane; popular in Kubernetes estates |
| **AWS API Gateway** ✅ | Amazon | Cloud-native, serverless integration, usage plans/API keys, WAF integration; AWS-centric estates |
| **Azure APIM** ✅ | Microsoft | Azure-native, policy-rich, developer portal; Azure estates |
| **MuleSoft Anypoint API Manager** ✅ | Salesforce | API manager across the Anypoint integration platform; strong in integration-heavy shops |
| **Tyk / Gravitee / WSO2 / 3scale** | Various | Open-source options with growing enterprise adoption; deployment flexibility |

Deployment models: SaaS/cloud (Apigee, AWS, Azure), self-managed (Kong OSS, Tyk, WSO2), hybrid (gateway in your VPC with cloud control plane — increasingly the norm for banks with data-residency constraints; see [cloud_providers_guide.md](cloud_providers_guide.md)).

### 7.3 Gateway Policy Templates

Governance is expressed as **policy templates** that teams must attach (and that the platform audits):

- **Security policies**: JWT/OAuth validation, API key checks, mTLS termination, IP allowlists for partner traffic, request size limits, header hygiene (no secrets in headers/logs).
- **Traffic policies**: rate limits and quotas per tier (internal / partner / public), spike arrest, caching rules, retry budgets.
- **Standardization policies**: enforce Content-Type, Problem Details error passthrough, Idempotency-Key acceptance, CORS rules, `Deprecation`/`Sunset` header passthrough.

### 7.4 Gateway vs. Service Mesh

A common governance confusion: **north-south vs. east-west**.

- **API gateway** = north-south (external-facing edge): external clients → APIs; handles authn, rate limiting, quotas, API-level policies, developer-facing concerns. Governs *consumption of APIs*.
- **Service mesh** (Istio, Linkerd) = east-west (internal service-to-service): mTLS between services, fine-grained traffic policy, retries/timeouts, observability *inside* the cluster. Governs *internal call topology*.
- **Relationship**: complementary, not either/or. The mesh secures internal traffic; the gateway is the governed front door for consumers. Governance standards should state which layer handles which control (e.g., "all external traffic via gateway; internal east-west mTLS via mesh where deployed") to avoid duplicated or contradictory policies.

### 7.5 The Gateway Table

| Gateway | Vendor | Features | Deployment |
|---|---|---|---|
| Apigee | Google | Full lifecycle, analytics, portal, monetization | SaaS / hybrid (Apigee hybrid) |
| Kong | Kong Inc. | Open-source core, plugin ecosystem, Konnect | Self-managed / hybrid / SaaS |
| AWS API Gateway | Amazon | Serverless, usage plans, WAF, Lambda integration | AWS cloud |
| Azure APIM | Microsoft | Policy engine, portal, Azure-native | Azure cloud / self-hosted gateway |
| MuleSoft API Manager | Salesforce | Anypoint integration, API governance across flows | SaaS / hybrid / on-prem |
| Tyk / Gravitee / WSO2 / 3scale | Various | Open-source, flexible policies | Self-managed / cloud |

---

## 8. API Security and Observability

### 8.1 Authentication and Authorization

Security standards are a core governance artifact: they decide *which* mechanisms are allowed, *where* they are enforced, and *how* they are configured. The standard toolbox:

| Mechanism | Standard | Use case | Governance notes |
|---|---|---|---|
| **OAuth 2.0** ✅ | RFC 6749 (2012) | Delegated authorization; scoped access to protected resources | The enterprise default for API authorization. Key flows: **client credentials** (server-to-server, no user), **authorization code + PKCE** (user-facing apps), implicit flow deprecated (RFC 8252 era guidance — flag: many older guides still reference it; do not adopt for new APIs) |
| **OIDC** (OpenID Connect) ✅ | OpenID Foundation, 2014 (built on OAuth 2.0) | Identity layer: verifiable user identity (`id_token` + `userinfo`) | Use when APIs need to know *who* the user is, not just which scopes apply; standard for SSO integration |
| **JWT** ✅ | RFC 7519 (2015) | Self-contained access tokens (claims, expiry, signature) | Governance: short expiry, `aud`/`iss` validation, signature algorithm pinning (reject `none`/weak algs), no secrets in claims |
| **API keys** ✅ | Vendor convention (no single RFC) | Lightweight identification of internal/partner callers; quotas | Weak authn (no identity proof) — governance rule: keys only for low-risk internal use, always paired with rate limiting; never for user-level authorization |
| **mTLS** ✅ | RFC 8705 (2020, OAuth mTLS) / TLS itself | Mutual certificate authentication; strong server-to-server identity | Governance rule for high-risk partner/system-to-system traffic (banking: interbank, vendor integrations); cert lifecycle management required |

Governance decisions to standardize: **token validation at the gateway** (JWT signature, expiry, audience, issuer), **scope design** (least privilege; per-resource scopes), **token storage rules** (no tokens in URLs or logs), **secret handling** (Vault/secret managers — cross-ref [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) for the secrets tooling angle), and **credential rotation** requirements.

### 8.2 Security Standards: OWASP API Security Top 10

The **OWASP API Security Top 10** is the reference threat list for API-specific risks; the **2023 edition** is current ✅ (first published 2019). Governance use: each API must be designed/verified against it; security review maps findings to the list.

| # | Risk (2023) | Short description |
|---|---|---|
| API1:2023 | **Broken Object Level Authorization (BOLA)** | Accessing another user's object by ID manipulation — the #1 API flaw |
| API2:2023 | Broken Authentication | Weak/leaked credentials, broken session/token handling |
| API3:2023 | Broken Object Property Level Authorization (BOPLA) | Reading/writing properties the caller shouldn't touch (mass assignment) |
| API4:2023 | Unrestricted Resource Consumption | No rate limits / unbounded payloads → DoS, cost abuse |
| API5:2023 | Broken Function Level Authorization | Calling admin functions without admin rights (horizontal/vertical privilege issues) |
| API6:2023 | Unrestricted Access to Sensitive Business Flows | Automated abuse of business logic (e.g., bulk account enumeration) |
| API7:2023 | Server-Side Request Forgery (SSRF) | Fetching arbitrary URLs from the server via user input |
| API8:2023 | Security Misconfiguration | Missing hardening, permissive CORS, verbose errors, missing headers |
| API9:2023 | Improper Inventory Management | Shadow APIs, undocumented/versionless endpoints, stale environments |
| API10:2023 | Unsafe Consumption of APIs | Blindly trusting third-party/partner API data |

Note the governance linkage: **API9 (Improper Inventory Management) is the security consequence of missing registry governance** — the catalog (Section 6) is a security control, not a convenience. API4 is enforced by gateway rate limiting/quotas (Section 7). Security testing practice cross-refs [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md).

### 8.3 Observability

An unobserved API cannot be governed: operations and governance both depend on telemetry.

- **Metrics**: the RED method (Rate, Errors, Duration) per API/endpoint — request rate, error rate (by status class and problem `type`), latency percentiles; plus business metrics (conversion, usage by consumer). Use limits (traffic), saturation.
- **Logs**: structured logs with correlation IDs; request/response summaries without sensitive payloads; audit logs for compliance (who accessed what — mandatory in banking).
- **Tracing**: distributed traces via W3C `traceparent` propagation and OpenTelemetry instrumentation, correlating gateway → service → downstream (mesh and gateway both propagate).
- **SLOs**: per-API service-level objectives (availability, latency percentiles) with error budgets; registry SLA metadata (Section 6.3) should match the SLOs being measured.
- **API analytics**: gateway analytics (Apigee/Kong/Azure APIM) give consumer-level usage, quotas consumed, and error patterns; feed capacity and monetization decisions.

### 8.4 The Security Table

| Control | Threat addressed | Implementation |
|---|---|---|
| OAuth 2.0 + scopes | Broken Function/Property Level Authorization, BOLA at scale | Authorization server, gateway token validation, least-privilege scopes |
| OIDC | Identity spoofing, session issues | IdP integration, `id_token` verification, SSO |
| JWT validation (alg/aud/iss/exp) | Token forgery, replay | Gateway JWT policy, short expiry, alg pinning |
| API keys + rate limits | Abuse, quotas, shadow usage | Gateway key auth + rate limiting/quotas |
| mTLS (RFC 8705) | Impersonation in system-to-system traffic | Certificates + PKI lifecycle management |
| OWASP API Top 10 review | BOLA, BOPLA, SSRF, misconfiguration, etc. | Security design review + automated scanning + pen test |
| Registry/governance | Improper Inventory Management (shadow APIs) | Catalog with mandatory metadata (Section 6) |
| Rate limiting/quotas | Resource consumption abuse (API4) | Gateway policies (Section 7) |
| RED metrics + tracing + SLOs | Silent degradation, regressions | OpenTelemetry, gateway analytics, alerting |
| Structured audit logs | Compliance/forensic gaps | Correlation IDs, log retention, access records |

---

## 9. Governance Roles and Operating Model

### 9.1 The Roles

Governance runs on people. The core role set (⚠ role names vary by organization; the *functions* are the stable part):

- **API Owner** — the accountable individual/team for one API or API group: the product owner of the API. Owns its roadmap, quality, SLA, consumers, deprecation. Accountability: answerable for the API's success and compliance.
- **API Steward** — the standards-enforcement function: reviews contracts, chairs/feeds the design review board, maintains the style guide and Spectral ruleset, arbitrates exceptions. Usually domain-architect-level people with a horizontal remit.
- **API Guild** — the community of practice: all API owners + stewards + platform engineers; shares patterns, publishes guidance, runs the design review board, owns the guidelines' evolution. The governance "court of appeal" for standards disputes.
- **Center of Enablement (CoE)** — the funded, dedicated team that operates the API program: platform, tooling (gateway, registry, linting, portal), training, standards maintenance, metrics. Sometimes called API Platform Team / API Center of Excellence.
- **Platform team** — runs the runtime machinery (gateway, developer portal, CI tooling) — the "management" layer from Section 1.3.

### 9.2 Operating Models

| Model | Shape | Strengths | Weaknesses |
|---|---|---|---|
| **Centralized** | One central team owns all APIs and standards | Maximum consistency, tight security control | Bottleneck, slow; teams disown their APIs; doesn't scale |
| **Federated** | Platform/CoE owns standards + tooling; domain teams own and build their APIs; stewards per domain enforce | Scales; domain expertise in APIs; the widely recommended enterprise model (⚠ practitioner consensus) | Requires strong standards + automated enforcement or consistency decays |
| **Hybrid** | Central standards + tooling; domain ownership; central review for high-risk/cross-domain APIs only | Balance of speed and control | Governance is more complex; exception handling needs discipline |

The modern enterprise default is **federated governance** (the platform-team + domain-teams pattern — the same shape as [data mesh architectures](data/data_mesh_architectures_guide.md) applied to APIs): the CoE/platform team provides the rails (standards, linting, gateway, registry, portal); domain teams own their APIs and are accountable for them; stewards and the review board keep conformance honest.

### 9.3 The Roles Table

| Role | Responsibilities | Authority |
|---|---|---|
| API owner | Roadmap, quality, SLA, consumers, deprecation of their API(s) | Accountable for the API; can approve additive changes |
| API steward | Enforce standards, maintain style guide + ruleset, review contracts, arbitrate exceptions | Can block non-conforming contracts (escalation to guild) |
| API guild | Community of practice: patterns, guidelines evolution, design review board | Owns the standards; court of appeal |
| Center of Enablement / platform team | Operate gateway, registry, portal, linting, training, metrics | Owns tooling and runtime policies; can enforce platform-wide policy |
| Security (AppSec) | Security review gate, OWASP alignment, pen-test coordination | Can block insecure APIs |

---

## 10. Banking Context and Worked Example

### 10.1 Open Banking and the Regulatory Drivers

API governance in banking is not optional — it is regulation-adjacent. The major regimes:

- **PSD2 (EU)** ✅ — Directive (EU) 2015/2366 (Revised Payment Services Directive): in force 13 January 2018; Strong Customer Authentication (SCA) technical standards from 14 September 2019; supervised by the EBA and national authorities. Forces banks to expose account data and payment initiation to licensed Third-Party Providers (TPPs) via APIs, under strict security and access rules. Successor: **PSD3/PSR** (proposed 2023, flag ⚠ status) continues the open-banking agenda. PSD2 is *not* an API-standards mandate per se (it mandates access + security; the API specs came from market schemes) — a common misreading worth flagging.
- **CMA Open Banking (UK)** ✅ — the Competition and Markets Authority's Open Banking remedy, live January 2018, requiring the **CMA9** largest UK banks to expose standardised open APIs (the UK Open Banking Standard: accounts, payments, funds confirmation) to regulated providers.
- **MAS (Singapore)** ✅ — the Monetary Authority of Singapore does **not** mandate a PSD2-style open-banking regime; it drives an *enabling* approach: the **"Finance-as-a-Service API Playbook"** published by **MAS and the Association of Banks in Singapore (ABS)** in **2016** (announced at the Singapore FinTech Festival), giving guidance on common, useful APIs financial institutions could make available; the **SGFinDex** financial-data exchange (2019, government-mediated consent-based data sharing); and API-security expectations inside the **Technology Risk Management (TRM) guidelines** (2021 update includes API-security expectations). ⚠ The often-quoted "MAS API Playbook" is this Finance-as-a-Service API Playbook — there is no separate standalone MAS-mandated API standard; Singapore's approach remains guidance + industry collaboration rather than regulation.
- **Open finance trend**: data-sharing regimes are widening from payments/accounts toward lending, insurance, and wealth (UK Smart Data, EU FIDA proposals, Singapore open-finance consultations) — all of which *increase* the governance burden on bank API platforms (⚠ forward-looking trend, flagged).

### 10.2 Bank API Governance in Practice

A bank's API platform context (companion guides): the [DBS Software Systems guide](../banking/dbs_software_systems_guide.md) documents a real-world bank API platform (DBS's API platform strategy, open API developer portal) — the *bank-specific* sibling to this guide; the [Oracle Banking Microservices Architecture guide](../banking/oracle_banking_microservices_architecture_guide.md) and [Oracle FLEXCUBE Data Model guide](../banking/oracle_flexcube_data_model_guide.md) show how REST APIs wrap core banking services (FLEXCUBE exposes REST APIs over its core services); [BIAN](../banking/bian_banking_architecture_guide.md) is the banking-domain service landscape many bank API programs map their API portfolio against.

Bank-specific governance dimensions beyond the generic framework:

- **Compliance evidence**: review decisions, contract versions, sunset records, and access logs must survive audit — governance artifacts are regulatory artifacts.
- **Data protection**: API payloads carry personal and financial data; governance sets field-level classification, masking, retention, and cross-border rules.
- **Third-party access**: partner/TPP onboarding (PSD2 AISP/PISP, open-finance providers) is a governed process: KYC/technical due diligence, credential issuance, quota tiers, monitoring.
- **Availability & resilience**: payment APIs have hard availability expectations; SLOs, runbooks, and change windows are governed, not ad hoc.
- **No shadow banking APIs**: the registry (Section 6) doubles as the control against undocumented/unmanaged APIs — a regulatory and risk requirement.

### 10.3 Worked Example: Merlion Bank's API Governance Operating Model

*Illustrative operating model for a mid-size Singapore bank (fictional "Merlion Bank") — concrete instantiation of every section above.*

**Context.** Merlion Bank runs ~400 internal/partner APIs across retail (accounts, transfers, cards, DDA), wealth, and payments. Past state: no standards, ~9 authn schemes, 0 catalog entries, 3 shadow payment endpoints discovered by pen test (API9!). Target: a governed, federated API program.

**1. Standards (Section 2).** Adopt **OpenAPI 3.1** as the sole contract format (migration from mixed Swagger 2.0/OAS 3.0 with a 12-month runway); adopt **RFC 9457** problem details everywhere; **URI versioning** (`/v1`, `/v2`) + semver contract versions; **cursor pagination** for transaction/statement endpoints; **Idempotency-Key mandatory** on all money-movement POSTs (transfer, payment, FX) with 24h replay window; sunset policy = 12 months minimum notice for internal, 18 months for partner APIs (regulator-adjacent: flagged ⚠ as internal policy choice).

**2. Design guidelines (Section 3).** Custom style guide derived from **Zalando** (base) + **Microsoft** (error/versioning) + bank rules: camelCase JSON, kebab-case URIs, PSD2-style scopes, no PII in URIs/logs, idempotency rules. The guide is *generated from* a **Spectral ruleset** (single source of truth): ~55 rules (must/should/may). Every contract change runs Spectral in CI; must-failures block merge.

**3. Lifecycle & processes (Sections 4–5).** Six-stage lifecycle with gates: design review (weekly **API design review board**: stewards + domain architect + AppSec + platform lead), security review (OWASP API Top 10 checklist), breaking-change review (consumer impact + migration plan), launch review (gateway policies + registry metadata + SLOs). API-first flow: contract → lint → review → **mock (Prism)** → develop → contract tests → deploy → register. CI contract-diff job alerts on drift between deployed and published contracts.

**4. Roles (Section 9).** **Federated model**: API Platform CoE (12 people: platform, gateway, registry, linting, training) + **API guild** (all owners + stewards, monthly) + per-domain **stewards** + **API owners** per API group (accountable, named in the catalog). AppSec embedded in the review board.

**5. Registry (Section 6).** **Backstage** catalog as the neutral system of record: API entities with owner/status/version/contract/SLA/scorecard; scorecard = lint pass + security review recency + contract-test coverage; "GA" requires scorecard green. Developers self-serve docs, mocks, and SDKs from the catalog.

**6. Gateway (Section 7).** **Kong** (open-source, hybrid: in-VPC data plane for residency) with policy templates: OAuth 2.0 (client-credentials for server-to-server, auth-code + PKCE for user apps) validated at the gateway; **mTLS** for interbank/vendor system-to-system; rate limits + quotas per tier (internal/partner/public); Idempotency-Key passthrough; `Deprecation`/`Sunset` header passthrough.

**7. Security & observability (Section 8).** OWASP API Top 10 2023 as the security review baseline; annual pen test of critical APIs (cross-ref PTES guide); RED metrics + OpenTelemetry tracing per API; SLOs (e.g., 99.9% availability, p95 < 500ms for transfers) with error budgets; audit logs retained 7 years for payment APIs.

**8. Worked flow — onboarding a new Payments API ("Instant Transfer v1").** Need → contract drafted (OpenAPI 3.1, style guide, Spectral green) → design review board (approved; conditions: idempotency window 24h, BOLA test cases) → Prism mock published → consumers (mobile app team, 2 partners) build against mock → implementation → contract tests + OWASP checklist + pen test findings closed → launch review (Kong policies attached: auth-code PKCE, tier quotas, 429 handling) → Backstage registration (owner: Payments API Team; status: GA; SLA: 99.9%, p95 500ms; scorecard green) → operate (RED dashboards, error budgets) → when v2 ships: deprecate v1, Sunset header, 12-month notice, migration guide, 410 Gone at sunset, retirement record archived. **Total governance overhead: ~2 review meetings + CI automation — the gates are cheap because linting and templates do the heavy lifting.**

### 10.4 API Maturity

There is **no single industry-standard API maturity model** (⚠ — flagged: the RMM of Section 2.1 measures *design* maturity, not *program* maturity; program-level models are vendor/practitioner constructs — Boomi's 4-level model, IBM's maturity framing, Capgemini's API maturity model, etc. — with similar shapes but no consensus). The recurring practitioner shape:

| Level | Program characteristics |
|---|---|
| 1 — Ad hoc | APIs exist per team; no standards, no catalog, no gateway policy; shadow APIs |
| 2 — Managed | Gateway + portal introduced; some standards; consistency partial |
| 3 — Governed | Standards + linting in CI + design review + registry; deprecation/sunset practiced |
| 4 — Productized / platform | APIs as products with SLAs, monetization, self-service onboarding, portfolio analytics; federated operating model |
| 5 — Ecosystem | External/partner ecosystem, open finance, API marketplaces; governance extends to third-party consumption |

Merlion Bank's model above targets **Level 3–4**. Maturity assessments should be run annually as a governance artifact (portfolio stats from the registry feed the assessment).

### 10.5 One-Page Summary: Governed APIs Are Products

**The final word.** API governance is not bureaucracy — it is the discipline that makes APIs *products* rather than *projects*: every API has an owner, a contract, a lifecycle, a price of admission (standards), and an audience. The five pillars (principles, standards, processes, roles, tools) plus the six governance scopes (design, lifecycle, security, registry, consumption, retirement) form the operating system; the gateway and CI linting make it mechanical; the registry and observability make it transparent; the roles make it somebody's job; the banking regimes (PSD2, CMA, MAS) make it, in finance, a regulatory expectation.

| Governance element | Merlion Bank instantiation |
|---|---|
| Principles | API-first, secure by default, additive over breaking |
| Standards | OpenAPI 3.1, RFC 9457, semver + URI versions, Idempotency-Key, cursor pagination |
| Guidelines | Zalando-derived custom guide + Spectral ruleset |
| Lifecycle gates | Design review, security review, breaking-change review, launch review |
| Processes | API-first flow, CI linting, drift detection, deprecation/sunset |
| Roles | CoE/platform, guild, stewards, owners; federated |
| Registry | Backstage catalog + scorecards |
| Gateway | Kong + OAuth 2.0/OIDC, mTLS, tiers, quotas |
| Security | OWASP API Top 10 2023 baseline, annual pen test |
| Observability | RED metrics, OTel tracing, SLOs/error budgets |
| Compliance | Audit records, 7-year retention, MAS/PSD2-adjacent readiness |

Start small: **one style guide, one ruleset in CI, one registry, one named owner per API** — and grow the framework as the estate grows. Governance that scales is governance that is mostly automated.

---

## 11. Verification and Claims-Status

✅ **Verified in this research pass (primary/secondary sources):**
- OpenAPI Specification: 3.1.0 released Aug 2021; **3.1.1 current patch release (Feb 2024)**; JSON Schema 2020-12 alignment and webhooks in 3.1.
- Richardson Maturity Model: proposed by **Leonard Richardson in 2008**, popularized by **Martin Fowler's 2010 article**; levels 0–3 (Level 3 = hypermedia).
- Problem Details: **RFC 7807 (Mar 2016) obsoleted by RFC 9457 (Apr 2023)**; members type/title/status/detail/instance; `application/problem+json`.
- Sunset header: **RFC 8594 (May 2019)**; Deprecation header: **RFC 9745 (Nov 2024)**.
- SemVer 2.0.0 (2013); OAuth 2.0 RFC 6749 (2012); JWT RFC 7519 (2015); mTLS-in-OAuth RFC 8705 (2020); OIDC (OpenID Foundation, 2014).
- Zalando RESTful API Guidelines: open-source on GitHub (`zalando/restful-api-guidelines`), ~200 numbered rules, REST + events, Zally linter.
- Microsoft API Guidelines (`microsoft/api-guidelines`) and Google API Design Guide (`googleapis/api-design-guide`, AIPs) both public and current.
- Spectral: open-source linting by Stoplight, OpenAPI 3.0/3.1 support; Stoplight platform includes Studio/Prism.
- Backstage: open-source developer portal/catalog by Spotify (open-sourced 2020, CNCF), API entities (`kind: API`), catalog-as-cache pattern.
- Gateways: Apigee (Google), Kong, AWS API Gateway, Azure APIM, MuleSoft Anypoint all verified as current products.
- OWASP API Security Top 10: **2023 edition current**, API1:2023 = Broken Object Level Authorization.
- PSD2: Directive (EU) 2015/2366; applied 13 Jan 2018; SCA RTS 14 Sep 2019; PSD3/PSR proposed.
- CMA Open Banking: UK, live Jan 2018, CMA9 banks.
- MAS: **"Finance-as-a-Service API Playbook" (2016, MAS + ABS)** — the "MAS API Playbook" often referenced; MAS TRM guidelines (2021 update) include API security expectations; SGFinDex launched 2019.
- RFC 9457/8594/9745, semver, OAuth/OIDC/JWT/mTLS standards references above.

⚠ **Flagged (inferred, approximate, no single standard, or single-source):**
- "API governance framework = five pillars" — practitioner synthesis; no single ISO-style standard named "API governance" exists.
- API management market size: analyst estimates vary widely by definition (API-management software vs API economy); figures from ~US$4–7B (API management software, 2024, growth ~15–20% CAGR) to much larger "API economy" estimates — **treat as approximate; no single authoritative figure**.
- Zalando guidelines maintenance: active repository updates wound down ~2024 (repo remains the reference; adapted forks continue).
- Idempotency-Key: Stripe-popularized convention with IETF draft; **not an RFC**.
- API maturity models (program-level): Boomi/IBM/Capgemini-style practitioner models; **no single standard**; recurring 4–5 level shape.
- "Governance failure sequence" (consistency → security → duplication → compliance) — structural inference from practitioner literature, not a measured law.
- PSD3/PSR status, FIDA, and open-finance widening — forward-looking, status changes over time.
- Merlion Bank worked example — entirely illustrative.
- Minor: "OAS 3.1.1" is the current *patch*; minor releases carry revision-date schemas per OpenAPI project policy.

---

## 12. Glossary

- **API governance** — The discipline of applying standards, processes, and controls across the API lifecycle (design → retire) so APIs are consistent, secure, discoverable, reusable, and compliant.
- **API management** — The runtime/operational layer of the API estate: gateway, rate limiting, quotas, analytics, developer portal, lifecycle operations. Enforces governance at runtime.
- **OpenAPI** — The industry-standard machine-readable format (JSON/YAML) for describing HTTP APIs; governed by the OpenAPI Initiative (Linux Foundation). Current: 3.1.x.
- **OAS** — OpenAPI Specification (e.g., "OAS 3.1").
- **REST** — Representational State Transfer; the dominant architectural style for HTTP APIs (resources + methods + statelessness; Roy Fielding, 2000).
- **Richardson Maturity Model** — Leonard Richardson's (2008) 4-level classification of REST maturity: 0 POX, 1 resources, 2 HTTP verbs, 3 hypermedia.
- **Resource-oriented** — Designing APIs around nouns/resources with standard verbs, rather than function calls.
- **API-first** — The mindset/movement of designing the API contract before implementation; the contract is the first artifact.
- **Contract-first** — API-first operationalized: OpenAPI contract → review → mock → codegen → develop → test; contract as source of truth.
- **Style guide** — The design guideline document (naming, errors, versioning, conventions) APIs must follow.
- **Linting** — Automated checking of code/contracts against rules; for APIs: checking OpenAPI documents against a ruleset.
- **Spectral** — Stoplight's open-source OpenAPI/JSON linter; the de-facto standard for contract linting.
- **Stoplight** — Commercial API design platform (Studio, Spectral, Prism, docs).
- **Versioning** — The scheme for managing API versions; URI/header/media-type strategies; semver numbering.
- **Semver** — Semantic Versioning (MAJOR.MINOR.PATCH; MAJOR = breaking).
- **Breaking change** — A contract/behavior change that breaks existing consumers (removal, rename, type change, semantic change).
- **Deprecation** — Marking an API/version as superseded while it remains operational (Deprecation header, RFC 9745).
- **Sunset** — The scheduled removal date of a deprecated API (Sunset header, RFC 8594).
- **Lifecycle** — The stages of an API: design → develop → test → deploy → operate → retire.
- **Design review** — The gate where contracts are reviewed for conformance and quality.
- **Review board** — The body (stewards, architects, security) running design and breaking-change reviews.
- **API registry / catalog** — The system of record of APIs: owners, status, versions, docs, SLAs.
- **Catalog** — See API registry.
- **Backstage** — Open-source developer portal (Spotify/CNCF) used as a vendor-neutral API catalog and IDP front end.
- **Gateway** — The runtime edge that applies policies (authn, rate limits, routing) to API traffic.
- **Apigee** — Google's full-lifecycle API management platform.
- **Kong** — Open-source API gateway (with Konnect commercial platform).
- **AWS API Gateway** — Amazon's cloud API management service.
- **Azure APIM** — Microsoft's cloud API management service.
- **MuleSoft** — Salesforce's integration + API management platform (Anypoint).
- **Rate limiting** — Capping request rate per consumer (e.g., 100 req/s).
- **Quota** — Capping usage over a period (e.g., 10k calls/day) — the consumption contract.
- **OAuth 2.0** — RFC 6749 authorization framework; delegated scoped access.
- **OIDC** — OpenID Connect; identity layer on OAuth 2.0.
- **mTLS** — Mutual TLS; certificate-based mutual authentication.
- **API key** — Lightweight caller identifier; weak authn.
- **JWT** — JSON Web Token (RFC 7519); self-contained signed access tokens.
- **OWASP API Security Top 10** — The reference API threat list; 2023 edition current.
- **Observability** — Metrics, logs, tracing used to understand and govern running APIs.
- **API owner** — The accountable product owner of an API.
- **API steward** — The standards-enforcement function (reviews, guides, ruleset).
- **API guild** — The community of practice owning guidelines and reviews.
- **Center of Enablement (CoE)** — The funded team operating the API program (platform, tooling, standards).
- **Federated governance** — Platform/CoE sets standards and tooling; domain teams own their APIs.
- **PSD2** — EU Revised Payment Services Directive (2015/2366); open-banking access for TPPs; SCA.
- **CMA** — UK Competition and Markets Authority; the CMA9 Open Banking remedy (Jan 2018).
- **MAS** — Monetary Authority of Singapore; Finance-as-a-Service API Playbook (2016, with ABS), TRM guidelines, SGFinDex.
- **Open banking** — The regulated opening of bank data/payment initiation to third parties via APIs.

---

*End of guide. Feedback welcome via the repository.*
