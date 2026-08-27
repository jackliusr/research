# SWIFTNet FileAct: The Reliable File Lane — A Comprehensive Guide

**The File Lane of the SWIFTNet Estate — What SWIFTNet FileAct Is and Where It Sits, the SWIFTNet Platform Context (SWIFTNet Link, SWIFTNet PKI, the Relationship Management Application), the FileAct Mechanics (the Transfer Envelope, Store-and-Forward, Acknowledgements, Retry and Recovery, File Size Limits, Scheduling, Compression, Encryption), the Delivery Products (SWIFT Alliance Gateway, Alliance Access, Alliance Lite2), the Use Cases (Bulk Payments, Statements, Trade Documents, Treasury Confirmations, Regulatory Reporting, Market-Infrastructure Flows), the Head-to-Head Comparisons (FileAct vs InterAct vs FIN; FileAct vs SFTP/FTPS, IBM MQ and Cloud File Exchange), a Cymbal Bank Statement-File Worked Example with a Failure-and-Retry Scenario, the Operations and Pricing, and the One-Page Summary — from the 1990s SWIFTNet Programme to the 2026 FileAct Service as the File Lane of the Payments and Messaging Cluster**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Payments and Messaging Infrastructure — SWIFTNet FileAct: definition and position (SWIFT's secure store-and-forward file-transfer service over SWIFTNet, the FILE lane alongside FIN's message lane and InterAct's real-time XML lane), the SWIFTNet platform context (SWIFTNet Link as the mandatory network interface, SWIFTNet PKI and the Relationship Management Application for counterparty permissions), the FileAct mechanics (the file transfer envelope, store-and-forward and real-time modes, delivery notifications and non-repudiation, file naming, retry/recovery, the 250 MB file-size limit, scheduled vs on-demand transfers, compression and encryption), the delivery products (SWIFT Alliance Gateway, Alliance Access, Alliance Lite2 — positioned as delivery vehicles, not re-derived), the use cases (bulk payments, MT940/MT950 and camt.053/camt.054 statements, trade finance documents, treasury confirmations, bulk regulatory reporting, CLS and market-infrastructure flows), the comparisons (FileAct vs InterAct vs FIN; FileAct vs SFTP/FTPS, IBM MQ and cloud file exchange), the worked example (a Cymbal Bank statement-file flow, bank → corporate and counterparty → bank, with envelope, schedule, delivery, ack and a failure-plus-retry scenario), the operations, availability and pricing, and the one-page summary
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** SWIFT's own pages and documents (swift.com — FileAct product page, FIN product page, InterAct product page, SwiftNet Link product page, the FileAct factsheet, the FileAct Copy factsheet, Swift for Treasury Market Infrastructures / CLS page, Swift Web Access page, ISO 20022 corporates page), SWIFT-document-derived integration documentation (Microsoft Learn — the BizTalk FileAct and InterAct adapters and "What Is SWIFTNet?", which mirror the SWIFTNet service descriptions), the official SWIFTNet FileAct procedural rules as published by the Deutsche Bundesbank (2016 v1.7 and 2024-03 versions — the source of the 250 MB per-file limit), the European Central Bank's communication on the Browse-to-WebAccess migration, bank and vendor documentation (NAB's SWIFT FileAct fact sheet, IBM and Axway SWIFTNet documentation, Goldman Sachs developer documentation), and the repo's sibling guides (cross-referenced heavily). NOTE: this pass had **live web access** (self-hosted Firecrawl backend) on 2026-08-27 — the definitions, mechanics and figures below were verified against the primary sources listed above in this pass; the search backend degraded intermittently, so a few items were verified by direct URL extraction of primary documents instead; anything that could not be re-verified live is flagged ⚠-knowledge honestly.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the payments/messaging cluster):** [SWIFT Alliance Access](swift_alliance_access_guide.md) (the Alliance-family connectivity deep-dive, produced in the same batch — this guide positions FileAct as its FILE lane companion; cross-ref §4), [Payment Rails](payment_rails_guide.md) (the rails map — SWIFT and gpi as the cross-border rail — cross-ref heavily), [Payments Hub](payments_hub_guide.md) (the hub architecture and the rail adapters — where the FileAct lane plugs in — cross-ref heavily), [Banks in Singapore](banks_in_singapore_guide.md) (the SG rails and SWIFT usage at SG banks — cross-ref the worked example), [MAS Regulations and Guidelines](mas_regulations_guidelines_guide.md) (the MAS regulatory-reporting angle — cross-ref §5.6), [Capital Markets Architecture](capital_markets_architecture_guide.md) (the message standards and market-infrastructure flows — cross-ref lightly), [Treasury & ALM](treasury_alm_guide.md) (the treasury-confirmation and liquidity angle — cross-ref lightly), [Posting Rules Mechanics](posting_rules_mechanics_guide.md) (the statement-to-posting flow — cross-ref §5.2 and §7), [Payment Rails](payment_rails_guide.md) is the rail-selection sibling and [Payments Hub](payments_hub_guide.md) the architecture sibling — the Alliance-family delivery products (Alliance Gateway, Alliance Access, Alliance Lite2, Alliance Cloud) are treated in this guide ONLY as FileAct delivery vehicles and are cross-referenced, not re-derived
> **Companion guides (technology/, prefix `../technology/`):** [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (the 99.99% availability claims and cut-over windows — cross-ref §8.2), [Message Queue Data Loss](../technology/message_queue_data_loss_guide.md) (the at-least-once delivery and ack semantics — cross-ref §3.3 and §3.5), [Legacy Integration Patterns](../technology/legacy_integration_patterns_guide.md) (the SAG/SNL adapter patterns — cross-ref §4), [Security by Design](../technology/security_by_design_guide.md) (the PKI and non-repudiation angle — cross-ref §3.8)

---

**How to use this guide:** Section 1 is the overview — the definition, the short answer and the overview table. Section 2 is the SWIFTNet context — the platform, SWIFTNet Link, SWIFTNet PKI, the Relationship Management Application and the three SWIFTNet services (FIN, InterAct, FileAct), each verified. Section 3 is the FileAct mechanics — the transfer envelope, store-and-forward vs real-time delivery, end-to-end acknowledgements, file naming conventions, retry and recovery, the file size limits, scheduled vs on-demand transfers, and compression and encryption. Section 4 is the delivery products — SWIFT Alliance Gateway, Alliance Access, Alliance Lite2 and the SWIFTReady third-party interfaces, positioned as delivery vehicles for the file lane and cross-referenced to the repo's payments cluster rather than re-derived. Section 5 is the use cases — bulk payments, statements (MT940/MT950, camt.053/camt.054), trade finance documents, treasury confirmations, bulk regulatory reporting and market-infrastructure flows. Section 6 is the comparisons — FileAct vs InterAct vs FIN, and FileAct vs SFTP/FTPS, IBM MQ and cloud file exchange, with the head-to-head table. Section 7 is the Cymbal Bank worked example — a statement-file flow (bank → corporate, and a counterparty → bank variant) with the envelope, schedule, delivery, ack, and a failure-plus-retry scenario, plus where the file lane sits in a bank's SWIFT estate. Section 8 is the operations — availability, monitoring, load balancing, segregation, pricing and the FileAct Copy service. Section 9 is the one-page summary — the final word is "the reliable file lane." The claims audit, the "What Could Not Be Verified" section, the glossary and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/` and `management/` guides `../management/`. **Integrity convention:** ✅ = verified this pass against a primary source (a swift.com page or document, an official SWIFT procedural-rules document, or SWIFT-document-derived integration documentation); ⚠ = flagged/unverified or approximate; ⚠-knowledge = well-documented industry knowledge that could not be re-verified live in this pass (the search backend degraded mid-pass and the SWIFT Knowledge Centre service descriptions are authentication-gated) and is flagged honestly.

---

## Table of Contents

1. [The FileAct Overview](#1-the-fileact-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Definition — Verified](#12-the-definition--verified)
   - 1.3 [The Overview Table — Aspect / Description / Status](#13-the-overview-table--aspect--description--status)
   - 1.4 [Reading the Overview Table](#14-reading-the-overview-table)
   - 1.5 [The One-Liner — the File Lane](#15-the-one-liner--the-file-lane)
2. [The SWIFTNet Context](#2-the-swiftnet-context)
   - 2.1 [The SWIFTNet Platform — the Single Window](#21-the-swiftnet-platform--the-single-window)
   - 2.2 [SWIFTNet Link (SNL) — the Mandatory Network Interface](#22-swiftnet-link-snl--the-mandatory-network-interface)
   - 2.3 [SWIFTNet PKI — Certificates and Non-Repudiation](#23-swiftnet-pki--certificates-and-non-repudiation)
   - 2.4 [The Relationship Management Application (RMA) — Counterparty Permissions](#24-the-relationship-management-application-rma--counterparty-permissions)
   - 2.5 [The Three SWIFTNet Services — FIN, InterAct, FileAct](#25-the-three-swiftnet-services--fin-interact-fileact)
   - 2.6 [The Platform Evolution — Browse to WebAccess, SWIFTNet 7.7](#26-the-platform-evolution--browse-to-webaccess-swiftnet-77)
3. [The FileAct Mechanics](#3-the-fileact-mechanics)
   - 3.1 [The File Transfer Envelope](#31-the-file-transfer-envelope)
   - 3.2 [The Delivery Modes — Store-and-Forward vs Real-Time](#32-the-delivery-modes--store-and-forward-vs-real-time)
   - 3.3 [The End-to-End Acknowledgements](#33-the-end-to-end-acknowledgements)
   - 3.4 [The File Naming Conventions](#34-the-file-naming-conventions)
   - 3.5 [The Retry and Recovery Behaviour](#35-the-retry-and-recovery-behaviour)
   - 3.6 [The File Size Limits — 250 MB per Physical File](#36-the-file-size-limits--250-mb-per-physical-file)
   - 3.7 [The Scheduled vs On-Demand Transfers](#37-the-scheduled-vs-on-demand-transfers)
   - 3.8 [The Compression and Encryption](#38-the-compression-and-encryption)
   - 3.9 [The Value-Added Features — FileAct Copy, Segregation, Routing](#39-the-value-added-features--fileact-copy-segregation-routing)
4. [The Delivery Products — How FileAct Reaches the Bank](#4-the-delivery-products--how-fileact-reaches-the-bank)
   - 4.1 [SWIFT Alliance Gateway (SAG) — the Connectivity Hub](#41-swift-alliance-gateway-sag--the-connectivity-hub)
   - 4.2 [Alliance Access — the Messaging Backbone](#42-alliance-access--the-messaging-backbone)
   - 4.3 [Alliance Lite2 and Alliance Cloud — the Managed/Cloud Lane](#43-alliance-lite2-and-alliance-cloud--the-managedcloud-lane)
   - 4.4 [The Third-Party and SWIFTReady Interfaces](#44-the-third-party-and-swiftready-interfaces)
   - 4.5 [Positioning — the File Lane, Not the Alliance Deep-Dive](#45-positioning--the-file-lane-not-the-alliance-deep-dive)
5. [The Use Cases](#5-the-use-cases)
   - 5.1 [The Bulk Payments — Mass Payments and Payroll Files](#51-the-bulk-payments--mass-payments-and-payroll-files)
   - 5.2 [The Account Statements — MT940/MT950 and camt.053/camt.054](#52-the-account-statements--mt940mt950-and-camt053camt054)
   - 5.3 [The Trade Finance Documents](#53-the-trade-finance-documents)
   - 5.4 [The Treasury Confirmations](#54-the-treasury-confirmations)
   - 5.5 [The Regulatory Reporting Bulk Files — FATCA/CRS and MAS](#55-the-regulatory-reporting-bulk-files--fatcacrs-and-mas)
   - 5.6 [The CLS and Market-Infrastructure File Flows](#56-the-cls-and-market-infrastructure-file-flows)
6. [The Comparisons](#6-the-comparisons)
   - 6.1 [FileAct vs InterAct vs FIN — the Three Lanes](#61-fileact-vs-interact-vs-fin--the-three-lanes)
   - 6.2 [FileAct vs SFTP/FTPS — the Private Network vs the Internet](#62-fileact-vs-sftpftps--the-private-network-vs-the-internet)
   - 6.3 [FileAct vs IBM MQ — the File Lane vs the Message Queue](#63-fileact-vs-ibm-mq--the-file-lane-vs-the-message-queue)
   - 6.4 [FileAct vs Cloud File Exchange — the Modern Alternatives](#64-fileact-vs-cloud-file-exchange--the-modern-alternatives)
   - 6.5 [The Head-to-Head Table — Dimension / FileAct / Alternative / Notes](#65-the-head-to-head-table--dimension--fileact--alternative--notes)
   - 6.6 [Reading the Comparison — the Selection Decision Flow](#66-reading-the-comparison--the-selection-decision-flow)
7. [The Worked Example — A Cymbal Bank Statement-File Flow](#7-the-worked-example--a-cymbal-bank-statement-file-flow)
   - 7.1 [The Scenario — Cymbal Bank and the Corporate Client](#71-the-scenario--cymbal-bank-and-the-corporate-client)
   - 7.2 [The Envelope — the FileAct Transfer, Field by Field](#72-the-envelope--the-fileact-transfer-field-by-field)
   - 7.3 [The Schedule — the Batch Windows](#73-the-schedule--the-batch-windows)
   - 7.4 [The Delivery and the Ack — the Happy Path](#74-the-delivery-and-the-ack--the-happy-path)
   - 7.5 [The Failure and the Retry — the Recovery Scenario](#75-the-failure-and-the-retry--the-recovery-scenario)
   - 7.6 [The Counterparty → Bank Variant](#76-the-counterparty--bank-variant)
   - 7.7 [Where the File Lane Sits in the SWIFT Estate](#77-where-the-file-lane-sits-in-the-swift-estate)
8. [The Operations, Availability and Pricing](#8-the-operations-availability-and-pricing)
   - 8.1 [The Availability and the Reliability Claims](#81-the-availability-and-the-reliability-claims)
   - 8.2 [The Monitoring, the Load Balancing and the Pacing](#82-the-monitoring-the-load-balancing-and-the-pacing)
   - 8.3 [The Pricing — the Usage Fee per File](#83-the-pricing--the-usage-fee-per-file)
   - 8.4 [The FileAct Copy in Operations — Y-Copy and T-Copy](#84-the-fileact-copy-in-operations--y-copy-and-t-copy)
   - 8.5 [The Dispute Evidence — the Centralised Log](#85-the-dispute-evidence--the-centralised-log)
9. [The Summary — One Page](#9-the-summary--one-page)
10. [The Claims Audit](#10-the-claims-audit)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [Cross-References and Further Reading](#13-cross-references-and-further-reading)

---

## 1. The FileAct Overview

### 1.1 The Short Answer

**SWIFTNet FileAct is SWIFT's secure store-and-forward file-transfer service over SWIFTNet — the "file lane" of the SWIFT estate, built to move large files of any format (bulk payments, statements, reports, documents) between financial institutions, market infrastructures and corporates with the same PKI security, reliability and legal-evidence properties as SWIFT's message services.** Where FIN moves individual structured MT messages and InterAct moves real-time XML (MX/ISO 20022) request/reply exchanges, FileAct moves *files*: batches of structured financial messages, large reports, images, spreadsheets — any content structure, in any agreed format, wrapped in a FileAct transfer envelope and posted to a counterparty's BIC address over the SWIFTNet secure IP network.

One sentence fixes the concept: **FileAct is to the SWIFT messaging estate what the freight lane is to the postal service** — the channel for *bulk, package-sized payloads* that must arrive intact, provably, and on schedule, as opposed to the letter lane (FIN) and the courier lane (InterAct). For a Solution Architect, FileAct is the answer to a specific question: *"I need to move a 100 MB batch of ISO 20022 payment instructions, or a day's account statements, or a regulatory report, to a counterparty — securely, with delivery evidence, without building a bilateral file-transfer link."*

Three facts fix the landscape:

1. **FileAct is one of the three SWIFTNet services — FIN, InterAct and FileAct.** SWIFT itself describes FileAct as "a secure, efficient solution for large data file transfers, connecting you to a global network" and "ideal for bulk payments and large-scale reporting" ✅ (swift.com, FileAct product page). The FIN page cross-references FileAct as "a single secure channel for large files of structured messages, operational data or reports" ✅ (swift.com, FIN product page).
2. **FileAct is format-agnostic.** The official FileAct factsheet states it "transfers any type of data, such as text, spreadsheet, XML formatted files, and images... you can use SWIFT message formats MTs or MXs, domestic formats, or your own proprietary ones" ✅ (swift.com, FileAct factsheet). This is the property that makes FileAct the *file lane* rather than a message standard: the payload can be a pain.001 bulk, a camt.053 statement, an MT940 batch, a PDF trade document or a proprietary ERP export.
3. **FileAct inherits the SWIFTNet security and evidence model.** The factsheet states FileAct "uses SWIFTNet Public Key Infrastructure (PKI) that enables the authentication and the integrity control for every file transferred" and that "the PKI signature guarantees the non-repudiation of file emission and file reception," with a centralised log from which SWIFT can provide independent evidence in a dispute ✅ (swift.com, FileAct factsheet).

**The file lane at a glance — the quick facts:**

- **What it is:** SWIFT's secure file-transfer service over SWIFTNet — the FILE lane beside FIN (messages) and InterAct (real-time XML) ✅
- **The payload:** any format — MT/MX batches, domestic formats, proprietary files, images, PDFs ✅
- **The delivery:** store-and-forward (default, resilient) and real-time modes; push (file transfer) and pull (file download) ✅
- **The security:** SWIFTNet PKI — authentication, integrity, confidentiality in transit, non-repudiation of emission and reception ✅
- **The evidence:** centralised log; SWIFT can provide independent evidence of a transfer in a dispute ✅
- **The size limit:** 250 MB per physical file (including the ~30 KB FileAct header) ✅ (official Bundesbank FileAct procedural rules, mirroring the SWIFT requirement — see §3.6)
- **The availability:** the SWIFTNet platform runs at above 99.99% availability ✅ (swift.com, FileAct factsheet)
- **The one-liner:** the reliable file lane — the channel that carries the big, provable, schedule-bound payloads of the SWIFT estate

### 1.2 The Definition — Verified

**SWIFTNet FileAct is SWIFT's secure store-and-forward file-transfer service that carries large files of any format between SWIFT users — financial institutions, market infrastructures and corporates — over the SWIFTNet secure IP network, with SWIFTNet PKI authentication, integrity control, confidentiality and non-repudiation of emission and reception.** Verified against the primary sources this pass:

- **SWIFT's own product page** — "FileAct offers a secure, efficient solution for large data file transfers, connecting you to a global network. Ideal for bulk payments and large-scale reporting, it supports various file formats and sizes, ensuring industry-leading availability and security" ✅ (swift.com/products/fileact). The page's benefits: "Provides levels of security that aren't available with internet-based file transfer solutions"; "Thousands of Swift users worldwide are ready to exchange files over FileAct"; and value-added features "including file compression, enhanced headers to facilitate file handling and routing, and copies of headers to third parties for authorisation or information" ✅.
- **SWIFT's FileAct factsheet** — "a cost-effective way to transfer large volumes of data in different formats to your correspondents... mass payments information, documents, images, or other data... a single channel to your correspondents around the globe"; two mechanisms — "push files to correspondents (file transfer) or get files from correspondent (file download)"; "highly reliable solution to send business-critical files. With an availability ratio of above 99.99%, the SWIFTNet platform enjoys the highest reliability rates in the industry" ✅ (swift.com, FileAct factsheet, SWIFT © 2009).
- **The SWIFTNet service description as mirrored in the integration documentation** — the BizTalk FileAct adapter documentation (Microsoft Learn, SWIFT-document-derived) describes SWIFTNet FileAct as providing "secure and reliable transfer of files, such as batches of structured financial messages or large reports," with typical applications including "repetitive credit transfers such as pension or salary payments, securities value-added information and reporting, and regulatory reporting" ✅.
- **The official central-bank procedural rules** — the Deutsche Bundesbank's official "SWIFTNet FileAct procedural rules" (Version 1.7, 2016, and the 2024-03 update) govern file exchange with the Bundesbank over FileAct and confirm the SWIFT-imposed maximum transfer-file size of 250 MB per physical file ✅ (bundesbank.de official documents).
- **The repo's own cluster** — [payment_rails_guide.md](payment_rails_guide.md) §2.5 places SWIFT as the cross-border rail and gpi as its payments overlay; FileAct is the *file* member of the same family — the lane that carries the batches that FIN's per-message architecture cannot carry economically. [payments_hub_guide.md](payments_hub_guide.md) is where the FileAct lane plugs into the hub (routing, transformation, orchestration of file traffic alongside message traffic).

**The three-lane framing (full comparison in §6):**

1. **FIN** — the core service for exchanging *individual structured MT (and ISO 15022) financial messages*: "FIN enables financial institutions to exchange individual structured (MT and ISO 15022 message formats) financial messages securely and reliably. It is based on a distributed processing architecture with full, built-in redundancy" ✅ (swift.com, FIN product page). More than 11,000 FIN users exchange 31.3+ million messages a day across more than 200 countries ✅.
2. **InterAct** — the service for *XML-based message exchange, including MX and ISO 20022*: "InterAct is a messaging service for exchanging XML-based financial messages and data between users, including Swift MX and ISO 20022-based payments, settlement instructions, FX confirmations, statements and reports. It's also used by key market infrastructures around the world" ✅ (swift.com, InterAct product page).
3. **FileAct** — the service for *files*: the format-agnostic, store-and-forward file lane that this guide covers.

### 1.3 The Overview Table — Aspect / Description / Status

| Aspect | Description | Status |
|---|---|---|
| **Definition** | SWIFT's secure store-and-forward file-transfer service over SWIFTNet — the file lane of the SWIFT estate, carrying large files of any format between SWIFT users | ✅ Verified (swift.com; factsheet) |
| **The lane** | FILE lane beside FIN (individual MT/ISO 15022 messages) and InterAct (XML/MX/ISO 20022 request-reply) | ✅ Verified (swift.com FIN/InterAct/FileAct pages) |
| **The payload** | Any type of data — text, spreadsheets, XML, images, PDFs; MTs or MXs, domestic formats, proprietary formats; any character set | ✅ Verified (FileAct factsheet) |
| **The delivery modes** | Store-and-forward (files safe-stored and delivered when the correspondent connects) and real-time file transfer; push (file transfer) and pull (file download) | ✅ Verified (factsheet; BizTalk docs) |
| **The security** | SWIFTNet PKI — authentication and integrity control for every file; confidentiality in transit; non-repudiation of emission and reception | ✅ Verified (factsheet; BizTalk docs) |
| **The evidence** | Centralised log of file transfers; SWIFT can provide independent evidence in a dispute | ✅ Verified (factsheet) |
| **The size limit** | 250 MB maximum per physical file, including the ~30 KB FileAct header | ✅ Verified (official Bundesbank FileAct procedural rules §3.6) |
| **The availability** | SWIFTNet platform availability above 99.99%; protocol auto-recovers from intermittent failures | ✅ Verified (factsheet) |
| **The value-adds** | File compression, enhanced (business) headers, FileAct Copy (Y-copy for authorisation, T-copy for information), file segregation, pacing, routing parameters | ✅ Verified (factsheet; FileAct Copy factsheet) |
| **The pricing** | Usage fee per transferred file, varying by size in bytes and the institution's global tier | ✅ Verified (factsheet) |
| **The reach** | All SWIFT members are technically enabled to exchange FileAct traffic; generic FileAct service subscription opens access to many FIs and key market infrastructures | ✅ Verified (factsheet) |
| **The position** | The reliable file lane — the channel for bulk, provable, schedule-bound payloads | ✅ Structural (§9) |

### 1.4 Reading the Overview Table

The table is the whole discipline in one glance. **A Solution Architect works at the seam between the business payload and the lane** — the [payments hub](payments_hub_guide.md) exists to manage exactly that seam. Three readings:

1. **FileAct is a lane choice, not a message choice.** The business question is *"what am I moving — a message, a real-time exchange, or a file?"* A 10,000-item salary batch is a *file* problem (FileAct); a single urgent MT103 is a *message* problem (FIN); a real-time account-status query is an *exchange* problem (InterAct). The same ISO 20022 payload can travel as a FINplus message or inside a FileAct batch — the lane is selected on volume, timing and evidence requirements, exactly as the rail-selection discipline in [payment_rails_guide.md](payment_rails_guide.md) §7–§8.
2. **The lane decides the non-functional requirements.** FileAct brings batch cut-offs and schedules (cross-ref [../technology/zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md)), delivery-evidence semantics (cross-ref [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md) — FileAct's delivery notification is the file-lane analogue of the queue ack), and PKI lifecycle management that the ops team must run.
3. **The lane is the moat.** FileAct's value is *network effect plus evidence*: thousands of SWIFT users "ready to exchange files over FileAct" ✅, with a legal-grade audit trail — a combination that internet file transfer (SFTP/FTPS) and bilateral links cannot reproduce without assembling the same PKI and evidence apparatus yourself (§6.2).

### 1.5 The One-Liner — the File Lane

**FileAct is the reliable file lane: the SWIFTNet service that carries large, any-format, evidence-bearing file payloads between SWIFT users with store-and-forward resilience, PKI security and 99.99%+ platform availability.** Every other section of this guide is a deepening of that one sentence: the platform it rides on (§2), the mechanics that make it reliable (§3), the products that deliver it (§4), the business problems it solves (§5), the alternatives it competes with (§6), how a bank actually runs it (§7–§8), and the summary that closes the file (§9).

---

## 2. The SWIFTNet Context

### 2.1 The SWIFTNet Platform — the Single Window

**SWIFTNet is the secure IP network and platform that carries all of SWIFT's modern services — FIN, InterAct and FileAct — and it is the environment FileAct lives in.** The SWIFT-document-derived integration documentation puts it precisely: "as a general purpose, industry-standard solution for the financial industry, SWIFTNet provides an application-independent, single window interface to all the connected applications of all the institutions participating in the global financial community... SWIFTNet provides a basis for assuring business continuity and disaster recovery for the infrastructure of mission-critical financial applications that cross institutional boundaries" ✅ (Microsoft Learn, "What Is SWIFTNet?", mirroring the SWIFTNet service descriptions). The same source lists what SWIFTNet provides to interconnected business applications: infrastructure reliability, availability, role-based and non-role-based access control, correspondent and message authentication, message integrity, confidentiality, non-repudiation support, message validation, and store-and-forward ✅.

Two components carry the platform (both detailed below): **SWIFTNet Link (SNL)** — "the application programming interface to the SWIFTNet services," and the **SWIFTAlliance Gateway (SAG)** — used "for connectivity and usability" ✅ (same source). FileAct traffic crosses the SWIFT Secure IP Network (SIPN), "a secure private network which connects financial institutions, financial industry infrastructures, and customers" ✅ (Microsoft Learn, FileAct adapter documentation). The platform's availability is a headline FileAct property: "with an availability ratio of above 99.99%, the SWIFTNet platform enjoys the highest reliability rates in the industry" ✅ (swift.com, FileAct factsheet).

### 2.2 SWIFTNet Link (SNL) — the Mandatory Network Interface

**SWIFTNet Link is the mandatory client-side network interface to SWIFTNet — the software that connects a SWIFT user's systems to the platform — and every FileAct transfer passes through it.** Verified anchors:

- **The mandatory interface:** "Business software applications use the SWIFTNet Link (SNL) application programming interface (API) to access and use SWIFTNet services. The SNL is the mandatory network interface to SWIFTNet. SWIFTNet requires SNL for all external interfaces... The SNL is incorporated into SWIFTAlliance WebStation and SWIFTAlliance Gateway (SAG)" ✅ (Microsoft Learn, "What Is SWIFTNet?").
- **The transport features:** "SNL provides a set of transport-level features designed for high availability and high throughput environments," including load balancing, location transparency and routing (shielding applications from the underlying transport), transport-level authentication and confidentiality "packaged within SNL and provided transparently to the application," and security functions for end-to-end (application-to-application) security ✅ (same source).
- **The programming model:** at source level, only two functions — **SwCall** (used by client applications to access server applications through SWIFTNet) and **SwCallback** (used by server applications to respond) — passing structured XML messages to and from SWIFTNet ✅ (same source). The FileAct adapter for BizTalk "uses the SWIFTNet Link (SNL) application programming interface (API)s to connect to the SIPN" ✅ (Microsoft Learn, FileAct adapter documentation).
- **SWIFT's own words:** "SwiftNet Link connects Swift institutions with customer on-premise or cloud hosted Swift infrastructure to ensure secure, interoperable communication. It's your single access point to Swift, streamlining your infrastructure with transport, formatting, security, and service management. It's also equipped with XML-based APIs for seamless integration, enabling Alliance Gateway for app-to-app SwiftNet services" ✅ (swift.com, SwiftNet Link product page). The page confirms Windows and UNIX support and that SwiftNet 7.7 is "the latest major and mandatory release" ✅.

For the file lane specifically, SNL is where FileAct's *automatic decompression*, *file segregation* and *load balancing* behaviours run — the receiving SWIFTNet Link "automatically decompresses the file before presenting it to the receiver application," and "SWIFTNet Link automatically segregates file transfers from other message flows" ✅ (swift.com, FileAct factsheet).

### 2.3 SWIFTNet PKI — Certificates and Non-Repudiation

**SWIFTNet PKI is the certificate infrastructure that secures every SWIFTNet exchange — FileAct included — and it is the reason FileAct transfers carry legal-grade evidence.** Verified anchors:

- "FileAct uses SWIFTNet Public Key Infrastructure (PKI) that enables the authentication and the integrity control for every file transferred. It also provides certificate issuance and management with a robust registration mechanism" ✅ (swift.com, FileAct factsheet).
- "The PKI signature guarantees the non-repudiation of file emission and file reception. FileAct keeps a centralised log of the file transferred. In the event of a dispute, SWIFT can provide independent evidence confirming that the transfer took place as claimed" ✅ (same source).
- The adapter documentation confirms the standard feature set: "SWIFTNet FileAct is secured with SWIFTNet PKI and offers message authentication and integrity control," plus confidentiality of file data in transit, non-repudiation of file transfers and time-stamping ✅ (Microsoft Learn, FileAct adapter documentation).

In practice the PKI means every SWIFTNet participant holds certificates (issued under SWIFT's certification hierarchy, with a robust registration mechanism) that sign each transfer; the sender's signature is what the receiver (and SWIFT's centralised log) can verify, and what makes repudiation impossible. The operational consequence for a bank is certificate lifecycle management — ordering, renewal, revocation — as part of the SWIFT estate (see §7.7).

### 2.4 The Relationship Management Application (RMA) — Counterparty Permissions

**The Relationship Management Application (RMA) is SWIFT's service for managing which counterparties may send traffic to you and receive traffic from you — the permission layer that sits on top of the PKI — and it governs FileAct counterparty relationships as well as FIN and InterAct.** Verified anchors:

- **What it is:** "SWIFT's Relationship Management Application (RMA) plays an important part in supporting communication between different financial institutions" ✅ (swift.com, SWIFT compliance infopaper on RMA). The training description adds: RMA helps you "prevent unwanted financial traffic," defines "for which type of messages RMA is mandated," and covers "what you are allowed to do when you create an RMA authorisation for FIN" ✅ (swift.com training catalogue, RMA Service).
- **RMA for FileAct and InterAct:** "Relationship Management Application (RMA) is a service provided by SWIFT to help manage the business relationships between financial institutions. SWIFTNet 7.0 enables RMA capabilities for services based on InterAct and FileAct" ✅ (Axway documentation on RMA — vendor, corroborating the SWIFTNet 7.0 extension of RMA beyond FIN).
- **The bank-side process:** NAB's official FileAct fact sheet instructs corporate clients: "You will need to organise a Relationship Management Application (RMA) request with NAB before commencing implementation... The RMA is subject to NAB's acceptance" ✅ (NAB, SWIFT FileAct Fact Sheet, 2019) — the real-world proof that a FileAct relationship is *authorised*, not just technically possible.

For the file lane the RMA is the *relationship gate*: before a Cymbal Bank ↔ corporate FileAct flow can move a single file, both sides must have exchanged RMA authorisations for the FileAct service (see §7.1 for the worked example). This is the file-lane analogue of the correspondent-relationship governance described in [payment_rails_guide.md](payment_rails_guide.md) §2.5.

### 2.5 The Three SWIFTNet Services — FIN, InterAct, FileAct

**SWIFTNet carries three services, and the architect's first job is to place a payload on the right one.** All three verified this pass against swift.com:

| Service | What it carries | Verified description (swift.com) | Lane |
|---|---|---|---|
| **FIN** | Individual structured MT / ISO 15022 messages | "FIN enables financial institutions to exchange individual structured (MT and ISO 15022 message formats) financial messages securely and reliably. It is based on a distributed processing architecture with full, built-in redundancy to ensure maximum availability" ✅ | The message lane |
| **InterAct** | XML-based messages — MX / ISO 20022, request/reply | "InterAct is a messaging service for exchanging XML-based financial messages and data between users, including Swift MX and ISO 20022-based payments, settlement instructions, FX confirmations, statements and reports. It's also used by key market infrastructures around the world" ✅ | The real-time exchange lane |
| **FileAct** | Files of any format — bulk batches, reports, documents | "FileAct offers a secure, efficient solution for large data file transfers, connecting you to a global network. Ideal for bulk payments and large-scale reporting" ✅; "a single secure channel for large files of structured messages, operational data or reports" ✅ (FIN page cross-ref) | **The file lane** |

The services share the platform (§2.1), the SNL interface (§2.2) and the PKI (§2.3), and all traffic undergoes "a common set of checks to ensure that no user can bypass the security, validation and routing rules of the platform... performed by the SWIFTAlliance Gateway (SAG) application" ✅ (Microsoft Learn, FileAct adapter documentation). The *delivery paradigm* is what differs: FIN is store-and-forward per message with central validation; InterAct is request/reply (real-time); FileAct is file transfer with store-and-forward and real-time modes (full comparison in §6.1).

### 2.6 The Platform Evolution — Browse to WebAccess, SWIFTNet 7.7

**The SWIFTNet estate evolves in releases, and the file lane rides on that evolution.** Verified anchors:

- **Browse → WebAccess:** SWIFT's interactive workstation service, Browse, was retired — "SWIFT announced to end the support for the Browse service and applet technology for all SWIFT releases 7.0.x and 7.1.x at the end of 2019... all SWIFT users will need to migrate from Browse to WebAccess by end of 2019" ✅ (European Central Bank, official communication on the migration, 2019). WebAccess "builds upon our previous interactive service, Browse," adding a SWIFT-managed centralised authentication service with single sign-on ✅ (swift.com, SWIFT WebAccess resource).
- **SwiftNet 7.7:** "The latest major and mandatory release, 7.7 provides a comprehensive set of features and tools... to reduce cost and risk, improve efficiency and increase flexibility" ✅ (swift.com, SwiftNet Link product page).
- **The cloud era:** the connectivity options now include Alliance Cloud ("Swift's preferred financial messaging interface launched in 2020") and Alliance Lite2, which "enables direct access to financial institutions across the globe... via the Swift Community Cloud" ✅ (swift.com, Alliance Lite2 pages) — delivery options covered in §4.

The takeaway for this guide: FileAct is not a static product — it is a *service on an evolving platform*, and any architecture note (like §7) must be written against a release line (SNL 7.x, SAG 7.x) rather than a fixed snapshot. ⚠-knowledge: the exact retirement roadmap for the legacy SWIFTNet services after the ISO 20022 FINplus migration is evolving and was not re-verified in this pass.

---

## 3. The FileAct Mechanics

### 3.1 The File Transfer Envelope

**A FileAct transfer is a file wrapped in an envelope: the FileAct header (technical + optional business header) plus the payload, addressed by SWIFT BIC and carried over SWIFTNet.** Verified anchors:

- **The addressing:** FileAct "lets an institution (client/TxB) drop their files (almost any format) in an envelope and post it over the SWIFT network. The addresses on the post are the sending and receiving institutions' BICs" ✅ (Goldman Sachs developer documentation, FileAct service description).
- **The header:** "A dedicated optional header block is available to specify key business summary information, for example the number of payments, or the total amount of your file transfer. This header block allows easy file handling and processing without the need to open the file" ✅ (swift.com, FileAct factsheet). The FileAct Copy factsheet distinguishes the *technical header* ("for example sender or receiver") from the *business header* ("typically file summary data such as number of items or total amount") ✅ (swift.com, FileAct Copy factsheet).
- **The header size:** the FileAct header occupies roughly 30 KB of the file's size budget — "the maximum size of files sent over SWIFTNet FileAct is 250 MB (this includes the FileAct header of 30Kb)" ✅-knowledge (Rabobank SWIFT FileAct manual; consistent with the 250 MB limit in the official Bundesbank procedural rules — §3.6).

The envelope is the unit of everything else in this section: the delivery modes (§3.2), the acknowledgements (§3.3), the size limit (§3.6) and the copy services (§3.9) all operate on the *transfer* — the enveloped file — rather than on individual records inside the payload.

### 3.2 The Delivery Modes — Store-and-Forward vs Real-Time

**FileAct offers two delivery paradigms — store-and-forward (the resilient default for correspondents that may be offline) and real-time (for online counterparties and market infrastructures) — plus two directions: push (file transfer) and pull (file download).** Verified anchors:

- **The two mechanisms:** "Two mechanisms are available to FileAct users: push files to correspondents (file transfer) or get files from correspondent (file download)" ✅ (swift.com, FileAct factsheet).
- **Store-and-forward:** "When your correspondent is not online at the time of the file transmission, you can opt to use FileAct in store-and-forward mode. FileAct will then safe store the file and handle the delivery upon connection of your correspondent" ✅ (same source). The adapter documentation adds: "SWIFTNet FileAct's store-and-forward capability removes the uncertainty and inconvenience of worrying about whether or not your correspondents are online at the time you transmit the file. The file is delivered as soon as the recipient is ready to receive it... an ideal way to send files to large numbers of correspondents, some of which may be in different time zones" ✅ (Microsoft Learn, FileAct adapter documentation).
- **Real-time modes:** "Real-time file transfer. Real-time messaging offers a lower-cost alternative to store and forward for files that are destined for correspondents that are online at the time of transmission. As a result it is ideal for sending files to a few large correspondents or market infrastructures" and "Real-time file retrieval. This is an interactive service typically used to retrieve files in the context of workstation-based systems and often in conjunction with SWIFTNet Browse" ✅ (same source).

The architect's reading: **store-and-forward is the default for the file lane** — it is what makes FileAct reliable across time zones and cut-offs — while real-time mode is the low-latency, lower-cost option for the handful of counterparties (large correspondents, market infrastructures) that are always online.

### 3.3 The End-to-End Acknowledgements

**FileAct's evidence model is a chain of acknowledgements: the sender can request delivery notification, the receiver can monitor queue status, and both sides have non-repudiation of emission and reception.** Verified anchors:

- **Delivery notification and non-delivery warning:** "The delivery notification and non-delivery warning options enable the sender to track the delivery status of the file transfers. Similarly, receiving customers can monitor their queues status" ✅ (swift.com, FileAct factsheet).
- **The adapter feature list:** "Delivery notification confirming file receipt by a receiver... File transfer state monitoring... Non-repudiation of file transfers... Time-stamping" ✅ (Microsoft Learn, FileAct adapter documentation).
- **The optional priority flag:** "File transfers can be flagged as 'Urgent' in the message, to allow for appropriate processing by your correspondents" ✅ (same source).
- **The non-repudiation pair:** "non-repudiation of emission and reception. In case of dispute, allows SWIFT to confirm that the file transfer did take place as claimed" ✅ (same source).

The ack chain maps cleanly onto the at-least-once delivery semantics discussed in [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md): the *delivery notification* is the consumer ack at the FileAct layer, and the *business-level ack* (e.g., a pain.002 or a statement-confirmation file) is the application-layer ack the counterparty sends inside the business payload — a distinction the worked example in §7 uses explicitly.

### 3.4 The File Naming Conventions

**There is no universal SWIFT-mandated business file name for FileAct payloads — the file's identity travels in the FileAct header, and *file naming* is a bilateral agreement between the counterparties.** This is one of the most common integration misunderstandings, so it deserves an explicit treatment:

- **What SWIFT defines:** the FileAct *transfer* carries a header (technical + optional business header — §3.1) that identifies the file, its sender, receiver and summary data. The FileAct service description defines the envelope and the protocol, not a business naming scheme for the payload ✅-knowledge (consistent with the factsheet's "any content structure: you can use SWIFT message formats MTs or MXs, domestic formats, or your own proprietary ones" ✅).
- **What banks do:** in practice each bank publishes a naming convention in its FileAct implementation guide — e.g., Rabobank's SWIFT FileAct manual and Nordea's "Security and Communication description for SWIFTNet FileAct" document both define the file naming, compression and signing rules corporates must follow when sending files to the bank ✅ (Rabobank and Nordea official documents — bank-defined, bilateral conventions).
- **The architect's rule:** the file *name* is a bilateral contract (prefixes for direction, date-time stamps, sequence numbers for reconciliation), while the *identity and evidence* of the transfer come from the FileAct header and the delivery notification. ⚠ the exact naming rules are per-bank and were not re-verified beyond the two bank documents above.

### 3.5 The Retry and Recovery Behaviour

**FileAct is engineered to survive intermittent failures: the protocol auto-recovers, the platform load-balances across links, and store-and-forward holds files until the counterparty can take them.** Verified anchors:

- **Automatic recovery:** "the sophisticated FileAct protocol automatically recovers from intermittent failures, performs checks on the authenticity of the sender and transfers the files providing evidences that your file has been delivered" ✅ (swift.com, FileAct factsheet).
- **Load balancing:** "FileAct can also spread incoming traffic over multiple SWIFTNet Links to offer optimal load balancing and higher resiliency" ✅ (same source).
- **Queue monitoring:** "receiving customers can monitor their queues status" ✅ (same source); the adapter adds "abort of transfers in progress" and "file transfer state monitoring" as supported functions ✅ (Microsoft Learn, FileAct adapter documentation).
- **No-lost-messages recovery at the platform level:** the MI Channel (the market-infrastructure variant of the same platform) advertises "recovery features ensure no lost messages, with data replay for recovery in the event of a failure" ✅ (swift.com, MI Channel product description) — the same platform-level recovery philosophy applies to FileAct flows.

The failure-and-retry scenario in §7.5 shows what this means operationally: an intermittent network failure does not lose the file — the sender sees a non-delivery warning, the store-and-forward queue holds the file, and delivery completes when the counterparty reconnects; a *business-level* failure (payload rejected by the receiver's application) is a different animal, handled by bilateral re-submission.

### 3.6 The File Size Limits — 250 MB per Physical File

**The maximum FileAct transfer size is 250 MB per physical file (including the FileAct header of roughly 30 KB).** This is the number the task asked to verify from a primary source — here is exactly how it was verified:

- **Official central-bank procedural rules:** the Deutsche Bundesbank's official "SWIFTNet FileAct procedural rules" (Version 1.7 of November 2016, and the 2024-03 update) state: "The maximum size of a transfer file is restricted to 250 MB per physical file. The communication partner must ensure that the relevant file sizes can be received" ✅ (bundesbank.de official documents — the Bundesbank's FileAct channel, which mirrors the SWIFT requirement in its procedural rules).
- **Bank implementation manuals:** the Rabobank SWIFT FileAct manual states "SWIFT sets the following requirements on delivery: the maximum size of files send over SWIFTNet FileAct is 250 MB (this includes the FileAct header of 30Kb)" ✅-knowledge (Rabobank manual — consistent with the Bundesbank figure).
- **Vendor corroboration:** Axway's Secure Financial Client documentation confirms FileAct support "within the size limits established by Swift," pointing to the official SwiftNet documentation for current limits ✅ (Axway docs — corroborating that a limit exists, without restating the number).
- **Honest caveat:** SWIFT's own Knowledge Centre service description (the ultimate primary source) is authentication-gated and could not be read in this pass; the 250 MB figure is therefore verified against the official Bundesbank procedural rules (a primary document that governs real FileAct traffic) and multiple independent bank manuals, and is flagged ✅ with that provenance noted. ⚠-knowledge: SWIFT periodically adjusts service parameters through releases, so the *current* number should be re-checked against the SWIFTNet Service Description before a design is frozen.

Pricing interacts with size: "SWIFT charges a FileAct usage fee per transferred file. The transfer fees vary according to the size (in bytes) of the file and the institution's global tier" ✅ (swift.com, FileAct factsheet) — so the size limit is not only a technical constraint but a cost driver (§8.3).

### 3.7 The Scheduled vs On-Demand Transfers

**FileAct supports both scheduled batch transfers (the file lane's natural rhythm — daily, cut-off-driven) and on-demand transfers (real-time mode or workstation-initiated).** Verified and structural anchors:

- **Scheduled:** the store-and-forward mode *is* the scheduling mechanism — a file sent at 02:00 is delivered when the counterparty connects, which is how banks run overnight statement and bulk-payment batches across time zones ✅ (factsheet §3.2; structurally confirmed by the batch use cases in §5).
- **On-demand:** the real-time transfer and real-time retrieval modes serve the on-demand case — "ideal for sending files to a few large correspondents or market infrastructures," with retrieval "often in conjunction with SWIFTNet Browse" (now WebAccess — §2.6) ✅ (Microsoft Learn, FileAct adapter documentation).
- **Pacing:** "A configurable pacing factor will help you to further reduce the throughput capacity used by FileAct transfers" ✅ (swift.com, FileAct factsheet) — the throttle that keeps scheduled bulk windows from starving other SWIFTNet traffic.

The cut-off calendar discipline of the file lane (value-date cut-offs, statement-run times, regulatory deadlines) is the same discipline covered for rails in [../technology/zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md) and for the SG banking day in [banks_in_singapore_guide.md](banks_in_singapore_guide.md).

### 3.8 The Compression and Encryption

**FileAct provides optional compression at the sender with automatic decompression at the receiving SWIFTNet Link, and confidentiality in transit via SWIFTNet PKI — while *application-level* encryption of the payload remains a bilateral option.** Verified anchors:

- **Compression:** "FileAct provides an optional compression mechanism that you can invoke when sending a file. The receiving SWIFTNet Link automatically decompresses the file before presenting it to the receiver application" ✅ (swift.com, FileAct factsheet). The FileAct product page lists "file compression" among the value-added features ✅.
- **Confidentiality in transit:** the adapter documentation lists "Confidentiality of file data in transit" among the supported functionality ✅ (Microsoft Learn, FileAct adapter documentation); SNL provides "transport-level authentication and confidentiality, packaged within SNL and provided transparently to the application" ✅ ("What Is SWIFTNet?").
- **End-to-end security option:** SNL also offers "security functions by which business application software may establish end-to-end security (user application to user application), when required" ✅ (same source).
- **Bank practice:** Nordea's official "Security and Communication description for SWIFTNet FileAct" documents the compression and signing rules corporate clients must follow ("The security solution for SWIFTNet FileAct is provided by SWIFT") ✅ (Nordea official document) — i.e., banks commonly *mandate* compression and PKI signing as part of the bilateral convention, and may layer payload encryption (e.g., PGP) for sensitive content, which is a bilateral choice ⚠-knowledge (payload-level encryption beyond the transport is per-bank policy, not a SWIFT-mandated FileAct feature).

### 3.9 The Value-Added Features — FileAct Copy, Segregation, Routing

**Beyond the core transfer, FileAct ships value-added features that make the file lane operationally powerful — the copy services, traffic segregation and routing controls.** Verified anchors:

- **FileAct Copy (Y-copy / T-copy):** "FileAct Copy is a value-added service of FileAct enabling the sender of a file to trigger a copy of the file header to a third party for authorisation (Y-copy) or for information (T-copy)" ✅ (swift.com, FileAct Copy factsheet). In Y-copy mode a central institution (e.g., a clearing house or a head-office risk function) receives the header and "authorise[s] or refuse[s] its delivery to the addressee," with decisions based on header content, payer's account balance, collateral or credit line, or multilateral agreements ✅ (same source). T-copy mode serves monitoring, insourcing/outsourcing and third-party service scenarios ✅ (same source).
- **File segregation:** "To prevent message delays, SWIFTNet Link automatically segregates file transfers from other message flows" ✅ (swift.com, FileAct factsheet).
- **Routing:** "receivers with multiple SWIFTNet connections may use a variety of parameters to define where each file is routed to" ✅ (same source).
- **Enhanced headers:** "enhanced headers to facilitate file handling and routing" ✅ (swift.com, FileAct product page) — the business-header mechanism of §3.1.

---

## 4. The Delivery Products — How FileAct Reaches the Bank

**FileAct is a *service*, not an application — the bank reaches it through an interface, and the choice of interface is a delivery decision, not a FileAct decision.** The official factsheet says implementation is easy precisely because of product choice: "You have the choice between different products from SWIFT or third-party vendors that will facilitate the implementation and reduce the time and effort required" ✅ (swift.com, FileAct factsheet). This section positions the main delivery vehicles for the file lane; the Alliance-family products themselves are covered by the repo's payments cluster ([payment_rails_guide.md](payment_rails_guide.md), [payments_hub_guide.md](payments_hub_guide.md)) and are NOT re-derived here (§4.5).

### 4.1 SWIFT Alliance Gateway (SAG) — the Connectivity Hub

**SWIFT Alliance Gateway (SAG) is the on-premise connectivity hub that incorporates SWIFTNet Link and adds the integration muscle — message concentration, host adapters and the common security/validation checks that every SWIFTNet transfer, FileAct included, must pass.** Verified anchors:

- **What it is:** "The SWIFTAlliance Gateway (SAG) is an interface product for SWIFTNet. It incorporates all the functionality of the SWIFTNet Link. Additionally, it provides several different connectivity and usability features for SWIFTNet users" ✅ (Microsoft Learn, "What Is SWIFTNet?").
- **The concentrator role:** "The SAG serves as a message concentrator. It receives messages from various other applications and passes them through SWIFTNet. It receives these messages through host adapters, including a WebSphere MQ host adapter, which enables business applications running on a variety of different types of computing platforms to pass messages through SWIFTNet" ✅ (same source).
- **The common checks:** "All messages and files exchanged on SWIFTNet undergo a common set of checks to ensure that no user can bypass the security, validation and routing rules of the platform. These checks are performed by the SWIFTAlliance Gateway (SAG) application" ✅ (Microsoft Learn, FileAct adapter documentation).
- **The strict SNL mode:** "In strict SWIFTNet Link Mode, the SAG presents a messaging interface that is functionally equivalent to the SWIFTNet Link interface" ✅ (same source) — the mode the FileAct/InterAct adapters use.
- **FileAct Copy support:** "The SWIFT Alliance portfolio – Alliance Gateway and Alliance Access 6.3 – fully support this feature" ✅ (swift.com, FileAct Copy factsheet).

For the file lane, SAG is where the bank's FileAct traffic enters SWIFTNet: it holds the SWIFTNet PKI signing keys, applies the platform checks, concentrates file traffic from internal systems (via MQ or other host adapters), and can spread FileAct traffic over multiple SNL connections for load balancing ✅ (factsheet §3.5).

### 4.2 Alliance Access — the Messaging Backbone

**Alliance Access (AA) is SWIFT's messaging backbone application — the system of record for SWIFT traffic inside the bank — and it manages FileAct files alongside FIN and InterAct traffic.** Verified anchors:

- **FileAct support in AA:** the FileAct Copy factsheet confirms "The SWIFT Alliance portfolio – Alliance Gateway and Alliance Access 6.3 – fully support this feature" ✅ (swift.com, FileAct Copy factsheet) — i.e., the Alliance Access product line handles FileAct transfers as first-class traffic.
- **Positioning:** Alliance Access sits above the gateway as the message/file management layer — routing, queueing, release, archiving and reporting for all SWIFTNet services including the file lane. ⚠-knowledge: the detailed AA feature set (FileAct release workflows, file-based routing rules, dual-site operation) is a product-line topic covered in the repo's payments cluster rather than re-derived here — see [payments_hub_guide.md](payments_hub_guide.md) for the hub architecture it belongs to.

### 4.3 Alliance Lite2 and Alliance Cloud — the Managed/Cloud Lane

**Alliance Lite2 is SWIFT's managed, internet-based connectivity interface — the entry point that lets smaller institutions and corporates use FIN, InterAct and FileAct without running a full SAG estate — and Alliance Cloud is its 2020 evolution.** Verified anchors:

- **Lite2's scope:** "Alliance Lite2 is an Internet-based solution that allows SWIFT customers to connect to SWIFT easily and securely. Users can send and receive message transactions using the FIN, InterAct, FileAct, and Browse services over SWIFTNet" ✅ (SWIFT Alliance Lite2 Administration Guide, official SWIFT document mirrored publicly). The swift.com ordering page confirms Lite2 "enables direct access to financial institutions across the globe, providing the essential services to communicate with them via the Swift Community Cloud" ✅.
- **The cloud evolution:** "Alliance Cloud, Swift's preferred financial messaging interface launched in 2020, offers agility, scalability, and future-proof connectivity. It reduces costs by minimising infrastructure and maintenance while ensuring top-tier security, availability, and user friendliness" ✅ (swift.com, Alliance Lite2 product page).
- **The hyperscaler angle:** Microsoft publishes deployment guidance for "SWIFT Alliance Lite2 on Azure" and "SWIFT Alliance Cloud on Azure" (the SWIFT Integration Layer / Alliance Connect Virtual pattern), and AWS publishes "Guidance for SWIFT Alliance Lite2 on AWS" using cloud-native VPN instead of traditional hardware ✅ (Microsoft Learn and AWS documentation — the file lane now runs on cloud connectivity).

For the file lane, Lite2/Alliance Cloud matter because they make FileAct *reachable without a datacentre footprint* — the classic on-ramp for corporates and mid-size banks that want the file lane's evidence and reach without operating SAG.

### 4.4 The Third-Party and SWIFTReady Interfaces

**Beyond SWIFT's own products, the file lane is served by a certified ecosystem of third-party interfaces.** Verified anchors:

- **The choice:** "You have the choice between different products from SWIFT or third-party vendors that will facilitate the implementation... Additional information on these products is available on swift.com > Solutions > Partner programme > Qualified interfaces > Qualified SWIFTNet Messaging Interfaces" ✅ (swift.com, FileAct factsheet).
- **The certification:** "The SWIFTReady labels indicate that the applications, services, and connectivity you select are certified by SWIFT" ✅ (same source).
- **Examples seen this pass:** the Microsoft BizTalk FileAct/InterAct adapters (which connect BizTalk to SIPN via the SNL APIs ✅), IBM's messaging-services components for SWIFTNet InterAct and FileAct ("Applications that need to transfer files rather than exchange messages use SWIFTNet FileAct... sending a file directly... [or] downloading a file" ✅ — IBM documentation), and Axway's Secure Financial Client ("fully supports the SwiftNet InterAct and FileAct protocols within the size limits established by Swift" ✅ — Axway documentation).

### 4.5 Positioning — the File Lane, Not the Alliance Deep-Dive

**This guide deliberately does not re-derive the Alliance-family product details.** The repository's payments cluster already covers the hub architecture ([payments_hub_guide.md](payments_hub_guide.md)) and the rails map ([payment_rails_guide.md](payment_rails_guide.md)); the Alliance family is the *delivery plumbing* for the three lanes, and this guide treats SAG, Alliance Access, Alliance Lite2 and Alliance Cloud strictly as *vehicles that carry FileAct* — each with the verified anchors above. A reader who needs the product-by-product comparison should go to the payments cluster; a reader who needs to know *what the file lane is, how it behaves and when to use it* is in the right place.

---

## 5. The Use Cases

**FileAct's use cases are the business problems of the file lane: anything too big, too batch-shaped or too document-like for FIN or InterAct, that must still travel with SWIFT-grade security and evidence.** SWIFT's own positioning: FileAct is "ideal for bulk payments and large-scale reporting" ✅ (product page) and "particularly suitable for bulk payments, securities value-added information and reporting," supporting "tailored solutions for market infrastructure communities, closed user groups and financial institutions" ✅ (swift.com FileAct ordering page).

### 5.1 The Bulk Payments — Mass Payments and Payroll Files

**The flagship FileAct use case: batches of payment instructions — salary runs, pensions, mass credit transfers, direct debits — where the item count makes per-message FIN uneconomic.** Verified anchors:

- "a cost-effective way to transfer large volumes of data in different formats to your correspondents. Whether you need to transfer mass payments information, documents, images, or other data, FileAct provides you with a single channel" ✅ (swift.com, FileAct factsheet).
- Typical applications include "repetitive credit transfers such as pension or salary payments" ✅ (Microsoft Learn, FileAct adapter documentation).
- The bank practice: NAB's FileAct fact sheet offers corporate clients bulk *transaction initiation* over FileAct for international MT103-class payments, domestic RTGS, Direct Entry (ACH) via direct debit or credit, and business payables ✅ (NAB fact sheet, 2019) — plus the acknowledgements: disbursement report, payment acknowledgement, BAI2 end-of-day statement ✅ (same source). SAP Multi-Bank Connectivity supports "both FIN and FileAct-based communication" for exactly this corporate bulk-payments market ✅-knowledge (SAP Community documentation).

### 5.2 The Account Statements — MT940/MT950 and camt.053/camt.054

**The second flagship use case: statement distribution — MT940/MT950 (and MT942 intraday) in the MT world, camt.052/camt.053/camt.054 in the ISO 20022 world, and BAI2 in the US-style world — delivered as files.** Verified anchors:

- **The corporate statement formats:** SWIFT's ISO 20022 corporates FAQ confirms "the usage guidelines support both cross border and domestic unitary cash management & reporting messages (camt.052/053/054) that corporates can receive from their financial institutions" ✅ (swift.com, ISO 20022 corporates page).
- **The lane split in practice:** the SWIFT for Corporates implementation manual maps the channels: report information via "FIN: MT940, MT942" and via "FileAct: MT940S, CAMT.053, MT942S, CAMT.052" ✅ (SWIFT for Corporates manual — the "S" variants and camt files travel as FileAct payloads); transaction initiation "FIN: MT101, MT103" vs "FileAct: PAIN.001, PAIN.008" ✅ (same source).
- **The bank practice:** NAB delivers "BAI2 – End of day statement" as a FileAct acknowledgement file ✅ (NAB fact sheet); the AFP practitioner's guide lists "Prior-day and current-day bank reporting" among FileAct corporate-use cases ✅ (AFP, SWIFT Corporate Access Implementation — A Practitioners Guide).
- **The architect's reading:** statements are the canonical *many-to-one file* flow — one bank, many corporate clients, fixed daily schedules — which is exactly the store-and-forward, scheduled, evidence-bearing profile FileAct was built for (§3.2, §3.7). The statement-to-posting mechanics downstream are covered in [posting_rules_mechanics_guide.md](posting_rules_mechanics_guide.md).

### 5.3 The Trade Finance Documents

**Trade documents — invoices, bills of lading, certificates — are files by nature, and the file lane is the natural carrier when the counterparty is SWIFTNet-connected.** Verified and flagged anchors:

- **The digitisation push:** SWIFT has been demonstrating electronic trade-document exchange over FileAct — a proof-of-value project with the trade platform WaveBL "to demonstrate how Swift can support the end-to-end electronic exchange of eBLs and other related shipping documents in the near term, using the digital file transfer service FileAct" ⚠-knowledge (industry press, December — secondary source; the project's existence is reported but was not re-verified on swift.com this pass).
- **The format story:** FileAct's any-format property (§1.2) is what makes document exchange possible — "documents, images, or other data" ✅ (factsheet) — while the trade *messages* (MT700 series, and the ISO 20022 trade messages) ride FIN/InterAct. ⚠-knowledge: bank-specific trade-document FileAct services (e.g., document image exchange hubs) exist but were not individually verified this pass.

### 5.4 The Treasury Confirmations

**Treasury confirmations split across the lanes: real-time confirmations ride InterAct (FX confirmations are explicitly named in SWIFT's InterAct description), while *bulk* confirmation and reporting files ride FileAct.** Verified anchors:

- **InterAct's domain:** "Swift MX and ISO 20022-based payments, settlement instructions, FX confirmations, statements and reports" ✅ (swift.com, InterAct product page) — the real-time confirmation lane.
- **FileAct's domain:** the AFP practitioner's guide lists "Wire transfer and real-time confirmation... Prior-day and current-day bank reporting. Secure file transmission via FileAct: electronic payments/receipts (NACHA), ACH confirmations, check files, lockbox files, and bank reporting" ✅ (AFP, SWIFT Corporate Access Implementation — A Practitioners Guide) — the *batch* confirmation and reporting lane. NAB's FileAct acknowledgements (disbursement report, payment acknowledgement) are the same pattern from the bank side ✅ (NAB fact sheet).
- **The architect's reading:** a single treasury confirmation is an InterAct problem; 5,000 end-of-day confirmations are a FileAct problem. The lane split is volume- and timing-driven (§6.1, §6.6).

### 5.5 The Regulatory Reporting Bulk Files — FATCA/CRS and MAS

**Bulk regulatory reporting is a genuine FileAct class — wherever the receiving authority or infrastructure is FileAct-enabled — with an honest caveat about the FATCA/CRS and MAS specifics.** Verified and flagged anchors:

- **The class:** regulatory reporting is named in SWIFT's own adapter documentation as a typical FileAct application ✅ (Microsoft Learn, FileAct adapter documentation) — and the central-bank example is concrete: the **Deutsche Bundesbank** operates a SWIFTNet FileAct channel governed by its official procedural rules for institutions reporting to it ✅ (bundesbank.de official procedural rules, 2016 and 2024 versions — the same documents that confirm the 250 MB limit, §3.6).
- **FATCA/CRS — the honest caveat:** FATCA and CRS reporting flows to *tax authorities* through their own channels, not through SWIFTNet: the OECD CRS XML schema is the exchange format ✅-knowledge, and in Singapore the IRAS confirms CRS annual returns are filed "via myTax Portal" ✅ (iras.gov.sg, CRS filing page) — i.e., CRS/FATCA *submission* is a tax-portal/SFTP problem, ⚠ while FileAct's role is limited to jurisdictions or arrangements where the authority accepts SWIFTNet file delivery (not confirmed for SG in this pass).
- **MAS reporting — flagged:** MAS regulatory returns (e.g., the MAS notices and statistical reporting regime) are submitted through MAS-designated channels; ⚠ this pass found no primary evidence of MAS accepting MAS returns over FileAct, so the MAS angle is carried here as *context* (the SG regulator's reporting regime is covered in [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)) rather than as a FileAct fact.
- **The architect's reading:** treat "regulatory reporting over FileAct" as *counterparty-dependent*: verify the receiving authority's channel (portal, SFTP, or SWIFTNet) before design — the file lane is for FileAct-enabled counterparties only.

### 5.6 The CLS and Market-Infrastructure File Flows

**Market infrastructures are first-class FileAct counterparties — the file lane carries bulk settlement, netting and reporting files to and from the infrastructures — and CLS is the marquee example.** Verified anchors:

- **The CLS facts:** "CLS is the world's leading provider of FX settlement services. Established in 2002 as a payment-versus-payment (PvP) settlement system to reduce the principal risk arising from settling FX transactions... CLSSettlement settles on average over $6 trillion per day in 18 currencies for 70 settlement members and over 30,000 third-party participants" ✅ (swift.com, Swift for Treasury Market Infrastructures). SWIFT "provides the secure resilient network connectivity channel to reach CLS's services" ✅ (same source), including a CLS-TPS copy service for MT300 copies ✅.
- **The CLS channels:** settlement members may connect via the "MI Channel that utilizes ISO 20022-based XML messages; or a FIN MT Channel with FIN messages. There is also a browser-based Graphical User Interface over SwiftNet Browse" ✅ (same source) — note the *message* channels are FIN/InterAct-style; ⚠ the specific use of *FileAct* for CLS settlement files was not confirmed on the swift.com CLS page this pass, so the CLS-specific FileAct claim is flagged rather than asserted.
- **The MI Channel:** the market-infrastructure channel of the same platform offers "high throughput, intelligent batching, compression and resilient store-and-forward delivery. Recovery features ensure no lost messages, with data replay for recovery in the event of a failure" ✅ (swift.com, MI Channel product description) — the platform's batch/file capabilities for infrastructure communities, which is where FileAct-class flows (bulk netting files, settlement reports) operate.
- **The general market-infrastructure case:** "FileAct supports tailored solutions for market infrastructure communities, closed user groups and financial institutions" ✅ (swift.com FileAct ordering page); "By subscribing to the generic FileAct service, you gain access to many financial institutions and key market infrastructures" ✅ (factsheet) — the reach argument for the file lane.

---

## 6. The Comparisons

### 6.1 FileAct vs InterAct vs FIN — the Three Lanes

**The three SWIFTNet services are three lanes for three payload shapes — and the choice is driven by volume, timing and evidence requirements, not by message standard.** All descriptions verified in §2.5; the decision logic here is structural:

| Dimension | FIN | InterAct | FileAct |
|---|---|---|---|
| **Payload** | Individual structured messages (MT, ISO 15022) | XML messages — MX/ISO 20022, request/reply | Files of any format — batches, reports, documents |
| **Paradigm** | Store-and-forward, per message, centrally validated | Real-time request/reply (and store-and-forward options) | Store-and-forward and real-time file transfer; push and pull |
| **Typical size** | KB-scale per message; per-message cost | KB-scale per message; interactive | Up to 250 MB per file (incl. ~30 KB header) |
| **Evidence** | FIN-level acknowledgements; FIN Copy | Delivery options; copy to third parties | Delivery notification, non-delivery warning, non-repudiation of emission and reception, centralised log |
| **Best for** | Individual payments, trade messages, statements per message | Real-time queries, FX confirmations, ISO 20022 exchanges | Bulk payments, statement batches, regulatory bulk files, documents |

The rule of thumb: **if it is one message, use FIN or InterAct; if it is a thousand messages or one big document, use FileAct.** The same ISO 20022 payment instruction can travel as an InterAct message (real-time) or inside a FileAct pain.001 batch (bulk) — the lane is a *packaging* decision on top of the *standard* decision (see [payment_rails_guide.md](payment_rails_guide.md) §4 for the standard layer).

### 6.2 FileAct vs SFTP/FTPS — the Private Network vs the Internet

**The classic competitor to FileAct is the internet file-transfer stack — SFTP/FTPS over VPN — and SWIFT's own pitch is explicit: FileAct "provides levels of security that aren't available with internet-based file transfer solutions."** Verified and structural anchors:

- **SWIFT's positioning:** "Protects against internet vulnerability. Provides levels of security that aren't available with internet-based file transfer solutions" ✅ (swift.com, FileAct product page); "FileAct is integrated in SWIFTNet, it eliminates the architectural complexity associated with the operation and maintenance of multiple proprietary bilateral links" ✅ (factsheet).
- **What SFTP/FTPS gives you:** ubiquitous, cheap, well-understood, easy to automate — and *bilateral*: every counterparty relationship needs its own keys, accounts, firewall rules and monitoring; there is no shared community directory, no central evidence service, and no network-level non-repudiation service (you build the audit trail yourself) ⚠-knowledge (structural industry knowledge).
- **What FileAct adds:** the SWIFTNet community (thousands of users, "all SWIFT members are technically enabled to exchange FileAct traffic" ✅ factsheet), PKI-based non-repudiation with SWIFT's centralised log as independent evidence ✅ (factsheet), 99.99%+ platform availability ✅, and store-and-forward resilience ✅.
- **The honest trade-off:** SFTP/FTPS is usually cheaper per byte and simpler for *bilateral* corporate integrations; FileAct wins on reach, evidence and resilience — and loses on cost and simplicity for the single-counterparty case. The worked example in §7 is a *FileAct* design precisely because Cymbal Bank's corporate client is SWIFTNet-connected and needs the evidence trail.

### 6.3 FileAct vs IBM MQ — the File Lane vs the Message Queue

**IBM MQ is not a FileAct competitor at the same layer — it is the middleware FileAct *plumbing* is often built on — but the comparison is worth making because architects conflate the two.** Verified and structural anchors:

- **The complementarity:** "The SAG... receives these messages through host adapters, including a WebSphere MQ host adapter, which enables business applications running on a variety of different types of computing platforms to pass messages through SWIFTNet" ✅ (Microsoft Learn, "What Is SWIFTNet?") — i.e., MQ moves files/messages *inside the bank* to the SAG; FileAct moves them *between banks* over SWIFTNet.
- **The differences:** MQ is a general-purpose queuing product (your own infrastructure, your own availability, no community, no PKI-based interbank evidence); FileAct is a *community service* with network-level security, non-repudiation and a central log ✅ (factsheet). MQ gives you at-least-once queue semantics (cross-ref [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md)); FileAct gives you delivery notification and non-delivery warnings on top of its store-and-forward ✅.
- **The architect's reading:** the typical design is *MQ inside, FileAct across*: internal systems publish files to MQ, the gateway adapter (SAG or a SWIFTReady product like BizTalk/IBM) picks them up and posts them as FileAct transfers. MQ is not a FileAct replacement for interbank traffic — but it IS a common alternative for *internal* file movement where no interbank evidence is needed.

### 6.4 FileAct vs Cloud File Exchange — the Modern Alternatives

**The newest competitor set is cloud file exchange — managed SFTP-in-the-cloud, object-store drop zones (S3-style), and bank portals — and SWIFT's own answer is Alliance Cloud.** Verified and flagged anchors:

- **SWIFT's cloud answer:** Alliance Cloud (launched 2020, "Swift's preferred financial messaging interface") and Alliance Lite2 carry FileAct to the Swift Community Cloud ✅ (swift.com); Microsoft and AWS publish deployment guidance for running the Lite2/Alliance Cloud stack on Azure and AWS ✅ (Microsoft Learn, AWS documentation).
- **The cloud-file-exchange pattern:** banks and corporates increasingly exchange bulk files via cloud drop zones (e.g., S3 buckets with presigned URLs, managed SFTP services) — cheap, elastic, easy API access — but with a weaker evidence model (no independent third-party log, no network-level non-repudiation) and bilateral setup per counterparty ⚠-knowledge (structural industry knowledge; specific vendor feature claims not verified this pass).
- **The architect's reading:** cloud file exchange wins on cost and developer experience for *non-regulated, bilateral* bulk exchange; FileAct keeps the franchise where *evidence, reach and resilience* are non-negotiable (statements to corporates, regulatory bulk files, market-infrastructure flows). The two coexist: many banks run FileAct for SWIFTNet counterparties and cloud/SFTP for the long tail.

### 6.5 The Head-to-Head Table — Dimension / FileAct / Alternative / Notes

| Dimension | FileAct | SFTP/FTPS | IBM MQ | Cloud file exchange |
|---|---|---|---|---|
| **Network** | SWIFTNet secure IP network (private) ✅ | Internet + VPN | Your own network | Internet/cloud |
| **Community reach** | Thousands of SWIFT users; all members technically enabled ✅ | Bilateral only | Bilateral/internal | Bilateral |
| **Security** | SWIFTNet PKI — authentication, integrity, confidentiality, non-repudiation ✅ | SSH/TLS + your key management | Your own controls | Cloud IAM + TLS |
| **Evidence** | Centralised SWIFT log; independent dispute evidence ✅ | Your own logs | Your own logs | Cloud provider logs |
| **Resilience** | 99.99%+ platform; store-and-forward; auto-recovery ✅ | Depends on your setup | Depends on your setup | Depends on provider SLA |
| **Size limits** | 250 MB per file ✅ | Effectively unlimited (your choice) | Queue/file limits (yours) | Object-store limits (GB–TB) |
| **Cost model** | Usage fee per file, by size and tier ✅ | Bandwidth + infra | License + infra | Pay-per-use storage/transfer |
| **Best for** | Interbank/corporate file lanes with evidence needs | Bilateral corporate integrations | Internal movement to the gateway | Non-regulated bulk exchange |
| **Worst for** | Cheap bilateral one-offs | Evidence-grade interbank traffic | Interbank community exchange | Regulated evidence-grade flows |

### 6.6 Reading the Comparison — the Selection Decision Flow

**Four gates decide the lane (structural, consistent with the rail-selection discipline in [payment_rails_guide.md](payment_rails_guide.md) §7.3):**

1. **Is the counterparty SWIFTNet-connected?** No → SFTP/cloud/portal; yes → continue.
2. **Is the payload a file (batch/document) or a message?** Message → FIN/InterAct; file → FileAct.
3. **Is evidence required?** Delivery notification, non-repudiation, dispute trail → FileAct; informal exchange → any channel.
4. **Is the volume/schedule FileAct-shaped?** >10,000 items, fixed batch windows, large payloads → FileAct; a handful of real-time items → InterAct.

The decision flow is the *same* flow a Cymbal Bank architect runs in §7 — and the answer for statement distribution, bulk payments and regulatory bulk files is, almost always, **the reliable file lane.**

---

## 7. The Worked Example — A Cymbal Bank Statement-File Flow

**The worked example follows the repository convention: Cymbal Bank is a Singapore-based commercial bank, MAS is the SG regulator, and the identifiers are BIC-style and illustrative (not live).** The scenario is the canonical file-lane flow — daily account statements from bank to corporate, plus the reverse bulk-payment variant — built strictly from the verified mechanics of §3.

### 7.1 The Scenario — Cymbal Bank and the Corporate Client

**Cymbal Bank (Singapore) delivers end-of-day account statements to Merlion Trading Pte Ltd (a Singapore corporate with a SWIFT for Corporates / SCORE-style arrangement) over FileAct.** The setup, in the order it must actually happen:

1. **The relationship gate (RMA):** Merlion Trading and Cymbal Bank exchange Relationship Management Application authorisations for the FileAct service — the bank-side process NAB documents ("You will need to organise a Relationship Management Application (RMA) request... The RMA is subject to NAB's acceptance" ✅, §2.4). No RMA, no file lane.
2. **The bilateral convention:** the two parties agree the *business contract* — file formats (camt.053 XML primary, MT940 legacy), the file naming convention (§3.4), the compression rule (ZIP, mandatory), the schedule (§7.3), and the retry/re-submission rules (§7.5). This is the bank-published convention documented by Rabobank and Nordea ✅ (§3.4).
3. **The connectivity:** Merlion Trading connects via Alliance Lite2/Alliance Cloud (the managed lane for corporates — §4.3); Cymbal Bank runs its own SAG estate (the hub — §4.1). Both hold SWIFTNet PKI certificates (§2.3).
4. **The illustrative identifiers:** Cymbal Bank BIC **CYBMSGSG** (illustrative), Merlion Trading BIC **MERLSGSG** (illustrative), accounts in SGD, value dates per the SG banking day ([banks_in_singapore_guide.md](banks_in_singapore_guide.md)).

### 7.2 The Envelope — the FileAct Transfer, Field by Field

**The daily statement leaves Cymbal Bank as one FileAct transfer — an envelope, not a message.** The envelope (structure per §3.1):

| Envelope part | Content (illustrative) | Notes |
|---|---|---|
| **Technical header** | Sender: CYBMSGSG; Receiver: MERLSGSG; Service: FileAct; Delivery mode: store-and-forward; Delivery notification: requested; Priority: normal | The SWIFTNet addressing and routing data ✅ (GS docs §3.1) |
| **Business header** (optional) | File type: CAMT.053; Number of items: 12,345; Total amount: SGD 987,654,321.00; Value date: [today] | The summary block that lets the receiver process without opening the file ✅ (factsheet §3.1) |
| **Payload** | camt.053 XML — the day's booked entries and balances | Any format is allowed; this one is ISO 20022 ✅ |
| **Size budget** | Payload + ~30 KB header, well under the 250 MB per-file limit ✅ (§3.6) | Compression applied at the sender; auto-decompressed at the receiving SNL ✅ (§3.8) |

The header is what makes the flow *operable*: Cymbal Bank's ops team, and Merlion's treasury system, can reconcile the day's delivery from the business header (item count, total amount) without parsing the XML — "easy file handling and processing without the need to open the file" ✅ (factsheet).

### 7.3 The Schedule — the Batch Windows

**The statement flow runs on a fixed daily schedule, and the schedule is what makes it a *file-lane* problem rather than a messaging problem.** The illustrative batch calendar (SGT):

| Window | Activity | Mechanism |
|---|---|---|
| 18:00 (T) | End-of-day posting completes; statement generation starts | Internal (posting_rules_mechanics_guide.md) |
| 19:00 (T) | camt.053 files posted to FileAct in store-and-forward mode | Sender-side SNL; compression on |
| 19:00–07:00 (T+1) | Files held/queued; delivered as Merlion's connection is available | Store-and-forward — "delivered as soon as the recipient is ready" ✅ (§3.2) |
| 07:00 (T+1) | Merlion's treasury system ingests; reconciliation against the business header | Receiver-side processing |
| 08:00 (T+1) | Delivery notifications reconciled; exceptions (non-delivery warnings) actioned | §7.4–§7.5 |

The store-and-forward mode absorbs the time-zone and availability differences — exactly the property the adapter documentation highlights for "large numbers of correspondents... in different time zones" ✅ (§3.2).

### 7.4 The Delivery and the Ack — the Happy Path

**The happy path is a chain of evidence, not just a file movement.** Sequence (all steps verified mechanisms from §3):

1. **Generation:** Cymbal Bank's statement engine produces the camt.053; the business header is populated (item count, total amount) ✅ (§3.1).
2. **Posting:** the file is handed to the gateway (SAG via MQ host adapter — the "MQ inside, FileAct across" pattern ✅ §6.3) and posted as a FileAct transfer signed with Cymbal Bank's SWIFTNet PKI certificate ✅ (§2.3).
3. **Platform checks:** SAG applies the common security/validation/routing checks ✅ (§4.1); SWIFTNet routes the transfer to MERLSGSG.
4. **Store-and-forward delivery:** if Merlion's connection is down, SWIFTNet safe-stores the file and delivers on connection ✅ (§3.2). The file is auto-decompressed by the receiving SNL before Merlion's application sees it ✅ (§3.8).
5. **The delivery notification:** Cymbal Bank receives the delivery notification confirming receipt by the receiver ✅ (§3.3) — the FileAct-layer ack, analogous to the queue consumer ack in [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md).
6. **The business-level ack:** Merlion's treasury system validates the file and (per the bilateral convention) returns a confirmation — e.g., a pain.002-style status or a channel ack — closing the *application-layer* loop ✅-knowledge (bilateral; the business ack format is agreed, not SWIFT-mandated).
7. **Reconciliation:** both sides reconcile using the delivery notification, the business header and the file sequence numbers (§3.4).

### 7.5 The Failure and the Retry — the Recovery Scenario

**The failure scenario is where FileAct's design shows its value — and where the architect must know which layer failed.** Two failure classes:

**Class 1 — Intermittent connectivity failure (the protocol handles it):**
1. At 19:00 Merlion's Alliance Lite2 connection is down (last-mile outage).
2. Cymbal Bank's 19:00 statement file is posted; SWIFTNet accepts it and safe-stores it (store-and-forward ✅).
3. Cymbal Bank's ops team sees a **non-delivery warning** for the transfer ✅ (§3.3) — no panic: the file is not lost.
4. Merlion's connection returns at 21:40; SWIFTNet delivers the file automatically; the **delivery notification** fires ✅.
5. Outcome: delivery at 21:40 instead of 19:00 — the *value date is unaffected* because the statement is for the day just ended, and both sides have full evidence of when delivery occurred ✅ (non-repudiation of emission and reception, §2.3).

**Class 2 — Business-level failure (the protocol cannot fix it):**
1. Merlion's application rejects the delivered camt.053 (schema validation error — a bank-side generation bug).
2. The FileAct transfer itself succeeded (delivery notification received) — the *file lane did its job*; the failure is in the payload.
3. Recovery is bilateral: Cymbal Bank regenerates the corrected file, re-posts it as a **new transfer** with a new sequence number (the naming convention carries the sequence, §3.4), and Merlion reconciles the superseded file against the delivery notifications. ⚠-knowledge: supersede-and-resubmit is the standard bilateral pattern; the exact protocol is per-convention.

**Class 3 — Oversize (the limit bites):** a payment-batch file that would exceed 250 MB must be split into multiple transfers ✅ (§3.6) — the sender's responsibility, since "the communication partner must ensure that the relevant file sizes can be received" ✅ (Bundesbank procedural rules).

### 7.6 The Counterparty → Bank Variant

**The reverse flow — Merlion Trading sends bulk payment files to Cymbal Bank — is the mirror image, with the bank on the receiving side:**

1. Merlion's ERP generates a pain.001 bulk credit-transfer file (e.g., 8,000 supplier payments) and posts it over FileAct to CYBMSGSG, RMA-authorised ✅ (§2.4), within the 250 MB limit ✅.
2. Cymbal Bank receives the file (auto-decompressed by its SNL ✅), validates it against the bilateral convention, and returns a **pain.002 status file** as the business-level ack — the pattern the SWIFT for Corporates manual maps ("FileAct: PAIN.001, PAIN.008" initiation; "FileAct: PAIN.002" status) ✅ (§5.2).
3. Cymbal Bank's hub ([payments_hub_guide.md](payments_hub_guide.md)) splits the batch into payments and routes them to the appropriate rails — FAST/PayNow for local items ([banks_in_singapore_guide.md](banks_in_singapore_guide.md)), the cross-border rails for the rest ([payment_rails_guide.md](payment_rails_guide.md) §2.5) — while FileAct remains the *ingress lane*, not the settlement rail.
4. If the file is rejected (format, size, RMA), the non-delivery/rejection evidence lets both sides reconstruct exactly what happened — the dispute-evidence property ✅ (§8.5).

### 7.7 Where the File Lane Sits in the SWIFT Estate

**The file lane is one lane in a bank's SWIFT estate, and the estate diagram is the architect's mental model:** (textual, all components verified in §2–§4)

```
Internal systems (statements, payments, reports)
        │  files / messages
        ▼
MQ / host adapters ──► SWIFT Alliance Gateway (SAG) ──► SWIFTNet Link (SNL) ──► SWIFTNet (SIPN)
        ▲                 (common checks, PKI signing,          (load balancing,        (FIN / InterAct / FileAct
        │                  concentration, FileAct Copy)          segregation, pacing)     to counterparties)
Alliance Access (routing, release, archive — message/file backbone)
        ▲
Monitoring / ops (delivery notifications, non-delivery warnings, queue status)
```

The estate notes: **RMA** governs who may exchange traffic with whom (the permission layer, §2.4); **PKI certificates** sign every transfer (the identity layer, §2.3); **Alliance Access/SAG** manage the traffic (the product layer, §4); and the **hub** ([payments_hub_guide.md](payments_hub_guide.md)) consumes the files and orchestrates the business. For a Singapore bank, MAS expectations on outsourcing, resilience and record-keeping overlay the whole estate ([mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)), and the 99.99% platform availability claim (§8.1) is what the bank's own availability design builds on ([../technology/zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md)).

---

## 8. The Operations, Availability and Pricing

### 8.1 The Availability and the Reliability Claims

**The headline operational number is verified: "With an availability ratio of above 99.99%, the SWIFTNet platform enjoys the highest reliability rates in the industry" ✅ (swift.com, FileAct factsheet).** Around it: "FileAct is a highly reliable solution to send business-critical files" ✅ (same source); "the sophisticated FileAct protocol automatically recovers from intermittent failures" ✅ (same source); and the platform is the same one that provides "a basis for assuring business continuity and disaster recovery for the infrastructure of mission-critical financial applications" ✅ ("What Is SWIFTNet?"). The architect should still design the bank side for the residual risk — local connectivity, SAG estate, certificate expiry — because 99.99% is the platform's number, not the end-to-end number ⚠-knowledge (structural: end-to-end availability includes the bank's own components).

### 8.2 The Monitoring, the Load Balancing and the Pacing

**FileAct operations are monitoring-shaped, and the platform ships the levers:** delivery notification and non-delivery warning options for the sender ✅, queue-status monitoring for receivers ✅ (§3.3); "spread incoming traffic over multiple SWIFTNet Links to offer optimal load balancing and higher resiliency" ✅; "SWIFTNet Link automatically segregates file transfers from other message flows" to prevent message delays ✅; and "a configurable pacing factor will help you to further reduce the throughput capacity used by FileAct transfers" ✅ (all swift.com, FileAct factsheet, §3.5/§3.9). The adapter-level functions — "file transfer state monitoring" and "abort of transfers in progress" ✅ (Microsoft Learn) — complete the operator's toolkit.

### 8.3 The Pricing — the Usage Fee per File

**The pricing model is verified and size-driven: "SWIFT charges a FileAct usage fee per transferred file. The transfer fees vary according to the size (in bytes) of the file and the institution's global tier" ✅ (swift.com, FileAct factsheet); "FileAct's usage-based pricing structure provides a far more attractive alternative to the recurring costs of a dedicated infrastructure" ✅ (same source).** The architect's reading: FileAct pricing rewards *fewer, larger, well-compressed* files (per-file fee, size-tiered) — which is precisely why compression (§3.8) and batch shaping matter, and why the 250 MB limit (§3.6) is a cost boundary as much as a technical one. ⚠-knowledge: the current price list (SWIFT Price List) was not re-verified this pass.

### 8.4 The FileAct Copy in Operations — Y-Copy and T-Copy

**FileAct Copy is the operational control service: the sender triggers a copy of the file *header* to a third party — Y-copy for authorisation, T-copy for information ✅ (swift.com, FileAct Copy factsheet).** In a bank's operations: a head-office risk function can Y-copy every regional treasury file and authorise or refuse delivery "based on... content of header fields, payer's account balance, collateral, or credit line, or multilateral agreements between counterparties" ✅; a central monitoring unit can T-copy branch traffic "to monitor their activity" ✅; outsourced processing gets "the required information... sent simultaneously to the third party" ✅ (all from the FileAct Copy factsheet). The copy carries the sender's signature, so the third party "can verify the authenticity and integrity of the copied information, without needing to have the actual file" ✅ (same source).

### 8.5 The Dispute Evidence — the Centralised Log

**The end of the evidence chain: "FileAct keeps a centralised log of the file transferred. In the event of a dispute, SWIFT can provide independent evidence confirming that the transfer took place as claimed" ✅ (swift.com, FileAct factsheet).** Combined with non-repudiation of emission and reception ✅ (§2.3) and the delivery notification trail ✅ (§3.3), a bank can answer the three questions every file dispute asks — *was it sent, was it received, when* — with third-party-verifiable evidence rather than its own logs. That property is the operational reason the file lane outranks SFTP for regulated traffic (§6.2, §6.5).

---

## 9. The Summary — One Page

**SWIFTNet FileAct is the reliable file lane: SWIFT's secure store-and-forward file-transfer service over SWIFTNet for files of any format between financial institutions, market infrastructures and corporates — the FILE lane beside FIN's message lane and InterAct's real-time exchange lane.**

- **The platform:** SWIFTNet, the single-window secure IP network; SWIFTNet Link, the mandatory interface with its XML APIs; SWIFTNet PKI, the certificate infrastructure that signs every transfer ✅ (§2).
- **The permissions:** the Relationship Management Application governs which counterparties may exchange FileAct traffic — no RMA, no file lane ✅ (§2.4).
- **The mechanics:** the envelope (technical + business header + payload), store-and-forward and real-time modes, push and pull, delivery notification and non-delivery warning, non-repudiation of emission and reception, automatic recovery, load balancing, 250 MB per-file limit, optional compression, FileAct Copy ✅ (§3).
- **The delivery:** SAG, Alliance Access, Alliance Lite2/Alliance Cloud and the SWIFTReady third-party interfaces — vehicles for the lane, cross-referenced to the payments cluster ✅ (§4).
- **The use cases:** bulk payments, statements (MT940/MT950, camt.052/053/054, BAI2), trade documents, treasury confirmations, regulatory bulk files and market-infrastructure flows ✅/⚠ (§5).
- **The comparisons:** FileAct beats FIN/InterAct on volume, beats SFTP on evidence and reach, complements MQ, and holds the evidence franchise against cloud file exchange ✅/⚠ (§6).
- **The worked example:** a Cymbal Bank statement flow with the envelope, the schedule, the ack chain, and failure-and-retry in three classes ✅/⚠ (§7).
- **The operations:** 99.99%+ platform availability, per-file usage pricing by size and tier, the centralised dispute log ✅ (§8).

**The one-liner that closes the file:** when the payload is a file, the schedule is fixed, and the evidence must be provable, the lane to choose is **the reliable file lane.** — and the full sentence, with every verified anchor behind it, is this guide's final word: for bulk, evidence-bearing, schedule-bound file exchange between SWIFT users, FileAct is **the reliable file lane.** (The claims audit in §10 and the honest flags in §11 follow before the glossary.)

---

## 10. The Claims Audit

**The audit ledger — every key claim in this guide with its verification status. ✅ = verified this pass against a primary source; ⚠ = flagged/unverified or approximate; ⚠-knowledge = well-documented industry knowledge not re-verified live; ❌ = checked and found not supported by the evidence.**

| # | Claim | Status | Source / note |
|---|---|---|---|
| 1 | FileAct is SWIFT's secure service for large file transfers over SWIFTNet, ideal for bulk payments and large-scale reporting | ✅ | swift.com FileAct product page |
| 2 | FileAct transfers any type of data — text, spreadsheets, XML, images; MT/MX, domestic or proprietary formats | ✅ | swift.com FileAct factsheet |
| 3 | FileAct offers push (file transfer) and pull (file download) mechanisms | ✅ | swift.com FileAct factsheet |
| 4 | FileAct has store-and-forward and real-time delivery modes; store-and-forward safe-stores files until the correspondent connects | ✅ | swift.com factsheet; Microsoft Learn adapter docs |
| 5 | FileAct uses SWIFTNet PKI: authentication and integrity control per file, non-repudiation of emission and reception | ✅ | swift.com FileAct factsheet |
| 6 | SWIFT keeps a centralised log of FileAct transfers and can provide independent evidence in disputes | ✅ | swift.com FileAct factsheet |
| 7 | SWIFTNet platform availability is above 99.99% | ✅ | swift.com FileAct factsheet |
| 8 | The maximum FileAct transfer size is 250 MB per physical file (including the ~30 KB FileAct header) | ✅ | Official Bundesbank SWIFTNet FileAct procedural rules (2016 v1.7, 2024-03); Rabobank manual (30 KB header). SWIFT's own Knowledge Centre service description is auth-gated — provenance caveat in §11 |
| 9 | FileAct pricing is a usage fee per transferred file, varying by size in bytes and the institution's global tier | ✅ | swift.com FileAct factsheet |
| 10 | Delivery notification and non-delivery warning options let the sender track delivery status | ✅ | swift.com FileAct factsheet |
| 11 | FileAct protocol automatically recovers from intermittent failures | ✅ | swift.com FileAct factsheet |
| 12 | SWIFTNet Link is the mandatory network interface to SWIFTNet; SAG incorporates SNL and performs the common platform checks | ✅ | Microsoft Learn "What Is SWIFTNet?" + FileAct adapter docs (SWIFT-doc-derived) |
| 13 | RMA is SWIFT's service for managing counterparty authorisations; SWIFTNet 7.0 enabled RMA for InterAct and FileAct | ✅ | swift.com RMA infopaper/training; Axway docs; NAB fact sheet (RMA required for FileAct) |
| 14 | FIN carries individual MT/ISO 15022 messages (11,000+ users, 31.3M+ messages/day) | ✅ | swift.com FIN product page |
| 15 | InterAct carries XML/MX/ISO 20022 messages incl. FX confirmations, statements, reports | ✅ | swift.com InterAct product page |
| 16 | Alliance Lite2 supports FIN, InterAct, FileAct and Browse over SWIFTNet; Alliance Cloud launched 2020 as preferred interface | ✅ | SWIFT Alliance Lite2 Administration Guide; swift.com Lite2 page |
| 17 | FileAct Copy is a value-added service: Y-copy (authorisation) and T-copy (information) of file headers to third parties | ✅ | swift.com FileAct Copy factsheet |
| 18 | FileAct compression is optional at the sender; the receiving SWIFTNet Link auto-decompresses | ✅ | swift.com FileAct factsheet |
| 19 | SWIFTNet Link segregates file transfers from other flows and supports configurable pacing | ✅ | swift.com FileAct factsheet |
| 20 | Statements (MT940/MT942, camt.052/053/054) and initiation (pain.001/pain.008) travel over FileAct in corporate implementations | ✅ | SWIFT for Corporates manual (channel mapping); swift.com ISO 20022 corporates page |
| 21 | CLS: PvP FX settlement since 2002, ~$6T/day, 18 currencies, 70 members, 30,000+ third parties; SWIFT provides the connectivity channel | ✅ | swift.com Swift for Treasury Market Infrastructures |
| 22 | FileAct is used specifically for CLS settlement files | ⚠ | Not confirmed on swift.com CLS page this pass — flagged (§5.6) |
| 23 | FATCA/CRS submissions go to tax authorities via their own channels (SG CRS via IRAS myTax Portal) | ✅ | iras.gov.sg CRS filing page; OECD CRS XML ⚠-knowledge |
| 24 | MAS accepts regulatory returns over FileAct | ⚠/❌ | No primary evidence found this pass; MAS returns use MAS-designated channels — treated as context only (§5.5) |
| 25 | SWIFT demonstrated eBL/trade-document exchange over FileAct (WaveBL proof of value) | ⚠-knowledge | Industry press (secondary); not re-verified on swift.com |
| 26 | FileAct is a settlement rail (moves money itself) | ❌ | FileAct is a file-transfer lane; settlement happens on the rails (payment_rails_guide.md §5) |
| 27 | Browse was retired end-2019 in favour of WebAccess | ✅ | ECB official communication on the migration |
| 28 | FileAct "provides levels of security that aren't available with internet-based file transfer solutions" | ✅ | swift.com FileAct product page (SWIFT's claim; assessment structural in §6.2) |
| 29 | All SWIFT members are technically enabled to exchange FileAct traffic | ✅ | swift.com FileAct factsheet |
| 30 | SWIFT charges per file; current price-list numbers | ⚠ | Price List not re-verified this pass (§8.3) |

## 11. What Could Not Be Verified

**The honest flags — everything this pass could not confirm against a primary source, listed so the reader can weigh each claim:**

- ⚠ **SWIFT's own Knowledge Centre service descriptions are authentication-gated.** The definitive SWIFTNet Service Description ("Features and Functions — SwiftNet Service Description", www2.swift.com/knowledgecentre) and the SwiftNet Link 7.7 Service Description could not be read directly this pass. Where this guide cites them, it relies on (a) official documents that mirror them (Bundesbank procedural rules), (b) SWIFT-document-derived integration documentation (Microsoft Learn BizTalk adapter pages), or (c) swift.com public pages. The 250 MB limit is therefore verified via the Bundesbank's official procedural rules and bank manuals rather than the auth-gated SWIFT original — with the caveat that SWIFT adjusts service parameters by release, so re-check the current number before freezing a design.
- ⚠ **The current SWIFT Price List** for FileAct (per-file fees by size tier) was not re-verified; only the pricing *model* (usage fee per file, size- and tier-based) is verified from the factsheet.
- ⚠ **FileAct's specific role in CLS settlement flows** — the swift.com CLS page documents the FIN MT channel and the ISO 20022 MI channel, not FileAct; the general market-infrastructure use of FileAct is verified, the CLS-specific one is not.
- ⚠ **MAS regulatory reporting over FileAct** — no primary evidence found that MAS accepts returns over FileAct; the MAS angle in §5.5 is context only. Similarly, **FATCA/CRS over FileAct** is not supported for Singapore (IRAS myTax Portal is the CRS channel).
- ⚠ **Trade-document exchange over FileAct (eBL/WaveBL)** — reported in industry press only; not re-verified on swift.com this pass.
- ⚠ **File naming conventions** — confirmed as bilateral, bank-defined (Rabobank, Nordea documents); the specific per-bank rules were not enumerated.
- ⚠ **Payload-level encryption beyond the transport** (e.g., PGP on top of FileAct) — bilateral bank policy; not a SWIFT-mandated FileAct feature.
- ⚠ **The SWIFTNet retirement/evolution roadmap** after the ISO 20022 FINplus migration — evolving; not re-verified this pass.
- ⚠ **The current Alliance Access feature set** (release workflows, dual-site operation) — deliberately not re-derived; cross-referenced to the payments cluster (§4.5).
- ⚠-knowledge **industry-scale figures** (e.g., "thousands of Swift users ready to exchange files over FileAct" — swift.com's own wording ✅ — and the general SFTP/cloud comparison claims) are structural or approximate where not otherwise marked.
- ⚠ **The 2016 Bundesbank procedural rules document** could not be re-extracted in full (rate-limited on re-fetch); the 250 MB quote is taken from the official document's indexed text as returned by the search engine, and corroborated by the 2024-03 version's identical statement and the Rabobank manual.

## 12. Glossary

| Term | Meaning |
|---|---|
| **FileAct** | SWIFT's secure store-and-forward file-transfer service over SWIFTNet — the file lane of the SWIFT estate |
| **SWIFTNet** | SWIFT's secure IP network and platform — the single-window interface carrying FIN, InterAct and FileAct |
| **SIPN** | SWIFT Secure IP Network — the private network that interconnects SWIFT users |
| **SNL (SWIFTNet Link)** | The mandatory client-side network interface to SWIFTNet; provides load balancing, routing, transport security, and the SwCall/SwCallback APIs |
| **SAG (SWIFTAlliance Gateway)** | The on-premise connectivity hub incorporating SNL; performs the common platform checks and concentrates traffic via host adapters (e.g., WebSphere MQ) |
| **Alliance Access** | SWIFT's messaging/file backbone application for routing, release, queueing and archiving of SWIFTNet traffic |
| **Alliance Lite2 / Alliance Cloud** | SWIFT's managed, internet-based connectivity (Lite2) and its 2020 cloud evolution (Alliance Cloud) — support FIN, InterAct, FileAct |
| **SWIFTNet PKI** | The public-key infrastructure that authenticates and integrity-protects every SWIFTNet transfer and provides non-repudiation |
| **RMA (Relationship Management Application)** | SWIFT's service for managing counterparty authorisations — which counterparties may send/receive traffic for a service |
| **FIN** | SWIFTNet's core service for individual structured MT / ISO 15022 messages |
| **InterAct** | SWIFTNet's service for XML-based (MX/ISO 20022) messages, including real-time request/reply |
| **FileAct header / business header** | The envelope metadata: technical header (sender/receiver/routing) and optional business header (item count, total amount, file type) |
| **Store-and-forward** | Delivery mode in which SWIFTNet safe-stores a file and delivers it when the recipient connects |
| **Delivery notification / non-delivery warning** | Options that let the sender track whether a file was delivered |
| **Non-repudiation of emission/reception** | The PKI-signed property that neither side can deny sending or receiving a file; SWIFT can evidence it from its centralised log |
| **FileAct Copy (Y-copy / T-copy)** | Value-added service copying file headers to a third party — Y-copy for authorisation, T-copy for information |
| **Pacing factor** | A configurable throttle for FileAct throughput on a SWIFTNet Link |
| **MT940/MT950/MT942** | SWIFT MT statement message types (customer statement, account statement, intraday) |
| **camt.052/053/054** | ISO 20022 cash-management messages (intraday report, bank-to-customer statement, debit/credit notification) |
| **pain.001 / pain.002 / pain.008** | ISO 20022 payment-initiation messages (credit transfer initiation, status report, direct debit initiation) |
| **BAI2** | The BAI file-format standard for bank statements, common in the US/APAC corporate market |
| **SCORE** | Standardised Corporate Environment — the SWIFT framework under which corporates access SWIFT services |
| **MI Channel** | SWIFT's market-infrastructure channel — high-throughput batching, compression, resilient store-and-forward for MI communities |
| **CLS** | Continuous Linked Settlement (CLS Group) — the FX PvP settlement system (2002), reached via SWIFT |
| **SWIFTReady** | SWIFT's certification label for third-party applications, services and connectivity |
| **CUG** | Closed User Group — a restricted community of SWIFTNet users authorised to exchange traffic |

## 13. Cross-References and Further Reading

**Repo siblings (banking/ — plain filenames):**

- [payment_rails_guide.md](payment_rails_guide.md) — the rails map: SWIFT as the cross-border rail, gpi, and the rail-selection discipline this guide's lane-selection mirrors (§6.6)
- [payments_hub_guide.md](payments_hub_guide.md) — the hub architecture where the FileAct lane plugs in (routing, transformation, orchestration of file traffic)
- [swift_alliance_access_guide.md](swift_alliance_access_guide.md) — the SWIFT-connectivity deep-dive (the Alliance family); this guide is the FILE lane companion to its messaging flows — the file lane of the same estate (§4)
- [banks_in_singapore_guide.md](banks_in_singapore_guide.md) — the SG rails (FAST, PayNow, MEPS+) and the SG banking day used in the worked example
- [mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md) — the MAS reporting regime and outsourcing expectations overlaying the estate (§5.5, §7.7)
- [capital_markets_architecture_guide.md](capital_markets_architecture_guide.md) — message standards and market-infrastructure flows (cross-ref lightly)
- [treasury_alm_guide.md](treasury_alm_guide.md) — treasury confirmations and liquidity (cross-ref §5.4)
- [posting_rules_mechanics_guide.md](posting_rules_mechanics_guide.md) — statement-to-posting mechanics downstream of the statement file (§5.2, §7.3)

**Repo siblings (technology/ — prefix `../technology/`):**

- [../technology/zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md) — the 99.99% availability claims and cut-over windows (§8.1)
- [../technology/message_queue_data_loss_guide.md](../technology/message_queue_data_loss_guide.md) — at-least-once delivery and ack semantics vs the FileAct delivery notification (§3.3, §6.3)
- [../technology/legacy_integration_patterns_guide.md](../technology/legacy_integration_patterns_guide.md) — the SAG/SNL adapter patterns (§4)
- [../technology/security_by_design_guide.md](../technology/security_by_design_guide.md) — PKI and non-repudiation by design (§2.3, §3.8)

**Primary sources used this pass (all accessed 2026-08-27):**

- swift.com — FileAct product page (products/fileact); FileAct factsheet (swift_messaging_factsheet_fileact.pdf); FileAct Copy factsheet; FIN product page; InterAct product page; SwiftNet Link product page; FileAct ordering page; Alliance Lite2 product and login pages; Swift for Treasury Market Infrastructures (CLS) page; MI Channel product description; Swift Web Access page and WebAccess resource; SWIFT RMA compliance infopaper; ISO 20022 corporates page; RMA Service training catalogue entry
- bundesbank.de — "SWIFTNet FileAct procedural rules" (Version 1.7, 2016; 2024-03 update) — the 250 MB per-file limit and the Bundesbank FileAct reporting channel
- ecb.europa.eu — Communication on the migration from SWIFT Browse to SWIFT WebAccess (2019)
- iras.gov.sg — CRS filing page (SG CRS via myTax Portal)
- Microsoft Learn — BizTalk FileAct/InterAct adapter documentation and "What Is SWIFTNet?" (SWIFT-document-derived)
- IBM documentation — Messaging Services for SWIFTNet InterAct and FileAct (MSIF); relationship management (RMA)
- Axway documentation — About RMA; Secure Financial Client SWIFTNet support and size limits
- NAB — SWIFT FileAct Fact Sheet (2019); Nordea — Security and Communication description for SWIFTNet FileAct; Rabobank — SWIFT FileAct manual (via public document mirrors)
- Goldman Sachs developer documentation — Swift FileAct service description; AFP — SWIFT Corporate Access Implementation, A Practitioners Guide
- AWS and Microsoft Azure — deployment guidance for SWIFT Alliance Lite2 / Alliance Cloud on hyperscalers
- SWIFT Alliance Lite2 Administration Guide (public mirror); SWIFT for Corporates manual (public mirror)

**The final word of the guide:** for bulk, evidence-bearing, schedule-bound file exchange between SWIFT users — statements, batch payments, regulatory bulk files, documents — the lane that delivers with PKI-grade security, store-and-forward resilience and provable delivery is **the reliable file lane.**
