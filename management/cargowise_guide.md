# CargoWise: The Feature List Is Not the Moat — The Shared Database Is

**Jack Liu Shurui, Solution Architect**

**A logistics platform is sold on its feature list and bought on its database. CargoWise, developed by WiseTech Global (ASX: WTC), is the clearest case in the industry: competitors can match individual features, some can match them all, and yet the platform's real hold on a freight forwarder is not the feature catalogue at all — it is that one job, one ledger, one reference master and one upgrade path live in a single shared database, and every office, every partner integration and every trained operator is wired into *that* database rather than into any one function. This guide reads CargoWise as an architectural and commercial object rather than a product tour. It separates the platform (CargoWise) from the listed company (WiseTech Global) that sells it and has bought dozens of other businesses; it examines what a single shared database means operationally and to a client's willingness to leave; it treats the switching cost as the load-bearing question, not the feature list; and it states plainly which parts of the architecture the vendor does not publish. Facts are flagged ✅ (verified this pass against a named primary source, with the retrieval date), ⚠ (vendor claim / approximate / single secondary source), ⚠-knowledge (well-established industry practice not re-verified this pass), or ❌ (could not be verified). No pricing figure, revenue figure or market-share estimate is reproduced; every seller claim is attributed, dated, and never restated in this guide's own voice. A claims-audit table and a "What Could Not Be Verified" section keep the honest ledger.**

> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** October 2026  
> **Companion guides (management/, same folder):** [Freight Forwarding Guide](freight_forwarding_guide.md) — **the forwarder's business model, and the guide that already carries the CargoWise vendor claims and the generic switching-cost argument at its §8.1 and §13 respectively. This guide cross-references those points and does not re-derive them**; where this guide says "see the freight-forwarding guide §8.1", read it as an instruction to go there for the vendor-claim facts rather than as a restatement here. [Logistics Warehouse Management Guide](logistics_warehouse_management_guide.md) — warehouse and inventory operations, WMS/WES/WCS; §3 and §10 cross-reference it for the storage-and-handling layer rather than re-explaining warehouse ops. [CLPA Contract Logistics Guide](clpa_contract_logistics_guide.md) — the 3PL outsourcing project model; §8 and §10 cross-reference it for the contract-logistics buyer rather than re-deriving it. [DHL Express Guide](dhl_express_guide.md) — the integrator/express-operator company profile and the repo's company-profile genre; §8 and §16 cross-reference it. [TMX Transform Guide](tmx_transform_guide.md) — the supply-chain advisory profile and the honesty/claims-audit conventions this guide follows.  
> **Companion guides (banking/, prefix `../banking/`):** [Trade Finance Guide](../banking/trade_finance_guide.md), [Trade Finance Systems Guide](../banking/trade_finance_systems_guide.md), [Supply Chain Finance Guide](../banking/supply_chain_finance_guide.md) and [Supply Chain Finance Technologies Guide](../banking/supply_chain_finance_technologies_guide.md) — the *financing* of trade sits there; §10 cross-references all four for the trade-finance and supply-chain-finance adjacency and re-derives none of them.  
> **Companion guides (technology/, prefix `../technology/`):** [API Governance Guide](../technology/api_governance_guide.md) — the enterprise discipline of governing an API/integration estate, cross-referenced for the §5 integration-boundary argument. [Data Governance Guide](../technology/data_governance_guide.md) — data ownership, lineage and egress discipline, cross-referenced for the §5 and §7 data-portability argument. [Cloud Providers Guide](../technology/cloud_providers_guide.md) — the hosted/tenancy models underlying a multi-tenant SaaS platform, cross-referenced for §5.  
> **Method note:** this pass had live web access on **2026-10-12**. Verification used **direct page extraction of primary vendor and corporate URLs** — wisetechglobal.com (the listed company's own investor, history, about, leadership, governance, businesses and news pages) and cargowise.com (the platform's own product, solutions, Value Pack, community-pricing and partner pages) — supplemented by dated company announcements (WiseTech's own news releases and its ASX-announcement PDFs, and the acquired companies' own releases). `web_search` was **only partly reliable** on this host; where a search returned nothing usable that is recorded as a **tool limitation, not as evidence of absence**. One product page (`cargowise.com/cargowise-next/`) returned a 404 and one developer URL (`developer.cargowise.com`) was refused by the extraction tool; both are recorded as ⚠/❌ rather than filled in from memory. Every seller claim (the customer count, the customer logos, the productivity/compliance figures, the "operating system for global trade and logistics" positioning, the CargoWise Next description, the Value Pack capability counts) is recorded as **the vendor's published claim, attributed and dated**, and is **not** restated in this guide's own voice as established fact. No fact, figure, date, quotation, URL or acquisition has been fabricated; where the guide could not confirm a claim it says so. Every illustrative figure is labelled illustrative and attached to the fictional Cymbal Bank scenario only, and **Cymbal Bank is the sole bank persona and the sole institution in any worked example.** No competitor is ranked or recommended.

---

**How to read this guide.** §1 states the product-versus-company distinction first, defines the vocabulary, states the thesis and draws the boundary. §2 is the company — founding, listing, leadership, attributed size, and the acquisition history with each announcing party and year. §3 is the core section: the product and what a single shared database actually means. §4 is CargoWise Next and the adjacent named evolutions. §5 is the architecture and the integration surface, separating what is published from what is not. §6 is the commercial structure — structures, never amounts. §7 is the switching cost and lock-in question, developed for a buyer. §8 is the competitive frame, structural and unranked. §9 is the buyer's question set. §10 is the trade-finance and banking adjacency. §11 is the failure modes and honest limits. §12 is the Cymbal Bank worked example, fictional and illustrative. §13 is the anti-patterns. §14 is the claims audit. §15 is what could not be verified. §16 is the glossary, cross-references and closing summary. **Completeness conventions:** ✅ = verified this pass against a named primary source and dated; ⚠ = vendor claim, approximate, or single secondary source; ⚠-knowledge = well-established industry knowledge not re-verified this pass; ❌ = could not be verified. Cross-references follow repo convention: same-directory guides by plain filename, `../banking/...`, `../technology/...` otherwise.

---

**Table of Contents**

1. [Overview, the Product-versus-Company Distinction, the Decoder, the Thesis and the Boundary](#1-overview-the-product-versus-company-distinction-the-decoder-the-thesis-and-the-boundary)
   - 1.1 [The Distinction Stated First: CargoWise the Platform vs WiseTech Global the Company](#11-the-distinction-stated-first-cargowise-the-platform-vs-wisetech-global-the-company)
   - 1.2 [The Vocabulary Decoder](#12-the-vocabulary-decoder)
   - 1.3 [The Thesis: The Shared Database, Not the Feature List](#13-the-thesis-the-shared-database-not-the-feature-list)
   - 1.4 [The Boundary — What This Guide Owns and What It Cross-References](#14-the-boundary--what-this-guide-owns-and-what-it-cross-references)
   - 1.5 [The Key-Facts Table](#15-the-key-facts-table)
   - 1.6 [Why a Bank Should Care](#16-why-a-bank-should-care)
2. [The Company — WiseTech Global](#2-the-company--wisetech-global)
   - 2.1 [Founding, Listing and Legal Identity](#21-founding-listing-and-legal-identity)
   - 2.2 [Leadership as Published](#22-leadership-as-published)
   - 2.3 [Size, Attributed and Dated](#23-size-attributed-and-dated)
   - 2.4 [The Acquisition History, with Announcing Party and Year](#24-the-acquisition-history-with-announcing-party-and-year)
   - 2.5 [The Governance and Conduct Note](#25-the-governance-and-conduct-note)
3. [The Product and the Single-Database Model](#3-the-product-and-the-single-database-model)
   - 3.1 [What the Platform Is](#31-what-the-platform-is)
   - 3.2 [What One Global Shared Database Actually Means Operationally](#32-what-one-global-shared-database-actually-means-operationally)
   - 3.3 [The Module Set, as the Vendor Describes It](#33-the-module-set-as-the-vendor-describes-it)
   - 3.4 [The Personas It Serves](#34-the-personas-it-serves)
   - 3.5 [No Independent Architecture Disclosure](#35-no-independent-architecture-disclosure)
4. [CargoWise Next](#4-cargowise-next)
   - 4.1 [The Vendor's Own Naming of CargoWise Next](#41-the-vendors-own-naming-of-cargowise-next)
   - 4.2 [The Value Packs as an Adjacent, Dated Evolution](#42-the-value-packs-as-an-adjacent-dated-evolution)
   - 4.3 [CargoWise Neo](#43-cargowise-neo)
   - 4.4 [What the Guide Will Not Invent](#44-what-the-guide-will-not-invent)
5. [The Architecture and the Integration Surface](#5-the-architecture-and-the-integration-surface)
   - 5.1 [What Is Publicly Documented and What Is Not](#51-what-is-publicly-documented-and-what-is-not)
   - 5.2 [The Integration Path: eAdaptor, Universal XML and the Partner Layer](#52-the-integration-path-eadaptor-universal-xml-and-the-partner-layer)
   - 5.3 [Named Integrations with Dated Announcements](#53-named-integrations-with-dated-announcements)
   - 5.4 [The Hosted and Multi-Tenant Model, as Described](#54-the-hosted-and-multi-tenant-model-as-described)
   - 5.5 [The Data-Integration Questions a Client Faces](#55-the-data-integration-questions-a-client-faces)
6. [The Commercial Structure](#6-the-commercial-structure)
   - 6.1 [The Legacy Structure: Seats, Modules and Hosting](#61-the-legacy-structure-seats-modules-and-hosting)
   - 6.2 [The Value Pack Model, as Publicly Described](#62-the-value-pack-model-as-publicly-described)
   - 6.3 [The Pass-Through Disbursement Structure](#63-the-pass-through-disbursement-structure)
   - 6.4 [The Published Community Pricing Page and the Absence Finding](#64-the-published-community-pricing-page-and-the-absence-finding)
7. [The Switching Cost and the Lock-In Question](#7-the-switching-cost-and-the-lock-in-question)
   - 7.1 [What the Client Owns and What the Vendor Holds](#71-what-the-client-owns-and-what-the-vendor-holds)
   - 7.2 [The Migration Is the Moat](#72-the-migration-is-the-moat)
   - 7.3 [Why Incumbency Survives Product Dissatisfaction](#73-why-incumbency-survives-product-dissatisfaction)
   - 7.4 [How a Buyer Reduces Lock-In](#74-how-a-buyer-reduces-lock-in)
8. [The Competitive Frame](#8-the-competitive-frame)
   - 8.1 [The Categories of Alternative](#81-the-categories-of-alternative)
   - 8.2 [Structural and Unranked](#82-structural-and-unranked)
9. [The Buyer's View](#9-the-buyers-view)
   - 9.1 [How the Category Is Actually Evaluated](#91-how-the-category-is-actually-evaluated)
   - 9.2 [The Question Set](#92-the-question-set)
10. [The Trade-Finance and Banking Adjacency](#10-the-trade-finance-and-banking-adjacency)
    - 10.1 [The eBL Link](#101-the-ebl-link)
    - 10.2 [What a Financier Could and Could Not Verify from Platform Data](#102-what-a-financier-could-and-could-not-verify-from-platform-data)
    - 10.3 [The Supply-Chain-Finance Connection](#103-the-supply-chain-finance-connection)
11. [The Failure Modes and Honest Limits](#11-the-failure-modes-and-honest-limits)
12. [The Cymbal Bank Worked Example (Fictional)](#12-the-cymbal-bank-worked-example-fictional)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, Cross-References and Closing Summary](#16-the-glossary-cross-references-and-closing-summary)

---
## 1. Overview, the Product-versus-Company Distinction, the Decoder, the Thesis and the Boundary

### 1.1 The Distinction Stated First: CargoWise the Platform vs WiseTech Global the Company

Everything in this guide is easier to read once the distinction is fixed:

- **CargoWise is a software platform** — the logistics execution and management system that a freight forwarder, a customs broker, a 3PL or a landside operator runs its business on. It is sold under the CargoWise brand and described on cargowise.com.
- **WiseTech Global (ASX: WTC) is the listed Australian company that develops and sells CargoWise** — and that has, over more than two decades, *bought dozens of other software businesses* and folded them into a group. CargoWise is the flagship; it is not the whole company.

That second sentence is the one that trips people up. A reader who says "WiseTech" when they mean "CargoWise" will misread the company's acquisition history as product releases, and will misread the product's architecture as the company's architecture. When this guide writes **CargoWise** it means the platform. When it writes **WiseTech Global** or **the group** it means the listed company and everything it owns. The vendor itself draws a line: its own "our businesses" page lists CargoWise as the flagship inside *each* of seven product groups (Forwarding and Customs, Landside Logistics, Digital Documents, TMS, Specialist WMS, Carrier and Rates, Enterprise), alongside dozens of acquired brands (wisetechglobal.com/what-we-do/our-businesses, retrieved 2026-10-12 ✅ — vendor page).

The practical consequence for a buyer: the feature list you are shown belongs to CargoWise; the counterparty you are contracting with, the vendor whose roadmap decides what gets built, and the entity that a regulator or a bank would assess are all WiseTech Global. §2 is about that company; §3 onward is about the platform.

### 1.2 The Vocabulary Decoder

This guide uses industry terms precisely. Definitions below are the ones used here; where a definition is the guide's own framing rather than a vendor statement, it is written as a definition, not attributed.

| Term | What it means in this guide |
|---|---|
| **Forwarding system** | The core software a freight forwarder runs its business on: it creates and manages a shipment as a **job**, captures parties/cargo/routing/references, generates and stores documents, exchanges messages with carriers, customs and customers, tracks milestones, and prices, invoices and accounts for the transaction. CargoWise is one. See the freight-forwarding guide §8.1 for the forwarder's business context. |
| **Logistics ERP** | The accounting-and-enterprise half of a forwarding platform: chart of accounts, billing/costing engine, multi-entity and multi-currency ledgers, accrual accounting, invoicing and reporting. A forwarder's platform is a *logistics ERP* as much as a shipment tracker — the accounting module is often the hardest part to leave. |
| **TMS (Transport Management System)** | The layer that plans and executes the physical road/rail/drayage legs: load and route optimisation, carrier tendering, dispatch, driver management and delivery tracking. In the forwarder's world a TMS is frequently a separate product from the forwarding platform, which is why the boundary between them is where integration projects live (freight-forwarding guide §8.2). |
| **Customs broker system** | The software that prepares and files customs declarations and clears goods: classification, valuation, origin, procedure/relief and the filing itself. It is jurisdiction-specific and connects to national customs systems. |
| **Single database** | The vendor's central architectural claim: that CargoWise runs its customers on **one shared, global database** rather than on per-office or per-region instances reconciled by file transfer. It is the thesis of this guide and is examined in §3.2. |
| **Module** | A functional component of the platform (Forwarding, Customs, Warehouse, Transport, Accounting, etc.) that can be licensed or bundled separately. A module is a component of a system, not a system of record in its own right. |
| **House bill** | The **house bill of lading (H-B/L)** — the transport document the *forwarder* issues to its customer (the shipper), as distinct from the **master bill of lading (M-B/L)** the carrier issues to the forwarder. The house bill is the forwarder's contract with its customer; the master bill is the carrier's contract with the forwarder. |
| **NVOCC** | A **Non-Vessel-Operating Common Carrier** — an entity that issues its own bills of lading and acts as a carrier to its customers while not operating the vessel: it buys space from ocean carriers and sells carriage as a consolidator. An NVOCC is a *carrier-facing* persona, distinct from a pure forwarder. |
| **Consolidator** | An operator that combines multiple shippers' cargo into one container (LCL consolidation) to earn the consolidation margin; often the same firm as an NVOCC. |
| **Tenant** | In a hosted/multi-tenant platform, one customer's logically-isolated slice of the shared service. "Single shared database" (above) and "multi-tenant" are related but not identical ideas; §5.4 states plainly which the vendor publishes and which it does not. |
| **Integration** | Any machine-to-machine connection between the platform and another system (carrier, customs authority, customer, partner) by API, EDI, file or portal. §5 is about the integration surface as *published* and as *not published*. |
| **eBL** | **Electronic Bill of Lading** — a digital transport document with a legal/functional equivalence claim, issued on an eBL platform. §10 cross-references the eBL discussion rather than re-deriving it. |
| **Disbursement** | A cost a forwarder pays on its customer's behalf and recovers on the customer's invoice, typically posting to a balance-sheet account rather than through the profit-and-loss account. §6.3 examines this as a *commercial structure*. |

### 1.3 The Thesis: The Shared Database, Not the Feature List

The thesis of this guide is in its title and is worth stating plainly:

> **The feature list is not the moat; the shared database is.**

The argument runs like this, and §3, §6 and §7 develop each step:

1. **Features are copyable and are, in fact, copied.** A feature catalogue is a list of things a competent competitor can build. WiseTech's own material frames the platform not around a feature count but around **one platform, one database** ("Unify your logistics operations on a single global database — pay for one system, learn one system and maintain one system", cargowise.com, retrieved 2026-10-12 ✅ — vendor claim). The vendor is telling you where it believes its advantage lives.
2. **The single shared database is a *structural* claim, not a feature.** If every office of a forwarder, across every country and currency, reads and writes the *same* job record, the *same* ledger and the *same* reference master, then there is no per-office reconciliation, no nightly file exchange between country systems, and no divergence between the document a branch issues and the accounting entry another branch posts. That is an architectural property of the platform, not a menu item. §3.2 examines it operationally.
3. **The database is also the lock-in.** A forwarder's willingness to leave is set not by how much it likes a screen but by what leaving would cost: the job history, the accounting history, the carrier and customs mappings, the customer portal, and the trained staff — all of which are attachments to *this* database. §7 develops the switching cost as the buyer's central question, cross-referencing the generic version in the freight-forwarding guide §13 and going further.
4. **Therefore the commercial model can change and the client stays.** The vendor can move from per-seat pricing to a per-transaction model (§6), and because the client's data and integrations are already in the shared database, the client's realistic alternative is not "leave" but "accept the new structure or renegotiate". §7.3 argues why incumbency survives product dissatisfaction.

The thesis is a *reading*, and the guide is honest about its limits: the vendor does not publish its internal architecture (§3.5, §5.1), so the "single database" is asserted by the vendor and is not independently verifiable from the outside. §11 and §15 carry that caveat.

### 1.4 The Boundary — What This Guide Owns and What It Cross-References

This guide does **not** re-derive what the repository already establishes. Specifically, and by name:

- **[freight_forwarding_guide.md](freight_forwarding_guide.md) §8.1 already carries the CargoWise vendor claims** — the "operating system for global trade and logistics" positioning, the "over 17,000 organizations trust CargoWise" line, the customer logos, the e2open acquisition facts, and the reported ACCC search warrant (19 August 2026, ⚠ press-reported). **This guide cross-references those points; it does not restate them as its own findings.** Where a fact is established there, this guide says "see the freight-forwarding guide §8.1" and moves to something that guide does not cover.
- **[freight_forwarding_guide.md](freight_forwarding_guide.md) §13 already develops the generic switching-cost argument** for incumbent forwarding platforms (jobs, documents, tariffs, customer records, integration mappings, accounting history; multi-year migration). This guide **goes further** — into what the client owns versus the vendor, the egress and termination questions, the migration-risk mechanism, and how a buyer can reduce lock-in (§7) — rather than repeating the generic point.
- **The forwarder's business model** (intermediary position, B/L, Incoterms, surcharges) is the freight-forwarding guide's subject and is not re-derived here.
- **Warehouse operations, WMS/WES/WCS and automation** are [logistics_warehouse_management_guide.md](logistics_warehouse_management_guide.md)'s subject; §3 and §10 cross-reference it.
- **The 3PL outsourcing engagement and its commercials** are [clpa_contract_logistics_guide.md](clpa_contract_logistics_guide.md)'s subject; §8 and §10 cross-reference it.
- **Trade finance, trade-finance systems, supply-chain finance and its technologies** are [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md), [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md), [../banking/supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md) and [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md); §10 cross-references all four and asserts no capability they do not establish.

**What this guide owns:** the company (WiseTech Global as an entity and an acquirer), the platform's architecture and single-database model, its commercial *structure*, the switching-cost/lock-in question developed for a buyer, the integration/API surface as published *and* as unpublished, the competitive frame (structural, unranked), the trade-finance adjacency, and the failure modes. That is the gap this guide fills.

### 1.5 The Key-Facts Table

| Fact | Value | Status and source |
|---|---|---|
| Platform | CargoWise | Vendor: cargowise.com (2026-10-12) ✅ |
| Vendor | WiseTech Global Limited | Vendor: wisetechglobal.com (2026-10-12) ✅ |
| Listing | Australian Securities Exchange, ticker **ASX: WTC** | Vendor investor page ✅ |
| Listing date | **11 April 2016** | Vendor: "Established in 1994, WiseTech Global listed on the Australian Securities Exchange on 11 April 2016" (wisetechglobal.com/investors, retrieved 2026-10-12) ✅ |
| Founded | **1994**, by Richard White and Maree Isaacs | Vendor: wisetechglobal.com/who-we-are/our-history/ ✅ |
| Current CEO | **Zubin Appoo** (assumed the role July 2025) | Vendor leadership page ✅ |
| Founder's role | Richard White — Co-Founder, Executive Director and Chief Innovation Officer; CEO until October 2024, Executive Chair February 2025–July 2026 | Vendor leadership page ✅ |
| Customers | "**20k+ customers**"; "more than 20,000 organizations use our software solutions" | **Vendor claim** (wisetechglobal.com/who-we-are/about-us/, 2026-10-12) ⚠ |
| Forwarder / 3PL coverage | "24 of the top 25 global freight forwarders and 46 of the top 50 global third-party logistics providers" | **Vendor claim** (about-us, 2026-10-12) ⚠ |
| Countries licensed | "193 countries licensed to use our software" | **Vendor claim** (about-us, 2026-10-12) ⚠ |
| Product development | "39 product development centers"; "1,827 product enhancements in FY26" | **Vendor claims** (about-us, 2026-10-12) ⚠ |
| Architecture (as stated) | "one global database"; "one platform, zero duplication" | **Vendor claim** (cargowise.com, 2026-10-12) ⚠ — not independently verifiable |
| Hosting (as stated) | "our three data centers in Europe, Australasia and the Americas delivering our CargoWise platform" | **Vendor claim** (Head of Information Systems bio, wisetechglobal.com/investors/our-leadership-team/, 2026-10-12) ⚠ |
| Commercial model (2026) | CargoWise Value Packs; per-eligible-transaction "CargoWise Automation Fee" | **Vendor** (cargowise.com/cargowise-value-pack/, 2026-10-12) ✅ as a described structure; ⚠ as to effect |
| Pricing amounts | **Deliberately not reproduced in this guide** | See §6.4 — the vendor publishes a community pricing page, but this guide prints no amounts |
| Largest acquisition | **e2open** (NYSE: ETWO), completed 3 August 2025 | See freight-forwarding guide §8.1; e2open releases ✅ — cross-referenced, not re-derived |
| Regulatory matter | ACCC search warrant reported executed 19 August 2026 | See freight-forwarding guide §8.1 ⚠ press-reported |

### 1.6 Why a Bank Should Care

A bank encounters CargoWise in three distinct ways, and only one of them involves the bank buying software:

1. **The bank's clients run on it.** Mid-to-large freight forwarders, customs brokers and 3PLs — the bank's trade-finance and working-capital obligors — frequently run their operations on CargoWise. When a bank takes comfort from "our client runs CargoWise", it is taking comfort from the client's operational data living in a widely-used platform; it is also inheriting a **vendor-concentration exposure** in the client's operations that the bank did not underwrite (freight-forwarding guide §13). The honest question is not "is CargoWise good?" but "what happens to our client's records and its ability to operate if that vendor changes terms, changes hands, or is disrupted?" (ACCC matter, freight-forwarding guide §8.1).
2. **The bank lends against trade whose evidence passes through it.** A financier may see a forwarder's platform data as corroboration of a shipment — a movement fact — while mistaking it for documentary evidence of legal status. §10.2 states precisely what a platform can and cannot evidence.
3. **The bank may be a customer itself.** A bank's own trade operations or a group entity that imports/exports may run CargoWise or its shipper-facing tools directly, and then the bank is the buyer with the same lock-in and integration questions as any forwarder (§9).

For all three, the guide's contribution is the same: the feature list is not the thing to assess; the database, the switching cost and the vendor dependency are. See §12 for a fictional Cymbal Bank worked example and §13 for the anti-patterns.

---

## 2. The Company — WiseTech Global

### 2.1 Founding, Listing and Legal Identity

The vendor's own history page states: "**1994** — Our company was founded by Richard White and Maree Isaacs. We began writing code for Australian freight forwarders." (wisetechglobal.com/who-we-are/our-history/, retrieved 2026-10-12 ✅ — company statement). The investor page repeats the anchor for a reader who wants it: "**Established in 1994**, WiseTech Global listed on the Australian Securities Exchange on **11 April 2016**." (wisetechglobal.com/investors, retrieved 2026-10-12 ✅).

The vendor's history page then records the platform's generations as the company's own milestones:

- **2004** — "Developed ediEnterprise, our second-generation global software solution."
- **2014** — "Launched our third-generation, deeply integrated global logistics platform, **CargoWise**."
- **2025** — "Rolled out our **4th generation platform, CargoWise Next**." (See §4.)

Two identity details are worth marking:

- The **original company name** appears in third-party material but not on the vendor's own history page. ⚠ A third-party encyclopaedia entry (Wikipedia, article on the founder, retrieved 2026-10-12) states the company was founded as "Eagle Developments International" and renamed WiseTech in 2011. **This pass did not confirm the original name from a WiseTech primary source**, so the guide treats it as ⚠ third-party and does not rely on it.
- The **legal entity** for the ASX listing is "WiseTech Global Limited". The vendor's exchange ticker is **ASX: WTC** (vendor investor page ✅). The guide does not attempt to enumerate the group's subsidiary entities; the vendor's "our businesses" page lists the group's product businesses by brand, which is the closest the vendor comes to a public group map (§2.4).

The listing valuation line — "**2016** — WiseTech Global goes public. Listed on the Australian Securities Exchange, **valued at $1 billion**" — is the vendor's own history-page statement (✅ as a vendor statement; ⚠ as a valuation, which this guide does not independently verify and does not treat as a market-cap figure).

### 2.2 Leadership as Published

The vendor publishes a leadership page (wisetechglobal.com/investors/our-leadership-team/, retrieved 2026-10-12 ✅). The names and roles below are as the vendor states them; the guide repeats them as *published*, not as an assessment of the people:

| Name | Role (as published) | Vendor-stated note |
|---|---|---|
| **Zubin Appoo** | Chief Executive Officer | "joined WiseTech in 2025 and assumed the role of Chief Executive Officer in **July**" (2025); previously at WiseTech 2004–2018 |
| **Richard White** | Co-Founder, Executive Director and **Chief Innovation Officer** | "co-founded WiseTech in 1994 and was Chief Executive Officer and Executive Director **up until October 2024**. He was Executive Chair from **February 2025 to July 2026**. He remains an Executive Director of the Board." |
| **Brett Shearer** | **CTO and Chief Architect** | "has worked with WiseTech Global since 1994, working on each successive generation of the software from Deliverance to CargoWise" |
| **Vlad Bilanovsky** | Chief Execution Officer | ~19 years at WiseTech; "leads the corporate development... via Mergers & Acquisitions and strategic partnerships" |
| **Mark Hall** | Chief Acquisition and Integration Officer | joined 2015 as an IPO advisor; subsequently permanent |
| **Matt Fielder** | Head of Information Systems | "responsible for... the secure operation of our **three data centers in Europe, Australasia and the Americas** delivering our CargoWise platform" |
| **Gene Gander** | General Manager, Global Sales | 20+ years at WiseTech |
| **Angelina McMenamin** | Head of Performance Culture | joined 2014 |
| **Marijana Okanovic** | Head of Corporate Communications | joined 2019 |
| **Steve Murdoch** | Global Head of Marketing | joined 2019 |
| **Damien Sullivan** | General Counsel | interim General Counsel November 2024; General Counsel November 2025 |
| **Gareth Russell** | Chief People Officer | joined 2025 |

Two published facts in this table are load-bearing for the rest of the guide and are therefore flagged: the **CTO has worked on every software generation since 1994** (the continuity claim a buyer reads as roadmap stability), and the vendor states that its CargoWise platform is delivered from **three data centres** across Europe, Australasia and the Americas (the only architecture datum the vendor volunteers on a leadership page; §5).

### 2.3 Size, Attributed and Dated

Size claims are the vendor's, and this guide prints them **only with attribution and date**. From wisetechglobal.com/who-we-are/about-us/ (retrieved 2026-10-12 ✅ — vendor claims):

| Vendor's figure | Vendor's label |
|---|---|
| **20k+** | "customers using our software" |
| **1,827** | "product enhancements in FY26" |
| **39** | "product development centers" |
| **193** | "countries licensed to use our software" |
| **More than 20,000** | "organizations use our software solutions, including **24 of the top 25 global freight forwarders and 46 of the top 50 global third-party logistics providers**" |

The vendor's positioning line on the same page — that its people are "aligned in our strong vision to be the **operating system for global trade and logistics**" — is the company's own self-description (⚠ vendor claim), and the freight-forwarding guide §8.1 already records that phrasing; it is not restated here as a finding.

The guide deliberately prints **no revenue figure and no market-share figure**. This is not an oversight: the repository convention for this guide series is to treat a market-share number as a claim that cannot be verified from the vendor's own pages and would require a third-party estimate the guide will not adopt. §15 records the absence as a finding.

Age of the size claims: the vendor's history page separately records earlier snapshots that show the trajectory, each a **vendor statement for its year**:

- **2020** — "All 25 of the top 25 global freight forwarders and 41 of the top 50 third party logistics providers use our solutions"; six new global-rollout customers including Aramex, Seafrigo, a. hartrodt, cargo-partner, Hellmann and CEVA (✅ vendor history page).
- **2022** — "43 Large Global Freight Forwarder rollouts, of which 10 are in the Top 25 Large Global Freight Forwarders ranking (Armstrong & Associates)"; "$180.8 million invested in research and development"; "19,000 certified CargoWise practitioners worldwide" (✅ vendor history page).
- **2023** — "Increased our global development capability to over 1,800 team members, resulting in 60% of our workforce now focused on product development"; "$261.9 million invested in research and development"; "29,500+ CargoWise Certified Practitioners globally" (✅ vendor history page).
- **2025** — "Closed our largest acquisition, e2open, bringing our total team to more than **7,000 people** around the world" (✅ vendor history page).

The guide reports the **2020** claim of "all 25 of the top 25" (vendor, for that year) alongside the **2026** claim of "24 of the top 25" (vendor, for this year) precisely because they differ: a vendor figure that moves down between snapshots is a useful reminder that these are self-reported counts, not audited statistics. A reader who wants the 2026 figure should use the 2026 one and note it is the vendor's.

Separately, a CargoWise customer-facing news item states "**Over 40,000 logistics professionals are now CargoWise Certified**" (cargowise.com news, "Certified to scale…", dated **22 July 2025** ✅ — vendor claim). The guide notes it as a vendor figure and does not add it to the 20,000-customer claim, because the two count different things.

### 2.4 The Acquisition History, with Announcing Party and Year

This is the section the dispatcher insisted be done properly: **each acquisition verified with its announcing party and year**, or written out and marked ❌. The guide's method was to prefer (a) WiseTech's own announcement, (b) the acquired company's own release, or (c) a dated exchange/press record; where none of those was retrievable this pass, the acquisition is **not** asserted from memory. The vendor's "our businesses" page (wisetechglobal.com/what-we-do/our-businesses/, retrieved 2026-10-12 ✅) confirms **ownership and grouping** of many brands but **does not** carry acquisition dates — the guide uses it as the group map, not as the dating source.

**Verified with a dated announcement and a named announcing party:**

| Business | Announcing party and date | What the vendor/acquirer says it is | Status |
|---|---|---|---|
| **e2open Parent Holdings, Inc. (NYSE: ETWO)** | e2open releases: announced **25 May 2025**, completed **3 August 2025** | Supply-chain/trade SaaS and multi-enterprise network | ✅ — **cross-referenced from the freight-forwarding guide §8.1**; not re-derived here |
| **Blume Global** | **WiseTech Global** announcement, **17 February 2023** (also filed to ASX 17 Feb 2023) | "provider of a leading solution facilitating intermodal rail in North America"; "being acquired from funds managed by **Apollo, EQT** and other minority shareholders" | ✅ (announcing party WiseTech; ASX-dated). The announcement stated a transaction value; **this guide does not reproduce the amount** (see H2 policy, §6.4) |
| **Envase Technologies** | **WiseTech Global** announcement, **24 January 2023** (ASX filing 25 Jan 2023) | "transport management system software for intermodal trucking, drayage (container haulage) and landside logistics in North America"; "acquired from private investment" | ✅ (announcing party WiseTech; ASX-dated). Transaction value stated in the announcement; **amount not reproduced** |
| **Bolero.net Limited** | **WiseTech Global / Bolero** announcement, **2022** | "a leading provider of electronic Bills of Lading (eBL) and digital documentation"; "extends WiseTech's digital documentation offering" | ✅ for the announcement and year; ⚠ terms — a dated trade press report (Global Trade Review) states "Financial terms of the transaction were not disclosed". A secondary data source (MarketScreener) dates completion to 1 July 2022 ⚠ |
| **Pierbridge** | **WiseTech Global** announcement; **2018** | "parcel shipping transportation management solution (TMS) provider to medium and large shippers in the United States", headquartered in Massachusetts | ✅ year — corroborated by a later vendor-partner release stating "In 2018, Pierbridge became part of the WiseTech Global group" (2026) |
| **FRDM** | **WiseTech Global** acquisition; dated press coverage **1 October 2026** | Forced-labour supply-chain risk data (the vendor's news feed carries a Forbes piece on the acquisition, 1 Oct 2026) | ⚠ press-reported/dated; the vendor's own news feed links it, but a standalone WiseTech press release was not retrieved this pass. Also carried in the freight-forwarding/management news feed |

**Group-listed by the vendor, but with no dated announcement retrieved this pass — presented as ownership, not as a dated acquisition:**

The vendor's "our businesses" page groups the following brands into its seven product lines. The guide lists them so a reader can see the *breadth of unrelated software brands the company has absorbed*, but it marks them ⚠ for dating:

- **Forwarding and Customs:** CargoWise (flagship), ACO Informatica (Italy), Bysoft (Brazil), CargoIT (Sweden/Baltics), CustomsMatters (Ireland), DataFreight / LSI Sigma (UK), EasyLog (France), Editrade S.A. (Chile), EXA-System (Japan), Forward (Argentina), Intris (Belgium/Netherlands), LSP Solutions (Netherlands), Opentecnologia S.A. (Colombia), Prolink (Taiwan), Ready Korea (South Korea), SISA (Switzerland), Softcargo (Uruguay), Systema AS (Norway), Taric (Spain), Ulukom (Turkey), znet Group (Germany). **CargoIT** appears here and also has a WiseTech announcement page (announcing party: WiseTech Global) **but the page itself does not carry the announcement date and the release cites 2017 Armstrong data and lists 2018–2019-era "recent acquisitions", so the guide marks CargoIT's year ⚠ not confirmed this pass.** **Bysoft** and **BorderWise** likewise appear in the group list (a WiseTech release names BorderWise among group companies) with **no dated announcement retrieved this pass** — ⚠.
- **Landside Logistics:** Blume Global, CMS Transport Systems (Australia), Containerchain (Singapore), Depot Systems (US), Envase Technologies, MatchBox Exchange, Trinium (US). CargoWise's landside page states "Trinium Technologies and Envase Technologies are now part of CargoWise Landside" (✅ vendor page).
- **Digital Documents:** Bolero International, BSM Global (Australia), ImpexDocs (Australia), Shipamax.
- **TMS:** Pierbridge, SaaS Transportation, SmartFreight (Australia).
- **Specialist WMS:** Microlistics (Australia).
- **Carrier and Rates:** Cargoguide (Netherlands), CargoSphere (US), Softship (Germany).
- **Enterprise:** Inobiz (Sweden), Xware (Sweden).

**Year-level acquisition activity the vendor does publish** (its own history page ✅), useful as an aggregate without naming targets:

- **1999** — "Made our first acquisitions to grow product capability and customer base in Australia."
- **2015** — "$35 million in equity funding to drive long-term growth, including acquisitions in **China and South Africa**."
- **2017** — "Accelerated growth with targeted acquisitions in **Germany, Italy, Singapore and the United States**"; "More than 250 CargoWise Partner organizations actively referring, promoting or implementing CargoWise."
- **2018** — "Accelerated growth with **15 valuable acquisitions** across Asia Pacific, Europe and the Americas."

**Deliberately excluded:** the guide does **not** enumerate the further businesses a third-party database (e.g. Tracxn, retrieved 2026-10-12 ⚠ third-party) attributes to WiseTech — including "**Customscentre**" and any additions beyond those above — because the guide could not confirm them against a vendor or dated announcement this pass. Tracxn's framing ("41 acquisitions") is noted only as a **third-party count** and is not adopted as the guide's own. §15 records this explicitly.

**What the acquisition history implies — structurally, attributed, not inferred:** the vendor states the strategy in its own words: "We are evolving rapidly. Expanding into more products, deeper functionality, more geographies and adjacencies, driving our long-term growth and market position with each new innovation and acquisition" (about-us, 2026-10-12 ✅ vendor claim); and its businesses page states "With a strong track record in successful acquisitions and integrations around the world, we will continue to invest in businesses where we see value and close product and strategic alignment" (✅ vendor). The structural reading a buyer draws — stated here as the guide's reading, clearly labelled — is that **the group's product direction is set by accretion as much as by organic build**: the platform is the flagship, but the company's surface area has grown by buying regional customs/forwarding products, landside/container systems, parcel TMS, eBL and trade-documentation assets, and data-integration middleware. §5 (integration) and §8 (competitive frame) pick up what that structure means for a client. The guide does **not** infer internal product roadmaps from acquisitions.

### 2.5 The Governance and Conduct Note

The rule this guide follows: **describe corporate facts only, factually, narrowly, with a dated source — neither ignore a material matter nor dwell on it.**

- **The ACCC matter is already established elsewhere in this repository.** A **search warrant executed by the Australian Competition and Consumer Commission (ACCC) at WiseTech** was **reported 19 August 2026**, in a competition-law investigation concerning global logistics services and software; it is recorded in the **freight-forwarding guide §8.1** (⚠ press-reported with date, not verified against the ACCC's own register). **This guide cross-references it rather than re-deriving it.** The buyer-relevant framing is in the freight-forwarding guide §13: when much of an industry runs on one vendor's platform, that vendor's regulatory behaviour becomes a **concentration risk for its customers and their banks** — which is why the ACCC scrutiny matters beyond WiseTech's shareholders.
- **Founder/leadership transition, as the vendor publishes it** (leadership page, 2026-10-12 ✅): Richard White was CEO and Executive Director **until October 2024**, was Executive Chair **February 2025 to July 2026**, and remains an **Executive Director**; **Zubin Appoo** became CEO in **July 2025**. These are corporate-fact statements from the vendor's own page and are the only governance-transition facts this guide asserts.
- **Not included:** this guide does **not** carry the personal allegations and sensational material that circulate around the founder in press and third-party encyclopaedia sources; that material is **not** corporate fact, is **not** material to a buyer's vendor assessment in the form those sources present it, and is **excluded by this guide's scope rule**. A reader assessing **vendor risk** should reason from the enterprise facts — ownership, integration depth, commercial model, regulatory matters, and the published governance framework — not from personality coverage.
- **What the vendor does publish on governance** (corporate-governance page, 2026-10-12 ✅): a Corporate Governance Statement, Board Charter, Audit & Risk Committee Charter, Nomination Committee Charter, People & Remuneration Committee Charter, the Constitution, Risk Management Principles, a Code of Conduct, Anti-bribery and Corruption Policy, Securities Trading Policy, Market Disclosure and Communications Principles, Diversity and Inclusion Principles, Whistleblower Protection Principles, Human Rights Principles, a Supplier Code of Conduct and an annual Modern Slavery Statement (filed on the Australian Border Force modern slavery register). The vendor states that all employees commit to the Code of Conduct and complete refresher training every two years, and that acquired businesses are brought under the policies. **That the framework exists is a published fact; its effectiveness is not something an external reader can verify from the page, and this guide does not claim it.**

---
## 3. The Product and the Single-Database Model

### 3.1 What the Platform Is

CargoWise is a **logistics execution and management platform** — the system a logistics service provider runs its business on. In the freight-forwarding guide's vocabulary (§8.1), it is the forwarder's **job-and-document coordination platform** plus its accounting, customs, warehouse and transport layers. The vendor's own one-line description is: "CargoWise is our deeply integrated, global software platform that provides logistics service providers with powerful productivity, extensive functionality, deep compliance capabilities and truly global reach" (wisetechglobal.com/investors, retrieved 2026-10-12 ✅ — vendor claim). CargoWise's own site adds the architectural framing: "All modes. All borders. **One platform.**" and "The single platform that powers global logistics from origin to destination" (cargowise.com, 2026-10-12 ✅ — vendor claim).

The platform is positioned to serve logistics service providers rather than shippers directly: the customer is the forwarder, the customs broker, the 3PL or the NVOCC/consolidator. The **shipper** is reached through a customer-facing portal layer (CargoWise Neo, §4.3) rather than by buying the platform.

### 3.2 What One Global Shared Database Actually Means Operationally

This is the heart of the guide. The vendor's central architectural claim is that CargoWise runs on **one global database** shared across a customer's operations, and the vendor states it in operational terms rather than as a technical diagram:

- "Unify your logistics operations on a **single global database** — pay for one system, learn one system and maintain one system." (cargowise.com, "One platform. Zero duplication.", 2026-10-12 ✅ — vendor claim).
- "Connect instantly with customers, partners, carriers, and government systems — **without the cost and complexity of custom integrations**." (cargowise.com, "Built-in integrations. Total connectivity.", ✅ — vendor claim).
- "Access and share **data in real-time** to respond sooner, plan smarter, and reduce operational risk across your supply chain." (cargowise.com, ✅ — vendor claim).

Translated — and this is the guide's explanation, not the vendor's — a single shared database means the following *operational* properties, each of which is a claim about architecture that the vendor asserts and the guide renders as claimant's logic rather than as verified fact:

| Property the vendor's "single database" claim implies | What it means day-to-day for a forwarder | Why it matters to a buyer |
|---|---|---|
| **One record of the job** | The shipment exists once, keyed once, and every office, mode and currency updates the same record rather than each country system holding its own copy reconciled by file transfer. | Eliminates per-office reconciliation of the *same* shipment — but also means the shipment's history is inseparable from *this* platform. |
| **One accounting ledger** | Costs, accruals, billing and multi-currency accounting post to one ledger tied to the same job records. | The general-ledger history becomes part of the lock-in: leaving the platform means leaving the ledger's lineage, not just the screens (see §7). |
| **One reference master** | Carriers, ports, agents, tariff/classification references, currencies and languages are maintained once and shared. | Reference data drift — a classic cause of document discrepancies — is structurally reduced, but the customer has *one* place to be right or wrong. |
| **No per-office reconciliation** | Branches in different countries do not run separate instances that must be reconciled across a boundary (intercompany, currency, document numbering). | Multi-entity accounting and intercompany work are handled in-platform (the vendor sells "multi-entity accounting" as a capability, §3.3), which is exactly the capability a growing forwarder does not want to re-implement. |
| **One upgrade path** | The vendor upgrades the platform as a whole; the customer is on the vendor's release train rather than maintaining many divergent installations. | A release-train dependency: the customer gains turnkey upgrades and loses the ability to freeze a version for years at no cost (the vendor's community pricing page carries a *non-current-version surcharge* structure, §6.4 — structure, no amount). |
| **One trained workforce** | An operator trained on one instance is trained on all; certification is transferable across the customer's own network. | Staff competency is a platform asset — and, from the client's side, the reason migration is expensive (retraining the whole population). |

The vendor's customer quotes reinforce the *shape* of the claim (all vendor-published testimonials, so ⚠ vendor claims): "one shipment in CargoWise... you can share the same information, the same documents, everything in the **same database**" (CEVA, quoted on cargowise.com/solutions/cargowise-forwarding/, 2026-10-12) and "CargoWise is the most highly qualified system I have ever seen for global use... every function and every module are developed together" (LEMAN, same page). These are the vendor's testimonials and are treated as claims.

**The honest caveat:** the vendor's claim that the database is *single and global* is exactly the kind of architectural assertion that a buyer cannot verify from outside. §3.5 and §11 carry that caveat; the guide does not assert the claim as fact, it reports it as the vendor's and examines what would follow *if* it holds.

### 3.3 The Module Set, as the Vendor Describes It

CargoWise's own solutions menu (cargowise.com, retrieved 2026-10-12 ✅ — vendor descriptions) presents the platform as this set of solution areas. The guide lists them as **the vendor describes them** and marks each as a vendor claim:

| Solution area | The vendor's own description (quoted/paraphrased) |
|---|---|
| **International Freight Forwarding** | "Automate and optimize your operations across modes and borders, from capacity planning and rate management to vessel booking and last-mile trucking." Includes **Air**, **Ocean**, and **Road and Rail** sub-solutions, with schedules search, buy/sell rate comparison and eBookings convertible to a B/L or AWB. |
| **Customs & Compliance** | "Simplify border compliance and stay on the right side of the law, no matter where you operate or where your goods are headed." Includes customs declarations, bonded-warehouse integration, advance filing/cargo security, **ComplianceWise** (screening and audit controls) and **BorderWise** (goods classification / border-compliance reference). |
| **International eCommerce** | "Keep pace with the rapid growth of ecommerce and rising supply chain demands..." — clearance of high-volume, low-value and dutiable shipments. |
| **Warehouse** | "Track and manage inbound and outbound inventory with a powerful, flexible warehouse management system." |
| **Landside** | "Manage marine and intermodal containers across trucking, rail, ocean, and logistics partners with integrated planning, execution, and visibility." (See §5.3 and the container-optimisation products.) |
| **Transport** | "Uncover bottlenecks and optimize delivery by planning capacity, loads, and routes, with real-time alerts and ETA updates..." |
| **Carrier connectivity** | "Automate your order to cash cycle from booking to invoice by putting your entire suite of services at the fingertips of the world's largest forwarders." (Sold to carriers/airlines to connect their capacity to forwarders.) |
| **Enterprise tools** | "Supported by integrated tools for documentation, data accuracy, tracking, and accounting..." — sub-tools the vendor names: **Accounting**, **Human Resources Manager**, **Customer Relationship Manager**, **Workflow and Productivity**, **eDocs** (document/image management) and **Customer Ticket Portal**. |

The vendor reports global capability counts on the platform home: "**30 languages**, **162 currencies**, **193 countries**" (cargowise.com, 2026-10-12 ✅ — vendor claims), and on the forwarding page a development figure: "**5,700+ Product enhancements added to the CargoWise application suite over the last 5 years**" (✅ — vendor claim). The customs page carries a coverage goal: "Our goal is to provide coverage of **90% of the world's international manufactured trade flows**" (✅ — vendor claim, goal). The enterprise page frames the accounting module as "a deeply integrated, enterprise-grade, accrual-based accounting system" (✅ — vendor claim).

The guide prints these figures **as the vendor's**, with the date, and draws no conclusion from them other than that the vendor markets around breadth and continuous enhancement. A feature catalogue of this size is, by the thesis of this guide, **the thing that is not the moat**.

### 3.4 The Personas It Serves

CargoWise is a *multi-persona* platform. The personas the guide uses, and what each is:

- **Freight forwarder** — arranges carriage; the primary buyer. Runs jobs, documents, accounting and customer service on the platform.
- **Customs broker** — licensed intermediary filing declarations; served by the customs module and the Stand-Alone Customs Value Pack (§6.2).
- **3PL (third-party logistics provider)** — contract-logistics operator combining warehousing, transport and value-added services; served by the warehouse, transport and landside layers (see [clpa_contract_logistics_guide.md](clpa_contract_logistics_guide.md) for the 3PL engagement model and [logistics_warehouse_management_guide.md](logistics_warehouse_management_guide.md) for warehouse operations — cross-referenced, not re-derived).
- **NVOCC / consolidator** — carrier-facing and consolidation personas; served by the liner/agency and container-freight-station layers (see the decoder, §1.2).
- **Carrier-side** — airlines, ocean carriers and their sales agents use **Carrier connectivity / AirlineConnect** to publish capacity and take bookings from forwarders (§5.3).
- **Landside / drayage / depot operator** — served by the CargoWise Landside products (drayage TMS, container exchange, yard management).
- **The shipper (via a portal)** — the forwarder's *customer* accesses a branded portal (CargoWise Neo) for quotes, bookings, tracking, documents, invoices and warehouse self-service; the shipper does not buy the platform.

### 3.5 No Independent Architecture Disclosure

The guide states this plainly because the dispatcher required it: **no independent architecture disclosure exists that this pass could retrieve.** What the public record offers is (a) the vendor's marketing-level claim of "one global database" and "one platform", (b) a single volunteered hosting datum — the Head of Information Systems' bio stating the platform is delivered from "**three data centers in Europe, Australasia and the Americas**" (wisetechglobal.com/investors/our-leadership-team/, 2026-10-12 ✅ — vendor claim), and (c) the vendor's own integration *product* names (§5.2). There is **no published database schema, no published data model, no published multi-tenancy description, no published infrastructure whitepaper, and no independent architectural audit** that this pass could find. Everything the guide says about "the single database" is therefore **the vendor's claim**, analysed for its commercial consequences, not verified as engineering fact. §5.1 enumerates exactly what is and is not published; §11 and §15 carry the limit.

---

## 4. CargoWise Next

### 4.1 The Vendor's Own Naming of CargoWise Next

The **vendor names CargoWise Next on its own history page** as the platform's fourth generation: the vendor's 2025 milestone reads "Rolled out our **4th generation platform, CargoWise Next**." (wisetechglobal.com/who-we-are/our-history/, retrieved 2026-10-12 ✅ — vendor statement). This is the strongest source for CargoWise Next this pass: the vendor's own corporate site calling it the 4th-generation platform rolled out in 2025.

What the vendor **does not** publish on the pages reached this pass is a dedicated CargoWise Next product page: the guessable URL `cargowise.com/cargowise-next/` returned a **404 "Page not found"** (retrieved 2026-10-12 ❌ — page not found, recorded as a tool/page limitation, not as evidence of absence). The guide therefore:

- **asserts** that CargoWise Next is the vendor's name for its fourth-generation platform, rolled out in 2025, per the vendor's own history page (✅ vendor statement); and
- **does not** describe CargoWise Next's internal feature set, architecture or roadmap, because the vendor's own dedicated material was not retrievable this pass. Third-party pages that describe CargoWise Next (software marketplaces, consulting blogs, LinkedIn) are **not** used as the vendor's own description and are marked ⚠ third-party in §14; the guide does not present a third-party description as though it were the vendor's.

This is the honest position the dispatcher asked for: where the vendor's own material could not be reached, the guide **says exactly that** and does not fill the gap. §15 records it.

### 4.2 The Value Packs as an Adjacent, Dated Evolution

CargoWise Next is not the only named evolution. The vendor's **2025–2026 commercial-and-product move is the "CargoWise Value Pack" model**, which the vendor does publish in detail (cargowise.com/cargowise-value-pack/, retrieved 2026-10-12 ✅ — vendor). The Value Packs are examined for their commercial *structure* in §6; here the point is that the vendor frames them as an evolution of the platform's commercial packaging alongside product enhancement:

- "Our new model is much simpler, more transparent, and more predictable, giving you access to the full suite of powerful CargoWise features and modules." (✅ vendor claim).
- The vendor counts the packaged capability as "**125+** new functions/modules/features in the past year", "**198+** capabilities", "**225+** Forwarding/Customs/Warehouse/Land Transport related functions and modules", and "**116+** new software capabilities targeted at your customers' needs" (✅ — all vendor claims, dated 2026-10-12).
- The vendor states the model "replaces the prior **STL model**" and moves to a published community model charged "per eligible logistics transaction" (✅ vendor claim, §6).
- The vendor states that AI tooling "have been made available **from 1 December 2025**", and that an "**Agentic AI Workflow Engine** and **AI Management Engines** capabilities will roll out over the coming months" (✅ vendor claim, dated as the vendor's statement) — a forward-looking claim the guide records without asserting it as delivered.

### 4.3 CargoWise Neo

**CargoWise Neo** is the vendor's customer-facing portal layer, and the vendor does publish a dedicated page for it (cargowise.com/solutions/cargowise-neo/, retrieved 2026-10-12 ✅ — vendor). As the vendor describes it: "Neo is more than just a dashboard — it's a command center for your customers' entire supply chain operations," "fully web-based and live", integrating with CargoWise so that the forwarder's *customer* sees transactions, shipments, declarations and orders; it offers track-and-trace, interactive mapping, e-conversation messaging, a "forwarder portal" with security groups and brand-identity configuration, a "staff portal" for the forwarder's own users, and modules for forwarding, customs, warehouse and accounting self-service. The vendor also markets "Neo Mega features" (route visualiser, cargo tracker, performance reports, order manager) and explicitly pitches a switch: "Already using a third-party platform? Consider the switch to CargoWise Neo." (✅ vendor claims throughout).

The guide's reading — labelled as the guide's, not the vendor's — is that Neo is where the "database is the moat" thesis becomes *commercially visible*: the portal is valuable to the forwarder's **customer** (visibility, self-service) and it is *wired into the same database* as the forwarder's operations. A forwarder that has put its customers onto Neo has added a second, outward-facing layer of switching cost — the customer's own experience is now an argument against the forwarder leaving the platform (developed in §7). The guide does **not** assert that Neo is a finished or complete portal suite; it reports what the vendor publishes.

### 4.4 What the Guide Will Not Invent

To keep this section honest:

- The guide will **not** invent a "CargoWise Next roadmap", a set of Next-specific modules, a migration timeline, or a claim that Next changes the single-database architecture. None of that was sourced.
- The guide will **not** present the vendor's forward-looking AI statements ("Agentic AI Workflow Engine", "AI Management Engines", "significant labor reduction over time") as delivered outcomes. They are recorded as **the vendor's statements of intent**, dated.
- The guide will **not** treat "125+/198+/225+/116+" as audited counts. They are vendor marketing counts, dated, and are printed as such.

---

## 5. The Architecture and the Integration Surface

### 5.1 What Is Publicly Documented and What Is Not

The dispatcher's rule for this section is that the **unpublished** parts be stated as unpublished, and that the guide never infer internal architecture. Applying it:

| Layer | Publicly documented? | What the public record actually shows (with status) |
|---|---|---|
| **Single global database** | ❌ Not documented technically | Only the vendor's marketing claim ("one global database", cargowise.com ✅ vendor claim). No schema, data model or tenancy description published. |
| **Hosting footprint** | ⚠ Partly, one datum | The vendor's Head of Information Systems states "three data centers in Europe, Australasia and the Americas" (leadership page ✅ vendor claim). No infrastructure whitepaper retrieved. |
| **Tenancy model (multi-tenant vs single-tenant)** | ❌ Not published | The vendor's community pricing page distinguishes "**CargoWise Cloud**" from "**Customer Hosted**" and "**Private Cloud**" and "**Self Hosted**" configurations (✅ vendor page), implying multiple hosting modes — but the *architecture* of each is not described. The guide does not infer multi-tenancy from "single database". |
| **Integration mechanism** | ⚠ Partly, via partners and third parties | Named integration products (eAdaptor, Universal XML, xHub) appear in the vendor ecosystem and third-party/partner documentation (§5.2). The vendor's own open developer portal was **not retrievable** this pass (`developer.cargowise.com` refused by the extraction tool ❌ tool limitation). |
| **API surface / endpoints** | ❌ Not published openly | No public, openly-accessible CargoWise API reference was retrieved this pass. Partner material states integrations run through eAdaptor/Universal XML rather than a simple REST API (§5.2). |
| **Data export / egress tooling** | ⚠ Partly, as a paid service structure | The vendor's community pricing page lists optional services including "CargoWise Cloud Read Only Access (Optional Service to allow direct SQL queries / customized reports etc.)", "CargoWise Cloud Tear Downs, Data Returns, Snapshots", and a "Full Backup Mounted ... Available For Secure File Download" item (✅ vendor page — structures recorded without amounts, §6.4). |
| **Source code / escrow** | ❌ Not published | No statement about source-code escrow was found this pass. Recorded in §15. |
| **Security certifications (e.g. SOC 2, ISO 27001)** | ❌ Not retrieved this pass | No certification page was reached; the vendor publishes governance documents (corporate-governance page ✅) but this pass did not confirm a specific security certification. Recorded in §15. |

### 5.2 The Integration Path: eAdaptor, Universal XML and the Partner Layer

The public record on CargoWise integration is **partner-mediated**, and the guide reports it as such:

- **The vendor publishes a global partner network with typed tiers and competences** (cargowise.com/partners/find-a-partner/, retrieved 2026-10-12 ✅ — vendor). The types are **Service Partner**, **Business Partner**, **Industry Partner** and **Education Partner**; the competence areas include **Accounting, Customs, Forwarding, Global Rollouts, Integration, PAVE, Warehouse and Workflow**; the competence tiers are **Platinum, Gold and Certified**. The directory is searchable by named partners (e.g. firms such as *Cen Global*, *Optimize Logistics Solutions*, *Emerald Freight Consulting*, *Integrity Matters* are listed with certified-staff counts and competences — ✅ vendor directory). The vendor markets Service Partners as the route to implementation: "Service Partners are logistics and IT experts who will guide you through your CargoWise implementation" (✅ vendor).
- **The named integration mechanism is eAdaptor / Universal XML, not a public REST API.** The *vendor's own ecosystem* names **eAdaptor** (and **xHub**) as the interface technology; a CargoWise **Integration** Platinum Partner, **Chain.io**, states plainly that "CargoWise connects through **eAdaptor and Universal XML**, not a simple REST API" and sells middleware that translates modern APIs into CargoWise XML (chain.io/cargowise-api-documentation/, retrieved 2026-10-12 ⚠ — a partner with a commercial interest in middleware, quoted for the *mechanism* it names, not for its product pitch). A third-party technical upload of an "**eAdaptor Developer Guide**" (dated 11 March 2024, Scribd ⚠ third-party) corroborates the names and describes "connectivity options, XML formats, and concepts like **code mapping and validation**" — i.e. the integration surface is an XML-and-mapping interface documented to partners, not an open REST developer platform.
- **What this means for a buyer** (the guide's reading): CargoWise integration is real, mature and widely implemented, but it is **partner-mediated and mapping-heavy**, and the *mappings* are the asset. A client's integrations are not a clean public-API surface it owns; they are mappings (code mapping, validation rules, XML schemas) frequently built and held by a Service Partner. That is one of the sharpest practical consequences of the platform's design and is developed in §7 and §9.

The guide asserts **only** the mechanism names (eAdaptor, Universal XML, xHub) and the partner structure, from the sources above; it does **not** assert the API's completeness, its versioning policy, its rate limits, or its documentation quality, none of which were verifiable this pass.

### 5.3 Named Integrations with Dated Announcements

The vendor's news feed carries integration announcements the guide can date (cargowise.com/news and cargowise.com/landside/news, retrieved 2026-10-12 ✅ — vendor announcements):

| Integration | Date (vendor announcement) | What the vendor says (paraphrased) |
|---|---|---|
| **PortConnect (New Zealand) Port Community System** | **1 October 2026** | "CargoWise integrates with PortConnect NZ Port Community System enabling more efficient landside container movements"; "Container level data and connectivity help derisk quay-landside movements." |
| **World Freight Company — ATC Aviation GSSA network → CargoWise AirlineConnect** | **30 September 2026** | Forwarders "can access a wider range of airline capacity, services, dynamic pricing and eBooking through ATC on CargoWise AirlineConnect." |
| **ShipDNA (Service Partner) — Safe Passage™ Clippership → Transtream migration program** | **29 September 2026** | A partner-led program to move legacy **Clippership** shipping-software users to the **Transtream** platform (a Pierbridge-era WiseTech product). |
| **Direct carrier connections (airline logos)** | Vendor page, retrieved 2026-10-12 | CargoWise's forwarding page shows direct-connection airline marks including Air Canada, ANA, Air France, Avianca, British Airways, Cathay Cargo, Emirates, Iberia, Finnair, Etihad, ITA, KLM, Korean Air, Lufthansa, Qatar, Turkish, United, Air China, Air New Zealand, Ethiopian and LOT (✅ vendor page — vendor's own customer/connection marks). |
| **Bolero eBL ↔ CargoWise** | Vendor relationship | Cross-reference: the **freight-forwarding guide §8.1** already records the Bolero (Galileo) eBL/CargoWise integration (⚠ vendor claim) and **Bolero.net Limited is listed among WiseTech's own group businesses** (§2.4 ✅). **This guide does not re-derive the eBL point; see §10.1 for the banking relevance.** |
| **FRDM (forced-labour risk data)** | Dated press, **1 October 2026** | ⚠ press-reported; the vendor's news feed links a Forbes piece on the FRDM acquisition and its supply-chain-risk/data angle. |

The pattern the guide reads from these (labelled as the guide's reading): the vendor's integration announcements cluster at the **edges** of the platform — a national port community system, an air-cargo GSSA network, a partner-led legacy migration, and a supply-chain-risk data acquisition — which is consistent with the thesis that the *core* differentiator is the shared database (§3.2) while the *announced activity* is about widening the set of things the database connects to.

### 5.4 The Hosted and Multi-Tenant Model, as Described

The vendor's community pricing page is the clearest public statement of the **hosting modes** (cargowise.com/cargowise-value-pack/cargowise-value-pack-community-pricing/, retrieved 2026-10-12 ✅ — vendor). From it the guide records these **structures** (amounts deliberately omitted, §6.4):

- The default offering is **CargoWise Cloud**, and the vendor states that under the Value Pack model "standard CargoWise Cloud hosting fees" are included for many customers (✅, §6.2).
- The page separately prices a **"CargoWise Cloud Base, Customer Hosted (Production/Non-Production) Minimum Fee - Per Private Cloud Instance"**, which confirms that **Private Cloud and Customer Hosted / Self Hosted** configurations exist as alternatives to the standard cloud (✅ vendor).
- Optional services confirm hosted-mode realities: a **"CargoWise Cloud Read Only Access"** service "(to allow direct SQL queries / customized reports etc.)", an **"Additional CPU Core/MS SQL Enterprise License"** for scaling customised reports/external SQL, **"Private Line Termination (Read Only and User Data Store access)"**, **"Legacy Email Hosting"**, **premium backup** services and **"CargoWise FailOver Services For Self Hosted Customers"** (✅ vendor).
- The vendor states a **non-current-version surcharge** structure for Private Cloud/Self-Hosted customers who are not on a "Supported Production Release", with an initial surcharge plus an additional per-release-not-upgraded surcharge — indicating a **release-train discipline** rather than per-customer frozen versions (✅ vendor — structure, no amounts).
- Two facts are volunteered on the leadership page: the platform is delivered from **three data centres** (Europe, Australasia, the Americas), and the Head of Information Systems owns "the secure operation" of them (✅ vendor claim).

What the guide **does not** claim: it does not claim the platform is single-tenant or multi-tenant, does not claim where data physically resides for a given customer, and does not claim a specific DR posture. The vendor page implies **multiple hosting modes and a standard cloud**, and that is the limit of what the sources establish.

### 5.5 The Data-Integration Questions a Client Faces

Setting aside vendor claims, a client integrating with and running on CargoWise faces a recurring set of **data questions**, and the guide states them as the questions (the guide's framing), noting where the vendor publishes an answer and where it does not:

| Question a client must answer | Does the vendor publish an answer? |
|---|---|
| **Who owns the data?** | Not stated on the pages reached this pass ❌. The vendor's terms of service would be the instrument; the guide did not retrieve them. |
| **Can the client export its data, and in what form?** | ⚠ Partly. The community pricing page lists *paid* "Data Returns", "Full Backup Mounted ... Secure File Download", "Snapshots", "Read Only Access (direct SQL)", and "Tear Downs" as optional services (✅ vendor). The *default* export format and completeness are not stated. |
| **Who holds the integration mappings (code maps, validation, XML schemas)?** | Not stated ❌. The partner-mediated model (§5.2) implies mappings frequently sit with a Service Partner, but the vendor does not publish a client-owned-mapping guarantee. |
| **What happens at termination — data return, deletion, timeline?** | ⚠ Partly. A "**CargoWise Cloud Service Tear Down (remove instance from CargoWise Cloud/delete data)**" is priced per main database, and backup mounting is priced and "held for two weeks" (✅ vendor page). The contractual *right* and *timeline* for data return are not published; only the service structure is. |
| **Read-access for the client's own analytics?** | ⚠ Partly. Direct SQL read access exists as a charged optional service ("CargoWise Cloud Read Only Access", plus CPU/SQL-licence scaling) (✅ vendor). This implies the *default* posture is not open read access to the underlying database. |
| **Egress / bandwidth cost?** | ⚠ Implied, not enumerated. The cloud model prices CPU, data transfer and private-line termination as chargeable items (✅ vendor page). |
| **Data-residency for a given jurisdiction?** | Not stated on the pages reached ❌. The three-data-centre datum (§5.4) is the only footprint statement found. |

The guide's conclusion, stated as its reading: for CargoWise, **the database is the moat and the *documented* egress path is commercial rather than automatic** — read-only SQL access, backups and tear-downs are *paid services* on the vendor's published page, which means a client that wants its data out can get it, but the vendor's public documents frame extraction as a **chargeable engineering event**, not a self-service export button. That framing is itself the finding. Whether the contract guarantees faster, freer egress is a question only the client's own agreement can answer (§7.4, §9.2). Cross-reference [../technology/data_governance_guide.md](../technology/data_governance_guide.md) for the discipline of owning data lineage and egress as an enterprise control, and [../technology/api_governance_guide.md](../technology/api_governance_guide.md) for governing a partner-mediated integration estate rather than an owned API surface.

---
## 6. The Commercial Structure

**Rule for this section (stated up front so the absence is not mistaken for an omission):** this guide describes commercial **structures** and prints **no amounts, no revenue figures and no total-cost-of-ownership comparison**. Where the vendor *does* publish a list price, the guide records **that the vendor publishes it** and describes its *shape* — it does not reproduce the numbers. This is a deliberate editorial decision, consistent with the repository's treatment of vendor pricing, and it is recorded as the finding in §6.4.

### 6.1 The Legacy Structure: Seats, Modules and Hosting

The vendor names its prior commercial model in its own FAQ: "The CargoWise Value Pack model is our new commercial model which **replaces the prior STL model**." The vendor describes that prior model's components as: "The **STL model combined transaction-based charges with individual module and feature fees, seat fees and cloud hosting fees**, which meant it could be difficult for customers to accurately estimate their monthly CargoWise costs." (cargowise.com/cargowise-value-pack/, retrieved 2026-10-12 ✅ — vendor statement about its own prior model.)

As *structures*, the legacy model therefore comprised:

| Legacy structure | What it is |
|---|---|
| **Per-user / seat fee** | A charge per named user, so headcount and adoption drove cost. |
| **Per-module / per-feature fee** | A charge for each module or feature, so expanding functionality drove cost. |
| **Per-transaction charge** | A charge tied to transaction volume. |
| **Cloud hosting fee** | A separate charge for the standard CargoWise Cloud hosting layer. |

The guide records that this *combination* of structures was the vendor's own description of why customers found estimating cost hard — a transparently commercial admission that the pre-2026 model had **predictability** as a weakness. That is structural, and it is the vendor's own word.

### 6.2 The Value Pack Model, as Publicly Described

The vendor's 2025–2026 model is the **CargoWise Value Pack**, and the vendor publishes its structure (cargowise.com/cargowise-value-pack/, retrieved 2026-10-12 ✅ — vendor). As described:

- **Charge basis:** "charged at a specific and published **Community Price per eligible logistics transaction**" — i.e. a **per-eligible-transaction** structure rather than seats-plus-modules (✅ vendor).
- **Named packs (four, per the vendor FAQ):**
  1. **Forwarding Value Pack** — "all freight forwarding and related customs procedures, including formal import and export customs procedures."
  2. **Stand-Alone Customs Formal Entry / Customs Broker Value Pack** — "all formal import and export customs-related features."
  3. **Contract Warehouse Manager Value Pack** — "Contract (Product based), eCommerce and Duty Deferral Warehouses, and all related features."
  4. **Land Transport Value Pack** — "all capabilities related to land transport."
- **Inclusions the vendor states:** "**no more User Seat Fees**" for many customers ("Whether you have 10 staff or 10,000, your teams can work freely across locations and functions without restriction. No per-user licenses, no incremental costs"), and "**none of the standard CargoWise Cloud hosting fees**" for many customers (user count, database size, CPU and data transfer not separately capped in the standard cloud) (✅ vendor claims).
- **Granularity:** the vendor's community pricing page shows the *charge basis* differs by pack — **per shipment** (banded by mode and direction: FCL/FTL vs LCL/LTL/AIR/NON-FCL; and by Gateway/Transit/Transfer/CrossTrade vs Export vs Import) for Forwarding; **per formal procedure** for Customs; **per order line** and per work order / BOM / kitting request for the warehouse pack; **per container / per consignment** for land transport; **per package** for the eCommerce pack; **per booking / bill of lading** for the liner pack; **per container pack/unpack** for the CFS pack; **per container booking** for gate/yard/VBS; and **per CTO booked container** for Container Transport Optimisation (✅ vendor page — structure described, **amounts not reproduced**).
- **Vendor's framing of the switch:** calling it "much simpler, more transparent, and more predictable" and stating that "Charging per transaction directly links cost to value" (✅ vendor claim).
- **Training inclusion:** the vendor states it is "including all WiseTech Academy courseware — except a small number of third-party courses — at no additional cost in the CargoWise Value Packs" (✅ vendor claim).
- **Exclusions:** the vendor states "most forwarding and formal customs capabilities are included at no additional charge, with only a small set of **Excluded Items and Optional/Premium Services** billed separately" (✅ vendor claim).

### 6.3 The Pass-Through Disbursement Structure

The most commercially interesting element of the new model is **not** the per-transaction basis but the **pass-through disbursement** default, and the guide describes it as a *structure* (cargowise.com/cargowise-value-pack/, 2026-10-12 ✅ — vendor):

- Under the **default configuration**, the Value Pack Transaction Fees for the **Forwarding** and **Stand-Alone Customs Formal Entry / Customs Broker** packs are "presented as a **pass-through disbursement** on the logistics customer's invoice as the '**CargoWise Automation Fee**'" (✅ vendor).
- The vendor's stated rationale: "Because disbursements typically post to **balance sheet accounts** rather than directly through the P&L, passing on CargoWise costs to your customers **may positively impact your margins**." The vendor notes many customers "have begun allocating software automation costs directly to their customer invoices, often as a disbursement", and that the new model "supports and enhances this important choice by allowing you to remove overheads from your P&L" (✅ vendor claims).
- **Discretion retained:** the vendor states the forwarder "retain[s] the ability to modify this disbursement" and that doing so books the expense through the P&L instead (✅ vendor).
- **The other two packs do not do this by default:** "Other packs such as **Contract Warehouse Manager** and **Land Transport** do not push charges to the customer invoice by default." (✅ vendor).
- **Uniformity claim:** the vendor states "**Every CargoWise logistics provider will see the same CargoWise Automation Fee** regardless of the size of the logistics provider or their customer" (✅ vendor claim).

The guide's reading, stated as its reading: this is a **commercial structure that turns the platform's cost into a line on the forwarder's *customer's* invoice**, positioned by the vendor as a disbursement (a balance-sheet item) rather than a P&L overhead. For a bank financing a forwarder's working capital or a shipper's trade, the structural question is not the amount but the **mechanics**: a cost that flows to the end customer's invoice via the forwarder's agency introduces a new line item with its own audit and disclosure character, and the vendor's stated "same fee for everyone" is a *claim of uniformity* the guide records without verifying. The guide prints **no amount** for the CargoWise Automation Fee (see §6.4).

### 6.4 The Published Community Pricing Page and the Absence Finding

Here is the finding the dispatcher asked for, stated plainly:

- **This vendor now publishes a list-price page.** CargoWise's "Value Pack Community Pricing" page (cargowise.com/cargowise-value-pack/cargowise-value-pack-community-pricing/, retrieved 2026-10-12 ✅) publishes per-unit list prices in USD for each pack, a table of **Optional Services**, an **Extended Credit Terms Surcharge** schedule (banded by payment term), and a **Non-Current Production Version Surcharge** schedule (an initial percentage plus an escalating per-release-not-upgraded percentage for Private-Cloud/Self-Hosted customers who fall off a "Supported Production Release"). The page is version-stamped ("version 26.10").
- **That the vendor publishes list prices is itself notable in this category.** In enterprise logistics-software procurement, public price lists are *unusual* — pricing is typically negotiated, quoted privately, and structured per deal. The vendor moving from the STL model to a "*published* community model" ("moves pricing from the prior STL model to a **published community model** that charges per eligible logistics transaction" — ✅ vendor) is a *structural* change of commercial posture: **from a negotiated, opaque-cost model to a published, per-transaction one.** The guide records that shift and its direction; it does **not** reproduce the prices.
- **The absence this guide records as the finding:** this guide prints **no CargoWise amounts and no legacy-model prices.** The legacy STL model's **actual** price levels are not published anywhere the guide could find (the vendor describes the legacy *structure*, not the legacy *prices*), so for a client on the legacy model, historic price levels are **not publicly knowable** — recorded in §15. For the new model, the prices *are* published, but reproducing them would breach this guide's no-amount rule; the reader is directed to the vendor's own page.
- **The structures the guide *does* record without amounts** (all ✅ vendor page, as structures): the per-transaction banding described in §6.2; the Optional Services menu (private-cloud instance minimum, secured links/VPN, customer-specific URL, read-only SQL access, additional CPU core/MS-SQL licence, private line termination, legacy email hosting, premium backup runs, failover services "POA", code/hosting changes, accounting-period changes, device testing/staging, backup mounting, service tear-down); the Extended Credit Terms Surcharge banding; and the Non-Current Version Surcharge structure.

The guide also records, as a **structure with no amount**, the vendor's statement that "Under no circumstances will payment terms **exceeding 60 days** from the invoice date be granted. Applicable surcharges are fixed and are not subject to negotiation." (✅ vendor) — a contractual term, not a price.

---

## 7. The Switching Cost and the Lock-In Question

The freight-forwarding guide §13 already makes the generic point: an incumbent forwarding platform's switching cost is the real story, because the platform holds jobs, documents, tariffs, customer records, integration mappings and accounting history, and migrating it is a multi-year, high-risk programme. **This guide does not repeat that.** It goes further, in the four ways the dispatcher asked for: what the client *owns* versus what the *vendor holds*; the migration mechanism that makes the moat; why incumbency survives *dissatisfaction*; and how a buyer can **reduce** lock-in.

### 7.1 What the Client Owns and What the Vendor Holds

The core question in any lock-in assessment is **who holds the thing that makes you leave-able**. For a CargoWise client, the honest split — derived from what the vendor publishes (§5.5) and what the platform's design implies (§3.2), with the guide's reasoning labelled as reasoning — is:

| Asset | Who effectively holds it | Basis |
|---|---|---|
| **The client's own data (jobs, documents, ledger)** | **Vendor-hosted, client-owned in principle but vendor-held in practice** | The vendor hosts it in CargoWise Cloud or a private cloud (§5.4); extraction is a *paid optional service* (read-only SQL access, backup mounting, data return, tear-down) per the vendor's own pricing page (§5.5). Ownership is a contract question, not a technical guarantee, and the vendor does not publish the ownership term. |
| **The integration mappings (code maps, XML schemas, validation rules)** | **Often the Service Partner, sometimes the client** | The integration model is partner-mediated (§5.2); the vendor does not publish a client-owned-mapping guarantee. The mapping is the "how we talk to carriers/customs/customers" asset and frequently sits with the implementing partner. |
| **The trained workforce's platform competency** | **The client, but only on this platform** | Certification is a platform asset (the vendor cites tens of thousands of CargoWise Certified professionals, §2.3). It transfers across the client's *own* network, not to a competitor's system. |
| **The customer portal experience (CargoWise Neo)** | **The client's brand, the vendor's technology** | Neo is configurable to the forwarder's brand (§4.3) but is wired into the same database; the forwarder's customers' familiarity is a switching cost the client *created* and cannot port. |
| **The reference masters (carriers, ports, tariffs, classifications)** | **Vendor-maintained as platform content** | The vendor sells customs content (BorderWise) and rate products (Cargoguide, CargoSphere) as group businesses (§2.4); the content is the vendor's, licensed into the platform. |
| **The upgrade path** | **Entirely the vendor's** | One release train; the vendor's non-current-version surcharge structure (§6.4) discourages divergence. |
| **The contract itself (exit/escrow/egress rights)** | **Whatever the client negotiated** | The vendor publishes **no** source-code escrow or guaranteed-egress commitment that this pass could find (§5.1, §15). This is the one asset the buyer can actually improve — §7.4. |

The guide's reading: the asset that **decides** whether a client can leave is not the data (extractable, at a price) but the **integration mappings and the trained-staff-plus-customer-portal combination** — the parts that are expensive to rebuild and are not sitting in a single exportable file. That is why migration, not egress, is the real barrier.

### 7.2 The Migration Is the Moat

The mechanism, developed:

1. **The migration is not a data copy; it is a re-implementation.** Moving off a CargoWise-class platform means re-creating, in the new system: the job model and numbering, the document templates and their regulatory content, the accounting configuration (multi-entity, multi-currency, accrual rules, revenue recognition), the customs connections per jurisdiction, the carrier connections, the customer portal, the workflow templates and exception handling, and the *mappings* for every partner integration (§5.2). Each of these is a project in its own right.
2. **It touches every team and the general ledger.** The freight-forwarding guide §13 names this; the sharper point is that the **accounting history** is the hardest part: a platform that holds the ledger lineage cannot be left without a plan for the ledger at the cutover, and a cutover that breaks the accounting period is a governance event, not an IT event (the vendor's own pricing page even sells "Accounting Period Configuration" changes, confirming that accounting periods are a platform-managed, deliberate, chargeable state — ✅ vendor, structure).
3. **It has a hard risk profile.** A migration runs alongside live operations (you cannot stop forwarding while you migrate), it is usually delivered by the *old* vendor's partners or a new partner, and a failed cutover strands jobs and invoicing. **Migration risk, not migration cost, is what keeps clients in place.** A rational buyer discounts a *promised* saving against a *possible* operational failure.
4. **The data comes at a price and a delay.** Even the extraction is a chargeable engineering event on the vendor's own published page (§5.5): read-only SQL access, backup mounting, data returns and tear-downs are services, not a self-service button. So leaving is *possible* — the vendor's page implies the mechanics exist — but it is *framed as a project the vendor charges for*.

The guide states this as the buyer's mechanism, not as a criticism: the moat is not a wall; it is the **cost and risk of the crossing**.

### 7.3 Why Incumbency Survives Product Dissatisfaction

A buyer's instinct is to assume dissatisfaction drives churn. In this category it often does not, for reasons that are structural:

- **The buyer who is dissatisfied is usually the operator, not the budget-holder.** The people who feel the platform's friction are the ops team; the people who decide whether to migrate weigh the multi-year programme in §7.2 against a diffuse, distributed dissatisfaction. **The migration looks worse than the friction.**
- **The alternatives are not obviously cheaper to move to.** Every alternative platform in the category (see §8) has *its own* migration and *its own* integration mappings to build. Switching cost is not "leave CargoWise", it is "re-enter the migration, with a different vendor, from a data model you have now customised to the first one".
- **The commercial model can change without triggering exit.** The vendor moved from per-seat/module pricing to a per-transaction model with a pass-through disbursement (§6) — a material repricing *structure* — and the client's realistic response is to **accept it or renegotiate**, because the data and integrations stay put. The vendor's own material even *pre-frames* the acceptance: it argues the pass-through "may positively impact your margins" and that the fee is a "miniscule fraction of the landed cost" (✅ vendor claim). **A repricing that the vendor can present as margin-neutral-or-positive to the client is exactly the repricing a client cannot easily refuse without migrating.**
- **Concentration cuts both ways.** As the freight-forwarding guide §13 argues, industry concentration on one platform is a *governance* risk for customers and their banks — and it is also a *stability* argument for staying: the vendor is too central to fail-quietly, and the ecosystem (partners, certifications, integrations) is built around it.

The guide's conclusion (its reading): **incumbency in this category is durable not because clients are satisfied but because the exit is a project.** A bank should therefore treat "our client runs CargoWise" as a signal about **data-integrity and integration maturity** — and, simultaneously, about **vendor dependency** — rather than as a signal about client satisfaction or switching readiness.

### 7.4 How a Buyer Reduces Lock-In

The constructive half of the lock-in question — the guide's practical contribution — is that most lock-in is **contractual before it is technical**, and a buyer can move the needle *before signing*:

1. **Negotiate data-ownership and egress in writing.** Name a **defined, machine-readable export** of the client's job, document and accounting data; a **maximum extract time**; a **cost cap** on extraction; and a **data-deletion certificate** on exit. The vendor's own pricing page shows extraction exists as a *paid service* (§5.5) — so the negotiation is about **who pays and how fast**, not whether it is possible.
2. **Own the integration mappings.** Contract that the client (not only the Service Partner) holds the **code maps, XML schemas and validation rules**, in a documented, exportable form. This is the asset §7.1 identifies as the real barrier; owning it is the single highest-leverage lock-in reduction.
3. **Consider source-code escrow or a continuity commitment.** The vendor publishes **no** escrow arrangement this pass found (§5.1, §15). Where an obligation is systemically important, an escrow or continuity-of-supply term is the standard mitigation; whether the vendor grants one is a procurement question this guide cannot answer.
4. **Cap the repricing exposure.** If the vendor can move from seats to per-transaction (§6), negotiate a **repricing cap or notice period** and a right to exit on a material change of commercial model — turning a "take-it-or-leave-it" repricing into a **contracted trigger**.
5. **Staged migration and dual-running.** Where the platform is strategically critical, plan the exit as a **staged, dual-run programme** (new system for new business, old system for run-off) rather than a big-bang cutover — which also de-risks the *failed-cutover* scenario in §7.2.
6. **Institute a periodic portability drill.** Test the extraction path (request a data return, prove the format, retain the mappings) on a schedule, so that "we could leave if we had to" is a **tested capability**, not an assumption.

Cross-reference [../technology/data_governance_guide.md](../technology/data_governance_guide.md) for the enterprise discipline of data ownership and egress, and [vendor_management_guide.md](vendor_management_guide.md) for the general vendor-concentration and exit-management practice this section instantiates for a single platform. The freight-forwarding guide §13 remains the generic statement this section builds on.

---
## 8. The Competitive Frame

**Rule for this section:** the frame is **structural and unranked**. The guide does not say any option is "best", does not rank vendors, does not recommend one, and prints **no market-share figure**. Competitors are named only to make the *categories* concrete, and every named competitor is flagged as to how it was sourced.

### 8.1 The Categories of Alternative

A forwarder choosing where its operations live is choosing among **categories**, not among a single ranked list. The categories, with what each *is*:

| Category | What it is | Examples (named structurally, unranked) and status |
|---|---|---|
| **Other dedicated forwarding platforms** | Products built specifically for the forwarder's job-and-document + customs + accounting workflow — the direct functional peers of CargoWise. | The freight-forwarding guide §13.1 already names **Descartes**, **Magaya**, **Logi-Sys**, **GoFreight** and "similar" in this band (⚠-knowledge there; **cross-referenced, not re-derived**). The guide adds no ranking and asserts no relative position. |
| **Logistics-adjacent suites** | Systems that perform part of the forwarder's stack (customs content, rate management, landside/drayage, parcel TMS, eBL/documentation) and can be assembled into an alternative architecture rather than bought as one platform. | WiseTech itself owns many of these as group businesses (§2.4) — e.g. **BorderWise** (classification), **Cargoguide**/**CargoSphere** (rates), **Blume Global**/**Trinium**/**Envase** (landside), **Pierbridge**/**SmartFreight** (parcel/parcel-TMS), **Bolero** (eBL) — which is the *point*: the adjacent suites are frequently the incumbent's own accreted pieces. Non-incumbent examples in the same adjacent-suite categories include **Descartes** (customs/global trade content, ⚠) among others. |
| **Enterprise / ERP entrants** | Large enterprise resource planning and supply-chain suites that a large forwarder or 3PL may run as the corporate ERP and that can subsume parts of the logistics workflow. | Named as the *category*, not as a recommendation. **e2open** — now part of WiseTech (§2.4) — is the incumbent's own multi-enterprise supply-chain/trade-software network (✅, cross-referenced from the freight-forwarding guide §8.1). Enterprise suites (SAP, Oracle, Microsoft, Infor and similar) are ⚠-knowledge as *categories* of enterprise software a 3PL may run; the guide asserts nothing about their logistics depth. |
| **Transport-management and visibility layers** | TMS products (road/rail/drayage planning and execution) and supply-chain visibility platforms (movement tracking). | The freight-forwarding guide §8.2 and §8.4 already names these categories, including **Transporeon** (⚠ reported as part of Trimble) **Descartes**, and the visibility band **project44**, **FourKites**, **Shippeo** (⚠ market-research-sourced there; **cross-referenced, not re-derived**). These are *adjacent* layers, not full forwarding platforms. |
| **In-house / legacy systems** | A forwarder's own bespoke software or an ageing domestic system, often maintained beyond its useful life. | The vendor frames this as its target: "Replacing ageing, legacy, proprietary and **domestic systems** with efficient, highly automated and integrated global capabilities" (wisetechglobal.com/who-we-are/about-us/, 2026-10-12 ✅ — vendor claim). The category is real; its modernisation cost is the forwarder's to bear. |
| **The spreadsheet / the partial stack** | Operations run on spreadsheets, email and disconnected point tools; common in smaller forwarders and as the "shadow" layer beside any platform. | Stated as the category the industry's integration problem (§5.2, and freight-forwarding guide §8.5) exists to solve. No vendor named. |

### 8.2 Structural and Unranked

Two structural observations, offered as the guide's reading and **not** as a ranking:

1. **The categories overlap inside a single vendor.** The incumbent does not sit in one category; through acquisitions (§2.4) it occupies **forwarding platform, customs content, rates, landside, parcel TMS, eBL/documentation and supply-chain network** simultaneously. That is a *breadth* structure, and it means a buyer comparing "CargoWise vs an alternative" is often comparing "one integrated vendor spanning many categories vs a best-of-breed assembly across the same categories". The trade-off is **integration simplicity against modularity**, not "good vs bad" — and the guide takes no side.
2. **The competitive pressure in this category is at the edges, not the core.** The freight-forwarding guide §13 concludes that new entrants compete at the edges (eBL, visibility, audit, customs content), not by replacing the core platform. This guide concurs and adds a *vendor-structural* reason: the incumbent has, by acquisition, **become** many of those edge players (§2.4), which raises the cost of competing at the edge and concentrates the alternative space in the "assemble a best-of-breed stack" model — which itself re-imports the integration problem the platform exists to solve.

The guide prints **no market share**, asserts **no "leader"**, and reaches **no recommendation**. A buyer selecting in this category should run the question set in §9 against *every* candidate, including the incumbent.

---

## 9. The Buyer's View

### 9.1 How the Category Is Actually Evaluated

A forwarder evaluating a platform in this category is not really choosing features; it is choosing **where its operational truth will live for the next decade**. The evaluation therefore has a shape the buyer should make explicit:

- **The decisive criteria are durability and portability, not function count.** Function parity across the mid-to-large platforms is largely a given; the differentiators a buyer should weight are the ones the freight-forwarding guide §13 and this guide's §7 identify — **integration depth, accounting continuity, data portability, and vendor stability**.
- **The migration in and the migration out are the same project, run in opposite directions.** A buyer should price the *exit* at the same time as the *entry* (§7.4). A vendor that makes entry easy and exit unspecified has sold a one-way door.
- **The hidden cost centres are the mappings and the people.** Whichever way the buyer goes, the recurring expense is the **integration mappings** (per carrier, per customs authority, per customer) and the **retraining of the whole operator population** (§7.1). The guide's advice is to budget those explicitly rather than treat them as implementation incidentals.
- **Partner dependency is a criterion in its own right.** In this category, implementation and integration are frequently delivered by a **Service Partner** (§5.2). A buyer is therefore choosing an *ecosystem*, and should assess the local partner market (competence tiers, certified-staff depth, the availability of an "Integration" or "Global Rollouts" partner) as part of the platform decision — because a strong platform with no local integration partner is a weak option in practice.

### 9.2 The Question Set

The practical contribution of this guide is a question set a buyer (a forwarder, a 3PL, or a bank evaluating a client's operational stack) can put to **any** vendor in this category — including the incumbent. Each question maps to the section that explains why it matters.

**A. Commercial model**
1. What is the **charge basis** — per seat, per module, per transaction, per branch, per entity? If it is per transaction, what counts as a "transaction", and by what mode/direction is it banded? (§6.2)
2. Is there an **implementation fee**, and who is permitted to charge it (vendor or partner)? (§5.2)
3. If costs can be **passed through as a disbursement**, is that a *default* or the buyer's *choice*, and does it change the buyer's P&L presentation? (§6.3)
4. What is the **repricing mechanism** — notice period, cap, and any right to exit on a material change of commercial model? (§7.4)
5. Are list prices published, negotiated, or both, and what is **excluded** from the base (optional/premium services, hosting, egress, backups)? (§6.4)

**B. Data ownership and egress**
6. Who **owns** the data contractually, in what document, and does the vendor guarantee a **defined, machine-readable export**? (§5.5, §7.4)
7. What does extraction **cost and take**, and is there a **maximum extract time** and a **deletion certificate** on exit? (The vendor's published page frames extraction as a *paid service* — §5.5.)
8. Is there **direct database read access** for the buyer's own analytics, and at what cost? (§5.5)
9. Where is the data **physically resident**, and can residency be constrained for a given jurisdiction? (§5.4 — the vendor publishes a three-data-centre footprint, not residency controls.)

**C. Integration surface**
10. What is the **integration mechanism** — a public REST API, a partner-mediated XML/mapping interface (the incumbent's case is eAdaptor/Universal XML, §5.2), EDI, or file transfer?
11. **Who owns the mappings** (code maps, schemas, validation) — the buyer, or the implementation partner? (§7.4)
12. Which **nameable integrations** exist today, with a dated announcement, for the buyer's actual carriers, customs authorities and customers? (§5.3, §13)

**D. Upgrade and release cadence**
13. How often does the platform **release**, and is the buyer on a **single release train** or free to freeze a version — and at what cost if it freezes? (The incumbent's non-current-version surcharge structure, §6.4.)
14. How are **breaking changes** communicated, and is there a test/sandbox environment? (A "Testing and Staging Platform Creation" item exists on the vendor's page — ✅ vendor, structure.)

**E. Support model**
15. What support tiers exist, what are the **response/restore** commitments, and is support delivered by the **vendor or a partner**? (Vendor's "My Account" self-service portal exists — ✅ vendor; SLA terms not published this pass, §15.)

**F. Implementation-partner dependency**
16. Who is **accredited** to implement in the buyer's region, and what **competence tiers** exist for the buyer's needs (forwarding, customs, integration, global rollouts)? (§5.2)
17. What is the **partner's** responsibility at go-live versus ongoing — and what happens if the partner exits the ecosystem? (§9.1)

**G. Multi-entity accounting**
18. Does the platform provide **multi-entity, multi-currency, accrual-based** accounting in-platform, and how does it handle **intercompany and revenue recognition**? (§3.2, §3.3 — vendor claims "multi-entity accounting" and accrual accounting.)
19. What is the **accounting-period and chart-of-accounts configurability**, and is changing it a *chargeable event*? (It is on the vendor's page — §6.4.)

**H. Customs content per jurisdiction**
20. Which **jurisdictions** have native customs connections, and which rely on the buyer's own arrangements? (The vendor names a set of coverage countries and a "90% of manufactured trade flows" *goal* — §3.3.)
21. Who maintains the **classification/tariff content**, and how current is it? (The vendor sells BorderWise and boasts classification data, §2.4, §3.3.)

**I. Exit**
22. What are the **exit rights** — notice, run-off support, escrow, and a negotiated **cutover plan**? (§7.4)
23. Can the buyer run a **portability drill** (a live extraction test) during the contract term? (§7.4)
24. What is the vendor's **continuity** position — is the platform's regulatory or ownership situation material to the buyer's risk register? (The freight-forwarding guide §8.1 carries the ACCC matter and §13 the concentration argument — **cross-referenced, not re-derived**.)

The guide's summary advice (its reading): a buyer who answers A–I honestly will choose on **durability, portability and integration ownership**, not on the feature demo — and will be negotiating the **exit** at the same table as the **entry**.

---
## 10. The Trade-Finance and Banking Adjacency

This section is the guide's bridge to the repository's finance cluster. The rule it obeys: **assert no capability the sources do not establish**, and cross-reference the banking guides rather than re-deriving them.

### 10.1 The eBL Link

The electronic bill of lading is where a logistics platform meets the **documentary and legal layer** that trade finance depends on. The **freight-forwarding guide §8.1 already records** that **Bolero (Galileo) eBL integrates with CargoWise** (⚠ vendor claim, dated there) — **this guide does not restate that finding.** What this guide adds, structurally and from its own sourcing:

- **Bolero is not merely an integration partner; it is a WiseTech group business.** The vendor's "our businesses" page lists **Bolero International** under the **Digital Documents** product line (wisetechglobal.com/what-we-do/our-businesses/, retrieved 2026-10-12 ✅ — vendor), and WiseTech's own announcement records its acquisition of **Bolero.net Limited** in **2022** (§2.4 ✅). So the eBL capability in the CargoWise orbit is **owned**, not just connected — a structural fact the freight-forwarding guide's vendor-integration framing does not carry.
- **Why that matters to a bank:** an eBL is a **negotiable/transferable transport document** whose value to a financier is precisely that it moves title-and-control questions into a verifiable electronic form. When the eBL capability sits *inside the same corporate group as the forwarding platform*, a bank assessing "the forwarder's document pipeline" is assessing a pipeline that runs through **one vendor from job record to electronic bill** — which is an **integration convenience and a concentration consideration** at the same time. The guide states both, and takes neither further.

The guide asserts **no** capability of the CargoWise/Bolero eBL stack beyond what §2.4 and the freight-forwarding guide establish — in particular, it does **not** claim that a CargoWise-generated document is a legally valid eBL in any jurisdiction, that any bank accepts it, or that any platform can substitute for the carrier's or the parties' own instruments. Those are legal and jurisdictional questions the sources do not settle.

### 10.2 What a Financier Could and Could Not Verify from Platform Data

This is the load-bearing banking point, and the guide states it plainly.

The **freight-forwarding guide §7.8 already establishes the "where the data lives" table** — the forwarder's platform is *rich but derivative*; it holds copies and joins, not the authoritative record of customs status, carriage or settlement — and **§8.4** establishes the **movement-versus-legal-status** distinction (a visibility layer tracks *movement*; it does not certify *legal status*). **This guide cross-references both and does not re-derive them.** What it adds is the CargoWise-specific application of that discipline:

| What a bank might want to rely on | Can a forwarder's CargoWise data evidence it? | Why |
|---|---|---|
| That a **job/shipment exists and is progressing** | ⚠ **Corroborating, not authoritative** | The job record is the *forwarder's own* creation; it evidences the forwarder's operations, not the underlying carriage. |
| That a **container moved** (movement fact) | ⚠ **Corroborating** | Movement/visibility data is the platform's join of carrier/terminal data; it is a *movement* fact, not a *legal-status* fact (freight-forwarding guide §8.4). |
| That a **customs declaration was lodged or released** | ❌ **Not from the platform** | The authoritative record is the **customs authority's system**; the platform holds a copy and a filing reference (freight-forwarding guide §7.8). |
| That a **bill of lading was validly issued or surrendered** | ❌ **Not from the platform alone** | The carrier (master) or forwarder (house) issues it; validity and surrender are documentary/legal facts, not platform facts. An eBL (Bolero/CargoWise, §10.1) is the mechanism that *aims* to make this verifiable, but the guide asserts no more than the sources do. |
| That the **freight invoice is payable/paid** | ⚠ **Corroborating** | The invoice and its audit status live in the forwarder's accounting module; settlement lives in the **banks** (freight-forwarding guide §7.8). |
| That a **company's books show** the receivable/payable | ⚠ **Suggestive, unverified** | The accounting module holds the entries; whether they reflect the true financial position is an audit question, not a platform query. |

The conclusion a bank should draw — the guide's framing, consistent with the freight-forwarding guide §7.8: **a forwarder's CargoWise data is a guide, not a register.** It is excellent for *operational* corroboration (is the client actually moving goods on the lanes it claims?) and **inadmissible-by-construction** for *legal-status* assertions (was the declaration released, was the bill surrendered, was the invoice paid). The **documentary-evidence caveat** is therefore: *treat platform data as a screening and monitoring signal; require the authoritative instrument (customs authority record, carrier document, bank settlement) for any legal or credit decision.* Cross-reference [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) for how a bank's own trade systems hold the authoritative trade record, and [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md) for the instrument-level (LC/documentary) discipline.

### 10.3 The Supply-Chain-Finance Connection

The connection between a logistics platform and supply-chain finance is **structural, not a product capability**, and the guide asserts nothing the banking guides do not establish:

- A forwarder's platform data is a **potential input** to a financier's view of a supply chain — who ships what, on which lanes, how fast, how consistently — which is the kind of operational signal supply-chain-finance programmes use to size and price receivables finance. **Cross-reference [../banking/supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md) for what supply-chain finance actually is** (payables/receivables programmes, the anchor-buyer model, the credit-risk mechanics) — **this guide does not re-derive it.**
- The **technology** layer — how a platform's data is ingested, normalised and used by a financier's systems — is [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md)'s subject, cross-referenced, not re-derived.
- The guide asserts **no** capability of CargoWise-to-supply-chain-finance integration that the sources establish. Specifically: it does **not** claim that CargoWise data is accepted as eligibility evidence, that any financier consumes it automatically, or that the platform offers a financing product. **The sources do not establish any of those, and the guide writes that plainly rather than inventing a capability.**

The honest one-line summary the guide offers a bank: **the platform is a place where operational truth about a client's trade is produced; it is not a place where the *financial* or *legal* truth about that trade is settled, and a financier that treats it as the latter has misread what a forwarding platform is.** Cross-reference [clpa_contract_logistics_guide.md](clpa_contract_logistics_guide.md) for the 3PL-side commercial context if the financed party is a contract-logistics operator rather than a forwarder.

---

## 11. The Failure Modes and Honest Limits

This section is deliberately substantial: where sources are thin, the honesty belongs here and in §15, not in padding.

### 11.1 Implementation Risk

- **Platform capability is not implementation outcome.** The vendor's own ecosystem is built around implementation risk: **Service Partners** exist (and are accredited) precisely because implementations are non-trivial (§5.2). A forwarder that buys CargoWise and does not resource the implementation — data migration, configuration, integrations, training, cutover — gets a fraction of the platform's value. The vendor's own customer testimonials (⚠ vendor claims) all emphasise the **implementation** as the point of achievement, not the licence.
- **The partner is a variable the platform cannot control.** Because integration is partner-mediated (§5.2), the *quality* of a given implementation depends on the local partner market, not on the vendor's product. A bank assessing a client's operational maturity should ask **who implemented it**, not only **which platform** it is.

### 11.2 Vendor Dependency and Concentration

- **The concentration is real and is a two-sided risk.** As the freight-forwarding guide §8.1 and §13 establish, much of mid-to-large forwarding runs on one vendor's platform; that makes the vendor's regulatory, pricing and reliability behaviour **systemic** to its customers. The **ACCC matter reported 19 August 2026** (freight-forwarding guide §8.1 ⚠ press-reported) is the live instance; the guide cross-references it and adds no claim about its outcome.
- **The acquisition strategy is itself a dependency risk.** Because the group has grown by absorbing many brands (§2.4), a client's *specific* product — a regional customs tool, a landside TMS, an eBL platform — may be one of the accreted pieces whose long-term roadmap the vendor decides. The vendor states an aim to "close product and strategic alignment" in acquisitions (✅ vendor); the *client-side* risk is that a product's standalone roadmap is absorbed into the flagship's. The guide states this as a structural risk, not an accusation, and **does not infer any specific product's fate.**

### 11.3 The Migration Reality

The migration reality is developed in full in §7.2. The honest summary: **leaving is possible and is framed by the vendor's own documents as a chargeable, deliberate engineering event**, not a self-service export. A client planning an exit must budget for the conversion of jobs, documents, the accounting cutover, the integration mappings and the retraining — and must accept the failed-cutover risk of running the migration alongside live operations. Cross-reference the freight-forwarding guide §13 for the generic statement.

### 11.4 Data-Egress Limits

- **The vendor's published posture frames extraction as a paid service** (read-only SQL access, backup mounting, data returns, tear-downs — §5.5, ✅ vendor). The **default** export's completeness, format and cost are **not published**, and the guide does not infer them (§15).
- **The mappings are the egress you cannot buy back.** Even a perfect data export does not export the **integration mappings** (§7.1): the code maps, schemas and validation rules often held by a Service Partner. A client can usually get its *data* out; getting its *integrations* out is a different project.

### 11.5 Customs-Content Jurisdictional Gaps

- **Customs is jurisdiction-by-jurisdiction, and coverage is claimed, not proven.** The vendor markets native connections to customs authorities worldwide and lists a set of coverage countries (Australia, Canada, China, South Africa, New Zealand, Singapore, Brazil, US, France, Germany, Ireland, Italy, UK, Spain, Puerto Rico — ✅ vendor page) plus a **goal** of "90% of the world's international manufactured trade flows" (§3.3). A **goal** is not coverage; a client operating on a lane not natively connected relies on its own arrangements, and the guide states no coverage map beyond what the vendor names.
- **Content currency is a maintenance dependency.** Tariff/classification content and customs rule content change constantly; the vendor sells them as products (BorderWise, §2.4) and reports updates (e.g. a "What's new in CargoWise Customs" feed, ✅ vendor). A client's compliance currency therefore depends on the **vendor's publishing cadence**, which the client does not control.

### 11.6 What the Public Sources Cannot Establish At All

Stated so the gap is explicit (expanded in §15):

- **The internal architecture** — database technology, schema, data model, tenancy model, and whether "one global database" is literally one physical database, one logical database, or a federated design presented as one. *Not published by the vendor; not independently auditable from outside.*
- **The real number behind "20,000+"** — the count is the vendor's; there is no independent verification, and the vendor's own year-by-year snapshots differ (§2.3).
- **The legacy model's actual prices** — the vendor describes the legacy *structure*, not its *prices* (§6.4).
- **The number and identity of real customer migrations, and their outcomes** — the vendor publishes *success* testimonials, not migration population statistics or failure rates.
- **Security certifications, DR/BCP specifications, and SLA commitments** — not reached from the vendor's pages this pass (§5.1).
- **CargoWise Next's vendor-side specifics** — the vendor names it as its 4th-generation platform, rolled out 2025 (§4.1 ✅), but dedicated vendor material was not retrievable this pass.
- **Revenue and market-share figures** — **deliberately not published by this guide** (the repository's convention for this series; recorded as a finding, not an omission).

The guide's overall honest limit: **it can describe what the vendor says, what the vendor publishes commercially, and what a buyer can infer structurally — and it cannot audit the vendor's engineering claims or its customers' outcomes.**

---
## 12. The Cymbal Bank Worked Example (Fictional)

**This entire section is illustrative and fictional.** Cymbal Bank is a fictional persona used only to work an example; **every figure in it is illustrative and labelled as such**; **Cymbal Bank is the only bank and the only institution named in the example**; the forwarder client is described only as "Cymbal Bank's mid-sized freight-forwarder client" with no real name. Nothing here is a recommendation to lend, not lend, buy or not buy, and nothing here is a fact about any real institution. The purpose is to show **how the guide's §7, §9 and §10 fit together in a bank's hands** — not to tour the product.

### 12.1 The Situation

Cymbal Bank's corporate banking desk has a mid-sized freight-forwarder client — a firm with operations in several countries, running its business on CargoWise — that has asked Cymbal Bank to **extend and increase a working-capital facility** to fund the disbursement gap on growing ocean-forwarding volumes, and to keep a **receivables-finance line** alongside it. Cymbal Bank's credit team has the usual financials. The credit officer has also been told, in the file, "the client runs the industry-standard platform, which de-risks operations." The example walks what a diligent Cymbal Bank analyst does with that sentence.

*(All amounts below are illustrative and dimensional only, not any real client's numbers: suppose the requested increase is a facility in the low tens of millions of US dollars, against an illustrative receivables pool and an illustrative disbursement cycle.)*

### 12.2 Step 1 — The Lock-In Assessment (§7): Is "Runs CargoWise" a Mitigant or a Dependency?

The analyst applies §7 rather than accepting the file's framing. Three questions:

1. **What does the client actually own?** (§7.1) The client's job history, ledger and documents live in the vendor's cloud; the **integration mappings** for its carriers and customs connections are held by an implementation partner; the **customer portal** experience is the client's brand on the vendor's technology; the **trained staff** are platform-specific. The client owns **none** of these outright in a portable form, and the **contract's exit/egress terms were not negotiated** (§7.1 last row).
2. **How expensive is the exit?** (§7.2) The analyst recognises that migration is a **re-implementation**, not a copy, and that the **accounting cutover** is the governance-sensitive part. The analyst concludes that the client is **operationally durable but structurally captive** — a *stable* obligor whose stability partly rests on a vendor the bank did not underwrite.
3. **What does that mean for credit?** (§7.3) The analyst reframes: "runs CargoWise" is evidence of **operational maturity and data discipline** (a **positive** for the bank: the client can actually produce the operational data the bank wants) *and* of **vendor-concentration dependency** (a **risk** the file should carry explicitly). The credit paper is amended from "runs the industry-standard platform" to two stated facts: **operational data is machine-producible**, and **the client's ability to leave its platform is not a tested capability.**

**Illustrative cost of the reframe:** the analyst adds a **vendor-concentration line to the risk register** and a **condition precedent** that the client retain (and, on renewal, negotiate) its **data-egress and mapping-ownership terms** (§7.4). No money moved; the paper got more honest.

### 12.3 Step 2 — The Question Set the Bank as Obligee Should Ask (§9)

Cymbal Bank is not buy-side, but §9's question set works as a **diligence instrument for a creditor**. The analyst adapts it:

| §9 question area | What Cymbal Bank asks the client | What a weak answer tells the bank |
|---|---|---|
| **Commercial model (§6)** | What is the platform's charge basis, and has it recently changed? (The vendor moved from seat/module pricing to per-transaction — §6.1–6.2.) If the platform cost is passed to the client's *customers* as a disbursement (§6.3), how is it booked? | A client that cannot say how its platform costs reach its P&L has weak cost governance — and a pass-through-disbursement structure can *flatter* margin (§6.3), which matters to a receivables lender reading the P&L. |
| **Data ownership / egress (§5.5, §7.4)** | Who owns the data, is there a defined export, and can the client run an extraction test? | "We've never tested it" means portability is an *assumption*, not a capability. |
| **Integration surface (§5.2)** | Who holds the carrier/customs mappings — the client or a partner? | A single-partner mapping lock is a second concentration risk, stacked on the platform's. |
| **Upgrade / release cadence (§6.4)** | Is the client on the current release? (A non-current-version surcharge exists — §6.4.) | A client on an unsupported release is running unpatched compliance content — a *trade-compliance* risk, which for a bank financing cross-border flows is its own concern. |
| **Support model (§9.2E)** | What are the vendor/client support and restore commitments? | Unclear restore commitments mean downtime is the client's problem — a working-capital timing risk. |
| **Implementation-partner dependency (§5.2)** | Who implements and supports — vendor or partner, and what happens if the partner leaves? | A thin local partner market is an operational single point of failure. |
| **Multi-entity accounting (§3.2)** | Does the platform's multi-entity, multi-currency accounting feed the client's *reported* financials directly, or is there a manual bridge? | A manual bridge between the operational platform and the audited books is a **control gap** the analyst should size. |
| **Customs content per jurisdiction (§11.5)** | On which lanes does the client rely on the platform's native customs connections, and where does it file independently? | Unmanaged filing on key lanes is a delay/penalty exposure on the *financed* flows. |
| **Exit (§7.4)** | Is migration planned or assumed? | No plan = maximum lock-in = minimum negotiating power on the next repricing. |

The analyst's conclusion from §9: the questions **do not** produce a platform feature score — they produce a **risk map** of where the client's operational dependence sits.

### 12.4 Step 3 — What the Platform Data Does and Does Not Evidence (§10)

Cymbal Bank wants to monitor the financed flows. The analyst applies §10.2 strictly:

- **What Cymbal Bank will accept as monitoring signal:** the client's platform data showing **jobs in progress, lanes, volumes and movement facts** — as **corroboration** that the client is genuinely moving goods on the financed lanes (§10.2 first rows). This is a *screening and monitoring* input, and the bank treats it as such.
- **What Cymbal Bank will not accept as evidence:** that a **declaration was released** (that is the customs authority's record, §10.2), that a **bill of lading was validly issued or surrendered** (a documentary/legal fact, §10.2), or that an **invoice is payable/paid** (settlement is the banks', §10.2). For the receivables line, Cymbal Bank requires the **authoritative instruments**, not the platform's copy.
- **The documentary-evidence caveat, applied:** the analyst writes into the paper that the platform is *a guide, not a register* (§10.2) — excellent for operational corroboration, **inadmissible-by-construction** for legal-status assertions. If the client's eBL pipeline runs on the same vendor group's stack (§10.1), the analyst notes the **integration convenience and the concentration** in the same breath, and asserts no legal validity the sources do not establish (§10.1).

### 12.5 Step 4 — The Decision, and What It Illustrates

**Illustratively**, Cymbal Bank approves the facility **with three conditions** that carry the guide's logic:

1. **A vendor-concentration disclosure** in the client's annual covenant pack (the client names its platform and its implementation partner; the bank tracks changes) — §7.1, §11.2.
2. **A portability condition on renewal** — the client negotiates documented **data-egress rights and mapping ownership**, and runs a **portability drill** — §7.4.
3. **Evidence-based monitoring** — operational platform data is used as a **signal**; legal-status assertions require **authoritative instruments** — §10.2.

If the bank had accepted the file's original line — "runs the industry-standard platform, which de-risks operations" — it would have taken comfort from the **wrong object**: it would have treated a *feature-breadth-and-ubiquity* signal as if it were a *portability-and-dependency* assessment. The example's whole point is that the **feature list was never the mitigant**; what mattered was **where the client's data and integrations actually live, and what it would take to move them** — which is exactly the thesis.

**Returning to the thesis (§1.3): for Cymbal Bank, as for the forwarder, the value and the risk both sit in the shared database — not in the feature list. A bank that underwrites the feature list underwrites the wrong thing; a bank that underwrites the database, its dependencies and its portability underwrites the real position. And the shared database is the moat precisely because leaving it is a project, not a purchase. That is the whole argument, and it is why this guide's title is what it is.**

---

## 13. The Anti-Patterns

Each row is **symptom → cause → guardrail**. These are the recurring mistakes this guide's subject invites, drawn from the sections above.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| 1 | **Buying the feature list.** The evaluation scores modules and functions; the contract's exit and egress terms are unread. | Features are visible and comparable; lock-in is invisible until it bites (§1.3). | Evaluate on **durability and portability** (§9.1); read the exit clauses before the feature matrix (§7.4). |
| 2 | **Treating vendor logos as references.** "CEVA/GEODIS/Yusen run it, so it's proven." | Vendor customer marks are **vendor-published claims** (§3.3, freight-forwarding guide §8.1), not independent references. | Ask *which* functions a named account actually uses, on *which* lanes, and **call the client** — do not infer scope from a logo. |
| 3 | **Assuming data portability.** "We can always export our data." | The vendor's published page frames extraction as a **paid service** (read-only SQL, backup mounting, data return, tear-down — §5.5); the default export's completeness is unpublished. | Negotiate a **defined export + cost cap + max time + deletion certificate** (§7.4); run a **portability drill**. |
| 4 | **Single-vendor mapping lock.** The integrations work, so nobody notices the mappings live with one partner. | The integration model is **partner-mediated** (§5.2); mappings (code maps, schemas, validation) frequently sit with the Service Partner. | Contract that the **client owns** the mappings, in an exportable, documented form (§7.4). |
| 5 | **Mistaking a module for a system of record.** "We have the customs module, so we have customs." | A **module** is a component; the **authoritative** customs record is the authority's system (freight-forwarding guide §7.8, this guide §1.2, §10.2). | Distinguish **platform copy** from **authoritative register** in every control and every credit decision (§10.2). |
| 6 | **Treating platform visibility as documentary evidence.** A tracking screen is cited as proof of carriage or release. | **Movement ≠ legal status** (freight-forwarding guide §8.4; this guide §10.2). | Use platform data as a **signal**; require the **authoritative instrument** for any legal or credit assertion (§10.2). |
| 7 | **Assuming one global upgrade is one global change.** "The vendor upgraded; we're on the new version." | A **single release train** upgrades the platform as a whole (§3.2, §6.4); the *client's* configuration, mappings and customisations must be re-validated per release, and divergence is charged (non-current-version surcharge, §6.4). | Treat every vendor release as a **client-side change project** with regression testing of integrations and customs content. |
| 8 | **Treating the vendor's customer count as a risk assessment.** "20,000 customers / 24 of the top 25 — it must be safe." | The count is a **vendor claim** (§2.3); ubiquity is a **concentration** signal as much as a stability signal (§11.2). | Separate **"is the product widely used"** from **"is my client's dependence on it managed."** Track the dependency (§12.5). |
| 9 | **Assuming "single database" means one tenant or no divergence.** | The vendor's "single database" is a **claim** (§3.2, §3.5); the tenancy and hosting model (Cloud, Private Cloud, Customer Hosted, Self Hosted) is **not fully published** (§5.4). | Do not infer tenancy, residency or isolation from the marketing phrase; **ask**, in writing (§9.2B). |
| 10 | **Reading the acquisition history as a product roadmap.** "They bought X, so X will become part of CargoWise." | The vendor states acquisitions aim at "product and strategic alignment" (§2.4) but **publishes no per-product integration roadmap**; the guide infers none. | Ask the vendor for a **written roadmap** for the specific acquired product you depend on; treat an accretion history as **breadth**, not as a promise. |
| 11 | **Underestimating customs-content jurisdictional gaps.** "It has global customs." | Coverage is named per jurisdiction plus a **goal** ("90% of manufactured trade flows", §3.3, §11.5); content currency is a **vendor publishing cadence**. | Map the client's **actual lanes** to **named** jurisdictions; keep a fallback filing path where there is no native connection (§9.2H). |
| 12 | **Assuming a published price list is the total cost.** The vendor now publishes list prices, so the cost is "known". | The published list is the **transaction fee**; **optional/premium services, hosting, egress, backups, changes and surcharges** sit outside it (§6.4), and the legacy model's prices are unpublished (§15). | Price the **whole surface**: base fees **plus** the optional-services menu **plus** implementation **plus** the client's own migration/retraining. |

Cross-reference [vendor_management_guide.md](vendor_management_guide.md) for the general anti-patterns of vendor concentration and exit management that rows 3, 4, 8 and 10 instantiate for a single platform.

---
## 14. The Claims Audit

Every material claim in this guide, with its source, the retrieval or announcement **date**, a **status** flag, and the **kind** of source. Conventions: ✅ verified this pass against the named primary source; ⚠ vendor claim / approximate / single secondary source; ❌ could not be verified. **Kind** is one of: *vendor page* (a seller's own web page), *company announcement* (the company's own news/ASX release), *dated press* (third-party reporting with a date), *third-party summary* (aggregators, directories, encyclopaedia).

| Claim | Source and date | Status | Kind |
|---|---|---|---|
| CargoWise is WiseTech Global's flagship platform | cargowise.com; wisetechglobal.com/what-we-do/our-businesses — retrieved 2026-10-12 | ✅ | vendor page |
| WiseTech "established in 1994"; listed on ASX **11 April 2016**; ASX: WTC | wisetechglobal.com/investors — retrieved 2026-10-12 | ✅ | vendor page |
| Founded 1994 by **Richard White and Maree Isaacs** | wisetechglobal.com/who-we-are/our-history/ — retrieved 2026-10-12 | ✅ | vendor page |
| Original company name "Eagle Developments International"; renamed WiseTech 2011 | Wikipedia (article on founder) — retrieved 2026-10-12 | ⚠ | third-party summary |
| **Zubin Appoo** CEO (assumed July 2025); **Richard White** Co-Founder/Executive Director/Chief Innovation Officer, CEO until Oct 2024, Executive Chair Feb 2025–Jul 2026 | wisetechglobal.com/investors/our-leadership-team/ — retrieved 2026-10-12 | ✅ | vendor page |
| "20k+ customers"; "more than 20,000 organizations"; "24 of the top 25 global freight forwarders / 46 of the top 50 3PLs"; "193 countries licensed"; "39 product development centers"; "1,827 product enhancements in FY26" | wisetechglobal.com/who-we-are/about-us/ — retrieved 2026-10-12 | ⚠ | vendor page (vendor claims) |
| "All 25 of the top 25 global freight forwarders" (2020 snapshot); "43 Large Global Freight Forwarder rollouts" (2022); "29,500+ Certified Practitioners" (2023); "$180.8m / $261.9m R&D" (2022/2023); "more than 7,000 people" post-e2open (2025) | wisetechglobal.com/who-we-are/our-history/ — retrieved 2026-10-12 | ⚠ | vendor page (vendor claims, per-year) |
| "Over 40,000 CargoWise Certified" professionals | cargowise.com news, "Certified to scale…" — dated **22 July 2025** | ⚠ | vendor page (vendor claim) |
| CargoWise runs on "one global database"; "one platform, zero duplication"; "no cost and complexity of custom integrations"; real-time data | cargowise.com — retrieved 2026-10-12 | ⚠ | vendor page (vendor claims) |
| Platform delivered from "three data centers in Europe, Australasia and the Americas" | wisetechglobal.com/investors/our-leadership-team/ (Head of Information Systems bio) — retrieved 2026-10-12 | ⚠ | vendor page (vendor claim) |
| **e2open** acquired; announced 25 May 2025, completed 3 Aug 2025 | e2open press releases — cross-referenced from the freight-forwarding guide §8.1 | ✅ | company announcement |
| **Blume Global** acquired, announced **17 Feb 2023**; from Apollo/EQT funds | WiseTech Global announcement + ASX filing 17 Feb 2023 | ✅ | company announcement |
| **Envase Technologies** acquired, announced **24 Jan 2023** | WiseTech Global announcement + ASX filing 25 Jan 2023 | ✅ | company announcement |
| **Bolero.net Limited** acquired, announced **2022**; terms not disclosed | Bolero/WiseTech announcement; GTR report (terms) | ✅ announcement; ⚠ terms | company announcement; dated press |
| **Pierbridge** joined the group in **2018** | WiseTech announcement; corroborated by a 2026 partner release | ✅ | company announcement |
| **FRDM** acquisition | Forbes, **1 Oct 2026**, linked from the vendor's news feed | ⚠ | dated press |
| **CargoIT** acquired; year not confirmed on the vendor's page | wisetechglobal.com news (CargoIT) — page retrieved 2026-10-12 carries no date | ⚠ (year) | company announcement (undated page) |
| Group brands listed (ACO Informatica, Bysoft, CustomsMatters, DataFreight, EasyLog, Editrade, EXA-System, Forward, Intris, LSP, Opentecnologia, Prolink, Ready Korea, SISA, Softcargo, Systema, Taric, Ulukom, znet, BorderWise, CMS, Containerchain, Depot Systems, MatchBox Exchange, Trinium, BSM Global, ImpexDocs, Shipamax, SaaS Transportation, SmartFreight, Microlistics, Cargoguide, CargoSphere, Softship, Inobiz, Xware) | wisetechglobal.com/what-we-do/our-businesses/ — retrieved 2026-10-12 | ✅ ownership; ⚠ acquisition dates (mostly not published) | vendor page |
| Vendor year-level acquisition counts (1999 first; 2015 China/South Africa; 2017 Germany/Italy/Singapore/US; 2018 15 acquisitions) | wisetechglobal.com/who-we-are/our-history/ — retrieved 2026-10-12 | ✅ | vendor page |
| "41 acquisitions" (aggregate count) | Tracxn — retrieved 2026-10-12 | ⚠ | third-party summary |
| **CargoWise Next** is the vendor's "4th generation platform", rolled out **2025** | wisetechglobal.com/who-we-are/our-history/ — retrieved 2026-10-12 | ✅ | vendor page |
| Dedicated CargoWise Next page | cargowise.com/cargowise-next/ — retrieved 2026-10-12 | ❌ 404 (page not found) | vendor page (absent) |
| **CargoWise Neo** portal (customer-facing, track/trace, booking, e-conversation, branding) | cargowise.com/solutions/cargowise-neo/ — retrieved 2026-10-12 | ⚠ | vendor page |
| **Value Pack** model replaces the "prior STL model"; four packs; "no more User Seat Fees / standard Cloud hosting fees" for many customers | cargowise.com/cargowise-value-pack/ — retrieved 2026-10-12 | ✅ as described structure; ⚠ as to effect | vendor page |
| Value Pack capability counts "125+/198+/225+/116+" | cargowise.com/cargowise-value-pack/ — retrieved 2026-10-12 | ⚠ | vendor page (vendor claims) |
| Value Pack transaction fees "presented as a pass-through disbursement… 'CargoWise Automation Fee'"; "may positively impact your margins" | cargowise.com/cargowise-value-pack/ — retrieved 2026-10-12 | ⚠ | vendor page (vendor claims) |
| Vendor **publishes** per-unit list prices, an Optional Services menu, Extended Credit Terms Surcharge and Non-Current Version Surcharge; **version 26.10** | cargowise.com/…/cargowise-value-pack-community-pricing/ — retrieved 2026-10-12 | ✅ (page exists; **amounts deliberately not reproduced**) | vendor page |
| Payment terms will not exceed **60 days**; surcharges fixed and non-negotiable | cargowise.com/…/community-pricing/ — retrieved 2026-10-12 | ✅ (structure/term; no amounts) | vendor page |
| Integration mechanism is **eAdaptor / Universal XML** (not a simple REST API) | Chain.io (CargoWise Integration Platinum Partner) — retrieved 2026-10-12 | ⚠ | dated press/vendor-with-interest |
| **eAdaptor / xHub** developer material (connectivity, XML, code mapping, validation) | third-party upload of "eAdaptor Developer Guide" — dated **11 March 2024** | ⚠ | third-party summary |
| Partner network types/tiers/competences (Service/Business/Industry/Education; Platinum/Gold/Certified; Accounting/Customs/Forwarding/Global Rollouts/Integration/PAVE/Warehouse/Workflow) | cargowise.com/partners/find-a-partner/ — retrieved 2026-10-12 | ✅ | vendor page |
| **PortConnect NZ** port community system integration | cargowise.com news — announced **1 Oct 2026** | ✅ | company announcement |
| **World Freight Company / ATC AirlineConnect** integration | cargowise.com news — announced **30 Sep 2026** | ✅ | company announcement |
| **ShipDNA** Clippership → Transtream migration program | cargowise.com news — announced **29 Sep 2026** | ✅ | company announcement |
| Vendor's customs coverage named countries + "90% of the world's international manufactured trade flows" **goal** | cargowise.com/solutions/cargowise-customs/ — retrieved 2026-10-12 | ⚠ | vendor page (vendor claim/goal) |
| Corporate governance framework exists (Code of Conduct, charters, policies, Modern Slavery Statement) | wisetechglobal.com/investors/corporate-governance/ — retrieved 2026-10-12 | ✅ | vendor page |
| **ACCC search warrant** reported **19 August 2026** | cross-referenced from the freight-forwarding guide §8.1 | ⚠ | dated press |
| Bolero (Galileo) eBL integrates with CargoWise | cross-referenced from the freight-forwarding guide §8.1 | ⚠ | vendor page (via parent guide) |
| Vendor's stated intent to keep acquiring ("continue to invest in businesses where we see value") | wisetechglobal.com/who-we-are/about-us/ and /what-we-do/our-businesses/ — retrieved 2026-10-12 | ✅ | vendor page |
| **Revenue and market-share figures** | none used | ❌ not published by this guide | — (deliberate) |

---

## 15. What Could Not Be Verified

Substantial and honest, as the dispatcher required. Each entry says **what is missing and why**.

1. **The internal architecture and database technology.** *Missing:* whether "one global database" is one physical database, one logical database, or a federated design presented as one; the DBMS; the schema/data model; the tenancy model. *Why:* the vendor publishes only marketing-level architecture claims (cargowise.com), one hosting datum (three data centres, leadership page), and integration *product* names; no technical whitepaper, schema, or independent architectural audit was retrievable this pass. This is the single most important unverified claim in the guide — the thesis rests on the vendor's word for it.

2. **The customer count behind "17,000+" / "20,000+".** *Missing:* independent verification, and the definition of "customer"/"organization". *Why:* the figures are the vendor's own (cargowise.com; wisetechglobal.com/about-us), and the vendor's own year-by-year snapshots differ (all-25 in 2020 vs 24-of-25 in 2026), which itself shows these are self-reported.

3. **Pricing under the legacy "STL" model.** *Missing:* the actual prices charged under the pre-2026 model (seat fees, module fees, hosting fees). *Why:* the vendor describes the legacy *structure* (cargowise.com Value Pack FAQ) but does **not** publish the legacy *prices*; this pass found no source for them. The **new** model's prices are published but deliberately not reproduced here (§6.4).

4. **The number and identity of real customer migrations, and their outcomes.** *Missing:* how many customers actually migrate onto or off the platform annually, and with what success/failure rates. *Why:* the vendor publishes *success testimonials* (e.g. CLASQUIN, Logistics Plus, CEVA, LOKE/LEMAN, Craft, Metro Shipping — all ⚠ vendor claims), not migration population statistics or failure rates.

5. **CargoWise Next's vendor-side specifics.** *Missing:* the vendor's own detailed description of CargoWise Next (what changed, what it means architecturally, its roadmap). *Why:* the vendor **names** it (history page: "4th generation platform, CargoWise Next", 2025 ✅) but the dedicated page `cargowise.com/cargowise-next/` returned a **404** this pass, and third-party descriptions were not adopted as the vendor's own. Whether the page was moved, unpublished, or never existed at that path is unknown — recorded as a **tool/page limitation, not as absence of the product.**

6. **The vendor's open developer/API portal.** *Missing:* a publicly-retrievable CargoWise API reference, developer portal, or documented endpoint catalogue. *Why:* `developer.cargowise.com` was **refused by the extraction tool** ("URL targets a private or internal network address") this pass, and the vendor's own site did not surface an open API reference. The integration mechanism is therefore reported only via a partner (eAdaptor/Universal XML, ⚠) and a third-party developer-guide upload (§5.2). **`web_search` reliability was partial this pass — recorded as a tool limitation, not as evidence of absence.**

7. **Security certifications, DR/BCP specifications, and SLA commitments.** *Missing:* whether the vendor holds named certifications (e.g. SOC 2, ISO 27001), its stated recovery objectives, and its support SLAs. *Why:* the vendor's corporate-governance page publishes charters and policies (✅), and the leadership page names the three data centres, but no certification or SLA page was reached this pass.

8. **Data-residency controls and the default export's completeness/format.** *Missing:* per-jurisdiction residency guarantees and the exact form/completeness of a default data export. *Why:* the vendor's community pricing page prices *optional* extraction services (read-only SQL, backup mounting, data return, tear-down — ✅), which implies extraction is a deliberate, chargeable event, but the default export's terms are not published.

9. **Source-code escrow.** *Missing:* whether any escrow or continuity-of-supply arrangement exists. *Why:* no source found this pass. Recorded because it is the standard mitigation for the dependency §11.2 describes.

10. **The exact year of several acquisitions.** *Missing:* dated announcements for **CargoIT**, **Bysoft**, **BorderWise**, and other group brands listed on the vendor's businesses page. *Why:* the businesses page confirms **ownership/grouping** but carries **no acquisition dates**; the CargoIT announcement page exists but is **undated** and cites 2017 data. The guide therefore lists these as **group companies with year ⚠**, and **excluded** further businesses (e.g. "Customscentre") that only a third-party database (Tracxn, ⚠) attributes to WiseTech. **No acquisition was included from memory.**

11. **Third-party acquisition/accounting prices.** *Missing:* the transaction amounts for Blume Global, Envase and other acquisitions. *Why:* the announcing parties stated amounts in some cases, but this guide's no-figure rule means no amount is reproduced (H2 policy, §6.4).

12. **The founder/leadership transition's full corporate record, and any reported governance outcomes.** *Missing:* a vendor-published account beyond the leadership role dates. *Why:* the vendor publishes the role dates (✅) and the **ACCC matter is recorded in the freight-forwarding guide §8.1** (⚠ press-reported); the guide deliberately excludes personal/sensational press material (§2.5) and does not re-verify the ACCC matter against the ACCC's own register, consistent with the parent guide.

13. **Revenue and market share.** *Missing by design.* *Why:* this guide-series convention treats a market-share number as requiring a third-party estimate the guide will not adopt, and revenue is a company-reporting figure outside this guide's scope. **Recorded as a deliberate absence, not an oversight.**

14. **Competitor market positions.** *Missing:* any ranking or share of the alternatives in §8. *Why:* the frame is deliberately **structural and unranked**, and several competitor facts (e.g. Transporeon/Trimble; the reported project44–FourKites consolidation) are **⚠ and already flagged in the freight-forwarding guide §16.3**, cross-referenced rather than re-verified here.

15. **Any independent, non-vendor assessment of the platform's operational claims** (productivity, compliance outcomes, "significant labor reduction" from AI). *Missing:* third-party measurement. *Why:* the claims are the vendor's (⚠), including the **forward-looking** AI statements ("Agentic AI Workflow Engine", "AI Management Engines", available from 1 December 2025 for some tools, rolling out thereafter — ✅ as a vendor statement, **unverified as delivered**).

---

## 16. The Glossary, Cross-References and Closing Summary

### 16.1 Glossary

- **ACCC** — Australian Competition and Consumer Commission, the Australian competition regulator; its reported search warrant at WiseTech (19 Aug 2026) is recorded in the freight-forwarding guide §8.1.
- **ASX** — Australian Securities Exchange; **ASX: WTC** is WiseTech Global's ticker.
- **BorderWise** — the vendor's goods-classification / border-compliance reference product (group business; year not confirmed, §2.4).
- **CargoWise** — the platform; the subject of this guide. **WiseTech Global** — the listed company that develops and sells it and has acquired many other businesses (§1.1).
- **CargoWise Next** — the vendor's name for its "4th generation platform", rolled out 2025 (§4.1).
- **CargoWise Neo** — the vendor's customer/shipper-facing portal layer (§4.3).
- **Consolidator** — an operator combining multiple shippers' cargo into one container (LCL consolidation).
- **Customs broker system** — software preparing/filing customs declarations (§1.2).
- **Customs declaration** — the filing that clears goods; its authoritative record is the **customs authority's system**, not the platform (freight-forwarding guide §7.8).
- **Disbursement** — a cost paid by the forwarder on the customer's behalf and recovered on the customer's invoice, typically posting to a balance-sheet account rather than P&L (§6.3).
- **eAdaptor / xHub / Universal XML** — the integration mechanisms named in the CargoWise ecosystem for machine-to-machine interfaces (§5.2).
- **eBL** — electronic Bill of Lading (§10.1); **Bolero** is the vendor's eBL group business (§2.4).
- **e2open** — a supply-chain/trade software multi-enterprise network, acquired by WiseTech (completed 3 Aug 2025; see freight-forwarding guide §8.1).
- **Forwarding system** — the forwarder's core job-and-document + accounting platform (§1.2).
- **House bill (H-B/L)** — the transport document the forwarder issues to its customer; distinct from the **master bill (M-B/L)** the carrier issues to the forwarder (§1.2).
- **Integration** — any machine-to-machine connection between the platform and another system (§1.2, §5).
- **Logistics ERP** — the accounting-and-enterprise half of a forwarding platform (§1.2).
- **Module** — a functional component of the platform; a component, not a system of record (§1.2).
- **Multi-tenant** — a hosted model where customers share a service instance with logical isolation; related to, but not identical with, "single shared database" (§1.2, §5.4).
- **NVOCC** — Non-Vessel-Operating Common Carrier; issues its own bills of lading without operating the vessel (§1.2).
- **Single database** — the vendor's claim that CargoWise runs customers on one global database; the thesis of this guide (§3.2, §3.5).
- **STL model** — the vendor's prior commercial model (seat + module + feature + hosting + transaction fees), replaced by the Value Packs (§6.1).
- **Tenant** — one customer's logically-isolated slice of a shared service (§1.2).
- **TMS** — Transport Management System; the road/rail/drayage planning-and-execution layer (§1.2).
- **Value Pack / CargoWise Automation Fee** — the vendor's 2025–2026 commercial model and its per-transaction fee, presented by default as a pass-through disbursement (§6.2–6.3).
- **WiseTech Global** — the listed company (ASX: WTC) (§1.1, §2).

### 16.2 Cross-References

**Same folder (`management/`):**
- [freight_forwarding_guide.md](freight_forwarding_guide.md) — **§8.1** carries the CargoWise vendor claims and the ACCC report; **§13** carries the generic switching-cost argument; **§7.8** carries the "where the data lives" table; **§8.4** carries the movement-vs-legal-status distinction. **All four are cross-referenced here, not re-derived.**
- [logistics_warehouse_management_guide.md](logistics_warehouse_management_guide.md) — warehouse operations, WMS/WES/WCS (§3, §10).
- [clpa_contract_logistics_guide.md](clpa_contract_logistics_guide.md) — the 3PL outsourcing engagement (§8, §10).
- [dhl_express_guide.md](dhl_express_guide.md) — the integrator/company-profile genre (§8, §16).
- [tmx_transform_guide.md](tmx_transform_guide.md) — the supply-chain advisory profile and the claims-audit/honesty conventions this guide follows.
- [vendor_management_guide.md](vendor_management_guide.md) — vendor concentration and exit-management practice (§7.4, §13).

**Banking (`../banking/`):**
- [../banking/trade_finance_guide.md](../banking/trade_finance_guide.md) — the instrument-level (LC/documentary) discipline (§10).
- [../banking/trade_finance_systems_guide.md](../banking/trade_finance_systems_guide.md) — the bank's own trade record (§10).
- [../banking/supply_chain_finance_guide.md](../banking/supply_chain_finance_guide.md) — what supply-chain finance is (§10.3).
- [../banking/supply_chain_finance_technologies_guide.md](../banking/supply_chain_finance_technologies_guide.md) — the technology layer (§10.3).

**Technology (`../technology/`):**
- [../technology/api_governance_guide.md](../technology/api_governance_guide.md) — governing the integration estate (§5.5).
- [../technology/data_governance_guide.md](../technology/data_governance_guide.md) — data ownership, lineage and egress (§5.5, §7.4).
- [../technology/cloud_providers_guide.md](../technology/cloud_providers_guide.md) — hosted/tenancy models (§5.4).

### 16.3 Closing Summary

CargoWise is the dominant mid-to-large logistics platform, developed and sold by **WiseTech Global (ASX: WTC)**, and the two must be kept apart: the platform is one product; the company is an acquirer of dozens (§1.1, §2.4). The platform's own architecture is sold as **one global database**, and the guide's whole argument is that this — not the feature catalogue — is what matters: features are copyable and *are* copied, whereas a single shared database of jobs, ledger, reference master and upgrade path is a **structural** property that no competitor re-creates by shipping a module (§1.3, §3.2).

The commercial model has moved from a seat-and-module structure to a **published, per-transaction Value Pack model** whose Forwarding and Customs fees default to a **pass-through disbursement** on the customer's invoice (§6) — a repricing the vendor can present as margin-protective, and therefore one a client cannot easily refuse without leaving. **Leaving is the real barrier, not the feature list** (§7): the client's data is hosted, its **integration mappings** are often held by a service partner, its customers sit on its branded portal, and its accounting history and trained staff are attachments to *this* database. Migration is a re-implementation, a chargeable engineering event rather than a self-service export, and it is **migration risk — not migration cost — that keeps clients in place** (§7.2–7.3).

For a bank (and for the buyer), the practical consequence is the guide's contribution (§9–§12): assess **durability, portability and integration ownership**, not the feature demo; treat a vendor's ubiquity as a **concentration** as well as a stability signal; use platform data as an operational **signal** and never as **documentary evidence** of legal status; and negotiate the **exit** at the same table as the **entry**. The one architectural claim everything rests on — that the database is genuinely single and global — is the vendor's word and is not independently verifiable from outside (§3.5, §11.6, §15): **the guide reports it as such, and reasons about its consequences rather than pretending to have audited it.**

**the feature list is not the moat; the shared database is.**
