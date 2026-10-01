# Zero Trust for the Legacy Estate

*The estate-wide deep-dive — zero trust applied to the platforms that cannot be a policy-enforcement point and that are not the mainframe: the AS/400 and IBM i midrange, the MultiValue and PICK-family databases, the OS tier that cannot run the agent, OT and SCADA, the legacy database and middleware layers, and the file-transfer estate — the six classes, their shared deficit, the wrapping pattern that goes in front of each, the order to do them in, and the residual risk that must be owned in writing — all resting on one thesis: zero trust for a legacy system is built at its boundary, never inside it.*

*Jack Liu Shurui, Solution Architect*

**Last Updated:** October 2026

**Purpose.** This guide is the estate-wide companion to the constrained-case deep-dive. The enterprise identity plane (Okta, Microsoft Entra ID, the ZTNA broker) and the enterprise audit plane (the SIEM) both assume a subject that can be challenged and a log that can be streamed. A mainframe cannot be challenged; neither can an AS/400, a MultiValue database, an end-of-life appliance OS, a programmable logic controller, or a shared file-transfer identity. This guide covers the classes of that estate **other than the mainframe**: what each platform actually is, what security model it actually offers (established from the vendor's own documentation, not assumed), where the shared authority actually converges, and the **boundary wrapping** that goes in front of it. It deliberately does **not** re-derive zero-trust architecture, its history, NIST SP 800-207, the pillars, the vendor landscape, or the estate-wide migration phasing — those belong to [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md), which this guide cross-references by name and treats as its parent.

**Verification-markers convention.** ✅ = verified against a primary source (vendor documentation, standards body, regulator page) during this pass or in a cross-referenced sibling's ledger. ⚠ = flagged or partial — practice varies, or the claim rests on industry-standard practice rather than a single primary source. ❌ = could not be verified at source; recorded honestly in §16. The consolidated ledger is the Claims Audit (§15).

**How this guide is organised.** §1 is the overview, the decoder and the boundary. §2 is the class map — the six classes of the legacy estate. §3 is the common deficit they share, stated once. §4 is the wrapping pattern, by layer. §5 is the midrange (AS/400 and IBM i). §6 is the MultiValue family. §7 is the OS tier that cannot run the agent. §8 is OT and SCADA. §9 is the legacy database and middleware layers — the shared-credential problem. §10 is the file-transfer estate. §11 is how to prioritise across the estate. §12 is what wrapping does not fix. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified, holds the glossary and the cross-references, and closes.

---

## Table of Contents

1. [The Overview, the Decoder and the Boundary](#1-the-overview-the-decoder-and-the-boundary)
   - 1.1 [The Thesis — Built at the Boundary, Never Inside](#11-the-thesis--built-at-the-boundary-never-inside)
   - 1.2 [The Scope — the Estate Other Than the Mainframe](#12-the-scope--the-estate-other-than-the-mainframe)
   - 1.3 [The Decoder](#13-the-decoder)
   - 1.4 [The Boundary — What This Guide Does Not Own](#14-the-boundary--what-this-guide-does-not-own)
2. [The Legacy Class Map](#2-the-legacy-class-map)
   - 2.1 [The Midrange — AS/400 and iSeries](#21-the-midrange--as400-and-iseries)
   - 2.2 [The MultiValue and PICK Family](#22-the-multivalue-and-pick-family)
   - 2.3 [The Agentless OS Tier](#23-the-agentless-os-tier)
   - 2.4 [OT and SCADA](#24-ot-and-scada)
   - 2.5 [The Legacy Database and Middleware Layers](#25-the-legacy-database-and-middleware-layers)
   - 2.6 [The File-Transfer Estate](#26-the-file-transfer-estate)
3. [The Common Deficit](#3-the-common-deficit)
   - 3.1 [The Five Incapacities](#31-the-five-incapacities)
   - 3.2 [The Boundary Consequence](#32-the-boundary-consequence)
   - 3.3 [The Correspondence With the Mainframe Case](#33-the-correspondence-with-the-mainframe-case)
4. [The Wrapping Pattern, by Layer](#4-the-wrapping-pattern-by-layer)
   - 4.1 [The Six Layers](#41-the-six-layers)
   - 4.2 [How to Read the Pointers](#42-how-to-read-the-pointers)
5. [The Midrange: AS/400 and iSeries](#5-the-midrange-as400-and-iseries)
   - 5.1 [The Platform and the Workload Model](#51-the-platform-and-the-workload-model)
   - 5.2 [The Platform's Own Security Model](#52-the-platforms-own-security-model)
   - 5.3 [The Interactive Path — 5250 and the Emulator Session](#53-the-interactive-path--5250-and-the-emulator-session)
   - 5.4 [What the Platform Cannot Do](#54-what-the-platform-cannot-do)
   - 5.5 [The Wrapping Pattern for the Midrange](#55-the-wrapping-pattern-for-the-midrange)
6. [The MultiValue Family](#6-the-multivalue-family)
   - 6.1 [The Data Model and the Account](#61-the-data-model-and-the-account)
   - 6.2 [Where the Security Actually Lives](#62-where-the-security-actually-lives)
   - 6.3 [The Direct-Access Path](#63-the-direct-access-path)
   - 6.4 [The Wrapping Pattern for MultiValue](#64-the-wrapping-pattern-for-multivalue)
7. [The OS Tier That Cannot Run the Agent](#7-the-os-tier-that-cannot-run-the-agent)
   - 7.1 [The Situation](#71-the-situation)
   - 7.2 [The Agentless Options, by Mechanism](#72-the-agentless-options-by-mechanism)
   - 7.3 [The Compensating Controls That Genuinely Work](#73-the-compensating-controls-that-genuinely-work)
   - 7.4 [The Honest Statement — Agentless Is Weaker](#74-the-honest-statement--agentless-is-weaker)
   - 7.5 [The Auditor Question](#75-the-auditor-question)
8. [OT and SCADA](#8-ot-and-scada)
   - 8.1 [The Inverted Priority](#81-the-inverted-priority)
   - 8.2 [Why the Plant Rejects the IT Cadence](#82-why-the-plant-rejects-the-it-cadence)
   - 8.3 [The Consequence — Where the Enforcement Point Can Sit](#83-the-consequence--where-the-enforcement-point-can-sit)
   - 8.4 [What This Guide Takes From the OT Guide](#84-what-this-guide-takes-from-the-ot-guide)
9. [The Legacy Database and Middleware Layers](#9-the-legacy-database-and-middleware-layers)
   - 9.1 [The Shared Credential — the Largest Unwrapped Hole](#91-the-shared-credential--the-largest-unwrapped-hole)
   - 9.2 [Why the Shared Credential Defeats the Application Controls](#92-why-the-shared-credential-defeats-the-application-controls)
   - 9.3 [The Middleware Equivalent](#93-the-middleware-equivalent)
   - 9.4 [The Wrapping Pattern — Brokering and Vaulting](#94-the-wrapping-pattern--brokering-and-vaulting)
   - 9.5 [The Honest Limit](#95-the-honest-limit)
10. [The File-Transfer Estate](#10-the-file-transfer-estate)
    - 10.1 [The Transfer Identity](#101-the-transfer-identity)
    - 10.2 [The Directory and Scope Question](#102-the-directory-and-scope-question)
    - 10.3 [The Protocol and Encryption Question](#103-the-protocol-and-encryption-question)
    - 10.4 [The Scheduling Dependency](#104-the-scheduling-dependency)
    - 10.5 [The Finding — the Transfer Is Usually the Weakest Link](#105-the-finding--the-transfer-is-usually-the-weakest-link)
11. [Prioritising Across the Estate](#11-prioritising-across-the-estate)
    - 11.1 [Inventory First](#111-inventory-first)
    - 11.2 [The Five Signals](#112-the-five-signals)
    - 11.3 [The Ordering Method](#113-the-ordering-method)
    - 11.4 [What This Section Is Not](#114-what-this-section-is-not)
12. [What Wrapping Does Not Fix](#12-what-wrapping-does-not-fix)
    - 12.1 [Boundary Versus System](#121-boundary-versus-system)
    - 12.2 [The Insider With Granted Authority](#122-the-insider-with-granted-authority)
    - 12.3 [The In-Application Vulnerability](#123-the-in-application-vulnerability)
    - 12.4 [The Inherited-Control Problem](#124-the-inherited-control-problem)
    - 12.5 [Replace Rather Than Wrap](#125-replace-rather-than-wrap)
    - 12.6 [The Residual Risk That Must Be Owned](#126-the-residual-risk-that-must-be-owned)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
    - 13.1 [The Estate and the Inventory](#131-the-estate-and-the-inventory)
    - 13.2 [The Prioritisation](#132-the-prioritisation)
    - 13.3 [The Wrapping Pattern Applied](#133-the-wrapping-pattern-applied)
    - 13.4 [The Auditor Question](#134-the-auditor-question)
    - 13.5 [The One System Cymbal Replaces Rather Than Wraps](#135-the-one-system-cymbal-replaces-rather-than-wraps)
    - 13.6 [The Thesis, Restated](#136-the-thesis-restated)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified](#161-what-could-not-be-verified)
    - 16.2 [The Glossary](#162-the-glossary)
    - 16.3 [The Cross-References](#163-the-cross-references)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview, the Decoder and the Boundary

### 1.1 The Thesis — Built at the Boundary, Never Inside

**Zero trust for a legacy system is built at its boundary, never inside it.** That sentence is the whole guide, and it is deliberately narrower than the mainframe guide's thesis — it is not a claim about the platform's character, it is a claim about where the architecture has to go.

The reason is structural, not cultural. A legacy platform of the kind this guide covers frequently **cannot enforce the enterprise's access policy on its own behalf**. It has no device posture, no conditional access, no enterprise single sign-on, no per-request policy decision, and often no current vendor to change any of that. What it does have — in the midrange case, and on several others — is a real and sometimes rigorous *authorisation* model of its own (§5). That model decides *what an identity may do*; it does not decide *whether the session should be admitted at all*. Those are different decisions, made at different points, and only the second one is zero trust's native territory.

So the design rule this guide carries through every class is:

> The trust decision — authenticate the human, evaluate the device, apply the enterprise's policy — is made by a component **in front of** the platform. The platform consumes the identity that component delivers and continues to enforce its own authorisation. The platform is a resource server; the boundary is the policy-enforcement point.

The corollary is the honest one: anything the boundary cannot reach is not wrapped, and anything not wrapped is residual risk that must be named and owned (§12.6). A boundary is a perimeter around *some* paths, not all of them, and pretending otherwise is the most expensive mistake in this space.

### 1.2 The Scope — the Estate Other Than the Mainframe

This guide owns the **legacy estate outside the mainframe**: the platforms that share the mainframe's incapacity to be a policy-enforcement point, but that are *not* the z/OS estate and are not covered by the mainframe guide.

Six classes, each a class rather than an individual system:

1. **The midrange** — AS/400, iSeries, IBM i (§5).
2. **The MultiValue and PICK family** — jBASE, UniVerse, UniData and relatives (§6).
3. **The agentless OS tier** — an unsupported or end-of-life OS, an embedded appliance OS, that cannot run the security agent (§7).
4. **OT and SCADA** — the industrial control estate (§8).
5. **The legacy database and middleware layers** — the shared database credential and the integration-broker service identity (§9).
6. **The file-transfer estate** — managed file transfer and its transfer identities (§10).

This guide's territory is:

- what each class is and who runs it (§2);
- the deficit the classes share (§3);
- the **wrapping pattern**, layer by layer (§4), applied per class (§5–§10);
- **how to order the work** across an estate rather than within one system (§11);
- what wrapping genuinely does **not** fix, and the **replace-versus-wrap** criterion (§12);
- a worked example (§13) and the anti-patterns (§14);
- the claims audit (§15) and what could not be verified (§16).

What this guide does **not** own: zero trust as a discipline, its history, NIST SP 800-207 and its tenets, the five pillars, the vendor landscape, the CISA Zero Trust Maturity Model, or the estate-wide migration phasing — all of which belong to [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md). **The mainframe guide, [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md), already develops the shared patterns at length for one platform — the identity bridge, the five weaknesses, privileged access, the interactive path, segmentation and the audit feed — and this guide does not restate them.** Where a pattern applies to another class, this guide names the platform's own shape and points at the mainframe guide's section that develops the mechanism.

### 1.3 The Decoder

Zero trust applied to the legacy estate fails at first contact for the same avoidable reason it fails on the mainframe: the two sides do not share a vocabulary. Eight terms carry the whole guide. Each is either the platform's own or the enterprise's, and each is used exactly this way throughout.

| Term | What it is | The enterprise-side analogue |
|---|---|---|
| **The legacy class** | A group of platforms that share a deficit profile and therefore a wrapping pattern — midrange, MultiValue, agentless OS, OT, database/middleware, MFT. The class, not the individual system, is the unit of design in this guide | A technology domain or a risk cohort |
| **The boundary** | The set of components placed in front of a platform that make the trust decision it cannot make itself — entry point, identity bridge, brokering, protocol gateway, network path, observation point (§4) | The PEP and its policy plane |
| **The enforcement point** | The specific component that *denies or admits* a session — the identity-aware gateway, the broker, the protocol gateway, the network control. In every class here it sits **in front of** the platform, never on it | The policy-enforcement point (PEP) |
| **The identity bridge** | The set of mechanisms that map enterprise identities to platform identities and stream platform records outward — the join between the two planes. Developed for the mainframe case in [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5 | The provisioning connector plus the log shipper |
| **The protocol gateway** | A component that terminates a legacy protocol (5250, a database wire protocol, an OT protocol, an MFT protocol) on the enterprise side, applies policy, and re-originates the session inward. The midrange case is the mainframe shape ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.2) | The application proxy / protocol broker |
| **The inherited control** | A security property a class *borrows* from the layer beneath it — an unpatched host OS's file permissions, an application's own role logic — rather than one the platform enforces. A class may have **no security model of its own** (§6.2) | A control with an unowned dependency |
| **The agentless posture** | The state of a system for which endpoint instrumentation (an EDR/security agent) cannot be deployed, so detection must come from network, protocol or log observation instead (§7) | Compensating, network-based detection |
| **The estate inventory** | The authoritative, dated list of legacy platforms, versions, entry points, identities, transfer flows and data classes. Nothing can be prioritised that is not inventoried (§11.1) | The asset register / CMDB |

Two conventions follow from the table and hold throughout. First, **"the platform cannot be the PEP"** always means *for the enterprise's policy* — a platform may enforce its own authorisation perfectly well and still be unable to admit or deny a session against device posture or conditional access. Second, **"inherited"** is not a compliment and not an insult; it is a statement about which layer a control actually lives in, and therefore which layer must be patched, governed and owned.

### 1.4 The Boundary — What This Guide Does Not Own

The repo's security and legacy clusters are deliberately de-duplicated. This guide re-derives nothing from its siblings; it cross-references them **by name** and states the boundary explicitly.

| Sibling guide | What it owns — do not re-derive here | How this guide uses it |
|---|---|---|
| [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) | The z/OS constrained case in full: **§1.2** the system that cannot be a PEP, **§4** the five weaknesses, **§5** the identity bridge, **§6** privileged access and vaulting, **§7** the 3270 path and session recording, **§9** segmentation and the observation point, **§10** the audit feed, **§11** the residual-risk register, **§12.1** the platform-side phasing | **The primary pattern source.** Where a general pattern applies to another legacy class, this guide names the class's own shape and points at the mainframe guide's section rather than restating the mechanism |
| [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) | Zero trust generally: §1 the ZTNA overview, §2 the history, §3 NIST SP 800-207 and the seven tenets, §4 the five pillars, §5 the architecture, §6 the vendors, §7 the implementation and the CISA ZTMM, §9.3 the phased migration plan | The parent. Cross-referenced for the discipline and the plan; this guide's §11 is the **estate-level ordering**, a different question from that plan's within-a-system phases |
| [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) | The Beyond Zero paper and the agent-era paradigm; §4 the architecture, §5 the mechanisms, §6 the vs-zero-trust | Cross-ref only |
| [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) | The estate from the **integration** side — §6 the legacy patterns, **§7 the modernisation patterns**, §10 "Integrate, Don't Replace" | Cross-ref in §12.5 for the replace-versus-wrap criterion; re-derive none of it |
| [network_packet_capture_guide.md](network_packet_capture_guide.md) | The network-side monitoring answer where the platform's own logging is not in the SIEM — the observation point, capture mechanics, the loss problem | Cross-ref in §7.2 and §7.3 as the observation point |
| [secops_guide.md](secops_guide.md) | Security operations — §3 the detection stack (**NDR is the only visibility for the legacy-core estate where endpoint agents cannot be installed**), §4 the response stack | Cross-ref in §7.2 for what consumes the agentless detection |
| [cybersecurity_guide.md](cybersecurity_guide.md) | The security-discipline overview | Cross-ref, no re-derivation |
| [threat_modeling_guide.md](threat_modeling_guide.md) | Threat-modelling methodology — **§3.6 / §9 draw core-banking hosts as external entities outside the model (the DFD-per-trust-zone rule)** | Cross-ref in §12.3 for the deferred-zone lens |
| [vpn_guide.md](vpn_guide.md) | Remote access and its alternatives; §8 the Cymbal Bank estate VPN | Cross-ref in §7.2 where remote access to an agentless host is the question |
| [scada_guide.md](scada_guide.md) | The OT estate in full — §4 the Purdue model, §5 the protocols, §6 NIST SP 800-82r3, ISA/IEC 62443 zones and conduits and NERC CIP, **§8 the defence including "Zero Trust for OT"** | Cross-ref in §8; the OT segmentation model is **not** re-derived here |
| [ibm_as400_guide.md](ibm_as400_guide.md) | The midrange platform in full — §2 architecture, §3 the operating system, §5 languages and DB, §9 a banking core on the AS/400 | Cross-ref in §5 for the platform; this guide owns only the security model and the wrapping |
| [jbase_universe_guide.md](jbase_universe_guide.md), [jbase_vs_infobasic_guide.md](jbase_vs_infobasic_guide.md) | The MultiValue data model, the PICK heritage, jBASE/UniVerse/UniData internals, the T24 context | Cross-ref in §6 for the data model and account structure |
| [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md), [linux_file_sharing_notification.md](linux_file_sharing_notification.md) | The file-sharing paths and their notification patterns (Connect:Direct, Transfer CFT, FTP/SFTP, MQ, watchers, CDC) | Cross-ref in §10 for the transfer-path mechanics |
| [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) | The MFT products — Transfer CFT §5 security features and §10 regulatory compliance; the Control-M integration §9 security considerations | Cross-ref in §10 for the product-level controls |

The rule for the rest of this guide: **state the platform's own facts; cross-reference the enterprise's, and cross-reference the mainframe guide's patterns rather than restating them.**

---

## 2. The Legacy Class Map

The six classes below are the units of design in this guide. Each is described concretely — what it is, who runs it, and which sibling guide owns the platform detail — because the wrapping pattern is chosen per class, not per system.

### 2.1 The Midrange — AS/400 and iSeries

The IBM AS/400 — renamed iSeries, then **IBM i** on Power Systems — is a self-contained system with its own integrated operating system (originally OS/400, now IBM i), its own object-based file system (the **library** and **IFS** namespaces), its own DB2-for-i database integrated into the OS, and its own user-profile security model (§5.2). It runs the back-office cores, ERP and core-adjacent systems of a very large number of mid-sized and large institutions, often in exactly the role a mainframe plays elsewhere: a system of record that the business cannot stop. The sibling platform detail is [ibm_as400_guide.md](ibm_as400_guide.md); the security model is this guide's own (§5).

### 2.2 The MultiValue and PICK Family

The MultiValue family — **jBASE**, Rocket **UniVerse**, Rocket **UniData** and relatives — descends from the PICK operating system and stores data as hashed, variable-length, delimiter-separated records in files addressed through a **VOC/MD** file, with dictionaries rather than a schema and a host-language runtime (BASIC) rather than a server-side stored-procedure language. The platform detail — the data model, the PICK heritage, the jBASE and UniVerse internals, the T24 context — is [jbase_universe_guide.md](jbase_universe_guide.md) and [jbase_vs_infobasic_guide.md](jbase_vs_infobasic_guide.md). The security consequence, which is this guide's own and is the important part, is developed in §6.

### 2.3 The Agentless OS Tier

The agentless OS tier is a class defined by incapacity rather than by brand: an **operating system that is out of support** (past end-of-life, no vendor patches), an **embedded or appliance OS** (a hardened and immutable image inside a network device, a storage array, a kiosk, a lab analyser, a building-management controller), or a **real-time or safety-certified OS** whose certification forbids installing a general-purpose agent. The class is defined by the inability to run the endpoint security agent the enterprise standard would deploy — and therefore by the need for network-, protocol- and log-based detection instead (§7). Its consumers are the SOC's detection and response functions ([secops_guide.md](secops_guide.md) §3–§4).

### 2.4 OT and SCADA

The operational-technology estate — supervisory control and data acquisition (SCADA), distributed control systems, programmable logic controllers, remote terminal units, and the industrial protocols between them — runs the physical plant: power, water, manufacturing, building systems, and in a bank, the facilities and the data-centre physical layer. The platform and segmentation detail is [scada_guide.md](scada_guide.md): its §4 Purdue model, §6 OT security standards and §8 defence including "Zero Trust for OT". This guide's contribution is only the part that bears on the estate-wide zero-trust design — the inverted priority and the placement of the enforcement point (§8).

### 2.5 The Legacy Database and Middleware Layers

This class is not a platform brand but a **layer**: the shared database credential through which many processes and many people reach the data, and the middleware equivalent — the queue manager's or integration broker's service identity through which many flows reach many systems. It is the class most often left out of an estate inventory entirely, because it is a *configuration* rather than a *box*, and it is developed fully in §9 as the single highest-value section of this guide.

### 2.6 The File-Transfer Estate

The file-transfer estate is the collection of managed file transfer (MFT) and ad-hoc transfer paths that move data between the estate and its partners, bureaux, regulators and other platforms: transfer products and their scheduling, the transfer identities, the directories in scope, and the protocols on the wire. Its mechanics are [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md), [linux_file_sharing_notification.md](linux_file_sharing_notification.md), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md). Its security consequences are this guide's own (§10) — and it is usually the weakest link in the chain.

---

## 3. The Common Deficit

This section states the shared deficit **once**. Everything after it is application; nothing after it restates the deficit per class.

### 3.1 The Five Incapacities

Across the six classes, the same five incapacities recur. Not every class exhibits all five, and the severity varies by class, but the pattern is what makes them one estate rather than six unrelated problems.

| # | The incapacity | What it means | Which classes exhibit it most |
|---|---|---|---|
| **i** | **It cannot be the policy-enforcement point for its own access** | It cannot authenticate a human against the enterprise's policy, evaluate device posture, apply conditional access, or make a per-session admission decision. It may enforce authorisation perfectly — that is a different decision | All six |
| **ii** | **It cannot run the agent you would deploy** | A current EDR/security agent cannot be installed — because the OS is out of support, because the image is immutable, because certification forbids it, or because the platform is a controller | Agentless OS tier, OT, and frequently midrange and MultiValue hosts |
| **iii** | **It cannot join the enterprise identity plane** | No native SSO, no joiner-mover-leaver from the enterprise directory, no per-person identity for the work that matters — the local identity directory is separate and manually maintained | Midrange in part, MultiValue, database/middleware, MFT |
| **iv** | **It cannot produce centralised audit output** | Its records exist locally but are not collected, normalised or correlated enterprise-side; or it produces no security-relevant record at the layer an agent would have produced | Midrange (rich records, poor collection), MultiValue (minimal), agentless OS tier, OT |
| **v** | **It frequently cannot be patched at all** | The vendor is gone or the release is frozen; the change window is prohibitive; certification or the counterparty forbids change. The control cannot be *fixed in place* | Agentless OS tier, OT, MultiValue, legacy middleware |

### 3.2 The Boundary Consequence

The five incapacities have one consequence, and it is the thesis of this guide:

> **The trust decision can only be made in front of them.**

If the platform cannot be the PEP for its own access (i), cannot host the agent (ii), cannot join the identity plane (iii), cannot emit to the audit plane (iv), and cannot always be patched (v), then every trust-relevant function has to be relocated outward — to an entry point, an identity bridge, a broker, a protocol gateway, a network path and an observation point (§4). That relocation is what "wrapping" means in this guide, and it is why the architecture is a boundary architecture rather than a platform-hardening exercise.

### 3.3 The Correspondence With the Mainframe Case

This is exactly the shape the mainframe guide develops on one platform. Its §1.2 establishes the system that cannot be a PEP; its §4 catalogues the five weaknesses that persist in the seam between the platform and the enterprise planes; its §5 builds the identity bridge; its §9 and §10 supply the network-side and audit-side answers. **The mechanism is not restated here.** What this guide adds is that the same shape recurs across six *classes* with different platforms, different protocols and different risk weights — and that the estate-level question (which class first, and why) is a different question from the platform-level question the mainframe guide answers.

Read §5–§10 of this guide as *instances* of the mainframe guide's patterns, and read §11–§12 as what is genuinely new at estate scale.

---

## 4. The Wrapping Pattern, by Layer

The boundary is not a product; it is a small set of layers. This section names them once, as a model, and points at the mainframe guide's section where the layer's mechanism is developed — because the mechanism is the same regardless of which legacy class it fronts.

### 4.1 The Six Layers

| Layer | What it is, in one line | Where the mechanism is developed |
|---|---|---|
| **(a) An identity-aware entry point** | The component that authenticates the user against the enterprise identity plane and re-originates the platform session under a per-user identity — the front door that makes the enterprise's policy decide admission | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§7.2** (the identity-aware gateway) — the pattern is identical in front of a 5250, a database wire protocol or an OT console |
| **(b) An identity bridge** | The join between the enterprise identity plane and the platform's own identity directory — provisioning, deprovisioning, per-user mapping, and the outward log path | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§5** (the identity bridge) — the two-plane model, the platform-side and enterprise-side halves, and the joiner-mover-leaver problem |
| **(c) Brokered privileged access** | Vaulting the platform's privileged and vendor identities and brokering the sessions, so that "who is doing this" has an answer and a holder | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§6** (vaulting, the firecall identity, session brokering, the named identity) |
| **(d) A protocol-level gateway** | A component that terminates the class's own protocol on the enterprise side — 5250, a database wire protocol, an OT protocol, an MFT control protocol — applies policy and inspection where an application proxy cannot, and re-originates the session inward | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§7.2** for the interactive protocol case, and its protocol-handling discussion in §7–§8 — the class's protocol differs; the gateway's role does not |
| **(e) Restricted network paths** | Segmentation around the platform so that only the entry points that must remain are reachable, and everything else — the direct path, the side door, the peer-to-peer route — is closed | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§9.1** (segmentation around the platform) and **§9.2** (the entry points that must remain) |
| **(f) An observation point** | The place where the class's traffic is captured, inspected and fed to detection when the platform's own logging is not in the SIEM — the network-side answer | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§9.3** (the observation point when the platform's own logging is not centralised), elaborated for the estate as a whole in [network_packet_capture_guide.md](network_packet_capture_guide.md) |

### 4.2 How to Read the Pointers

The six layers are **not** a build order and **not** a phase plan. They are a checklist for a wrapped class: in front of any legacy platform, ask whether each layer is present, and if not, why not. The build order is a different question, and it is settled by two facts established elsewhere in this guide: the identity bridge (b) is first because every other layer depends on it ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5.6), and the estate-level sequence across classes is set by §11 of this guide.

Two clarifications the reader should carry into §5–§10:

- **The protocol gateway (d) is the layer that varies most by class**, because every class speaks a different protocol. The *role* — terminate, inspect, apply policy, re-originate — is constant; the *specifics* are per class, and where a class's protocol detail belongs to a sibling guide, this guide cross-references rather than re-derives it (OT protocols → [scada_guide.md](scada_guide.md) §5; MFT protocols → [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) §3).
- **The observation point (f) is the layer most often mistaken for a substitute for the others.** A network capture point that sees traffic to an unwrapped platform observes an unwrapped platform. It supplies detection, not admission, and §7 makes that limit explicit.

---

## 5. The Midrange: AS/400 and iSeries

### 5.1 The Platform and the Workload Model

The AS/400 — iSeries, then IBM i on Power — is not a database server with an operating system bolted on; it is an integrated system in which the operating system, the database (**DB2 for i**), the file system and the security model are one design. Work is organised into **jobs** (interactive jobs driven from a workstation, and batch jobs submitted to job queues and run by a subsystem), the file system has two namespaces (the classic **library** namespace, where objects live in libraries, and the **IFS** — Integrated File System — which also exposes a UNIX-like directory tree and mounts the library namespace), and the database is accessed both natively (RPG, COBOL, CL through data-description specifications and record-level access) and through SQL (embedded, ODBC/JDBC, DRDA). [ibm_as400_guide.md](ibm_as400_guide.md) §2–§5 owns the platform, the OS, the languages and the DB; this section owns only the security model and its boundary.

The practical point for an architecture discussion: the same data is reachable through **several distinct entry surfaces** — the 5250 interactive path, an SQL/DRDA connection from a distributed application, file sharing (IBM i NetServer, an SMB/CIFS surface), FTP and other transfer services, and data queues or message queues. Each surface authenticates and authorises through the same underlying user-profile model, but each is a separate network entry point that must be enumerated and wrapped separately.

### 5.2 The Platform's Own Security Model

The midrange security model is real, is documented by IBM, and is more granular than its reputation suggests. Four components:

**User profiles.** An IBM i user profile is a first-class system object that identifies a user (or a service, or a group) and carries its authorities and attributes. Profiles are created and maintained with the documented commands — `CRTUSRPRF` (create), `CHGUSRPRF` (change), and the work/display commands (`WRKUSRPRF`, `DSPUSRPRF`) ✅. There are three profile roles: a **user profile** for a person, a **group profile** for a set of authorities shared by members (a profile can be a group profile, and users can belong to it — IBM documents the group-profile relationship), and profiles for system functions. Each profile carries **special authorities** (the broad system-wide authorities such as `*ALLOBJ`, `*SECADM`, `*JOBCTL`, `*SPLCTL`, `*AUDIT`, `*SAVSYS` and others) ✅ — `*ALLOBJ` in particular is the "access everything" authority that a midrange control review treats the way a mainframe review treats a powerful `SPECIAL` user.

**Object authority.** Access is enforced at the level of the **object**, not the row and not merely the file. The documented authority values are the system-defined sets — **`*ALL`, `*CHANGE`, `*USE`** — plus **`*EXCLUDE`**, which is "different than having no authority" (it is an explicit denial) ✅, and **`*AUTL`**, which directs the object's public authority to come from an **authorization list** rather than from the object itself ✅. Underlying the sets are object authorities (`*OBJOPR`, `*OBJMGT`, `*OBJEXIST`, `*READ`, `*ADD`, `*UPD`, `*DLT`, `*EXECUTE` and so on) and data authorities for files and members ✅ (IBM's "commonly used authorities" documentation names `*ALL`, `*CHANGE`, `*USE` and `*EXCLUDE` explicitly). Authority is granted and revoked with `GRTOBJAUT`/`RVKOBJAUT`, and edited or displayed with `EDTOBJAUT`, `DSPOBJAUT`, and — for IFS objects — `CHGAUT`, `DSPAUT`, `WRKAUT` ✅.

**The `*PUBLIC` authority and the authorization list.** Every object carries a **`*PUBLIC`** authority — the authority granted to *all* users who are not otherwise authorised — and this is the single most consequential setting on the platform. An object left at `*PUBLIC *CHANGE` or `*PUBLIC *ALL` is readable (or writable) by anyone on the system regardless of application design. The standard midrange control is to point sensitive files at an **authorization list** whose `*PUBLIC` is `*EXCLUDE` and whose entries are per-user or per-group ✅. The platform's own SQL services expose this for review: `QSYS2.OBJECT_PRIVILEGES`, `QSYS2.AUTHORIZATION_LIST_USER_INFO` and `QSYS2.PROGRAM_INFO` let a reviewer enumerate every object's `*PUBLIC` authority, its authorization list, its owner and its private authorities ✅ (IBM i Services; the review queries are the standard practitioner pattern).

**Adopted authority.** A program can carry a *`*USER`* attribute of `*OWNER`, meaning it runs with the authority of **its owner** rather than of the user who called it — **adopted authority** ✅. This is the mechanism that lets a tightly secured application expose a function to users who have no direct authority to the underlying files: the program adopts the owner's authority and performs the work. It is a legitimate and powerful design pattern, and it is also the reason a midrange application's *effective* access is often broader than its object-level grants suggest — the enforcement lives in the program, not only in the object. Related attributes (`USER`/`USEADPAUT`) control whether a program already running under adopted authority may adopt another program's authority deeper in the call stack.

**The audit journal and the system values.** IBM i writes security-relevant events to the **security audit journal**, `QAUDJRN`, which IBM documents as "the primary source of auditing information about the system" ✅. Entries are read with `DSPJRN`/`RCVJRNE`; the legacy `DSPAUDJRNE` command is documented but IBM notes it "does not support all security audit record types" and recommends receiving journal entries directly ✅. What is audited is controlled by the **security system values** — including `QSECURITY` (the system security level, the coarse platform-wide setting) and the auditing controls `QAUDCTL`/`QAUDLVL` ✅ (the precise option behaviour of individual system values is release-specific). This is a genuinely comprehensive platform audit facility by the standards of its era, and it is the midrange's equivalent of the mainframe's SMF feed (§3.1 (iv)).

### 5.3 The Interactive Path — 5250 and the Emulator Session

Interactive access to IBM i arrives either through a **5250** terminal data stream (a real terminal or, overwhelmingly, a desktop or browser **5250 emulator** such as IBM i Access Client Solutions) or through the newer browser and web-service fronts that applications have added. The 5250 path has the same three security properties the mainframe guide identifies on the 3270 path ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.1): the platform authenticates the user profile at sign-on and enforces authorisation thereafter; the emulator client frequently sits outside the enterprise SSO estate; and the session content is protocol data that generic enterprise observation tooling will not decode.

The consequence is the same as on the mainframe, and it is the reason this class gets the mainframe's interactive pattern rather than a new one: shared sign-on identities on the 5250 path (an operator profile used by several staff), no device posture, and sessions that are unobserved because nothing decodes them. The answer is the identity-aware entry point of §4 layer (a) — the gateway of [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.2 — carrying a per-user identity through the bridge of §4 layer (b).

### 5.4 What the Platform Cannot Do

The honest list, for the class:

- **It cannot join the enterprise identity plane by itself.** IBM i has its own user-profile directory and its own password model; it is not a member of the enterprise directory in the way an application on AD/Entra is. Directory integration exists as a project (a connector that provisions user profiles from the enterprise directory, and/or authentication offload through the platform's Kerberos and directory-services support ⚠ — the exact product and release detail must be verified against IBM before a design asserts it), but it is a bridge to be built, not a property the platform has.
- **It cannot emit a complete, normalised enterprise audit feed by itself.** `QAUDJRN` is rich and local. Getting it into the SIEM — collected, normalised, retained, correlated — is a pipeline project, exactly the audit-feed problem the mainframe guide develops ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §10.2), not a setting.
- **It cannot host the enterprise's endpoint security agent** in the way a commodity server can. Some agent classes have midrange support and some do not; where the OS is at a frozen release (a common state on an old AS/400), the agent question is decided by the release, not by preference ⚠ (agent availability by release varies and must be checked per product).
- **It cannot be the PEP for its own sign-on path.** It authenticates the profile it is given; it does not evaluate device posture or apply the enterprise's conditional-access policy. The gateway is the PEP.

None of this makes the platform weak — it makes it **un-integrated**, which is the same distinction the mainframe guide draws ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §1.1). The midrange has the mainframe's *shape* of problem, with a different vocabulary.

### 5.5 The Wrapping Pattern for the Midrange

Applying the six layers of §4 to an IBM i estate:

1. **Identity-aware entry point** (a) — in front of the 5250/emulator path and, separately, in front of the SQL/DRDA and file-sharing (NetServer) surfaces. The pattern is [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.2; the protocol differs.
2. **Identity bridge** (b) — provision user profiles from the enterprise directory, align personal identities to personal profiles so that the 5250 shared-operator profile disappears from the interactive path, and deprovision on the same event the enterprise uses for a leaver. The two-plane model and the joiner-mover-leaver problem are [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5.1–§5.4, and are **not** restated here.
3. **Brokered privileged access** (c) — vault the `*ALLOBJ`/`*SECADM`-class profiles, the service profiles used by batch and integration jobs, and any vendor-support profile. The vaulting-and-brokering pattern is [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6.
4. **Protocol gateway** (d) — policy and inspection in front of the 5250 stream and in front of the database wire protocol for the SQL surface, so that an application-tier connection can be attributed and can be restricted to the data it is meant to reach (§9's shared-credential problem is exactly this surface).
5. **Restricted network paths** (e) — close the entry points that must not remain (raw FTP, direct IFS shares, unneeded DRDA listeners) and allow only the wrapped ones. Segmentation model: [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §9.1–§9.2.
6. **Observation point** (f) — capture the 5250 and database traffic at the boundary, because the platform's own records are local until the audit feed of layer (b) is built. Network-observation method: [network_packet_capture_guide.md](network_packet_capture_guide.md).

Where the midrange differs from the mainframe in practice — and it is worth stating once — is that the midrange estate is usually **smaller, more numerous, and less governed** than the mainframe estate. There may be several IBM i systems, each with its own local user-profile directory, its own `*PUBLIC` exposure and its own `QAUDJRN`, and no single owner who has looked at all of them together. That is an inventory problem before it is a control problem (§11.1).

---

## 6. The MultiValue Family

### 6.1 The Data Model and the Account

To reason about access in a MultiValue database you have to know three things, and only three, at the level this guide needs. First, data lives in **hashed files** — variable-length records addressed through a hash of the record key, stored as ordinary files in the host operating system's file system. Second, every file is reached through a **dictionary**: the **VOC** file (in jBASE terminology the **MD** file) is the account's directory of files, commands, verbs and pointers, and each data file has its own dictionary describing the fields. Third, the unit of containerisation is the **account** — a directory in the host file system that holds the VOC, the file dictionaries and the data files, and from which a **LOGTO** (or equivalent) command switches the session. The data model, the PICK heritage and the jBASE/UniVerse/UniData internals are [jbase_universe_guide.md](jbase_universe_guide.md) §1–§5 and [jbase_vs_infobasic_guide.md](jbase_vs_infobasic_guide.md) §1–§3; this section uses only the access-relevant part.

The decisive structural fact — verified at the vendor's own documentation — is that **the account is a host directory, and the VOC, the dictionaries and the data files all live at the same level inside it**. Rocket's own UniVerse documentation states it plainly: "in any UniVerse account, the VOC file, all file dictionaries, and all data files are stored at the same level — that is, in the UNIX or Windows directory where the UniVerse account resides" ✅. That single sentence is the whole security architecture of the class, once you follow it through.

### 6.2 Where the Security Actually Lives

This guide's conclusion, and it is an assessment rather than a vendor capability claim: **the MultiValue family's access control is largely *inherited* — from the host operating system's file permissions and from the application layer's own logic — rather than enforced by a database-level privilege engine comparable to an RDBMS `GRANT` model.** The evidence, from the vendors' own documentation:

- **The files are host files, and the host protects them.** The storage layer is the UNIX/Windows file system ✅. Whatever that file system's permissions say about the account directory and the hashed files is what stands between a process and the data — not a database privilege model invoked by the database on each read.
- **jBASE's own account model carries an application-level password, not a data-level privilege system.** The jBASE `SYSTEM` file (located via `JEDIFILENAME_SYSTEM`) holds one record per account; field 2 is the **absolute host path of the account directory**, and field 7 is an **encrypted account password** maintained through the jBASE `PASSWORD` command ✅. The `LOGTO` command uses that record to switch accounts, and `JEDIFILENAME_MD` (with `JEDIFILEPATH`) defines where the account's MD/VOC and files are found ✅. This is a *sign-on-to-an-account* control — a door on the account — not a per-file, per-user authorisation model.
- **UniVerse's remote access is the same shape.** Access to files on another system runs through the **UniRPC** facility — the `unirpcd` daemon (or the `unirpc` service on Windows), gated by the `unirpcservices` file, which spawns a per-user `uvnetd` daemon to service remote file requests ✅ — and the documentation's own chapter heading for the topic is **"Remote file permissions"**, describing how the connection is permitted per host pair ✅. The control is *can this host reach that account over UniRPC*, layered on host permissions.
- **UniData follows the same family design**: an account-based, host-file-system-backed model with the platform's administration guides covering accounts and remote access rather than a database privilege catalogue ⚠ (the UniData administration guide documents accounts and file placement; a native per-record authorisation model is **not** something this guide could verify at source — see §16).

So the honest architecture statement is: **in the MultiValue class the security is mostly *not* in the database.** It is in (a) the host OS's file permissions on the account directory, which are only as good as the host's patch level and configuration (§7), and (b) the application's own logic and its dictionary/menu structure, which is only as good as the application's design. A MultiValue database on a well-secured host running a disciplined application is not automatically weak; a MultiValue database on an unpatched host with a shared OS login and a developer who can read the account directory is a direct route to every record in it.

### 6.3 The Direct-Access Path

Because the data is host files and the access model is account-level, there are two direct-access paths that **bypass the application's controls entirely**:

1. **The client/terminal straight into the database runtime.** Any account holder who can reach the runtime — a shell session that logs into the account, a client connection through the platform's remote-file facility — can open files and read records without passing through the application's menus, its validation, its role checks and its audit. The terminal session or the `REMOTE.B`-style remote call is a *data* path, not an application path.
2. **The OS-level read of the hashed files.** Because the hashed files are ordinary host files, anyone with file-system read access to the account directory can copy the files and reconstruct the records — the hash is an addressing scheme, not an encryption scheme. No database login is required. This is the path that makes the host OS's permissions the platform's real security boundary, and it is the reason §7 (the agentless OS tier) and §9 (the shared credential) sit immediately below this section.

Both paths are *bypasses*, and both are invisible to the application's own audit. That is the finding this class contributes to the estate: a MultiValue reporting database reached through the application looks controlled; the same database reached through the runtime or the file system does not, and the estate inventory usually records only the first path.

### 6.4 The Wrapping Pattern for MultiValue

The MultiValue class has **no platform-side PEP to build on**, so the boundary carries almost all of the load:

- **The application-tier proxy is the primary control.** All legitimate access should go through an application tier that owns the database connection and enforces the enterprise's identity and authorisation — the proxy becomes the enforcement point, and the database becomes reachable only from it. This is the §4 layer (a)/(d) pattern in its most literal form: because the platform cannot enforce, the front tier must.
- **The access broker and the credential question.** The application tier must authenticate the human (via the identity bridge, [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5) and then use a **scoped, vaulted service identity** to reach the database, not a shared account password embedded in a configuration file. The vaulting-and-brokering pattern is [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6; the shared-credential problem it solves is developed in §9 of this guide.
- **Close the direct paths.** The runtime shell and the remote-file facility are network entry points and must be segmented away from everything except the application tier and the administrators who need them (§4 layer (e), mainframe guide §9.1). OS-level file access must be restricted by host file permissions **and** by removing the accounts that have no business reading the account directory — which returns the problem, unavoidably, to the host OS's security and patch level (§7).
- **Observe at the boundary.** Because the platform produces minimal centralised audit, the observation point (§4 layer (f)) is doing more work here than in any other class: the network path between the application tier and the database, and the file-system access on the host, are where the detectable signal lives ([network_packet_capture_guide.md](network_packet_capture_guide.md); [secops_guide.md](secops_guide.md) §3).

The class-level truth to carry forward: **for MultiValue, wrapping is mostly *host hardening plus application-tier brokering*, because there is no database-level control to delegate to.** Saying so is the honest architecture, and it is better than asserting a security model the vendor does not document.

---

## 7. The OS Tier That Cannot Run the Agent

### 7.1 The Situation

There is a tier of the estate on which the enterprise's standard control — install the endpoint agent, let it report to the console, let the SOC see it — simply cannot be applied. The reasons are structural and none of them is a failure of will:

- **The OS is out of support.** The release is past end-of-life; there are no vendor patches and often no supported agent build for it. The platform still runs; the vendor has moved on.
- **The OS is embedded or appliance-based.** The image is immutable or vendor-controlled — a storage array's controller, a network device's management OS, a lab analyser, a kiosk, a building-management appliance. You cannot install software on it and were never meant to.
- **The OS is real-time or safety-certified.** Certification constrains what may run on it, and a general-purpose security agent is not on the list.

The class is defined by that incapacity and by **nothing else** — by brand it is not one platform but many. This is the only class in this guide whose members are not "products" at all.

### 7.2 The Agentless Options, by Mechanism

Where the agent cannot go, detection has to come from outside the host. Five mechanisms are available, and they are not equivalent:

1. **Network-based detection (NDR).** Traffic to and from the host is the instrument. This is the mechanism [secops_guide.md](secops_guide.md) §3 already names as **the only visibility for the legacy-core estate where endpoint agents cannot be installed** — it is a citation, not a re-derivation. NDR sees what crosses the wire and infers behaviour; it does not see what happens inside the host.
2. **Protocol-level inspection.** Where the host speaks an inspectable protocol, a gateway in the path (the §4 layer (d)/[zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.2 pattern) can terminate it, decode it and enforce or log policy on the content — which is strictly more than raw packet inspection where the protocol is understood.
3. **Log collection from a syslog-capable host.** If the platform can emit syslog (or any structured log) to a collector, that stream is genuinely useful and should be collected — but "syslog-capable" is the qualifier, and many of the systems in this class are not, or emit only what their own code chose to emit.
4. **SNMP and management interfaces.** Management planes (SNMP, vendor management APIs, IPMI-class interfaces) can yield configuration, health and sometimes change events — useful for inventory and availability, weak as a security signal, and themselves an attack surface that must be wrapped, not trusted.
5. **The observation point itself.** The capture point of [network_packet_capture_guide.md](network_packet_capture_guide.md) is the physical locus where mechanisms 1–3 are actually implemented; its §2 (where you can capture) and §4 (the loss problem) determine how much of the truth the SOC ever sees.

### 7.3 The Compensating Controls That Genuinely Work

Detection is not the only compensation, and the ones that work best are the ones that **remove paths rather than observe them**:

- **Restrictive network paths.** The single most effective agentless control (see §4 layer (e); [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §9.1): if only two systems can reach the host, the exploitable surface is two systems, not the segment. This is what the legacy-appliance class can actually be governed by.
- **Brokered access.** No direct human session to the box; access goes through a broker that authenticates against the enterprise plane and records the session ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6, §7.3). Brokering substitutes an enterprise-controlled door for a local one.
- **Protocol-level inspection and enforcement.** Where the protocol is understood, the gateway can block as well as observe — a prevention control, not a detection one.
- **Aggressive removal of the widest authority.** If the host's own administrative account is shared, that is a control gap that needs no agent to fix: name it, scope it, vault it ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6.4).

### 7.4 The Honest Statement — Agentless Is Weaker

**Agentless coverage is weaker than agented coverage, and the difference cannot be closed by insisting that it does not exist.** This sentence is the section. Precisely, and without reassurance:

- An agent sees **process execution, file access, registry/configuration change and memory** on the host. Network observation sees **flows between hosts**, and protocol inspection sees **the content of the protocols it terminates**. The gap between those two views is not a tuning problem; it is a difference in what is observable at all.
- An agent can **block** an action on the host in real time. Network and protocol controls can block a *flow* or a *session*; they cannot stop a local process the host itself launched.
- An agent attests **host state** — patch level, running services, file integrity. Without it, host state is inferred from the outside, or taken on trust, or unverified.
- Unknown-behaviour detection on the network degrades **exactly where the estate is most sensitive** — east-west traffic inside a trusted segment, encrypted protocols the observation point cannot decrypt ([network_packet_capture_guide.md](network_packet_capture_guide.md) §9), and low-and-slow activity that looks like normal operations.

The consequence for an architecture is not despair; it is **scoping**. The correct statement to a governance forum is: "This host is covered by compensating controls — network restriction, brokered access, protocol inspection and network detection — and the compensating-control limit is that we do not have host-level process and file visibility, we cannot block a local process, and we do not attest its state. The residual risk is recorded and owned." That is a stronger position than pretending the dashboard is complete, and it is the same honest-accounting discipline the mainframe guide applies to its own compensating controls ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.4, §11.1).

### 7.5 The Auditor Question

The practical question a legacy-security programme is actually asked is: **what do you tell an auditor about a system you cannot instrument?**

The defensible answer has four parts, and none of them is "we monitor it":

1. **Name it as a class.** "This host is in the agentless class, for a documented structural reason — end-of-life release / immutable image / certification constraint — and it is inventoried as such."
2. **State the actual controls.** Which network paths remain and which were closed; whether access is brokered; whether the protocol is inspected; what is fed to the SOC ([secops_guide.md](secops_guide.md) §3) and what is not.
3. **State the compensating-control limit, in the auditor's language.** What the controls do not see (host process/file activity, host state) and therefore what a detection capability cannot promise on this host — the §7.4 statement, volunteered rather than extracted.
4. **Produce the ownership.** The risk that survives is described, owned by a **named role**, reviewed on a date, and accepted in writing — the §12.6 requirement and the [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §11.2–§11.3 register.

An auditor's real objection is rarely to the control gap; it is to the gap that was not disclosed. Disclosing it, scoping it and owning it converts an unknown into a registered, reviewed exposure — the same move the mainframe guide makes on its own platform.

---

## 8. OT and SCADA

### 8.1 The Inverted Priority

Every other class in this guide inverts the *implementation* of zero trust, but OT inverts the **priority order itself**. In the IT estate the classic triad ordering — confidentiality, integrity, availability — is, at minimum, a defensible starting point. In the operational estate it is not: **safety and availability outrank confidentiality**, and a control that improves confidentiality at the cost of availability is not a control the plant will accept. This is not a cultural preference to be argued away; it is the correct ordering for a system whose outage can injure a person, flood a plant or dark a district.

The security consequence is immediate and it is the one point this guide makes about OT: **a control that stops production is not a control.** An access-control decision that can interrupt a control loop, a safety interlock, a protective relay or a real-time setpoint is, from the plant's perspective, an availability attack that arrived wearing a security badge. Zero trust does not get an exemption from this — it has to be designed around it.

The estate and its models belong to [scada_guide.md](scada_guide.md): its §1 overview, §2 the ICS family taxonomy, §3 the control loop, §4 the Purdue model, §5 the protocols, §6 the OT security standards, §8 the defence including "Zero Trust for OT", §10 the Cymbal Bank OT estate, and §15 the closing summary. **None of that is re-derived here.** This section answers only the question the estate-wide zero-trust design must answer: *given §3's five incapacities and §4's wrapping pattern, where can the enforcement point even sit in an OT estate?*

### 8.2 Why the Plant Rejects the IT Cadence

Three structural facts make the IT playbook inapplicable as-is:

- **The plant cannot be patched on the IT cadence.** A controller firmware upgrade is an engineering change: it may require a production outage, vendor recertification, revalidation of the process, and a maintenance window measured in months. The patch that closes a vulnerability may be a change the plant will not schedule for a year. This is incapacity (v) of §3.1 in its most severe form.
- **The plant has no agent.** Controllers, RTUs and safety systems are the archetype of the agentless class (§7): immutable, certified, resource-constrained. Endpoint instrumentation is not available, and the honest agentless statement of §7.4 applies in full.
- **The plant's protocol is the process.** The protocols (Modbus, DNP3, IEC 61850, OPC and the rest; [scada_guide.md](scada_guide.md) §5) carry the control loop. They are not request-response transactions that tolerate a proxy's latency; several are designed without authentication and cannot be changed without replacing the devices that speak them. A gateway that inspects them is possible; a gateway that *remediates* them is constrained by what the endpoints accept.

### 8.3 The Consequence — Where the Enforcement Point Can Sit

The consequence follows from §8.1–§8.2 and it is the OT contribution to the wrapping pattern:

> **The enforcement point has to be placed where it cannot interrupt the process.**

In practice that means the security decision is pushed **outward and upward** from the process, to the layers where admission can be denied without touching the loop:

- **At the IT/OT boundary** — the zone boundary of the segmentation model ([scada_guide.md](scada_guide.md) §4 Purdue, §6 zones and conduits under ISA/IEC 62443) — where identity, policy and inspection can apply to *enterprise* access to the OT estate without sitting in the control loop. This is where zero trust for OT is actually enforceable, and it is the "Zero Trust for OT" discussion [scada_guide.md](scada_guide.md) §8 already owns.
- **At the remote-access path** — vendor and engineer remote access into the plant is the OT estate's own version of the vendor-support-identity problem ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.6): brokered, time-bound, recorded, and admitted only on the enterprise side of the boundary. The brokering pattern is [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6.
- **At the observation point** — passive monitoring that reads the process network without participating in it. This is the OT archetype of the agentless detection of §7 and of [network_packet_capture_guide.md](network_packet_capture_guide.md): a tool that observes and cannot affect the loop is the only kind accepted inside a process zone.
- **Never inside the loop.** A control placed between a controller and its process equipment, or between the controller and its I/O, is a control whose failure mode is the plant stopping. The design rule is to keep the enforcement point's *denial* incapable of reaching the loop.

The identity half of zero trust still applies at the boundary: the OT estate's users (engineers, vendors, operators) are still people who should be identified, provisioned and deprovisioned through the enterprise identity plane ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5), and the OT estate's administrative identities are still shared-and-static credentials of the §9/§3.1(i) kind. The wrapping is real; it just lives one zone up.

### 8.4 What This Guide Takes From the OT Guide

Four things, all cross-referenced and none restated:

1. **The segmentation model** — [scada_guide.md](scada_guide.md) §4 (Purdue) and §6 (ISA/IEC 62443 zones and conduits; NIST SP 800-82r3; NERC CIP where the jurisdiction's grid obligations apply) — is the model this guide's §4 layer (e) takes as given for the OT class, exactly as [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §9.1 takes segmentation as given for the mainframe.
2. **The OT threat landscape and defence** — [scada_guide.md](scada_guide.md) §7–§8 — is not re-derived.
3. **The OT worked example and residual-risk treatment** — [scada_guide.md](scada_guide.md) §11–§12 — supplies the class-specific incident-response and acceptance discipline; §12.6 of this guide adds only the estate-wide ownership requirement.
4. **The one-line rule** — the enforcement point sits where it cannot interrupt the process — is this guide's own contribution to the class, and it is the answer to "can zero trust be applied to OT?" for the estate-level design: yes, at the boundary, with the same placement logic as every other class, and with a stricter definition of what "cannot interrupt" means.

---

## 9. The Legacy Database and Middleware Layers

This is the highest-value section in the guide, because this class is where the largest unwrapped hole usually sits — and because it is the one class that is a *configuration*, not a box, so it is routinely missing from the estate inventory that §11 begins with.

### 9.1 The Shared Credential — the Largest Unwrapped Hole

The pattern: **one database credential — one username and password — through which many processes and many people reach the data.** It is used by the batch jobs, the application servers, the reporting extract, the ETL feed, the reconciliation script, the ad-hoc analyst query, the vendor tool, and often by humans who were given it because giving them a personal account was slower. It is almost always privileged, frequently static, frequently long-lived, and frequently stored in the same configuration file wherever it is used.

The middleware equivalent is the same shape one layer up: **the queue manager's or integration broker's service identity**, through which many flows reach many systems. One MQ service account, one integration-broker service principal, one ESB credential, through which hundreds of message flows to dozens of back ends are authenticated. The integration broker is *designed* to be a hub with broad reach, and its identity is therefore the most powerful identity in the estate that nobody classified as privileged.

Both are the shared-and-static-credential weakness the mainframe guide catalogues as its first weakness ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.1 (i), §4.2) — and the reason this guide treats it as its own section is that **on the distributed legacy estate it is the dominant exposure, whereas on the mainframe the platform's own STARTED/class model at least gives it a shape.** Nothing platform-side does that here.

### 9.2 Why the Shared Credential Defeats the Application Controls

The application above a legacy database usually implements controls that are real and that an auditor has tested: roles, segregation of duties, maker-checker, field-level masking, an audit trail of who did what. **The shared credential bypasses every one of them**, because it is the credential the application itself uses.

- **The application's audit trail attributes to the application, not the person.** If the connection is the shared service account, the database's notion of the actor is that account; the person's identity lives only in the application's own log, and any access *not* through the application has no person attached at all.
- **Anyone who holds the credential has the application's access, without the application.** They hold the same reach as the application — every table it can read and write — while skipping the role logic, the maker-checker control and the field masking. The control the auditor tested is not in the path.
- **The credential spreads.** It is copied wherever it is used: config files, scripts, schedulers, a developer's local environment, a backup. Each copy is a full-privilege key to the data, and rotation is avoided precisely because rotation breaks every copy at once.
- **It collapses the identity chain.** The chain from person to data becomes: *a person, somewhere, used a shared account*. Everything the estate's modern systems do to preserve attribution through each hop is lost at the one hop that reaches the data.

This is why the prioritisation in §11 ranks the shared credential first: it is not a single missing control, it is a control-bypass that invalidates a whole column of controls the institution believes it has.

### 9.3 The Middleware Equivalent

The integration tier repeats the pattern with a wider blast radius:

- **The queue manager's service identity** authenticates the flows, not the people or the source applications, and a queue manager with a hub topology reaches every connected endpoint.
- **The integration broker / ESB service identity** is typically a credential with access to many back ends, because the broker's job is to reach them. The broker's own access-control layer (its policies over which flow may call which endpoint) is often configured *after* the fact and often incompletely — the same "integration gap, not a controls gap" the mainframe guide describes ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §1.2 applied one tier up).
- **The consequence** is that one compromised broker identity can issue, for every connected system, exactly the calls the broker is authorised to issue — and on a legacy estate, the broker is usually authorised to issue most of them.

The middleware tier's security features themselves are documented per product — for the MFT products, [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) §5 (security features) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) §9 (security considerations) — and this guide does not re-derive them. What this guide adds is the **identity layer above the product feature**: the broker's service identity is the thing to vault, scope and monitor, and the product's own controls do not decide that.

### 9.4 The Wrapping Pattern — Brokering and Vaulting

Applying §4 to the shared credential:

- **Vault it, do not delete it (yet).** Move the credential into a secrets vault and have every consumer retrieve it from the vault under an accountable identity, rather than read a static copy from a file. This converts "who has the password" into "who can request the secret", which is a governable question. The vaulting-and-brokering pattern is [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6.1.
- **Give each *workload* its own identity.** The single highest-value structural change is to stop one credential serving many consumers: one credential per workload, per application, per flow — the same remedy the mainframe guide specifies for batch and subsystem identities ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.2), applied where no platform model constrains you and the change is therefore *easier*, not harder.
- **Give each *person* their own identity to the data.** Human access through the shared credential is endable today: route analysts and engineers through a broker that authenticates them personally (the identity bridge, [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5) and issues a scoped, logged session, rather than handing them the service account.
- **Scope what the credential can reach.** Independent of brokering, reduce the privilege: the extract account needs read on the reporting schema, not `SELECT` on everything; the batch account needs write on the tables it writes, not `DBA`. Scoping is the control that survives even when the brokering project is delayed.
- **Watch it.** Once the credential is vaulted and brokered, the broker's issuance log is a detection source: unusual time, unusual source, unusual volume, a new consumer — the classic misuse signals, and now attributable.

### 9.5 The Honest Limit

**A broker in front of a shared credential is only as good as the paths it can close.**

If the same credential is also reached by a path the broker does not front — a scheduler on the host, a script on a developer's machine, an application that connects directly, a backup or a replication path, a database listener reachable from a segment the broker does not control — then the broker has changed the *governed* path and left the ungoverned one exactly as it was. The control's effectiveness is the fraction of paths that traverse it, and the honest statement in a design document is not "the credential is now brokered" but "**these paths are brokered; these paths still hold a copy of the credential and are on the schedule to be closed or re-pointed**".

This is the same limit the mainframe guide states for its transfer identity — the account is frequently also the scheduler's, so scoping it is a workload change ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.3, §8.4) — and the same discipline follows: enumerate the paths, broker what you can, scope what you cannot, and record the residue (§12.6).

---

## 10. The File-Transfer Estate

### 10.1 The Transfer Identity

The transfer path is authenticated by an identity, and that identity is, on most estates, **more privileged than it needs to be and shared more widely than anyone intends**. It is the account under which the MFT product, the scheduler, the file-watcher and the partner connection all authenticate; on the mainframe it is the FTP/SFTP/Connect:Direct/Transfer CFT userid, and on the distributed estate it is the equivalent service account. Its properties are the §9 properties — shared, static, over-scoped, copied — with one addition: **it usually has a counterparty on the other end**, so tightening it has an external consequence and is therefore deferred.

The finding is not that transfer identities are unusual; it is that they concentrate two of §3.1's incapacities at once — they cannot join the identity plane (iii) and they are the weakest link in the chain (iv, §10.5). The transfer product's own security features are documented per product and are cross-referenced, not re-derived: [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) §5 (security features), §10 (regulatory compliance) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) §9 (security considerations).

### 10.2 The Directory and Scope Question

The scope of a transfer account — the directories or datasets it can read and write — is the control that determines the blast radius of a compromised transfer identity, and it is the control most often left wide. A transfer account that can reach one partner directory and one landing zone is a bounded exposure; the same account with a home directory on the application host and access to the data root is a full data-access path with a scheduled job attached.

The remedy is the same shape the mainframe guide gives for its own platform — least-privilege, per-flow transfer identities scoped by directory or dataset high-level qualifier ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §8.2) — applied on the distributed estate, where there is no platform class model to work with and the scoping is therefore pure file-permission and application configuration. The transfer-path mechanics for both worlds are [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) and [linux_file_sharing_notification.md](linux_file_sharing_notification.md).

### 10.3 The Protocol and Encryption Question

Every transfer path is a protocol, and the protocol is either encrypted and authenticated or it is neither. The documented protocols of the estate include the file-transfer and messaging paths catalogued in [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) (§2 Connect:Direct, §3 Axway Transfer CFT/PeSIT, §4 FTP/SFTP over USS, §5 MQ file-as-message, §7 Sterling B2B, §10 z/OS Connect, §11 Zowe API ML, §12 Kafka/CDC) and [linux_file_sharing_notification.md](linux_file_sharing_notification.md) §1. The security question in each case is the same three-part question:

1. **Is the path encrypted in transit?** If not — and plain FTP and unencrypted PeSIT still exist on estates because a counterparty requires them — the credential and the payload are exposed to anyone on the path.
2. **Is the endpoint authenticated, and how?** Certificate, key, or password; per-flow or shared, in which case §10.1 applies to the authentication itself.
3. **Is the payload protected at rest at both ends?** Encryption in transit does not protect the landing zone, the staging directory or the archive — the transfer path's endpoints are data stores in their own right, and they inherit the weakest property of both sides.

Where a legacy application cannot be changed to add TLS, the transparent-encryption answer the mainframe estate uses — AT-TLS under policy, so the application is unmodified ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §8.3, verified there against IBM) — has a distributed-estate analogue in TLS termination at the transfer gateway, which is the §4 layer (d) protocol gateway applied to the transfer protocol.

### 10.4 The Scheduling Dependency

Transfers do not run because someone logs in; they run because a scheduler tells them to. The scheduling dependency is the reason transfer tightening is hard, and it is on the mainframe and distributed estates alike: **the scheduler's identity is frequently the same identity as the transfer's**, so the credential that runs the job and the credential that moves the file are one account, and scoping the transfer breaks the batch chain ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §8.4, and §4.3 on the same convergence). The workload-automation side of this — the Control-M integration and its security configuration — is [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) §9.

The architecture note: the scheduling identity is a *privileged* identity and belongs in the §9 brokering discussion, not only in the transfer discussion. Fixing the transfer account while leaving the scheduler holding the same privilege moves the problem rather than solving it.

### 10.5 The Finding — the Transfer Is Usually the Weakest Link

The conclusion this guide draws, and the mainframe guide draws on its own platform: **the file transfer is usually the weakest link in the chain.** The reasoning is structural, and it holds across both estates:

- It is the **integration surface** — it faces counterparties, other platforms and other domains, so it must be reachable where the protected core is not.
- It is **identity-poor** — the identity is the shared static credential of §9, not a person or a per-flow principal.
- It is **scope-wide** — the account reaches directories, not records.
- It is **scheduled and unattended** — no human is watching, and the transfer runs at scale on a timer.
- It is **often the least encrypted path in the estate**, because the counterparty's protocol constrains the answer.

The mainframe guide reaches the same conclusion for its platform and develops the transfer-path remedies at length ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.3 and §8); this guide points there and does not repeat it. The estate-level statement is simply that **on the distributed legacy estate, the transfer path is where the identity problem and the scope problem and the encryption problem meet** — which is why §11 ranks external reachability and shared transfer identity so high, and why §10 is the class most often fixed by *narrowing* rather than by adding a component.

---

## 11. Prioritising Across the Estate

This is the guide's most practical contribution after the class patterns. The mainframe guide's §12.1 phasing answers *what to do within a chosen system* — inventory, identity bridge, privileged access, interactive path, transfer path, audit feed. This section answers a different question: **given six classes and an estate of many systems, which one do you wrap first?** The mainframe guide's phases are what you do once you have chosen; this ordering is how you choose.

### 11.1 Inventory First

**You cannot wrap what you have not inventoried.** This is not a slogan; it is the prerequisite, and it is where most legacy programmes actually fail, because the inventory is unglamorous and the holes it reveals are the finding.

The estate inventory must record, per system:

- **Class and platform** (§2) — which of the six, and what it actually is (release, vendor, support status).
- **Owner** — a named business owner and a named technical owner. A system with no owner cannot be prioritised, because there is nobody to do the work or accept the risk.
- **Entry points** — every network surface by which it is reached: the interactive path, the database wire protocol, file sharing, transfer services, management interfaces, APIs.
- **Identities** — every identity that reaches it: personal, shared, service, vendor, emergency. This is the list that exposes §9.
- **Data classes** — what data the system holds and moves, and which of it is regulated, personal or otherwise sensitive. Concentration of a data class is a prioritisation signal (§11.2 signal 4).
- **Transfer flows** — every inbound and outbound file flow, its counterparty, its protocol, its scope (§10).

The inventory problem in the legacy estate is the **same inventory problem that affects regulatory obligations**: an institution cannot attest to the controls around data it has not enumerated, and a regulator's finding is frequently not "your control was weak" but "you did not know the system existed". The inventory is therefore both the prioritisation input and a compliance artifact in its own right, and it is the reason §11.1 is first rather than a preliminary.

### 11.2 The Five Signals

Once the inventory exists, five signals rank the systems against each other. They are deliberately qualitative — this guide names them as *rankable signals*, not as scores with weights, because assigning weights that look precise to inputs that are not measured would be a fabricated statistic wearing a spreadsheet.

**Signal 1 — A shared or static credential across processes or people.** The presence of one credential used by many consumers is the strongest single signal, because it is a control-bypass (§9.2) rather than a missing control: it invalidates the controls above it. Rank systems by how many distinct consumers reach for the same identity, and by how privileged that identity is. A system where the same account is used by a scheduled job, an application and a human analyst outranks one where the same account is used only by two copies of one job.

**Signal 2 — External or third-party reachability.** A system reachable from outside the enterprise's trust boundary — a counterparty connection, a partner interface, a vendor support channel, an internet-facing transfer — is exposed to identities the enterprise does not control. Rank by the number and nature of external paths, and weight the transfer estate (§10) heavily: it is the class whose whole purpose is to be reachable externally.

**Signal 3 — Direct database or file-level access that bypasses an application's own controls.** The paths of §6.3 and §9.2 — a runtime straight into the database, an OS-level read of the data files, a shared credential used outside the application. Rank by how much of the system's data these paths can reach and how many consumers hold the ability to take them.

**Signal 4 — The concentration of a class of data.** A system that holds a concentrated, sensitive data class — the whole customer master, the whole transaction history, the identity data — outranks a system holding a sample of the same data scattered across many others. Concentration is what determines the *impact* half of the risk equation, and it is readable directly from the inventory's data-class field.

**Signal 5 — The length of the identity chain.** The number of hops between a human identity and the data. A system reached directly by a named person under their own identity has a chain of one; a system reached through a shared credential behind a service account behind an application behind a scheduler has a chain of five, and every hop is a place where attribution is lost. Rank by chain length: longer is worse, because a longer chain means more places where the estate cannot answer "who did this".

### 11.3 The Ordering Method

The method is deliberately simple, and it is defensible because every input is named:

1. **Complete the inventory** (§11.1) for a class or a domain. An inventory that covers three of six classes ranks only three.
2. **Score the systems against the five signals** (§11.2), qualitatively — high / medium / low per signal, with the evidence recorded, not a synthetic number.
3. **Rank by the number of high signals, not by any single signal.** A system high on signal 1 *and* signal 2 *and* signal 5 is the first candidate; a system high on only one is a later candidate. The signals compound — which is the whole point of using several.
4. **Break ties by effort and by class pattern.** Where two systems rank alike, prefer the one whose wrapping pattern is already built for another class (§4 layers are reusable) and the one whose owner is engaged. A system nobody owns ranks last regardless of its signals, because it cannot be wrapped — it can only be escalated.
5. **Re-rank after each cycle.** The inventory changes as systems are wrapped, retired and discovered; a ranking is a snapshot, not a schedule. The discipline is to re-run it, not to defend it.

The output is a **dated, evidence-backed ordering** — which is exactly what a governance forum or a regulator can be shown. What it is not is a score out of 100, because the inputs are not measured quantities and a number would imply a precision the estate does not have.

### 11.4 What This Section Is Not

- **It is not a substitute for the mainframe guide's phasing.** Within a chosen system, the phases are [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §12.1 — inventory, identity bridge, privileged access, interactive path, transfer path, audit feed — and this guide uses those phases unchanged for the systems it wraps. §11 chooses *which system*; the mainframe guide's §12.1 says *what to do to it*.
- **It is not a risk-scoring framework.** It does not quantify likelihood or impact, and it asserts **no figure** — no percentage of the estate, no cost, no count of occurrences. Where the estate-level practice varies by institution, it is recorded as varying (§16).
- **It is not a claim that the first-ranked system is the only urgent one.** Ranking produces an order, not an exclusive; the signals that make a system rank first are also reasons its neighbours need inventory work immediately.

---

## 12. What Wrapping Does Not Fix

The mainframe guide ends its control discussion with what genuinely cannot be changed and the residual risk that must be owned ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §11). This section **extends** that treatment rather than repeating it: it names the things that wrapping, as a boundary technique, structurally cannot fix — and it ends on the same ownership requirement, applied to the estate rather than to one platform.

### 12.1 Boundary Versus System

The first limit is inherent to the technique: **wrapping protects the boundary, not the system.** Every layer of §4 sits *in front of* the platform. That is what makes it deployable without changing the platform — and it is also what it cannot do.

- A wrapped system is protected **against access that arrives through the boundary**. It is not protected against access that reaches it another way — the direct path the boundary did not close, the peer-to-peer route, the management interface, the backup network.
- A wrapped system's **own weaknesses remain**. If the platform has an unpatched vulnerability exploitable by a process that is already on it, the boundary being present changes nothing about that.
- **The boundary's coverage is a fraction, and it should be measured as one.** The honest design statement is not "this system is wrapped" but "these paths are wrapped; these paths remain, and here is why they are on the accepted list or the closure list". This is the same limit §9.5 states for a brokered credential, generalised: a control is only as good as the paths that traverse it.

The corrective is not to abandon wrapping — it is to stop describing it as protection of the system. Wrapping is protection of the *approach* to the system; the system's own state is a separate question, and on the agentless classes (§7) it is a question that cannot be fully answered at all.

### 12.2 The Insider With Granted Authority

The second limit is about *who* the boundary is guarding against. Zero trust is often summarised as "never trust, always verify" — but a boundary that verifies a person and then admits them to a system whose own authorisation grants them broad access has verified the person and trusted the grant. Wrapping does not fix the **insider whose access is legitimately granted**.

Concretely, on the legacy estate:

- A DBA with the shared database credential (§9) is *authorised* to hold it. Wrapping the credential (vaulting, brokering, scoping) reduces the number of holders and makes the access attributable — it does not make the DBA's legitimate access illegitimate, and it does not stop a holder from using what they are granted.
- An engineer who can reach the account directory of a MultiValue database (§6.3) is doing their job. The control that matters is not "deny the engineer" but "know which engineers, under what authority, with what trace" — attribution, least privilege and review.
- The **inherited authority of a job or an application** (adopted authority on the midrange, §5.2) means the *effective* access may exceed what any person was granted, because a program is using the owner's authority. Wrapping the front door does not change what the program can do behind it.

What wrapping *does* fix, for this case, is the **unattributable** and the **over-scoped** part: it converts "somebody with the credential" into "a named holder, in a brokered session, with a scope", and the audit trail that makes the insider's action reviewable. That is the real contribution, and it is a detective and governance contribution as much as a preventive one — which is exactly why §9.4 ranks scoping and monitoring alongside brokering.

### 12.3 The In-Application Vulnerability

The third limit: **an application vulnerability that legitimately reaches the platform is not stopped by a boundary that admits the application.**

If the application tier is authorised to call the database, and the application has an injection flaw, a broken access-control check, an IDOR or a deserialisation bug, then the attacker arrives through a *legitimately admitted* path. The boundary sees an authorised application doing ordinary-looking work. It is the application's own defect that turns an admitted call into unauthorised data access — and on the legacy estate the application layer is frequently the *only* control (§6.2), so an application defect there has nothing behind it to catch it.

The correct framing for the estate-level design:

- **The boundary is not a substitute for application security, and application security is not a substitute for the boundary.** They cover different threats and neither subsumes the other.
- **The threat model has to be drawn per trust zone.** This is exactly the discipline [threat_modeling_guide.md](threat_modeling_guide.md) establishes: its §3.6/§9 draw core-banking hosts as **external entities outside the model** — the DFD-per-trust-zone rule — precisely because a host the analyst cannot decompose (its internals unmodelled, its vendor gone) must be treated as an external entity across a trust boundary rather than as a component inside one. That rule is why the wrapped legacy estate appears in a DFD as a boundary-to-external-entity edge, and why the application in front of it must be modelled as a *separate* zone whose own flaws are assessed on their own account. This guide cross-references that method rather than re-deriving it.
- **The application-tier proxy of §6.4 is not a WAF.** It enforces identity and scope at the tier; it does not fix the application's input handling. Where the application is the only control, application testing and remediation is the actual fix, and it belongs to the application, not to the wrapping project.

### 12.4 The Inherited-Control Problem

The fourth limit is the one the MultiValue class surfaced in §6.2: **a class whose security is inherited from an unpatched lower layer has a control whose owner is not the platform.**

The chain of dependency is:

> The application's control depends on the database's account model → the database's account model depends on the host OS's file permissions → the host OS's file permissions depend on the host OS being patched and configured → the host OS cannot be patched (incapacity (v), §3.1).

Every link is real, and the top of the chain *looks* like a control while its foundation is the weakest thing in the estate. Wrapping does not fix this, because the fix is either to **patch a layer that cannot be patched** (impossible) or to **move the control up** so it no longer depends on the unpatched layer (the §6.4 application-tier broker — a real remedy, and a project). What wrapping *can* do is ensure the inherited control is **known to be inherited**: named as inherited in the inventory, with its dependency on the unpatched layer explicit, so that the residual risk register records the dependency rather than a false assurance.

This is the same distinction the mainframe guide draws between a control that exists and a control that is *activated and scoped* ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §11.1) — extended here to the case where the control is not merely unactivated but **owned by a different, unmanaged layer**.

### 12.5 Replace Rather Than Wrap

The fifth limit is the decision this guide forces explicitly: **some systems should be replaced, not wrapped.** Wrapping is the right answer for a system that must stay and cannot be changed; it is the wrong answer for a system whose wrapping cost approaches its replacement cost, or whose continued existence imposes a permanent tax on the estate.

The integration-side treatment of this decision is [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) — its §7 (The Modernization Patterns: the strangle-fig, the abstraction layer, the incremental replacement and their relatives) and its §10 conclusion, "Integrate, Don't Replace". **Neither is re-derived here.** What this guide adds is the *security-side* criterion, stated as questions a design review can apply:

1. **Is the platform able to be the PEP at all, now or after any realistic change?** If not, the wrapping is permanent, and its total cost is the wrapping plus its permanent residual risk (§12.6).
2. **Is the class's security inherited from a layer that cannot be patched?** (§12.4) If so, the estate is carrying a control it does not own, indefinitely.
3. **Does the system hold a concentrated sensitive data class** (signal 4, §11.2) that would be better placed on a platform the enterprise controls end to end? Concentration plus un-patchability plus inherited controls is the combination that justifies replacement.
4. **Is the shared credential the only reason the data is reachable?** (§9) If the whole system exists to serve a reporting extract that could be produced by a modern platform reading a controlled feed, the wrap is solving a problem that removal would eliminate.
5. **What is the movement trigger?** (§12.6) If the answer to "what would let us eliminate this exposure?" is "replacing the system", then the replacement is on the register as the movement, and the wrapping is a bridge to it rather than a destination.

The default posture of this guide is **wrap, because wrapping is deployable now and the estate cannot be replaced at once** — and the explicit exception is the system where the five questions above all point the same way. §13.5 works one such case through.

### 12.6 The Residual Risk That Must Be Owned

Wrapping produces, in every class, some exposure it cannot close. The requirement for those is the one the mainframe guide specifies in full — [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) **§11.2** (a description, a **named accountable owner** — a *role*, not a team, a compensating control with its limit stated, a **review date**, and a **written acceptance**) and **§11.3** (the register, whose **movement** field records the condition under which the exposure can be closed). **That requirement is not restated here; it is adopted unchanged.**

What this guide adds, because the estate is many systems and not one, is the **estate-level roll-up**:

- **The same register carries every class.** A single residual-risk register, with a class field, so that the exposures of the midrange, the MultiValue database, the agentless appliance, the OT estate, the shared credential and the transfer path are visible together — because their *combined* exposure is what a counterparty or a supervisor sees, not the per-system view.
- **Ownership rolls to the same accountable roles as the estate inventory.** Every system in the inventory has a named business owner and technical owner (§11.1); the residual for that system is owned by the same role, so that the register and the inventory cannot drift apart.
- **The register's movement field is where the estate's roadmap lives.** If the movement for a residual is "close the direct path", that is a project; if it is "replace the system" (§12.5), that is a roadmap item; if it is "the counterparty changes protocol", that is a contract dependency with a name against it. A register without movements is a graveyard, and the movement column is what keeps the estate's accepted risk *tracked toward elimination* rather than merely tolerated.
- **The compounding discipline.** No system's residual is reviewed in isolation: a system that is high on signals 1, 2 and 5 of §11.2 but has an accepted residual is a different risk from a system whose residual sits on a low-signal system. The register is reviewed as an estate, not as a per-system paper exercise.

The one sentence to carry from the mainframe guide into every class of this guide, unchanged: **a residual risk that is named is a managed risk; the same risk unstated is the finding.**

---

## 13. The Cymbal Bank Worked Example

*Cymbal Bank is fictional and used here illustratively. The platforms, vendors and standards named are real subject matter; nothing in this section asserts that any real bank, vendor or service provider uses, deploys or is a client of anything.* Cymbal Bank is the only bank persona in this guide.

### 13.1 The Estate and the Inventory

Cymbal Bank's legacy-estate inventory (§11.1) surfaces three systems, among others, for the first wrapping cycle:

1. **A core-adjacent AS/400 (IBM i)** — call it `MID-CORE-1`. It runs a loan-servicing and general-ledger workload feeding the bank's main core. It is at an older, frozen OS release; it has its own user-profile directory, a set of operator profiles used by several branch-ops staff (a shared interactive identity), and a service profile that the batch jobs and an integration job use. Its data is reached by the 5250 path, by an SQL/DRDA connection from a distributed reporting application, and by nightly extracts.
2. **A MultiValue reporting database** — call it `MV-REPORT-1` — a UniVerse-based reporting and reconciliation store, populated by nightly extracts from the core. It is accessed through a thin application that runs in the bank's application tier, and directly by two analysts and one vendor tool that connect straight to the account. Its files sit on a host whose OS is out of support.
3. **A file-transfer path** — call it `MFT-BUREAU-1` — the nightly exchange with a settlement bureau and two internal feeds, running through the bank's managed file transfer estate on a schedule, under a single transfer identity that also runs its own Control-M job.

The inventory also records, as its own line, the **shared database credential** the reporting application and the analysts both use (`§9.1`), and the **transfer identity** `MFT-BUREAU-1` uses (`§10.1`).

### 13.2 The Prioritisation

Applying the five signals of §11.2 to the three systems and their two shared credentials:

| Item | S1 shared credential | S2 external reachability | S3 direct bypass path | S4 data concentration | S5 identity-chain length | Rank |
|---|---|---|---|---|---|---|
| `MFT-BUREAU-1` transfer identity | **High** — one identity for bureau, feeds and the scheduler | **High** — faces the bureau and the bank's boundary | Medium — account can reach a broad landing area | Medium — settlement data | High — scheduler → transfer → partner | **1** |
| Shared reporting DB credential (§9) | **High** — application + analysts + vendor tool | Low | **High** — direct connection and OS-level file read | **High** — full reporting copy of core data | High — person → app/vendor → account → DB → files | **2** |
| `MID-CORE-1` (IBM i) | High — shared operator and service profiles | Medium — DRDA and extract path | Medium — SQL/DRDA direct | **High** — loan and ledger data | High — person → emulator → shared profile | **3** |
| `MV-REPORT-1` (as a platform) | Medium — its own access is via the shared credential above | Low | **High** — direct analyst/vendor connection, host file read | High | Medium | folded into item 2 |

**Why the transfer identity and the shared credential rank first, ahead of the IBM i core-adjacent system** — and the reasoning is the guide's own, not a preference:

- Both are **control-bypasses, not missing controls** (§9.2, §11.2 signal 1). The shared reporting credential invalidates the reporting application's role logic, masking and audit; the shared transfer identity invalidates the scoping and segregation the MFT estate believes it has. Ranking them first is ranking the things that make *other* controls untrue.
- Both are **shared across processes and people** — the condition zero trust cannot tolerate, in the mainframe guide's phrase, and the strongest single signal.
- The transfer identity additionally faces **outside the boundary** (signal 2) and is **scheduled and unattended** (§10.5), which is being the weakest link in the chain exactly as the mainframe guide finds on its own platform.
- `MID-CORE-1` ranks third **not because it matters less** — it holds the concentrated data (signal 4, high) — but because its shared identities are the *same problem in a platform that also has a real authorisation model* (§5.2) and an audit journal (§5.2), so its remediation has somewhere to land. The two credentials rank first because there is no platform model beneath them at all.

### 13.3 The Wrapping Pattern Applied

Cymbal applies the six layers of §4 to each item, **pointing at the mainframe guide's mechanisms rather than re-inventing them**, and noting where the class shape differs:

**`MFT-BUREAU-1` (the transfer identity).** The wrapping is §4 layers (b), (c), (e), (f) — and it is deliberately *narrowing-first* (§10.5):
- *Per-flow identities* (§9.4, mirroring [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.2/§8.2): each bureau flow, each internal feed and the scheduler get **separate** identities with directory-scoped reach, breaking the single-identity convergence the mainframe guide identifies between the transfer account and the scheduler's ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §4.3, §8.4).
- *Vaulting and brokering* ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6): the transfer identities move into the vault and are issued to the MFT product under an accountable automation identity, not read from configuration.
- *Protocol and encryption* (§10.3): the encrypted protocol is required where the counterparty accepts it, and where the bureau's protocol constrains the answer, the residual is recorded **with the bureau named as the dependency** (§12.6 movement).
- *Observation*: the transfer path's flows are captured at the boundary ([network_packet_capture_guide.md](network_packet_capture_guide.md)) because the MFT product's own records reach the SIEM only partially ⚠.

**The shared reporting credential (§9).** The wrapping is §4 layers (b), (c), (d):
- *Vault and broker* (§9.4, [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6.1): the credential moves to the vault; the application retrieves it under an accountable identity; the analysts are routed through a broker that authenticates them **personally** (identity bridge, [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5) and issues a scoped, logged session, so human access through the shared account ends first.
- *Per-workload identities* (§9.4): the application, the extract and each reporting consumer stop sharing one credential.
- *Scope* (§9.4): the credential's reach is reduced to the reporting schema it needs, independent of the brokering project's timeline.
- *The honest limit stated* (§9.5): Cymbal **enumerates the paths that still hold a copy** — the vendor tool's configuration and the analyst workstations — and records them as paths to close or re-point, rather than declaring the credential brokered.

**`MID-CORE-1` (the IBM i).** The wrapping is the midrange pattern of §5.5, which is the mainframe shape:
- *Identity-aware entry point in front of 5250* (§4 layer (a); [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.2), ending the shared operator profile on the interactive path, with session recording scoped as a detective control and its proof-limits adopted from [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §7.4.
- *Identity bridge* (§4 layer (b); [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5) to provision the bank's personal identities as user profiles and deprovision on the enterprise's leaver event.
- *Brokered privileged access* (§4 layer (c); [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §6) for the `*ALLOBJ`-class and service profiles.
- *Object-authority hygiene as a parallel workstream* (§5.2) — the `*PUBLIC` and authorization-list review the platform's own SQL services make possible (`QSYS2.OBJECT_PRIVILEGES`, `QSYS2.AUTHORIZATION_LIST_USER_INFO`) — because the platform has a real authorisation model and leaving it exposed while wrapping the front door would repeat anti-pattern 1 (§14).
- *Audit feed* (§4 layer (b)/[zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §10.2): `QAUDJRN` collected into the SIEM — the same collection-gap project, on a different platform.

**`MV-REPORT-1` (the MultiValue database).** The wrapping is §6.4, and it is *mostly host hardening plus application-tier brokering* because the class has no database-level control to delegate to:
- The application tier becomes the enforcement point and the only path to the data; the direct analyst and vendor-tool connections are closed (§6.3, §6.4).
- The host's OS is the hard case: **it is out of support** (§7), so Cymbal treats the host as an **agentless-class system** — network-restricted, brokered, protocol-inspected and observed at the boundary — and states the §7.4 limit honestly rather than claiming host-level visibility it does not have.
- The OS-level file read of the hashed files is closed by host permissions **and** by removing the accounts that could take it — and because the host cannot be patched, that control is recorded as **inherited** (§12.4), with the unpatched layer named as its dependency.

### 13.4 The Auditor Question

Cymbal's auditors ask about `MV-REPORT-1`'s host: **"You cannot install a security agent on this system. What is your monitoring?"** The answer follows §7.5, in four parts, and none of them is "we monitor it":

1. **Class and reason.** "This host is in the agentless class: the OS is out of support, so no supported agent build exists. It is inventoried as such."
2. **Actual controls.** "Only the application tier and two brokered administrative identities can reach it; the direct database and vendor-tool paths were closed; access is brokered and recorded; and its flows are captured at the boundary into the SOC."
3. **The limit, volunteered.** "We do not have host-level process or file visibility, we cannot block a local process, and we do not attest its patch state. Network detection is the visibility we have, and it does not see inside the host."
4. **Ownership.** "It is on the residual-risk register as an agentless-class exposure, owned by the Head of Core Platforms, with a review date and a signed acceptance; its movement is the replacement decision in §13.5."

The disclosure is what the auditor is actually testing for. The control gap is registered and owned; the *undisclosed* gap would have been the finding.

### 13.5 The One System Cymbal Replaces Rather Than Wraps

Cymbal runs the §12.5 questions against each item. For `MFT-BUREAU-1` and `MID-CORE-1`, the answers point to **wrap** — both must stay, both can be wrapped usefully, and the wrapping is a bridge with a movement. For the MultiValue reporting database, `MV-REPORT-1`, **every one of the five questions points the same way**:

1. **Can the platform be the PEP, now or after any realistic change?** No — it is a reporting store with no policy layer and no vendor-supported path to one, and its host cannot be patched.
2. **Is its security inherited from an un-patchable layer?** Yes — entirely (§6.2, §12.4): host file permissions on an out-of-support OS, plus application logic, with nothing platform-side beneath them.
3. **Does it hold a concentrated sensitive data class?** Yes — a complete reporting copy of core data, which is signal 4 at its worst: a *duplicate* of the concentrated data class, in a *less* controlled place than the original.
4. **Is the shared credential the only reason the data is reachable?** Effectively yes — the store exists to serve reporting consumers who reach it through one shared account and two direct connections; strip the credential and what remains is a workload that a controlled feed could serve.
5. **What is the movement?** Replacement.

**Cymbal replaces rather than wraps `MV-REPORT-1`.** The reasoning, stated as it would be to the governance forum: the store is a *duplicate* of core data sitting on an *un-patchable* host behind a *shared credential*, reached by a *direct path* the application cannot see — so wrapping it means permanent compensating controls and a permanent residual, to protect a second copy of data the bank already protects at the source. Rebuilding the reporting workload on a platform the bank controls end to end removes the class from the estate instead of governing it forever. The replacement is the **movement** recorded against the residual — the register's movement field doing its job (§12.6) — and the wrapping of the *other* two systems proceeds in parallel, because they are bridges to live systems and not candidates for removal.

The decision rule it illustrates, in one line: **wrap what must stay and cannot change; replace the system whose entire control story is inherited from a layer you cannot fix, and whose data you already hold somewhere better.**

Note what Cymbal does **not** do: it does not describe the reporting store as an incompetent system, and it does not describe the analysts who used the direct path as a problem. The subject is architecture — a store whose controls were inherited from a layer that stopped being maintainable — not the people who ran it. The replacement is a consequence of the dependency, not an indictment.

### 13.6 The Thesis, Restated

Cymbal's cycle is the guide in miniature. Nothing was fixed *inside* a legacy platform: the IBM i's own authorisation model was left to do its job and was joined to the enterprise's identity and audit planes from outside; the transfer identities were narrowed and vaulted from outside; the shared credential was brokered and scoped from outside; the reporting store was removed rather than wrapped, precisely because there was nothing inside it to build on. The trust decisions that the estate needed were made **in front of** the systems, or the systems were retired.

That is the whole architecture, and it is why the guide closes where it opened.

---

## 14. The Anti-Patterns

Each anti-pattern is stated as **symptom → cause → guardrail**, so a review can use it as a checklist. Every guardrail points at the section that develops the fix.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **Wrapping the outside while leaving the shared credential inside.** | A gateway, a broker or a segment is deployed in front of the platform; the shared database or transfer credential remains, and attribution is unchanged | Sequencing the boundary before the identity work, and treating "deployed a component" as "changed who can do what" | Do the identity work with the boundary, not after it: per-workload identities and vaulting (§9.4); a per-flow transfer identity (§10.1); measure the control by whether *who can do what* changed |
| **2** | **Treating an agentless system as monitored because a dashboard shows it.** | A console tile is green; the system's host state, process activity and file access are not actually observed | Network visibility presented as host visibility; the §7.4 gap smoothed over | State the compensating-control limit explicitly (§7.4) and the auditor answer (§7.5); put the residual on the register (§12.6) |
| **3** | **Importing the IT patch cadence into an OT estate.** | A patch or control is scheduled on the enterprise's cycle and either never lands or trips the plant | IT-security instinct applied without the inverted priority (§8.1) | Safety and availability first; place the enforcement point where it cannot interrupt the process (§8.3); use [scada_guide.md](scada_guide.md) §6 §8 for the OT models and standards |
| **4** | **Assuming a platform has a modern security model because it is still supported.** | A system's controls are taken on trust; the actual model is never established from the vendor's documentation | Familiarity with the brand substituted for evidence about the model | Establish each class's security model at source (§5.2 for the midrange; §6.2 for MultiValue) and be willing to conclude a class's security is **inherited** (§12.4) |
| **5** | **Wrapping a system that should be replaced.** | Permanent compensating controls and a permanent residual accrete around a system whose removal would be cleaner | Wrapping treated as the only option; the replacement criterion never applied | Run the §12.5 five questions; cross-ref [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §7 §10; record replacement as the register's movement (§12.6) |
| **6** | **Declaring residual risk without an owner.** | The risk exists in practice but in no register; nobody has signed anything | Exposure never classified, or classified and then left unassigned | [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §11.2–§11.3: description, **named role**, compensating control with its limit, review date, **written acceptance**, and a movement (§12.6) |

The thread through all six is the same thread the mainframe guide finds on its own platform: **the failure is almost never a missing component — it is a missing owner and a missing inventory.** The legacy estate can be wrapped; what it cannot do is decide that somebody in the enterprise is accountable for connecting it.

---

## 15. The Claims Audit

Verified this pass = checked against the source named, on **1 October 2026**, unless a source date is given. The convention follows the repo: ✅ verified, ⚠ flagged/partial/practice-varies, ❌ could not be verified at source (see §16). Every platform component name in this guide is traced below; where a name could not be traced at source this pass, it is flagged rather than smoothed over.

| Claim | Status | Source | Source date | Quality |
|---|---|---|---|---|
| IBM i object authority includes the system-defined sets `*ALL`, `*CHANGE`, `*USE`, and `*EXCLUDE` ("different than having no authority") | ✅ | IBM i documentation, "Commonly used authorities" (ibm.com/docs/en/i/7.6.0) | Current | Vendor primary, quoted |
| Object authority is also managed through **authorization lists** (`*AUTL`), with `*PUBLIC` authority sourced from the list; `*PUBLIC` is "the authority for an object granted to all users" | ✅ | IBM i documentation, "Monitoring public authority to objects for IBM i" (ibm.com/docs/en/i/7.1.0); practitioner review queries using IBM i Services | Current | Vendor primary + practitioner |
| Adopted authority: a program's `*USER` attribute of `*OWNER` makes it run with the program owner's authority (`use_adopted_authority` in `QSYS2.PROGRAM_INFO`); standard practice is that programs adopt authority except the initial program | ✅ | IBM i Services query patterns and the object-authority review pattern (MC Press Online, excerpted from *IBM i Security Administration and Compliance*) | Current article; IBM i Services current | Vendor-documented object model + established practitioner source |
| The review of `*PUBLIC` authority, authorization lists, ownership and private authorities is exposed through IBM i SQL services `QSYS2.OBJECT_PRIVILEGES`, `QSYS2.AUTHORIZATION_LIST_USER_INFO`, `QSYS2.PROGRAM_INFO`, `QSYS2.IFS_OBJECT_STATISTICS` | ✅ | Same practitioner source, using the IBM i Services catalog | Current | Vendor service names, practitioner usage |
| The IBM i **security audit journal** is `QAUDJRN` and is "the primary source of auditing information about the system" | ✅ | IBM i documentation, "Using the security audit journal" (ibm.com/docs/en/i/7.4.0) | Current | Vendor primary, quoted verbatim |
| `DSPAUDJRNE` is documented but IBM "has stopped providing enhancements" for it and it "does not support all security audit record types"; IBM recommends `RCVJRNE` on `QAUDJRN` | ✅ | IBM i documentation, "Analyzing audit journal entries" (ibm.com/docs/en/i/7.5.0) | Current | Vendor primary, quoted verbatim |
| Midrange security is managed through **system values** (including `QSECURITY`) and **user profiles** alongside object security, as the platform's three core aspects | ✅ (three aspects) / ⚠ (individual system values) | Practitioner source (*Mastering IBM i Security* excerpt) | Current article | Established practitioner; **individual `QSECURITY`/`QAUDLVL`/`QAUDCTL` option behaviour is release-specific and must be re-checked** |
| IBM i special authorities include `*ALLOBJ`, `*SECADM`, `*JOBCTL`, `*SPLCTL`, `*AUDIT`, `*SAVSYS` | ⚠ | Widely documented IBM i security material; not re-verified item-by-item this pass | — | ⚠-structural: names are standard but the full list is release-specific — verify before a design asserts one |
| The midrange security commands `CRTUSRPRF`, `CHGUSRPRF`, `WRKUSRPRF`, `DSPUSRPRF`, `GRTOBJAUT`, `RVKOBJAUT`, `EDTOBJAUT`, `DSPOBJAUT`, `CHGAUT`, `DSPAUT`, `WRKAUT`, `DSPJRN`, `RCVJRNE` | ⚠ | Standard IBM i command names; the authority-value and journal claims above rest on IBM documentation, but the full command list was not individually re-verified at source this pass | — | ⚠-structural: verify against IBM i command documentation before a change references one |
| The 5250 data stream and 5250 emulation (e.g. IBM i Access Client Solutions), and the SQL/DRDA, NetServer (SMB/CIFS) and FTP entry surfaces | ⚠ | Structural knowledge of the platform; entry surfaces not re-verified at source this pass | — | ⚠-structural: verify availability/support per release before a design depends on one |
| Directory/Kerberos integration exists as an option for IBM i, i.e. it is a project and not a platform property | ⚠ | Not re-verified at source this pass | — | ⚠: product and release detail must be verified against IBM before a design asserts it |
| In UniVerse, "in any UniVerse account, the VOC file, all file dictionaries, and all data files are stored at the same level — that is, in the UNIX or Windows directory where the UniVerse account resides" | ✅ | Rocket Software, *Rocket UniVerse Guide for Pick Users*, V11.3.5 (docs-be.rocketsoftware.com) | v11.3.5 (current) | Vendor primary, quoted verbatim |
| UniVerse remote file access runs through **UVNet**: the `unirpcd` daemon (or the `unirpc` service on Windows) gated by the `unirpcservices` file, which starts a per-user `uvnetd` daemon; the documentation's chapter is "Remote file permissions" | ✅ | Rocket Software, *Rocket UniVerse UVNet User Guide*, V11.3.5, January 2023 | Jan 2023 | Vendor primary; component names quoted |
| jBASE account model: the `SYSTEM` file (via `JEDIFILENAME_SYSTEM`) holds one record per account; field 2 is the absolute account **path**; field 7 is an **encrypted password** maintained "only through the jBASE `PASSWORD` command"; `LOGTO` switches accounts; `JEDIFILENAME_MD`/`JEDIFILEPATH` locate the MD/VOC and files | ✅ | Zumasys jBASE documentation, "SYSTEM file entries" (jBASE 3.0 manual set) | jBASE 3.0 manual (historic; names still current in the family) | Vendor primary, quoted; **release-specific — verify against the current jBASE release** |
| **Conclusion:** the MultiValue family's access control is largely *inherited* from the host OS's file permissions and the application layer, not enforced by a database-level privilege engine comparable to an RDBMS `GRANT` model | ⚠ | This guide's assessment, supported by the UniVerse and jBASE filings above and the absence of a verified native per-record authorisation model | — | ⚠: an architectural assessment, stated as such — **not** a vendor capability claim |
| UniData follows the account-based, host-file-system model, with a native per-record authorisation model **not verified** | ❌ | UniData administration guide exists (Rocket) but was not extracted this pass; no primary source confirmed a UniData privilege model | — | ❌: recorded in §16; do not assert a UniData security mechanism without vendor verification |
| OT standards **NIST SP 800-82r3**, **ISA/IEC 62443** (zones and conduits), and **NERC CIP** as the OT security frameworks, and the **Purdue model** for OT segmentation | ⚠ | Carried from the sibling ledger [scada_guide.md](scada_guide.md) §4 and §6; **not re-verified this pass** | — | ⚠: cross-referenced, not re-derived or re-verified |
| The agentless detection that NDR provides is "the only visibility for the legacy-core estate where endpoint agents cannot be installed" | ⚠ | [secops_guide.md](secops_guide.md) §3 (sibling ledger) | — | ⚠: cross-referenced, not re-verified |
| Core-banking hosts are drawn as **external entities outside the model** — the DFD-per-trust-zone rule | ⚠ | [threat_modeling_guide.md](threat_modeling_guide.md) §3.6, §9 (sibling ledger) | — | ⚠: cross-referenced, not re-verified |
| Modernisation patterns (strangle-fig, abstraction, incremental replacement) and "Integrate, Don't Replace" | ⚠ | [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §7, §10 (sibling) | — | ⚠: cross-referenced, not re-derived |
| MFT product security features and the Control-M integration's security considerations | ⚠ | [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) §5, §10; [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) §9 (siblings) | — | ⚠-structural: **no capability claim beyond the sibling's ledger** |
| **"Agentless coverage is weaker than agented coverage"**, and the specific observability gap | ⚠ | This guide's reasoning from what each control class can observe — stated as an argument, not sourced to a measurement | — | ⚠: an architectural argument; **no measurement or figure is asserted** |
| Prevalence of shared credentials, shared transfer identities, or direct-access paths across legacy estates | ⚠ | **Not sourced** — stated as the common practitioner pattern, with the explicit note that **practice varies by institution and by estate era**, and that **no figure is asserted anywhere in this guide** | — | Deliberately unquantified; see §16 |
| Regulatory framing (DORA applicability, MAS TRM / Notice 645, SWIFT CSP) where a jurisdiction's obligation bears on the estate | ⚠ | Regulator/scheme material — established in sibling guides; **not re-verified this pass** | — | ⚠: cross-referenced, not re-derived |

**The honesty note.** Every **platform component name** in this guide — the IBM i authority values (`*ALL`, `*CHANGE`, `*USE`, `*EXCLUDE`, `*AUTL`), `*PUBLIC`, adopted authority, the authorization list, `QAUDJRN`, `DSPAUDJRNE`/`RCVJRNE`, the IBM i SQL services, and the jBASE `SYSTEM` file fields, the `PASSWORD`/`LOGTO` commands, `unirpcd`/`uvnetd`/`unirpcservices` and the UniVerse account/VOC model — is traceable above, with its source and date. The claims marked ⚠ are exactly the ones where the component name is standard but was not re-verified this pass, where individual option/command behaviour is release-specific, where a class's security model is an **assessment** rather than a vendor statement, or where a prevalence statement cannot honestly be quantified. The one ❌ is the UniData authorisation question, recorded in §16 rather than papered over.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

The following could not be confirmed against a primary source during this pass, and are recorded honestly rather than asserted:

- **`web_extract` could not read IBM's documentation pages** (`ibm.com/docs/en/i/...`) during this pass — every attempt returned a scraper failure rather than page content. This is a **tool limitation on this host**, not evidence that the material is absent. The IBM i component claims in this guide therefore rest on IBM documentation that the search index surfaced (page titles and quoted snippets — e.g. "Commonly used authorities", "Monitoring public authority to objects for IBM i", "Using the security audit journal", "Analyzing audit journal entries") plus an established practitioner source that uses IBM's documented object model and IBM i Services. A reader taking a change decision should **re-open those IBM pages directly** and confirm the release-specific detail.
- **The individual IBM i special-authority list and the individual security system values** (`QSECURITY`, `QAUDCTL`, `QAUDLVL`) — the names are standard and were surfaced, but their **option-by-option behaviour is release-specific** and was not re-verified this pass (⚠). Do not name a specific system value in a change without checking IBM's current documentation.
- **The IBM i security command list** (`CRTUSRPRF`, `GRTOBJAUT`, `RVKOBJAUT`, `EDTOBJAUT`, `DSPOBJAUT`, `CHGAUT`, `DSPAUT`, `WRKAUT`, `DSPJRN`, `RCVJRNE`) — surfaced as standard command names, not individually re-verified at source this pass (⚠).
- **IBM i directory/Kerberos integration and endpoint-agent availability by release** — the *existence* of directory integration as a project and the release-dependence of agent availability are stated as ⚠; their product and release detail was not verified, and must be checked against IBM before a design depends on them.
- **UniData's security model** — ❌. A Rocket UniData administration guide exists, but this pass did not extract it and **no primary source confirmed a native UniData per-record authorisation model**. This guide therefore asserts **no** UniData security mechanism beyond the family-level structural facts (an account-based, host-file-system-backed model) and records the gap here. The UniData claim should not be repeated without vendor verification.
- **The MultiValue "inherited security" conclusion** — ⚠ by design. It is this guide's **architectural assessment** supported by the UniVerse and jBASE filings, not a vendor statement that the products have no security model. Individual installations may have configured compensating controls this guide does not know about; practice varies.
- **OT standards detail** (NIST SP 800-82r3 revision, ISA/IEC 62443 zone/conduit specifics, NERC CIP applicability) — carried from [scada_guide.md](scada_guide.md) §4/§6 and **not re-verified this pass** (⚠). The OT estate and its segmentation model are cross-referenced, not re-derived.
- **MFT product security capabilities** — the products named ([axway_transfer_cft_guide.md](axway_transfer_cft_guide.md), [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md)) are named factually as the subject matter, and this guide makes **no capability claim beyond those siblings' ledgers** and asserts **no institution's deployment** (⚠-structural).
- **Prevalence across the industry of shared credentials, shared transfer identities, direct database access and agentless hosts** — **no figure is asserted anywhere in this guide**, by design. The findings are stated as the common practitioner pattern with the explicit qualification that *practice varies by institution and by estate era* (⚠, unsourced by intent). No share-of-estate, cost or count is given.
- **Regulatory specifics** (DORA requirements and dates; MAS TRM / Notice 645 provisions; SWIFT CSP control wording) — carried from the sibling guides' established ledgers and cross-referenced, not re-derived or re-verified this pass (⚠).
- **The comparison "agentless is weaker than agented"** — stated as an architectural argument about what each kind of control can observe, not as a measured result (⚠). No measurement is claimed.

### 16.2 The Glossary

- **Agentless posture** — the state of a system on which the enterprise's endpoint agent cannot be installed (end-of-life OS, immutable appliance image, certification constraint), so detection must come from network, protocol or log observation (§7).
- **Boundary** — the set of components placed in front of a legacy platform that make the trust decision the platform cannot make for itself: entry point, identity bridge, brokering, protocol gateway, network path, observation point (§4).
- **Class (legacy class)** — a group of platforms sharing a deficit profile and therefore a wrapping pattern; the unit of design in this guide (§1.3, §2).
- **Control bypass** — an exposure that does not merely lack a control but **invalidates controls above it**; the shared database credential and the shared transfer identity are the two canonical cases (§9.2, §11.2).
- **Enforcement point** — the component that admits or denies a session; in every class here it sits **in front of** the platform (§1.3).
- **Estate inventory** — the authoritative, dated list of platforms, versions, owners, entry points, identities, data classes and transfer flows; the prerequisite for prioritisation (§11.1).
- **Identity bridge** — the mechanisms mapping enterprise identities to platform identities and streaming platform records outward; developed in [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5.
- **Inherited control** — a security property a class borrows from the layer beneath it (an unpatched host OS's file permissions, an application's own role logic) rather than one it enforces itself (§6.2, §12.4).
- **Joiner-mover-leaver** — the enterprise identity lifecycle; the reason the identity bridge is first among the layers ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §5.4, §5.6).
- **PEP (policy-enforcement point)** — the component that enforces the enterprise's access policy; the thing a legacy platform cannot be (§1.2, §1.3).
- **Protocol gateway** — a component that terminates a legacy protocol on the enterprise side, applies policy, and re-originates the session inward (§4 layer (d)).
- **Shared credential** — one credential used by many processes and many people to reach data, bypassing the application's own controls; the largest unwrapped hole in most estates (§9).
- **Wrapping** — the practice of building the trust decision in front of a platform rather than inside it; the subject of this guide (§4).
- **Movement (register field)** — the condition under which an accepted residual risk can be closed (a workload modernisation, a replacement, a contract change); what keeps the residual register from becoming a graveyard ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) §11.3, §12.6 here).

### 16.3 The Cross-References

Because this guide is the estate-wide branch of the security cluster, the reader's next stop depends on the question:

- **The mainframe constrained case in full** — [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md). This guide's pattern source: §1.2 (the system that cannot be a PEP), §4 (the five weaknesses), §5 (the identity bridge, first because everything depends on it), §6 (vaulting and brokering), §7 (the interactive path and session recording), §8 (the transfer path), §9 (segmentation and the observation point), §10 (the audit feed), §11 (the residual-risk register), §12.1 (the within-a-system phasing).
- **Zero trust generally — the discipline, NIST SP 800-207, the pillars, the vendors, the CISA maturity model and the estate migration plan** — [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) (§1–§7, §9.3).
- **The Beyond Zero agent-era paradigm** — [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) (§4–§6).
- **The integration-side view of the legacy estate and the modernisation patterns** — [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (§6, §7, §10).
- **The network-side observation answer** — [network_packet_capture_guide.md](network_packet_capture_guide.md) (§2 where you can capture, §4 the loss problem, §9 the decryption problem).
- **The SOC that consumes the agentless detection** — [secops_guide.md](secops_guide.md) (§3 detection, §4 response); the security discipline overall — [cybersecurity_guide.md](cybersecurity_guide.md).
- **Threat-modelling the wrapped estate (the DFD-per-trust-zone rule)** — [threat_modeling_guide.md](threat_modeling_guide.md) (§3.6, §9).
- **Remote access to agentless hosts and its alternatives** — [vpn_guide.md](vpn_guide.md) (§6 the VPN versus the modern alternatives, §8 the estate).
- **The OT estate, its protocols, its standards and its segmentation model** — [scada_guide.md](scada_guide.md) (§4 Purdue, §5 protocols, §6 standards, §8 defence, §10–§11 estate and worked example).
- **The midrange platform** — [ibm_as400_guide.md](ibm_as400_guide.md) (§2 architecture, §3 the OS, §5 languages and DB, §9 a banking core on the AS/400).
- **The MultiValue platforms** — [jbase_universe_guide.md](jbase_universe_guide.md) and [jbase_vs_infobasic_guide.md](jbase_vs_infobasic_guide.md).
- **The file-transfer paths and their notification patterns** — [mainframe_file_sharing_notification.md](mainframe_file_sharing_notification.md) and [linux_file_sharing_notification.md](linux_file_sharing_notification.md).
- **The MFT and workload-automation products** — [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md).

### 16.4 The Closing Summary

This guide set out to write zero trust for the estate that is not the mainframe, and it arrived at a thesis with two halves — the same two halves as the constrained-case guide, generalised across six classes.

The first half is the **shape of the deficit**. The midrange, the MultiValue family, the agentless OS tier, OT, the database and middleware layers, and the file-transfer estate share five incapacities (§3.1): they cannot be the policy-enforcement point for their own access, they cannot run the agent, they cannot join the enterprise identity plane, they cannot produce centralised audit, and they frequently cannot be patched at all. That is not a judgement about the platforms or the people who run them — it is an architectural fact about systems built before the identity and audit planes existed, and it has one consequence: **the trust decision has to be made in front of them** (§3.2).

The second half is the **answer**. The wrapping pattern is six layers (§4) — an identity-aware entry point, an identity bridge, brokered privileged access, a protocol gateway, restricted network paths and an observation point — and it is the same pattern the mainframe guide develops for one platform, applied here per class without being restated. The midrange gets the mainframe's shape with its own vocabulary and, unusually, a real authorisation model of its own (§5). The MultiValue family gets host hardening plus application-tier brokering, because its security is **inherited** rather than native and that has to be said plainly (§6). The agentless tier gets compensating controls and is told the truth that agentless coverage is weaker than agented coverage (§7). OT gets the enforcement point placed where it cannot interrupt the process (§8). The database and middleware layers get their shared credential vaulted, brokered, per-workload-split and scoped — with the honest limit that a broker is only as good as the paths it closes (§9). The transfer estate gets narrowed, because it is usually the weakest link in the chain (§10). And the work is ordered by inventory first and five rankable signals, with no figure invented to make the ordering look precise (§11).

What wrapping does not fix is named too (§12): the boundary protects the approach, not the system; it does not stop the insider with a granted authority, nor the application vulnerability that arrives through an admitted path; it does not repair a control inherited from a layer that cannot be patched; and there is a point at which replacing a system is cleaner than wrapping it forever. Everything that survives all of that is residual risk that must be distinguished from the merely unattempted, given a compensating control with its limits stated, owned by a **named accountable role**, reviewed on a date, and **accepted in writing** — exactly as Cymbal Bank did for the two credentials it brokered, the midrange it wrapped, and the reporting store it chose to replace rather than wrap (§13). A residual risk that is named is a managed risk; the same risk unstated is the finding.

And so the guide closes on the sentence it opened with, because it is the whole argument:

**zero trust for a legacy system is built at its boundary, never inside it.**
