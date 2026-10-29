# Open-Source Payment Systems: The Software, the Licence, and What Neither Gives You

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank, Singapore
> **Context:** Financial Services Technology -- the open-source payments landscape across rail, core, orchestration, ledger and API categories, plus the licence and governance analysis a bank has never done
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Financial Services Software-Systems Guides -- the **cross-category landscape and licence/governance** companion to the three open-source deep-dives already in this series: [Apache Fineract Guide](apache_fineract_guide.md) (core banking ledger), [Mojaloop Guide](mojaloop_guide.md) (interoperability switch) and [OpenBankProject Guide](openbankproject_guide.md) (open-banking API layer). It is built on the structural precedent of [Open-Source Insurance Systems and Commercial Systems](insurance_open_source_commercial_guide.md) (the open-vs-commercial method), the rail taxonomy of [Payment Rails Guide](payment_rails_guide.md), the hub architecture of [Payments Hub Guide](payments_hub_guide.md) and the core taxonomy of [Core Banking Systems Guide](core_banking_systems_guide.md). **Those guides' architectures, APIs and deployment detail are cross-referenced here by section, not re-derived.**

**Verification convention used throughout.** Every licence, archive flag, last-push timestamp and release in this guide was read on **2026-10-29** from the GitHub REST API (`/repos/{owner}/{repo}`) plus the raw `LICENCE`/`LICENSE` file at the canonical repository, and citations name the repository explicitly. Anything not read that way is marked as an assumption or listed in the claims audit (Sec. 15.1) and in what could not be verified (Sec. 15.2). No star counts, download counts, adoption numbers or market-share figures appear anywhere in this guide, by design: they are the numbers vendors quote and architects cannot audit.

**How to read this guide.** Sec. 1 gives the decoder and the thesis. Sec. 2 states what open source does and does not transfer. Sec. 3 is the five-category map; Sec. 4 places the three projects this series already owns inside that map. Sec. 5 and Sec. 6 cover the two layers this repository has never written about -- orchestration and ledger/billing. Sec. 7 and Sec. 8 are the licence and governance analysis, which is the part that cannot be delegated to engineering. Sec. 9 to Sec. 11 cover commercial comparison, selection and the regulated angle. Sec. 12 to Sec. 14 are the failure modes, a worked example, and the one-page summary. Sec. 15 is the audit trail.

---

## Table of Contents

1. [The Overview, the Decoder, and the Thesis](#1-the-overview-the-decoder-and-the-thesis)
   - 1.1 [The Question Behind the Guide](#11-the-question-behind-the-guide)
   - 1.2 [The Decoder: Ten Words That Are Not Interchangeable](#12-the-decoder-ten-words-that-are-not-interchangeable)
   - 1.3 [The Thesis](#13-the-thesis)
   - 1.4 [The Boundary of This Guide](#14-the-boundary-of-this-guide)
   - 1.5 [Why This Guide Exists at All](#15-why-this-guide-exists-at-all)
2. [What Open Source Does and Does Not Give You](#2-what-open-source-does-and-does-not-give-you)
   - 2.1 [The Nine Things a Payment System Needs](#21-the-nine-things-a-payment-system-needs)
   - 2.2 [The One Place the Boundary Genuinely Moves](#22-the-one-place-the-boundary-genuinely-moves)
3. [The Landscape by Category](#3-the-landscape-by-category)
   - 3.1 [The Five Categories](#31-the-five-categories)
   - 3.2 [Category One: The Rail](#32-category-one-the-rail)
   - 3.3 [Category Two: The Core Banking System](#33-category-two-the-core-banking-system)
   - 3.4 [Category Three: The Orchestrator](#34-category-three-the-orchestrator)
   - 3.5 [Category Four: The Ledger and Billing Layer](#35-category-four-the-ledger-and-billing-layer)
   - 3.6 [Category Five: The Open-Banking API Layer](#36-category-five-the-open-banking-api-layer)
   - 3.7 [The Category-Confusion Table](#37-the-category-confusion-table)
4. [The Three Foundation Projects, Situated](#4-the-three-foundation-projects-situated)
   - 4.1 [Apache Fineract: The Core Ledger in the Category Map](#41-apache-fineract-the-core-ledger-in-the-category-map)
   - 4.2 [Mojaloop: The Switch a Scheme Operates](#42-mojaloop-the-switch-a-scheme-operates)
   - 4.3 [The Open Bank Project: The Facade That Holds No Money](#43-the-open-bank-project-the-facade-that-holds-no-money)
   - 4.4 [What the Three Together Prove](#44-what-the-three-together-prove)
5. [The Orchestration Layer](#5-the-orchestration-layer)
   - 5.1 [What an Orchestrator Actually Does](#51-what-an-orchestrator-actually-does)
   - 5.2 [Why Open Source Matters Here Specifically](#52-why-open-source-matters-here-specifically)
   - 5.3 [Hyperswitch: The Verified Facts](#53-hyperswitch-the-verified-facts)
   - 5.4 [The Honest Limits](#54-the-honest-limits)
6. [The Ledger and Billing Layer](#6-the-ledger-and-billing-layer)
   - 6.1 [Billing Is Not a Ledger](#61-billing-is-not-a-ledger)
   - 6.2 [Kill Bill](#62-kill-bill)
   - 6.3 [Lago](#63-lago)
   - 6.4 [Blnk](#64-blnk)
   - 6.5 [Formance](#65-formance)
   - 6.6 [The Layer Table](#66-the-layer-table)
7. [The Licence Question](#7-the-licence-question)
   - 7.1 [The Four Families](#71-the-four-families)
   - 7.2 [The Family-by-Family Table](#72-the-family-by-family-table)
   - 7.3 [The Two Questions Legal Actually Asks](#73-the-two-questions-legal-actually-asks)
   - 7.4 [Why Licence Matters More in Payments](#74-why-licence-matters-more-in-payments)
   - 7.5 [The Live Cases a Bank Will Meet](#75-the-live-cases-a-bank-will-meet)
   - 7.6 [The Trap: Metadata Is Not the Licence](#76-the-trap-metadata-is-not-the-licence)
8. [The Governance Question](#8-the-governance-question)
   - 8.1 [Foundation-Governed](#81-foundation-governed)
   - 8.2 [Single-Vendor](#82-single-vendor)
   - 8.3 [The Four Risks Governance Decides](#83-the-four-risks-governance-decides)
   - 8.4 [What a Bank Should Require Contractually](#84-what-a-bank-should-require-contractually)
   - 8.5 [No Licence Gives You an SLA](#85-no-licence-gives-you-an-sla)
9. [The Commercial Comparison](#9-the-commercial-comparison)
   - 9.1 [What You Actually Pay For](#91-what-you-actually-pay-for)
   - 9.2 [The Honest TCO View](#92-the-honest-tco-view)
   - 9.3 [The Dimension Table](#93-the-dimension-table)
   - 9.4 [Where Open Source Genuinely Wins](#94-where-open-source-genuinely-wins)
   - 9.5 [Where It Does Not](#95-where-it-does-not)
10. [The Selection Guidance](#10-the-selection-guidance)
   - 10.1 [Test A: Which Category Does My Problem Belong To](#101-test-a-which-category-does-my-problem-belong-to)
   - 10.2 [Test B: Can We Operate a Payment-Critical Service](#102-test-b-can-we-operate-a-payment-critical-service)
   - 10.3 [Test C: Does the Licence Permit the Deployment](#103-test-c-does-the-licence-permit-the-deployment)
   - 10.4 [Test D: What Is the Exit Path](#104-test-d-what-is-the-exit-path)
   - 10.5 [Test E: Who Is Accountable](#105-test-e-who-is-accountable)
   - 10.6 [The OSS Due-Diligence Checklist](#106-the-oss-due-diligence-checklist)
11. [The Banking and Regulated Angle](#11-the-banking-and-regulated-angle)
   - 11.1 [Who Carries the Regulated Activity](#111-who-carries-the-regulated-activity)
   - 11.2 [PCI DSS: The Shape of Scope, Not the Size of Blame](#112-pci-dss-the-shape-of-scope-not-the-size-of-blame)
   - 11.3 [Scheme Rules and Certification](#113-scheme-rules-and-certification)
   - 11.4 [Outsourcing and Third-Party Risk](#114-outsourcing-and-third-party-risk)
   - 11.5 [Audit Evidence and Version Traceability](#115-audit-evidence-and-version-traceability)
   - 11.6 [Settlement, Liquidity, and the Part No Software Supplies](#116-settlement-liquidity-and-the-part-no-software-supplies)
12. [The Failure Modes](#12-the-failure-modes)
   - 12.1 [The Failure-Mode Table](#121-the-failure-mode-table)
   - 12.2 [The Pattern Behind the Six](#122-the-pattern-behind-the-six)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
   - 13.1 [The Scenario](#131-the-scenario)
   - 13.2 [Decision One: The Rail Pilot](#132-decision-one-the-rail-pilot)
   - 13.3 [Decision Two: The Orchestrator](#133-decision-two-the-orchestrator)
   - 13.4 [Decision Three: The Ledger That Was Rejected](#134-decision-three-the-ledger-that-was-rejected)
   - 13.5 [The Licence Review](#135-the-licence-review)
   - 13.6 [The Exit Paths](#136-the-exit-paths)
   - 13.7 [The Two Places the Thesis Was Tested](#137-the-two-places-the-thesis-was-tested)
14. [The One-Page Summary](#14-the-one-page-summary)
15. [Claims Audit, What Could Not Be Verified, Glossary and Closing](#15-claims-audit-what-could-not-be-verified-glossary-and-closing)
   - 15.1 [Claims Audit](#151-claims-audit)
   - 15.2 [What Could Not Be Verified](#152-what-could-not-be-verified)
   - 15.3 [Glossary](#153-glossary)
   - 15.4 [Cross-References and Closing](#154-cross-references-and-closing)

---

## 1. The Overview, the Decoder, and the Thesis

### 1.1 The Question Behind the Guide

A bank asking "can we do payments with open source?" is usually asking three different questions at once and does not know it. It may be asking whether it can stop paying per-transaction fees to a PSP, whether it can stop paying a licence for a core, or whether it can join a new instant-payment scheme without a vendor. Those are three different layers, with three different economics, and only one of them is a software question at all.

This guide separates them. It treats open-source payment software as a **portfolio of five categories** -- rail, core banking, orchestration, ledger/billing, and open-banking API -- and asks, for each: what does the software do, what does it not do, and what must the bank already own for the software to be useful. The answer to the last question is the whole point of the guide, because in payments the software is the cheap part.

### 1.2 The Decoder: Ten Words That Are Not Interchangeable

The vocabulary of payments is where most procurement mistakes begin. These ten terms are used interchangeably in vendor decks and in internal business cases, and they are not interchangeable. The last column is the column that matters: for each thing, who must supply it, whether or not any software is involved.

| Term | What it is | Who must supply it |
|---|---|---|
| **Payment rail** | The infrastructure and rules that carry a payment from payer to payee: the message flows, clearing, and settlement mechanism (RTGS, ACH, instant retail, card network). Taxonomy in [Payment Rails Guide](payment_rails_guide.md) Sec. 2 | A central bank, a scheme operator, or a card network. Never a software vendor, never a bank acting alone |
| **Scheme** | The rulebook and membership arrangement that sits on a rail: who may participate, what messages are valid, how disputes and settlement work, what certification is required | The scheme owner (often the central bank or a bank-owned operator). Membership is granted, not purchased with software |
| **Sponsor / acquiring bank** | The licensed institution whose membership, BIN range, or settlement account a non-member uses to reach a scheme, taking the scheme risk and the settlement obligation | A licensed bank or acquirer. This is a commercial and regulatory relationship, not a product |
| **PSP** | Payment service provider: the entity contracted to process payments for a merchant or bank, often bundling gateway, acquiring, tokenisation, and reporting | A licensed or sponsored provider. Can be a bank, an acquirer, or a technology company on someone else's licence |
| **Orchestrator** | Software that sits between the merchant or bank and multiple PSPs/acquiring connections: routing, retries, failover, one reporting surface, one integration for the channels | The bank itself, or a vendor. This is genuinely a software category (see Sec. 5) |
| **Ledger** | The double-entry book of accounts and postings that records what is owed and what is held: accounts, journals, postings, balances, immutability | The institution that is accountable for the numbers. Software can implement it; the accountability stays with the licensed entity (Sec. 6) |
| **Core banking system** | The system of record for customer accounts, products, interest, and the general ledger -- the book of record for the bank's own balance sheet. Taxonomy in [Core Banking Systems Guide](core_banking_systems_guide.md) Sec. 1 | The bank, under its own licence and its own audit. Software is substitutable; the licence is not |
| **Licence** | Two different things that share a word: the **software licence** (the legal terms of the code) and the **regulatory licence** (the permission to take deposits, issue e-money, or provide payment services) | Software licence: the copyright holder. Regulatory licence: the regulator. Confusing the two is the single most expensive category error in this space |
| **Settlement account** | The account at the central bank or a commercial bank through which the net or gross obligation of a payment is actually discharged in central-bank or commercial money | A central bank or a settlement bank, under a membership or correspondent relationship. No software supplies it |
| **Liquidity** | The funding standing behind the settlement account so that obligations can be met when they fall due, including intraday | The bank's treasury, backed by its balance sheet and its regulatory capital |

Read the table down the "who must supply it" column: of the ten, five are supplied by institutions and cannot be supplied by code, three are supplied by the institution that carries the accountability, and only two -- orchestrator and ledger implementation -- are ordinary software purchases. That ratio is the guide in one table.

### 1.3 The Thesis

**Open source gives you the software. It does not give you the licence, the settlement account, or the scheme membership -- and those are the product.**

Everything else in this guide is a consequence of that sentence. It is why an open-source core can be genuinely production-grade and still not make a bank into a payments business (Sec. 4). It is why the orchestration layer is the one place where open source changes the competitive position of an existing bank rather than just its cost base (Sec. 5). It is why the licence question in Sec. 7 is a regulatory question wearing legal clothing, and why the governance question in Sec. 8 is a continuity question wearing a community-health costume.

The thesis has a corollary that is less comfortable: because the software is the cheap part, the total cost of an open-source payment stack is dominated by the things that are *not* software -- integration, certification, 24x7 operations, audit evidence, and the staff to run a payment-critical service. A business case that compares a zero licence fee to a vendor quote and stops there has not done the analysis; it has done the sticker price (Sec. 9.2).

### 1.4 The Boundary of This Guide

This guide deliberately does not re-derive three bodies of work that already exist in this series. Each is cited by section, and anything that would read the same in those guides has been cut from this one.

| Project | Where the detail lives | What this guide does with it |
|---|---|---|
| Apache Fineract | [Apache Fineract Guide](apache_fineract_guide.md): capabilities Sec. 2, architecture Sec. 3, modules Sec. 4, API Sec. 6-Sec. 7, accounting Sec. 9, deployment Sec. 11, commercial comparison Sec. 15, role in a bank Sec. 16, production considerations Sec. 17 | Situates it as the **core ledger** category and asks what its existence proves about the thesis (Sec. 4.1) |
| Mojaloop | [Mojaloop Guide](mojaloop_guide.md): switch concept Sec. 2, FSPIOP API Sec. 5, hub architecture Sec. 6, settlement models Sec. 11, liquidity Sec. 12, governance Sec. 20, commercial alternatives Sec. 21, banking context Sec. 22 | Situates it as the **rail** category and asks who must be a scheme member for it to be useful (Sec. 4.2) |
| OpenBankProject | [OpenBankProject Guide](openbankproject_guide.md): API Sec. 2, products Sec. 3, PSD2 angle Sec. 4, open-source model Sec. 7, comparison Sec. 8, worked example Sec. 9 | Situates it as the **API facade** category and asks what it means that it holds no money (Sec. 4.3) |

The structural precedent for this guide's shape and voice is [Open-Source Insurance Systems and Commercial Systems](insurance_open_source_commercial_guide.md) -- in particular its open-vs-commercial method (Sec. 5), its selection framework (Sec. 7.3) and its honesty about TCO (Sec. 5.2). This guide applies that method to a category where the answer is much more favourable to open source than it was in insurance, and says so where it is.

### 1.5 Why This Guide Exists at All

The measurement is worth stating plainly, because it explains the shape of what follows. This repository held 682 markdown files before this guide was added, by a tracked-file count on 2026-10-29. The three open-source payment projects are owned deeply: Apache Fineract is anchored across roughly 25 files, Mojaloop across 9, and the Open Bank Project has its own 619-line guide. The *category* was unowned.

Measured on 2026-10-29 across every markdown file in the repository:

| Measurement | Result |
|---|---|
| Files matching `\bHyperswitch\b` | 0 |
| Files matching `\bKill ?Bill\b` | 0 |
| Files matching `\bLago\b` | 0 |
| Files matching `\bBlnk\b` | 0 |
| Files matching `\bFormance\b` | 1 -- three lines, all inside a vendor-testimonial table in [Antithesis Guide](../technology/antithesis_guide.md) Sec. 14, nothing to do with payments |
| Files where a single paragraph contains "open source" and "licence" and "payment" | 13 -- and every one is incidental: a cost row in a comparison table, a product description, a glossary line. Not one treats the licence of payment software as a subject |

So the two layers where open-source payments is actually moving fastest -- orchestration and ledger/billing -- had zero coverage, and the question that decides whether a bank may legally deploy any of it had no coverage either. That is the gap this guide fills: not another deep-dive on a project the series already knows, but the landscape those projects sit in, and the licence and governance analysis that surrounds all of them.

---

## 2. What Open Source Does and Does Not Give You

### 2.1 The Nine Things a Payment System Needs

The thesis is easier to defend as a table than as an argument. For each thing a payment system needs in order to exist, the question is not "is there open-source software for this" but "can any open-source project supply this at all".

| Requirement | What it is | Who must supply it | Can any OSS project supply it? |
|---|---|---|---|
| **Software** | The code: routing logic, ledger postings, APIs, connectors | Whoever writes or adopts it | **Yes.** This is the entire open-source proposition, and it is real |
| **Software licence** | The legal terms under which the code may be used, modified, and offered as a service | The copyright holder (a foundation, a vendor, or a community) | **Yes, but not on your terms.** The terms are whatever the project chose, including copyleft terms that constrain service delivery (Sec. 7) |
| **Settlement account** | The account in central-bank or commercial money where obligations are discharged | A central bank, or a settlement/correspondent bank | **No.** It is granted under a membership or banking relationship |
| **Scheme membership** | The right to participate in a scheme: to send and receive its messages, to be bound by its rulebook | The scheme owner, on application and certification | **No.** Even where the software implements the scheme's protocol, membership is a contractual and regulatory grant |
| **BIN sponsorship** | The card range and sponsorship under which card transactions are authorised and settled | A licensed acquirer or issuer, or a sponsor bank | **No.** It is an entitlement attached to a licence and a scheme relationship |
| **Correspondent banking relationship** | The bilateral account and credit arrangement through which cross-border and correspondent settlement happens | Another bank, under its own risk appetite, KYC, and credit approval | **No.** It is a commercial relationship between institutions |
| **Regulatory capital** | The capital held against payment and settlement risk, operational risk, and credit exposure | The licensed institution's balance sheet | **No.** Software does not appear in a capital return |
| **PCI attestation** | The independent assessment that cardholder data is handled to the standard | A QSA, against the institution's own scope and controls | **No.** Self-hosting can change the *shape* of scope; it never removes the attestation obligation (Sec. 11.2) |
| **Operational staff** | The people who run a payment-critical service: on-call, change control, incident management, reconciliation | The institution | **No.** An open-source project gives you no rota, no runbook, and no one to call at 02:00 |

Three of these nine are genuinely supplied by open source, and of those three, one (the software licence) is supplied on terms the adopter does not choose. Six are institutional. The table is the thesis, line by line.

### 2.2 The One Place the Boundary Genuinely Moves

There is a real exception, and it is worth stating precisely rather than dismissing, because it is where the interesting architecture lives.

**Case one: software that a bank runs inside its own regulated perimeter.** Here nothing moves. A bank running an open-source ledger, an open-source orchestrator, or an open-source core inside its own perimeter is still the regulated entity for every payment that software touches. The software changes the bank's cost base and its control over the code; it does not change who answers to the regulator, who holds the settlement account, or who is a scheme member. This is the ordinary case, and it is where almost all bank open-source adoption sits.

**Case two: a project that ships scheme-facing conformance tooling.** This is where something genuinely changes, and it is narrower than it sounds. When a project ships a test suite, a simulator, or a conformance harness that a scheme or a participant uses to prove protocol compliance, it changes the *cost and speed of certification* -- a bank can self-test against the same artefact the scheme tests against, instead of discovering non-conformance during scheme certification. Mojaloop's testing tools are the clearest example in this series ([Mojaloop Guide](mojaloop_guide.md) Sec. 19). What changes: the certification timeline, and the ability to fail early. What does not change: certification is still granted by the scheme, not by the tooling; the tooling makes the exam cheaper, it does not sit the exam for you.

**Case three: a central switch that a central bank or scheme operator runs.** Here the boundary moves the furthest, and again in a specific way. When a central bank operates an open-source switch, the *operator* is the institution that already owns the settlement account and the rulebook -- so the software is genuinely the missing piece, and adopting it is the difference between a scheme existing and not existing. That is the Mojaloop model, and it is why Mojaloop adoption is a scheme decision rather than a bank procurement decision ([Mojaloop Guide](mojaloop_guide.md) Sec. 2, Sec. 17). It is also why a commercial bank adopting the same software does not become a scheme: for the bank, the switch is a participant-side or infrastructure-side component, and the scheme membership still has to be granted.

The pattern across all three cases: **the boundary moves where the adopter already holds the institutional asset.** A central bank that owns settlement and the rulebook gains the most from open-source switch software; a bank that owns neither gains least from pretending it does. Open source amplifies an institutional position; it does not create one.

## 3. The Landscape by Category

### 3.1 The Five Categories

Every open-source payment project worth naming belongs to one of five categories. The categories are not a taxonomy of technology; they are a taxonomy of **who is accountable**, which is why a project that looks adjacent in a feature list can be in a completely different category in an architecture review.

| Category | The question it answers | Representative open-source projects | Who ends up accountable |
|---|---|---|---|
| **1. Rail** | How does money move between institutions, and under whose rules? | Mojaloop (`mojaloop/central-ledger`, Apache-2.0), Interledger Rafiki (`interledger/rafiki`, Apache-2.0), the Open Payments protocol spec (`interledger/open-payments`, Apache-2.0) | The scheme operator or central bank; the participant for its own obligations |
| **2. Core banking** | What does this customer hold, what do they owe, and what is on the bank's books? | Apache Fineract (`apache/fineract`, Apache-2.0) | The licensed bank |
| **3. Orchestration** | Which PSP or acquirer should carry this transaction, and what happened across all of them? | Hyperswitch (`juspay/hyperswitch`, Apache-2.0) | The bank or merchant that operates it |
| **4. Ledger and billing** | What has each customer been charged, what have they paid, and do the books balance? | Kill Bill (`killbill/killbill`, Apache-2.0), Lago (`getlago/lago`, AGPL-3.0), Blnk (`blnkfinance/blnk`, detected Apache-2.0 -- see Sec. 6.4), Formance (`formancehq/ledger`, no declared licence -- see Sec. 6.5) | The institution whose books they are |
| **5. Open-banking API layer** | How do third parties read account data and initiate payments, under consent? | OpenBankProject (`OpenBankProject/OBP-API`, AGPL-3.0) | The bank that exposes the API |

Two of these five categories -- orchestration and ledger/billing -- are where the open-source ecosystem has moved most in the last two years, and both were absent from this repository before this guide (Sec. 1.5). The other three are the ones this series already owns, and Sec. 4 does nothing more than place them.

### 3.2 Category One: The Rail

**What it does.** Moves value between institutions. A rail implementation handles the message flows of a scheme -- the request, the quote, the transfer preparation and fulfilment, the timeout and the reversal -- and produces positions that are then settled through the scheme's settlement mechanism. The mechanism itself (deferred net settlement, RTGS integration, liquidity management) is the rail's core intellectual content; [Mojaloop Guide](mojaloop_guide.md) Sec. 11-Sec. 13 is the detailed treatment.

**What it does NOT do.** It does not hold customer accounts, does not compute interest, does not decide who may participate, and does not settle anything by itself: settlement happens in central-bank or commercial money, at a settlement account the operator already holds. It also does not make its adopter a scheme member.

**The one-line question it answers:** *how does a payment get from a customer of institution A to a customer of institution B, and who is bound by what rules when it does?*

**The projects that occupy it.** Mojaloop is the reference implementation of an inclusive instant payment switch, designed to be operated by a scheme or central bank. Rafiki and the Open Payments protocol occupy an adjacent position: an interoperability model built around Interledger rather than a domestic switch, with an openly specified protocol and a reference implementation. Bitcoin-based payment processors such as BTCPay Server (`btcpayserver/btcpayserver`, MIT) sit at the edge of this category for a different reason: they are a rail plus a merchant-facing acceptance layer in one, which is exactly the category confusion Sec. 3.7 warns about.

**The standard deployment posture.** Operated by a scheme operator or central bank as national infrastructure, with participants connecting through an API; or, for a bank, consumed as a participant-side integration rather than operated. A bank that finds itself *operating* a rail implementation has usually taken on a scheme-operator role, which is a regulatory and commercial decision that vastly exceeds a technology choice.

### 3.3 Category Two: The Core Banking System

**What it does.** Holds the book of record: customer accounts, products, balances, interest, fees, and the general ledger. In Fineract's case the emphasis is loans and savings for inclusive-finance institutions, with a real double-entry general ledger underneath ([Apache Fineract Guide](apache_fineract_guide.md) Sec. 4.3, Sec. 4.4, Sec. 9).

**What it does NOT do.** It is not a card acquirer, does not connect to card schemes, is not a scheme member, does not hold a settlement account, and is not a payments hub. It posts what it is told to post; the instruction to post arrives from somewhere else.

**The one-line question it answers:** *what does this customer hold and owe, and what does the bank's own book look like as a result?*

**The projects that occupy it.** Apache Fineract is the only production-grade open-source core in this series' coverage, and the commercial landscape around it is deep and well documented ([Core Banking Systems Guide](core_banking_systems_guide.md) Sec. 5; [Apache Fineract Guide](apache_fineract_guide.md) Sec. 15).

**The standard deployment posture.** Self-hosted by the institution, inside its own perimeter, under its own audit. The commercial variant of the same posture is a managed or cloud core. Note what does not change between them: the institution is the system of record's accountable owner in both cases.

### 3.4 Category Three: The Orchestrator

**What it does.** Sits between a merchant or bank and several PSPs or acquiring connections, and turns N integrations into one: it routes each transaction to a processor by rule (cost, success rate, currency, issuer country, ticket size), retries and fails over when one processor declines or times out, presents one reporting and reconciliation surface across all of them, and increasingly sits adjacent to tokenisation and vaulting. The pattern is the same N-to-M reduction the payments hub performs for rails ([Payments Hub Guide](payments_hub_guide.md) Sec. 1.2, Sec. 6), applied at the acquiring edge instead of the scheme edge.

**What it does NOT do.** It does not acquire. It does not sponsor a BIN. It does not remove the bank from PCI scope -- it may enlarge it, because the card data now flows through software the bank operates (Sec. 11.2). It does not certify anything with a scheme. It does not create an acquiring relationship that did not already exist.

**The one-line question it answers:** *given that we already have several ways to accept a payment, which one should carry this one, and what happened across all of them?*

**The projects that occupy it.** Hyperswitch is the reference open-source orchestrator in this guide (Sec. 5.3). The commercial equivalents are the payment orchestration vendors and the PSPs' own routing products; commercially, orchestration is usually bought as a feature of a PSP rather than as a separate product.

**The standard deployment posture.** Self-hosted in front of existing acquirers, or consumed as a managed service. Self-hosting is the posture that produces the strategic benefit -- routing logic and card data stay in-house -- and also the posture that produces the operational burden (Sec. 5.4).

### 3.5 Category Four: The Ledger and Billing Layer

**What it does.** Two related but distinct jobs. A **billing** system rates usage or subscriptions, produces invoices, runs dunning, and tracks receivables. A **ledger** records double-entry postings between accounts and keeps balances that reconcile. Some products do both; many do one and claim the other (Sec. 6.1).

**What it does NOT do.** Neither replaces a core banking book of record for a licensed bank's balance sheet, and neither moves money: they record obligations and postings, they do not instruct a rail. A ledger that is not the bank's book of record is a subledger, and calling it "the ledger" is the most common abuse of the word in this space.

**The one-line question it answers:** *what has each customer been charged, what have they paid, and does the book balance?*

**The projects that occupy it.** Kill Bill (subscription billing and invoicing), Lago (usage-based billing and metering), Blnk (a ledger with a payments-adjacent surface), Formance (a programmable financial ledger). All four are covered with dated licence facts in Sec. 6.

**The standard deployment posture.** Self-hosted as a subledger or billing engine beside the core, or SaaS. For a bank, the realistic posture is a bounded subledger for a product line -- prepaid balances, wallet float, merchant receivables -- not a replacement for the core posting engine (Sec. 13.4).

### 3.6 Category Five: The Open-Banking API Layer

**What it does.** Exposes a bank's accounts, transactions, payees, and payment initiation to third parties under consent, translating between the bank's internal systems and external standards (PSD2 XS2A, UK Open Banking, Berlin Group). It is a facade: it reads from the core and writes instructions back to it.

**What it does NOT do.** It holds no money, posts no entries, and computes no interest -- [OpenBankProject Guide](openbankproject_guide.md) Sec. 1.1 is explicit on this, and the point is load-bearing for the thesis: a project can be genuinely important to a bank's payments strategy and still be, functionally, a translation layer.

**The one-line question it answers:** *how does a regulated third party get consented access to this bank's data and payment initiation, in a standard shape?*

**The projects that occupy it.** The Open Bank Project is the long-running open-source option (Sec. 4.3). Most banks in practice build or buy a gateway; the open-source option's value is in the standard mappings and the consent model, not in the runtime.

**The standard deployment posture.** In front of the core, inside the bank's perimeter, under the bank's API security and consent controls. Its availability requirement is high but its blast radius is bounded by the fact that it cannot move money on its own authority.

### 3.7 The Category-Confusion Table

The most expensive mistakes in this space are category mistakes, and they are made in business cases rather than in architectures. Each row below is a real pattern of confusion, stated as the symptom a stakeholder brings and the category the problem actually belongs to.

| What the stakeholder says | What they probably need | Category | The confusion |
|---|---|---|---|
| "We need to accept cards online" | An acquiring relationship plus, if there are several, an orchestrator | Orchestration (+ sponsor/acquirer) | Confused with ledger: accepting a card is not the same as recording what was charged |
| "We need to move money between two banks in this country" | Scheme membership on a rail | Rail | Confused with core banking: a core posts the result, it does not carry the payment |
| "We need to know what each customer owes and has paid" | Billing and/or a subledger | Ledger/billing | Confused with the core: a billing engine is not a book of record |
| "We need to expose account data to third parties" | An API facade plus consent management | Open-banking API | Confused with core: OBP-style layers hold no money |
| "We need to replace our core" | A core banking programme | Core banking | Confused with orchestration: routing transactions is not posting them |
| "We need to cut per-transaction PSP fees" | Renegotiation, plus possibly routing | Orchestration (commercial) | Confused with licence cost: the fee is a scheme/acquirer charge, not a software licence |
| "We need an open-source Stripe" | Probably three categories at once | Orchestration + ledger + rail access | The most common single confusion: "Stripe" is a company spanning four categories, so no single project can be its equivalent |
| "We are joining a national instant payment scheme -- which OSS do we install?" | Scheme onboarding, then integration | Rail (as participant) | Confused with procurement: the membership is granted, and the software is the smaller half of the work |

## 4. The Three Foundation Projects, Situated

Each of these three projects has its own guide in this series. What follows is deliberately not a summary of those guides: it is a placement exercise. For each project, the questions are (a) which of the five categories does it actually occupy, (b) what does its existence prove about the thesis, and (c) what does it explicitly not hand the adopter. Where a sentence would read identically in the project's own guide, it has been removed -- follow the section pointers for architecture, API, and deployment.

### 4.1 Apache Fineract: The Core Ledger in the Category Map

**Category occupied: core banking (category 2), and within it, the book of record.** Fineract is a modular monolith whose centre of gravity is the loan and savings portfolio and the double-entry general ledger beneath it ([Apache Fineract Guide](apache_fineract_guide.md) Sec. 3, Sec. 4.3-Sec. 4.4, Sec. 9). Its default branch is `develop`; the last push was 2026-10-29T20:58Z and the latest release is 1.16.1 (2026-10-26), both read from the GitHub REST API on 2026-10-29.

**What it proves about the thesis.** Fineract is the strongest counter-example to the lazy version of the thesis, and it needs to be handled honestly. It proves that a foundation-governed, Apache-2.0 core can be genuinely production-grade for its niche -- inclusive finance institutions that need a real double-entry book and a loan engine and cannot pay commercial licence fees. It proves that open source can be the *core*, not just the edges. That is a significant result and this guide does not argue with it.

What Fineract does not prove is that the software was the hard part. Its adopters are institutions that already hold, or are sponsored into, the institutional assets: a licence to lend and take deposits, an audit relationship, a settlement position with their central bank or correspondent. Fineract gave them the book; they already had the accountability. That is exactly the division of labour the thesis predicts, and Fineract is the cleanest illustration of it in this series.

**What it does not give you.** Not a card acquirer; not a scheme member; not a payments hub; not an orchestration layer; not a settlement account; not a compliance pack. A bank that installs Fineract and expects to be "doing payments" has confused category 2 with categories 1 and 3. Its payment connectivity comes from elsewhere -- which is precisely why the Mifos ecosystem's payment hub exists as a separate component ([Payments Hub Guide](payments_hub_guide.md) Sec. 11).

**Where to read the detail.** Capabilities Sec. 2, architecture Sec. 3, functional modules Sec. 4, REST API Sec. 6, hands-on API Sec. 7, accounting integration Sec. 9, multi-tenancy Sec. 10, deployment Sec. 11, ecosystem integrations Sec. 13, commercial comparison Sec. 15, role in a bank's architecture Sec. 16, production considerations Sec. 17.

### 4.2 Mojaloop: The Switch a Scheme Operates

**Category occupied: rail (category 1) -- and specifically, the operator side of a rail.** Mojaloop is a reference implementation of an inclusive instant payment switch: the hub through which participating providers interoperate, with deferred net settlement and RTGS integration at the back ([Mojaloop Guide](mojaloop_guide.md) Sec. 2, Sec. 6, Sec. 11). The repository measured here is `mojaloop/central-ledger`: Apache-2.0 via a `LICENSE.md` whose preamble reads "Copyright (c) 2020-2025 Mojaloop Foundation ... Apache License, Version 2.0"; last push 2026-10-29T19:38Z; latest release v20.3.1 (2026-10-22).

Note the measurement trap in the same repository family: `mojaloop/mojaloop` -- the onboarding and documentation repository -- last pushed 2025-03-11. A due-diligence pass that samples the wrong repository concludes the project is dormant. It is not; the code repositories are active. This is a general lesson about health signals, returned to in Sec. 10.6.

**What it proves about the thesis.** Mojaloop is the clearest case in the whole landscape of the thesis operating *in favour* of open source, because it moves the boundary by moving who the adopter is. When a central bank or a scheme operator runs Mojaloop, the adopter already owns the settlement account, the rulebook, and the membership decisions. For that adopter, the software genuinely is the missing piece, and the open-source model is what makes a national switch affordable at all -- the alternative was a commercial switch licence that the market could not bear ([Mojaloop Guide](mojaloop_guide.md) Sec. 1.1, Sec. 21). The Gates Foundation lineage and the foundation governance are what made it credible ([Mojaloop Guide](mojaloop_guide.md) Sec. 20).

For a commercial bank, the same software proves something different and less flattering: adopting it does not make the bank a scheme. It makes the bank a participant, or a technology provider to a scheme, and the membership still has to be granted. The interesting failure mode is a bank that mistakes "we deployed the switch software" for "we are the scheme".

**What it does not give you.** Not settlement: settlement happens in central-bank or commercial money at an account the operator holds ([Mojaloop Guide](mojaloop_guide.md) Sec. 11). Not liquidity: positions and limits are managed by the operator against real funding (Sec. 12 of that guide). Not membership, not rulebook authority, not a dispute regime. The software implements the scheme's protocol; the scheme is an institution.

**Where to read the detail.** Switch concept Sec. 2, design goals Sec. 3, FSPIOP API Sec. 5, hub architecture Sec. 6, core services Sec. 7, SDK/scheme adapter Sec. 8, transfer flow Sec. 9, state machine Sec. 10, settlement models Sec. 11, liquidity and limits Sec. 12, deployment Sec. 16, national deployments Sec. 17, DFSP integration patterns Sec. 18, testing tools Sec. 19, governance Sec. 20.

### 4.3 The Open Bank Project: The Facade That Holds No Money

**Category occupied: open-banking API layer (category 5) -- a facade in front of the core.** OBP-API is a Scala RESTful platform that exposes accounts, transactions, counterparties, payments, entitlements, and metadata, mapping to Berlin Group NextGenPSD2 XS2A and UK Open Banking consent scoping alongside its own API standard ([OpenBankProject Guide](openbankproject_guide.md) Sec. 1.1, Sec. 2). Measured on 2026-10-29: AGPL-3.0, archived false, default branch `develop`, last push 2026-10-29T08:01Z, and **no GitHub "releases" at all** -- only tags (v3.0.0, v2.1.0, v1.4.0). That release-cadence observation is recorded honestly rather than smoothed: the code is actively pushed, but the project does not publish versioned releases through GitHub's release mechanism, so "latest release" is not a usable health signal for it (Sec. 10.6).

**What it proves about the thesis.** OBP proves that a project can be strategically important to a bank and still be, in money terms, nothing. It holds no money, posts no entries, and cannot move funds on its own authority; its entire value is standard mapping and consent structure. A bank can adopt it, expose its accounts to third parties, and remain exactly as accountable as before -- because nothing about the money moved. The category is real and the software is real; the institutional assets were never in question.

It also proves something about licensing that the rest of the open-source payments world tends to skip: a strategically important facade can be copyleft. OBP is AGPL-3.0 with a commercial licence sold by TESOBE ([OpenBankProject Guide](openbankproject_guide.md) Sec. 7.1). Banks do deploy AGPL software -- usually under the commercial licence, or after a legal review that concludes internal use is unaffected. That is the live case discussed in Sec. 7.5.

**What it does not give you.** Not a core, not a ledger, not a scheme membership, not a payment rail. Its availability matters to channel partners, not to settlement. And its dual licensing means the licence question is a commercial negotiation, not a reading exercise.

**Where to read the detail.** Overview Sec. 1, the API Sec. 2, products Sec. 3, the PSD2 angle Sec. 4, deployments Sec. 5, the open-source model Sec. 7, comparison with alternatives Sec. 8, the worked developer-portal example Sec. 9.

### 4.4 What the Three Together Prove

Read as a set rather than individually, the three projects make the thesis concrete in a way no single one does:

| Project | Category | Institutional asset the adopter must already hold | What open source actually contributed |
|---|---|---|---|
| Fineract | Core ledger | Banking licence, audit relationship, settlement position | A double-entry book and a loan engine, at zero licence cost |
| Mojaloop | Rail (operator side) | Settlement account, rulebook authority, scheme mandate -- or a scheme that has them | A switch implementation a national scheme could afford |
| OBP | API facade | Core systems of record, consent and API security controls | Standard mappings and a consent model |

The pattern: **in every case the project supplied exactly the layer that is not regulated, and the adopter supplied exactly the layer that is.** Fineract did not become a bank. Mojaloop did not become a central bank. OBP did not become a custodian of anything. Each one is a well-built answer to "who writes this code", and none is an answer to "who is allowed to do this".

That is also why the three projects *can* be Apache-2.0 or AGPL-3.0 without destabilising anything: the licences govern code, and the regulated permissions were never in the code to begin with. The next two sections move to the two categories where the repository had nothing, and where the licence question bites hardest because the software sits closest to the money.

## 5. The Orchestration Layer

### 5.1 What an Orchestrator Actually Does

An orchestrator is the software a merchant or a bank puts between its own channels and its several ways of accepting a payment. It exists because most institutions end up with more than one: a primary acquirer, a backup, a regional acquirer for local methods, a wallet provider, a BNPL partner. Without an orchestrator, each channel integrates with each processor, and the N-times-M integration problem that [Payments Hub Guide](payments_hub_guide.md) Sec. 1.2 describes at the scheme edge reappears at the acquiring edge.

| Function | What it does in practice | What it does not do |
|---|---|---|
| **Smart routing** | Chooses a processor per transaction by rule: cost, historical success rate, issuer country, currency, ticket size, card type, BIN range | Improve the acquirer's authorisation rate; that is the acquirer's and the issuer's business |
| **Retries and failover** | Re-submits or re-routes a declined or timed-out transaction according to rules, with de-duplication so a retry is not a double charge | Recover money already lost to a fraud or a chargeback |
| **Unified reporting** | One transaction record across processors, one reconciliation surface, one place to look when a merchant says "where is my money" | Reconcile against the settlement account; that is a treasury and reconciliation function |
| **Tokenisation adjacency** | Holds or brokers network tokens and vaulted credentials so channels do not each store cards | Remove the institution from PCI scope; storing or transmitting card data is exactly what creates scope |
| **Routing governance** | Makes the routing rules explicit, versioned, and reviewable -- the part that is usually buried inside a PSP's black box | Decide the commercial terms with the acquirer |

The last row is the strategic one. In most banks, the routing logic that decides which acquirer sees which transaction is a configuration inside a PSP. It is invisible, unversioned from the bank's point of view, and impossible to audit. Owning the orchestrator makes that logic a first-class artefact the bank can review, change, and explain.

### 5.2 Why Open Source Matters Here Specifically

Orchestration is the one category in this guide where open source changes a bank's *position*, not just its cost line, and the reason is structural.

**The orchestration layer is where PSP lock-in is manufactured.** Lock-in in payments is rarely created by the acquirer relationship, which is commercial and renegotiable. It is created by the integration: once every channel speaks one PSP's API and every report comes from one PSP's portal, switching cost is measured in channel rewrites. An orchestrator that the bank operates reduces the PSP to a pluggable connection with a contract behind it. The lock-in moves from the code to the contract, where it belongs and where it can be priced.

**Self-hosting keeps card data and routing logic inside the perimeter.** If the bank operates the orchestrator, the card data flows through systems the bank controls and audits, and the routing rules are the bank's own artefact. That is a genuine sovereignty gain, and it is also a genuine scope increase (Sec. 11.2). Both halves of that sentence are true and both have to be in the business case.

**The routing logic becomes auditable end to end.** For a bank that has to explain to a regulator, a scheme, or an internal audit why a given transaction went where it went, an open-source orchestrator makes the answer a code review rather than a vendor conversation. This is the same transparency argument the insurance guide makes for open rating logic ([Open-Source Insurance Systems and Commercial Systems](insurance_open_source_commercial_guide.md) Sec. 5.4), and it is stronger here because the money moves.

### 5.3 Hyperswitch: The Verified Facts

Measured on 2026-10-29 from the GitHub REST API and the repository's root licence file:

| Attribute | Value (read 2026-10-29) |
|---|---|
| Repository | `juspay/hyperswitch` |
| Licence | **Apache-2.0** -- the root `LICENSE` file is the Apache License 2.0 text |
| Archived | false |
| Last push | 2026-10-29T21:23Z |
| Latest release | v1.128.0 (2026-10-16) |
| Prior coverage in this repository | **Zero files.** `\bHyperswitch\b` matched no markdown file in the repository before this guide (Sec. 1.5) |

The release cadence -- a version number in the v1.128 range with a release ten days before the measurement date, and a push on the measurement date itself -- is the kind of signal that distinguishes an actively developed project from a maintained one. It is also the only kind of health claim this guide makes: a dated push timestamp and a dated release, not a description of momentum.

**What it means for a bank.** Hyperswitch is the reference open-source implementation of the orchestration category in this guide, and it is the single most directly relevant project here for a bank that already accepts cards. It is Apache-2.0, which for the two legal questions in Sec. 7.3 means: yes, modify privately; yes, run as a service, subject to the ordinary Apache-2.0 notice obligations rather than a source-disclosure obligation.

### 5.4 The Honest Limits

The limits are not small, and a business case that omits them is the failure mode in Sec. 12.

1. **You still need acquiring relationships and a sponsor.** The orchestrator routes to acquirers; it does not become one. If the bank has no acquiring relationship, an orchestrator has nothing to route to. This is the thesis in its most concrete form: the software is available, the BIN sponsorship is not.
2. **Certification and PCI obligations do not move.** Self-hosting does not reduce the number or difficulty of scheme certifications the bank needs, and it does not reduce PCI DSS obligations -- it may increase the bank's own scope by bringing card data into systems it operates (Sec. 11.2). A QSA will still attest; a scheme will still certify.
3. **You now operate a payment-critical service.** An orchestrator sits in the authorisation path. That means latency budgets measured in hundreds of milliseconds, an on-call rota, change control that assumes every deploy can decline transactions, capacity planning against peak shopping windows, and an incident process that can talk to acquirers and merchants at the same time. This is the largest hidden cost in the category and it is a staffing decision, not a technology decision.
4. **Routing is only as good as the data behind it.** Smart routing needs per-processor performance data to learn from. A bank with two acquirers and modest volume will get less from routing than a large merchant with five, and should not assume the routing benefit materialises at its scale.
5. **The vendor question does not disappear, it changes shape.** A single-vendor project (Hyperswitch is a Juspay project, Sec. 8.2) means the bank depends on one company's continued investment. The Apache-2.0 licence guarantees the code stays available; it guarantees nothing about the roadmap, the security patches, or the company (Sec. 8.5).

The honest summary of the category: open source makes orchestration *ownable*, which is a strategic gain for a bank with the engineering organisation to run it, and a liability for a bank without one. It does not make card acceptance free, and it does not move a single institutional asset.

---

## 6. The Ledger and Billing Layer

### 6.1 Billing Is Not a Ledger

The word "ledger" is the most abused noun in fintech, and the abuse is expensive because it causes banks to evaluate billing software as if it were a book of record.

| | **Billing system** | **Double-entry ledger** |
|---|---|---|
| Core objects | Plans, prices, meters, subscriptions, invoices, credit notes | Accounts, journals, postings, balances |
| Core operation | Rate usage and produce an amount due | Post balanced debits and credits atomically |
| Correctness test | Does the invoice match the contract and the meter? | Do the debits equal the credits, always, and can every balance be derived from its postings? |
| Failure that matters | A wrong invoice (annoying, correctable, reputational) | An unbalanced book (an audit finding, potentially a restatement) |
| Idempotency | Nice to have: re-running a billing run should not double-charge | Mandatory: a retried posting must not double-post, and this is the hardest engineering property in the category |
| Time model | Billing periods, proration, retroactive rating | Effective dates, immutability, and no deletion -- corrections are postings, not edits |

A billing engine can live without a ledger (it produces invoices and hands them to receivables). A ledger can live without billing (postings arrive from somewhere). Products that do both are useful, but the bank must know which of the two it is buying, because the acceptance test is different: you test billing against a contract, and you test a ledger against arithmetic that must never fail.

Why the confusion persists: "ledger" sells. A billing product described as a billing product is a finance-ops tool; the same product described as a ledger sounds like infrastructure. The four projects below span the whole range from "billing only" to "ledger-first", and the label on the repository is not a reliable guide to which.

### 6.2 Kill Bill

**What it is.** Subscription billing and invoicing: plans, phases, add-ons, usage metering, invoicing, payments retry, dunning, entitlements. The category is billing (left column of Sec. 6.1), with enough of an account-and-invoice model to support subscription businesses; it is not a general double-entry book of record.

**Verified on 2026-10-29:** `killbill/killbill`, **Apache-2.0** -- the root `LICENSE` file was confirmed as the Apache License 2.0. Archived: false. Last push 2026-10-28T23:42Z. Latest release `killbill-0.24.22` (2026-09-29).

**Prior coverage in this repository: zero files** (`\bKill ?Bill\b` matched nothing). The dated release and a push the day before measurement are consistent with a maintained project; the version numbering (0.24.x after many years) is a project-specific convention and should not be read as immaturity without checking the release history.

**Where it fits a bank.** Subscription and recurring-revenue products where the bank is the biller -- a merchant-acquiring fee engine, a software or platform business inside the group, a wealth-management fee schedule. Not the core posting engine.

### 6.3 Lago

**What it is.** Usage-based billing and metering: events in, metered aggregates, rated invoices out, with the subscription constructs needed around them. It is the open-source answer to metered pricing (API calls, transactions, seats, storage), which is the pricing shape most fintech and platform businesses actually use.

**Verified on 2026-10-29:** the canonical repository is **`getlago/lago`** (note: `lago-access/lago` returns 404 -- a stale path that appears in some third-party lists). Licence **AGPL-3.0**: the root `LICENSE` file is the GNU Affero General Public License v3. Archived: false. Last push 2026-10-29T20:37Z. Latest release v1.55.0 (2026-10-07).

**Prior coverage in this repository: zero files** (`\bLago\b` matched nothing).

**The licence is the headline here, not the features.** Lago is the clearest AGPL case in the ledger/billing category, and the consequence is examined in Sec. 7.5: a bank running it internally is unaffected, a bank (or its SI) offering metered billing as a hosted service to third parties is in the territory where the AGPL's network-service obligation has to be analysed by counsel rather than by an architect.

### 6.4 Blnk

**What it is.** A ledger-oriented product with a payments-adjacent surface: accounts, balances, transactions, and reconciliation constructs, positioned as the book underneath a payments product rather than as a billing engine. Of the four, it is the one that most directly claims the "ledger" word.

**Verified on 2026-10-29:** `blnkfinance/blnk`. GitHub's licence detection reports **Apache-2.0**. Archived: false. Last push 2026-10-24T03:16Z. Latest release v0.15.7 (2026-10-24).

**The caveat, stated rather than smoothed:** in this pass, **no root `LICENSE` file was resolvable at branch `main`** -- the raw fetch returned 404. GitHub's licence field is a detector's inference from files it found, not a legal instrument, and here the instrument itself could not be read. That is a flag, not a disqualification: the file may exist under another name, in another path, or on another branch, and the detector may have found it there. But until a human opens the actual licence text and dates it, the licence of Blnk is *asserted by metadata, not verified by document*. Any bank evaluating it must resolve that before anything else (Sec. 15.2).

**Prior coverage in this repository: zero files** (`\bBlnk\b` matched nothing).

### 6.5 Formance

**What it is.** A programmable financial ledger: accounts, transactions, and a query/scripting model for building money-movement logic on top of a double-entry core. Its sibling products extend toward payments and a broader stack.

**Verified on 2026-10-29 -- and this is the most important licence finding in the guide:**

- `formancehq/ledger`: GitHub's API reports **no licence at all** (`license: null`). There is **no LICENCE file at the repository root**, and the README carries **no licence statement** as of 2026-10-29. That is the exact state of the record: not "MIT", not "Apache", not "source-available" -- undeclared.
- Its sibling repositories `formancehq/payments` and `formancehq/stack` **do** carry a split licence: **MIT** for the community edition, with a separate enterprise licence applying to anything under an `ee/` directory, or cloud-subscription terms where the adopter has a Formance account. That is textbook **open core**.
- Last push 2026-10-29T21:02Z; latest release v2.4.17 (2026-10-28).

**What this means.** For `formancehq/ledger` the honest engineering answer is: the code is publicly readable, and the terms under which it may be used, modified, and served are **not stated in the repository**. A permissive default is a legal assumption, not a fact; an MIT inference from the sibling repositories is exactly the reasoning that fails in an audit. Until the vendor states terms in the repository, a bank treats this as "source-available under terms to be negotiated", and Sec. 7.6's rule -- read the licence file, dated -- produces no file to read, which is itself the finding.

**The open-core illustration.** The sibling split is the clearest worked example in this guide of why "is it open source?" is the wrong question. The same vendor, the same week, ships a MIT community edition and an enterprise edition whose terms live in a directory boundary (`ee/`). The licence is not a property of the company; it is a property of each path in each repository, and it can change as the product packaging changes.

### 6.6 The Layer Table

| Project | Repository | Licence (read 2026-10-29) | Last push (2026-10-29 read) | Latest release | Billing or ledger? |
|---|---|---|---|---|---|
| Kill Bill | `killbill/killbill` | Apache-2.0 (root LICENSE confirmed) | 2026-10-28T23:42Z | killbill-0.24.22 (2026-09-29) | Billing, with invoices and payments |
| Lago | `getlago/lago` | AGPL-3.0 (root LICENSE confirmed) | 2026-10-29T20:37Z | v1.55.0 (2026-10-07) | Usage-based billing and metering |
| Blnk | `blnkfinance/blnk` | Detected Apache-2.0; **root LICENSE not resolvable** (404) | 2026-10-24T03:16Z | v0.15.7 (2026-10-24) | Ledger-oriented, payments-adjacent |
| Formance | `formancehq/ledger` | **None declared** (`license: null`, no root file, no README statement) | 2026-10-29T21:02Z | v2.4.17 (2026-10-28) | Ledger-first, programmable |

All four are new material in this repository: Kill Bill, Lago and Blnk had zero prior mentions; Formance had three lines in an unrelated vendor-testimonial table in [Antithesis Guide](../technology/antithesis_guide.md) Sec. 14. None of the four is a substitute for a bank's core posting engine, and for three of the four the licence file is either a live constraint (AGPL) or a live gap (missing or undeclared). That is not a coincidence: this is the layer where vendors experiment with licensing precisely because the software is close enough to the money to be worth paying for.

## 7. The Licence Question

This repository has never had a licence analysis of payment software. It has licence *facts* scattered through project guides -- Apache-2.0 for Fineract, Apache-2.0 for Mojaloop, AGPL-3.0 for OBP -- but no place where the licence question is treated as the decision it is. This section is that place.

### 7.1 The Four Families

**Permissive (Apache-2.0, MIT).** The code may be used, modified, and redistributed with minimal conditions: keep the notice, and in Apache-2.0's case state changes and carry the patent grant. There is no obligation to publish your modifications. Apache-2.0 adds an express patent licence from contributors and a patent retaliation clause; MIT is shorter and says nothing about patents or trademarks.

**Copyleft (GPL-3.0, AGPL-3.0).** If you distribute a modified version, you must license the whole derivative under the same terms and make the source available. AGPL-3.0 extends this to software **offered to users over a network**: the obligation can be triggered by *service delivery*, not only by shipping binaries. This is the single most consequential sentence in this section for a bank, because it means the trigger is a business model rather than a distribution event.

**Source-available (SSPL, BUSL).** The source is published, but the terms are not open source by the OSI definition: SSPL adds a service-provisioning condition designed to reach the operator's whole stack, and BUSL (Business Source License) grants broad use now and converts to an open licence after a change date, with restrictions in the interim (typically on competing production use). Both are drafted per vendor and neither should be read as "open source with a caveat".

**Open core.** Not a licence but a packaging model: a permissively or copyleft-licensed community edition, plus enterprise features under separate terms -- commonly demarcated by a directory (`ee/`), by a separate repository, or by cloud subscription. Formance's sibling repositories are the example in this guide (Sec. 6.5). The licence question then becomes "which licence applies to which path", and it can change with a release.

### 7.2 The Family-by-Family Table

| Family | Modify privately? | Run internally? | Offer as a service to third parties? | Must publish modifications? | Patent grant | Trademark | Payments example from this guide |
|---|---|---|---|---|---|---|---|
| **Apache-2.0** | Yes | Yes | Yes, with notice obligations | No | Yes, express, with retaliation clause | No trademark rights granted | Hyperswitch, Kill Bill, Fineract, Mojaloop (`central-ledger`), Rafiki, Firefly, Fabric |
| **MIT** | Yes | Yes | Yes | No | Not addressed | Not addressed | BTCPay Server |
| **GPL-3.0** | Yes | Yes | Yes, but distribution of a derivative triggers source disclosure | Yes, on distribution | Yes, express | No trademark rights granted | None among the projects measured here -- but a GPL-3.0 dependency inside any of them would apply |
| **AGPL-3.0** | Yes | Yes (internal use is not "offering over a network" to third parties) | **Only with source disclosure of the modified work to service users** -- this is the constraint | Yes, on network service | Yes, express | No trademark rights granted | Lago, OpenBankProject (dual-licensed with a commercial option from TESOBE) |
| **SSPL / BUSL** | Yes | Yes | Usually restricted, and the terms are vendor-drafted | Per terms; BUSL typically restricts competing production use | Not a standard open-source patent grant | Per terms | None measured in this guide -- but the relicense risk in Sec. 8.3 is exactly this |
| **Open core (split)** | Per path: the community path yes, the `ee/` path per enterprise terms | Per path | Per path -- the enterprise path usually requires a subscription | Per path | Per path | Per terms | Formance (`formancehq/payments`, `formancehq/stack`) |

Read the AGPL row against the permissive rows and the commercial consequence is immediate: an Apache-2.0 stack imposes essentially no constraints on what a bank builds on top of it, and an AGPL stack imposes a constraint that attaches to the *service*, not to the code. That is why the two questions below are the ones counsel asks.

### 7.3 The Two Questions Legal Actually Asks

Every licence review in a bank reduces, eventually, to two questions. Everything else is elaboration.

**Question 1: can we modify it and keep the modification private?**
- Apache-2.0, MIT: yes, unconditionally. You must preserve notices (and in Apache-2.0, state that you changed files), and you publish nothing.
- GPL-3.0, AGPL-3.0: yes, **as long as you do not distribute it and do not offer it as a network service to third parties**. Internal modification and internal use are fine. The moment the modified work leaves the perimeter -- as a distributed product, or as a service to third parties -- the obligation to provide the corresponding source attaches.
- Source-available: read the terms. "Keep it private" is usually true for internal use and false for competing production use.
- Open core: true for the community path, false for the `ee/` path.

**Question 2: can we run it as a service -- for our customers, or as a utility for other banks?**
- Apache-2.0, MIT: yes. This is the family that lets a bank, or an SI on its behalf, operate the software as a service without a source-disclosure obligation.
- AGPL-3.0: **this is the question that decides AGPL deals.** Offering the modified software as a service to third parties brings the network-service obligation into play. Note the two boundaries carefully: (a) *modified* is doing real work in that sentence -- an unmodified deployment is a different analysis; (b) *third parties* is doing real work too -- internal service to your own legal entity and its employees is not the same as a white-label service to other institutions. Both boundaries are for counsel, not for an architect, and the practical instruction is: **if the deployment model is "we host it for others", assume AGPL is a commercial negotiation until proven otherwise.**
- Source-available and open core: assume the service question is the one the vendor's terms were written to answer, and negotiate.

The reason these two questions are the right ones is that they map onto the two deployment models banks actually use: run it for ourselves, or run it as a utility. Almost no bank asks a third question.

### 7.4 Why Licence Matters More in Payments

Licence matters in every software category. In payments it matters more, for four specific reasons.

1. **The code touches money movement.** A defect in a billing engine produces a wrong invoice. A defect in a posting engine produces an unbalanced book. A licence obligation that forces disclosure of a modified payment component does not expose a marketing site -- it exposes the logic by which the institution decides where money goes.
2. **The code sits inside PCI scope and audit scope.** Everything in the card path is examined by a QSA and by internal audit. "Who owns this code, under what terms, and can we show the licence file to an auditor" is a question that arrives with a deadline attached (Sec. 11.5).
3. **It is often forked by an SI who then sells hosting.** The most common commercial pattern in this space is: a systems integrator takes an open-source payment component, customises it for a bank, and offers to host it for that bank and then for others. If the component is AGPL, the SI's hosting business is the exact scenario the licence was drafted for. The bank inherits the dispute even when the bank did nothing wrong, because the bank is the one operating the service.
4. **A copyleft obligation can be triggered by a hosted white-label service rather than by distribution.** This is the point that traditional software licence review misses. A bank's legal team is trained to think about distribution -- shipping software to a third party. In payments, the more common exposure is a service: a white-label acquiring front end, a billing utility for other institutions, an API facade offered to partners. Those are service deliveries, and under AGPL they are the trigger.

### 7.5 The Live Cases a Bank Will Meet

| Case | Project | What it means in practice |
|---|---|---|
| **AGPL, actively maintained, strategically useful** | Lago (`getlago/lago`, AGPL-3.0, last push 2026-10-29T20:37Z) | Internal metered billing: fine, no disclosure obligation. Metered billing offered as a service to other institutions: counsel review, and probably a commercial licence or a clean-room boundary around the modifications |
| **AGPL with a commercial option** | OpenBankProject (AGPL-3.0 from TESOBE, with a commercial licence available -- [OpenBankProject Guide](openbankproject_guide.md) Sec. 7.1) | The dual licence *is* the answer to question 2: a bank that wants to offer OBP-based services without the AGPL analysis buys the commercial licence. Budget for it; do not discover it late |
| **Permissive default** | Hyperswitch, Kill Bill, Fineract, Mojaloop (`central-ledger`) -- all Apache-2.0, all read 2026-10-29 | The two questions both answer "yes" with only notice obligations. This is why the permissive projects are the ones a bank can build a product on without a licence negotiation |
| **Same vendor, different terms per directory** | Formance (`formancehq/ledger` undeclared; siblings MIT + `ee/` enterprise terms) | The licence is a property of a path, not of a company. Any architecture that spans the community and enterprise paths needs a per-path licence map, reviewed at every upgrade |

The pattern to carry into a procurement: **the licence question is a deployment-model question.** A bank that only ever runs software for itself can adopt almost anything, including AGPL, subject to notice obligations. A bank that intends to offer the capability to others -- and many banks do, through white-label arrangements with subsidiaries, partners, or other institutions -- has a materially narrower set of options, and the narrowing is invisible until someone reads the licence.

### 7.6 The Trap: Metadata Is Not the Licence

Two measurements from 2026-10-29 make the trap concrete.

- **`mojaloop/central-ledger`** carries Apache-2.0 via a `LICENSE.md` whose preamble reads "Copyright (c) 2020-2025 Mojaloop Foundation ... Apache License, Version 2.0". GitHub's licence detector classifies the repository as **"Other"**, because the custom preamble does not match its template. The licence is Apache-2.0; the metadata says "Other". A due-diligence process that reads the API and stops has recorded a false finding about a foundation-governed project.
- **`formancehq/ledger`** reports `license: null`, has no root licence file, and says nothing in its README. The metadata is accurate here, and the finding is the absence itself. Meanwhile the sibling repositories carry a split MIT/enterprise licence, so an inference from the family would produce a confident, wrong answer.

The rule that follows is short and non-negotiable: **open the licence file at the canonical repository, read it, record the date you read it, and record the commit or branch you read it at.** Where there is no file, record that -- "no licence file found at branch `main` on 2026-10-29" is a finding an auditor can act on, and "GitHub says Apache-2.0" is not. This is the same discipline Sec. 10.6 applies to health signals, and for the same reason: the metadata is a convenience for humans browsing, not evidence.

---

## 8. The Governance Question

Licence tells you what you may do with the code today. Governance tells you who decides what the code becomes, and whether anyone will still be maintaining it when your auditor asks in three years. For a bank, governance is the longer-lived risk of the two.

### 8.1 Foundation-Governed

**Apache Fineract** is an Apache Software Foundation project, governed by a project management committee under the ASF's process: meritocratic committer status, ASF ownership of the trademark, a documented release process, and the ASF's licence and provenance discipline. The practical consequences for a bank are that no single company can relicense the code, the project cannot be sold, and the roadmap is set in public. The costs are equally real: ASF projects are volunteer-and-employer funded, feature delivery is slower than a vendor's, and there is no one contractually obliged to fix anything (Sec. 8.5). The Fineract guide covers the governance and community model directly ([Apache Fineract Guide](apache_fineract_guide.md) Sec. 1.3).

**Mojaloop** is governed by the Mojaloop Foundation, a nonprofit that took over stewardship from the Gates Foundation's Level One Project lineage, with sponsor members funding the work and the Foundation owning the trademark and the specification. The governance history -- who founded it, which organisations sponsored it, how maintainership is distributed -- is covered in [Mojaloop Guide](mojaloop_guide.md) Sec. 1.2 and Sec. 20 and is not re-derived here. The point that matters for this guide is structural: a foundation that owns the specification *and* the trademark is the strongest available protection against the failure mode in Sec. 8.3, because a fork can take the code but not the name, and a scheme that certifies against "Mojaloop" is certifying against a specification a foundation controls.

Foundation governance is not a guarantee of health. It is a guarantee about *control*: the project cannot be relicensed out from under you, and it cannot be discontinued by a company decision. Whether it is well maintained is a separate, measurable question (Sec. 10.6).

### 8.2 Single-Vendor

Most of the projects in this guide are single-vendor, and the guide says so plainly rather than treating "open source" as a governance statement:

| Project | Company behind it | What the company controls |
|---|---|---|
| Hyperswitch | Juspay | Roadmap, release cadence, the majority of commits in practice |
| Lago | Lago (the company) | Roadmap, hosted offering, the AGPL/commercial packaging decision |
| Blnk | Blnk | Roadmap, and the licence file itself, which is currently not resolvable (Sec. 6.4) |
| Formance | Formance | Roadmap, and the boundary between the MIT community path and the `ee/` enterprise path |
| OpenBankProject | TESOBE GmbH, Berlin | The codebase, the trademark, and the commercial licence that sits beside the AGPL ([OpenBankProject Guide](openbankproject_guide.md) Sec. 1.2, Sec. 7.1) |

Single-vendor projects are not worse software. They are frequently better software than foundation projects, because a company with revenue ships faster than a volunteer committee. But the risk profile is different, and a bank must price it: the company's commercial interest, its funding, and its willingness to keep the permissive path permissive are all outside the bank's control and all inside the bank's dependency.

### 8.3 The Four Risks Governance Decides

**Abandoned-project risk.** The project stops. Commits slow, releases stop, security issues go unfixed, and the bank discovers it during an audit rather than during procurement (Sec. 12). Foundation governance reduces the probability of a company deciding to stop; it does not eliminate dormancy. The only defence is measurement, repeated: last push, release cadence, issue responsiveness, security advisories, dated (Sec. 10.6).

**Fork risk.** Who can fork, who owns the trademark, and whether a fork can be certified. Anyone can fork Apache-2.0 or AGPL code -- that is what the licence gives you. What a fork cannot take is the name (trademarks are not licensed by Apache-2.0, and a foundation owns its mark) and, in a scheme context, the certification: if a scheme certifies implementations against a specification the foundation controls, a fork is a new implementation that must be certified on its own. So a fork is a real exit but not a cheap one, and the cost is measured in re-certification, not in engineering.

**Contributor concentration.** A health signal a bank should measure rather than assume: how many organisations contribute meaningfully, and what share of recent commits comes from one employer. A project where one company writes the overwhelming majority of the code is a single-vendor project wearing a community badge, whatever its governance documents say. The measurement is available on the repository (commit history by author and affiliation); the judgement is the bank's.

**Relicence risk.** The licence you adopted is not necessarily the licence you will have. The pattern is well established in infrastructure software: a permissive project becomes source-available or open-core when the company needs revenue, and the change applies to future releases. For a bank, the exposure is precise: you built on Apache-2.0, the next version ships under BUSL or SSPL, and you now face a choice between forking the last permissive release (and maintaining it yourself) and accepting terms you did not price. This is why Sec. 8.4's licence-change protection is a contractual item and not a legal nicety.

### 8.4 What a Bank Should Require Contractually

No open-source licence contains a single obligation on the maintainer. Everything a bank needs beyond the licence has to be bought, negotiated, or built. For any component the bank depends on in a payment-critical path, the following should be on the table before go-live:

| Requirement | What it protects against | Realistic form |
|---|---|---|
| **Source-availability covenant or escrow** | The project's repository disappearing, or a licence change to terms the bank cannot accept | A support contract with a source escrow, or an internal mirror plus a written commitment to maintain it |
| **Security-patch SLA** | A CVE with no fix, in the authorisation path | Named severity tiers with fix windows, from a commercial supporter or an SI; a foundation cannot sign one |
| **Named maintainer commitment** | The key person leaving | Named engineers with contractual response obligations, or an SI retainer that names them |
| **Licence-change protection** | Relicensing after the bank has built on it | A contractual commitment that the version the bank runs stays under its current licence, plus a fork-and-maintain plan |
| **Indemnity** | IP claims against the bank | Vendor or SI indemnity; Apache-2.0 gives no indemnity, and neither does any other OSS licence |
| **Exit plan with data and format portability** | Being unable to leave | Documented data model, export formats, and a rehearsed migration -- tested, not asserted |

Two observations about this table. First, none of it is unusual: it is the same set of demands a bank makes of any software supplier, and the open-source suppliers and SIs in this market expect them. Second, **the bank is buying the same things it would buy from a commercial vendor, minus the licence fee** -- which is the honest shape of the open-vs-commercial comparison in Sec. 9.

### 8.5 No Licence Gives You an SLA

The sentence needs to be stated without decoration, because it is the most commonly skipped fact in open-source business cases: **no open-source licence gives you an SLA.** Apache-2.0 is distributed "AS IS", without warranties or conditions of any kind. So is MIT. So is AGPL-3.0. The licence grants permissions; it does not create obligations on the part of anyone to fix, respond, or continue.

Everything that looks like an SLA in the open-source world is a separate commercial instrument: a support contract from the vendor, a retainer with an SI, or the bank's own staff. That is not a defect of open source; it is what open source is. But it does mean that the correct comparison in a business case is not "vendor licence plus vendor SLA" versus "zero", it is "vendor licence plus vendor SLA" versus "support contract or SI retainer plus the platform team", and the second number is not small (Sec. 9.2).

## 9. The Commercial Comparison

### 9.1 What You Actually Pay For

The reason open-vs-commercial comparisons go wrong is that they compare a licence fee to nothing. A commercial payment platform's price covers several distinct things, and an open-source stack only removes one of them.

| What the money buys | In a commercial deal | In an open-source stack |
|---|---|---|
| **Licence** | Recurring fee, usually sized to volume, seats, or institutions | Zero. This is the only line that genuinely disappears |
| **Implementation** | Vendor professional services and certified partner implementation; accelerators for scheme connectivity | SI or in-house; no accelerators unless the SI brings its own |
| **Support and SLA** | Tiered support with response and resolution targets, escalation paths, named contacts | Community, a support contract from the project's vendor where one exists, or an SI retainer |
| **Scheme certification support** | The vendor has done the certification before and brings the evidence pack | The bank and its SI run certification themselves; conformance tooling may help (Sec. 2.2) |
| **Compliance packs** | Audit evidence templates, control mappings, PCI documentation, regulator-facing material | Adopter-built from scratch, every time |
| **Roadmap influence** | Contractual, or at least a customer council | Public issue tracker, at best |

The last three rows are where the real money is in a regulated institution, and they are the rows an open-source business case most often omits.

### 9.2 The Honest TCO View

**Licence-free is not cost-free.** The honest comparison is total cost of ownership across the platform's life (method in [FinOps Guide](../technology/finops_guide.md); the dated-numbers treatment of the same question for cores is in [Apache Fineract Guide](apache_fineract_guide.md) Sec. 15).

- **What a commercial deal bundles:** the vendor's engineering organisation, its certification history, its compliance documentation, its 24x7 support, and its contractual assumption of the *maintenance* obligation. The bank pays for that bundle whether or not it uses every part.
- **What an open-source stack moves to the bank:** the platform team (integration, upgrade, patching, capacity), the certification work, the audit evidence, the incident response, and the maintainer risk if the community or vendor stalls. None of it appears on a licence line, and all of it appears in a headcount plan.
- **The structural result:** for a large bank with an existing platform engineering function, an open-source orchestrator or ledger can be genuinely cheaper and strategically better, because the bank already pays for most of the moved costs. For a smaller institution with no platform team, the same stack is usually more expensive than the licence it replaces -- and the business case that says otherwise has priced the software and not the organisation.
- **The estimate this guide will not make:** exact TCO figures. They depend on volume, scheme, staffing market and scope, and any number quoted here would be a fabrication. The *shape* of the comparison is the deliverable; the numbers are the bank's own exercise, and they should be flagged as estimates inside the business case (Sec. 15.2).

### 9.3 The Dimension Table

| Dimension | Open source | Commercial |
|---|---|---|
| Licence cost | Zero | Recurring, volume- or seat-linked |
| Implementation | SI or in-house; adopter owns the build | Vendor plus certified partner; accelerators included |
| TCO | Lower licence, higher platform and compliance cost; favourable only with an existing platform team | Higher licence, lower marginal build; predictable and contractually governed |
| Functional depth | Deep in the category it targets; shallow everywhere else -- an orchestrator is not a ledger | Broad by design, because the vendor sells a suite |
| Support | Purchasable, never bundled; no SLA from the licence (Sec. 8.5) | Bundled with tiered SLAs |
| Compliance readiness | Adopter-built evidence; transparency of logic is a genuine offset | Control mappings, audit packs, certification history |
| Roadmap | Community or single-vendor; can stall | Vendor-funded, contractual influence possible |
| Lock-in | None legally; integration effort is the real lock-in, and the bank owns it | Contractual and technical; exit rights are negotiated, not automatic |
| Risk bearer | The bank, entirely | The vendor carries reputational and contractual weight, not legal liability |
| Best fit | Banks with platform engineering, a clear single category, and a strong exit plan | Banks buying a suite, needing certification support, or without a platform team |

### 9.4 Where Open Source Genuinely Wins

Four wins are real, and they are the reason this guide is not simply a restatement of the insurance guide's conclusion.

1. **It removes per-transaction and per-seat licence fees.** In payments, commercial pricing is often volume-linked, which means the fee grows with the bank's success. Removing a marginal cost per transaction changes the economics of high-volume, low-value products -- exactly the products inclusive-finance and embedded-payment strategies depend on.
2. **It removes the integration lock-in that PSPs create.** An orchestrator the bank operates makes the acquiring relationship commercial and switchable (Sec. 5.2). This is a competitive-position gain, not a cost gain, and it is the strongest argument in this guide.
3. **The logic becomes auditable end to end.** Routing rules, ledger postings, and API consent mapping can be reviewed as code by internal audit, by a regulator, or by the bank's own model-validation function. No commercial vendor offers that depth of inspection, and in a dispute about why a payment went where it went, it is decisive.
4. **Self-hosting keeps card data inside the perimeter, and keeps the deployment sovereign.** For a bank with data-residency obligations or a strategic preference for in-perimeter processing, an open-source orchestrator is one of the few ways to accept cards without the card data living in a third party's estate -- at the cost of a larger scope (Sec. 11.2).

### 9.5 Where It Does Not

1. **Support.** There is no SLA in any licence, and a foundation cannot sign one (Sec. 8.5). The support line moves; it does not vanish.
2. **Scheme certification.** No open-source project certifies the bank against a scheme. Conformance tooling makes the exam cheaper; the exam is still the scheme's (Sec. 2.2).
3. **Specialist skills.** Running a payment-critical service needs people who have done it: on-call engineers who understand authorisation flows, reconciliation analysts, scheme specialists. That market is thin, and the bank competes for it with the PSPs it is trying to disintermediate.
4. **The acquiring relationship.** No open-source project hands a bank a sponsor, a BIN range, or a settlement account. This is the thesis, and it is the one limitation that no amount of engineering removes.

---

## 10. The Selection Guidance

### 10.1 Test A: Which Category Does My Problem Belong To

The first test is the cheapest and prevents the most expensive mistakes. Walk the symptom down this tree before opening a single repository:

```
Symptom
  |
  +-- "we cannot accept a payment method our customers want"
  |      -> orchestration (if we have acquiring) / sponsor bank (if we do not)
  |
  +-- "our money movement depends on one PSP and we cannot switch"
  |      -> orchestration
  |
  +-- "we need to send/receive payments with other institutions in a country"
  |      -> rail, and therefore scheme membership first, software second
  |
  +-- "our book of record cannot support a new product"
  |      -> core banking (or a bounded subledger, if the product is narrow)
  |
  +-- "we cannot tell what a customer owes, or why the invoice is wrong"
  |      -> billing
  |
  +-- "our balances do not reconcile and we cannot explain a posting"
  |      -> ledger, and probably the core posting engine -- not a startup ledger
  |
  +-- "partners need our account data and payment initiation under consent"
  |      -> open-banking API layer
```

The tree's purpose is to stop a category-2 or category-4 project being bought for a category-1 or category-3 problem, which is the failure mode in Sec. 12 and Sec. 3.7.

### 10.2 Test B: Can We Operate a Payment-Critical Service

Open source moves operations to the bank. Before adopting anything in the authorisation or posting path, answer honestly:

- **Staffing:** do we have named engineers for this component, with a second line, or is it one person's side project?
- **On-call:** is there a rota that covers 24x7, with escalation paths to the acquirer, the scheme, and the core team?
- **Change control:** can we deploy this component without a change advisory board cycle measured in weeks, and do we have a tested rollback for a bad routing rule?
- **Incident process:** have we run an exercise where this component declines every transaction, and do we know what we would do in the first fifteen minutes?
- **Observability:** can we answer "how many transactions failed in the last hour, by processor, and why" without asking a vendor?

Four or more "no" answers means the bank is not ready to operate the component, whatever the licence says. The honest alternative is a managed service or a commercial platform until the capability exists.

### 10.3 Test C: Does the Licence Permit the Deployment

Apply Sec. 7.3's two questions to the *intended* deployment, not to the pilot:

1. Will we modify it, and do we need those modifications private? (Almost always yes to the first, and the answer to the second is what rules out some families in some models.)
2. Will we offer the resulting service to third parties -- customers, subsidiaries, partner banks, white-label clients? If yes, AGPL and source-available components move from "engineering dependency" to "commercial negotiation".

Then check the paths, not the company: an open-core project can have a permissive community path and a restrictive enterprise path, and the architecture decides which one the bank is on (Sec. 6.5, Sec. 7.5).

### 10.4 Test D: What Is the Exit Path

For every component, write down the exit before go-live, and make it specific:

- **Data:** what is the export format, is it documented, and have we run an export into a test environment?
- **Logic:** if routing rules or posting logic are configuration, can we read them out? If they are code, do we have the source and the build?
- **Contract:** what notice do we owe the vendor or SI, and what do they owe us?
- **Fork:** if the project is abandoned or relicensed, can we maintain the version we run, and for how long?
- **Re-certification:** if we had to change the component, what scheme certification would we have to redo, and how long does that take?

A component whose exit cannot be exercised is a lock-in regardless of its licence. "No lock-in" is a legal fact about the code, not a migration-free guarantee.

### 10.5 Test E: Who Is Accountable

The last test is not technical at all. For each component, name the person and the institution accountable when it fails: the accountable executive for the payment service, the licence holder for the regulated activity, the scheme member of record, and the owner of the settlement account. If any of those names is a vendor, the design is wrong -- not because vendors are untrustworthy, but because accountability in a regulated activity cannot be outsourced by contract.

### 10.6 The OSS Due-Diligence Checklist

A checklist a bank can actually run, with the evidence each item produces. Every item is a dated artefact, because an undated finding is an opinion.

| # | Check | Evidence to record |
|---|---|---|
| 1 | **Licence file read** | The file, the repository, the branch or commit, and the date read. "GitHub says Apache-2.0" is not evidence (Sec. 7.6) |
| 2 | **Last push measured** | The timestamp from the API, dated. Where a project's health is unclear, say so rather than inferring |
| 3 | **Release cadence measured** | Release list with dates. Note projects that push code but publish no releases -- OBP has tags only, no GitHub releases (Sec. 4.3) |
| 4 | **Right repository identified** | Docs/onboarding repositories go quiet while code repositories stay active (Sec. 4.2). Sample the code |
| 5 | **Contributor concentration** | Commit share by employer over the last 12 months; one dominant employer means single-vendor in practice (Sec. 8.3) |
| 6 | **Security disclosure policy** | A published policy, a security contact, and a history of advisories with fixes |
| 7 | **Release signing and provenance** | Signed tags or artefacts, checksums, and a documented build |
| 8 | **Path to commercial support** | A named vendor or SI who will sign an SLA, with named engineers (Sec. 8.4) |
| 9 | **Licence-change history** | Has the project relicensed before? A permissive-to-source-available history is a forecast |
| 10 | **Data and format exit** | An export that has been run, not a promise that one exists |

---

## 11. The Banking and Regulated Angle

### 11.1 Who Carries the Regulated Activity

The regulated activity is carried by the licensed institution, always, and no software decision changes that. A bank that runs an open-source rail participant, an open-source orchestrator, and an open-source ledger is still the entity that holds the licence, joins the scheme, holds the settlement account, and answers to the regulator for every payment those systems touch.

The sentence that matters, and that should be in every architecture decision record on this subject: **"we used open source" changes nothing about who is accountable to the regulator.** It changes who writes the code, who patches it, and who is on call. It does not change the accountable executive, the licence, the scheme membership, or the audit obligation.

### 11.2 PCI DSS: The Shape of Scope, Not the Size of Blame

Self-hosting an orchestrator changes the *shape* of PCI scope without shrinking accountability, and both halves of that sentence need to be in the business case.

- **What changes:** where cardholder data lives and flows. If the bank operates the orchestrator, the card data passes through systems the bank controls, which brings those systems, their configuration, their logging, and their access controls into scope. A bank that previously outsourced card handling to a PSP had a smaller scope and a larger dependency; self-hosting trades one for the other.
- **What does not change:** the obligation to be assessed against PCI DSS, the need for a QSA, the segmentation work, or the fact that a breach is the bank's breach. No licence, and no architecture, removes the attestation requirement.
- **The architectural consequence:** scope minimisation becomes an explicit design goal -- tokenise early, segment hard, keep the orchestrator out of the card-data environment where possible, and document the flows so the QSA can follow them. The orchestrator's tokenisation adjacency (Sec. 5.1) is where this design work happens.

### 11.3 Scheme Rules and Certification

A switch or an orchestrator does not make the bank a scheme participant. Scheme rules are a separate body of obligations: membership criteria, certification of the implementation, message standards, dispute and chargeback regimes, settlement obligations, and operational requirements such as availability and incident reporting. Software can implement a scheme's protocol; it cannot grant participation, and it cannot certify.

The practical test: **can the bank point to a certificate, a membership agreement, and a settlement account?** If any of the three is missing, the software is not the constraint. This is why Sec. 10.1's tree routes "we need to move money between two banks in a country" to scheme membership before it routes to a repository.

### 11.4 Outsourcing and Third-Party Risk

The model guide's MAS angle applies here almost unchanged, which is worth stating because it is the part banks most often think open source exempts them from ([Open-Source Insurance Systems and Commercial Systems](insurance_open_source_commercial_guide.md) Sec. 5.4).

- **Outsourcing frameworks still apply** -- to the SI, to the support vendor, and to the cloud provider. Open source does not create an unregulated zone; it relocates the counterparty.
- **Every OSS component is a supply-chain item.** Each dependency has an owner, a licence, a patch cadence, and a security posture, and a bank's third-party risk process should treat material components the way it treats material vendors. The practical manifestation is a software bill of materials with licence and version data, maintained and reviewed.
- **Data residency and access** are unaffected by the licence and affected by the hosting choice: self-hosting can improve sovereignty, at the price of operating the platform (Sec. 9.4).

### 11.5 Audit Evidence and Version Traceability

A payment system must be able to answer, for any transaction: which version of which component processed it, under which rules, and who changed what when.

- **Version traceability:** the exact release (and ideally commit) running in production, per component, recorded over time -- not just "current".
- **Rule traceability:** the routing rules, fee tables, and posting logic in force at the time of the transaction, with their change history.
- **Licence traceability:** the licence of each component at the version deployed, dated, so an auditor can see that the bank's use was permitted at the time.
- **Certification traceability:** which scheme certification covered the deployed version, and what changed since.

Open source helps with the first three -- the artefacts are the bank's own -- and does nothing for the fourth, which belongs to the scheme.

### 11.6 Settlement, Liquidity, and the Part No Software Supplies

The part of payments that no software supplies is the part where money actually moves: the settlement account, the funding behind it, and the liquidity management that keeps obligations met when they fall due, including intraday. A switch computes positions; a settlement bank or central bank discharges them. A ledger records a balance; the balance is only money if the account behind it is funded. This is the last mile of the thesis, and it is the mile that a software-led business case forgets, because it appears nowhere in a feature comparison.

---

## 12. The Failure Modes

### 12.1 The Failure-Mode Table

| Symptom | Cause | Guardrail |
|---|---|---|
| A billing engine is deployed as if it were a payment rail, and the bank discovers it cannot move money | Category confusion: an invoicing product was read as a payment system (Sec. 6.1) | Test A (Sec. 10.1): name the category before the vendor; require the product to state what it does not do |
| Legal discovers, after go-live, that an AGPL component is being offered as a hosted service to partner banks | Licence review done on the code, not on the deployment model (Sec. 7.3) | Test C (Sec. 10.3), answered for the intended service model, with counsel's sign-off recorded |
| A fork nobody can patch: the SI forked the project for the bank, upstream diverged, and security fixes stopped arriving | Fork taken without a maintenance plan; re-certification cost not priced (Sec. 8.3) | Contractual patch obligation on the SI, an upstream-tracking plan, and a fork budget approved before the fork |
| A project went quiet and nobody noticed until an audit asked for its security advisories | Health monitored at procurement only, never afterwards (Sec. 10.6) | Annual re-measurement of last push, releases, and advisories; recorded, dated |
| An orchestrator added a compliance obligation nobody scoped: card data is now inside the bank's perimeter and the QSA's scope doubled | Self-hosting benefit claimed without the scope consequence (Sec. 11.2) | PCI scope impact assessment as a gate in the architecture review, before the build |
| "Open source" was used to mean "free" in the business case, and the platform team was never costed | TCO compared licence fee to zero (Sec. 9.2) | Fully loaded TCO including platform staff, support contract, certification, and exit |

### 12.2 The Pattern Behind the Six

Five of the six failures share one root: **a decision was made about software when the constraint was institutional.** The billing-engine-as-rail, the unscoped card data, and the unpatched fork are all cases where the software was available and the institutional asset, obligation, or capability was not. The sixth -- the licence surprise -- is the same error in legal form: the licence was read as a property of the code when it is a property of the deployment.

The guardrail that catches all six is the same discipline this guide applies throughout: name the category, name the accountable institution, read the licence at the date of use, and measure the health of the thing you depend on. None of that is novel. What is novel, in most banks, is doing it for open-source payment software at all.

## 13. The Cymbal Bank Worked Example

*Cymbal Bank is a fictional institution used throughout this series for illustration. The projects named are real and their licence and repository facts are the dated ones measured in this guide; the bank, its scheme, its volumes, and its decisions are invented, and no claim is made that any real institution has made the same choices.*

### 13.1 The Scenario

Cymbal Bank is a mid-size bank with an existing card-acquiring relationship, a core banking platform it is not replacing, and a payments strategy with three asks on the table:

1. **A real-time account-to-account rail pilot**, where the national scheme is standing up an instant payment service and Cymbal wants to be an early participant.
2. **A reduction in PSP concentration**, because 80% of card volume goes through one acquirer and switching has been judged impossible twice.
3. **A new product line with prepaid balances**, which the core cannot represent without a change request measured in quarters.

The three asks look like one programme and are, in fact, three different categories. That is the point of the example.

### 13.2 Decision One: The Rail Pilot

**Category: rail. Institutional assets required: scheme membership, settlement account, liquidity.**

The scheme has chosen an open-source switch implementation -- Mojaloop-shaped, with the scheme operating the hub and participants connecting through the FSPIOP API ([Mojaloop Guide](mojaloop_guide.md) Sec. 5, Sec. 6). Cymbal's work is to become a participant: implement the participant-side API, pass the scheme's certification, fund its settlement position, and manage liquidity and limits ([Mojaloop Guide](mojaloop_guide.md) Sec. 11, Sec. 12).

**Why open source was right here:** Cymbal is not buying software, it is joining a scheme whose operator chose the software. The open-source choice was made above Cymbal's head and benefits Cymbal, because the scheme could afford to exist. Cymbal's cost is integration and certification -- and the scheme's conformance tooling makes the certification cheaper than a bespoke one would be (Sec. 2.2).

**What the thesis says about it:** nothing about the software made Cymbal a participant. The membership, the settlement account, and the liquidity are the scheme's and Cymbal's, and they were secured before a line of integration code was written. Had Cymbal tried to run the same software without membership, it would have had a laboratory, not a rail.

### 13.3 Decision Two: The Orchestrator

**Category: orchestration. Institutional assets required: nothing new -- Cymbal already has acquirers.**

Cymbal deploys Hyperswitch in front of its existing acquirers: the primary, a secondary for failover, and a local-methods provider. Channels integrate once, with the orchestrator. Routing rules move from the primary acquirer's configuration portal into Cymbal's own version-controlled configuration. Reporting is one surface. The acquirers become commercial relationships behind pluggable connectors.

**Licence position (measured 2026-10-29):** Apache-2.0, root `LICENSE` file confirmed, archived false, last push 2026-10-29T21:23Z, latest release v1.128.0 (2026-10-16). Both of Sec. 7.3's questions answer yes: Cymbal may modify privately and may run it as a service without a source-disclosure obligation, subject to the ordinary Apache-2.0 notice requirements.

**Why open source was right here:** this is the category where the thesis holds *and* the bank gains position. Cymbal's acquiring relationships already exist, so no institutional asset is missing; what was missing was control of the routing logic and the ability to switch. Open source supplied exactly that, and the licence imposes no constraint on the deployment model Cymbal intends.

**What Cymbal had to add, and cost:** a platform team for the orchestrator, an on-call rota covering the authorisation path, a PCI scope re-assessment (Sec. 11.2), a support contract with a named escalation path (Sec. 8.4), and a routing-rule governance process. The business case included all five; a case that had compared the acquirer's per-transaction fee to zero would have included none.

### 13.4 Decision Three: The Ledger That Was Rejected

**Category: ledger of record. Institutional assets required: the bank's own book, its audit relationship, its regulatory reporting.**

The prepaid product needs balances the core cannot represent. The proposal on the table was to adopt an open-source ledger -- Formance or Blnk -- as the balance-of-record for the product, and to treat the core as downstream.

**Why open source was wrong here** -- and it is worth being precise, because the software was not the problem:

- **The licence was not established.** `formancehq/ledger` reports no licence, has no root licence file, and states nothing in its README as of 2026-10-29 (Sec. 6.5). `blnkfinance/blnk` is detected as Apache-2.0 but no root licence file was resolvable at `main` in this pass (Sec. 6.4). Neither is a basis on which to make a book of record the bank must be able to defend to an auditor.
- **The accountability does not transfer.** A ledger is not just software; it is the artefact the bank's financial statements rest on. Adopting a startup's ledger as the balance of record puts the bank's audit evidence in a dependency with a single vendor, no escrow, no certification, and no regulatory track record. That is the thesis binding: the licence and the software were available; the regulated accountability, the settlement position, and the audit evidence were not.
- **The failure mode is the worst one.** A wrong invoice is correctable. An unbalanced book is a restatement.

**What Cymbal did instead:** a bounded subledger -- the open-source ledger records prepaid balances for one product, reconciles to the core daily, and the core remains the book of record. This is the pattern Sec. 3.5 recommends: the ledger is a subledger beside the core, not a replacement for the posting engine. The open-source component earned its place by being excellent at the narrow job, not by being asked to be the bank.

### 13.5 The Licence Review

Cymbal's review, run once across the three decisions, produced this record. Note what each row's evidence is -- a file, a date, a deployment model -- because that is what an auditor will ask for (Sec. 11.5).

| Component | Licence (read 2026-10-29) | Deployment model | Question 1: modify privately? | Question 2: service to third parties? | Outcome |
|---|---|---|---|---|---|
| Scheme switch (Mojaloop-shaped, scheme-operated) | Apache-2.0 via `LICENSE.md` with a Mojaloop Foundation preamble; GitHub detector says "Other" (Sec. 7.6) | Cymbal is a participant, not the operator | Yes | Yes (participant integration) | Proceed; recorded the detector discrepancy so audit does not re-litigate it |
| Hyperswitch | Apache-2.0, root LICENSE confirmed | Self-hosted, Cymbal's own customers | Yes | Yes | Proceed; notice obligations only |
| Formance ledger | **None declared** | Would have been book of record for a product | Not determinable | Not determinable | **Rejected as book of record**; considered only as a bounded subledger after vendor terms are stated |
| Blnk | Detected Apache-2.0, no root file resolvable | Would have been book of record for a product | Not determinable from a document | Not determinable from a document | **Not adopted**; licence document must be produced before any further evaluation |
| Lago (evaluated for metered fees, not adopted) | AGPL-3.0, root LICENSE confirmed | Internal metering only | Yes | No -- internal only | Acceptable for internal use; flagged that any future hosted offering reopens the analysis |

### 13.6 The Exit Paths

- **Rail:** exit is not a software question. Leaving a scheme is a membership decision with notice periods and settlement consequences; the software exit is trivial next to it. Recorded as a scheme risk, not a technology risk.
- **Orchestrator:** exit is real and was tested -- an export of routing rules and transaction history into a test environment, a documented connector interface, and a rehearsal of moving one channel to a second acquirer without touching channel code. The exit was the *reason* for the decision, so it had to be exercisable.
- **Subledger:** exit is a migration of one product's balances back to the core, which the core team confirmed it could absorb. Documented with the export format and a dry run.

### 13.7 The Two Places the Thesis Was Tested

**Tested and held: the rail.** Cymbal had the software available and still could not have become a rail participant without a membership, a settlement account, and funded liquidity. The software was necessary and was nowhere near sufficient. The bank's programme plan spent its first quarter on scheme onboarding, and that was correct.

**Tested and held, in the bank's favour: orchestration.** Cymbal had the institutional assets already -- acquirers, a licence, a PCI programme -- and the missing piece genuinely was software it could own. This is the case that makes open source strategically interesting in payments rather than merely cheap: where the institutional assets exist, open source converts a dependency into an owned capability.

**Tested and broken: the ledger of record.** The software was available, the licence was arguably available, and the decision was still wrong -- because the bank would have been outsourcing the one artefact it can never outsource, its own book. The rejection was not a judgement about code quality. It was the thesis, applied honestly, against the proposal that looked most modern.

The example's lesson is not "adopt open source" or "avoid open source". It is that the same bank, in the same programme, correctly reached three different answers in three categories, and the variable that decided each was not the software.

---

## 14. The One-Page Summary

- **The thesis.** Open source gives you the software. It does not give you the licence, the settlement account, or the scheme membership -- and those are the product (Sec. 1.3).
- **The decoder.** Ten words run through every payments decision and only two of them -- orchestrator and ledger implementation -- are ordinary software purchases; the rest are institutional grants (Sec. 1.2).
- **Five categories, one question each.** Rail moves value between institutions; core banking is the book of record; orchestration chooses and manages the path to processors; ledger and billing record what is owed and posted; the open-banking API layer exposes data under consent (Sec. 3).
- **The three owned projects, situated.** Fineract is a core ledger that made a bank-shaped institution possible without a licence fee, and did not make anyone a bank. Mojaloop is a switch a scheme operates, and adopting it does not make the adopter a scheme. OBP is a facade that holds no money, and its strategic importance is unrelated to its money-handling (Sec. 4).
- **Orchestration is where open source changes position, not just cost.** It moves PSP lock-in from the integration to the contract, keeps card data and routing logic in-house, and makes the routing auditable -- at the price of operating a payment-critical service (Sec. 5).
- **Ledger and billing are two jobs, and "ledger" is the abused word.** Billing rates and invoices; a ledger posts and balances. Only the second is a book of record, and none of the four projects measured here should be a bank's book of record (Sec. 6).
- **Licence is a deployment-model question.** Two questions decide it: can we modify privately, and can we run it as a service for others. Apache-2.0 answers yes to both; AGPL-3.0 answers yes to the first and "only with source disclosure" to the second (Sec. 7).
- **Metadata is not the licence.** GitHub called Mojaloop's Apache-2.0 `LICENSE.md` "Other", and reported nothing for Formance's ledger, which has no licence file at all. Read the file, at the canonical repository, and date what you read (Sec. 7.6).
- **Governance is the longer-lived risk.** Foundation-governed projects cannot be sold or relicensed out from under you, and cannot sign an SLA. Single-vendor projects ship faster and can relicense. No open-source licence gives you an SLA (Sec. 8).
- **Licence-free is not cost-free.** The licence line disappears; the platform team, the certification, the audit evidence, the support contract, and the exit plan do not (Sec. 9.2).
- **Selection is five tests.** Which category, can we operate it, does the licence permit the deployment, what is the exit, and who is accountable (Sec. 10).
- **The regulator is unmoved.** "We used open source" changes nothing about who is accountable, and no software supplies a settlement account or liquidity (Sec. 11).
- **The failures are all one failure.** A decision was made about software when the constraint was institutional (Sec. 12).
- **The worked example reaches three answers in one programme** -- adopt for the rail as a participant, adopt for orchestration, reject for the ledger of record -- and the deciding variable was never the code (Sec. 13).

---

## 15. Claims Audit, What Could Not Be Verified, Glossary and Closing

### 15.1 Claims Audit

Every licence, archive flag, last-push timestamp and release below was read on **2026-10-29** from the GitHub REST API (`/repos/{owner}/{repo}`) plus, where stated, the raw licence file at the canonical repository. Two status marks appear in the table below, and nowhere else in this guide: the **check mark** means verified in this research pass; the **warning triangle** means flagged (inference, absent document, single source, or not verifiable here).

| Claim | Status | Basis (dated) |
|---|---|---|
| Apache Fineract is Apache-2.0, not archived, default branch `develop`; last push 2026-10-29T20:58Z; latest release 1.16.1 (2026-10-26) | ✅ | GitHub REST API + licence, 2026-10-29 |
| `mojaloop/central-ledger` is Apache-2.0 via `LICENSE.md` with a "Copyright (c) 2020-2025 Mojaloop Foundation ... Apache License, Version 2.0" preamble; GitHub's detector classifies it "Other"; last push 2026-10-29T19:38Z; latest release v20.3.1 (2026-10-22) | ✅ | GitHub REST API + raw `LICENSE.md`, 2026-10-29 |
| `mojaloop/mojaloop` (onboarding/documentation repository) last pushed 2025-03-11 and is not a health signal for the code repositories | ✅ | GitHub REST API, 2026-10-29 |
| `OpenBankProject/OBP-API` is AGPL-3.0, not archived, default branch `develop`; last push 2026-10-29T08:01Z; **no GitHub releases**, tags v3.0.0 / v2.1.0 / v1.4.0 only | ✅ | GitHub REST API, 2026-10-29 |
| `juspay/hyperswitch` is Apache-2.0 (root LICENSE = Apache License 2.0), not archived; last push 2026-10-29T21:23Z; latest release v1.128.0 (2026-10-16) | ✅ | GitHub REST API + raw `LICENSE`, 2026-10-29 |
| `killbill/killbill` is Apache-2.0 (root LICENSE confirmed), not archived; last push 2026-10-28T23:42Z; latest release killbill-0.24.22 (2026-09-29) | ✅ | GitHub REST API + raw `LICENSE`, 2026-10-29 |
| `getlago/lago` is AGPL-3.0 (root LICENSE = GNU AGPL v3), not archived; last push 2026-10-29T20:37Z; latest release v1.55.0 (2026-10-07); canonical repo is `getlago/lago` (`lago-access/lago` returns 404) | ✅ | GitHub REST API + raw `LICENSE`, 2026-10-29 |
| `blnkfinance/blnk` is detected as Apache-2.0, not archived; last push 2026-10-24T03:16Z; latest release v0.15.7 (2026-10-24); **no root LICENSE file resolvable at branch `main`** (raw fetch 404) | ✅ detection / ⚠ document | GitHub REST API, 2026-10-29; licence document absent |
| `formancehq/ledger` reports **no licence** (`license: null`), has **no root licence file**, and its README carries **no licence statement** as of 2026-10-29; last push 2026-10-29T21:02Z; latest release v2.4.17 (2026-10-28) | ✅ (state of record) / ⚠ (usable terms unknown) | GitHub REST API + root listing + README, 2026-10-29 |
| `formancehq/payments` and `formancehq/stack` carry a split licence: MIT community edition, separate enterprise terms for `ee/` paths or cloud subscription | ✅ | GitHub REST API + repository licence files, 2026-10-29 |
| `interledger/rafiki` is Apache-2.0; last push 2026-10-29T18:06Z; latest release v2.4.2-beta (2026-10-29) | ✅ | GitHub REST API, 2026-10-29 |
| `interledger/open-payments` (protocol specification) is Apache-2.0; last push 2026-10-29T16:06Z | ✅ | GitHub REST API, 2026-10-29 |
| `btcpayserver/btcpayserver` is MIT; last push 2026-10-29T14:35Z; latest release v2.4.7 (2026-10-29) | ✅ | GitHub REST API, 2026-10-29 |
| `hyperledger-firefly/firefly` is Apache-2.0, last push 2026-10-23; `hyperledger/fabric` is Apache-2.0, last push 2026-10-26 | ✅ | GitHub REST API, 2026-10-29 |
| Repository measurement: 682 markdown files before this guide was added; `\bHyperswitch\b` 0 files, `\bKill ?Bill\b` 0, `\bLago\b` 0, `\bBlnk\b` 0, `\bFormance\b` 1 file (3 lines, vendor-testimonial table in [Antithesis Guide](../technology/antithesis_guide.md) Sec. 14); 13 files contain a paragraph with "open source", "licence" and "payment", all incidental | ✅ | Full-text scan of every markdown file in the repository, 2026-10-29 |
| Apache-2.0 grants an express patent licence with a retaliation clause and grants no trademark rights; MIT is silent on patents and trademarks; AGPL-3.0/GPL-3.0 grant a patent licence and no trademark rights; AGPL-3.0's network-service obligation attaches to offering the modified work to users over a network | ✅ | Licence texts as published by the respective stewards |
| No open-source licence provides a warranty or an SLA; Apache-2.0 and MIT distribute "AS IS" | ✅ | Apache-2.0 Secs. 7-8; MIT text |
| Fineract is governed by an Apache Software Foundation PMC; Mojaloop by the Mojaloop Foundation, which took over from the Gates Foundation Level One lineage | ✅ | Cross-ref [Apache Fineract Guide](apache_fineract_guide.md) Sec. 1.3; [Mojaloop Guide](mojaloop_guide.md) Sec. 1.2, Sec. 20 |
| Hyperswitch is a Juspay project; OBP is built and commercially licensed by TESOBE GmbH, Berlin | ✅ | Cross-ref [OpenBankProject Guide](openbankproject_guide.md) Sec. 1.2, Sec. 7.1; repository ownership |
| TCO shape (platform team, certification, support contract, exit) and the absence of vendor pricing figures | ⚠ | Structural analysis; no pricing or adoption figures are asserted anywhere in this guide, by design |

### 15.2 What Could Not Be Verified

1. **Blnk's licence document.** GitHub's detector reports Apache-2.0, but no root `LICENSE` file was resolvable at branch `main` on 2026-10-29 (raw fetch 404). The licence may exist elsewhere in the tree or on another branch. Until a human reads and dates the text, the licence is asserted by metadata, not verified by document, and this guide says so wherever Blnk appears (Sec. 6.4, Sec. 13.5).
2. **Formance's ledger licence.** `formancehq/ledger` has no declared licence, no root licence file, and no README statement as of 2026-10-29. The terms under which it may be used, modified, or served are therefore unknown from the repository. The MIT-plus-`ee/` split seen in the sibling repositories is evidence of the vendor's *usual* packaging, not a statement about this repository.
3. **Whether any named regulated bank runs Hyperswitch, Blnk, Formance or Lago in production.** Not verifiable in this pass, and not asserted. Vendor case studies and conference talks are not evidence of production use at a regulated institution, and this guide makes no adoption claims for any of them.
4. **Exact commercial TCO figures** for either side of the comparison. Volume, scheme, staffing market and scope drive them, and any figure here would be invented. Sec. 9.2 gives the shape of the comparison and flags the numbers as the bank's own exercise.
5. **Release cadence as a health signal for OBP.** The repository is pushed daily, but GitHub publishes no releases for it, only tags. Whether that reflects a deliberate release process elsewhere or an absence of one was not determined in this pass; the observation is recorded rather than interpreted (Sec. 4.3).
6. **Contributor concentration for any project in this guide.** Measureable from commit history, but not measured in this pass. Sec. 8.3 and Sec. 10.6 specify how a bank should measure it rather than asserting a result.

### 15.3 Glossary

- **Rail** -- the infrastructure and rules that carry a payment between institutions: message flows, clearing and settlement mechanism (RTGS, ACH, instant retail, card network).
- **Scheme** -- the rulebook and membership arrangement on a rail: who may participate, what messages are valid, how disputes and settlement work, what certification is required.
- **Sponsor bank** -- the licensed institution whose membership, BIN range, or settlement account a non-member uses to reach a scheme, taking the scheme and settlement risk.
- **Acquirer** -- the institution that acquires card transactions from a merchant, authorises them through the scheme, and settles to the merchant.
- **PSP** -- payment service provider: the contracted entity that processes payments, often bundling gateway, acquiring, tokenisation and reporting.
- **Orchestrator** -- software between the institution and several PSPs or acquirers: routing, retries, failover, unified reporting, tokenisation adjacency.
- **Smart routing** -- rule-based selection of the processor for each transaction by cost, success rate, currency, issuer geography, ticket size or BIN.
- **Ledger** -- the double-entry book of accounts and postings that records what is held and owed; a book of record when it is the institution's authoritative record.
- **Double-entry** -- the accounting method in which every transaction posts equal debits and credits, so the books balance by construction.
- **Journal** -- the ordered record of transactions before or as they are posted to accounts; the audit trail of what was recorded and when.
- **Posting** -- a single debit or credit entry against an account; the atomic unit a ledger must apply idempotently.
- **Idempotency** -- the property that repeating an operation produces the same result as performing it once; mandatory for retried postings and payment instructions.
- **Open banking** -- the regime and practice of exposing bank account data and payment initiation to regulated third parties under customer consent.
- **XS2A** -- "access to account", the PSD2 interface obligation; in practice the Berlin Group NextGenPSD2 API standard and its peers.
- **PSPI** -- payment service provider issuing payment instruments: the PSD2 category covering card issuers, as distinct from the ASPSI (account servicing PSP) that hosts the XS2A interface.
- **DFSP** -- digital financial service provider: the Mojaloop term for any participant in the switch, whether bank, mobile money operator or fintech.
- **FSPIOP** -- the Mojaloop interoperability API specification that participants implement to send and receive transfers through the switch.
- **Core banking** -- the system of record for customer accounts, products, interest, fees and the general ledger; the book of record for the bank's balance sheet.
- **PCI DSS** -- the card industry data security standard; an attestation obligation on any entity that stores, processes or transmits cardholder data.
- **BIN** -- bank identification number: the leading digits of a card number that identify the issuer and programme; BIN sponsorship is the right to transact under an acquirer's range.
- **Settlement account** -- the account at a central or commercial bank through which payment obligations are discharged in central-bank or commercial money.
- **Correspondent banking** -- the bilateral account and credit arrangement through which one bank settles or services flows for another, including across borders.
- **Copyleft** -- a licence family requiring derivative works to carry the same terms, with source disclosure on distribution (GPL) or on network service (AGPL).
- **AGPL** -- the GNU Affero GPL: copyleft extended so that offering the modified software to users over a network triggers the source-disclosure obligation.
- **Permissive** -- a licence family (Apache-2.0, MIT, BSD) allowing use, modification and redistribution with minimal conditions and no source-disclosure duty.
- **Source-available** -- published source under terms that are not open source by the OSI definition (SSPL, BUSL); use is permitted, but restricted, typically on competing production use.
- **Open core** -- a packaging model with a permissively or copyleft-licensed community edition plus paid enterprise features, often demarcated by an `ee/` directory.
- **Escrow** -- an arrangement placing source code or artefacts with a third party for release on defined triggers; the usual answer to abandoned-project risk.
- **Fork** -- an independent copy of a project taken by another party; the licence permits it, the trademark does not come with it, and scheme certification does not transfer.
- **Contributor concentration** -- the share of contributions from one employer or individual; a health signal that distinguishes a real community from a single-vendor project.
- **TCO** -- total cost of ownership over the platform's life: licence, implementation, integration, run, change, support, compliance and exit.

### 15.4 Cross-References and Closing

**Within this series:**

- [Apache Fineract Guide](apache_fineract_guide.md) -- the core-ledger deep-dive: capabilities Sec. 2, architecture Sec. 3, modules Sec. 4, API Sec. 6, accounting Sec. 9, deployment Sec. 11, commercial comparison Sec. 15, role in a bank Sec. 16, production considerations Sec. 17.
- [Mojaloop Guide](mojaloop_guide.md) -- the rail deep-dive: switch concept Sec. 2, FSPIOP API Sec. 5, hub architecture Sec. 6, settlement Sec. 11, liquidity Sec. 12, governance Sec. 20, commercial alternatives Sec. 21, banking context Sec. 22.
- [OpenBankProject Guide](openbankproject_guide.md) -- the API-facade deep-dive: API Sec. 2, PSD2 angle Sec. 4, open-source model Sec. 7, comparison Sec. 8.
- [Open-Source Insurance Systems and Commercial Systems](insurance_open_source_commercial_guide.md) -- the structural precedent: open-vs-commercial method Sec. 5, selection Sec. 7.3, verification discipline Sec. 10.
- [Payment Rails Guide](payment_rails_guide.md) -- the rail taxonomy Sec. 2, the real-time rails map Sec. 3, clearing and settlement Sec. 5.
- [Payments Hub Guide](payments_hub_guide.md) -- hub architecture Sec. 3, routing and orchestration Sec. 6, the Mifos Payment Hub EE open-source implementation Sec. 11.
- [Core Banking Systems Guide](core_banking_systems_guide.md) -- what a core is Sec. 1, the core in bank architecture Sec. 2, the vendor landscape Sec. 5, composable banking Sec. 6.
- [FinOps Guide](../technology/finops_guide.md) -- TCO method for Sec. 9.2.

**Closing.** The landscape in this guide is real: five categories, a dozen projects with dated licences and dated pushes, two layers -- orchestration and ledger -- that this repository had never examined, and a licence analysis it had never performed. What the landscape shows is consistent across every category, every licence family, and every governance model: the code is available, the accountability is not, and the difference between a bank that gains from open source and one that acquires a liability is whether it understood which of the two it was buying. The three projects this series already owned proved it in their own domains; the two new layers prove it where the money is closest. open source in payments gives you the software; it does not give you the licence, the settlement account, or the scheme membership - and those are the product.
