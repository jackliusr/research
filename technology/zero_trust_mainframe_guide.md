# Zero Trust for the Mainframe Estate

*The constrained-case deep-dive — zero trust applied to the platform that cannot be a policy-enforcement point: what the mainframe's own security model already does (and does better than most modern stacks), the five weaknesses that are actually real, the identity bridge that integrates it with the enterprise identity and audit planes, the 3270 access path, the file-transfer and spool exposure, the network-side and audit-side answers, and the residual risk that must be owned in writing — all resting on one thesis: the mainframe is not insecure; it is un-integrated.*

*Jack Liu Shurui, Solution Architect*

**Last Updated:** October 2026

**Purpose.** This guide is the dedicated deep-dive for the **constrained case** of zero trust: the systems that cannot themselves be a policy-enforcement point (PEP). The enterprise identity plane (Okta, Microsoft Entra ID, the ZTNA broker) and the enterprise audit plane (the SIEM) both assume a subject that can be challenged and a log that can be streamed. A z/OS mainframe estate is neither, by default — and yet it runs the workloads that matter most. This guide states precisely what the platform's own security model does, names what it cannot do, and designs the bridge between the two planes. It deliberately does **not** re-derive zero-trust architecture, its pillars, NIST SP 800-207, the vendor landscape or the estate-wide migration phasing — those belong to [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md), which this guide cross-references by name and treats as its parent.

**Verification-markers convention.** ✅ = verified against a primary source (vendor documentation, standards body, regulator page) during this pass or in a cross-referenced sibling's ledger. ⚠ = flagged or partial — practice varies, or the claim rests on industry-standard practice rather than a single primary source. ❌ = could not be verified at source; recorded honestly in §16. The consolidated ledger is the Claims Audit (§15).

**How this guide is organised.** §1 is the overview, the decoder and the boundary. §2 describes what the platform *is*. §3 is the platform's own security model — what it already does. §4 is where the real weaknesses are (the five). §5 is the identity bridge, the central architectural section. §6 is privileged access, vaulting and the named identity. §7 is the 3270 access path and session recording. §8 is the file-transfer and spool exposure. §9 is the network-side answer. §10 is the audit-side answer. §11 is what genuinely cannot be changed, and the residual risk that must be owned. §12 is the phased application. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified and closes. The glossary sits after §16.

---

## Table of Contents

1. [The Overview, the Decoder and the Boundary](#1-the-overview-the-decoder-and-the-boundary)
   - 1.1 [The Thesis — Not Insecure, Un-Integrated](#11-the-thesis--not-insecure-un-integrated)
   - 1.2 [The Scope — the System That Cannot Be a PEP](#12-the-scope--the-system-that-cannot-be-a-pep)
   - 1.3 [The Decoder](#13-the-decoder)
   - 1.4 [The Boundary — What This Guide Does Not Own](#14-the-boundary--what-this-guide-does-not-own)
2. [What the Platform Actually Is](#2-what-the-platform-actually-is)
   - 2.1 [An Operating Environment, Not a Server](#21-an-operating-environment-not-a-server)
   - 2.2 [The Workload Managers](#22-the-workload-managers)
   - 2.3 [The Job-Entry Subsystem and the Spool](#23-the-job-entry-subsystem-and-the-spool)
   - 2.4 [The Dataset and File Model](#24-the-dataset-and-file-model)
   - 2.5 [The Network Entry Points](#25-the-network-entry-points)
   - 2.6 [The Platform Table](#26-the-platform-table)
3. [What the Platform's Own Security Model Already Does](#3-what-the-platforms-own-security-model-already-does)
   - 3.1 [The Access-Control Facility — RACF and the SAF Interface](#31-the-access-control-facility--racf-and-the-saf-interface)
   - 3.2 [Resource Classes and Profiles](#32-resource-classes-and-profiles)
   - 3.3 [Dataset and Resource Protection](#33-dataset-and-resource-protection)
   - 3.4 [The Identity Classes — Users, Groups, Started Tasks](#34-the-identity-classes--users-groups-started-tasks)
   - 3.5 [The Audit Records](#35-the-audit-records)
   - 3.6 [What This Model Does That a Modern Stack Frequently Does Not](#36-what-this-model-does-that-a-modern-stack-frequently-does-not)
4. [Where the Real Weaknesses Are](#4-where-the-real-weaknesses-are)
   - 4.1 [The Five Weaknesses — the Table](#41-the-five-weaknesses--the-table)
   - 4.2 [(i) Shared and Static Credentials](#42-i-shared-and-static-credentials)
   - 4.3 [(ii) The File-Transfer Path](#43-ii-the-file-transfer-path)
   - 4.4 [(iii) The Spool and Printed Output Queue](#44-iii-the-spool-and-printed-output-queue)
   - 4.5 [(iv) The 3270 Access Path and the Emulator Session](#45-iv-the-3270-access-path-and-the-emulator-session)
   - 4.6 [(v) Third-Party and Vendor Support Access](#46-v-third-party-and-vendor-support-access)
5. [The Identity Bridge](#5-the-identity-bridge)
   - 5.1 [The Two Planes](#51-the-two-planes)
   - 5.2 [The Platform-Side Mechanisms](#52-the-platform-side-mechanisms)
   - 5.3 [The Enterprise-Side Half](#53-the-enterprise-side-half)
   - 5.4 [Provisioning and Deprovisioning — the Joiner-Mover-Leaver Problem](#54-provisioning-and-deprovisioning--the-joiner-mover-leaver-problem)
   - 5.5 [The Group and Profile Model on the Platform Side](#55-the-group-and-profile-model-on-the-platform-side)
   - 5.6 [Why the Bridge Is the First Thing to Build](#56-why-the-bridge-is-the-first-thing-to-build)
6. [Privileged Access, Vaulting and the Named Identity](#6-privileged-access-vaulting-and-the-named-identity)
   - 6.1 [Vaulting and Brokering](#61-vaulting-and-brokering)
   - 6.2 [The Firecall (Emergency) Identity](#62-the-firecall-emergency-identity)
   - 6.3 [Session Brokering to the Platform](#63-session-brokering-to-the-platform)
   - 6.4 [The Named Identity — the Most Wasted Capability](#64-the-named-identity--the-most-wasted-capability)
7. [The 3270 Access Path](#7-the-3270-access-path)
   - 7.1 [The Emulator and the Interactive Session](#71-the-emulator-and-the-interactive-session)
   - 7.2 [The Identity-Aware Gateway](#72-the-identity-aware-gateway)
   - 7.3 [Session Recording as a Compensating Detective Control](#73-session-recording-as-a-compensating-detective-control)
   - 7.4 [What Recording Does and Does Not Prove](#74-what-recording-does-and-does-not-prove)
8. [The File-Transfer and Spool Exposure](#8-the-file-transfer-and-spool-exposure)
   - 8.1 [The Transfer Path End to End](#81-the-transfer-path-end-to-end)
   - 8.2 [The Transfer Identity and Directory Scoping](#82-the-transfer-identity-and-directory-scoping)
   - 8.3 [Encryption and Authentication on the Path](#83-encryption-and-authentication-on-the-path)
   - 8.4 [The Batch-Scheduling Dependency](#84-the-batch-scheduling-dependency)
   - 8.5 [The Spool and Printed Output as a Disclosure Path](#85-the-spool-and-printed-output-as-a-disclosure-path)
9. [The Network-Side Answer](#9-the-network-side-answer)
   - 9.1 [Segmentation Around the Platform](#91-segmentation-around-the-platform)
   - 9.2 [The Entry Points That Must Remain](#92-the-entry-points-that-must-remain)
   - 9.3 [The Observation Point When the Platform's Own Logging Is Not Centralised](#93-the-observation-point-when-the-platforms-own-logging-is-not-centralised)
10. [The Audit-Side Answer](#10-the-audit-side-answer)
   - 10.1 [The Platform's Audit Records](#101-the-platforms-audit-records)
   - 10.2 [The Collection Gap](#102-the-collection-gap)
   - 10.3 [Integrity, Retention and Volume](#103-integrity-retention-and-volume)
   - 10.4 [What Becomes Possible Once the Feed Exists](#104-what-becomes-possible-once-the-feed-exists)
11. [What Genuinely Cannot Be Changed — and the Residual Risk That Must Be Owned](#11-what-genuinely-cannot-be-changed--and-the-residual-risk-that-must-be-owned)
   - 11.1 [Immutable Versus Merely Unattempted](#111-immutable-versus-merely-unattempted)
   - 11.2 [The Ownership Requirement](#112-the-ownership-requirement)
   - 11.3 [The Residual-Risk Register](#113-the-residual-risk-register)
12. [The Phased Application](#12-the-phased-application)
   - 12.1 [The Phases](#121-the-phases)
   - 12.2 [Mapping to the ZTNA Plan and the CISA Maturity Model](#122-mapping-to-the-ztna-plan-and-the-cisa-maturity-model)
   - 12.3 [Which Phase Delivers the Most per Unit of Effort](#123-which-phase-delivers-the-most-per-unit-of-effort)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
   - 13.1 [The Scenario](#131-the-scenario)
   - 13.2 [The Findings](#132-the-findings)
   - 13.3 [The Design — Bridge First](#133-the-design--bridge-first)
   - 13.4 [The Accepted Residual Risk](#134-the-accepted-residual-risk)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified and the Closing Summary](#16-what-could-not-be-verified-and-the-closing-summary)
- [The Glossary](#the-glossary)

---

## 1. The Overview, the Decoder and the Boundary

### 1.1 The Thesis — Not Insecure, Un-Integrated

**The mainframe is not insecure. It is un-integrated.** That sentence is the whole guide, and every section that follows is an unpacking of it. The platform was built with a security model that is, in several respects, *more* rigorous than the average modern application stack: a resource-level access-control facility that has been enforcing dataset-by-dataset, transaction-by-transaction, command-by-command authorisation for decades; its own audit records, long-established and comprehensive; its own workload identities and started-task model; and an external security manager (ESM) that predates, by a very long way, the phrase "zero trust" — and implements more of it, at the resource level, than most of the systems that use the phrase.

What the platform does **not** do is participate in the two planes the modern enterprise security function has come to depend on. It is usually not a subject of the enterprise identity plane (no aligned joiner-mover-leaver provisioning, no single sign-on, no device posture, no per-person identity for the work that matters most) and it is usually not a complete source for the enterprise audit plane (its rich records exist on the platform but are frequently neither collected nor correlated). The gap is an integration gap, not a controls gap. The mainframe is the estate's most controlled platform and, simultaneously, one of its least observed.

Two mischaracterisations follow from that gap, and this guide rejects both:

1. **"The mainframe is a legacy box we cannot secure."** False. The platform's own security model is mature and granular (§3). That it is not pointed at by the enterprise's identity tooling is an integration failure, not evidence of platform weakness.
2. **"The mainframe is secure precisely because it is closed, so we can leave it alone."** Also false. The five weaknesses in §4 — shared and static credentials, the ungoverned file-transfer path, the spool as a disclosure path, the 3270/emulator path, and ungoverned vendor support access — are real, they persist because nobody owns the integration, and a network control in front of them changes none of it.

The corrective is a **bridge**: extend the enterprise identity plane onto the platform and extend the platform's audit records out to the enterprise audit plane, then let the platform's own resource-level model do the enforcement it was always designed to do (§5, §10). Where the bridge genuinely cannot reach, the remaining exposure is *named, owned, reviewed and accepted in writing* (§11) — because a residual risk that is named is a managed risk, and the same risk unstated is the finding.

### 1.2 The Scope — the System That Cannot Be a PEP

This guide owns the **constrained case**: zero trust applied to a platform that **cannot be the policy-enforcement point** for its own access decisions. The canonical modern pattern — an identity-aware proxy or ZTNA broker that challenges every session, evaluates device posture and brokered policy, and grants per-application access — assumes the protected resource sits behind a PEP that the enterprise controls. A z/OS estate's interactive and batch entry points do not, by default, sit behind such a PEP, and the platform itself does not enforce device posture, conditional access or the enterprise's session-level policy. It enforces *authorisation* — extremely well — but it is not a modern access-decision point.

So this guide's territory is:

- the platform's actual security model and what it already covers (§2, §3);
- the honest list of real weaknesses (§4);
- the **identity bridge** that joins the platform to the enterprise identity plane (§5);
- privileged access, vaulting and the named identity on the platform (§6);
- the **3270 access path** and session recording as a compensating control (§7);
- the **file-transfer and spool** exposure (§8);
- the **network-side** and **audit-side** answers (§9, §10);
- what genuinely cannot change, and the residual risk that must be owned (§11);
- the phased application (§12) and the Cymbal Bank worked example (§13).

What this guide does **not** own: zero trust as a discipline, its history, NIST SP 800-207 and its tenets, the five pillars, the vendor landscape, or the estate-wide migration phasing — all of which belong to [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md). This guide presents its own phasing (§12) as an **application** of that guide's §9.3 plan to the constrained platform, not a competing one.

### 1.3 The Decoder

Zero trust applied to the mainframe fails at first contact for one avoidable reason: the two sides do not share a vocabulary. The table below is the decoder this guide uses throughout. Every term is either the platform's own or the enterprise's, and every mapping is the thing the bridge actually does.

| Term | What it is | Enterprise-side analogue |
|---|---|---|
| **Access-control facility / ESM** | The external security manager that enforces authorisation on the platform — IBM **RACF** (Resource Access Control Facility), or Broadcom **CA ACF2** / **CA Top Secret**; all plug into the **SAF** (System Authorization Facility) interface ✅ | The IAM/policy-decision engine |
| **SAF** | The *system interface* by which z/OS resource managers route authorisation requests; SAF "conditionally directs control to the Resource Access Control Facility (RACF), if RACF is present, and/or a user-supplied processing routine" ✅ — an interface, **not** an ESM | The PDP/PEP plumbing between resource and policy |
| **Userid** | A 1-to-8-character identity string; "Every job, started task, or transaction on z/OS has associated with it an identity" ✅ | The account / principal |
| **ACEE** | The Accessor Environment Element, the z/OS control block representing a user's identity; created by the ESM on request by resource managers (UNIX System Services, JES, CICS, IMS) via SAF APIs such as `RACROUTE REQUEST=VERIFY` and `InitACEE` ✅ | The authenticated session's principal token |
| **Profile / class** | The ESM's unit of protection: a **profile** inside a resource **class** (e.g. `DATASET`, `USER`, `GROUP`, `STARTED`, `JESSPOOL`) grants or denies access ✅ | A policy rule / role binding |
| **Started task** | A long-running address space with an assigned identity; the **STARTED** class "can assign different user IDs and group names to the same started member, depending on the job name" ✅ | A service account / workload identity |
| **Workload manager** | A subsystem that runs work with its own identity semantics — CICS TS (transactions), IMS (TM and DB), DB2 for z/OS, IBM MQ for z/OS — distinct from the z/OS **Workload Manager (WLM)**, which is the goal-based *performance* manager ✅ (do not conflate) | The runtime / service |
| **Region** | An address space hosting a subsystem (e.g. a CICS region, an IMS control region); region-level security settings gate the resource-level checks | The service instance / namespace |
| **Dataset protection** | Authorisation applied to individual datasets, dataset prefixes (high-level qualifiers) and generic patterns — **at the platform level**, not inside an application ✅ | File/object ACLs, but platform-enforced |
| **Audit record** | The platform's long-established record stream: **SMF** (System Management Facilities), e.g. record type 80 = RACF processing, type 14/15 = dataset activity ✅ | The log event / audit event |
| **Spool** | The job-entry subsystem's output queue (JES2/JES3), readable via SDSF/SAPI; protectable via the RACF **JESSPOOL** class ✅ | The print/log queue — a disclosure path |
| **Emulator** | The 3270 terminal-emulation client (or browser emulator) that presents an interactive session — the path whose identity is frequently shared | The remote-access client |
| **Transfer identity** | The userid under which file transfer (FTP/SFTP, Connect:Direct, Transfer CFT) authenticates — frequently privileged and widely scoped ✅ | The file-transfer service account |
| **Firecall / emergency ID** | A break-glass identity for emergencies, with elevated authority, used when normal access is unavailable ✅-structural | The break-glass account |
| **The bridge** | The set of mechanisms that map enterprise identities into platform userids and stream platform records out — RACF **identity propagation** (IDID/ICRX/RACMAP/IDIDMAP), certificate mapping, Kerberos, PassTicket on the way in; **IBM Z Common Data Provider** and SMF on the way out ✅ | The provisioning connector + the log shipper |

### 1.4 The Boundary — What This Guide Does Not Own

The repo's security cluster is deliberately de-duplicated. This guide re-derives nothing from its siblings; it cross-references them **by name** and states the boundary explicitly.

| Sibling guide | What it owns — do not re-derive here | How this guide uses it |
|---|---|---|
| [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) | Zero trust generally; §1 the ZTNA overview, §2 the history (Kindervag 2010, BeyondCorp 2014), §3 NIST SP 800-207 and the seven tenets, §4 the five pillars, §5 the architecture, §6 the vendors, §7 the implementation and the CISA ZTMM, §9.3 the Cymbal Bank phased plan (Phase 0 Assess → 1 Identity hardening → 2 ZTNA the remote estate → 3 Segment the data plane → 4 Automate) | Cross-ref §9.3 and §7.1 by name; present §12 here as an **application** of that plan to the constrained platform |
| [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) | The Beyond Zero paper and its thesis | Cross-ref only |
| [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) | The legacy estate from the **integration** side — §6 the patterns, §7 the modernisation patterns, §10 "Integrate, Don't Replace" | Cross-ref for the estate's shape; re-derive none of it |
| [network_packet_capture_guide.md](network_packet_capture_guide.md) | The network-side monitoring answer where the platform's own logging is not in the SIEM | Cross-ref in §9 as the complementary observation point |
| [cybersecurity_guide.md](cybersecurity_guide.md) | The security-discipline overview | Cross-ref, no re-derivation |
| [secops_guide.md](secops_guide.md) | Security operations — detection, response, the SOC | Cross-ref in §10 for the feed's consumer |
| [threat_modeling_guide.md](threat_modeling_guide.md) | Threat-modelling methodology (STRIDE, DFDs) | Cross-ref in §4 for the threat-model lens |
| [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) and [linux_file_sharing_notification.md](linux_file_sharing_notification.md) | The file-sharing paths and their notification patterns | Cross-ref in §8 for the transfer-path mechanics |
| [control_m_guide.md](control_m_guide.md), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) | The MFT / workload-automation products and their integration | Cross-ref in §8 for the transfer and scheduling path |

The rule for the rest of this guide: **state the platform's own facts; cross-reference the enterprise's.** Where a fact is the enterprise's (the identity plane, the SIEM, the ZTNA broker), name the sibling and move on.

---

## 2. What the Platform Actually Is

### 2.1 An Operating Environment, Not a Server

The first step in securing the constrained platform is refusing to call it "a server" or "an application." **IBM z/OS is an operating system — a large, mature, multi-address-space operating system — running on IBM Z hardware.** It is virtualised at the hardware level through **PR/SM** logical partitions (LPARs), which is why a single frame can host several independent z/OS images and, alongside them, **z/VM** (a hypervisor for hosting many guest operating systems) and **LinuxONE** (Linux on the same hardware family) ✅. Each z/OS image has its own operating system, its own security database, its own workload managers, and its own audit records.

The consequence for security architecture is immediate: **you are not securing one thing, you are securing an operating environment with an internal structure** — multiple images, multiple subsystems per image, multiple identities per subsystem, and multiple entry paths. A "mainframe estate" is a fleet of operating systems, and the identity bridge (§5) has to reach each image's security database, not one central directory.

### 2.2 The Workload Managers

The work the enterprise cares about runs under subsystems, each with its own identity and resource semantics:

- **CICS TS** — the transaction server. CICS applies transaction-level and resource-level security through the ESM: "TCICSTRN is the general resource class for transactions while GCICSTRN is the resource group class for transactions" ✅ (IBM CICS TS documentation). A CICS transaction runs under an identity that may be the region's (a serious finding, §4) or, properly, a propagated one.
- **IMS** — the transaction manager (IMS TM) and database manager (IMS DB), with its own ESM integration and its own dual-identity model.
- **DB2 for z/OS** — the database; it can assign authorisation based on connection characteristics via **trusted context and roles**, including source user ID and connection source ✅ (IBM).
- **IBM MQ for z/OS** — messaging, with queue-level authority checks routed through SAF.
- **Batch** — job streams written in JCL and executed by the job-entry subsystem, typically scheduled by an external workload-automation product (the MFT/scheduler guides in §8's cross-references).
- **z/OS Workload Manager (WLM)** — the *performance* manager that dispatches work to meet goals. It is **not** a transaction monitor and **not** a security component; the naming collision with CICS/IMS "workload managers" is a genuine source of confusion ✅ (flagged in the decoder).

### 2.3 The Job-Entry Subsystem and the Spool

Batch and much systems work enters the platform through a **job-entry subsystem** — **JES2** or **JES3** — which accepts jobs, manages queues, dispatches work, and holds output in the **spool**. The spool is read and searched through **SDSF** (System Display and Search Facility) and programmatically through **SAPI** (the spool-access programming interface) ✅ (IBM). Two properties matter for this guide: the spool is an **information-disclosure path** if left under-scoped (§4(iii), §8.5), and it is **protectable** via the RACF **JESSPOOL** class — which is the honest rebuttal to "the mainframe leaks output" (§3.3).

### 2.4 The Dataset and File Model

The platform's data model predates POSIX and then absorbed it:

- **Sequential datasets** and **partitioned datasets (PDS/PDSE)** — the classic z/OS dataset world, named by **high-level qualifiers** (HLQs) and dots (`PROD.PAYMENTS.EOD.FILE`), protected as *datasets* by the ESM.
- **VSAM** — indexed and relative-record datasets used by transaction workloads.
- **z/OS UNIX** — the POSIX side: an **HFS/zFS** file system with directories, files, **UID/GID** mapping for userids (an OMVS segment in the user profile), and file permissions that coexist with, and can be overlaid by, the ESM's own checks ✅-structural.
- **RACF key rings and digital certificates** — keystores for TLS and code signing, managed through `RACDCERT` ✅ (IBM).

The security-relevant point: **dataset protection is a platform primitive, not an application feature** (§3.6) — but it operates in two worlds (dataset-name space and the UNIX file space) whose scopes do not automatically align, which is exactly how "wide directory scope" becomes a transfer-path weakness (§8.2).

### 2.5 The Network Entry Points

The platform is reached through a small number of well-defined entry classes, each of which is a chapter in this guide:

| Entry class | Protocol / path | Guide coverage |
|---|---|---|
| **Interactive** | TN3270 / 3270 emulation (and browser emulators) into TSO/E, ISPF, CICS | §7 |
| **File transfer** | FTP/FTPS, z/OS OpenSSH (SFTP/SCP), Connect:Direct, Transfer CFT | §8 |
| **Transaction** | CICS and IMS transaction ports (external communication) | §2.2, §9.2 |
| **Messaging** | IBM MQ channels | §9.2 |
| **Web / API** | WebSphere, and **z/OS Connect** exposing CICS/IMS as REST/JSON APIs ⚠ (verify product detail before asserting) | §5.3, §9.2 |
| **Systems management / console** | Operator consoles, z/OSMF (z/OS Management Facility), SDSF, systems-management tooling | §6, §9.2 |

### 2.6 The Platform Table

| Aspect | The platform's reality | Security consequence |
|---|---|---|
| **Operating system** | IBM z/OS on IBM Z, with PR/SM LPARs, plus z/VM and LinuxONE ✅ | Multiple independent images, each with its own security database — the bridge must reach each |
| **Virtualisation** | Hardware LPARs (PR/SM); z/VM hypervisor optionally ✅ | Segmentation starts at the LPAR, not at a network zone |
| **Security facility** | An ESM (RACF, or ACF2/Top Secret) behind the SAF interface ✅ | Resource-level authorisation already exists — the gap is integration, not enforcement |
| **Identity primitive** | 1–8-char userid; ACEE per job/started task/transaction ✅ | Short, numerous, hand-managed identities — the JML problem (§5.4) |
| **Workload identity** | STARTED class assigns userids/groups to started tasks ✅ | Where workload identities are good (per-service) and where shared batch identities hide (§4.2) |
| **Audit** | SMF records, long-established, with RACF type 80 ✅ | The richest trail in the estate, frequently uncollected (§10) |
| **Job entry / spool** | JES2/JES3, SDSF/SAPI; spool protectable via JESSPOOL ✅ | Disclosure path if un-scoped (§4.4, §8.5) |
| **Interactive path** | TN3270 emulation | Cannot be the PEP for its own logon; needs a gateway (§7.2) |
| **Data** | Dataset-name space + z/OS UNIX file space | Two scopes that do not automatically align (§8.2) |

---

## 3. What the Platform's Own Security Model Already Does

The single most important correction this guide makes is here: **the platform's security model is not a gap to be filled; it is a working, mature, resource-level authorisation system that most enterprise security teams have simply never operated.** What follows is established from IBM's own documentation, with the verified seed facts marked ✅.

### 3.1 The Access-Control Facility — RACF and the SAF Interface

**RACF — the Resource Access Control Facility — is IBM's External Security Manager, a component of the z/OS Security Server** ✅. It is not an add-on; it is part of the operating system's security architecture, and it has been for decades. RACF sits behind **SAF — the System Authorization Facility** — which is the system interface, not the security manager itself: SAF "conditionally directs control to the Resource Access Control Facility (RACF), if RACF is present, and/or a user-supplied processing routine when receiving a request from a resource manager" ✅ (IBM, *System Authorization Facility (SAF)*), and "SAF either processes security authorization requests directly or works with RACF, or other security product, to process them," controlling access to "resources, such as data sets and MVS commands" ✅ (IBM, *What is SAF?*).

The SAF/RACF split matters architecturally for three reasons:

1. **SAF is an interface, so the ESM is replaceable.** IBM RACF is one implementation; Broadcom's **CA ACF2** and **CA Top Secret** are the other two, and they plug into the same SAF interface ✅. The design implication for the bridge: platform-side mechanisms that work "through SAF" work across all three ESMs; mechanisms that are RACF-specific must be re-checked per ESM.
2. **Authorisation is routed, not embedded.** Resource managers (JES, CICS, IMS, UNIX System Services, DB2, MQ) call SAF; SAF calls the ESM; the ESM decides. Security is a *shared service of the operating system*, which is precisely why it is resource-level and uniform across subsystems.
3. **It predates and outlives the application.** Application changes do not change the authorisation model; the authorisation model is a property of the platform the application runs on.

### 3.2 Resource Classes and Profiles

RACF protects resources through **classes** and **profiles**. A class is a category of resource (datasets, users, groups, general resources, started tasks, spool, operator commands, network access, and so on); a profile names a specific resource or pattern within the class and carries the access list. Verified class names used in this guide include **DATASET**, **USER**, **GROUP**, **GENERAL**, **FACILITY**, **STARTED**, **JESSPOOL**, **SURROGAT**, **OPERPARM**, **SERVAUTH**, **TSOAUTH** and **IDIDMAP** ✅ (IBM).

`SETROPTS` is the options command that tunes the facility's global behaviour (class activation, auditing options, erasure on delete, security levels and so on); **each option must be verified against IBM's current documentation before it is named in a change** — the class and command names here are as IBM documents them, but a specific option's behaviour is version-specific ⚠.

### 3.3 Dataset and Resource Protection

This is the platform's strongest, most under-appreciated property: **authorisation is applied at the resource, by the operating system, whether or not the accessing program cooperates.**

- **Datasets** are protected by the DATASET class, including generic profiles and protection by **high-level qualifier** — so `PROD.PAYMENTS.**` can be governed as a scope, not dataset by dataset ✅-structural.
- **General resources** (transactions, queues, commands, functions) are protected by the GENERAL or subsystem-specific classes — CICS transactions via `TCICSTRN`/`GCICSTRN`, for instance ✅.
- **Spool datasets** are protectable via the **JESSPOOL** class — IBM documents this explicitly as "Protecting data sets on spools" ✅. *The platform can protect its spool; the common finding is that it is not fully activated or scoped* — an operational fact, not a platform limitation.
- **Network access** can be constrained by the ESM through the **SERVAUTH** class on the IP stack side ✅-structural.

The nuance worth stating plainly: the **STARTED** class and the **trusted** attribute interact with resource protection. The STARTED class "can assign different user IDs and group names to the same started member, depending on the job name" ✅; the started-procedures table is the older alternative; and the **trusted** attribute exists for started procedures and SAPI applications ✅ (IBM, *Protecting data sets on spools*). Trusted status is the mechanism by which a subsystem can act beyond its own authority — and, mishandled, it is how authority leaks (§4).

### 3.4 The Identity Classes — Users, Groups, Started Tasks

The identity model is where the platform's age shows most sharply, and where the bridge has the most work:

- **Userids** are 1-to-8 characters. "Every job, started task, or transaction on z/OS has associated with it an identity — a 1 to 8 character string called the User ID" ✅ (IBM, *RACF Identity Propagation on z/OS — Who Are You?*, SHARE session 8352, 2011).
- **ACEEs** are the runtime identity control blocks: "The Accessor Environment Element (ACEE) is the z/OS control block which represents a user's identity," created by the ESM "on request by resource managers, such as UNIX System Services, JES, CICS, IMS" ✅ (same source). An ACEE can be held at **address-space level** (ASXBACEE) or at **task level** (TCBSENV), and access checks use the **task-level ACEE first, then the address-space ACEE** ✅. This is the mechanism behind identity propagation *inside* a subsystem — and it is why a poorly-configured CICS or batch environment can execute work under the wrong identity without anyone noticing.
- **Groups** bind userids to access lists; **group and profile structures** are the platform's role approximation, and the bridge's job is to keep them coherent with enterprise roles (§5.5).
- **Started tasks** get their identity from the STARTED class or the started-procedures table ✅ — the platform's workload identity, and the thing the enterprise must govern like a service account.

### 3.5 The Audit Records

**SMF — System Management Facilities — is the platform's long-established record source, and RACF's records are among the richest in the estate.** Verified specifics:

- **Record type 80 is RACF processing** — IBM's documentation title is literally "Record type 80: RACF processing record," and RACF writes it when the `ALL` or `SUCCESS` logging option is set in the resource profile (via `ADDSD`/`ALTDSD`/`RALTER`/`RDEFINE`) ✅ (IBM, *Record type 80: RACF processing record*). SMF 80 is therefore per-resource-event, with the accessor, the resource, the access intent and the decision.
- **Identity propagation enriches the record.** RACF identity propagation adds the **IDID** (domain + user id) to SMF type 80 records and to type 83 records (subtype 2 and above), and IBM's unload utilities **IRRADU00** (SMF unload) and **IRRDBU00** (database unload) support them ✅ (IBM).
- **Other record types are individually verifiable.** IBM's SMF layout documentation lists "the predefined SMF record types" ✅, and confirms for example that **record type 14 (X'0E') is INPUT or RDBACK dataset activity**, with type 15 the OUTPUT/UPDAT/INOUT counterpart ✅, that **type 110 is CICS/TS statistics** ✅, that **type 118 is "TCP/IP Statistics (stabilized)" and type 119 is "TCP/IP Statistics"** (both from z/OS Communications Server: IP) ✅, that **type 109 is TCP/IP syslogd messages** ✅, that **type 6 is printer output** (the spool/print record, with the JES2/JES3 output-writer and PSF subtypes) ✅, and that **type 30 is "Common Address Space Work"** — the job and step accounting record ✅ (all per IBM's *MVS System Management Facilities (SMF)* manual as consolidated in Cheryl Watson's *SMF Reference Summary*, which lists 219 record types and states plainly that the platform "provides the most comprehensive metrics about its actions of any platform we are aware of"). The pattern is checkable per type — and this guide does not name a type it has not checked (§15). Notably, SMF 80 is not a RACF-only format: Broadcom publishes a **"SMF Type 80 Record Layout"** for its Top Secret ESM ✅ — evidence that the record families are a shared industry format across ESMs.
- **zERT** (z/OS Encryption Readiness Technology) goes further on the network side, writing **SMF type 119** records — subtype 11 (zERT detail) and subtype 12 (zERT summary) — to report on the cryptographic protection (TLS, IPsec, SSH) of observed network flows, and IBM has enhanced it to recognise failed TLS/SSH handshakes and to carry certificate serial numbers and expiry ✅ (IBM; see §15).

### 3.6 What This Model Does That a Modern Stack Frequently Does Not

It is worth stating the strengths explicitly, because they are the answer to the myth, and because they change the design: the bridge should *lean on* these, not replace them.

| Platform property | Why it beats the common modern equivalent |
|---|---|
| **Dataset-level protection as a platform primitive** | On a modern stack, file authorisation is typically an *application* feature: permissions are enforced by the app, and the storage layer trusts whoever holds the credentials. On z/OS, the operating system enforces dataset authorisation for *every* accessor — batch, CICS, IMS, DB2, a utility, an ad-hoc user — whether or not the program cooperates ✅ |
| **Resource-level granularity across subsystems** | The same authorisation facility governs datasets, CICS transactions, IMS resources, MQ queues, operator commands and spool output ✅. The modern equivalent is a different policy engine per service, with different failure modes |
| **Comprehensive, long-established audit records** | SMF predates the SIEM by decades and records resource-level decisions (type 80) as a platform function ✅, not as an application logging choice |
| **Identity is attached to *all* work, not just interactive sessions** | Batch, started tasks and transactions all carry an identity and an ACEE ✅ — batch is not a blind spot by design, only by governance |
| **MFA and stronger-than-password authentication exist on the platform** | **IBM Z Multi-Factor Authentication** "allows RACF to use alternative authentication mechanisms in place of the standard z/OS password … RSA SecurID-based authentication systems, such as Apple Touch ID devices, certificate authentication options such as PIV/CAC cards, RADIUS, and more," and secures logins to z/OS, z/VM and Linux on Z ✅ (IBM). RACF calls IBM MFA during logon for configured userids. The myth "mainframes are password-only" is false — *deployment*, not capability, is the gap |
| **One-time credentials for application sign-on** | The **RACF PassTicket** is "a one-time-only password that is generated by a requesting product or function … an alternative to the RACF password that removes the need to send RACF passwords across the network in clear text" ✅ (IBM) |
| **Cryptographic identity mapping** | RACF maps digital certificates to userids one-to-one (issuer DN + subject DN + serial number) or many-to-one (mapping rules), via `RACDCERT`; "The issuer's distinguished name and subject's distinguished name appear in all the RACF SMF records created. This provides end-user accountability" ✅ (IBM) |
| **Multi-image administration** | **RRSF** — the RACF Remote Sharing Facility — "allows RACF to communicate with other MVS systems that use RACF, allowing you to maintain remote RACF databases," historically over SNA/APPC and also over TCP/IP ✅ (IBM) |

**The honest counterweight.** None of this is a substitute for the enterprise identity and audit planes. The platform can do MFA — but if the enterprise's conditional-access policy, device posture and MFA are enforced at the *broker*, and the platform's interactive path bypasses the broker, the platform's MFA capability is not being exercised on that path. The platform has the richest audit records in the estate — but if `IRRADU00` output never leaves the platform, the SIEM never sees them. And the platform can protect spool, datasets and network access — but "can" is not "configured," and the distance between them is where §4's weaknesses live.

---

## 4. Where the Real Weaknesses Are

The weaknesses are not in the platform's security model. They are in **how the platform is operated and how it is connected** — and specifically in five places, each of which survives precisely because it sits in the seam between the mainframe team and the enterprise security function, owned by neither. Threat-modelling discipline for these is the sibling [threat_modeling_guide.md](threat_modeling_guide.md)'s machinery; this section names the weaknesses and the controls that reduce them.

### 4.1 The Five Weaknesses — the Table

| # | Weakness | Mechanism | Why it persists | Reducing control | Limit of that control |
|---|---|---|---|---|---|
| **i** | **Shared and static credentials** | A batch or subsystem identity used by several processes *and* several people; no per-person accountability; rarely rotated | Convenience ("the job just uses `BATCH01`"); a shared identity never breaks when a person leaves; no owner for the identity | Give each workload its own STARTED-class identity; align human access to personal userids; drive provisioning from the enterprise | Cannot make a genuinely shared *service* identity personal; and legacy job streams may embed the userid in JCL (hard to change) |
| **ii** | **The file-transfer path** | A privileged transfer identity, wide directory/dataset scope, and an unencrypted or weakly authenticated protocol; a single account that can move files anywhere | Transfer "just works"; scoping it breaks partners; encryption is often a project, not a default | Least-privilege transfer identity; dataset-HLQ and UNIX-path scoping; TLS/SFTP; AT-TLS for apps that cannot be changed | The counterparty may require a protocol you cannot harden; the scheduler identity is often the same account |
| **iii** | **The spool and printed output queue** | Job output containing sensitive data sits on the spool, readable by anyone with SDSF/SAPI authority; often not purged | Output is transient, so nobody treats it as a data store; JESSPOOL protection may be unactivated or over-broad | Activate and scope **JESSPOOL**; restrict SDSF/SAPI authority; purge output on schedule | Legacy jobs trust the spool as a shipping mechanism; changing them is a workload change, not a security change |
| **iv** | **The 3270 access path and the emulator session** | Interactive access via 3270 emulation, frequently shared identities, no device posture, session content unobserved | The path is decades old and assumed internal; the tooling rarely supports modern MFA | Identity-aware gateway in front of TN3270; per-user identity via the bridge; session recording | Recording is detective and review-dependent; the platform cannot be the PEP for its own logon |
| **v** | **Third-party and vendor support access** | A privileged external identity, often with broad authority, granted for a support case and never re-approved | Support access is operationally urgent; the vendor "needs it"; re-approval is nobody's calendar item | Vault the vendor identity; time-bound and case-bound access; named sponsor; re-approval cadence | Vendors may require standing access for SLAs; the platform-side tooling for scoped, just-in-time vendor access is limited |

The five are ordered by how often they appear together in a review — but *how often* is a matter of institutional practice and this guide will not put a number on it (§15). What can be said without inventing statistics is the pattern: these are the five places where a mainframe estate's control converges on *shared authority*, and shared authority is the one thing zero trust cannot tolerate.

### 4.2 (i) Shared and Static Credentials

**Mechanism.** A single userid — commonly something like a batch or subsystem name — is used by several distinct processes, and often known to several people. It may be configured as a **started task** identity (§3.4) or, worse, be a plain userid with a static password embedded in JCL or a configuration file. Every action taken under it is attributable to *the identity*, not to a person, and the audit record proves only which shared principal acted (§3.5). Rotation is avoided because rotation breaks the jobs that embed the credential.

**Why it persists.** A shared identity is operationally frictionless: the job always runs, the operator always has access, and nobody has to maintain a mapping. The failure mode is silent — nothing breaks until an incident demands attribution and there is none.

**Reducing control.** Enumerate every shared identity; give each *workload* its own **STARTED**-class identity with the minimum authority (the STARTED class supports per-member userid/group assignment ✅); give each *person* a personal userid; and move authentication to the bridge (§5) so that human access through the shared id is eliminated first. Where the credential is a static password, replace it with **PassTicket** (one-time, no clear-text password on the wire ✅) or certificate authentication.

**The limit.** A genuinely shared service identity — a started task that several processes legitimately share — cannot be made personal; the remedy there is scoping and monitoring, not personalisation. And legacy job streams that embed userids cannot always be re-parameterised without a workload change, which is out of scope for a security project (§11.1).

### 4.3 (ii) The File-Transfer Path

**Mechanism.** File transfer on the platform is performed by an identity that is almost always **more privileged than it needs to be**, with a scope that covers a wide range of datasets or UNIX directories. FTP, FTPS, z/OS OpenSSH (SFTP/SCP), Connect:Direct and Transfer CFT are all serviced by such accounts. If the path is unencrypted or weakly authenticated, the credential is exposed in transit; if the scope is wide, one compromised account reaches everything the transfer job can reach.

**Why it persists.** Transfers are the estate's integration surface — they interface with partners, other systems and other platforms — and tightening scope has an immediate external consequence. The section that owns the transfer mechanics is the sibling [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) (and [linux_file_sharing_notification.md](linux_file_sharing_notification.md)); the product mechanics are [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [control_m_guide.md](control_m_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) — this guide owns only the security consequences (§8).

**Reducing control.** Replace generic transfer ids with least-privilege, per-flow accounts; scope by dataset high-level qualifier and UNIX path; require TLS (FTPS) or SSH-based transfer (SFTP/SCP) on every path; and where the application cannot be re-written, use **AT-TLS** to add TLS transparently under policy ✅ (IBM, §8.3).

**The limit.** Some counterparties require protocols you cannot harden, and the transfer identity is frequently *the same identity the scheduler uses* (§8.4) — so scoping the transfer account can break the batch chain. That is a workload-change conversation, not a security setting.

### 4.4 (iii) The Spool and Printed Output Queue

**Mechanism.** Job output — reports, extracts, dumps, error messages containing data — lands on the **spool** (§2.3). Anyone with SDSF or SAPI authority can search and browse it. If **JESSPOOL** protection is not activated or is scoped too broadly, the spool is an information-disclosure path that bypasses every dataset control the estate has.

**Why it persists.** Output looks transient. Nobody classifies a print queue as a data store, and the jobs that write to it often predate the data-classification programme.

**Reducing control.** Activate the **JESSPOOL** class and scope it to the jobs and users that need it ✅ (IBM documents spool protection explicitly); restrict SDSF/SAPI authority; and purge or archive output on a defined schedule.

**The limit.** Many legacy jobs use the spool as an intended *shipping* mechanism (a report routed onward from spool output), so removing or restricting it is a workload change (§11.1) — the honest answer is protection plus purging, not elimination.

### 4.5 (iv) The 3270 Access Path and the Emulator Session

**Mechanism.** Interactive access arrives via **3270 emulation** — desktop clients or browser emulators — into TSO/E, ISPF or CICS. The emulator session is the path on which shared identities are most often used, device posture is typically absent, and session content is typically unobserved. The platform authenticates the userid at logon and enforces authorisation thereafter — but it is not enforcing the *enterprise's* access policy, and it cannot evaluate the device.

**Why it persists.** The path is decades old, is assumed to be "inside", and modern identity tooling historically did not speak TN3270. The emulator is often outside the SSO estate entirely.

**Reducing control.** Put an **identity-aware gateway** in front of the 3270 path (§7.2) so the enterprise-side half enforces identity, MFA and policy; carry a **per-user identity through the bridge**; and record sessions as a compensating detective control (§7.3).

**The limit.** The platform **cannot be the PEP for its own logon path**, and session recording proves less than people assume (§7.4). The gateway is the answer; the platform's role is to consume the identity the gateway delivers.

### 4.6 (v) Third-Party and Vendor Support Access

**Mechanism.** Vendor support access is a *privileged external identity* — sometimes with broad authority, sometimes standing, sometimes shared within the vendor — granted to resolve a problem and rarely re-approved afterward. It is frequently the **least governed** identity in the estate, because it sits outside the enterprise's joiner-mover-leaver process (the vendor's staff are not the enterprise's joiners) and outside the platform team's usual change control.

**Why it persists.** Support access is granted under time pressure to unblock an outage, and the "temporary" access outlives the case. Nobody owns the vendor identity's lifecycle.

**Reducing control.** Vault the vendor identity; make access **case-bound and time-bound**; require a named internal sponsor; re-approve on a fixed cadence; and route it through the same session brokering and recording as internal privileged access (§6).

**The limit.** Vendors may contractually require standing access for SLA response, and the platform-side tooling for scoped, just-in-time external access is limited — which is exactly why this is the first candidate for written residual-risk acceptance (§11) when the control cannot be fully applied.

---

## 5. The Identity Bridge

**This is the central architectural section of the guide.** Everything else is either a control the platform already has (§3), a weakness the bridge reduces (§4), or a compensating measure where the bridge cannot reach (§7, §11). The bridge is the thing that converts "a mature but isolated security model" into "a mature security model that participates in the enterprise identity plane."

### 5.1 The Two Planes

Zero trust is, at bottom, two planes working together: an **identity plane** (who is this subject, what is its posture, what is it allowed to do, and can we revoke that quickly?) and an **audit plane** (what happened, attributable to whom, and can we see it in one place?). Modern stacks delegate the first to an identity provider and the second to a SIEM. The mainframe predates both, runs its own versions of both, and does not join either by default.

| Plane | Enterprise side | Platform side | The gap the bridge closes |
|---|---|---|---|
| **Identity** | Okta, Microsoft Entra ID, the ZTNA broker; provisioning, SSO, MFA, conditional access | RACF/ACF2/Top Secret userids, groups, profiles, STARTED-class workload ids ✅ | No automated provisioning/deprovisioning; no per-person identity through the interactive path; no SSO federated into 3270 |
| **Audit** | SIEM/security analytics; the SOC under [secops_guide.md](secops_guide.md) | SMF records, RACF type 80, zERT type 119 ✅ | Records exist on-platform but are not collected, normalised or correlated enterprise-side |

The bridge is two directional flows: **identity in** (enterprise principal → platform userid → platform authorisation) and **audit out** (platform record → enterprise SIEM). §5 covers identity-in; §10 covers audit-out. Getting identity-in right is what makes every downstream control meaningful; getting audit-out right is what makes the whole thing provable.

### 5.2 The Platform-Side Mechanisms

The platform side of the bridge is not hypothetical — it is documented, shipped functionality, most of it from the identity-propagation capability IBM introduced in **z/OS V1.11** ✅. The mechanisms below are the platform's own, established from IBM's *RACF Identity Propagation on z/OS — Who Are You?* (SHARE session 8352, 2011) ✅ unless otherwise noted.

1. **RACF identity propagation.** Introduced with z/OS V1.11, it allows "the mapping of arbitrary user identities and realms/domains into RACF z/OS user IDs" and "the recording of the distributed user's identity in RACF log records" ✅. This is the foundational mechanism: an identity that arrives from *outside* the platform — a distributed user, an application, a realm — is mapped to a z/OS userid, and both identities appear in the audit record.
2. **IDID and ICRX.** The propagation model introduces the **IDID** — the Distributed Identity Data control block (distinguished name, UTF-8, max 246 bytes; registry name, UTF-8, max 255 bytes) — and the **ICRX**, the Identity Context Reference Extended, which carries it ✅. These are the data structures that let a subsystem receive and act on a foreign identity.
3. **RACMAP and the IDIDMAP class.** The **RACMAP** command manages the mapping rules, and the **IDIDMAP** general-resource class holds them ✅. Rules support **one-to-one** and **many-to-one** mappings; authorisation to manage them is via **SPECIAL** authority or the **`IRR.IDIDMAP.function`** resource in the **FACILITY** class ✅. This is the platform-side provisioning surface: it is where an enterprise lifecycle connector writes the identity mapping.
4. **ENF event code 71.** Profile changes that affect identity — `CONNECT`, `REMOVE`, `ALTUSER REVOKE` — signal **ENF event code 71**, so subsystems can flush cached authorisations ✅. This matters operationally: it is how a deprovisioning action on the platform propagates quickly into running subsystems rather than waiting for a recycle.
5. **Digital-certificate mapping.** RACF maps certificates to userids **one-to-one** (the certificate is loaded to the RACF database and matched by issuer DN + subject DN + serial number) or **many-to-one** (mapping rules by issuer/subject DN), managed via **`RACDCERT`** ✅. "The issuer's distinguished name and subject's distinguished name appear in all the RACF SMF records created. This provides end-user accountability" ✅.
6. **Kerberos.** IBM names Kerberos principals and realms as another identity-mapping mechanism for bringing enterprise identities onto the platform ✅ (the AD/Kerberos route into z/OS; cross-check IBM Network Authentication Service for z/OS before asserting detail ⚠).
7. **DB2 trusted context and roles.** DB2 for z/OS can assign authorisation based on connection characteristics — source user ID, source of connection (IP address, domain name, SERVAUTH), and encryption level of the connection ✅. This is a *data-plane* bridge: the database decides privilege from *how* the connection arrived, not only *who* claims to have made it.
8. **PassTicket.** For application-to-application sign-on, the one-time PassTicket removes the need to send a RACF password in clear text ✅ (IBM) — the mechanism that lets a brokered session authenticate to the platform without handling the user's password.
9. **RRSF.** The RACF Remote Sharing Facility lets RACF images share and administer remote RACF databases, historically over SNA/APPC and also over TCP/IP ✅ — relevant because a multi-image estate needs one coherent identity administration surface, and RRSF is the platform's own answer ⚠ (scope per deployment).

**The design consequence.** The platform *has* the receiving end of the bridge. What it does not have is the enterprise side pulling on it: nobody is writing to IDIDMAP on a joiner-mover-leaver event, nobody is calling RACMAP from the identity provider's lifecycle engine, nobody is using PassTicket to remove passwords from the application path. **The bridge is unbuilt because the two teams never met, not because the platform cannot do it.**

### 5.3 The Enterprise-Side Half

The enterprise-side half is the identity provider and the connector/proxy. Concretely, it is **Okta** or **Microsoft Entra ID** as the identity source, plus whatever lifecycle-provisioning connector writes platform identities and IDIDMAP rules, plus a gateway (§7.2) for the interactive path, and, where an API surface exists, **z/OS Connect** exposing CICS/IMS as REST/JSON services behind the enterprise API gateway ⚠ (the product's exact capabilities and versions must be verified against IBM before a design asserts detail; see §15). Vendors in the privileged-access and session-brokering space — **CyberArk**, **Broadcom Privileged Access Manager**, **Wallix**, **Delinea** — name z/OS support among their mainframe capabilities ⚠-structural: these products exist and are used in the industry for mainframe privileged access and session management; this guide does not assert any institution's deployment.

The enterprise side owns four things the platform side cannot:

1. **The authoritative identity store** — who exists, what their role is, when they join and leave.
2. **The lifecycle trigger** — the event (hire, role change, termination) that must drive platform provisioning and deprovisioning.
3. **Authentication policy** — MFA, device posture, conditional access (and, for the interactive path, the gateway that enforces it *before* the session reaches 3270).
4. **The correlation key** — the stable identifier (a certificate subject/DN, a Kerberos principal, or an enterprise subject id) that the platform-side mapping rules key on. **Choosing the correlation key is the single most important design decision in the bridge**, because it determines what a deprovisioning event can actually revoke.

### 5.4 Provisioning and Deprovisioning — the Joiner-Mover-Leaver Problem

**Platform accounts on most estates are created by hand.** A ticket is raised, a specialist runs the administrative commands to create a userid, connect it to groups, and define its profiles — and the reverse, when someone leaves, is often *not* raised at all, because the team that knows the person left is not the team that holds the platform userid. This is the joiner-mover-leaver problem in its purest form: the enterprise's JML process ends at the identity provider and does not extend onto the platform.

The bridge's provisioning flow, therefore, has to be built in both directions:

| Event | Enterprise action | Platform action (platform-side mechanism) |
|---|---|---|
| **Joiner** | Identity created, role assigned in Okta/Entra ID | Create/align the platform userid; write IDIDMAP mapping rules (one-to-one or many-to-one) via RACMAP ✅; connect to platform groups reflecting the role |
| **Mover** | Role changes | Re-evaluate platform group connections and profile access; re-write mapping rules; lose the old role's authority |
| **Leaver** | Identity disabled/deleted | Revoke platform userid and group connections; remove IDIDMAP rules and certificate mappings; rely on ENF event 71 (`ALTUSER REVOKE` etc.) to flush subsystem caches ✅ |
| **Vendor/support** | Case opened with a sponsor | Time-bound, case-bound platform identity or brokered session (§6); re-approve on cadence (§4.6) |

**The deprovisioning direction is the one that matters most and is built least.** A joiner who waits a day is an inconvenience; a leaver whose platform userid survives for months is a standing finding. The design rule: **deprovisioning must be event-driven and must cover every identity class on the platform** — personal userids, group connections, certificate mappings, IDIDMAP rules, and the shared/service ids that person was the custodian of.

### 5.5 The Group and Profile Model on the Platform Side

The platform's authorisation is **group- and profile-based**, not role-based in the modern sense (§3.2–3.4). The bridge has to translate between the two, and the decisions are consequential:

- **Groups are the platform's role approximation.** Mapping enterprise roles to platform groups (not to individual profiles) keeps the authorisation model administrable and keeps the bridge's surface small.
- **Profiles carry the actual access lists** — dataset profiles, CICS transaction profiles, general-resource profiles. A bridge that only creates userids and connects groups, without a policy for who owns profiles, produces orphaned and over-broad authorisation.
- **Generic profiles and high-level qualifiers** are the platform's scoping primitive (e.g. `PROD.PAYMENTS.**`), so the bridge's role-to-scope mapping should be expressed at the prefix level, then refined, rather than dataset-by-dataset.
- **STARTED-class workload identities** are the *non-human* half of the bridge and must be governed like service accounts: named owner, defined purpose, review cadence — not left to the batch team's memory (§4.2).

### 5.6 Why the Bridge Is the First Thing to Build

**The bridge is first because every other control depends on it — and because it is the thing most often skipped in favour of a network control that changes nothing about identity.**

Consider what a network control in front of a shared batch identity actually accomplishes: traffic is filtered, the path may be encrypted, the segment may be tightened — and the same shared identity still performs the work, still cannot be attributed to a person, and still cannot be revoked without breaking the process. **Zero trust is an identity property before it is a network property.** A segmentation rule around a shared identity is a well-netted shared identity; it is not zero trust. (The network answer still matters — §9 — but it is the *second* control, not the first.)

The bridge is also first because it is the **prerequisite for everything downstream**: vaulting and brokering need a per-person identity to broker (§6); session recording is only worth reviewing if sessions are attributable (§7.4); the transfer path can only be scoped if flows have identities (§8.2); and the audit feed is only valuable if records carry an identity worth correlating (§10.4). Build the bridge, and all of the following controls compound. Build the network control first, and the estate has spent its budget and changed nothing about *who can do what*.

---

## 6. Privileged Access, Vaulting and the Named Identity

### 6.1 Vaulting and Brokering

Privileged access on the platform follows the same discipline as privileged access anywhere else — with the platform's own constraints on what can be vaulted and what can be brokered. The enterprise pattern is: **no human knows the privileged credential**; the vault holds it, checks out or injects it, and records the session. On the platform, that pattern runs into three realities:

1. **The platform has its own privileged identities** — the security administrator (RACF `SPECIAL`/`OPERATIONS`/`AUDITOR` authorities), the systems programmer, the started-task owner, the operator. These are the identities a vault must control, and they number far more than a modern estate expects because of the platform's admin structure.
2. **Credential injection is not ubiquitous.** Some access paths accept a **PassTicket** or a certificate (✅ §3.6, §5.2) and can therefore be brokered without the credential ever reaching the human; older interactive tooling may not support credential injection at all, so the vault's value is checkout-and-record rather than injection.
3. **The vault is only as good as the brokered path.** If a privileged user can also reach the platform by a route the vault does not control (a direct 3270 route, a local FTP path, a console), the vault is theatre.

The design rule: **vault the platform privileged identities, broker the sessions, and close the bypass routes** — then the vault plus the bridge gives the estate per-person attribution for privileged work.

### 6.2 The Firecall (Emergency) Identity

Every platform estate needs a break-glass path: an identity that works when the normal access path has failed — when the identity provider is unreachable, the vault is down, or the console is the only way in. **The firecall (emergency) identity is legitimate and necessary.** The failure is not having one; it is having one that is ungoverned.

The controls that make a firecall identity defensible:

- **It exists, it is documented, and its break-glass use is approved in advance** — not improvised during an incident.
- **It is vaulted like any other privileged credential**, with the emergency release itself logged.
- **It is monitored**: any use triggers immediate review, because a firecall identity is by definition an exception.
- **It is re-secured after use** — the credential that came out of the vault goes back and is rotated where the platform supports it.
- **Its use is *not* a substitute for the bridge.** Firecall access that becomes routine access is a governance failure wearing an emergency badge.

**The platform nuance.** Platform-side, the strongest control available for the firecall path is the one the platform already provides: an action can be attributed to a **named identity**, and the audit record (SMF type 80 ✅) will show it. So even an emergency identity should be *named* — a specific firecall userid with a specific accountable owner — not a generic shared "EMERGENCY" account. §11's residual-risk discipline applies here directly: if the firecall identity cannot be fully brokered, the remaining exposure is documented, owned and reviewed.

### 6.3 Session Brokering to the Platform

Session brokering is the enterprise-side mechanism that puts a **privileged-access broker** between the human and the platform, presenting the platform session through a controlled path that enforces identity, records the session, and can be revoked centrally. For the mainframe, this is the same architectural move as the 3270 gateway in §7.2 — a broker or gateway that terminates the user's session on the enterprise side and re-originates a platform session under the right identity.

The platform's role in brokering is to **accept the identity the broker delivers** — via certificate mapping, IDIDMAP mapping rules, or PassTicket ✅ — so that the session, once it reaches the platform, is attributable end-to-end. When that works, a brokered privileged session produces: enterprise-side authentication and authorisation, enterprise-side session recording, and a platform-side SMF record under the named identity. Three independent records of the same act, which is exactly what an audit needs.

The honest limit: not every platform access path can be brokered today. Console access, some systems-management tooling and some vendor paths remain outside the broker — and those are precisely the paths that appear in §11's residual-risk register.

### 6.4 The Named Identity — the Most Wasted Capability

**The strongest control the platform already offers is that an action can be attributed to a named identity.** This is not a modern import; it is a property the platform has had for decades, built into the security model (§3.4–3.5): every job, started task and transaction carries an identity, and the audit records name it. An estate that runs per-person identities and reads its SMF records can attribute platform actions to individuals without deploying anything new.

**And yet the common practitioner pattern is the opposite.** Across the industry, mainframe estates frequently run **shared identities for exactly the work that matters most** — the batch that moves money, the transfer that ships files, the subsystem that processes transactions — and reserve personal identities for the least consequential interactive work. *Practice varies widely by institution and by era of the estate*, and this guide will not quantify it; what it can state without inventing a figure is the observed pattern and its cost. The cost is that the platform's single best attribution capability is spent precisely where attribution matters least, and left unused where it matters most.

The corrective is not new technology — it is the bridge (§5) plus the decision to make the shared work *named*. A shared batch identity under a named owner, with a defined purpose and a review cadence, is already materially better than an anonymous one; a per-workload STARTED-class identity is better still; and a per-person identity through the bridge is the goal. The point of §6.4 is that **this is the platform's most wasted capability and the cheapest to reclaim** — because the capability is already there and only the configuration is missing.

---

## 7. The 3270 Access Path

### 7.1 The Emulator and the Interactive Session

Interactive access to z/OS reaches the platform through **3270 terminal emulation** — a desktop emulator or a browser-based emulator — delivering a 3270 data stream to **TN3270** services attached to TSO/E, ISPF, CICS or another subsystem. The session is the platform's classic interactive path, and it has three security properties worth stating plainly:

- **The platform authenticates the userid at logon and authorises every subsequent action** through the ESM ✅ (§3). What it does *not* do is enforce the enterprise's authentication policy — device posture, conditional access, MFA *as the enterprise defines it* (the platform has IBM MFA ✅, but deploying it is a platform-side configuration, and it is not the same thing as the enterprise's conditional-access policy).
- **The emulator client is outside the enterprise SSO estate on many estates.** Browser-emulator access may be federated, but desktop emulators frequently authenticate directly to the platform — which is how the enterprise's identity plane stops at the client boundary.
- **The session content is protocol data**, not HTML or JSON, so generic enterprise observation tooling will not decode it. That is *the* reason the 3270 path is the least observed interactive path in the estate.

### 7.2 The Identity-Aware Gateway

The architectural answer to the 3270 path is the **identity-aware gateway**: a broker/proxy that sits in front of the TN3270 service on the **enterprise side**, authenticates the user against the enterprise identity plane (with MFA and posture as the enterprise defines them), and then re-originates the platform session under the correct platform identity — via certificate mapping, IDIDMAP rules or PassTicket ✅ (the mechanisms of §5.2).

The division of labour is the point, and this guide states it carefully because getting it wrong is the most common architecture error in this space:

| Layer | What enforces it | What it can and cannot do |
|---|---|---|
| **Enterprise side (the gateway)** | The identity-aware broker, using **Okta / Entra ID** and the enterprise's access policy | **Can** enforce the enterprise's identity, MFA, device posture, conditional access and session recording. **Cannot** decide platform resource authorisation — it is not the platform |
| **Platform side (the ESM)** | RACF/ACF2/Top Secret through SAF ✅ | **Can** enforce resource-level authorisation (datasets, transactions, commands, spool) against the identity the gateway delivers. **Cannot** be the PEP for the logon path, evaluate the device, or participate in enterprise SSO on its own |

**The platform cannot be the policy-enforcement point for its own logon path.** It authenticates the userid it is given; it does not challenge the human with the enterprise's policy. The gateway is the PEP; the platform is the resource server. An architecture that tries to make the mainframe the PEP has the roles inverted.

The gateway must also carry a **per-user identity through the bridge** — not a shared emulator service account. A gateway that authenticates the user enterprise-side and then logs onto the platform with a shared id has moved the SSO boundary without moving the identity, and has changed nothing about attribution.

### 7.3 Session Recording as a Compensating Detective Control

Where a session cannot be fully brokered or policy-controlled, **session recording** is the standard compensating control — and on the 3270 path it is genuinely valuable, because the path is otherwise unobserved. The enterprise-side market offers mainframe session recording and monitoring (within the privileged-access and session-management products named in §5.3 ⚠-structural), and recording is usually implemented at the **gateway or broker**, not on the platform.

Recording is a **detective** control, and it should be labelled as such: it does not prevent an action, it makes the action reviewable after the fact. A recording that nobody reviews is not a control; it is storage. The design requirements are therefore not just "record" but: **record attributable sessions, retain them for a defined period, and put them in front of a reviewer with an expectation of review** — with the same review cadence discipline as any other log-based detection, and the SOC consumption pattern of [secops_guide.md](secops_guide.md).

### 7.4 What Recording Does and Does Not Prove

This is the honest statement the guide owes the reader, because session recording is routinely over-claimed in vendor material and in assurance decks:

**What recording proves:**

- **That a session occurred**, on the recorded path, at a recorded time, under the recorded identity (where the gateway is delivering a per-user identity).
- **What was displayed and what was typed**, to the extent the recording captured it — keystrokes and screen output, screen-by-screen.
- **Which commands were entered and which screens were reached** in the recorded stream.

**What recording does not prove:**

- **Intent.** A recording shows what a user did, not why, and not whether the action was authorised or benign. It is evidence, not judgement.
- **What happened in batch.** A session recording captures the interactive session only. Work executed via **batch jobs, started tasks, or a file transfer** does not appear in the recording at all — and on a mainframe estate, batch and transfer are exactly where the volume and the sensitivity live.
- **What happened outside the recorded path.** Any access that does not traverse the gateway — console, a direct emulator route, a transfer path — is invisible to the recording.
- **Completeness of the record.** Fields typed but masked (passwords), data streamed to disk outside screen output, and processing that occurred without a screen update may not be captured. The recording is as good as its capture fidelity.
- **That the recording will be reviewed.** Nothing in the mechanism guarantees review.

The implication is direct: **recording is a compensating control for the interactive path, not a substitute for identity, and not a control over batch or transfer.** The identity bridge (§5) and the audit feed (§10) are what make batch and transfer provable; recording covers the gap the gateway cannot close, and it should be scoped and defended as exactly that.

---

## 8. The File-Transfer and Spool Exposure

The transfer path is, on most estates, the **weakest link in the chain** — a privileged identity, wide scope, sometimes a weak protocol, and a direct dependency on the batch schedule. The section that owns the *mechanics* of transfers is the sibling [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) (and [linux_file_sharing_notification.md](linux_file_sharing_notification.md)); the products are [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [control_m_guide.md](control_m_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md). This section owns the **security consequences** and states none of the mechanics twice.

### 8.1 The Transfer Path End to End

A platform file transfer has five security-relevant components, and a review should name all five:

1. **The protocol** — FTP/FTPS, z/OS OpenSSH (SFTP/SCP), Connect:Direct, Transfer CFT, or a product-specific channel. The protocol determines whether the path is encrypted and how authentication works.
2. **The transfer identity** — the platform account the transfer authenticates as. This is the component most often over-privileged (§4.3).
3. **The scope** — the datasets (by high-level qualifier) and the UNIX/HFS paths the transfer identity can reach (§8.2).
4. **The encryption** — whether the data and the credential are protected in transit, and by what (TLS, SSH, AT-TLS) (§8.3).
5. **The scheduler** — the workload-automation identity that starts the transfer, which is frequently more privileged than the transfer account itself (§8.4).

The end-to-end finding is usually a combination: a **privileged transfer identity**, with **wide scope**, invoked by a **scheduler running under a still-more-privileged account**, on a path whose encryption was added late or not at all. Each component looks acceptable in isolation; the composition is the exposure.

### 8.2 The Transfer Identity and Directory Scoping

**Identity.** A transfer account should be scoped to the flows it serves and no more. In practice, transfer accounts are frequently **generic and shared** — one id for all transfers, or one per partner connection but with platform-wide authority — which means one compromised credential reaches everything the transfer service can touch. The corrective follows §4.2–4.3: per-flow identities, least privilege, vaulted where interactive use exists, and named ownership.

**Scoping is two-dimensional on this platform**, and this is a finding in its own right:

- **The dataset-name space** — scope by **high-level qualifier** and generic profile (e.g. `PROD.PAYMENTS.**` for payment flows, `PROD.REGULATORY.**` for reporting), so that a transfer identity cannot read `HR.**` because it happens to be able to read `PROD.**` ✅-structural. The dataset model (§2.4) makes this natural; it is only unnatural if nobody scoped it.
- **The z/OS UNIX (HFS/zFS) side** — scope the transfer identity's **UID/GID** and directory permissions on the UNIX file system, which is a **separate scope** from the dataset-name space and does not inherit its protections. A transfer path that lands files on the UNIX side is governed by UNIX permissions and the ESM's UNIX checks, not by the dataset profiles — and the two must be scoped *together* or the tighter of the two is defeated by the looser.

### 8.3 Encryption and Authentication on the Path

Three platform-side mechanisms cover the encryption question, and they should be described accurately because vendor material often blurs them:

- **Protocol-native TLS** — FTPS (FTP over TLS) for a legacy FTP path; **z/OS OpenSSH** (SFTP/SCP) for an SSH-based path. z/OS OpenSSH ships with z/OS from V2R2 ✅ (previously as IBM Ported Tools for z/OS: OpenSSH); TLS-secured FTP sessions are supported and verified, and IBM's FTP server can verify the user's access to the profile "whether or not that session is secured" ✅ (IBM).
- **AT-TLS — Application Transparent Transport Layer Security** — "a capability of z/OS Communications Server that can create a secure session on behalf of z/OS applications … provides encryption and decryption of data based on policy statements that are coded in the Policy Agent" ✅ (IBM). This is the mechanism that adds TLS to applications that **were never written for it**, by policy, in the **Policy Agent (PAGENT)** — directly relevant to the transfer and 3270 paths (§4.3, §7.2).
- **Protocol choice itself** — moving a flow from FTP to SFTP/SCP, or from a custom protocol to a TLS-protected one, is a workload change and therefore a candidate for phased treatment (§11, §12).

**Encrypting the path is not the same as identifying the user.** AT-TLS protects the data in transit; it does not give the transfer per-person identity. That is the bridge's job (§5), and the two controls are complementary, not alternatives.

**Where to find out which flows are actually encrypted:** the platform can tell you. **zERT** (z/OS Encryption Readiness Technology) reports the cryptographic protection of observed network flows and writes **SMF type 119** records (subtype 11 detail, subtype 12 summary), covering providers including System SSL, zERTJSSE, z/OS OpenSSH and z/OS IPsec, with enhancements for failed TLS/SSH handshakes and certificate serial/expiry in subtype 12 ✅ (IBM). **zERT is the platform's own answer to "which of my flows are encrypted?"** — and it is a far better answer than inference from a network tap, because it sees the platform's side of every connection.

### 8.4 The Batch-Scheduling Dependency

The transfer path is nearly always driven by **batch** — a scheduler (the workload-automation products cross-referenced above) submits the transfer, and the scheduler runs under a **privileged scheduling identity**. This produces the compounding exposure of §8.1: the scheduler can submit jobs that use the transfer identity, so the scheduler's authority is the *ceiling* of what the transfer can do, regardless of how tightly the transfer account is scoped. Scoping the transfer identity without scoping the scheduler identity reduces nothing.

The controls: scope the scheduler identity to the workloads it must run (typically via the **SURROGAT** class, which lets one userid submit jobs for another — "a surrogate designation … lets one USERID submit jobs for another USERID" ✅); govern the scheduler as a *named, owned, reviewed* privilege; and treat scheduler-to-transfer authority as an explicit, documented delegation rather than an inherited default. Where the delegation cannot be narrowed without breaking the batch chain, it is a §11 residual-risk candidate.

### 8.5 The Spool and Printed Output as a Disclosure Path

The spool (§2.3) is the platform's most under-classified data store. Job output — reports, extracts, error dumps, and the occasional spill of production data — sits on the spool, readable by whoever holds SDSF or SAPI authority, until it is purged (or, often, until it ages out on a schedule nobody set).

**The rebuttal to the myth:** the spool is **protectable** — the RACF **JESSPOOL** class protects datasets on spools ✅ (IBM) — and SDSF/SAPI authority is itself authorisable. The platform is not the problem. **The finding is that JESSPOOL protection is frequently not activated, or activated too broadly, and that purge schedules are absent or untuned.**

**The controls:**

- Activate **JESSPOOL** and scope it so that a user sees only the output they are entitled to ✅.
- Restrict **SDSF/SAPI** authority to the users and roles that genuinely need queue-wide visibility.
- Set and enforce **retention and purge** schedules for output that contains sensitive data — and identify that output, because the spool is where data-classification programmes usually stop.
- **Monitor spool reads** where the platform records them, so that an unexpected browse of a sensitive job's output is visible.

**The limit:** legacy jobs may use spool output as an intended *shipping* mechanism (a report routed onward from spool output), so eliminating spool usage is a workload change, not a security setting (§11.1). Protection plus purging is the achievable answer; elimination is the workload project.

---

## 9. The Network-Side Answer

The network is the second control, not the first (§5.6) — but the constrained platform still needs it, and on this platform the network answer has a specific shape: **segmentation around an asset that cannot move, protection of a small number of entry classes, and an observation point that does not depend on the platform's own logging.**

### 9.1 Segmentation Around the Platform

The mainframe cannot be microsegmented the way a modern workload can: it is a large, long-lived asset with many subscribers, and its entry points serve flows that predate the estate's current network design. What *can* be done is **segmentation at the boundary**, which is still valuable because it constrains who can even attempt a connection to the platform's entry classes:

- **Segment by entry class.** Interactive (TN3270), transfer, transaction (CICS/IMS), messaging (MQ), web/API and systems-management traffic should arrive on separate network paths with separate policy, not share a flat "mainframe VLAN" that everyone can reach. This is the platform's version of the pillar approach in [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §4.
- **Restrict source ranges per path.** The transfer path should accept connections only from the systems that legitimately transfer; the 3270 path only from the gateway; the transaction path only from the application tier that calls it. This is standard segmentation, and on a mainframe estate it is frequently the one control that is *missing entirely* because "everything is internal."
- **Segment the platform from itself.** Where multiple LPARs or images serve different trust levels, the inter-image paths deserve the same segmentation discipline — the estate is a fleet of operating systems (§2.1), and lateral movement *between* images is a real path.
- **Use the platform's own network access control.** The ESM can constrain network access through the **SERVAUTH** class on the IP stack side ✅-structural — so that network authorisation is not only a firewall's job but also a platform-side decision tied to identity.

The honest caveat: segmentation around a mainframe is coarse compared to microsegmentation around a container, and its value is bounded by the fact that the flows it protects are also the flows the business requires. It reduces reachability; it does not address identity (§5).

### 9.2 The Entry Points That Must Remain

A segmentation project on this platform quickly discovers that nearly every entry class has a non-negotiable consumer. The list should be stated explicitly so that the residual is *chosen*, not accidental:

| Entry class | Who uses it | Why it must remain | The control that applies |
|---|---|---|---|
| **Interactive (TN3270)** | Operators, developers, business users | The platform's native interactive access | Identity-aware gateway (§7.2), per-user identity, session recording (§7.3) |
| **File transfer** | Batch, partners, other platforms | Integration and reporting | Tightened identity and scope, TLS/SSH, AT-TLS (§8) |
| **Transaction (CICS/IMS)** | Channel applications, middleware, other systems | The business transactions themselves | Network restriction to known callers; identity propagation onto the platform (§5.2); transaction-level security via `TCICSTRN`/`GCICSTRN` ✅ |
| **Messaging (MQ)** | Application tiers, partners | Asynchronous integration | Queue-level authority, channel authentication, network restriction |
| **Web / API** | Modern channels, API consumers | The platform's modern integration surface (e.g. z/OS Connect exposing CICS/IMS as REST/JSON ⚠) | Enterprise API-gateway policy in front; identity mapped onto the platform |
| **Systems management / console** | Systems programmers, operators | The platform cannot be run without it | Vaulting, brokering, session recording; the least brokerable path (§6.3) — a residual-risk candidate (§11) |

The last row is the one that ends up in the residual-risk register most often, because it is both necessary and hard to broker. That is not a failure of the design; it is the design being honest about what it can and cannot reach.

### 9.3 The Observation Point When the Platform's Own Logging Is Not Centralised

The classic problem: the platform's records are rich (§3.5, §10) but not centralised, so the enterprise cannot *see* platform activity in its normal monitoring. There are two complementary answers, and a mature estate uses both:

1. **Centralise the platform's own records** — the feed described in §10. This is the better answer, because it observes the platform from the inside (identity, resource decision, outcome) rather than inferring from packets.
2. **Observe the network path** — where the platform's logging is not in the SIEM, the network is the fallback observation point, and the repo's dedicated guide is [network_packet_capture_guide.md](network_packet_capture_guide.md). Network observation gives what the platform's records cannot: **which flows are actually encrypted** (though zERT answers this platform-side, ✅ §8.3), **what protocols are in use**, and **whether unexpected peers are connecting to platform entry points**. It does not give identity or resource-level decisions — those are the platform's records.

The two are complementary, not competing. The rule: **the network observes reachability and transport; the platform records identity and authorisation.** An estate that has only the first knows that something connected; an estate that has only the second cannot see the connection it did not know to look for.

---

## 10. The Audit-Side Answer

**This is the guide's most useful practical revelation: the richest audit trail in the estate is the one nobody reads.**

### 10.1 The Platform's Audit Records

The platform's record system is **SMF — System Management Facilities** — and its coverage is genuinely deep (§3.5). For security work, the important records are the ones that tie an **identity** to a **resource decision**:

- **SMF type 80 — RACF processing** — written per resource event when logging is enabled on the profile; carries the accessor, the resource, the intent and the decision ✅. This is the platform's authorisation log, and it exists whether or not any application logs anything.
- **SMF type 14/15** — dataset open/close activity for INPUT/RDBACK and OUTPUT/UPDAT/INOUT respectively ✅ — the data-access record.
- **SMF type 118 / 119 (TCP/IP statistics)** and **type 110 (CICS/TS statistics)** ✅ — the network and transaction-subsystem volumes; **type 109** is the separate TCP/IP syslogd-messages record ✅. Note that both FTP and Telnet server records are generated as part of the TCP/IP record family ✅ (IBM's type-118 mapping macro `EZASMF76` produces DSECTs for the Telnet server/client and FTP server/client records) — so the interactive and transfer paths *do* have their own records, if the enterprise collects them.
- **SMF type 119 — zERT** — the cryptographic protection of observed flows, subtypes 11 and 12 ✅ (the "which flows are encrypted" record; §8.3).
- **Identity enrichment** — RACF identity propagation adds the **IDID** (domain + user id) to SMF type 80 and to type 83 subtypes 2 and above, and the unload utilities **IRRADU00** and **IRRDBU00** support them ✅.

The pattern to state for the reader: **the platform has been recording identity-to-resource decisions for decades**, at a granularity most modern systems do not reach. The asymmetry is not record quality; it is record *consumption*.

### 10.2 The Collection Gap

**The gap in most enterprises is not the absence of records; it is the absence of collection.** The platform writes SMF records to its own datasets, which stay on the platform unless somebody ships them to the enterprise SIEM/analytics platform. That is the integration gap (§1.1), and it is operational: the platform can stream its records, but the connection was never built.

The mechanism exists, and this is what makes the gap a *process* failure rather than a platform limitation:

- **IBM Z Common Data Provider (CDP)** — "provides the infrastructure for accessing IT operational data from z/OS systems and streaming it to the analytics platform in a consumable format … can provide a near real-time data feed of z/OS operational data, like System Management Facilities (SMF) data and z/OS log data" ✅ (IBM). CDP is a component of several IBM products/suites and streams to analytics targets including IBM's z/OS data-and-analytics offerings; the exact target set is product- and version-specific ⚠.
- **IBM Security zSecure** — IBM's RACF administration, audit and SMF analysis/correlation product family, which consumes and reports on SMF (including RACF type 80) ✅-structural (product naming should be confirmed against IBM before a procurement references it; see §15).
- **Collection directly from the platform's own utilities** — `IRRADU00` and `IRRDBU00` (§10.1) exist as record unloads; what is missing is the pipeline and, above all, the *ownership* of the pipeline.

The finding, stated as a finding: **the estate has a security feed it is not collecting, and the enterprise cannot answer "who accessed this dataset, and when?" because nobody built the pipe.**

### 10.3 Integrity, Retention and Volume

Collecting the feed raises three legitimate questions, and each has a platform-appropriate answer:

- **Integrity.** SMF records are produced by the platform's own facilities and can be written to protected datasets; the integrity argument is that the record source is the operating system's logging facility, not an application. Where the record stream is transported off-platform, protect it in transit (the same TLS/AT-TLS mechanisms of §8.3) and in the analytics platform. The record's authority depends on the platform's logging not being disabled at the resource — hence the RACF `ALL`/`SUCCESS` logging option on the profile matters ✅ (IBM).
- **Retention.** Retention is a policy decision, not a platform limitation: the platform can retain SMF data, and the analytics platform's retention policy governs the aggregate. The mainframe discipline is to retain the *security-relevant* record families (type 80 in particular) for the enterprise's audit-retention period, and to archive rather than discard.
- **Volume.** SMF volume is real — type 80 at resource-event granularity across a busy estate is a firehose — and it is the usual reason a collection project stalls. The answer is not to collect less blindly but to **tune what is logged at the resource** (log `ALL`/`FAILURES` where it matters, not everywhere ✅-structural), and to filter the feed at the collector. Tuning-logging and building-the-pipe are the two halves of the same job.

The regulator-facing framing (each flagged per the repo's verification convention): **MAS TRM / Notice 645** expects technology-risk management and auditability in Singapore; **EU DORA (Regulation (EU) 2022/2554)**, applicable **17 January 2025**, raises the operational-resilience bar including detection and incident reporting in the EU; **SWIFT CSP** requires the security controls and monitoring of the SWIFT-connected estate. None of these *mandate* SMF collection by name — the correct statement is that they require **auditability and detection** that an uncollected SMF feed cannot satisfy ⚠.

### 10.4 What Becomes Possible Once the Feed Exists

Once SMF is in the analytics platform, the platform stops being opaque and starts answering the questions the enterprise has been asking about everything else:

- **"Who accessed this dataset, and when?"** — run SMF type 80 by resource and accessor.
- **"Which transfers are actually encrypted, and which are not?"** — read zERT SMF type 119 (§8.3) rather than infer.
- **"Which identities did this, and were they supposed to?"** — correlate type 80 with the identity mapping (IDID where propagation is in use, ✅ §3.5).
- **"Did a leaver's identity survive?"** — compare platform access records against the identity plane's leaver events (§5.4) — the deprovisioning check that is otherwise impossible.
- **"Which privileged identities are active, and from where?"** — the standing review that closes the shared-identity finding (§4.2, §6.4).
- **"Did the firecall identity get used, and by whom?"** — the break-glass review (§6.2).

The wrap-around is that the audit-side answer and the identity-side answer reinforce each other: the feed proves the bridge is working, and the bridge makes the feed worth reading. That is the whole "un-integrated" thesis, made concrete: **the platform already produces the evidence; the estate simply never built the pipe.**

---

## 11. What Genuinely Cannot Be Changed — and the Residual Risk That Must Be Owned

This section is deliberately not a defeat. It is the honest accounting that separates a security programme that is *complete* from one that is *comfortable*.

### 11.1 Immutable Versus Merely Unattempted

The distinction that matters is between **genuinely immutable** exposure and **merely unattempted** control. The first must be owned; the second must be done.

| Category | Examples on this platform | Verdict |
|---|---|---|
| **Genuinely immutable (for now)** | A decade-old batch process whose JCL embeds a userid and cannot be re-parameterised without a business change; a vendor-supplied interface that requires a specific protocol or a standing support identity; a counterparty that will not accept a stronger transfer protocol; a console/system-management path that no broker can mediate | **Accepted, owned, reviewed in writing** (§11.2) |
| **Merely unattempted** | JESSPOOL protection not activated (✅ the class exists); SMF never collected (✅ the feed mechanism exists); a shared batch identity left unnamed when STARTED-class identities are available (✅); a direct emulator path bypassing the gateway; a transfer account never scoped by HLQ | **Not a residual — a task.** Assign it, fund it, do it |
| **Deferred (sequenced, not refused)** | Moving a flow from FTP to SFTP; replacing an embedded credential with PassTicket; onboarding a legacy path to the bridge | **On the phased plan (§12), with a date** |

The discipline: **before any exposure is written into the residual-risk register, it must be classified as genuinely immutable.** "We have not done it" is not a residual risk; it is an open control gap wearing a risk-acceptance costume. The register (§11.3) is for the first row only.

### 11.2 The Ownership Requirement

**A residual risk that is named is a managed risk; the same risk unstated is the finding.**

That sentence is the requirement in one line. For every exposure that survives the controls above, the programme must produce, in writing:

1. **A description of the exposure** — the specific path, identity or workload, and what it can reach.
2. **A named accountable owner** — a *role*, not a team ("the Head of Mainframe Platform Engineering", "the CIO", an accountable committee), personally answerable for the exposure. An exposure owned by "the team" is owned by nobody.
3. **A compensating control** — what reduces the exposure in the meantime (brokering, recording, monitoring, scoping), and an honest statement of that control's limits (§7.4, §4).
4. **A review date** — a cadence, not a one-off. Residual risk changes as the estate changes, and a risk accepted in year one without review is a risk the programme has forgotten.
5. **A written acceptance** — a signed, dated, retrievable record. Acceptance in a meeting is not acceptance; acceptance in a register an auditor can read is.

This converts the honest-admission sections of this guide (§4.3, §4.6, §6.3, §7.4, §9.2) from vulnerabilities into governance artifacts. The platform's exposure is the same either way; what changes is whether the institution knows it owns it.

### 11.3 The Residual-Risk Register

The register is a table that lives with the programme, not with this guide — but its shape belongs here:

| Field | What it holds |
|---|---|
| **ID** | A stable identifier, referenced by the risk register and by audit |
| **Exposure** | The specific path/identity/workload and its reach |
| **Why it is immutable** | The constraint (a workload, a vendor, a counterparty) — the test of §11.1 |
| **Compensating control** | What reduces it, with its honest limit |
| **Accountable owner (role)** | A named role, personally accountable |
| **Review date / cadence** | When it is next examined, and how often |
| **Written acceptance** | Where the signed acceptance lives |
| **Movement** | What would let the exposure be eliminated (the trigger to revisit) |

The **movement** field is the one that keeps a residual register from becoming a graveyard: every accepted exposure should name the condition under which it can be closed — a workload modernisation, a vendor contract change, a protocol upgrade — so that the residual is *tracked toward elimination*, not merely tolerated.

---

## 12. The Phased Application

This guide does not propose a new migration plan. It applies the existing one — the phased plan of [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §9.3 and the CISA Zero Trust Maturity Model (v2, April 2023) that same guide establishes in §7.1 — to the **constrained platform**. Where the ZTNA guide's plan modernises the whole estate, this section answers: *given a platform that cannot be a PEP, which phases apply, in what order, and why is the identity phase always first?*

### 12.1 The Phases

| Phase | Goal | Platform-side activity | Exit criterion |
|---|---|---|---|
| **0. Assess (inventory)** | Know what exists | Inventory every identity (personal, shared, started task, vendor, firecall); every entry class (§2.5); every transfer flow and its scope (§8); the current logging state (which profiles log `ALL`/`SUCCESS`, why JESSPOOL is off) | A complete, dated identity and entry-point inventory, with the shared-identity list explicit |
| **1. The identity bridge** | Join the enterprise identity plane | Build the bridge (§5): lifecycle connector writing userids, group connections and IDIDMAP rules; certificate/Kerberos/PassTicket paths; event-driven deprovisioning; ENF 71 propagation | Joiners provisioned automatically; leavers revoked automatically and *provably* via the audit feed |
| **2. Privileged access** | Control the privileged identities | Vault platform privileged ids; broker sessions; bound and re-approve vendor support access; govern the firecall identity (§6) | Every privileged platform identity vaulted or brokered; vendor access case-bound; firecall use reviewed |
| **3. The interactive path** | Restore identity on the 3270 path | Identity-aware gateway in front of TN3270 (§7.2); per-user identity carried through; session recording with a review cadence (§7.3) | No shared identity on the interactive path; sessions attributable and reviewed |
| **4. The transfer path** | Close the weakest link | Per-flow transfer identities; HLQ/UNIX scoping; TLS/SFTP or AT-TLS on every path; scheduler authority narrowed via SURROGAT (§8) | Every transfer flow has a scoped identity and an encrypted path, or a §11 residual entry |
| **5. The audit feed** | Make it provable | Stand up SMF collection (IBM Z CDP ✅ or equivalent) into the SIEM; tune resource logging; zERT reports for encryption (§10) | Platform records queryable enterprise-side; the §10.4 questions answerable |

The phases map cleanly onto the ZTNA guide's plan: **Phase 0 = its Phase 0 Assess; Phase 1 = its Phase 1 Identity hardening** (the mainframe leg of it); **Phases 2–3 = the platform's contribution to its Phase 2 remote-access work**, applied to the platform's own access paths; **Phase 4 = its Phase 3 data-plane segmentation discipline**, applied to the transfer surface; **Phase 5 = the visibility/analytics cross-cutting capability** of the CISA model that the ZTNA guide's §7.1 establishes. Nothing here competes with that plan; this is the constrained-case branch of it.

### 12.2 Mapping to the ZTNA Plan and the CISA Maturity Model

The CISA maturity model's load-bearing property, established in the ZTNA guide §7.1, is that **each pillar can progress at its own pace** (Traditional → Initial → Advanced → Optimal, across Identity, Devices, Networks, Applications & Workloads, Data). That property is what makes a mainframe workstream viable: the estate does not have to bring the whole organisation to "Optimal" before the mainframe identity work starts.

| CISA pillar | Where the mainframe sits (typical) | The phase that moves it | Honest ceiling on this platform |
|---|---|---|---|
| **Identity** | Initial → Advanced via the bridge (§5) | Phase 1 | **Optimal is reachable** — the platform has the mechanisms; this is the pillar to invest in |
| **Devices** | The platform cannot evaluate device posture | Phase 3 (at the gateway only) | **Ceiling: the gateway.** Device posture is enterprise-side; the platform never sees it |
| **Networks** | Segmentation at the boundary (§9) | Phase 4 | Coarse — the platform is a fixed, multi-subscriber asset |
| **Applications & Workloads** | Resource-level authorisation is already strong (§3) | Phase 4 (scoping) + Phase 1 (workload identity) | Strong on authorisation, weak on per-workload attribution until Phase 1 |
| **Data** | Dataset-level protection is a platform primitive (§3.6) | Phase 4 (scoping) + Phase 0 (classification) | Strong on enforcement, weak on classification — the spool is the blind spot (§8.5) |

The table is the honest maturity picture: **the mainframe can reach "Optimal" on Identity and is structurally capped below it on Devices** (it cannot see the device), while Networks and Data are strong on enforcement and weak on classification. That is a far more accurate picture than either "the mainframe is insecure" or "the mainframe is fine."

### 12.3 Which Phase Delivers the Most per Unit of Effort

**Phase 1 — the identity bridge — delivers the most risk reduction per unit of effort on this platform, and it is almost always the identity work.**

The reason is structural, not sentimental:

- **It reduces the shared-identity finding, which is the root of the other four.** A transfer path is dangerous because of the identity on it; the 3270 path is dangerous because of the identity on it; vendor access is dangerous because of the identity. Bridge the identity, and the other four shrink.
- **It unlocks every downstream control.** Vaulting needs per-person identities to broker (§6), session recording needs attributable sessions to be worth reviewing (§7.4), and the audit feed needs an identity worth correlating (§10.4).
- **The platform-side mechanisms already exist.** RACF identity propagation, IDIDMAP, certificate mapping, PassTicket and the STARTED class are shipped functionality (§5.2, §3.4) ✅ — the phase is integration work, not invention.
- **It is comparatively cheap** — a lifecycle connector, mapping rules, and configuration, against the cost of re-platforming workloads, replacing protocols, or modernising decades-old batch.

**By contrast, the network-first sequence — the instinct to start with segmentation and a gateway — delivers the least per unit of effort**, because it changes reachability while leaving *who can do what* untouched (§5.6). A gateway in front of a shared identity and a segmentation rule around a shared identity are both well-netted shared identities. The network and gateway phases matter, but they matter *after* the identity that gives them something to enforce.

---

## 13. The Cymbal Bank Worked Example

*The following is a fictional, explicitly illustrative worked example. Cymbal Bank is the repo's only bank persona; every institution, finding and date below is invented for teaching purposes. No real bank, vendor or service provider is asserted to be a client, deployment or user of anything.*

### 13.1 The Scenario

Cymbal Bank runs a z/OS mainframe estate — two LPARs plus a z/VM guest estate — hosting its core deposit and payments batch, a CICS transaction estate, an IMS application, DB2 for z/OS, MQ, and the regulatory-reporting batch. Interactive access is via desktop 3270 emulators. File transfer is by a mix of FTP, SFTP and a managed-file-transfer product. The bank's ZTNA programme (per [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §9) has modernised remote access for the distributed estate, and the security function now turns to the mainframe — with a prevailing internal assumption that it is "the legacy risk we cannot fix."

The review's first job is to rebut that assumption. It documents the platform's actual controls (§3) — dataset-level protection, CICS transaction security via `TCICSTRN`/`GCICSTRN` ✅, SMF type 80 records already being written ✅, JESSPOOL protection *available* ✅ — and reframes the estate as **controlled but un-integrated**: the controls exist; the enterprise just is not connected to them.

### 13.2 The Findings

| # | Finding | Evidence | Category |
|---|---|---|---|
| **1** | **A shared batch identity** — `CYMBATCH` — is used by four production batch streams and known to nine operations staff. Work performed under it cannot be attributed to a person, and the identity's password is embedded in two JCL members. | SMF type 80 records all name `CYMBATCH` ✅; the shared list from Phase 0 | §4.2 shared/static credentials |
| **2** | **A vendor support identity** — granted for a database-vendor escalation eleven months earlier — remains active, is not vaulted, has not been re-approved, and carries authority broader than the support case required. | Identity inventory (Phase 0); no re-approval record exists | §4.6 vendor support access |
| **3** | **Audit records the enterprise was not collecting.** The platform has been writing RACF SMF type 80 and dataset-open records for years; none of it reaches the SIEM. The security function's answer to "who accessed the customer master file last month?" was, before this review, "we would have to ask the mainframe team." | SMF datasets on-platform; no feed configured ✅ (the mechanism exists: IBM Z CDP ✅) | §10.2 collection gap |
| **4** | **Spool output containing data it should not.** A nightly extract job's output sits on the spool in full, in a queue readable by any user with SDSF authority; JESSPOOL protection is not activated. The output includes account identifiers. | SDSF browse; `JESSPOOL` class inactive ✅ (the class exists) | §4.4, §8.5 spool disclosure |
| **5** | **Transfer identity over-scoped.** The principal transfer id can read dataset high-level qualifiers well beyond its flows, and the scheduler identity can submit jobs under still broader authority. One path between the bank and a partner is still clear-text FTP. | HLQ profile review; SURROGAT delegations; protocol inventory | §4.3, §8.2, §8.4 |
| **6** | **Interactive path identity is shared.** Three operations roles share one emulator id on the overnight shift; the emulator estate is outside SSO. | Logon records under the shared id; SSO inventory | §4.5, §7 |

### 13.3 The Design — Bridge First

Cymbal applies §12's phases, and deliberately *does not* start with the network:

1. **Phase 0** produces the identity and entry-point inventory, and the shared-identity list becomes the programme's worklist.
2. **Phase 1** builds the **identity bridge**: a lifecycle connector from the bank's identity provider creates and revokes platform userids with role-mapped group connections, writes IDIDMAP mapping rules via RACMAP for the mapped identities ✅, uses certificate mapping and PassTicket for brokered application access ✅, and drives deprovisioning event-first (relying on ENF event 71 to flush subsystem caches ✅). `CYMBATCH` is retired in favour of per-stream STARTED-class identities with named owners; the shared operations identity is replaced by per-user identities.
3. **Phase 2** vaults the platform privileged ids, brokers privileged sessions, and — crucially — converts the vendor support identity to **case-bound, time-bound access with a named internal sponsor** (finding 2's remedy).
4. **Phase 3** places an **identity-aware gateway** in front of TN3270 (§7.2), carries per-user identity through the bridge, and begins **session recording with a defined review cadence** (§7.3) — labelled honestly as a detective control that covers the interactive path only (§7.4).
5. **Phase 4** moves the clear-text FTP path to SFTP and narrows HLQ scope and SURROGAT delegations, using **AT-TLS** where an application cannot be changed ✅.
6. **Phase 5** stands up the **SMF feed** into the SIEM via IBM Z Common Data Provider ✅ — turning finding 3 from "we would have to ask" into "the SIEM answers it," and enabling the zERT SMF 119 view of which flows are encrypted ✅.

The sequence is the guide's thesis in practice: **the platform's controls were never the problem; the missing pipe was.**

### 13.4 The Accepted Residual Risk

Cymbal's review is honest about what it cannot fix, and it produces **one explicitly accepted residual risk** rather than a quiet gap:

> **Residual Risk RR-07 — vendor-supplied settlement-validation interface.**
> The settlement-validation interface is vendor-supplied and requires the vendor's support tooling to authenticate to the platform with a **standing, non-brokered identity** for SLA response. The vendor's contract and the tooling's architecture do not support case-bound, just-in-time access.
> **Compensating controls:** the identity is vaulted and checked out only for an open support case; the vendor must open a case with a named internal sponsor; all sessions under the identity are recorded and reviewed; and its platform activity is monitored through the SMF feed (Phase 5). **Stated limit:** the identity retains standing authority, and the recording covers the interactive session only — it does not cover batch or transfer actions under that identity (§7.4).
> **Accountable owner:** the Head of Mainframe Platform Engineering (a named role, personally accountable).
> **Review date:** first review **31 March 2027**, then **quarterly** thereafter, against the vendor-contract renewal and the tooling's next release.
> **Movement:** the exposure is re-examined at each contract renewal and each tooling upgrade; if the vendor supports brokered identity, RR-07 is closed.

The point of RR-07 is not that Cymbal failed to fix a vendor path. **The point is that Cymbal named it, owned it, compensated it, dated it, and wrote it down** — which is the difference between a managed residual risk and an unstated finding (§11.2).

The review ends where this guide ends: Cymbal did not find an insecure mainframe. It found a **heavily controlled** mainframe that had never been connected to the enterprise's identity plane or its audit plane. The controls were there; the integration was not. **The mainframe was not insecure; it was un-integrated** — and once the bridge was built, the estate's own security model did the rest.

---

## 14. The Anti-Patterns

Each anti-pattern below is stated as **symptom → cause → guardrail**, so that a review can use it as a checklist.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **"The mainframe is insecure, therefore beyond repair."** | The platform is excluded from the security programme as "legacy"; findings are recorded and forgotten | The myth in §1.1, usually from unfamiliarity with the ESM model | Rebut with §3: dataset-level protection, resource-level authorisation, SMF type 80 — *controlled but un-integrated*, not insecure |
| **2** | **A network control in front of a shared identity, called zero trust.** | A gateway/segment is deployed; the shared batch id remains; attribution is unchanged | Sequencing the network phase before the identity phase (§5.6) | Phase 1 first (§12); measure the control by whether *who can do what* changed |
| **3** | **Sessions recorded, never reviewed.** | Recording is on; storage grows; nobody looks | Recording treated as prevention rather than detection (§7.3) | Assign a review cadence and a reviewer; a recording without review is storage, not a control |
| **4** | **The vendor support identity never re-approved.** | A "temporary" external identity from an old case is still active | No owner for the vendor identity's lifecycle (§4.6) | Case-bound, time-bound access; named sponsor; scheduled re-approval (§6, §13.4) |
| **5** | **The audit feed never collected.** | The platform cannot answer "who accessed this?"; the SIEM has no mainframe events | No pipeline and no owner for the pipeline (§10.2) | Phase 5: stand up SMF collection (IBM Z CDP ✅) and own it |
| **6** | **A compensating control treated as a permanent substitute.** | Recording or a gateway is presented as the fix, and the identity remediation is quietly dropped | Compensating controls are easier than the identity change (§7.3, §11) | Every compensating control carries a limit statement and a movement trigger toward the real fix (§11.3) |
| **7** | **Residual risk accepted silently.** | The risk exists in practice but in no register; nobody has signed anything | Exposure never classified as immutable vs unattempted (§11.1) | §11.2: description, named owner, compensating control, review date, written acceptance |

The thread through all seven is the same: **the failure is almost never a missing platform control — it is a missing owner.** The platform can protect its spool, record its decisions, identify its work and encrypt its flows. What it cannot do is decide that somebody in the enterprise is accountable for connecting it.

---

## 15. The Claims Audit

Verified this pass = checked against the source named, on **1 October 2026**, unless a source date is given. The convention follows the repo: ✅ verified, ⚠ flagged/partial/practice-varies, ❌ could not be verified at source (see §16).

| Claim | Status | Source | Source date | Quality |
|---|---|---|---|---|
| RACF is IBM's External Security Manager, part of the z/OS Security Server | ✅ | IBM z/OS Security Server RACF documentation | Current (z/OS 2.x/3.x docs) | Vendor primary |
| SAF is an *interface*: "conditionally directs control to the Resource Access Control Facility (RACF), if RACF is present, and/or a user-supplied processing routine when receiving a request from a resource manager" | ✅ | ibm.com/docs/en/zos/2.2.0?topic=system-authorization-facility-saf | Current | Vendor primary, quoted verbatim |
| SAF "either processes security authorization requests directly or works with RACF, or other security product, to process them"; resources "such as data sets and MVS commands" | ✅ | ibm.com/docs/en/zos-basic-skills?topic=zos-what-is-saf | Current | Vendor primary, quoted verbatim |
| Broadcom CA ACF2 and CA Top Secret are alternative ESMs plugging into SAF | ✅ | Vendor product documentation (Broadcom) | Current | Vendor, structural |
| RACF protects resources via classes and profiles (DATASET, USER, GROUP, GENERAL, FACILITY, STARTED, JESSPOOL, SURROGAT, OPERPARM, SERVAUTH, TSOAUTH, IDIDMAP); SETROPTS is the options command | ✅ | IBM RACF documentation (multiple pages) | Current | Vendor primary; *individual SETROPTS options are version-specific and must be re-checked* |
| "Every job, started task, or transaction on z/OS has associated with it an identity — a 1 to 8 character string called the User ID" | ✅ | IBM, *RACF Identity Propagation on z/OS — Who Are You?* (Mark Nelson, SHARE session 8352) | 2011 | Vendor presentation, quoted verbatim |
| ACEE is the z/OS control block representing a user's identity; created by the ESM on request by resource managers (USS, JES, CICS, IMS) via SAF APIs (`RACROUTE REQUEST=VERIFY`, `InitACEE`/IRRSIA00) | ✅ | Same IBM SHARE 8352 presentation | 2011 | Vendor presentation |
| RACF identity propagation (z/OS V1.11) maps "arbitrary user identities and realms/domains into RACF z/OS user IDs" and records the distributed identity in RACF log records; introduces IDID, ICRX, RACMAP, the IDIDMAP class; one-to-one and many-to-one rules; authorisation via SPECIAL or `IRR.IDIDMAP.function` in FACILITY; ENF event code 71 on CONNECT/REMOVE/ALTUSER REVOKE | ✅ | Same IBM SHARE 8352 presentation | 2011 | Vendor presentation; *introduced V1.11 — behaviour per release should be re-checked* |
| RACF certificate mapping one-to-one (issuer DN + subject DN + serial) or many-to-one (mapping rules), via RACDCERT; "issuer's distinguished name and subject's distinguished name appear in all the RACF SMF records created. This provides end-user accountability" | ✅ | Same IBM SHARE 8352 presentation | 2011 | Vendor presentation, quoted verbatim |
| DB2 trusted context and roles assign authorisation from connection characteristics (source user ID, source of connection — IP/domain/SERVAUTH, encryption level) | ✅ | Same IBM SHARE 8352 presentation | 2011 | Vendor presentation |
| Kerberos principals/realms named as an identity-mapping mechanism | ✅ | Same IBM SHARE 8352 presentation | 2011 | Vendor presentation; *IBM Network Authentication Service detail not re-verified — see §16* |
| STARTED class "can assign different user IDs and group names to the same started member, depending on the job name"; started-procedures table is the alternative; the trusted attribute exists for started procedures/SAPI applications | ✅ | ibm.com/docs/en/zos/2.2.0?topic=ids-started-class; ibm.com/docs/en/zos/2.1.0?topic=data-protecting-sets-spools | Current | Vendor primary |
| ACEE can be at address-space (ASXBACEE) or task (TCBSENV) level; checks use task-level first, then address-space | ✅ | IBM SHARE 8352 presentation | 2011 | Vendor presentation |
| RACF JESSPOOL class protects data sets on spools | ✅ | ibm.com/docs/en/zos/2.1.0?topic=data-protecting-sets-spools | Current | Vendor primary |
| CICS transaction security uses TCICSTRN (general resource class) and GCICSTRN (resource group class) | ✅ | ibm.com/docs/en/cics-ts/6.x?topic=racf-classes-profiles-resources | Current | Vendor primary, quoted verbatim |
| RACF PassTicket is a one-time-only password generated by a requesting product/function; "removes the need to send RACF passwords across the network in clear text" | ✅ | ibm.com/docs/en/zos/2.1.0?topic=interfaces-racf-secured-signon-passticket | Current | Vendor primary, quoted verbatim |
| IBM Z Multi-Factor Authentication "allows RACF to use alternative authentication mechanisms in place of the standard z/OS password … RSA SecurID … certificate … PIV/CAC … RADIUS"; secures logins to z/OS, z/VM and Linux on Z | ✅ | ibm.com/docs/en/zos/2.4.0?topic=z-multi-factor-authentication; ibm.com/products/ibm-multifactor-authentication-for-zos | Current | Vendor primary |
| SMF record type 80 = RACF processing; written when ALL/SUCCESS logging is set via ADDSD/ALTDSD/RALTER/RDEFINE | ✅ | ibm.com/docs/en/zos/2.2.0?topic=records-record-type-80-racf-processing-record | Current | Vendor primary |
| SMF record type 14 (X'0E') = INPUT/RDBACK dataset activity; type 15 = OUTPUT/UPDAT/INOUT | ✅ | ibm.com/docs/en/zos/3.1.0?topic=sr-record-type-14; BMC SMF field documentation | Current | Vendor primary + vendor secondary |
| SMF type 118 = "TCP/IP Statistics (stabilized)"; type 119 = "TCP/IP Statistics" (incl. zERT subtypes 11/12); type 109 = TCP/IP syslogd messages; type 110 = CICS/TS statistics; type 6 = printer output; type 30 = Common Address Space Work | ✅ | IBM *MVS System Management Facilities (SMF)* manual, as consolidated in Cheryl Watson's *SMF Reference Summary* (219 record types); ibm.com/docs/en/zos/2.1.0?topic=reference-type-118-smf-records | 2021 (summary) / current (IBM) | Vendor primary + highly regarded practitioner secondary |
| Identity propagation adds IDID to SMF type 80 (all except 68/71/79/81/82) and to type 83 subtype 2+; IRRADU00 and IRRDBU00 support them | ✅ | IBM SHARE 8352 presentation | 2011 | Vendor presentation |
| Broadcom publishes an "SMF Type 80 Record Layout" for Top Secret — SMF 80 is a shared industry format across ESMs | ✅ | Broadcom techdocs/ftpdocs | Current | Vendor secondary, structural |
| zERT reports cryptographic protection of TLS/IPsec/SSH flows and writes SMF type 119 subtype 11 (detail)/subtype 12 (summary); providers include System SSL, zERTJSSE, z/OS OpenSSH, z/OS IPsec; enhanced for failed handshakes and certificate serial/expiry | ✅ | ibm.com/docs/en/zos/3.2.0?topic=zert-using-zos-encryption-readiness-technology; IBM zERT overview presentations (public.dhe.ibm.com) | 2023 | Vendor primary + vendor presentation |
| AT-TLS "can create a secure session on behalf of z/OS applications … provides encryption and decryption of data based on policy statements that are coded in the Policy Agent"; Policy Agent (PAGENT) holds the policy | ✅ | ibm.com/docs/en/zos/2.1.0?topic=reference-application-transparent-transport-layer-security-tls | Current | Vendor primary, quoted verbatim |
| z/OS OpenSSH ships with z/OS from V2R2 (previously IBM Ported Tools for z/OS: OpenSSH, 5655-M23); z/OS FTP server can verify profile access whether or not the session is secured | ✅ | IBM z/OS OpenSSH User's Guide; ibm.com/docs/en/zos/2.1.0?topic=server-steps-controlling-user-access-ftp | Current | Vendor primary |
| IBM Z Common Data Provider "provides the infrastructure for accessing IT operational data from z/OS systems and streaming it to the analytics platform in a consumable format … near real-time … SMF data and z/OS log data" | ✅ | ibm.com/docs/en/z-logdata-analytics/5.1.0?topic=overview-z-common-data-provider; ibm.com/docs/en/zcdp/5.1.0 | Current | Vendor primary, quoted verbatim |
| SURROGAT allows one userid to submit jobs for another (surrogate designation) | ✅ | ibm.com/docs/en/zos?topic=racf-allowing-another-user-submit-your-jobs; Broadcom CA 7 documentation | Current | Vendor primary + vendor secondary |
| RRSF "allows RACF to communicate with other MVS systems that use RACF, allowing you to maintain remote RACF databases"; historically SNA/APPC, also TCP/IP | ✅ | ibm.com/docs/en/zos/2.1.0?topic=guide-racf-remote-sharing-facility-rrsf | Current | Vendor primary |
| z/OS Workload Manager (WLM) is the goal-based performance manager, not a transaction monitor | ✅ | IBM z/OS documentation | Current | Vendor, structural |
| IBM Security zSecure is IBM's RACF administration/audit and SMF analysis family | ⚠ | IBM product pages (not re-verified this pass) | — | Product naming to confirm before procurement |
| z/OS Connect exposes CICS/IMS as REST/JSON APIs | ⚠ | IBM product documentation (not re-verified this pass) | — | Verify product detail/version before design asserts it |
| Privileged-access/session-management vendors (CyberArk, Broadcom PAM, Wallix, Delinea) provide mainframe/z/OS privileged access and session capability | ⚠ | Vendor pages, industry practice | — | ⚠-structural: products exist and are used; no deployment asserted |
| "Most mainframe estates still run shared identities for the work that matters most" | ⚠ | **Not sourced** — stated as the common practitioner pattern, with the explicit note that practice varies by institution and no figure is asserted | — | Deliberately unquantified; see §16 |
| DORA (Regulation (EU) 2022/2554) applicable 17 January 2025 | ⚠ | EU regulation — established in sibling guides; not re-verified this pass | — | Cross-referenced, not re-derived |
| MAS TRM / Notice 645; SWIFT CSP expectations | ⚠ | Regulator/scheme material — established in sibling guides; not re-verified this pass | — | Cross-referenced, not re-derived |

**The honesty note.** Every platform component name in this guide — RACF, SAF, the class names, the identity-propagation constructs, SMF record types, JESSPOOL, AT-TLS, zERT, SURROGAT, the CICS classes, PassTicket, IBM MFA, IBM Z CDP — is traceable to IBM (or Broadcom, for the competing ESM) in the table above, with the source and its date. The claims marked ⚠ are exactly the ones where practice varies, where a product's detail was not re-verified this pass, or where a prevalence statement cannot honestly be quantified — and they are marked rather than smoothed over.

---

## 16. What Could Not Be Verified and the Closing Summary

### 16.1 What Could Not Be Verified

The following could not be confirmed against a primary source during this pass, and are recorded honestly rather than asserted:

- **Live web search returned empty results repeatedly** for several queries (for example "IBM zSecure RACF administration reporting", "z/OS Connect EE REST API", "IBM MFA multi-factor authentication z/OS", "mainframe session recording privileged access management z/OS", "RACF JESSPOOL class spool data set protection"). Where an empty result occurred, it is a **tool limitation on this host** — it is not evidence that the material does not exist. Each such fact is either carried from the parent's verified seed set (✅) or flagged (⚠).
- **`web_extract` could not read the IBM zERT/FTP-AT-TLS presentation PDFs at public.dhe.ibm.com** during this pass (the scraping engines failed on the URLs), so the zERT and AT-TLS facts here rest on the parent's earlier successful extraction of IBM documentation and IBM presentation PDFs, not on a fresh extraction this run.
- **IBM Security zSecure product naming** — the family exists and is described by IBM as its RACF administration/audit and SMF analysis product line, but the exact product naming was not re-verified this pass (⚠). Confirm before a procurement references it.
- **z/OS Connect** — the product exposes CICS/IMS resources as REST/JSON APIs, but its exact capabilities and version behaviour were not re-verified this pass (⚠). Verify against IBM before a design asserts detail.
- **IBM Network Authentication Service for z/OS (Kerberos)** — the AD/Kerberos route into z/OS is named by IBM as an identity-mapping mechanism, but its detailed configuration was not re-verified this pass (⚠).
- **Individual `SETROPTS` option behaviour** — the class and command names are as IBM documents them; the specific behaviour of any option (RACLIST, AUDIT, ERASE, SECLEVEL and so on) is release-specific and must be checked against IBM's current z/OS documentation before it is named in a change (⚠).
- **Prevalence of shared identities, vendor support access and spool exposure across the industry** — **no figure is asserted anywhere in this guide**, by design. The findings are presented as the common practitioner pattern with the explicit statement that *practice varies by institution and by estate era* (⚠, unsourced by intent).
- **Regulatory specifics** (DORA's exact applicability date and requirements; MAS TRM/Notice 645 provisions; SWIFT CSP control wording) — carried from the sibling guides' established ledgers and cross-referenced, not re-derived or re-verified this pass (⚠).
- **Session-recording product capability details** — the products named (§5.3) exist and are used in the industry for mainframe privileged access and session management, but this guide makes **no capability claim beyond that** and asserts **no institution's deployment** (⚠-structural).

### 16.2 The Cross-References

Because this guide is the constrained-case branch of the security cluster, the reader's next stop depends on the question:

- **The whole-estate zero-trust plan, the tenets, the pillars, the vendors and the maturity model** — [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) (§§1–7, and §9.3's phased plan and §7.1's CISA model, applied here in §12).
- **The Beyond Zero thesis** — [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md).
- **The legacy estate's shape and the "integrate, don't replace" modernisation patterns** — [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (§6, §7, §10).
- **The network-side observation answer where the platform's logging is not centralised** — [network_packet_capture_guide.md](network_packet_capture_guide.md) (used in §9.3).
- **The security discipline and the SOC that consumes the audit feed** — [cybersecurity_guide.md](cybersecurity_guide.md), [secops_guide.md](secops_guide.md).
- **Threat-modelling the findings** — [threat_modeling_guide.md](threat_modeling_guide.md) (STRIDE against the five weaknesses in §4).
- **The file-sharing paths and their notification patterns** — [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) and [linux_file_sharing_notification.md](linux_file_sharing_notification.md) (§8 here owns only the security consequences).
- **The transfer and scheduling products** — [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [control_m_guide.md](control_m_guide.md), [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md).
- **The identity mechanics underneath the bridge** — [distributed_auth_guide.md](distributed_auth_guide.md); the enterprise AI-gateway variant of the identity-aware proxy — [enterprise_ai_gateway_guide.md](enterprise_ai_gateway_guide.md).

### 16.3 The Closing Summary

This guide set out to write the constrained case of zero trust — the platform that cannot be a policy-enforcement point — and it arrived at a thesis with two halves.

The first half is a correction: **the mainframe's security model is mature, granular and resource-level, and it enforces authorisation for every accessor whether or not the accessing program cooperates** (§3). Dataset-level protection is a platform primitive, not an application feature; the audit records are comprehensive and decades-old; workload identities, MFA, certificate mapping, one-time PassTickets and network access control all exist on the platform today. The myth that the mainframe is an insecure legacy box is false, and it is dangerous, because it justifies doing nothing.

The second half is the honest gap: **the platform does not participate in the enterprise identity plane or the enterprise audit plane by default, and the five real weaknesses all live in that seam** — shared and static credentials, the file-transfer path, the spool, the 3270/emulator path, and ungoverned vendor support access (§4). Each is reduced by the **identity bridge** (§5, first because it is the prerequisite for every other control), by **vaulting and brokering the privileged identities** (§6), by an **identity-aware gateway and honestly-scoped session recording** on the interactive path (§7), by **identity, scope and encryption discipline** on the transfer path (§8), by **segmentation and network observation** (§9), and by **collecting the audit feed the platform has been producing all along** (§10).

What remains after all of that must not be hidden: it must be distinguished from what is merely unattempted, given a compensating control with its limits stated, owned by a **named accountable role**, reviewed on a date, and accepted in writing (§11) — exactly as Cymbal Bank did with RR-07 (§13.4). A residual risk that is named is a managed risk; the same risk unstated is the finding.

And so the guide closes on the sentence it opened with, because it is the whole argument:

**the mainframe is not insecure; it is un-integrated.**

---

## The Glossary

| Term | Definition |
|---|---|
| **ACEE** | Accessor Environment Element — the z/OS control block representing a user's identity; created by the ESM on request by resource managers via SAF APIs ✅ |
| **ACF2 / Top Secret** | Broadcom's alternative External Security Managers, plugging into SAF alongside IBM RACF ✅ |
| **AT-TLS** | Application Transparent Transport Layer Security — z/OS Communications Server capability adding TLS on behalf of applications that were never written for it, by policy in the Policy Agent ✅ |
| **Audit record** | The platform's long-established record stream, SMF — e.g. type 80 (RACF processing), type 14/15 (dataset activity), type 119 (zERT) ✅ |
| **Bridge (the identity bridge)** | The mechanisms that map enterprise identities into platform userids and stream platform records out — RACF identity propagation, IDIDMAP, certificate mapping, Kerberos, PassTicket in; IBM Z CDP / SMF out ✅ |
| **Class** | A category of protected resource in the ESM (DATASET, USER, GROUP, STARTED, JESSPOOL, SURROGAT, …) ✅ |
| **Dataset protection** | Authorisation applied to datasets and dataset prefixes (high-level qualifiers) at the platform level ✅ |
| **Emulator** | The 3270 terminal-emulation client (or browser emulator) presenting an interactive session — the path whose identity is frequently shared |
| **ENF event code 71** | The signal raised on identity profile changes (CONNECT/REMOVE/ALTUSER REVOKE) so subsystems flush cached authorisations ✅ |
| **ESM** | External Security Manager — the access-control facility (RACF, ACF2, Top Secret) enforcing authorisation behind SAF ✅ |
| **Firecall / emergency ID** | The break-glass identity for emergencies, with elevated authority, used when normal access is unavailable; must be named, vaulted and reviewed |
| **High-level qualifier (HLQ)** | The leading component of a z/OS dataset name, used as a scoping primitive (e.g. `PROD.PAYMENTS.**`) ✅-structural |
| **ICRX / IDID** | Identity Context Reference Extended, and the Distributed Identity Data control block (DN ≤ 246 bytes, registry name ≤ 255 bytes) that carries a foreign identity into the platform ✅ |
| **IDIDMAP** | The RACF general-resource class holding distributed-identity-to-userid mapping rules, managed via RACMAP ✅ |
| **IBM Z CDP** | IBM Z Common Data Provider — streams z/OS operational data (SMF, log data) to an analytics platform in near real time ✅ |
| **JES2 / JES3** | The z/OS job-entry subsystems that accept jobs and manage queues ✅ |
| **JESSPOOL** | The RACF class that protects data sets on spools ✅ |
| **PEP / PDP** | Policy Enforcement Point / Policy Decision Point — the modern access-decision roles the platform itself does not fill; the gateway is the PEP for the interactive path |
| **Profile** | The ESM's unit of protection within a class: a named resource or pattern carrying an access list ✅ |
| **RACF** | Resource Access Control Facility — IBM's External Security Manager, a component of the z/OS Security Server ✅ |
| **RACMAP / RACDCERT** | The commands to manage identity-mapping rules and RACF digital certificates respectively ✅ |
| **Region** | An address space hosting a subsystem (a CICS region, an IMS control region) |
| **RRSF** | RACF Remote Sharing Facility — lets RACF images communicate and share RACF databases ✅ |
| **SAF** | System Authorization Facility — the system interface that routes authorisation requests to the ESM (RACF and/or a user routine); an interface, **not** an ESM ✅ |
| **SAPI** | The spool-access programming interface for programmatic spool access ✅ |
| **SDSF** | System Display and Search Facility — the interactive tool for viewing and controlling jobs and spool output ✅ |
| **SMF** | System Management Facilities — the platform's long-established record source ✅ |
| **Spool** | The job-entry subsystem's output queue (JES2/JES3) — an information-disclosure path if un-scoped, protectable via JESSPOOL ✅ |
| **Started task** | A long-running address space with an identity assigned via the STARTED class or started-procedures table ✅ |
| **SURROGAT** | The RACF class enabling one userid to submit jobs for another (surrogate authority) ✅ |
| **Userid** | The platform's identity primitive — a 1-to-8-character string attached to every job, started task or transaction ✅ |
| **Workload manager** | A subsystem that runs work with its own identity semantics (CICS TS, IMS, DB2, MQ) — distinct from z/OS Workload Manager (WLM), the performance manager ✅ |
| **zERT** | z/OS Encryption Readiness Technology — reports the cryptographic protection of network flows, writing SMF type 119 (subtypes 11/12) ✅ |
