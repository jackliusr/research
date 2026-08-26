# The Open Bank Project (OBP): A Comprehensive Guide to the Open-Source Banking-API Platform

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Financial Services Technology — Open-Source Banking-API Platforms, Open Banking / PSD2, Developer Portals, Platform Comparison
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Financial Services Software-Systems Guides — the **open-source banking-API platform** deep-dive. The open-source-core precedent is [Apache Fineract Guide](apache_fineract_guide.md) + [Fineract Database Models Guide](fineract_database_models_guide.md); the commercial-platform contrast is [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) + [Nasdaq Calypso Guide](nasdaq_calypso_guide.md); the commercial-core contrast is [Temenos T24 Guide](temenos_t24_guide.md) + [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md); the core-systems umbrella is [Core Banking Systems Guide](core_banking_systems_guide.md); the API-governance angle is [API Governance Guide](../technology/api_governance_guide.md); the OAuth/OIDC angle OBP uses is [Distributed Auth Guide](../technology/distributed_auth_guide.md); the open-source-vs-commercial analysis pattern is [Open-Source vs Commercial Insurance Guide](insurance_open_source_commercial_guide.md); the digital-bank contexts (where OBP-style APIs matter) are [Trust Bank Guide](trust_bank_guide.md), [GXS Bank Guide](gxs_bank_guide.md) and [MariBank Guide](maribank_guide.md).

**Verification convention used throughout: ✅ = verified in this research pass (official OBP/TESOBE site, the OpenBankProject/OBP-API GitHub repo, the OBP blog, press); ⚠ = flagged (vendor-stated, single-source, approximate, or not independently confirmed); unmarked = structural/industry knowledge presented as such. Consolidated claims-status notes are in §11. This guide was researched with a search-only web backend (no page extraction), so several fine-grained claims are flagged rather than asserted.**

**How to read this guide:** read §1 for what OBP is and who builds it, §2 for the API itself, §3 for the product family, §4 for the PSD2/XS2A angle, §5 for deployments, §6 for hackathons, §7 for the open-source business model, §8 for the comparison with alternatives, §9 for a worked developer-portal example at a mid-size bank, and §10 for the one-page summary. The glossary in §12 closes the guide.

---

## Table of Contents

1. [The OBP Overview](#1-the-obp-overview)
   - 1.1 [What OBP Is: The Open-Source Banking-API Platform](#11-what-obp-is-the-open-source-banking-api-platform)
   - 1.2 [The Company: TESOBE, Berlin](#12-the-company-tesobe-berlin)
   - 1.3 [The Founder: Simon Redfern](#13-the-founder-simon-redfern)
   - 1.4 [The Overview Table](#14-the-overview-table)
   - 1.5 [Positioning in the Series](#15-positioning-in-the-series)
2. [The OBP API](#2-the-obp-api)
   - 2.1 [The API Standard and Versioning](#21-the-api-standard-and-versioning)
   - 2.2 [The API Feature Set: Accounts, Transactions, Customers, KYC — and Beyond](#22-the-api-feature-set-accounts-transactions-customers-kyc--and-beyond)
   - 2.3 [The API Table](#23-the-api-table)
   - 2.4 [Authentication, Authorisation and Consent](#24-authentication-authorisation-and-consent)
   - 2.5 [Architecture: Views, Connectors, and the Connector to the Core](#25-architecture-views-connectors-and-the-connector-to-the-core)
3. [The Products](#3-the-products)
   - 3.1 [OBP API](#31-obp-api)
   - 3.2 [API Explorer](#32-api-explorer)
   - 3.3 [Developer Portal](#33-developer-portal)
   - 3.4 [Sandbox](#34-sandbox)
   - 3.5 [The Rest of the Platform: Gateway, Manager, Auth & Consent, API Agent](#35-the-rest-of-the-platform-gateway-manager-auth--consent-api-agent)
   - 3.6 [The Products Table](#36-the-products-table)
4. [The PSD2 Angle](#4-the-psd2-angle)
   - 4.1 [PSD2 and XS2A in One Paragraph](#41-psd2-and-xs2a-in-one-paragraph)
   - 4.2 [How OBP Maps the XS2A Flows](#42-how-obp-maps-the-xs2a-flows)
   - 4.3 [The PSD2 Table](#43-the-psd2-table)
   - 4.4 [Flags on the PSD2 Claims](#44-flags-on-the-psd2-claims)
5. [The Deployments](#5-the-deployments)
   - 5.1 [Verified Deployments: NMB (Tanzania), Intercam (Mexico), the Public Sandbox](#51-verified-deployments-nmb-tanzania-intercam-mexico-the-public-sandbox)
   - 5.2 [The Client List: Vendor-Stated](#52-the-client-list-vendor-stated)
   - 5.3 [The ABN AMRO Question — Flagged](#53-the-abn-amro-question--flagged)
   - 5.4 [The Deployments Table](#54-the-deployments-table)
6. [The Hackathons](#6-the-hackathons)
   - 6.1 [Hackathons as a Go-To-Market Engine](#61-hackathons-as-a-go-to-market-engine)
   - 6.2 [The Hackathon Suite](#62-the-hackathon-suite)
   - 6.3 [The Hackathon Table](#63-the-hackathon-table)
7. [The Open-Source Model](#7-the-open-source-model)
   - 7.1 [The License: AGPLv3 + Commercial — Not Apache](#71-the-license-agplv3--commercial--not-apache)
   - 7.2 [The Community Model](#72-the-community-model)
   - 7.3 [The Business Model: Open Core With Services](#73-the-business-model-open-core-with-services)
   - 7.4 [The Model Table](#74-the-model-table)
8. [The Comparison: OBP vs the Alternatives](#8-the-comparison-obp-vs-the-alternatives)
   - 8.1 [OBP vs General-Purpose API Platforms](#81-obp-vs-general-purpose-api-platforms)
   - 8.2 [OBP vs Open-Source Financial Platforms](#82-obp-vs-open-source-financial-platforms)
   - 8.3 [OBP vs Commercial Open-Banking / API Vendors](#83-obp-vs-commercial-open-banking--api-vendors)
   - 8.4 [The Comparison Table](#84-the-comparison-table)
9. [Worked Example: A Mid-Size Bank's Developer Portal on OBP](#9-worked-example-a-mid-size-banks-developer-portal-on-obp)
   - 9.1 [The Scenario: Meridian Bank](#91-the-scenario-meridian-bank)
   - 9.2 [The OBP Deployment Design](#92-the-obp-deployment-design)
   - 9.3 [Phasing the Programme](#93-phasing-the-programme)
   - 9.4 [The Lessons](#94-the-lessons)
10. [The Summary: One Page](#10-the-summary-one-page)
11. [Consolidated Verification Notes](#11-consolidated-verification-notes)
12. [Glossary](#12-glossary)
13. [Primary Sources and References](#13-primary-sources-and-references)

---

## 1. The OBP Overview

### 1.1 What OBP Is: The Open-Source Banking-API Platform

**✅ Verified.** The Open Bank Project (OBP) is, in the words of its own GitHub description, *"an open source RESTful API platform for banks that supports Open Banking, XS2A, PSD2 and Open Finance through access to accounts, transactions, counterparties, payments, entitlements and metadata — plus a host of internal banking and management APIs"* (OpenBankProject/OBP-API, verified August 2026). The corporate site (openbankproject.com) leads with the same positioning: *"Build, manage, and secure APIs in compliance with open banking standards worldwide"* — one API platform aimed at banks (open-banking compliance and beyond), regulators (regulatory advisory, regulatory sandboxes), and developers.

The crucial positioning point for a solution architect: **OBP is not a core banking system.** It does not post entries, compute interest, or keep the ledger. It is the **API layer** that sits *in front of* a bank's existing core and systems of record, exposing accounts, transactions, customers, KYC documents, payments and products through a standardised, versioned REST API, and translating third-party / partner / regulatory standards (PSD2 XS2A, UK Open Banking, Berlin Group) onto that layer. That makes it the natural sibling of the API-first layers discussed in [Core Banking Systems Guide](core_banking_systems_guide.md) and the natural contrast to both the open-source core [Apache Fineract Guide](apache_fineract_guide.md) and the commercial cores [Temenos T24 Guide](temenos_t24_guide.md) / [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md).

Key verified facts:

- **Open source:** the core platform is public on GitHub (`OpenBankProject/OBP-API`), written in **Scala**, created **20 November 2012**, default branch `develop`, with (as of August 2026) **1,736 stars, 477 forks, 209 open issues** — a real, inspectable, forkable codebase, not a marketing wrapper. ⚠ The community metrics are modest by open-source standards (see §7.2).
- **Domain breadth:** the catalogue runs from accounts and payments to KYC, FX and products (verified on the OBP platform page), plus internal banking and management APIs (users, roles, entitlements, views, consents, branches, ATMs).
- **Standards support:** the platform implements the **Berlin Group NextGenPSD2 XS2A** interface (API standard `Berlin Group`, version `v1.3`, seen both in OBP's own site payload examples and in the release notes' Berlin Group references) and **UK Open Banking** consent scoping, alongside OBP's own native API standard (versioned `OBPv1.x` → `OBPv5.x`).
- **Transparency heritage:** the project's founding idea (2010-2012) was financial transparency — account holders sharing *configurable views* of their transaction data — which is why "views" remain a core OBP abstraction today.

### 1.2 The Company: TESOBE, Berlin

**✅ Verified.** OBP is built and commercially backed by **TESOBE GmbH** — the name is an acronym for **T**echnical **S**olutions **B**erlin — a Berlin-based software consultancy founded in **2005** by Simon Redfern (verified via Crunchbase, FintechForum interviews, and OBP site footer: *"© Copyright 2010-2025 TESOBE GmbH"*). The OBP sandbox page states it plainly: *"Pioneering open banking concepts and technologies since 2010, the Open Bank Project is the leading open source Open Banking technology built for banks by the Berlin-based consulting firm TESOBE."*

TESOBE's role is three-fold and visible across OBP's own site:

1. **Original developer and maintainer** of the OBP-API codebase and the whole OBP product family (API Explorer, Developer Portal, Sandbox, Gateway, Manager).
2. **Commercial licensor** — the OBP-API is dual-licensed AGPLv3 + commercial licenses *from TESOBE GmbH* (see §7.1), and TESOBE sells services around it: sandbox-as-a-service, training, API strategy, regulatory advisory, hackathon delivery.
3. **Consultancy** — the FinovateEurope 2018 profile lists clients including BNP Paribas, Societe Generale, RBS, Emirates NBD, UniCredit and others (⚠ vendor-stated, see §5.2), and TESOBE claims a hand in open-banking regulation itself (§1.4).

### 1.3 The Founder: Simon Redfern

**✅ Verified (identity and role) — ⚠ flagged (any deeper biographical claims).** Simon Redfern is the **founder and CEO of the Open Bank Project and CEO of TESOBE**. Verified from multiple independent sources: Crunchbase (*"Simon Redfern is the current Founder and CEO of Open Bank Project. In 2005 Simon started a technical consultancy called TESOBE — Technical Solutions Berlin"*), a FintechForum interview (*"I'm Simon Redfern, programmer, composer, CEO of TESOBE (an agile web/mobile agency based in Berlin) and founder of the Open Bank Project. I started programming 32 years ago, and have worked on database driven web applications since 1996"*), The Next Web's early coverage (*"Open Bank Project (OBP) is being spearheaded by TESOBE founder and CEO Simon Redfern"*), and the FinovateEurope 2018 stage appearance alongside COO Ismail Chaib.

What is **not** independently verifiable in this pass: precise dates of OBP's founding (the site says "since 2010" while the GitHub repo was created in 2012), the exact size of TESOBE, funding history, and the details of Redfern's earlier career. The "composer" detail and 1996-onwards web-programming history are single-source (his own interviews). Treat the founder's *role* as solid and the *biography* as flagged.

### 1.4 The Overview Table

| Aspect | Description | Status |
|---|---|---|
| **What it is** | Open-source RESTful API platform for banks: accounts, transactions, counterparties, payments, entitlements, metadata + internal banking/management APIs | ✅ GitHub repo description |
| **Layer** | API layer in front of core banking / systems of record — not a core itself | ✅ structural |
| **Standards** | Own OBP API standard (v1.x–v5.x) + Berlin Group NextGenPSD2 XS2A (v1.3) + UK Open Banking scopes | ✅ repo release notes, site payloads |
| **Licence** | Dual: AGPLv3 + commercial from TESOBE GmbH (⚠ *not* Apache 2.0 — see §7.1) | ✅ repo README |
| **Company** | TESOBE GmbH, Berlin; founded 2005 ("Technical Solutions Berlin") | ✅ Crunchbase / OBP site |
| **Founder** | Simon Redfern — founder & CEO of OBP, CEO of TESOBE | ✅ multiple sources; ⚠ bio detail |
| **Codebase** | Scala, GitHub since 20 Nov 2012, ~1.7k stars / ~477 forks (Aug 2026) | ✅ GitHub API |
| **Regulatory claim** | "The team that inspired the first Open Banking regulation in the world" | ⚠ vendor claim, partially supported (see below) |
| **Use cases** | Open-banking compliance, API sandboxes, hackathons, API strategy, blockchain banking, API Agent (natural-language API access) | ✅ OBP site solutions pages |

On the regulatory claim: OBP's blog post *"How to Regulate Open Banking"* (co-authored by Redfern) states that the OBP team's *"ideas about openness, transparency and data sovereignty found echo in both the European Commission and the UK Treasury"*, and Redfern's Medium posts document OBP's work with the British Embassy in Mexico and the UK Prosperity Fund on the Mexican Open Banking Working Group. That supports an *advisory/inspirational* role, not a verifiable claim of drafting any regulation — flagged accordingly.

### 1.5 Positioning in the Series

This guide is the dedicated deep-dive on the open-source **banking-API platform** in the repo's open-source + platform series:

- **Open-source core precedent:** [Apache Fineract Guide](apache_fineract_guide.md) + [Fineract Database Models Guide](fineract_database_models_guide.md) — Fineract *is* the core with an API; OBP *is* the API over a core. Together they show the two open-source answers to banking software.
- **Commercial platform contrast:** [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) + [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) — closed-source, licence-fee platforms for trading/back-office; OBP is the open-source API-layer platform.
- **Commercial core contrast:** [Temenos T24 Guide](temenos_t24_guide.md) + [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md) — the cores OBP typically fronts via connectors.
- **Open-source-vs-commercial analysis pattern:** [Open-Source vs Commercial Insurance Guide](insurance_open_source_commercial_guide.md) — the same ✅/⚠ verification and vendor-landscape method applied to insurance.
- **API governance and security:** [API Governance Guide](../technology/api_governance_guide.md) and [Distributed Auth Guide](../technology/distributed_auth_guide.md) — the discipline OBP APIs still need from the bank.
- **Digital-bank contexts:** [Trust Bank Guide](trust_bank_guide.md), [GXS Bank Guide](gxs_bank_guide.md), [MariBank Guide](maribank_guide.md) — modern digital banks whose partner APIs look like what OBP productises.

### 1.6 A Brief Timeline (Verified Where Marked)

| Date | Milestone | Status |
|---|---|---|
| **2005** | TESOBE ("Technical Solutions Berlin") founded by Simon Redfern | ✅ Crunchbase |
| **~2010** | OBP "pioneering open banking concepts" begins (site's own claim); TESOBE starts powering hackathons | ⚠ site claim |
| **2012-11-20** | OBP-API repo created on GitHub | ✅ GitHub API |
| **2013** | Level39 Canary Wharf (London) hackathon organised by TESOBE | ✅ Finextra |
| **2015-2016** | OBP ideas "found echo in the European Commission and the UK Treasury" (blog); OBP-API resource-documented versions v1.2.1/v1.3.0 era | ⚠ blog; ✅ repo history |
| **2017** | OBP International Hackathon with BNP Paribas, Berlin | ⚠ TESOBE site |
| **2018** | FinovateEurope presentation (Redfern + COO Ismail Chaib); "30+ global banks" client claim | ⚠ vendor-stated |
| **2023-06** | Corporate consents / multi-sig blog (Nordea, ING, Deutsche Bank, ABN AMRO, OP benchmark) | ✅ OBP blog |
| **2024-2025** | API Agent (natural-language API interaction); conversational AI in sandbox | ✅ OBP site |
| **2026-08** | Active release notes (SCA/consent hardening, Berlin Group IG §4.11, Open Corridor); live NMB instance on OBPv5.1.0 | ✅ repo + NMB explorer |

---

## 2. The OBP API

### 2.1 The API Standard and Versioning

**✅ Verified (existence and version line); ⚠ flagged (exact endpoint count).** The OBP API is a **RESTful, JSON-based API** with a long, explicit version history. Public evidence of the version line:

- GitHub repo created **2012**; the OBP public API Explorer historically exposed `OBPv1.2.1`, `OBPv1.3.0`, and later `OBPv2.x`, `OBPv3.x`, `OBPv4.x`.
- The OBP site's own payload examples today use paths under **`/obp/v4.0.0/...`** (e.g., transaction-request challenge links: `/obp/v4.0.0/banks/BANK_ID/accounts/...`).
- A **live third-party deployment** — NMB Bank's Tanzania sandbox API Explorer (obp-apiexplorer-sandbox.nmbbank.co.tz) — is served with **`OBPv5.1.0`** operations. So the current major line is **v5.x** as of 2026.
- The repo's `release_notes.md` is actively maintained (entries dated **13 August 2026**), confirming the project is alive and versioned.

**Endpoint count — flagged.** The official OBP sandbox page states the platform ships *"over 500 banking and management API endpoints"*; the platform page describes the catalogue as running *"from accounts & payments to KYC, FX, and Products"*. ⚠ This "500+" figure is a vendor-stated, configuration-dependent number (endpoints are gated per API version and per deployment via `api_enabled_versions` / `api_enabled_endpoints` properties — see the repo FAQ), so do not quote it as an exact contract. Older marketing materials used other counts; the honest statement is: *several hundred endpoints across the v1–v5 version families, with the exact surface enabled per deployment.*

**API design characteristics (verified from the repo and docs):**

- **Resource-documented:** every endpoint is registered in a resource-docs registry (`allResourceDocs`) that drives the API Explorer and generated documentation — OBP is *documentation-first* in design.
- **Versioned in the URL:** `/obp/v4.0.0/...` style paths; versions can be enabled/disabled per deployment (⚠ per-version endpoint inventories are large and overlapping, so "N endpoints per version" is not a clean figure).
- **Standards-mapped:** alongside the native standard, the same platform serves **Berlin Group** (`/berlin-group/v1.3/...`) and **UK Open Banking** interfaces — verified in the release notes (Berlin Group Implementation Guidelines references, e.g. IG §4.11 resource scoping; UK consent GET/DELETE scoping) and in site payloads (`"api_standard": "Berlin Group", "api_version": "v1.3"`).
- **Evolving stack:** the FAQ confirms the platform has migrated request dispatch from the Lift web framework to **native http4s** (`HttpRoutes[IO]`), with Lift retained for the resource-docs registry — i.e., a modern Scala/functional stack under the hood.
- **OpenAPI-adjacent:** the API Explorer consumes the resource documentation to render browsable, testable API docs (verified in the OpenBankProject/API-Explorer repo description).

### 2.2 The API Feature Set: Accounts, Transactions, Customers, KYC — and Beyond

**✅ Verified (feature existence; the OBP platform page and docs enumerate exactly these domains).** The four headline features the task asked to verify are all real, public parts of the OBP API:

- **Accounts** — bank, account and balance endpoints: list banks (`GET /banks`), get account details, account balances, and the famous OBP abstraction **views** (public / private / shared / custom views) that let account holders expose *selected fields* of an account to selected parties. This is the transparency mechanism OBP was founded on.
- **Transactions** — transaction history endpoints (list/get transactions per account and view), transaction types, and **transaction requests** (the OBP payment-initiation primitive: a request object with `to_sandbox_tan`, `to_sepa`, `to_counterparty`, `to_transfer_to_phone`, `to_transfer_to_atm`, `to_transfer_to_account`, `to_sepa_credit_transfers` variants — all visible in the OBP site's own sample payloads), including challenge/SCA steps and charges.
- **Customers** — customer endpoints (get/create customers for a user, customer attributes) and **customer-facing relationships**: account access, entitlements, user↔customer↔account linkage. The repo FAQ even points to `getCustomersForUser` in `APIMethods300.scala` as the canonical endpoint-creation example.
- **KYC** — a full KYC endpoint family: get/create **KYC documents**, KYC checks, KYC media and KYC statuses (the operation *"Get Customer KYC Documents"* is visible on NMB's live OBPv5.1.0 API Explorer).

**Beyond the headline four** (verified on the platform page and docs): counterparties, payments, **entitlements** (fine-grained role/right grants), **metadata** (user-generated comments, tags, images on accounts/transactions — another transparency-era feature), **FX rates**, **products** (banking products catalogue), branches and ATMs, cards, direct debits and standing orders, consents, and a large **internal banking + management API** set (users, roles, entitlements, views, consents, sandbox/portal administration).

### 2.3 The API Table

| Feature | Description | Notes |
|---|---|---|
| **Accounts** | Banks, accounts, balances; per-view field-level exposure | Views are the OBP differentiator: same account, different field sets per audience |
| **Transactions** | History listing/getting per account+view; transaction types | Read side of the transparency story |
| **Transaction requests** | Payment initiation: SEPA, TAN, counterparty, phone, ATM, account transfers | Multi-step with challenges (SCA) and charges; the PIS primitive |
| **Customers** | Customer records, attributes, customer↔user↔account links | Foundation for onboarding and corporate consents |
| **KYC** | KYC documents, checks, media, statuses | Directly reusable for onboarding/regulatory workflows; ⚠ depth varies by deployment |
| **Counterparties** | Managed third-party payees | Named/credited counterparty abstraction |
| **Metadata** | Comments, tags, images on accounts/transactions | Legacy of the "financial transparency" founding idea |
| **Entitlements & roles** | Fine-grained rights on endpoints/views | The authorisation spine; roles gate endpoint access |
| **Consents** | Consent objects with statuses (INITIATED→…), API standard + version fields | Bridges OBP-native and Berlin Group/UK consent flows |
| **FX & products** | FX rates, banking product catalogue | Platform page lists FX and Products in the catalogue |
| **Branches/ATMs/cards** | Location and card data endpoints | Common open-banking/PSD2-adjacent surface |
| **Internal banking & management APIs** | Users, roles, views administration, sandbox setup | Distinguishes OBP from pure customer-facing API gateways |

### 2.4 Authentication, Authorisation and Consent

OBP's security model (structural knowledge, consistent with the repo's `docs/` — e.g. `MTLS.md`, `MTLS_TOPOLOGIES.md`, IdP docs — and with the consent machinery in the release notes):

- **OAuth 2.0 / OIDC-style flows** for third-party app access, with **direct login** for first-party apps; the platform supports **mTLS** (documented topologies in the repo docs) — the transport security required by PSD2 RTS for TPP↔ASPSP communication.
- **Consents** are first-class objects: created per API standard (`Berlin Group`, `UK`, OBP-native), carrying `api_version`, status (`INITIATED` …), and JWT evidence in payloads. The release notes (Aug 2026) show active hardening: consent resolution now bound to the **Consumer (TPP) that lodged it**, SCA-front-end consumer IDs required for Redirect SCA deployments, and Berlin Group IG §4.11 / UK scoping of consent reads and revocations.
- **SCA challenges** are part of transaction-request flows (challenge objects with `allowed_attempts`, `challenge_type: OBP_TRANSACTION_REQUEST_CHALLENGE`, challenge links) — i.e., strong customer authentication is *modelled in the API*, not bolted on.
- **Entitlements** grant fine-grained rights (endpoint + view + role combinations), the OBP answer to "who may call what, on whose data".

Cross-reference [Distributed Auth Guide](../technology/distributed_auth_guide.md) for the general OAuth2/OIDC design space and [API Governance Guide](../technology/api_governance_guide.md) for how a bank governs this surface.

### 2.5 Architecture: Views, Connectors, and the Connector to the Core

Three architectural ideas make OBP coherent (structural, from the repo and docs):

1. **Views as the data-sharing primitive.** Every account exposes multiple views; each view is a *filtered projection* (fields, permissions) of the account. A customer shares their "transactions view" with an accountant, their "balance view" with a budgeting app, nothing with the public — unless they create a public view. All downstream features (consents, transaction requests, metadata) operate on views. This is OBP's core abstraction and the cleanest way to reason about its API.
2. **The connector pattern.** OBP is deliberately decoupled from the bank's systems of record: a **connector** (data-access layer) maps OBP's domain model (banks, accounts, transactions, customers, products) onto the core's real data. Evidence of this pattern in the wild: NMB's API Explorer displayed *"Possible Errors: OBP-50000: Unknown Error. no connector set."* — the platform ships without a core behind it, and *you* plug the connector in. OBP provides reference/example connectors (e.g., Mapper-based, plus the sandbox's own in-memory/database data source).
3. **Standards as profiles over one model.** Berlin Group XS2A and UK Open Banking are not separate products — they are interface profiles over the same banks/accounts/transactions/consents model. That is the architectural bet that makes OBP "comply with many leading Open Banking standards" from one platform.

### 2.6 Key OBP Concepts at a Glance

| Concept | What it is | Why it matters |
|---|---|---|
| **View** | A filtered, permissioned projection of an account (fields + rights) | The data-sharing primitive: same account, different field sets per audience (public/private/shared/custom) |
| **Consent** | A first-class object: standard, version, status (INITIATED→…), JWT evidence, TPP scoping | The compliance spine; bridges OBP-native, Berlin Group and UK flows |
| **Entitlement / Role** | Fine-grained grant of rights on endpoints/views to a user | Authorisation without admin firefighting; gates endpoint access per deployment |
| **Transaction Request** | The payment-initiation object: type (SEPA/TAN/counterparty/phone/ATM/account), amount, challenges, charges | OBP's PIS primitive; multi-step and SCA-capable |
| **Consumer / App** | Registered third-party application with keys | Developer-portal identity; consents now resolve against the lodging Consumer (Aug 2026 hardening) |
| **Connector** | The data-access layer between OBP and the bank's core/systems of record | The integration contract — the bank's main implementation work |
| **Resource docs** | Registry of all endpoints (methods, params, errors) driving the Explorer | Documentation-first API design; powers in-browser testing |
| **API version** | URL-scoped version line (OBPv1.x–5.x); per-version endpoint enablement | Safe evolution; deployments choose their surface |

### 2.7 Two Walkthrough Flows

**Flow 1 — AIS (account information) under a Berlin Group consent.** (Structural reconstruction from OBP's consent/SCA machinery; the artefacts named are all verified — consent payloads, `POST /berlin-group/v1.3/consents/{consentId}/authorisations`, IG §4.11 scoping, Redirect SCA properties.)

1. TPP (AISP) registers on the Developer Portal → gets a Consumer key.
2. AISP creates an account-access consent (`POST /berlin-group/v1.3/consents`); the consent is scoped *to the TPP that created it* (IG §4.11).
3. PSU authenticates — embedded, or via **Redirect** at the ASPSP's own approval screen (`sca_front_end_consumer_ids` identifies that screen's Consumer; without it, OBP-35015 `ConsentDoesNotMatchConsumer` blocks SCA — the Aug 2026 behaviour change).
4. SCA challenges complete → consent reaches an authorised status.
5. AISP reads accounts/balances/transactions through consent-gated views; every read is entitlement- and consent-checked per request.

**Flow 2 — PIS (payment initiation) via transaction request.** (Artefacts verified in the OBP site's own `to_sepa_credit_transfers` sample payload.)

1. PISP creates a transaction request: `to_sepa_credit_transfers` with debtor IBAN, instructed amount, creditor IBAN/name — or a domestic variant (`to_transfer_to_account`, `to_transfer_to_phone`, `to_counterparty`).
2. The request carries a `challenges` array (`challenge_type: OBP_TRANSACTION_REQUEST_CHALLENGE`, `allowed_attempts`, challenge link at `/obp/v4.0.0/banks/{BANK_ID}/accounts/{ACCOUNT_ID}/{VIEW_ID}/transaction-request-types/.../transaction-requests/{ID}/challenge`).
3. PSU answers the challenge (SCA) → status progresses (sample shows `COMPLETED`) → the charge block records the fee → the payment executes via the connector into the core/payments hub.

## 3. The Products

OBP is a *family* of products around the core API — the same pattern as any serious API-platform vendor, but open source at the centre. All components below are verified on OBP's site (sandbox, hackathon and platform pages) and GitHub org.

### 3.1 OBP API

**✅ Verified.** The open-source core: the REST API platform itself (`OpenBankProject/OBP-API`), which can be run by the bank on its own infrastructure, in the cloud, or consumed as TESOBE-operated environments. It is the engine everything else hangs off: endpoints, versions, consents, entitlements, views, connectors, sandbox data generation. Dual-licensed AGPLv3/commercial (§7.1). Deployable from source (the repo ships build/run scripts and Docker-based guides — including community walkthroughs like "Open Bank Project API Local Install").

### 3.2 API Explorer

**✅ Verified.** A separate open-source web app (`OpenBankProject/API-Explorer` — *"A Scala / Liftweb application that consumes the OBP-API resource documentation so that developers can browse and interact with the OBP REST API endpoints"*). It is the developer-facing catalogue: browse the API catalogue and documentation, **request roles/entitlements, and test endpoints directly from the browser** (verified wording on OBP's platform page). Public instances exist (e.g., the OBP sandbox's explorer; NMB's explorer runs one against OBPv5.1.0). Think of it as OBP's answer to Swagger UI, productised per bank.

### 3.3 Developer Portal

**✅ Verified.** The onboarding surface: developers **register an account, connect their applications, and start consuming the bank's APIs** (verified on the hackathon-suite page). The sandbox offer includes the portal with **self-service registration and onboarding**, prebuilt auth/consent screens and flows, and the portal is where API keys/consumers, app registrations and role requests live. This is the "developer portal" product a bank white-labels for its own ecosystem.

### 3.4 Sandbox

**✅ Verified.** The controlled technical environment for testing without production data. Verified contents (sandbox page): **a developer portal, over 500 banking and management API endpoints, localised synthetic data, and a collection of SDKs** to accelerate development; plus **self-service registration/onboarding, a real-world persistent data model with constraints, region-specific data generation, compliance with leading Open Banking standards, prebuilt auth & consent screens and flows, and conversational AI for API discovery and support**. TESOBE operates public sandboxes (the OBP public sandbox, NMB's Tanzania sandbox, Intercam's Mexico sandbox — §5) and sells bank-branded sandboxes as a service.

### 3.5 The Rest of the Platform: Gateway, Manager, Auth & Consent, API Agent

Verified on the hackathon-suite and platform pages:

- **API Gateway** — traffic control, access control, rate limiting for the sandbox or live environment.
- **API Manager** — administration: view connected applications, grant roles and entitlements, manage the environment.
- **Auth & Consent** — prebuilt consent and authentication flows "securing APIs according to regulation" (Berlin Group/UK/OBP consent screens).
- **API Agent** — the 2024-era addition: *"interact with APIs and data using natural language"* — a conversational layer over the same catalogue (also marketed as the sandbox's conversational AI for API discovery).
- **Training and regulatory advisory** — services, not software: API training courses and the "Regulatory Advisory — from the team that inspired the first Open Banking regulation in the world" offer (⚠ vendor claim).

### 3.6 The Products Table

| Product | Function | Notes |
|---|---|---|
| **OBP API** | The open-source REST API platform: endpoints, versions, consents, entitlements, views, connectors | Scala; AGPLv3 + commercial; run it yourself or TESOBE-hosted |
| **API Explorer** | Browse catalogue & docs, request roles, test endpoints in-browser | Separate OSS repo; powered by the resource-docs registry |
| **Developer Portal** | Self-service registration, app onboarding, API-key/consumer management, consent screens | The white-label developer face of a bank's open-banking programme |
| **Sandbox** | Controlled environment: 500+ endpoints, synthetic localised data, SDKs, prebuilt auth/consent flows, conversational AI | Compliance sandbox and innovation sandbox variants; ⚠ "500+" vendor figure |
| **API Gateway** | Traffic control, access control, rate limiting | Secures sandbox or live APIs |
| **API Manager** | Admin of apps, roles, entitlements, environments | The ops console |
| **Auth & Consent** | Prebuilt authentication + consent flows per regulation | Berlin Group / UK / OBP-native consent screens |
| **API Agent** | Natural-language interaction with APIs and data | AI layer; conversational API discovery |
| **Services** | Training, API strategy, hackathon delivery, regulatory advisory | The TESOBE consulting business around the code |

---

## 4. The PSD2 Angle

### 4.1 PSD2 and XS2A in One Paragraph

**✅ Verified (context).** The EU's revised **Payment Services Directive (PSD2, EU 2015/2366)**, with the EBA Regulatory Technical Standards, obliged European banks (ASPSPs) to expose account access to licensed third-party providers (TPPs): **AISPs** (account information service providers) and **PISPs** (payment initiation service providers). **XS2A — "access to account"** — is the umbrella term for this interface, and the industry-standard implementation framework is the **Berlin Group NextGenPSD2 "Access to Account" (XS2A) framework**, published by the Berlin Group (the body also behind SEPA standards). The Berlin Group spec defines REST resources for AIS (accounts, balances, transactions), PIS (payment initiations, authorisations), consents, and the SCA/challenge choreography, plus the mutual TLS and eIDAS certificate requirements.

### 4.2 How OBP Maps the XS2A Flows

**✅ Verified (that OBP implements Berlin Group interfaces); ⚠ flagged (the "mapping" is OBP's own engineering, not a formal certification).** OBP does not bolt PSD2 on; it *maps* the XS2A interface onto its own domain model, which is why the same platform serves native OBP, Berlin Group and UK interfaces:

- **AIS ↔ accounts/transactions + views + consents.** The XS2A account-information consent maps onto OBP consent objects (with `api_standard: "Berlin Group"`, `api_version: "v1.3"` — verified in OBP's own site payloads), and the account/balance/transaction reads map onto OBP's bank/account/transaction endpoints behind consent-gated views.
- **PIS ↔ transaction requests.** XS2A payment initiation (SEPA credit transfers, etc.) maps onto OBP's **transaction-request** flow — the site's own example payload shows a full `to_sepa_credit_transfers` request with debtor/creditor IBAN, instructed amount, challenge objects and status `COMPLETED`. OBP transaction requests even carry a `challenges` array with `allowed_attempts` and challenge-type — the SCA choreography in API form.
- **SCA ↔ challenges.** XS2A's strong-customer-authentication steps (embedded or **Redirect**) map onto OBP's challenge machinery; the Aug 2026 release notes discuss **Berlin Group Redirect SCA** deployments and the `sca_front_end_consumer_ids` property — i.e., OBP actively maintains the Redirect flavour where the PSU authenticates at the ASPSP's own front end.
- **Consent lifecycle per IG §4.11.** The release notes explicitly track the Berlin Group Implementation Guidelines' rule that a dynamically created resource is scoped *to the TPP that created it* — evidence of genuinely standards-conformant consent handling, not just look-alike endpoints.
- **mTLS + certificates.** The repo ships mTLS documentation (`docs/MTLS.md`, `docs/MTLS_TOPOLOGIES.md`) — the transport security layer PSD2 RTS requires.

The honest caveat: "PSD2-compliant" in OBP materials means *implements the Berlin Group XS2A framework and UK Open Banking interfaces*; a deployment's regulatory standing still depends on the bank's own certification, eIDAS certificate usage, and local supervisory treatment. ⚠ No independent third-party compliance certification was found in this pass.

### 4.3 The PSD2 Table

| PSD2/XS2A element | OBP mapping | Evidence |
|---|---|---|
| **XS2A account information (AIS)** | Accounts, balances, transactions behind consent-gated views | OBP docs/platform page; consent payloads with `api_standard: Berlin Group` |
| **XS2A payment initiation (PIS)** | Transaction-request flow incl. `to_sepa_credit_transfers` | OBP site sample payloads |
| **Consents** | First-class consent objects (status INITIATED→…, API standard + version fields) | Site payloads; release notes |
| **SCA / challenges** | Challenge objects, `allowed_attempts`, challenge links; embedded + **Redirect** SCA | Site payloads; Aug 2026 release notes |
| **Consent scoping** | Resource scoped to the lodging TPP (Berlin Group IG §4.11); UK consent read/revoke scoping | Release notes (13/08/2026) |
| **mTLS / eIDAS-adjacent transport** | mTLS topologies documented in repo | `docs/MTLS.md`, `docs/MTLS_TOPOLOGIES.md` |
| **Standards profile** | Berlin Group NextGenPSD2 v1.3 base; UK Open Banking scopes; OBP-native standard | Site payloads; release notes; live NMB explorer |

### 4.4 Flags on the PSD2 Claims

- ✅ OBP implements Berlin Group XS2A interfaces and UK Open Banking consent scoping — verified in the codebase's release notes and site payloads.
- ⚠ "PSD2-compliant platform" is a vendor characterisation; the mapping quality and regulatory standing are deployment-specific.
- ⚠ The XS2A version served is stated as `v1.3` in OBP's own payloads; Berlin Group's framework has evolved over time, so a bank should pin the exact IG version for its jurisdiction.

---

## 5. The Deployments

### 5.1 Verified Deployments: NMB (Tanzania), Intercam (Mexico), the Public Sandbox

**✅ Verified (official site + live evidence).** OBP's own sandbox page names three open sandboxes:

1. **NMB Bank, Tanzania** — *"The first fintech sandbox in Tanzania."* The page quotes NMB's Paul Shilla, Senior Specialist Digital Lab: *"We're extremely happy to have TESOBE as our partner on this journey. With the Open Bank Project sandbox powering our initiative, NMB will be able to catalyse innovation in Tanzania and beyond."* Corroborated by the **live NMB API Explorer at obp-apiexplorer-sandbox.nmbbank.co.tz serving OBPv5.1.0** (verified in this pass) — a real, running, production-facing OBP instance.
2. **Intercam, Mexico** — *"Compliant sandbox for Mexico's Intercam."* Consistent with OBP's Mexico regulatory work (Redfern's posts on the Mexican Open Banking Working Group with the British Embassy and UK Prosperity Fund). ⚠ No independent technical review of the Intercam instance was found in this pass.
3. **The OBP public sandbox** — TESOBE's own demo/data environment, used for evaluation and hackathons.

### 5.2 The Client List: Vendor-Stated

**⚠ Flagged.** The FinovateEurope 2018 presentation (Simon Redfern, CEO TESOBE & Founder OBP; Ismail Chaib, COO) states: *"TESOBE works with some of the largest banks in Europe and abroad. Our clients include over 30 global banks such as BNP Paribas, Societe Generale, RBS, Emirates NBD, UniCredit and others."* The OBP hackathon page adds Societe Generale, BNP Paribas, Santander, and HSBC as hackathon clients, and the sandbox page says *"Some of the largest Financial Institutions have used OBP technology."* These are credible marketing claims with named institutions, but **no public case studies or contracts** were found in this pass to independently confirm the *scope* (pilot vs production vs hackathon) of each relationship. Treat "30+ banks" as vendor-stated.

### 5.3 The ABN AMRO Question — Flagged

**⚠ ABN AMRO is NOT verified as an OBP platform deployment.** The task premise that ABN AMRO deployed OBP could not be confirmed from any primary source in this pass. What *is* verified:

- ABN AMRO appears in OBP's own blog *"Managing Corporate Consents with OBP"* (29 June 2023) — but as one of **five banks benchmarked for their corporate APIs** (Nordea, ING, Deutsche Bank, ABN AMRO, OP Financial Group), i.e., an *example OBP studied*, not an OBP customer.
- ABN AMRO runs its **own developer portal** (developer.abnamro.com) and its **own** hackathons ("Beyond Banking Days", 2017-2018 — verified via Finextra, Crowdfund Insider, FinancialIT coverage) — neither of which is attributed to TESOBE/OBP in the sources found.
- The LinkedIn hit "Projectmedewerker Open Bank Project · ABN AMRO Bank" is a personal-profile artifact (1992-1997 dates, i.e., unrelated to the OBP project) and is not evidence of an OBP deployment.

Conclusion: **flag the ABN AMRO deployment claim as unverified; do not repeat it as fact.** The verified deployment anchors are NMB (Tanzania), Intercam (Mexico) and the OBP public sandbox, plus the vendor-stated client list in §5.2.

### 5.4 The Deployments Table

| Deployment | Context | Notes |
|---|---|---|
| **NMB Bank, Tanzania** | First fintech sandbox in Tanzania; digital-lab innovation initiative | ✅ Verified: official site + live OBPv5.1.0 API Explorer; NMB quote on OBP site |
| **Intercam, Mexico** | Compliant sandbox in Mexico | ✅ Named on OBP site; ⚠ no independent technical review found |
| **OBP public sandbox** | TESOBE's public demo sandbox | ✅ Verified: openbankproject.com |
| **"30+ global banks" (BNP Paribas, SocGen, RBS, Emirates NBD, UniCredit…)** | FinovateEurope 2018 client list; hackathon clients incl. Santander, HSBC | ⚠ Vendor-stated; scope (pilot/production/hackathon) unconfirmed per bank |
| **ABN AMRO** | — | ⚠ **Flagged: NOT verified as an OBP deployment**; appears only as a corporate-API benchmark in OBP's 2023 blog; runs its own portal + hackathons |

---

## 6. The Hackathons

### 6.1 Hackathons as a Go-To-Market Engine

**✅ Verified (that OBP runs hackathons at scale); ⚠ flagged (exact event counts).** Hackathons are not a side activity for OBP — they are the **go-to-market engine**: the hackathon page states *"We've been powering hackathons since 2010 for clients such as Societe Generale, BNP Paribas, Santander, and HSBC"* and *"We've run over 100 fintech hackathons with financial institutions from Brazil to Australia."* The event formula: a full open-banking sandbox environment, a stack of secure fintech APIs, test data, and the "field-tested technology" + "unrivalled expertise" pitch — i.e., a hackathon is a *demo of the sandbox product in action*, and it recruits talent and builds developer community for the bank at the same time.

Verified event-level evidence:

- **London, Level39 Canary Wharf (2013):** Finextra coverage — *"Financial technology lovers will make their way to the Level39 site in London's Canary Wharf later this month for a day-long hackathon organised by Open Bank Project parent Tesobe."* — the earliest independently-covered OBP event found in this pass. ✅
- **Berlin, International Hackathon 2017 (with BNP Paribas):** TESOBE's own hackathons page lists "Open Bank Project International Hackathon 2017 · BNP Paribas · Berlin, Germany." ⚠ single-source (TESOBE site).
- **Mexico, Hackathon Fincluye:** the OBP hackathon page quotes organizer Juan Guerra: *"The TESOBE and Open Bank Project team contribution was critical in making this first edition of Hackathon Fincluye a big success. Perhaps the best hackathon in Financial Services in Mexico to date!"* ✅ official site testimonial; ⚠ single-source.
- **"Over 100 hackathons since 2010"** — ⚠ vendor aggregate figure, plausible but not independently audited.

### 6.2 The Hackathon Suite

The hackathon offer bundles the whole platform (verified, hackathon page): **API Gateway** (traffic control, access control, rate limiting), **Developer Portal** (registration, app onboarding), **API Manager** (administration, roles/entitlements), **API Explorer** (catalogue, docs, in-browser testing), **Auth & Consent** (regulation-compliant authentication/consent), and the **Sandbox** (controlled environment). In other words: the exact same estate a bank would run in production, deployed for 48 hours of competition.

### 6.3 The Hackathon Table

| Event | Where / When | Organiser | Status |
|---|---|---|---|
| **Level39 Canary Wharf hackathon** | London, 2013 | TESOBE (OBP parent) | ✅ Finextra coverage |
| **International Hackathon 2017** | Berlin, 2017, with BNP Paribas | TESOBE/OBP | ⚠ single-source (TESOBE site) |
| **Hackathon Fincluye** | Mexico | OBP/TESOBE supporting | ✅ official testimonial; ⚠ single-source |
| **"100+ hackathons, 2010–present"** | Brazil → Australia | TESOBE/OBP | ⚠ vendor aggregate |
| **Clients named** | SocGen, BNP Paribas, Santander, HSBC | — | ✅ named on OBP site; ⚠ scope unconfirmed |

---

## 7. The Open-Source Model

### 7.1 The License: AGPLv3 + Commercial — Not Apache

**✅ Verified — and this corrects the common "Apache" assumption.** The task premise that OBP is Apache-licensed is **wrong**. The repo's README states verbatim: *"This project is dual licensed under the AGPL V3 (see NOTICE) and commercial licenses from TESOBE GmbH."* The repo root contains `GNU_AFFERO_GPL_V3_19_Nov_1997.txt`, a `NOTICE`, and a **Harmony Individual Contributor Assignment Agreement** (iCLA) governing contributions. The repo FAQ repeats the terms: custom forks must *"follow the terms of the AGPL or obtain a proprietary license from TESOBE or our partners."* (The GitHub API reports `license: null` only because the license file is not auto-detected on the `develop` branch — the README and license files are unambiguous.)

What the AGPLv3 choice means, practically:

- **Copyleft with a network clause:** AGPLv3 closes the "SaaS loophole" of plain GPL — a bank running a modified OBP as a web service must offer its modified source to its users. For banks this is usually *not* acceptable, which is precisely why TESOBE sells the **commercial license** for closed-source internal deployments.
- **The dual-license model** (open copyleft + paid commercial) is the classic open-core monetisation pattern — same family as the pattern analysed in [Open-Source vs Commercial Insurance Guide](insurance_open_source_commercial_guide.md). It is *not* the Apache-style "use it however you like" model: **Apache 2.0 has no copyleft and no network clause; AGPLv3 is the strongest copyleft.** Any procurement deck that says "OBP is Apache" is wrong and should say "OBP is AGPLv3 + commercial."
- **Contributions:** the Harmony iCLA means outside contributors assign/arm-license their code so TESOBE can relicense commercially — standard for dual-license projects.

### 7.2 The Community Model

**✅ Verified (metrics); ⚠ flagged (community health assessment).** The community is **small but real**: ~1.7k GitHub stars, ~477 forks, 209 open issues (Aug 2026), a single primary maintainer organisation (OpenBankProject), active release notes (last entries 13 Aug 2026), and a developer-community surface built from the sandbox + API Explorer + training rather than from mass contributor inflow. ⚠ No contributor-count or bus-factor analysis was performed in this pass; the honest characterisation is: **a vendor-led open-source project (open core), not a foundation-governed one** — the opposite governance pole from the Apache Foundation's [Apache Fineract Guide](apache_fineract_guide.md), where the foundation owns the brand and the code. OBP's "community" is mostly the *ecosystem of banks and developers consuming it*, not a large contributor base.

### 7.3 The Business Model: Open Core With Services

Structural (consistent with the dual license, the services pages, and the sandbox/hackathon offers):

1. **Open core:** the API platform is free under AGPLv3; anyone can evaluate, fork, and run the community edition.
2. **Commercial licenses:** paid licenses for banks that cannot (or will not) comply with AGPLv3's source-sharing obligations — TESOBE GmbH is the licensor.
3. **Services:** sandbox-as-a-service, hackathon delivery, training, API strategy, regulatory advisory, custom connector development.
4. **Hosted/white-label products:** bank-branded sandboxes, developer portals and API Explorers operated by TESOBE.
5. **Regulatory advisory** as a premium positioning: "the team that inspired the first Open Banking regulation" (⚠ vendor claim).

⚠ **Funding:** no public funding details (VC rounds, etc.) for TESOBE were found in this pass — the company appears to be privately held and revenue-funded via the model above. Flag accordingly.

### 7.4 The Model Table

| Dimension | OBP reality | Status |
|---|---|---|
| **License** | **AGPLv3 + commercial from TESOBE GmbH** — *not* Apache 2.0 | ✅ repo README, LICENSE file, FAQ |
| **Governance** | Vendor-led (TESOBE/OpenBankProject org); no independent foundation | ✅ structural (GitHub org) |
| **Contributions** | Harmony iCLA so code can be dual-relicensed | ✅ repo file |
| **Community size** | ~1.7k stars / ~477 forks / 209 open issues (Aug 2026) | ✅ GitHub API |
| **Maintenance** | Active — release notes dated Aug 2026 | ✅ repo |
| **Monetisation** | Commercial licenses + services + sandbox-as-a-service + hackathons + training | ✅ site offerings |
| **Funding** | Private, revenue-based (no public funding info found) | ⚠ unverified |
## 8. The Comparison: OBP vs the Alternatives

### 8.1 OBP vs General-Purpose API Platforms

The first confusion to clear: OBP is **not** another MuleSoft / Apigee / IBM API Connect / Kong / Tyk. Those are **general-purpose API management platforms** — gateway, portal, policy, analytics — that know nothing about banking. OBP is a **banking-domain API platform**: it ships a banking data model (banks, accounts, transactions, customers, counterparties, KYC, consents, views), banking standards profiles (Berlin Group XS2A, UK Open Banking), and banking flows (SCA, transaction requests, entitlements) out of the box. A bank typically needs *both*: OBP (or its equivalent) for the banking API surface, and an API gateway/management layer (often OBP's own Gateway component, or a corporate standard like Kong/Apigee) for organisation-wide policy. OBP's gateway component actually *is* a general-purpose-ish gateway productised into the banking context.

### 8.2 OBP vs Open-Source Financial Platforms

- **vs [Apache Fineract Guide](apache_fineract_guide.md):** Fineract is an open-source **core banking system** (ledger, loans, savings, accounting, clients) with its own REST API, governed by the Apache Foundation under Apache 2.0. OBP is an **API platform that fronts an existing core** via connectors. They are complementary, not rivals: a digital bank could run Fineract as the core *and* OBP in front of it for the open-banking/partner surface (the connector layer is exactly where the two would meet). ⚠ No published Fineract+OBP reference architecture was found in this pass — that combination is a design proposal, not a verified deployment.
- **vs [Mojaloop Guide](mojaloop_guide.md):** Mojaloop is the open-source **payments interoperability** hub (DFS/fintech ecosystem switching), not a bank API platform — different layer again.
- **vs other open-source API/banking projects:** there is no true same-slot open-source rival at OBP's specificity; the closest analogues are regulatory/standards toolkits (e.g., Berlin Group's own open-source artefacts) rather than platforms. OBP's niche — open-source, bank-side, multi-standard open-banking API platform — is largely **uncontested** (⚠ assessment based on the open-source banking landscape known at writing time; the [DeFi Guide](../technology/defi_guide.md) shows how quickly the adjacent crypto/open-finance terrain moves).

### 8.3 OBP vs Commercial Open-Banking / API Vendors

- **vs TPP-side aggregators (Yapily, TrueLayer, Token, Salt Edge):** these are *third-party provider* businesses — they consume bank APIs (usually under PSD2) and resell aggregated access to fintechs. They are the *clients* of the interface OBP helps banks serve, not competitors on the bank side.
- **vs bank-side commercial open-banking platforms** (e.g., vendor platforms behind many banks' developer portals): same slot as OBP, closed source, licence + services pricing. OBP's open-source differentiators: source transparency, no licence lock-in on the API layer, self-hosting, and the sandbox/hackathon play. Its handicaps vs commercial vendors: smaller support organisation, fewer reference deployments, and the AGPLv3 obligations unless the commercial license is taken (which erodes the "free" advantage for most banks).
- **vs the commercial cores** ([Temenos T24 Guide](temenos_t24_guide.md), [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md)): not competitors either — OBP connects *to* them; T24/Flexcube are the cores behind the connector.

### 8.4 The Comparison Table

| Criterion | OBP | General-purpose API mgmt (MuleSoft/Apigee/Kong) | Apache Fineract (open-source core) | Commercial open-banking vendors | TPP aggregators (Yapily/TrueLayer…) |
|---|---|---|---|---|---|
| **Layer** | Banking API platform | Gateway/portal/policy | Core banking | Bank-side open-banking platform | TPP-side access |
| **Banking data model** | ✅ built-in (accounts, views, consents, KYC) | ❌ (bring your own) | ✅ (ledger, clients, loans) | ✅ | Read-only consumer |
| **PSD2/XS2A profiles** | ✅ Berlin Group + UK + native | ❌ | ❌ (own API) | ✅ (usually Berlin Group/UK) | Uses others' |
| **Open source** | ✅ AGPLv3 + commercial | Mixed (Kong OSS; MuleSoft/Apigee closed) | ✅ Apache 2.0, foundation-governed | ❌ | ❌ |
| **Sits in front of existing core** | ✅ via connectors | ✅ (generic) | ❌ (is the core) | ✅ | n/a |
| **Typical buyer** | Bank building open-banking/partner API estate | Any enterprise | Digital banks, microfinance | Banks wanting managed compliance | Fintechs/TPPs |
| **Ecosystem/community** | ⚠ small, vendor-led | Large | Large, foundation | n/a | n/a |
| **Best-fit** | Bank-side open banking, sandboxes, hackathons, partner APIs | Organisation-wide API governance | New core for digital bank | Managed turnkey compliance | Consuming bank APIs |

---

## 9. Worked Example: A Mid-Size Bank's Developer Portal on OBP

### 9.1 The Scenario: Meridian Bank

**Meridian Bank** (fictional) is a mid-size regional bank — think the scale of the digital banks in [Trust Bank Guide](trust_bank_guide.md) / [GXS Bank Guide](gxs_bank_guide.md) / [MariBank Guide](maribank_guide.md), or of NMB in Tanzania — running a commercial core (T24/Flexcube-class, see [Temenos T24 Guide](temenos_t24_guide.md) / [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md)) and an ageing online-banking channel. The board has approved an **open-banking programme**: a public **developer portal**, partner APIs for SME accounting integrations, a PSD2-style compliance posture in any EU market it enters, and an innovation programme (sandbox + hackathon) to court fintechs. The architecture team must decide how to stand up the API estate without a seven-figure licence.

**Why OBP fits this familiar context:** mid-size banks cannot swallow a commercial open-banking platform's licence plus integration fees, cannot wait for the core vendor's roadmap, and — crucially — *already own* the systems of record. OBP's model (API layer over a connector, self-hosted, open source) matches exactly. The same reasoning appears in the verified NMB Tanzania case: a national bank catalysing its fintech ecosystem with TESOBE's sandbox.

### 9.2 The OBP Deployment Design

**Logical architecture (structural design, informed by OBP's documented components):**

1. **Core integration — the connector.** The single most important workstream. OBP's domain model (banks, accounts, transactions, customers, products) must be mapped onto the core's real data. Meridian plans a **connector service**: an adapter (OBP's connector SPI) calling the core's integration layer (T24's IRIS/AA APIs, Flexcube's OFS/OLBM, or a payments hub as in [Payments Hub Guide](payments_hub_guide.md)) for balances/transactions, and the core's own customer/KYC stores for customer data. The "no connector set" error NMB's explorer showed is the reminder that *the core is not included*.
2. **OBP API core** (Scala, versioned v4/v5 line) deployed behind an **API Gateway** (OBP's own, or the bank's enterprise gateway) with **mTLS** termination for PSD2-grade TPP traffic and OAuth2/OIDC for partner apps — cross-ref [Distributed Auth Guide](../technology/distributed_auth_guide.md).
3. **Developer Portal + API Explorer + Sandbox** as the public face: self-service registration, app onboarding, role/entitlement requests, in-browser endpoint testing, synthetic **region-localised data** (Meridian's sandbox generates its market's currency/format conventions). This is the white-label estate TESOBE sells as a service or the bank runs from source.
4. **Consent & SCA flows** prebuilt from OBP's Auth & Consent: Berlin Group v1.3 profile for EU corridors (AIS consent, PIS transaction requests with challenges, Redirect SCA), OBP-native consents for domestic partners, **quorum-based corporate consents** (the multi-sig design in OBP's 2023 corporate-consents blog: quorum set in an Account Attribute, successive SCA challenges until the quorum is reached) for SME treasury clients.
5. **Governance & ops:** roles/entitlements mapped to Meridian's API-governance process ([API Governance Guide](../technology/api_governance_guide.md)), versioned endpoint enablement (`api_enabled_versions`/`api_enabled_endpoints`), monitoring, rate limiting in the gateway.
6. **Legal/licensing:** AGPLv3 review → **commercial license from TESOBE** for the closed-source deployment (the realistic outcome for a bank, per §7.1), or a fork with compliance obligations; iCLA terms for any in-house contributions.

**Runbook shape (phased):**

- **Phase 0 — evaluation (weeks 1-6):** run OBP from source against the sandbox data source; stand up the public sandbox; map core→OBP data gaps; license review.
- **Phase 1 — partner API pilot (months 2-5):** connector v1 for accounts/transactions; Developer Portal + API Explorer live; two pilot SME accounting partners consuming read APIs under OBP-native consents.
- **Phase 2 — hackathon (month 6):** launch the fintech hackathon on the sandbox — the NMB/Fincluye play: recruit talent, stress-test the API surface, generate partner pipeline (§6).
- **Phase 3 — production open banking (months 7-12):** Berlin Group profile enabled for EU corridors, mTLS + SCA hardening, payment initiation (SEPA) via transaction requests, corporate multi-sig consents, entitlements aligned to the governance process.
- **Phase 4 — ecosystem (year 2):** API Agent for natural-language discovery, product/FX endpoints, and expansion of the connector to more core capabilities.

**The target architecture in one diagram:**

```
                        ┌────────────────────────────────────────────────┐
                        │              Meridian Bank estate              │
                        │                                                │
  Third parties         │   ┌──────────────┐      ┌──────────────────┐   │
  (TPPs / fintechs) ────┼──▶│ API Gateway  │─────▶│ Developer Portal │   │
  mTLS + OAuth2/OIDC     │   │ mTLS, rate-  │      │ + API Explorer   │   │
                        │   │ limit, auth  │      │ + Sandbox        │   │
                        │   └──────┬───────┘      └──────────────────┘   │
                        │          │                                     │
                        │   ┌──────▼─────────────────────────────┐       │
                        │   │        OBP API core (Scala)        │       │
                        │   │ views · consents · entitlements ·  │       │
                        │   │ transaction requests · KYC · SCA   │       │
                        │   └──────┬─────────────────────────────┘       │
                        │          │  Connector (the real work)          │
                        │   ┌──────▼─────────────────────────────┐       │
                        │   │ Core systems: T24/Flexcube-class   │       │
                        │   │ core · payments hub · KYC stores   │       │
                        │   └────────────────────────────────────┘       │
                        └────────────────────────────────────────────────┘
```

**The partner-onboarding data flow (what actually happens at runtime):**

1. Partner developer registers on the Developer Portal → Consumer key issued.
2. Developer uses the API Explorer to browse the catalogue, test endpoints on the Sandbox (synthetic localised data), and request roles/entitlements.
3. The partner's app connects with OAuth2/OIDC (mTLS for PSD2 corridors); every request is checked against entitlements + consents.
4. A customer grants access through the prebuilt consent screen (OBP-native, or Berlin Group AIS consent for EU corridors) → consent object created, SCA where required.
5. The connector translates OBP's domain calls into core calls; responses flow back through the same stack, with gateway logging/rate limiting feeding the API-governance process.

### 9.3 The Lessons

1. **The connector is the project.** OBP delivers the API estate; the bank delivers the mapping to its core. Budget the connector workstream as the critical path — the same truth as every commercial open-banking platform, minus the licence premium.
2. **Open source ≠ free for banks.** AGPLv3's network clause almost always pushes a bank to the **commercial license**; the real OBP value is *source transparency, no vendor lock-in on the API layer, and the ability to fork/host* — not a zero licence cost. Procurement decks must say AGPLv3+commercial, never "Apache" (§7.1).
3. **Standards are profiles over one model.** One deployment serves Berlin Group, UK, and OBP-native interfaces from the same accounts/consents/transaction-requests model — Meridian gets multi-jurisdiction posture without separate platforms (§2.5, §4).
4. **Sandbox-first is a proven go-to-market.** NMB's first-sandbox-in-Tanzania and the 100+ hackathon record show the pattern: sandbox → hackathon → partners → production. For a mid-size bank this is how you buy credibility with fintechs cheaply (§5-§6).
5. **Community risk is real.** OBP's community is vendor-led and small; Meridian should (a) take TESOBE's services or training, (b) keep customisations in `custom/` folders (the FAQ's own advice for minimising merge conflicts on forks), and (c) maintain the fork discipline any open-core dependency demands (§7.2).
6. **Cross-sell the rest of the series:** the API layer only makes sense atop a sound core ([Core Banking Systems Guide](core_banking_systems_guide.md)), with governance ([API Governance Guide](../technology/api_governance_guide.md)) and auth ([Distributed Auth Guide](../technology/distributed_auth_guide.md)); the open-source-vs-commercial trade-off method is in [Open-Source vs Commercial Insurance Guide](insurance_open_source_commercial_guide.md).

---

## 10. The Summary: One Page

**The Open Bank Project (OBP) is the open-source banking-API platform: a Scala-based, AGPLv3 + commercially dual-licensed REST API estate that sits in front of a bank's core and exposes accounts, transactions, customers, KYC, payments, products and management APIs through versioned, documented, standards-mapped interfaces — OBP's own standard plus Berlin Group NextGenPSD2 XS2A (v1.3) and UK Open Banking.**

- **Who builds it:** TESOBE GmbH, Berlin (founded 2005 by Simon Redfern, who remains OBP's founder-CEO) — a consultancy turned open-core product vendor, selling licenses, sandboxes, hackathons, training and regulatory advisory around the code.
- **What it is not:** a core banking system (that's Fineract/T24/Flexcube — OBP connects to them), nor a generic API gateway (that's Apigee/Kong — OBP is the banking-domain layer).
- **Verified anchors:** the codebase has been public since November 2012 (~1.7k stars); NMB Tanzania runs a live OBP-powered sandbox (OBPv5.1.0); Intercam Mexico and the OBP public sandbox are named on the official site; OBP has powered 100+ hackathons since 2010 for names like SocGen, BNP Paribas, Santander and HSBC.
- **Flagged honestly:** the "Apache license" claim is wrong (AGPLv3 + commercial); "30+ global bank clients" is vendor-stated; **ABN AMRO is not verified as an OBP deployment**; "500+ endpoints" is a vendor figure; "inspired the first open-banking regulation" is a marketing claim with partial support; funding is undisclosed.
- **The verdict for a solution architect:** OBP is the strongest open-source option in a largely uncontested slot — bank-side, multi-standard open-banking API platform. For a mid-size bank it offers a credible path from public sandbox to hackathon to production partner APIs over the existing core, with source transparency and no API-layer lock-in, in exchange for owning the connector work, accepting a small vendor-led community, and — realistically — paying TESOBE for the commercial license and services. OBP does not replace the core, the gateway, or governance; it is the platform that finally makes a bank's data programmable — the open-source banking API.

---

## 11. Consolidated Verification Notes

| Claim | Status | Evidence |
|---|---|---|
| OBP is an open-source RESTful banking-API platform | ✅ | GitHub repo description; OBP site |
| Supports Open Banking / XS2A / PSD2 / Open Finance | ✅ (as implemented interfaces) | Repo description; release notes; site payloads |
| Berlin Group XS2A v1.3 + UK Open Banking support | ✅ | Site payloads (`api_standard: Berlin Group`); Aug 2026 release notes (IG §4.11, UK scopes, Redirect SCA) |
| TESOBE GmbH, Berlin; founded 2005; "Technical Solutions Berlin" | ✅ | Crunchbase; OBP site footer; FintechForum interview |
| Simon Redfern = founder & CEO OBP, CEO TESOBE | ✅ | Crunchbase, Finovate, FintechForum, TNW |
| License = Apache 2.0 | ❌ **Wrong** — AGPLv3 + commercial from TESOBE | Repo README, GNU_AGPL file, FAQ |
| "Over 500 endpoints" | ⚠ Vendor figure, config-dependent | OBP sandbox page; repo FAQ (per-version gating) |
| NMB Tanzania sandbox (first in Tanzania) | ✅ | OBP site + live nmbbank.co.tz API Explorer (OBPv5.1.0) |
| Intercam Mexico sandbox | ✅ named / ⚠ un-reviewed | OBP site |
| ABN AMRO deployment | ⚠ **Unverified — flagged** | Only OBP 2023 blog benchmark mention found; ABN AMRO runs own portal/hackathons |
| "30+ global banks" client list | ⚠ Vendor-stated | FinovateEurope 2018 presentation |
| 100+ hackathons since 2010 | ⚠ Vendor aggregate; events at Level39 (2013, ✅ Finextra), Berlin 2017 (⚠), Fincluye Mexico (⚠) | OBP site; Finextra; TESOBE site |
| "Inspired the first open-banking regulation" | ⚠ Vendor claim, partial support | OBP blog (EC/UK Treasury echo); Mexico working-group posts |
| Community size | ✅ ~1.7k stars / 477 forks / 209 issues (Aug 2026) | GitHub API |
| Funding | ⚠ Undisclosed in this pass | — |
| Endpoint/API versions | ✅ v1.x–v5.x line; v4.0.0 site paths; live OBPv5.1.0 instance | OBP site; NMB explorer; repo release notes |

---

## 12. Glossary

- **Open Bank Project (OBP):** the open-source RESTful API platform for banks described in this guide — accounts, transactions, customers, KYC, payments, consents, entitlements and management APIs, with Berlin Group XS2A, UK Open Banking and its own native standard profiles. Launched publicly around 2010-2012 by TESOBE.
- **OBP:** the acronym for Open Bank Project; also the prefix of the native API version line (OBPv1.x–OBPv5.x) and of OBP error codes (e.g., OBP-50000).
- **TESOBE:** Technical Solutions Berlin — the Berlin-based company (GmbH), founded 2005 by Simon Redfern, that builds, maintains, licenses and commercialises the Open Bank Project.
- **Berlin:** the German city where TESOBE is based; also shorthand for the **Berlin Group**, the standards body behind NextGenPSD2 XS2A that OBP implements.
- **Simon Redfern:** founder and CEO of the Open Bank Project and CEO of TESOBE (programmer, composer; started TESOBE in 2005).
- **API (Application Programming Interface):** the machine-readable contract through which applications request data or actions; in banking, the surface third parties use to reach accounts, transactions and payments.
- **API standard:** a formalised, versioned specification of an API's resources, semantics and errors — OBP's own standard, the Berlin Group NextGenPSD2 XS2A framework, and UK Open Banking are the ones in this guide.
- **API Explorer:** OBP's browsable catalogue + in-browser testing tool, driven by the platform's resource documentation (open source: OpenBankProject/API-Explorer).
- **Developer Portal:** the self-service web surface where developers register, onboard applications and request API access/roles — the bank's white-label developer face.
- **Sandbox:** a controlled test environment with synthetic, localised data — OBP's sandbox ships 500+ endpoints, SDKs, prebuilt auth/consent flows, and conversational AI.
- **PSD2 (Payment Services Directive 2):** EU Directive 2015/2366 (with EBA RTS) requiring banks to open account access to licensed third parties (AISPs/PISPs) under consent.
- **XS2A (Access to Account):** the PSD2-mandated interface between banks (ASPSPs) and third-party providers; standardised by the Berlin Group's NextGenPSD2 framework.
- **ABN AMRO:** the Dutch bank named in the task's premise as an OBP deployment — **flagged unverified** in this guide; it appears in OBP's corporate-API benchmark blog but runs its own developer portal and hackathons.
- **Hackathon:** a time-limited development event (typically 24-72h) where teams build prototypes on an API platform; OBP's go-to-market engine — 100+ events since 2010 (Level39 London 2013, Berlin 2017, Mexico's Fincluye, and more).
- **Apache:** shorthand for the Apache Software Foundation and its permissive **Apache 2.0** license — the license Fineract uses and OBP **does not** (OBP is AGPLv3 + commercial).
- **Open source:** software whose source is publicly available for inspection, use and modification under a license — with very different obligations between permissive (Apache 2.0), copyleft (GPL) and network-copyleft (AGPL) families.
- **License:** the legal terms governing use/modification/distribution — for OBP, dual AGPLv3 (community) and commercial (TESOBE GmbH) terms, with a Harmony iCLA for contributors.
- **Accounts:** the core resource OBP exposes (bank, account, balance) behind field-level **views** — the transparency abstraction at the heart of OBP.
- **Transactions:** history and payment objects; read via views, created via **transaction requests** (SEPA, TAN, counterparty, phone, ATM transfers) with challenges and charges.
- **Customers:** the party records OBP links to users, accounts and KYC documents — including corporate customers with multi-signatory consent (quorum) support.
- **KYC (Know Your Customer):** the identity-verification domain — OBP's KYC documents, checks, media and statuses endpoints support onboarding and regulatory workflows.
- **Developer portal:** the productised web experience (registration, apps, roles, docs, testing) through which a bank's API programme is consumed — also the worked example in §9.
- **Banking API:** the API surface exposing banking data and capabilities to internal teams, partners and regulated third parties — the thing OBP productises for banks.

---

## 13. Primary Sources and References

**Primary (verified in this pass, August 2026):**

- Open Bank Project — official site: openbankproject.com (home, platform, sandbox, hackathon, blog: "Managing Corporate Consents with OBP" 29 Jun 2023, "How to Regulate Open Banking")
- OBP-API GitHub repository: github.com/OpenBankProject/OBP-API (README, FAQ.md, release_notes.md, LICENSE files, docs/ — MTLS.md, MTLS_TOPOLOGIES.md)
- OBP API Explorer repository: github.com/OpenBankProject/API-Explorer
- TESOBE — tesobe.com/hackathons
- GitHub REST API (repo metadata: created 2012-11-20, Scala, 1,736 stars / 477 forks / 209 open issues, Aug 2026)
- NMB Bank sandbox API Explorer (live OBPv5.1.0 instance): obp-apiexplorer-sandbox.nmbbank.co.tz

**Press and third-party:**

- Finextra — "Geeks descend on Canary Wharf for fintech hackathon" (2013); "ABN Amro hosts hackathon to help build the bank of the future" (2018)
- The Next Web — "Open Bank Project: Financial Transparency With Web 2.0"
- FinovateEurope 2018 — TESOBE Open Bank Project video listing
- Crunchbase — Simon Redfern profile; FintechForum — "7 Questions with Simon Redfern"
- Crowdfund Insider, FinancialIT — ABN AMRO Beyond Banking Days coverage
- Berlin Group — NextGenPSD2 Access to Account (XS2A) framework pages
- BBVA API Market — "Open Bank Project: how open APIs are changing the banking sector"

**Related guides in this repository:** [Apache Fineract Guide](apache_fineract_guide.md) · [Fineract Database Models Guide](fineract_database_models_guide.md) · [Murex MX.3 Platform Guide](murex_mx3_platform_guide.md) · [Nasdaq Calypso Guide](nasdaq_calypso_guide.md) · [Temenos T24 Guide](temenos_t24_guide.md) · [Oracle Flexcube Data Model Guide](oracle_flexcube_data_model_guide.md) · [Core Banking Systems Guide](core_banking_systems_guide.md) · [Open-Source vs Commercial Insurance Guide](insurance_open_source_commercial_guide.md) · [Mojaloop Guide](mojaloop_guide.md) · [Payments Hub Guide](payments_hub_guide.md) · [BIAN Banking Architecture Guide](bian_banking_architecture_guide.md) · [Trust Bank Guide](trust_bank_guide.md) · [GXS Bank Guide](gxs_bank_guide.md) · [MariBank Guide](maribank_guide.md) · [API Governance Guide](../technology/api_governance_guide.md) · [Distributed Auth Guide](../technology/distributed_auth_guide.md) · [DeFi Guide](../technology/defi_guide.md)
