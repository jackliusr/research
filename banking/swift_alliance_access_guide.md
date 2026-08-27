# SWIFT Alliance Access: The SWIFT-Connectivity Deep-Dive — A Comprehensive Guide

**The SWIFT Connectivity Layer and the Alliance Family (Alliance Access, Alliance Gateway, Alliance Lite2), the Messaging (FIN MT, MX/ISO 20022, the STP flows), the Connectivity and the Security (SWIFTNet, the PKI/2FA, the RMA), the Deployment (on-prem vs cloud, the back-office integration), the Operations (message monitoring, queue management, gpi), the Banking Context (the Cymbal Bank SWIFT estate), a Cymbal Bank SWIFT-Connectivity Worked Example (an Alliance Access deployment + RMA setup + the MT103/pacs.008 and MT700 flows), and the Summary — the final word is "the secure lane"**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / SWIFT Connectivity — the SWIFT connectivity layer and the Alliance family positioning (Alliance Access — the messaging interface and the secure hub; Alliance Gateway — the file-transfer and hub interface; Alliance Lite2 — the cloud-based access), the Messaging (FIN MT, MX/ISO 20022, the straight-through-processing flows), the Connectivity and the Security (SWIFTNet, the PKI/2FA, the RMA — the counterparty access control), the Deployment (on-prem vs the Lite2 cloud, the back-office integration), the Operations (message monitoring, queue management, gpi), the Banking Context (the Cymbal Bank SWIFT estate), the Worked Example (a Cymbal Bank SWIFT-connectivity design — an Alliance Access deployment, an RMA setup, and the payments MT103/pacs.008 + trade MT700 message flows), the Summary — the final word is "the secure lane"
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the swift.com product pages — Alliance Access, Alliance Gateway, Alliance Lite2 / Alliance Cloud, SWIFTNet Link, the Hardware Security Module (HSM), Swift gpi (extracted 2026-08-27); the swift.com factsheets and information papers — "RMA and RMA Plus: managing your correspondent connections" (July 2016) and the FileAct messaging factsheet; the industry and partner documentation — Oracle (the SWIFTNet messaging services — InterAct, FileAct, Browse, FIN), Microsoft Learn (SWIFTNet Link as the mandatory network interface), IBM (the SWIFTNet definition and the SAG/RAHA/MQHA adapters), Axway (the SAG interface), AWS (the SWIFT Customer Security Programme whitepaper), the Wolfsberg Group (the RMA due-diligence guidance), Coupa (gpi January 2017 — press, flagged); and the repo's sibling guides (cross-referenced heavily — their verification ledgers are the authority for the SWIFT founding, the gpi launch, and the CBPR+ timeline). NOTE: this pass had **live web access** — the product-page and factsheet claims below were verified against swift.com on 2026-08-27; anything that could not be verified in this pass is flagged ⚠ honestly.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the banking cluster):** [Payment Rails](payment_rails_guide.md) (the SWIFT founding 3 May 1973, gpi January 2017, the MT103/MT202/MT700/MT940 classes — cross-ref heavily, do not re-derive), [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) (CBPR+, the UETR/gpi tracker, InterAct/FileAct, the Business Application Header, the 22 Nov 2025 end of MT coexistence — cross-ref heavily), [Capital Markets Architecture](capital_markets_architecture_guide.md) (§7.3 the post-trade language and "the bank's connectivity hub (SWIFT Alliance-class, or a vendor hub)" — the anchor this guide builds on), [Payments Hub](payments_hub_guide.md) (the hub architecture and the rail adapters — the STP angle), [Crédit Agricole Software Systems](credit_agricole_software_systems_guide.md) (§3 the Cymbal Bank estate — SWIFT membership and the MT700 trade flow), [Trade Finance](trade_finance_guide.md) (the LC lifecycle), [Core Banking Systems](core_banking_systems_guide.md) (the back-office angle)
> **Companion guides (technology/, prefix `../technology/`):** [Cybersecurity](../technology/cybersecurity_guide.md) (the PKI/2FA and key-management angle — cross-ref lightly), [Message Queue Data Loss](../technology/message_queue_data_loss_guide.md) (the delivery-guarantee discipline for the SWIFT message flows), [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (the 24/7 messaging estate — cross-ref lightly)

---

**How to use this guide:** Section 1 is the overview — where the SWIFT connectivity layer sits between the back office and SWIFTNet, and where the Alliance family fits, in an aspect/description table. Section 2 is the Alliance family — Alliance Access, Alliance Gateway and Alliance Lite2, each verified against the swift.com product pages, with honest flags where the web could not confirm something. Section 3 is the messaging — FIN MT (cross-referencing the payment-rails guide's MT classes rather than re-deriving them), MX/ISO 20022 (cross-referencing the ISO 20022 guide's CBPR+ facts), and the straight-through-processing flows. Section 4 is the connectivity and the security — SWIFTNet, the PKI/2FA, and the RMA. Section 5 is the deployment — on-prem vs the Lite2 cloud, and the back-office integration. Section 6 is the operations — message monitoring, queue management, and gpi (cross-referenced, not re-derived). Section 7 is the banking context — the Cymbal Bank SWIFT estate. Section 8 is the worked example — a Cymbal Bank SWIFT-connectivity design, with the Alliance Access deployment, the RMA setup, and the MT103/pacs.008 and MT700 flows. Section 9 is the summary — the final word is "the secure lane". The glossary, the claims audit, the what-could-not-be-verified list and the verification ledger close the file. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/` and `management/` guides `../management/`. **Integrity convention:** ✅ = verified this pass against a primary source or in a cross-referenced guide's ledger; ⚠ = flagged/unverified or approximate; ⚠-knowledge = well-documented industry knowledge that could not be re-verified live in this pass and is flagged honestly.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Alliance Family](#2-the-alliance-family)
3. [The Messaging](#3-the-messaging)
4. [The Connectivity and the Security](#4-the-connectivity-and-the-security)
5. [The Deployment](#5-the-deployment)
6. [The Operations](#6-the-operations)
7. [The Banking Context](#7-the-banking-context)
8. [The Worked Example — A SWIFT-Connectivity Design](#8-the-worked-example--a-swift-connectivity-design)
9. [The Summary](#9-the-summary)
10. [Glossary](#10-glossary)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Verification Ledger](#13-verification-ledger)

---

## 1. The Overview

### 1.1 The Short Answer

**SWIFT** — the Society for Worldwide Interbank Financial Telecommunication — is the member-owned cooperative that runs the private messaging network banks use for cross-border financial communication, founded in Brussels on **3 May 1973** ✅ (the [payment_rails_guide.md](payment_rails_guide.md) ledger), with FIN messaging live since **1977** ⚠-knowledge. SWIFT is a *messaging* cooperative, not a settlement system: it carries the instructions — MT103, MT202, MT700, pacs.008 — while the money moves separately through nostro accounts and the RTGS rails ✅ (cross-ref [payment_rails_guide.md](payment_rails_guide.md) §2.5). The scale: the swift.com RMA information paper itself states "more than 11,000 banking and securities organisations, market infrastructures and corporate customers in more than 200 countries and territories" ✅ (verified 2026-08-27) — which upgrades the repo's ⚠-knowledge figure to a primary-source ✅. This guide does **not** re-derive the rails guide's SWIFT facts; it cross-references them.

**The connectivity layer** is the estate between the back office and SWIFTNet: the interfaces that speak FIN/InterAct/FileAct, the gateway that owns the network session, the hardware security module that signs the traffic, and the operations tooling around them. It is where the bank's internal formats become SWIFT messages and where inbound SWIFT messages become back-office entries — and it is the layer the [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3 calls "the bank's connectivity hub (**SWIFT Alliance-class**, or a vendor hub)" ✅ — the exact mention this guide is built to expand.

**The Alliance family** is SWIFT's product line for that layer. The three classic members, in the order an architect meets them:

- **Alliance Access** — the messaging interface and the secure hub: message creation, validation, authorisation, routing, monitoring, and the built-in RMA enforcement ✅ (swift.com product page, 2026-08-27).
- **Alliance Gateway** — the network-facing hub: a single secure entry point to SWIFTNet that concentrates FIN, InterAct, FileAct and WebAccess flows ✅ (swift.com product page, 2026-08-27).
- **Alliance Lite2** — the cloud-based access for institutions that do not want an on-premise estate; per swift.com (2026) it is now "available for existing clients only", with **Alliance Cloud** (launched 2020 ✅) as its successor ✅ (swift.com).

### 1.2 The Connectivity Layer — the Three-Zone Picture

A bank's SWIFT estate divides into three zones ✅/⚠ (the zone model is standard industry practice; swift.com and the AWS Customer Security Programme whitepaper describe the components): the **back-office zone** (core banking, the payments hub, trade finance — the business applications that generate and consume messages), the **interface zone** (Alliance Access — the messaging interface, typically on-premise on Windows, Linux or AIX ✅), and the **network zone** (Alliance Gateway in a demilitarised zone between two firewalls ✅, the SWIFTNet Link, and the HSM that holds the SWIFTNet PKI certificates and generates the signatures ✅). The AWS whitepaper on the SWIFT Customer Security Programme describes the same components — "messaging interface, SwiftNet link (SNL), hardware security model (HSM), SWIFT connector, jump server, operator PC" — as mandated to be deployed in a segmented, controlled "secure zone" ✅/⚠ (AWS whitepaper, 2021 framework; that the *mandate* exists is SWIFT's Customer Security Programme — flagged ⚠-knowledge as the framework versioning evolves and the whitepaper itself is dated).

The architectural point: the connectivity layer is a **translation and control choke point**, not plumbing. Every cross-border instruction crosses it twice (outbound and inbound), so its reliability, its security posture and its message-management discipline set the ceiling for the bank's straight-through processing rate.

### 1.3 The Overview Table — Aspect / Description

| Aspect | Description | Status |
|---|---|---|
| The network | SWIFTNet — the private, secure IP backbone; no public-internet path ✅ (Microsoft Learn; Payments Signal) | ✅ |
| The messaging interface | Alliance Access — the secure hub; MT and MX, FIN/InterAct/FileAct exchange ✅ | ✅ |
| The network interface | Alliance Gateway — the single secure entry point to SWIFTNet ✅ | ✅ |
| The cloud access | Alliance Lite2 (existing clients only) → Alliance Cloud (launched 2020) ✅ | ✅ |
| The message standards | FIN MT (ISO 15022) + MX (ISO 20022); CBPR+ since Nov 2022; MT↔MX coexistence ends 22 Nov 2025 ✅ (iso_20022 guide) | ✅ |
| The security | SWIFTNet PKI (HSM-held certificates, signatures), 2FA in the interface products, RMA counterparty control ✅ | ✅ |
| The counterparty control | RMA/RMA Plus — the SWIFT-mandated authorisation for FIN ✅ (swift.com RMA paper) | ✅ |
| The transport services | InterAct (store-and-forward MX), FileAct (file transfer), Browse, FIN ✅ (Oracle) | ✅ |
| The operations | Routing points and queues, monitoring, audit trail, gpi tracking ✅ | ✅ |
| The bank's estate | Cymbal Bank — SWIFT membership ✅; Alliance-Access-class connectivity ⚠ inferred; MT700 LC issuance ✅ | ✅/⚠ |

### 1.4 Reading the Overview Table

The rows interlock in a fixed order: the message standards (row 5) ride on the transport services (row 8); the transports run over SWIFTNet (row 1); the interface and the gateway (rows 2–3) are the bank's two ends of that network; the security rows (6–7) gate *who* may use the lane and *with whom*; the operations row (9) is what the ops room watches; and the estate row (10) anchors all of it to the Cymbal Bank context of §7–§8. An architect designing a SWIFT estate in 2026 starts from rows 5 and 7 — the CBPR+ end-of-coexistence deadline and the RMA mandate — and works outward.

### 1.5 The Five Layers of the Connectivity Stack

Zooming in one more level, the connectivity layer decomposes into five functional layers, each mapped to a verified component or concept from this guide:

| Layer | What it does | Component / concept | Status |
|---|---|---|---|
| 1. The application integration | Moves instructions between the back office and the interface | Basic Integration Package (files), Full Integration Package (IBM MQ, SOAP) ✅ | ✅ |
| 2. The message management | Validates, repairs, authorises, routes, monitors | Alliance Access — the secure hub ✅ | ✅ |
| 3. The counterparty control | Gates who may send what, and to whom | RMA / RMA Plus, built into the interface ✅ | ✅ |
| 4. The network edge | Owns the sessions, concentrates the flows, survives failures | Alliance Gateway (DMZ), SNL ✅ | ✅ |
| 5. The identity and the signature | Proves the bank's identity on the network | SWIFTNet PKI + HSM ✅ | ✅ |

The five layers are the mental model the rest of this guide uses: §2 covers layers 1–4 (the products), §3 covers layer 2 (the messages), §4 covers layers 3–5 (the security), §5 covers layers 1 and 4 (the deployment), and §6 covers layer 2 again from the operations side (the queues). The stack is also the fault-containment story: a failure in layer 4 (a Gateway outage) is absorbed by layer 2's automatic switchover ✅ (swift.com), and a failure in layer 5 (an HSM issue) is a network-identity event, not a business-logic one ✅/⚠ (the HSM's role is verified ✅; the incident-classification framing is analysis ⚠).

---

## 2. The Alliance Family

### 2.1 Alliance Access — the Messaging Interface and the Secure Hub ✅

Verified against the swift.com product page (2026-08-27):

- **The positioning** — "our leading messaging interface, connecting banks and market infrastructures globally", processing "millions of messages daily" and scaling with the business ✅ (swift.com).
- **The footprint** — "over 2,000 installations worldwide" ✅ (swift.com — a vendor figure, but from the primary source; treat the precise count as indicative ✅/⚠-ish, the *scale* as solid ✅). The repo's [credit_agricole_software_systems_guide.md](credit_agricole_software_systems_guide.md) §3.2 records the Cymbal Bank estate's SWIFT connectivity as "Alliance-Access-class inferred ⚠" — the vendor identity is non-public, flagged honestly.
- **The platforms** — Windows, Linux and AIX; packaged with the browser-based **Alliance Web Platform SE** as the default graphical user interface ✅ (swift.com).
- **The standards** — supports MT and MX message standards, and FIN, InterAct and FileAct exchange ✅ (swift.com).
- **The message lifecycle** — "message creation, verification, authorisation, modification, repair, template creation, priority updates and queries"; format-and-field validation for both MT and MX, which is the "message repair reduction" story ✅ (swift.com).
- **The routing** — "extensive message routing features through configuration of routing points and queues", a "complete audit trail", and robust monitoring ✅ (swift.com).
- **The integration** — the Basic Integration Package (file-based back-office integration) and the Full Integration Package (the IBM MQ and SOAP channels plus business modules); "several business application providers have certified their applications to integrate natively with Alliance Access" ✅ (swift.com).
- **The resilience** — 24×7 operations with no routine-maintenance window, automatic switchover between Alliance Gateway instances (including across sites), and an optional Database Recovery licence ✅ (swift.com).
- **The security integration** — RMA functionality is built into Alliance Access ✅ (swift.com RMA paper).

The **"secure hub"** framing: Alliance Access is where the authorisation workflow, the validation, and the RMA gating actually execute — it is the security control point of the messaging estate. Note the integrity caveat: "secure hub" is this guide's architectural framing of the swift.com facts (single window to SWIFT + message management + RMA built in), **not** a SWIFT product name ⚠.

### 2.2 Alliance Gateway — the File-Transfer and Hub Interface ✅

Verified against the swift.com product page (2026-08-27):

- **The positioning** — "a single, secure entry point to SwiftNet that simplifies connectivity and reduces costs"; it consolidates message flows from multiple applications ✅ (swift.com).
- **The concentration** — concentrates FIN, InterAct, FileAct and WebAccess flows "through a single instance using technical adapters like RAHA and MQHA" ✅ (swift.com); IBM's documentation describes the SAG/RAHA connectivity from the vendor side, corroborating the adapter model ✅.
- **The direct connection** — business adapters such as **TDA** enable direct application connections to Alliance Gateway ✅ (swift.com).
- **The demilitarised zone** — "can be placed between two firewalls — the 'demilitarised zone' — enabling tighter IP protocol filtering as well as segregating connectivity and business processing" ✅ (swift.com).
- **The throughput** — "supports over 100 transactions per second" ✅ (swift.com — a vendor performance claim; treat as indicative ⚠-ish, but it is the primary source's own number, so ✅ with that caveat).
- **The administration** — the four-eyes principle for administration, optional LDAP-based central operator authentication, SNMP monitoring notifications, and an integrity-verification mechanism that detects tampering with software and database components ✅ (swift.com).
- **The developer story** — the Gateway Developers Toolkit, "an easy-to-use framework for application vendors to offer their products on Swift" ✅ (swift.com).

Honest flags: the product line was long known as **SWIFTAlliance Gateway (SAG)** — the legacy name still appears throughout vendor documentation (IBM, Axway) ✅, while swift.com's current page brands it simply "Alliance Gateway" ✅. The *file-transfer* role: Alliance Gateway is the network-facing owner of the FileAct and InterAct sessions (client mode, real-time and store-and-forward) ✅ (Oracle), and Alliance Access rides on it for automatic connection switchover ✅ (swift.com). What could **not** be verified this pass: the current production version number of Alliance Gateway — the swift.com knowledge-centre resource links reference "Alliance Gateway 7.7" documents ⚠, but that is a document reference, not a confirmed live version, so it is flagged ⚠ rather than asserted.

### 2.3 Alliance Lite2 — the Cloud-Based Access ✅/⚠

Verified against the swift.com Alliance Lite2 page (2026-08-27):

- **The current status** — "Alliance Lite2 is available for existing clients only, as we have brought in Alliance Cloud, a more efficient and powerful solution" ✅ (swift.com). This is the single most important Lite2 fact for an architect planning new connectivity in 2026: **Lite2 is closed to new clients** ✅.
- **Alliance Cloud** — "Swift's preferred financial messaging interface launched in 2020" ✅ (swift.com FAQ): cloud-native infrastructure, "multi-active operating centres for better disaster recovery", upgrades every six weeks, REST API for automation, and integration with **Alliance Connect Virtual** (Swift's virtual VPN offering for back-office applications running natively on public cloud) ✅ (swift.com).
- **The Lite2 support status** — "There is currently no specified End of Life timeline for Alliance Lite2… Alliance Lite2 remains fully supported" ✅ (swift.com).
- **The launch date** — Lite2 launched around 2010 as the low-footprint, SWIFT-managed access for smaller institutions ⚠-knowledge: the launch year could not be re-verified on swift.com this pass (the current page is migration-focused), so it is flagged honestly ⚠ rather than asserted.

### 2.4 The Family Table — Product / Role / Deployment / Status

| Product | Role | Deployment | Status flag |
|---|---|---|---|
| Alliance Access | Messaging interface — the secure hub: create/validate/authorise/route/monitor/repair; RMA built in ✅ | On-premise, Windows/Linux/AIX ✅; 2,000+ installations ✅ | ✅ active, flagship |
| Alliance Gateway | Single secure entry point to SWIFTNet; FIN/InterAct/FileAct/WebAccess concentration via RAHA/MQHA/TDA ✅ | DMZ between two firewalls ✅; >100 tps ✅ | ✅ active; version ⚠ unverified |
| Alliance Lite2 | Cloud-based managed access ✅ | SWIFT-hosted cloud ✅ | ⚠ closed to new clients ✅; no EOL ✅ |
| Alliance Cloud | Lite2 successor — cloud-native messaging interface ✅ | Cloud-native, multi-active operating centres ✅ | ✅ launched 2020 ✅ |
| Alliance Web Platform SE | Browser-based GUI for the Alliance products ✅ | Ships with Access/Gateway ✅ | ✅ |
| Alliance Messaging Hub | Modular high-volume messaging hub, alongside or instead of Access ⚠ | On-prem/cloud ⚠ | ⚠ ecosystem mention only; not extracted this pass |
| Alliance RMA (standalone) | Standalone RMA management where not built into an interface ✅ | On-prem ✅ | ✅ (swift.com RMA paper) |

### 2.5 The Family Logic — Why the Layering Exists

The division of labour is deliberate: **Alliance Access** carries the business logic of messaging — validation, authorisation, repair, routing, RMA — and can itself be "a single window to our FIN, InterAct and FileAct messaging services" ✅ (swift.com); **Alliance Gateway** carries the network plumbing — sessions, concentration, DMZ security, load balancing — and can sit entirely inside the network zone; **Lite2/Alliance Cloud** collapse both roles into a SWIFT-managed service for institutions that want zero footprint ✅ (swift.com). A bank that already runs Alliance Access still buys Alliance Gateway for the network edge (the switchover story in §2.1 depends on it ✅); a bank that wants the smallest possible estate buys the cloud option. The family is therefore not three competing products but three layers of one architecture — which is exactly how the worked example of §8 assembles them.

### 2.6 The Alliance Messaging Hub — the Newer Sibling ⚠

The swift.com ecosystem now also lists **Alliance Messaging Hub (AMH)** — described in the search results as "a modular financial messaging solution designed for our network… handles messages and files across multiple formats simultaneously", and positioned by industry coverage as a high-volume enterprise messaging hub used by large banks alongside or instead of Alliance Access ⚠ (swift.com ecosystem listing — search-verified snippet ✅/⚠; the product page itself was not extracted this pass, so the details are flagged ⚠). Microsoft's and AWS's guidance pages describe deploying AMH with Alliance Connect Virtual on Azure/AWS ✅/⚠ (vendor guidance pages, 2026 — the *existence* of the guidance is verified ✅; the guidance content is vendor framing ⚠-ish). For this guide's purposes AMH is noted as the family's modular, high-volume sibling — the architect's alternative when the messaging estate outgrows the interface model — but every AMH-specific claim beyond its existence is flagged ⚠ and parked in §12. The Cymbal Bank class-level design of §8 stays with the Alliance Access + Gateway + Lite2/Cloud trio that this pass could verify against the primary source ✅.

### 2.7 Alliance Entry — the Smaller Interface ✅/⚠

One more family member appears in the primary sources: the swift.com RMA paper states that RMA functionality "is built into the Alliance Access and **Alliance Entry** SWIFT interfaces" ✅ — which verifies that Alliance Entry exists, carries RMA, and sits in the same interface family ✅. Its market role — the entry-level, lower-volume messaging interface for smaller institutions, the little sibling of Alliance Access — is well-documented industry knowledge ⚠-knowledge (no Alliance Entry product page was extracted this pass; the role description is flagged ⚠). For the estate map: Alliance Entry is the interface a smaller group entity or a new subsidiary would run before graduating to Alliance Access ✅/⚠ (the graduation pattern is this guide's inference from the verified family structure ⚠). It is included here for completeness and excluded from the §8 design, which is sized for the Alliance Access class ✅.

---

## 3. The Messaging

### 3.1 The FIN MT — Cross-Referenced, Not Re-Derived

FIN is SWIFT's store-and-forward messaging service for the legacy MT message set (ISO 15022). The MT classes the connectivity layer carries every day are documented in the [payment_rails_guide.md](payment_rails_guide.md) ledger and the [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3 — this guide cross-references rather than re-derives them:

| MT class | Use | Cross-ref |
|---|---|---|
| MT103 | Customer credit transfer — the serial cross-border payment ✅ | payment_rails_guide.md §2.5 |
| MT202 | Bank-to-bank transfer / cover payment ✅ | payment_rails_guide.md §2.5 |
| MT700 | Documentary credit (LC) issuance ✅ — the Cymbal Bank estate issues LCs over MT700 ✅ | credit_agricole_software_systems_guide.md §3 |
| MT940 | Account statement ✅ | payment_rails_guide.md §2.5 |
| MT54x / MT5xx | Settlement instructions / securities settlement ✅ | capital_markets_architecture_guide.md §7.3 |
| MT300 / MT304 | FX and derivatives confirmations ✅ | capital_markets_architecture_guide.md §7.3 |

The operational facts that matter at the interface layer: FIN is **store-and-forward** (the network holds the message until the recipient's connection accepts it), it is **RMA-gated** (§4.3), and it is **end-to-end signed** for most user-to-user messages — the swift.com RMA paper states: "The use of RMA or RMA Plus is mandatory for user-to-user messages that require end-to-end signature, which covers most types of SWIFT FIN messages" ✅. The MT7xx series beyond MT700 (MT707 amendments, MT720 transfers) is standard industry knowledge ⚠-knowledge and flagged as such.

### 3.2 The MX / ISO 20022 — Cross-Referenced

MX messages (ISO 20022) travel over **InterAct** (store-and-forward for CBPR+, gpi, securities) and **FileAct** (file transfer: pain.001 batches, camt.053 statement files) ✅ (the [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md) ledger; Oracle's SWIFTNet services documentation). The CBPR+ facts — go-live November 2022 ⚠-knowledge with MT↔MX coexistence, the Business Application Header mandatory, the UETR in every payment, and the **end of MT↔MX coexistence for in-scope FI-to-FI payment instructions on 22 November 2025** ✅ — are in the ISO 20022 guide's ledger (§13) and are **not re-derived here** ✅. The canonical mapping **MT103 ↔ pacs.008** is lossy ✅ (iso_20022 guide) — which is exactly why the connectivity hub's translation layer must be designed, not improvised; the [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3 makes the same point: the golden-copy data model is what makes the translation lossless ✅.

### 3.3 The Message Flows — the STP

Straight-through processing is the operational goal of the whole connectivity layer: an instruction enters the bank in one format and exits to the correspondent without manual re-keying. The flow through the Alliance estate, verified component by component:

1. **Initiation** — the back office (the payments hub, trade finance) hands the instruction to Alliance Access over the integration channel: files (Basic Integration Package) or IBM MQ/SOAP (Full Integration Package) ✅ (swift.com).
2. **Validation and repair** — Alliance Access validates format and fields for both MT and MX; repairable messages are repaired, non-repairable ones rejected ✅ (swift.com).
3. **Authorisation** — the four-eyes workflow: a second operator authorises before release ✅/⚠ (swift.com confirms authorisation as a message-management feature ✅; the exact workflow configuration is not public ⚠; on the Gateway side, administration is explicitly four-eyes ✅).
4. **The RMA check** — outbound: the counterparty must hold a valid RMA with the bank, else the traffic is blocked at the sender level ✅ (swift.com RMA paper).
5. **Routing** — routing points and queues direct the message to the right network service — FIN, InterAct or FileAct ✅ (swift.com).
6. **Transport** — Alliance Gateway owns the SWIFTNet session; the HSM signs with the SWIFTNet PKI private key; the message crosses the private IP backbone ✅ (swift.com; Microsoft Learn).
7. **The inbound mirror** — the correspondent's estate does the same in reverse: the bank receives, validates, screens, and books into the back office.

### 3.4 The Messaging Table

| Message class | Standard | Transport | Typical flow | Status |
|---|---|---|---|---|
| MT103 / MT202 | FIN MT (ISO 15022) ✅ | FIN, store-and-forward ✅ | Payments — serial and cover ✅ | ✅ cross-ref |
| MT700 | FIN MT ✅ | FIN ✅ | Trade — LC issuance (the Cymbal Bank estate ✅) | ✅ cross-ref |
| MT940 | FIN MT ✅ | FIN ✅ | Cash management — account statements | ✅ cross-ref |
| pacs.008 / pacs.009 | MX (ISO 20022, CBPR+) ✅ | InterAct ✅ | Payments — customer / cover credit transfer | ✅ cross-ref |
| camt.053 | MX ✅ | InterAct or FileAct ✅ | Statements, incl. file delivery | ✅ cross-ref |
| pain.001 batches | MX ✅ | FileAct ✅ | Mass-payment files | ✅ cross-ref |
| MT300/304, MT54x, MT5xx | FIN MT ✅ | FIN ✅ | Markets — confirmations, settlement instructions | ✅ cross-ref |

### 3.5 The STP Measures — What the Ops Room Watches

Straight-through processing is not a feature; it is a measured outcome. The connectivity layer produces the metrics the ops room lives by — the definitions are standard industry practice ⚠-knowledge (the *controls* that produce them are verified in §6):

| Measure | What it means | Which §6 control feeds it |
|---|---|---|
| The STP rate | The share of instructions processed end-to-end without manual intervention ✅/⚠ | Validation + repair (Alliance Access ✅), routing points ✅ |
| The repair rate | The share of messages needing manual repair before release ✅/⚠ | Format-and-field validation, the "message repair reduction" story ✅ (swift.com) |
| The authorisation backlog | The queue of messages waiting for the second signature ✅/⚠ | The four-eyes workflow ✅/⚠ |
| The queue age | How long a message waits in a routing queue ✅/⚠ | Routing points and queues ✅; delivery notifications / non-delivery warnings ✅ (FileAct factsheet) |
| The RMA coverage | The share of counterparty traffic covered by a valid, current RMA ✅/⚠ | RMA lifecycle management ✅ (swift.com RMA paper) |
| The gpi exception rate | Payments whose tracker shows a delay or a stop ✅/⚠ | The gpi tracker, cross-ref iso_20022 §9.3 ✅ |

The honest flag: the *measure definitions* (STP rate, repair rate, queue age) are standard industry practice ⚠-knowledge — no primary source was extracted defining them this pass — while the *controls* that produce them (validation, repair, queues, notifications, RMA lifecycle, tracker) are each ✅-verified above. The worked example's operations table in §8.4 reuses this measure set.

### 3.6 The Dual-Standard Calendar — the Translation Deadlines

The connectivity layer's message-management design is pinned to a calendar, and the dates are in the sibling ledgers (cross-referenced, not re-derived):

| Date | Event | Ledger |
|---|---|---|
| 3 May 1973 | SWIFT founded ✅ | payment_rails_guide.md |
| 1977 | FIN messaging live ⚠-knowledge | payment_rails_guide.md |
| January 2017 | gpi live ✅ | payment_rails_guide.md (SWIFT press release 16 Feb 2017) |
| 21 March 2022 | T2 (EU RTGS) on ISO 20022 ✅ | iso_20022_core_processes_guide.md §13 |
| November 2022 | CBPR+ go-live; MT↔MX coexistence begins ⚠-knowledge | iso_20022_core_processes_guide.md §13 |
| **22 November 2025** | **End of MT↔MX coexistence — in-scope FI-to-FI payment instructions must be MX on CBPR+ corridors** ✅ | iso_20022_core_processes_guide.md §13 |
| 14 November 2026 | SWIFT will no longer accept fully unstructured postal addresses in CBPR+ messages (addresses must be structured or hybrid) ✅ | iso_20022_core_processes_guide.md §13 |

Two consequences for the interface estate: (1) every routing point, template and validation rule that still assumes MT-only payment flows is already past its sell-by date ✅ (the end-of-coexistence fact ✅); (2) the address-structuring requirement is a *data-quality* project that lands in the interface's validation layer — the same layer that does the MT/MX format-and-field validation ✅ (swift.com Alliance Access) — and in the golden-copy model of the connectivity hub ✅ (capital_markets §7.3).

---

## 4. The Connectivity and the Security

### 4.1 SWIFTNet — the Network ✅

SWIFTNet is SWIFT's advanced IP-based messaging network: a private, secure backbone that "no one reaches over the public internet" ✅ (Microsoft Learn describes SWIFTNet and its mandatory interface; Payments Signal's 2026 connectivity explainer states the private-backbone point plainly). The network facts an architect needs:

- **The mandatory access point** — **SWIFTNet Link (SNL)** is the mandatory network interface: "SWIFTNet requires SNL for all external interfaces", and the SNL API is the way business applications access SWIFTNet services; the SNL is incorporated into the SWIFTAlliance products ✅ (Microsoft Learn). swift.com's current product page positions SwiftNet Link as "your single access point to Swift, streamlining your infrastructure with transport, formatting, security, and service management" ✅ (search-verified snippet, 2026-08-27).
- **The four messaging services** — SWIFTNet InterAct, FileAct, Browse, and FIN ✅ (Oracle's SWIFTNet documentation). The interface estate maps onto them: FIN for the legacy MT set, InterAct for MX store-and-forward (CBPR+, gpi, securities), FileAct for files, Browse for session-based WebAccess ✅.
- **InterAct** — store-and-forward and real-time exchange for MX messages ✅ (Oracle); the ISO 20022 guide's ledger documents it as the CBPR+/gpi transport ✅.
- **FileAct** — the file-transfer service, verified from the swift.com factsheet (2026-08-27): two mechanisms — "push files to correspondents (file transfer)" and "get files from correspondents (file download)" ✅; a **store-and-forward mode** that "safe stores the file and handles the delivery upon connection of your correspondent" ✅; **delivery notifications and non-delivery warnings** so the sender can track delivery status ✅; an optional **file header** block carrying business summary information (number of payments, total amount) without opening the file ✅; **FileAct Copy** (Y-copy for authorisation, T-copy for information) ✅; optional compression and automatic segregation of file traffic from other message flows ✅; and open formats — "MTs or MXs, domestic formats, or your own proprietary ones" ✅.
- **Browse** — the session-based service (WebAccess) ✅ (Oracle; swift.com's Gateway page lists WebAccess among the flows it concentrates ✅).
- **The availability claim** — the FileAct factsheet asserts SWIFTNet availability "above 99.99%" ✅/⚠ (a vendor claim from the primary source; marked as such — treat as indicative, not audited).

The network layer is where the CSP (Customer Security Programme) boundary sits: the connectivity components must live in a secure zone, and every message crossing the backbone is signed and authenticated under SWIFTNet PKI (§4.2) ✅/⚠ (AWS whitepaper; swift.com security pages).

### 4.2 The PKI and the 2FA ✅

- **SWIFTNet PKI** — the certificate-based security layer for the whole network: "FileAct uses SWIFTNet Public Key Infrastructure (PKI) that enables the authentication and the integrity control for every file transferred", with certificate issuance and management under a robust registration mechanism; "the PKI signature guarantees the non-repudiation of file emission and file reception" ✅ (swift.com FileAct factsheet). The same PKI underpins FIN and InterAct traffic ✅/⚠ (the factsheet states it for FileAct explicitly ✅; the network-wide application is standard industry knowledge ⚠-knowledge).
- **The HSM** — the Hardware Security Module "is the security device that contains your critical SwiftNet Public Key Infrastructure (PKI) certificates and generates signatures for your traffic" ✅ (swift.com HSM product page, search-verified 2026-08-27). The HSM is the crown jewel of the network zone: whoever holds the signing keys controls the bank's identity on SWIFTNet.
- **The 2FA** — as part of its Customer Security Programme, SWIFT ran a campaign "to raise awareness for Swift's Relationship Management Application (RMA) and 2-Factor Authentication (2FA) in Swift products" ✅ (swift.com news item, search-verified 2026-08-27). The two-factor story is operator-level: human access to the interface products requires the second factor, layered on top of the machine-level PKI ✅/⚠ (the campaign's existence is verified ✅; the precise 2FA mechanics per product are configuration-level and not public ⚠).
- **The CSP secure zone** — the AWS whitepaper on the SWIFT Customer Security Controls Framework documents that SWIFT mandates the connectivity components — messaging interface, SNL, HSM, SWIFT connector, jump server, operator PC — to be deployed in a "secure zone": a segmented and controlled environment bounded by the CSP control framework ✅/⚠ (AWS whitepaper, 2021; the framework versioning evolves — flagged ⚠-knowledge on the current version number). Cross-ref the [Cybersecurity Guide](../technology/cybersecurity_guide.md) lightly for the general PKI/2FA and network-segmentation patterns — this guide covers the SWIFT-specific instantiation only.

### 4.3 The RMA — the Relationship Management Application ✅

The RMA is the counterparty access control of the FIN world, verified from the swift.com information paper "RMA and RMA Plus: managing your correspondent connections" (July 2016, extracted 2026-08-27):

- **The definition** — "The RMA is a SWIFT-mandated authorisation that enables financial institutions to define which counterparties can send them FIN messages. RMA is the Relationship Management Application, though in common use when discussing an RMA, what is described is the key exchange and authorisation process between two institutions" ✅ (swift.com).
- **The blocking model** — "Any unwanted traffic is blocked at the sender level, reducing the operational risks associated with handling unwanted messages" ✅ — the RMA is enforced *before* the message reaches the receiving institution, at the sender's own interface ✅.
- **The lineage** — RMA was introduced **in 2009 as a replacement for the Bilateral Key Exchange (BKE)** ✅ (swift.com).
- **The mandate** — RMA or RMA Plus is "mandatory for user-to-user messages that require end-to-end signature, which covers most types of SWIFT FIN messages" ✅ (swift.com).
- **The granularity** — **RMA Plus** lets institutions "specify which message type(s) they want to send to, and receive from each of their counterparties. For example, an institution might only wish to receive letters of credit from a particular correspondent" ✅ (swift.com) — the exact trade-finance pattern the worked example of §8 uses.
- **The non-customer case** — "RMAs can also be established with non-customers — in other words, counterparty financial institutions of the bank for which the bank does not hold an account" ✅ (swift.com).
- **The product placement** — RMA functionality "is built into the Alliance Access and Alliance Entry SWIFT interfaces, and Alliance RMA is also available as a standalone product" ✅ (swift.com).
- **The compliance angle** — the Wolfsberg Group's guidance on RMA due diligence tells institutions to "incorporate RMA due diligence standards into their Financial Crime/AML/KYC programmes" ✅ (Wolfsberg Group; swift.com paper cites it). Some regulators require full due diligence on correspondents "whenever an RMA is present, regardless of whether a business relationship is actually in place" ✅ (swift.com).
- **The lifecycle risk** — dormant or inactive RMAs are a fraud and cost risk: "Legacy RMAs can also create the opportunity for payments to be sent to destinations which may no longer be wanted or authorised" ✅ (swift.com) — the operational rationale for RMA hygiene programmes (§6.4).

### 4.4 The Security Table

| Control | What it does | Source | Status |
|---|---|---|---|
| SWIFTNet PKI | Authentication + integrity for every message/file; non-repudiation of emission and reception ✅ | swift.com FileAct factsheet | ✅ |
| HSM | Holds the SWIFTNet PKI certificates, generates the signatures ✅ | swift.com HSM page | ✅ |
| 2FA | Second factor for operator access to the interface products ✅ | swift.com (CSP awareness campaign) | ✅ |
| RMA | Blocks unwanted FIN at the sender level; mandatory for end-to-end-signed user messages ✅ | swift.com RMA paper | ✅ |
| RMA Plus | Message-type-level counterparty authorisation (e.g. LC-only) ✅ | swift.com RMA paper | ✅ |
| CSP secure zone | Connectivity components in a segmented, controlled environment ✅/⚠ | AWS whitepaper (2021 framework) | ✅/⚠ |
| Four-eyes administration | Administration requires two operators (Gateway) ✅ | swift.com Gateway page | ✅ |
| LDAP | Central operator authentication (Gateway) ✅ | swift.com Gateway page | ✅ |
| Integrity verification | Tamper checks on software and database components (Gateway) ✅ | swift.com Gateway page | ✅ |

### 4.5 The CSP Context — Why the Security Programme Exists

The Customer Security Programme is the industry's response to the 2016 cyber-heist era ⚠-knowledge (the February 2016 central-bank cyber theft, executed through SWIFT-related credentials and messaging, is well-documented industry history; the specific incident narrative is outside this guide's verified scope and flagged ⚠-knowledge). What is verified: the CSP exists as SWIFT's community-wide security framework ✅/⚠ (swift.com's RMA/2FA awareness campaign is explicitly "part of its Customer Security Programme" ✅), it mandates the secure-zone deployment of the connectivity components ✅/⚠ (AWS whitepaper), and it is enforced through the Customer Security Controls Framework with mandatory, annually audited controls — HSMs for protecting signing keys ✅/⚠ (the AWS whitepaper documents the framework; the annual-audit and HSM-mandate specifics are corroborated by practitioner documentation ⚠-ish but not by a swift.com page extracted this pass, so they are flagged).

The architectural consequence: the security table above is not a menu — the PKI/HSM row, the 2FA row and the RMA row are *mandatory* rows in any Alliance-class design, and the secure-zone row is the boundary condition that shapes the deployment (§5) ✅/⚠. The worked example treats them as non-negotiable requirements, not options.

---

## 5. The Deployment

### 5.1 The On-Prem vs the Lite2 Cloud ✅/⚠

The deployment decision in 2026 is simpler than it was a decade ago, because the cloud option changed shape ✅ (swift.com, 2026-08-27):

- **The on-prem estate** — Alliance Access on Windows/Linux/AIX + Alliance Gateway in the DMZ + SNL + HSM: the classic architecture, still fully sold and supported, with the multi-site switchover story (§2.1) as its resilience argument ✅ (swift.com).
- **The Lite2 estate** — the SWIFT-managed cloud access: **closed to new clients**; existing clients remain fully supported with no specified end-of-life ✅ (swift.com).
- **The Alliance Cloud estate** — the successor (launched 2020 ✅): cloud-native, multi-active operating centres for disaster recovery, upgrades every six weeks, REST API for automation and integration (Transaction Screening, Payment Controls, Alliance Warehouse are named integration examples ✅), and **Alliance Connect Virtual** — "Swift's virtual VPN offering" — for securely connecting back-office applications running natively on public cloud ✅ (swift.com).
- **The architectural reading** — this guide's reading of those verified facts: new connectivity in 2026 means *on-prem Alliance Access* or *Alliance Cloud*; Lite2 is a migration path, not a target ⚠ (analysis — the swift.com statements support it, but the "migration path, not a target" formulation is this guide's, not SWIFT's ⚠).
- **The CSP consequence** — the secure zone moves with the deployment: on-prem means the bank runs the zone; Alliance Cloud means SWIFT runs it, and the bank's public-cloud back office connects through Alliance Connect Virtual ✅/⚠ (swift.com describes the VPN ✅; the cloud-zone mapping is the AWS/Azure guidance pages' framing — Microsoft's guidance describes running Alliance Messaging Hub with Alliance Connect Virtual on Azure ✅/⚠ — that is the AMH product, not Access, and it is a vendor guidance page ⚠-ish; flagged).

### 5.2 The Back-Office Integration ✅/⚠

The interface is only as good as its integration channels, verified from the swift.com product pages (2026-08-27):

- **The Basic Integration Package** — file-based automation of back-office integration ✅ (swift.com).
- **The Full Integration Package** — "the IBM MQ and the SOAP channel" plus business modules ✅ (swift.com); IBM's own documentation covers connecting to Alliance Access through MQHA ✅.
- **The technical adapters at the Gateway** — RAHA and MQHA concentrate the flows ✅ (swift.com); Axway documents the SAG interface and the SWIFTNet RA remote API ✅.
- **The business adapters** — TDA enables direct application connections ✅ (swift.com).
- **The certified ecosystem** — "several business application providers have certified their applications to integrate natively with Alliance Access" ✅ (swift.com) — the ecosystem exists; the specific vendors are not enumerated on the page and were not verified ⚠.
- **The SAP angle — flagged** ⚠: enterprise back offices (SAP S/4HANA — the Payment Medium Workbench and the treasury modules — and the other ERP-class systems) typically integrate with SWIFT through the interface's file or MQ channels, or through vendor hubs; but the specific certified SAP↔Alliance Access adapter set **could not be verified this pass** (no primary source extracted), so it is flagged ⚠ honestly rather than asserted.
- **The payments-hub angle** — cross-ref [payments_hub_guide.md](payments_hub_guide.md): the hub is the translation layer between the internal canonical format and MT/MX; the interface is the hub's SWIFT rail adapter ✅/⚠ (the guide's architecture is cross-referenced ✅; the specific hub-vendor integration is per-bank ⚠).

### 5.3 The Deployment Table

| Decision | On-prem Alliance Access | Alliance Gateway | Lite2 / Alliance Cloud | Status |
|---|---|---|---|---|
| Footprint | Full estate: Access + Gateway + SNL + HSM ✅ | DMZ component ✅ | SWIFT-hosted, zero-footprint ✅ | ✅ |
| New clients | Open ✅ | Open ✅ | Lite2 closed ✅; Alliance Cloud open ✅ | ✅ |
| Back-office integration | Files (Basic), IBM MQ + SOAP (Full) ✅ | RAHA/MQHA/TDA ✅ | REST API + Alliance Connect Virtual ✅ | ✅ |
| Resilience / DR | Multi-site Gateway switchover ✅ | Automatic instance switching ✅ | Multi-active operating centres ✅ | ✅ |
| SAP adapter set | ⚠ not verified | ⚠ not verified | ⚠ not verified | ⚠ |
| Secure zone | Bank-operated zone ✅/⚠ | DMZ + zone ✅/⚠ | SWIFT-operated zone + cloud VPN ✅/⚠ | ✅/⚠ |

### 5.4 The Sizing Sketch — the On-Prem Estate

For the bank that chooses the on-prem path (the §8 worked example), the sizing sketch below is this guide's synthesis of the verified component facts — the *component roles* are ✅-verified, the *quantities* are illustrative ⚠ and marked as such:

| Estate element | Role (verified) | Illustrative sizing ⚠ | Verified basis |
|---|---|---|---|
| Alliance Access nodes | The messaging interface and the secure hub ✅ | 2 nodes (primary + DR), Linux, clustered ✅/⚠ (the multi-node pattern follows from the verified switchover/DR story ✅; node counts are the example's ⚠) | swift.com (platforms, resilience ✅) |
| Alliance Gateway pair | The single secure entry point ✅ | 2 instances in the DMZ with automatic switching ✅ (multi-instance switching is verified ✅; the pair count is the example's ⚠) | swift.com (Gateway page ✅) |
| SNL + HSM | The network session and the PKI signing ✅ | 1 SNL per site + 1 HSM per site, the HSM in the secure zone ✅/⚠ (the secure-zone placement is the CSP model ✅/⚠; the per-site counts are the example's ⚠) | Microsoft Learn ✅; swift.com HSM ✅; AWS whitepaper ✅/⚠ |
| Integration channels | Files / MQ / SOAP ✅ | MQ to the payments hub, file channel to trade finance ⚠ | swift.com ✅; trade platform ⚠ |
| RMA book | The counterparty control ✅ | ~120 RMA relationships, RMA Plus for the LC-only subset ⚠ | swift.com RMA paper ✅; the count ⚠ |

The sizing logic: the estate grows with the message volumes (Alliance Access "can be scaled to the needs of customers that exchange several million messages and files a day" ✅ — swift.com), the network edge grows with the session load (Gateway >100 tps ✅ — swift.com), and the security layer grows with the *counterparty* book, not the volume — the RMA table is sized by relationships, not messages ✅ (swift.com RMA paper).

---

## 6. The Operations

### 6.1 The Message Monitoring ✅

The operations story is verified from the swift.com product pages and the FileAct factsheet (2026-08-27):

- **Alliance Access** — a "complete audit trail and robust monitoring functions provide transparency and control over system operations" ✅ (swift.com).
- **Alliance Gateway** — "monitoring functionality supporting single network management protocol (SNMP) notifications", plus "tight control over business flows and your Swift environment" ✅ (swift.com).
- **FileAct visibility** — "the delivery notification and non-delivery warning options enable the sender to track the delivery status of the file transfers. Similarly, receiving customers can monitor their queues status" ✅ (swift.com factsheet).
- **The support envelope** — Alliance Access ships with 24/7/365 support across all time zones, and the interface "doesn't stop for daily routine maintenance activities" ✅ (swift.com) — the operational contract behind the 24×7 estate, cross-ref [zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md) lightly.

The ops-room picture: the monitors that matter are the interface queues (age, depth, repair rate), the network sessions (Gateway, SNL), the RMA health (expiring and dormant authorisations — §4.3), and the gpi tracker for the payment flows (§6.3).

### 6.2 The Queue Management ✅

- **Routing points and queues** — Alliance Access routes "through configuration of routing points and queues" ✅ (swift.com); the queue is where a message waits between validation, authorisation and release.
- **Store-and-forward** — FIN and InterAct hold messages in the network until the recipient's connection accepts them ✅ (Oracle; the ISO 20022 guide's ledger documents InterAct store-and-forward ✅); FileAct offers the same for files ✅ (swift.com factsheet).
- **Non-delivery warnings** — the sender learns when delivery has not happened ✅ (swift.com factsheet) — the trigger for investigations.
- **Load balancing** — FileAct "can also spread incoming traffic over multiple SWIFTNet Links to offer optimal load balancing and higher resiliency" ✅ (swift.com factsheet); Alliance Gateway maps multiple applications to a single PKI profile and switches between instances automatically ✅ (swift.com).
- **The delivery-guarantee discipline** — the [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3 makes the point this guide inherits: "a settlement instruction must not be lost, duplicated, or reordered — the delivery-guarantee decision is the difference between a fail and a fix" ✅; the queueing theory and the at-least-once/exactly-once trade-offs are in the [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) — cross-ref, not re-derived ✅.

### 6.3 The gpi — Cross-Referenced, Not Re-Derived

SWIFT gpi (Global Payments Innovation) is the payments service that added end-to-end tracking, faster settlement and fee transparency to cross-border payments. The verified facts live in the sibling ledgers and are **not re-derived here**:

- **Live January 2017** ✅ (payment_rails_guide.md ledger — SWIFT press release 16 February 2017); the Coupa press write-up corroborates "In January 2017, SWIFT introduced the gpi Service" ✅/⚠ (press, flagged as such).
- **The tracker and the UETR** — gpi tracker events ride on the UETR (UUID RFC 4122, 32 hex characters in PmtId/UETR) ✅ (iso_20022_core_processes_guide.md §9.3).
- **The interface's job** — at the connectivity layer, gpi changes two things: the UETR must be **preserved end-to-end** across the translation layer (MT↔MX is lossy — §3.2), and the tracker message flows must be routed like any other MX traffic ✅/⚠ (this is this guide's architectural reading of the cross-referenced facts — the UETR-preservation requirement is documented in the ISO 20022 guide ✅; the "interface's job" framing is analysis ⚠).
- swift.com's gpi product page describes the service as offering "real-time, end-to-end tracking, improving transparency, efficiency, and enabling action on delays" ✅ (search-verified snippet, 2026-08-27). Third-party statistics on gpi market share (e.g. "over 50% of cross-border SWIFT payments") are press-only ⚠ and are not asserted here.

### 6.4 The Operations Table

| Ops function | Mechanism | Status |
|---|---|---|
| Message lifecycle ops | Create / verify / authorise / repair / modify / template / priority / query ✅ (swift.com) | ✅ |
| Monitoring | Audit trail + monitoring in Access; SNMP in Gateway ✅ | ✅ |
| Queue visibility | Delivery notifications, non-delivery warnings, queue-status monitoring ✅ (FileAct factsheet) | ✅ |
| Resilience ops | 24×7, no maintenance window; Database Recovery licence; Gateway switchover ✅ | ✅ |
| gpi ops | Tracker + UETR preservation — cross-ref payment_rails_guide.md / iso_20022_core_processes_guide.md ✅ | ✅ |
| RMA ops | RMA lifecycle management; dormant-RMA rationalisation ✅ (swift.com RMA paper) | ✅ |

### 6.5 The Queue-Health Dashboard — the Ops View

The operations table decomposes into a dashboard the ops room can actually run — the *indicators* are standard industry practice ⚠-knowledge, the *feeds* are the verified mechanisms of §6.1–§6.4:

| Indicator | Green | Amber | Red | Feeds (verified) |
|---|---|---|---|---|
| FIN queue age | Minutes ✅/⚠ | Hours ⚠ | Past the service-level target ⚠ | Routing points and queues ✅; store-and-forward ✅ |
| FileAct delivery | Delivered + notification ✅/⚠ | Non-delivery warning ⚠ | Delivery failure ⚠ | Delivery notifications / non-delivery warnings ✅ (FileAct factsheet) |
| Repair rate | Within the target ⚠ | Rising ⚠ | Above the target ⚠ | Format-and-field validation ✅ (swift.com) |
| RMA expiries | None due ⚠ | Due within 30 days ⚠ | Expired ⚠ | RMA lifecycle management ✅ (swift.com RMA paper) |
| gpi exceptions | None ⚠ | Tracker delay ⚠ | Tracker stop ⚠ | The gpi tracker, cross-ref iso_20022 §9.3 ✅ |
| Gateway load | < 60% ⚠ | 60–80% ⚠ | > 80% ⚠ | SNMP notifications ✅; >100 tps headroom ✅ (swift.com) |

The honest flag, once more: the *thresholds* (green/amber/red bands, percentages) are this guide's illustrative operations design ⚠ — every bank sets its own — while the *feeds* they read are the ✅-verified mechanisms listed in the last column. The dashboard is the operational face of the §8 worked example's estate.

### 6.6 The Incident Playbook — the Message-Level Responses

The dashboard triggers a small set of playbooks — the *triggers* are verified mechanisms, the *responses* are standard operations practice ⚠-knowledge and flagged as such:

| Trigger (verified feed) | The playbook response ⚠ | Cross-ref |
|---|---|---|
| Non-delivery warning on a FileAct transfer ✅ (swift.com factsheet) | Check the counterparty's connection status; confirm the file header; resend or investigate ✅/⚠ | The delivery-guarantee discipline — [Message Queue Data Loss Guide](../technology/message_queue_data_loss_guide.md) ✅ |
| Tracker stop on a gpi payment ✅ (iso_20022 §9.3) | Open the gpi exception case; contact the correspondent; check the UETR chain ✅/⚠ | payment_rails_guide.md (gpi) ✅ |
| RMA expiry or dormant RMA ✅ (swift.com RMA paper) | Reconfirm the relationship; refresh the authorisation or revoke; update the KYC file ✅/⚠ | The Wolfsberg RMA due-diligence guidance ✅ |
| A repair-rate spike ✅ (swift.com — validation) | Trace the offending template or field; fix the source system, not the message ✅/⚠ | The golden-copy model — capital_markets §7.3 ✅ |
| An HSM or PKI event ✅/⚠ (swift.com HSM) | Stop outbound signing; fail over to the DR site's HSM; involve the security officer ✅/⚠ | The CSP secure zone — §4.5 ✅/⚠ |

The playbook discipline matters because the estate's failure modes are *message-level*: a lost settlement instruction is not an outage ticket, it is a markets incident ✅ (capital_markets §7.3) — the response is the queue discipline of §6.2 plus the playbook above ✅/⚠.

---

## 7. The Banking Context

### 7.1 The Cymbal Bank SWIFT Estate ✅/⚠

The Cymbal Bank estate is documented in [credit_agricole_software_systems_guide.md](credit_agricole_software_systems_guide.md) §3 and the post-trade language in [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3; this section assembles the SWIFT-connectivity view of that estate, inheriting every flag from those ledgers:

- **SWIFT membership** ✅ — the estate guide records SWIFT membership as verified; membership is the precondition for everything in this guide.
- **SWIFT connectivity** ✅/⚠ — the estate guide §3.2 records the estate's SWIFT connectivity as "Alliance-Access-class inferred ⚠": the vendor identity is non-public and flagged. This guide therefore designs the *class*, not the vendor — the §8 worked example is an Alliance-Access-class design ⚠.
- **Trade finance** ✅/⚠ — the estate issues letters of credit over SWIFT **MT700** ✅ (the estate guide's worked example: "Cymbal Bank issues the LC (SWIFT MT700) after limit check" ✅); the trade-finance platform vendor identity is non-public ⚠.
- **Payments** ✅ — cross-border payments ride SWIFT — MT103/MT202 today, pacs.008 under CBPR+ — with the group's domestic layer (STET/SEPA) for Europe ✅/⚠ (estate guide §2; the rails themselves are cross-referenced in [payment_rails_guide.md](payment_rails_guide.md) and [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)).
- **Markets** ✅/⚠ — FX/rates/derivatives on Murex-class platforms ⚠ (estate guide §3.1, flagged); the *post-trade* language is SWIFT — MT300/MT304 confirmations, MT54x settlement instructions, MT5xx securities settlement, MT202/MT103 payments ✅ (capital_markets §7.3).
- **Securities services** ✅/⚠ — the CACEIS servicing arm: entity-level facts solid ✅, internal platforms non-public ⚠.
- **The Singapore hub** ✅/⚠ — the Asia-Pacific hub is Singapore (estate guide) — the natural home for the SWIFT-connectivity design of §8.
- **The honesty rule** — the estate guide's verdict is inherited wholesale: the franchise is documented, the system inventory is not; every named system is either flagged ⚠ or class-level. No fabrication, and no vendor names invented.

### 7.2 The Banking Table — Business Line / SWIFT Message Set / Estate Note / Status

| Business line | SWIFT message set | Estate note | Status |
|---|---|---|---|
| Cross-border payments | MT103, MT202 → pacs.008 (CBPR+) ✅ | Alliance-Access-class connectivity ⚠ inferred; SWIFT membership ✅ | ✅/⚠ |
| Trade finance (LCs) | MT700 ✅ (MT707/MT720 class ⚠-knowledge) | LC issuance over MT700 ✅; platform vendor non-public ⚠ | ✅/⚠ |
| Cash management | MT940, camt.053 ✅ | Statements inbound/outbound; the corporate cash-management layer non-public ⚠ | ✅/⚠ |
| Markets post-trade | MT300/304, MT54x, MT5xx, MT202/MT103 ✅ | The connectivity hub — SWIFT Alliance-class or a vendor hub ✅ (capital_markets §7.3) | ✅ |
| Securities services | MT5xx class ✅ | CACEIS entity ✅; systems ⚠ | ✅/⚠ |

### 7.3 The Payments Angle — the Serial and the Cover

The payments estate runs the classic cross-border pattern, cross-referenced rather than re-derived: the **serial** MT103 (customer credit transfer, bank to bank down the chain) and the **cover** MT202 (the bank-to-bank transfer that moves the money) ✅ (payment_rails_guide.md §2.5), with **pacs.008** as the CBPR+ MX equivalent and **pacs.009** for cover ✅ (iso_20022_core_processes_guide.md §12). Under CBPR+, the UETR must survive the journey end-to-end ✅ (iso_20022 guide §9.3), which makes the connectivity layer's translation discipline a *payments* control, not just a plumbing concern: an MT↔MX translation that drops or mangles the UETR breaks gpi tracking ✅/⚠ (UETR preservation is documented ✅; the consequence framing is analysis ⚠). The payments hub — cross-ref [payments_hub_guide.md](payments_hub_guide.md) — is the natural owner of the canonical format; the Alliance estate is the rail adapter at the edge ✅/⚠ (hub architecture cross-referenced ✅; per-bank integration ⚠).

### 7.4 The Capital-Markets Angle — the Post-Trade Language

[capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) §7.3 is the sibling anchor: "Where FIX owns the front of the lifecycle, SWIFT owns the back: confirmation, settlement instructions, and the payments underneath" ✅. Three claims carry over into this guide's ledger:

1. **FIN (ISO 15022) is the legacy external language** — MT300/MT304, MT54x, MT5xx, MT202/MT103 ✅ (capital_markets §7.3).
2. **ISO 20022 is the modern language, and the bank is in a dual-standard world** — MT and MX in parallel, "which is exactly the translation-layer problem every architect must budget for" ✅ (capital_markets §7.3).
3. **The connectivity hub is the translation layer** — "the bank's connectivity hub (SWIFT Alliance-class, or a vendor hub) translates between internal canonical messages, MT, and MX; the golden-copy data model is what makes the translation lossless" ✅ (capital_markets §7.3) — the sentence this whole guide is the deep-dive for.

The markets angle also sets the delivery-guarantee bar: a settlement instruction must not be lost, duplicated, or reordered ✅ (capital_markets §7.3) — the queueing discipline of §6.2 is a *markets* control, not a payments one.

### 7.5 The Estate's SWIFT Inventory — the One-Page Map

Pulling §7.1–§7.4 together into the one-page inventory an architect carries into the §8 design:

| Estate element | Class / status | SWIFT-connectivity consequence |
|---|---|---|
| SWIFT membership | ✅ verified | The precondition for everything ✅ |
| SWIFT connectivity | Alliance-Access-class ⚠ inferred | §8 designs the class, vendor-neutral ✅/⚠ |
| Payments (cross-border) | MT103/MT202 → pacs.008 ✅ | The dual-standard estate is mandatory post-22 Nov 2025 ✅ |
| Trade finance (LCs) | MT700 issuance ✅; platform ⚠ | The RMA Plus LC-only pattern applies ✅ |
| Markets post-trade | MT300/304, MT54x, MT5xx, MT202/MT103 ✅ | The delivery-guarantee bar: no loss, no duplication, no reordering ✅ |
| Securities services | CACEIS entity ✅; systems ⚠ | MT5xx-class flows, servicing arm ✅/⚠ |
| The Singapore hub | Asia-Pacific hub ✅/⚠ | The natural site for the §8 design ✅/⚠ |
| The honest boundary | Franchise documented, inventory not ⚠ | Every named system stays class-level or flagged ⚠ |

The map is deliberately short: the estate guide's verdict — franchise documented, systems non-public — means the *SWIFT-connectivity* facts are the most solid part of the estate picture (membership ✅, MT700 ✅, the message classes ✅), and the *platform* facts are the weakest (vendor identity ⚠). The §8 design leans on the solid column and flags the weak one.

---

## 8. The Worked Example — A SWIFT-Connectivity Design

### 8.1 The Scenario

Cymbal Bank's Singapore hub is refreshing its SWIFT-connectivity estate (the estate of §7). The design brief, in this guide's worked-example framing — the numbers are illustrative ⚠ unless a source is cited:

- **Payments**: outbound MT103 (serial) and pacs.008 (CBPR+) to ~120 correspondent relationships ⚠ (illustrative count), and inbound MT103 from the Asia correspondent book ✅/⚠ (the franchise is real ✅; the count is the example's ⚠).
- **Trade**: MT700 LC issuance to beneficiary banks — the estate's documented franchise ✅.
- **Counterparty control**: RMA/RMA Plus across the correspondent book ✅ (swift.com RMA paper — the LC-only RMA Plus pattern is the paper's own example ✅).
- **The CBPR+ deadline**: the end of MT↔MX coexistence for in-scope FI-to-FI payment instructions on **22 November 2025** ✅ (iso_20022 guide) — the design must carry both standards from day one.
- **Resilience**: no single point of failure; a primary site and a DR site.
- **The estate constraint**: the connectivity class is Alliance-Access-class ⚠ (estate guide) — the design is class-level and vendor-neutral, and every component below is mapped to a verified swift.com fact so the class choice is auditable.

### 8.2 The Design — the Alliance Access Deployment

| Component | Role | Design decision | Verified basis |
|---|---|---|---|
| Alliance Access — node 1 (Singapore) | The messaging interface and the secure hub: validation, authorisation, routing, RMA ✅ | Linux, Full Integration Package (IBM MQ) to the payments hub; file channel to trade finance ⚠ (platform non-public) | swift.com (platforms ✅, packages ✅) |
| Alliance Access — node 2 (DR site) | Same role, DR ✅ | Warm standby; Database Recovery licence for full recovery ✅ | swift.com (resilience ✅) |
| Alliance Gateway — pair in the DMZ | The single secure entry point to SWIFTNet; FIN/InterAct/FileAct concentration ✅ | Between two firewalls ✅; automatic switchover between instances, cross-site ✅ | swift.com (Gateway page ✅) |
| SNL + HSM | The network session and the PKI signing ✅ | HSM holds the SWIFTNet PKI certificates and generates signatures ✅ | Microsoft Learn ✅; swift.com HSM ✅ |
| Routing points | FIN for MT, InterAct for CBPR+, FileAct for statements/files ✅ | One routing point per service class; queues per counterparty ✅ | swift.com (routing ✅) |
| RMA/RMA Plus | The counterparty access control ✅ | RMA Plus for the LC-only correspondents; RMA for the rest ✅ | swift.com RMA paper ✅ |

The deployment reads top to bottom as the three-zone picture of §1.2: the back-office zone (payments hub, trade finance) hands instructions to the interface zone (the two Alliance Access nodes), which routes through the network zone (the Gateway pair, the SNL, the HSM) onto SWIFTNet.

### 8.3 The RMA Setup

| Counterparty class | RMA type | Messages allowed | Rationale |
|---|---|---|---|
| Payments correspondents (the ~120 ⚠) | RMA ✅ | FIN payments (MT103/MT202) + InterAct MX (pacs.008) ✅ | The standard correspondent book ✅ |
| LC-only correspondents | RMA Plus ✅ | MT700 only ✅ | "an institution might only wish to receive letters of credit from a particular correspondent" ✅ (swift.com RMA paper — the paper's own example) |
| Non-customer counterparties | RMA (non-customer) ✅ | As agreed ✅ | "RMAs can also be established with non-customers" ✅ (swift.com) |
| Dormant relationships | RMA revocation ⚠ | None | Dormant-RMA rationalisation is a fraud and cost control ✅ (swift.com); the quarterly review cadence is the example's ops choice ⚠ |

The mechanics: RMA is mandatory for end-to-end-signed user-to-user FIN messages ✅ (swift.com); unwanted traffic is blocked at the sender level ✅ (swift.com) — so the bank's own RMA table is enforced by the *counterparties'* interfaces for inbound, and by the bank's own interface for outbound ✅/⚠ (the enforcement point for outbound is the bank's interface enforcing the counterparty's authorisations — the swift.com "blocked at the sender level" model ✅; the two-direction reading is analysis ⚠).

### 8.4 The Message Flows

**Flow A — outbound MT103 (payments, serial):**

| Step | Component | Control |
|---|---|---|
| 1. Initiation | Payments hub → IBM MQ → Alliance Access | Full Integration Package ✅ (swift.com) |
| 2. Validation | Alliance Access | Format and field validation (MT) ✅ (swift.com) |
| 3. Authorisation | Alliance Access | Four-eyes release ✅/⚠ (authorisation feature ✅; workflow config ⚠) |
| 4. RMA check | Alliance Access | Counterparty must hold the bank's RMA; else blocked at sender ✅ (swift.com) |
| 5. Routing | Routing point → FIN queue | Routing points and queues ✅ (swift.com) |
| 6. Transport | Alliance Gateway → SNL → HSM → SWIFTNet | PKI signature; store-and-forward delivery ✅ (swift.com; Microsoft Learn) |

**Flow B — outbound pacs.008 (payments, CBPR+):** the same spine, but the routing point targets **InterAct** (store-and-forward MX) ✅ (Oracle; iso_20022 guide), the message carries the **BAH** and the **UETR** ✅ (iso_20022 guide), and the RMA gate applies to the MX flow under the same authorisation model ✅/⚠ (RMA is documented for FIN explicitly ✅; its application to MX traffic is the industry model ⚠-knowledge).

**Flow C — outbound MT700 (trade):**

| Step | Component | Control |
|---|---|---|
| 1. Initiation | Trade finance → file channel → Alliance Access | File-based integration ✅ (swift.com Basic Integration Package) |
| 2. Template + validation | Alliance Access | LC template creation, MT700 format validation ✅ (swift.com) |
| 3. Authorisation | Alliance Access | Four-eyes — the LC is a commitment instrument ✅/⚠ |
| 4. RMA check | Alliance Access | The beneficiary bank must hold the bank's RMA; RMA Plus gates the MT700 class ✅ (swift.com) |
| 5. Transport | FIN → Gateway → SWIFTNet | Store-and-forward to the beneficiary bank ✅ |

**Flow D — inbound MT103:** the mirror — correspondent → SWIFTNet → Gateway → Alliance Access → validation → sanctions screening ⚠ (screening is a compliance control; the swift.com ecosystem references Transaction Screening ✅/⚠ — the specific screening vendor is non-public ⚠) → payments hub booking.

### 8.5 The Sequence — Three Transactions, End to End

1. **09:14 — the MT103.** A Singapore corporate instructs a USD payment to a Jakarta supplier. The payments hub builds the instruction, MQ delivers it to Alliance Access, validation passes, an operator authorises, the RMA check confirms the Jakarta correspondent's authorisation, the message enters the FIN queue, the Gateway session signs and sends. The correspondent's interface accepts it into its store-and-forward queue; the money moves through the nostro chain (§7.3) ✅ (component-by-component: swift.com).
2. **10:02 — the pacs.008.** The same corporate's euro payment to Frankfurt rides CBPR+: Alliance Access builds the MX with the BAH and the UETR, the InterAct routing point carries it store-and-forward, and the gpi tracker sees the UETR at every hop ✅ (iso_20022 guide §9.3, §13; payment_rails_guide.md).
3. **14:30 — the MT700.** Trade finance issues an import LC for a turbine supplier's bank. The file channel delivers the LC data, the template fills the MT700, two operators authorise, the RMA Plus gate confirms the beneficiary bank is authorised for the MT700 class only, and FIN delivers the LC ✅ (swift.com RMA paper — the LC-only example; estate guide — the MT700 franchise).

### 8.6 The Lessons

- **The translation layer is the design.** MT↔MX is lossy and the golden copy is what makes it lossless ✅ (capital_markets §7.3); the Alliance estate must be configured around the canonical model, not around the two formats.
- **RMA hygiene is a fraud control.** Dormant authorisations create "the opportunity for payments to be sent to destinations which may no longer be wanted or authorised" ✅ (swift.com) — the RMA table is a risk asset, reviewed, not archived.
- **The cloud path is Alliance Cloud, not Lite2.** Lite2 is closed to new clients ✅ (swift.com); a 2026 design that wants zero footprint buys Alliance Cloud and connects the public-cloud back office through Alliance Connect Virtual ✅ (swift.com).
- **The secure zone moves with the deployment** ✅/⚠ (AWS whitepaper) — the DR site is not an exception to the zone model.
- **The 22 November 2025 deadline is already in the past tense for new builds** ✅ (iso_20022 guide) — the dual-standard estate is not a migration project; it is the baseline.

### 8.7 The Design Checklist — the Audit Trail of the Example

The worked example closes with the checklist an architect would hand to a vendor or an internal build team — every line maps to a verified claim from this guide's ledger, so the class-level design is auditable end to end:

| # | Check | Verified basis |
|---|---|---|
| 1 | The interface validates MT **and** MX formats and fields | ✅ swift.com (Alliance Access) |
| 2 | The authorisation workflow enforces a second signature before release | ✅/⚠ swift.com (authorisation feature ✅; config ⚠) |
| 3 | The RMA table is enforced at the sender level for FIN | ✅ swift.com (RMA paper) |
| 4 | RMA Plus gates the LC-only correspondents to the MT700 class | ✅ swift.com (RMA paper — the paper's own example) |
| 5 | Routing points separate FIN, InterAct and FileAct flows | ✅ swift.com (routing) |
| 6 | The Gateway pair sits between two firewalls, with automatic switching | ✅ swift.com (Gateway page) |
| 7 | The HSM holds the SWIFTNet PKI certificates and signs all traffic | ✅ swift.com (HSM) |
| 8 | The UETR survives the MT↔MX translation for gpi payments | ✅ iso_20022 guide §9.3 (UETR ✅); ⚠ the preservation mechanism is the bank's translation design |
| 9 | The CBPR+ flows carry the BAH and run over InterAct | ✅ iso_20022 guide §13 |
| 10 | Every component lives in the CSP secure zone | ✅/⚠ AWS whitepaper |
| 11 | The queue discipline prevents loss, duplication and reordering | ✅ capital_markets §7.3; cross-ref the Message Queue Data Loss Guide |
| 12 | The cloud path, if chosen, is Alliance Cloud — not Lite2 | ✅ swift.com (Lite2 page) |

A design that passes all twelve checks is an Alliance-class estate built entirely from verified facts — the only ⚠ items left are the quantities (the correspondent count), the vendor identities (the trade platform, the screening tool), and the configuration specifics (the workflow rules), all flagged rather than guessed.

---

## 9. The Summary

### 9.1 The One-Page Summary

| The question | The one-line answer | Verified in |
|---|---|---|
| What is the connectivity layer? | The estate between the back office and SWIFTNet — interfaces, gateway, HSM, operations ✅ | §1 |
| What is Alliance Access? | The messaging interface and the secure hub: validate, authorise, route, repair, monitor ✅ | §2 |
| What is Alliance Gateway? | The single secure entry point to SWIFTNet, concentrating FIN/InterAct/FileAct/WebAccess ✅ | §2 |
| What happened to Lite2? | Closed to new clients; Alliance Cloud (2020) is the successor ✅ | §2 |
| Which standards ride the lane? | FIN MT and MX/ISO 20022 — dual-standard by mandate since 22 Nov 2025 ✅ | §3 |
| What gates the counterparties? | RMA / RMA Plus, blocked at the sender level ✅ | §4 |
| What signs the traffic? | The HSM, under SWIFTNet PKI — authentication, integrity, non-repudiation ✅ | §4 |
| On-prem or cloud? | On-prem Alliance Access, or Alliance Cloud; Lite2 is not a target ✅ | §5 |
| What does the ops room watch? | Queues, delivery notifications, the gpi tracker, RMA health ✅ | §6 |
| Whose estate is this? | The Cymbal Bank estate — membership ✅, Alliance-Access class ⚠, MT700 ✅ | §7 |
| How does it all fit? | The §8 worked example — Access + Gateway + HSM + RMA table + the three flows ✅ | §8 |

The SWIFT connectivity layer is where a bank's cross-border ambitions meet the network's reality: a private backbone (SWIFTNet) that no one reaches over the public internet ✅; a messaging interface (Alliance Access) that validates, authorises, routes and repairs — the secure hub of the estate ✅; a network gateway (Alliance Gateway) that concentrates every flow at a single secure entry point ✅; and a cloud option that has already moved on from Lite2 to Alliance Cloud ✅. The messaging is dual-standard by law of the calendar — FIN MT for the legacy flows and the trade instruments, MX/ISO 20022 for the CBPR+ payments, with the MT103↔pacs.008 translation as the eternal design problem ✅. The security is layered: SWIFTNet PKI signed by an HSM, two-factor operator access, and the RMA — the SWIFT-mandated counterparty authorisation that blocks unwanted FIN at the sender level and, in its RMA Plus form, gates whole message classes like the LC-only MT700 ✅. The operations are queue discipline and monitoring, with gpi riding on top and the UETR as the thread that must never break ✅. For the Cymbal Bank estate, the class is Alliance-Access-class ⚠, the franchise is documented, and the design discipline is the same one this guide has followed: verify what can be verified, flag what cannot, and never fabricate. The worked example assembled the estate from verified components — two Access nodes, a Gateway pair, an HSM, routing points, and an RMA table with a place for every counterparty — and ran three transactions through it end to end. When the design is done, the estate reduces to a single sentence: the bank's cross-border future runs on the secure lane

---

## 10. Glossary

| Term | Definition |
|---|---|
| SWIFT | The Society for Worldwide Interbank Financial Telecommunication — the member-owned cooperative running the private messaging network for cross-border financial communication; founded 3 May 1973 ✅ |
| Alliance Access | SWIFT's messaging interface — the on-premise secure hub: message creation, validation, authorisation, repair, routing, monitoring; RMA built in ✅ |
| Alliance Gateway | The single secure entry point to SWIFTNet, concentrating FIN/InterAct/FileAct/WebAccess flows via RAHA/MQHA/TDA; DMZ-deployable ✅ |
| Alliance Lite2 | SWIFT's cloud-based managed access; available for existing clients only, with Alliance Cloud as the successor ✅ |
| SWIFTNet | SWIFT's private, secure IP network; reachable only through the mandatory SWIFTNet Link (SNL) interface ✅ |
| FIN | The store-and-forward messaging service carrying the legacy MT (ISO 15022) message set ✅ |
| MT | Message Type — the legacy SWIFT message format (MT103, MT202, MT700, MT940…) ✅ |
| MX | The ISO 20022 XML message format (pacs.008, camt.053…) ✅ |
| ISO 20022 | The global financial messaging standard; SWIFT is the Registration Authority ✅ |
| RMA | Relationship Management Application — the SWIFT-mandated authorisation controlling which counterparties can send you FIN messages; blocked at the sender level ✅ |
| PKI | Public Key Infrastructure — SWIFTNet's certificate-based authentication, integrity and non-repudiation; the keys live in the HSM ✅ |
| STP | Straight-through processing — instructions processed end-to-end without manual re-keying ✅ |
| gpi | SWIFT Global Payments Innovation — the payments service (live January 2017) with end-to-end tracking via the UETR ✅ |
| message flow | The path an instruction takes from the back office through the interface, the gateway and SWIFTNet to the counterparty — and back ✅ |
| back office | The bank's processing systems — the payments hub, trade finance, core banking — that generate and consume SWIFT messages ✅ |
| connectivity | The layer of interfaces, gateways, network links and security devices that connect a bank's systems to SWIFTNet ✅ |

---

## 11. Claims Audit

| Claim | Marker | Basis |
|---|---|---|
| SWIFT founded 3 May 1973 | ✅ | payment_rails_guide.md ledger (cross-referenced) |
| SWIFT is a messaging cooperative, not a settlement system | ✅ | payment_rails_guide.md §2.5; swift.com RMA paper (11,000+/200+ ✅) |
| FIN messaging live 1977 | ⚠ | ⚠-knowledge, not re-verified this pass |
| Alliance Access is SWIFT's leading messaging interface; MT+MX, FIN/InterAct/FileAct; 2,000+ installations; Windows/Linux/AIX; message lifecycle; routing points and queues; audit trail; Basic/Full Integration Packages; 24×7; Gateway switchover; Database Recovery | ✅ | swift.com Alliance Access page, extracted 2026-08-27 |
| Alliance Gateway is a single secure entry point to SWIFTNet; DMZ; RAHA/MQHA/TDA; >100 tps; four-eyes admin; LDAP; SNMP; integrity verification; Developers Toolkit | ✅ | swift.com Alliance Gateway page, extracted 2026-08-27 |
| Alliance Gateway current production version (7.7) | ⚠ | swift.com knowledge-centre document reference only |
| Alliance Lite2 is available for existing clients only | ✅ | swift.com Alliance Lite2 page, extracted 2026-08-27 |
| Alliance Cloud launched 2020, the preferred messaging interface; 6-week upgrades; REST API; Alliance Connect Virtual; multi-active centres | ✅ | swift.com Alliance Lite2 page FAQ |
| Alliance Lite2 launch year (~2010) | ⚠ | ⚠-knowledge, not re-verified this pass |
| SWIFTNet is a private secure IP network; SNL is the mandatory interface; four services (InterAct, FileAct, Browse, FIN) | ✅ | Microsoft Learn; Oracle; swift.com SwiftNet Link page |
| FileAct: push/download, store-and-forward, delivery notifications/non-delivery warnings, file header, FileAct Copy, compression, segregation, open formats | ✅ | swift.com FileAct factsheet |
| SWIFTNet availability above 99.99% | ✅/⚠ | swift.com FileAct factsheet — vendor claim, marked as such |
| SWIFTNet PKI: authentication, integrity, non-repudiation | ✅ | swift.com FileAct factsheet |
| HSM holds the SWIFTNet PKI certificates and generates signatures | ✅ | swift.com HSM product page |
| 2FA campaign in the interface products under the CSP | ✅ | swift.com news item |
| CSP secure zone mandate for the connectivity components | ✅/⚠ | AWS whitepaper (2021 framework); current framework version ⚠ |
| RMA: SWIFT-mandated FIN authorisation; blocked at sender; introduced 2009 replacing BKE; mandatory for end-to-end-signed user messages; non-customer RMAs; built into Alliance Access/Entry; standalone Alliance RMA | ✅ | swift.com RMA information paper (July 2016) |
| RMA Plus: message-type-level authorisation (the LC-only example) | ✅ | swift.com RMA information paper |
| Wolfsberg RMA due-diligence guidance | ✅ | Wolfsberg Group; swift.com RMA paper |
| gpi live January 2017 | ✅ | payment_rails_guide.md ledger; Coupa press ✅/⚠ |
| UETR (UUID RFC 4122, 32 hex) + gpi tracker | ✅ | iso_20022_core_processes_guide.md §9.3 |
| CBPR+ coexistence ends 22 Nov 2025; BAH mandatory; MT103↔pacs.008 lossy | ✅ | iso_20022_core_processes_guide.md §13, §9.3 |
| Cymbal Bank: SWIFT membership ✅; Alliance-Access-class connectivity ⚠; MT700 LC issuance ✅ | ✅/⚠ | credit_agricole_software_systems_guide.md §3 |
| The connectivity hub — SWIFT Alliance-class or a vendor hub; the golden-copy translation layer | ✅ | capital_markets_architecture_guide.md §7.3 |
| SAP↔Alliance Access certified adapter set | ⚠ | not verifiable this pass |
| gpi market share statistics (press "over 50%") | ⚠ | press-only; not asserted |
| Contradicted claims this pass | ❌ none | every claim checked either verified ✅ or flagged ⚠ |

---

## 12. What Could Not Be Verified

The honest list of everything flagged in this pass — none of it is asserted as fact anywhere in this guide:

1. **The Alliance Lite2 launch year (~2010)** ⚠-knowledge — swift.com's current Lite2 page is migration-focused and does not state it; the year is well-documented industry knowledge that could not be re-verified live.
2. **The Alliance Gateway current production version** ⚠ — the swift.com knowledge-centre resource links reference "Alliance Gateway 7.7" documents, but that is a document reference, not a confirmed live version.
3. **The SAP↔Alliance Access certified adapter set** ⚠ — enterprise back-office integration (SAP S/4HANA Payment Medium Workbench, treasury modules) is standard industry practice, but no primary source was extracted this pass; flagged rather than guessed.
4. **The Cymbal Bank SWIFT vendor identity** ⚠ — the estate guide records "Alliance-Access-class inferred"; the vendor is non-public. This guide designs the class, never a named vendor.
5. **"Secure hub" as a SWIFT product name** ⚠ — the term is this guide's architectural framing of the verified Alliance Access facts, not SWIFT branding.
6. **The SWIFTNet 99.99% availability figure** ✅/⚠ — a vendor claim from the swift.com FileAct factsheet; marked as such, not audited.
7. **The gpi market-share statistics** ⚠ — press-only ("over 50% of cross-border SWIFT payments"); not asserted here.
8. **The worked-example numbers** ⚠ — the ~120 correspondent count and the quarterly RMA review cadence are the example's illustrative choices; every *component* fact is verified, the *quantities* are not.
9. **Alliance Messaging Hub details** ⚠ — the product appears in swift.com ecosystem results and vendor guidance (AWS/Azure), but its page was not extracted this pass; mentioned by name only.
10. **The current CSP/CSCF framework version** ⚠-knowledge — the AWS whitepaper documents the 2021 framework; SWIFT's framework versioning evolves and was not re-verified.
11. **FIN live in 1977** ⚠-knowledge — standard industry history, not re-verified this pass.
12. **The CBPR+ November 2022 go-live** ⚠-knowledge — the *coexistence end date* (22 Nov 2025) is ✅ in the ISO 20022 guide's ledger; the go-live year is ⚠-knowledge.
13. **The 2FA mechanics per product** ⚠ — the campaign's existence is verified; the per-product configuration is not public.
14. **The RMA enforcement direction for outbound MX flows** ⚠/⚠-knowledge — RMA is documented for FIN explicitly; its application to the bank's outbound MX flows is the industry model, flagged as such.

---

## 13. Verification Ledger

| Claim | Source | Status | Date |
|---|---|---|---|
| Alliance Access: leading messaging interface; MT/MX; FIN/InterAct/FileAct; 2,000+ installations; Windows/Linux/AIX; message lifecycle; routing points and queues; audit trail; Basic/Full Integration Packages; 24×7; Gateway switchover; Database Recovery | swift.com product page (alliance-access) | ✅ | 2026-08-27 |
| Alliance Gateway: single secure entry point; DMZ; RAHA/MQHA/TDA; >100 tps; four-eyes; LDAP; SNMP; integrity verification; Developers Toolkit | swift.com product page (alliance-gateway) | ✅ | 2026-08-27 |
| Alliance Lite2: existing clients only; no EOL; Alliance Cloud launched 2020; 6-week upgrades; REST API; Alliance Connect Virtual; multi-active centres | swift.com product page (alliance-lite2) | ✅ | 2026-08-27 |
| HSM: holds the SWIFTNet PKI certificates, generates signatures | swift.com HSM product page (search-verified) | ✅ | 2026-08-27 |
| FileAct: push/download; store-and-forward; notifications; file header; FileAct Copy; compression; segregation; PKI non-repudiation; 99.99% availability claim | swift.com FileAct factsheet (PDF) | ✅ (availability ✅/⚠) | 2026-08-27 |
| RMA: SWIFT-mandated; blocked at sender; 2009 replacing BKE; mandatory for end-to-end-signed user messages; non-customer RMAs; built into Alliance Access/Entry; Alliance RMA standalone; RMA Plus message-type granularity (LC-only example); dormant-RMA fraud risk; Wolfsberg guidance | swift.com "RMA and RMA Plus: managing your correspondent connections" information paper (PDF, July 2016) | ✅ | 2026-08-27 |
| SWIFTNet: private secure IP network; SNL mandatory; four services (InterAct, FileAct, Browse, FIN) | Microsoft Learn; Oracle SWIFTNet documentation; swift.com SwiftNet Link page (search-verified) | ✅ | 2026-08-27 |
| CSP secure zone for the connectivity components | AWS whitepaper — SWIFT Customer Security Controls Framework (2021) | ✅/⚠ | 2026-08-27 |
| gpi live January 2017 | payment_rails_guide.md ledger (SWIFT press release 16 Feb 2017); Coupa press | ✅ (press ✅/⚠) | ledger / 2026-08-27 |
| UETR (UUID RFC 4122, 32 hex) + gpi tracker; InterAct/FileAct transports; BAH mandatory | iso_20022_core_processes_guide.md §9.3, §13 | ✅ | ledger |
| CBPR+ MT↔MX coexistence ends 22 Nov 2025; MT103↔pacs.008 lossy | iso_20022_core_processes_guide.md §13, §17 | ✅ | ledger |
| SWIFT founded 3 May 1973; MT103/MT202/MT700/MT940 classes; SWIFT a messaging cooperative | payment_rails_guide.md ledger, §2.5 | ✅ | ledger |
| The connectivity hub — SWIFT Alliance-class or a vendor hub; the golden-copy translation layer; MT300/304, MT54x, MT5xx | capital_markets_architecture_guide.md §7.3 | ✅ | ledger |
| Cymbal Bank: SWIFT membership ✅; Alliance-Access-class inferred ⚠; MT700 LC issuance ✅ | credit_agricole_software_systems_guide.md §3 | ✅/⚠ | ledger |
| SWIFT scale: 11,000+ organisations, 200+ countries | swift.com RMA information paper (corroborates the repo's ⚠-knowledge figure) | ✅ | 2026-08-27 |
| Alliance Lite2 launch year (~2010) | not re-verified this pass | ⚠ | 2026-08-27 |
| Alliance Gateway 7.7 as a live version | swift.com knowledge-centre document reference only | ⚠ | 2026-08-27 |
| SAP↔Alliance Access certified adapters | no primary source extracted | ⚠ | 2026-08-27 |
| FIN live 1977; CBPR+ go-live Nov 2022 | not re-verified this pass | ⚠-knowledge | 2026-08-27 |
| gpi market share ("over 50%") | press-only (ohmyfin) | ⚠ | 2026-08-27 |
