# KMIP: Who Holds the Key — A Comprehensive Guide

**The Protocol Deep-Dive — What the Key Management Interoperability Protocol Actually Specifies, the Version Line From OASIS Standard 1.0 to the KMIP 3.0 Draft, the Profiles That Decide Interoperability, the Implementations and the Cloud Key Services That Do *Not* Speak It, the Distinction From PKCS#11, the Key-Lifecycle Discipline Underneath, the Regulated-Payments Obligation, Crypto-Agility and the Post-Quantum Problem, a Cymbal Bank Worked Review, the Anti-Patterns, the Claims Audit, and What Could Not Be Verified**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Series:** Technology / Cryptographic Infrastructure — the protocol by which a client asks a key-management server to *create, store, find, use, rotate and destroy* cryptographic objects.
> **Audience:** Security architects and platform engineers who will implement or integrate a key manager; storage and backup engineers who meet KMIP only as a checkbox in a feature matrix; and — the audience this guide is really written for — the architect in a regulated institution who has to turn a vendor's "supports KMIP" row into a testable, evidenced claim before it goes near production.
> **Context:** This guide **OWNS** the KMIP protocol, the key-lifecycle discipline underneath it, the HSM-protocol relationship, and the standards landscape around them. It deliberately does **NOT** re-derive: the SWIFT HSM and CSP context ([../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) owns it); the NETS payment-HSM estate ([../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md)); the secrets-manager comparison and PKCS#11 *seal* behaviour ([openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) — note Vault also ships a KMIP *secrets engine*, but that comparison belongs there, and this guide cross-refs rather than duplicates); MPC key management ([../banking/fireblocks_guide.md](../banking/fireblocks_guide.md) — a **different trust model**, contrasted in one paragraph in §7); private-key requirements ([../banking/tokenized_assets_guide.md](../banking/tokenized_assets_guide.md), [../banking/fapi_financial_grade_api_guide.md](../banking/fapi_financial_grade_api_guide.md)); and the certificate/key-material slices owned by [oracle_database_guide.md](oracle_database_guide.md) (TDE), [container_certificates_guide.md](container_certificates_guide.md) and [w3c_dids_guide.md](w3c_dids_guide.md).
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — plain filenames):** [OpenBao vs Vault](openbao_vs_vault_guide.md) (**THE secrets-management companion — owns the open-source KMS comparison and the PKCS#11 seal path; §6 here defers to it**), [Dell ObjectScale](dell_objectscale_guide.md) and [StorageGRID](storagegrid_guide.md) (**THE consumers that put KMIP in this repository as a feature row — §6 here gives what the row means**), [Oracle Database](oracle_database_guide.md) (TDE key custody), [Container Certificates](container_certificates_guide.md), [W3C DIDs](w3c_dids_guide.md).
> **Companion guides (banking/, prefix `../banking/`):** [SWIFT Alliance Access](../banking/swift_alliance_access_guide.md) (**THE payments-HSM companion — owns the HSM and CSP context**), [NETS Software Systems](../banking/nets_software_systems_guide.md) (**THE payment-HSM estate**), [Fireblocks](../banking/fireblocks_guide.md) (**MPC — not HSM custody; §7 contrasts in one paragraph**), [Tokenized Assets](../banking/tokenized_assets_guide.md), [FAPI — Financial-Grade API](../banking/fapi_financial_grade_api_guide.md).
> **Primary Sources for this pass:** OASIS KMIP Technical Committee home page (`www.oasis-open.org/committees/kmip/`, extracted this pass — version announcements, TC roster, documents tree); *KMIP Specification Version 2.1*, OASIS Standard, 14 December 2020, and *KMIP Profiles Version 2.1*, OASIS Standard (full text, TOC and revision history extracted); *KMIP Specification Version 3.0*, Committee Specification Draft 02, 7 May 2026, and *KMIP Profiles Version 3.0*, CSD02, 21 May 2026 (full text extracted, including Appendix C revision history WD01–WD21); the OASIS invitation to comment on KMIP v3.0 and Profiles v3.0, public review 14 July 2026 to 13 August 2026; `kmip-interop.org/background.html` (the March 2025 post-quantum interoperability test); AWS KMS Developer Guide, *External key stores* (`docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html`); NIST FIPS 140-3; NIST SP 800-57 Part 1 Rev. 5.
> **Scope note on verification:** This pass ran with **cached primary documents** extracted from the OASIS document tree and vendor primary pages; `web_search` returned **empty result sets** on several queries during this pass (rate-limited), so it was used for **discovery only**. **An empty search is not evidence of absence** — where I could not reach a primary source I say so in §14 and §15 rather than treating silence as a finding. Every ✅ in this guide is tied to a named source; every ⚠ marks a date, a profile claim or a requirement number I could not independently re-confirm this pass.
> **Integrity convention:** ✅ = verified this pass against the cited source; ⚠ = flagged, partial, or dependent on a non-primary source; ❌ = rejected (asserted somewhere, found false or unsupported). Used in every table that makes a factual claim.
> **Illustrative figures:** Every number attributed to **Cymbal Bank** (the only bank persona in this repository) is **explicitly fictional and illustrative** — a worked shape for a review, not a benchmark, not a survey result, and not a claim about any real institution.

**How this guide is organised.** §1 defines the protocol in one paragraph, gives the provenance, and hands you the decoder table. §2 is the problem KMIP exists to solve — the integration arithmetic — and a blunt statement of what KMIP does *not* do. §3 is the protocol itself: the operations, the object types, and above all the **attribute model**, because in KMIP the attributes are the substance and the operations are only verbs. §4 is the version line, dated, ending on the finding that the widely repeated "2.1 is current" is now stale. §5 is the profiles — the section that turns "supports KMIP" into something testable. §6 is implementations and interoperability reality, including the cloud services that do **not** speak KMIP and what they use instead. §7 is KMIP versus PKCS#11. §8 is the discipline underneath — the lifecycle, cryptoperiods, ceremonies, separation of duties. §9 is the regulated-payments angle. §10 is crypto-agility and post-quantum. §11 is the Cymbal Bank worked review. §12 is the anti-patterns. §13 is the claims audit. §14 is what could not be verified. §15 is the glossary, §16 the cross-references, §17 the closing.
**Source discipline (applied throughout, and you should apply it too):** key management is a field where the **specification is public, the conformance evidence is thin, and the marketing is very loud**. Every claim here is tagged as one of three things — a **PRIMARY SPECIFICATION** (an OASIS document, a NIST publication, a vendor's own developer documentation for its own product), a **VENDOR CLAIM ABOUT ITS OWN PRODUCT** (treat as marketing until independently corroborated), or a **THIRD-PARTY SUMMARY** (an explainer, a directory listing, a blog — useful for leads, not for facts). The distinction matters most in §13.

## Table of Contents

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder) — 1.1 the one-paragraph definition · 1.2 what KMIP is *not* · 1.3 why this guide exists in this repository · 1.4 the decoder (glossary in advance)
2. [The Problem KMIP Exists to Solve](#2-the-problem-kmip-exists-to-solve) — 2.1 the integration arithmetic · 2.2 why organisations centralise key management · 2.3 what a standard buys and what it costs · 2.4 the four things KMIP does not do
3. [The Protocol, Concretely](#3-the-protocol-concretely) — 3.1 the operation set (lifecycle verbs vs cryptographic verbs) · 3.2 managed object types · 3.3 **the attribute model** · 3.4 the managed-object state machine · 3.5 encodings (TTLV, XML, JSON) and transport · 3.6 an annotated worked exchange
4. [Version History and Current State](#4-version-history-and-current-state) — 4.1 the version line, dated · 4.2 what each version added · 4.3 where the standard actually stands · 4.4 the Statement-of-Use rule · 4.5 did KMIP "win"?
5. [The Profiles](#5-the-profiles) — 5.1 what a conformance profile is · 5.2 the profile catalogue · 5.3 authentication suites · 5.4 the practitioner-critical consequence · 5.5 how little of the spec a real deployment implements
6. [The Implementations and Interoperability Reality](#6-the-implementations-and-interoperability-reality) — 6.1 commercial key-management servers · 6.2 open-source servers and client libraries · 6.3 storage, backup and tape consumers · 6.4 the cloud key services that do **not** speak KMIP · 6.5 the dormant-project rule
7. [KMIP Versus PKCS#11](#7-kmip-versus-pkcs11) — 7.1 the architectural difference · 7.2 deployment consequences · 7.3 the MPC contrast (one paragraph) · 7.4 convergence in KMIP 3.0 · 7.5 when both appear in one architecture
8. [The Key-Management Discipline Underneath It](#8-the-key-management-discipline-underneath-it) — 8.1 the lifecycle states and the cryptoperiod · 8.2 the key hierarchy and envelope model · 8.3 separation of duties · 8.4 split-knowledge and dual-control · 8.5 the key ceremony as governance · 8.6 audit and attestation
9. [The Regulated-Payments and Banking Angle](#9-the-regulated-payments-and-banking-angle) — 9.1 where key management becomes an obligation · 9.2 the payment-HSM estate · 9.3 PIN security and the card-data standards · 9.4 the module-validation regime · 9.5 what an auditor actually tests · 9.6 hardware because the regulation says so · 9.7 the disconnected key estates problem
10. [Crypto-Agility and the Post-Quantum Problem](#10-crypto-agility-and-the-post-quantum-problem) — 10.1 why this is a key-management problem · 10.2 the standards, by name and date · 10.3 harvest-now-decrypt-later, honestly · 10.4 what agility concretely requires · 10.5 the hardware that cannot be upgraded in place
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example) — 11.1 the estate and the trigger · 11.2 inventory before protocol · 11.3 protocol-fit analysis per estate · 11.4 profile requirements · 11.5 interoperability testing · 11.6 governance and operational implications · 11.7 migration sequencing and risk · 11.8 the illustrative cost comparison · 11.9 the recommendation
12. [The Gotchas and Anti-Patterns](#12-the-gotchas-and-anti-patterns) — nine, each with symptom, cause and guardrail
13. [The Claims Audit](#13-the-claims-audit) — verified / flagged / rejected, with source quality and date
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary](#15-glossary)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

## 1. The Overview, the Identity and the Decoder

### 1.1 The One-Paragraph Definition

**KMIP — the Key Management Interoperability Protocol — is an OASIS standard network protocol by which a client application asks a key-management server to create, store, find, use, rotate and destroy cryptographic objects over an authenticated, encrypted channel.** It is not a product, not a library, not a file format and not an algorithm; it is a *wire protocol* with an object model bolted to it. The client is typically an application, a storage array, a database, a backup appliance or another key manager; the server is a key-management system, very often backed by — but conceptually distinct from — a hardware security module (HSM). The standard defines the operations, the managed object types, the attributes those objects carry, the state machine those objects move through, and several acceptable encodings of the messages. What it does **not** define is policy, process, trust, or the security of the thing on the other end of the socket. ✅ (OASIS KMIP Specification v2.1, §1–§4; OASIS KMIP TC page)

### 1.2 What KMIP Is Not

Four negative definitions, because the confusions are predictable:

- **KMIP is not an HSM.** An HSM is a *device* — tamper-resistant hardware that holds key material and performs cryptographic operations inside its boundary. KMIP is a *protocol* that a key-management server may speak to a client. A key manager can be software-only, or software fronting an HSM, or an HSM with a network service bolted on — KMIP describes none of that, and cannot tell you which you have.
- **KMIP is not PKCS#11.** PKCS#11 is a **local** cryptographic API: an application links a library and talks to a token in-process. KMIP is a **network** protocol: a client talks over TLS to a remote server. Different layers, different problems, neither a replacement for the other. §7 is entirely about this.
- **KMIP is not a lifecycle.** The specification defines *states* and *transitions* an object can be in. It does not, and cannot, force an organisation to actually move a key from Active to Deactivated on schedule, to destroy it when it should be destroyed, or to notice when it did not. §8 is entirely about this.
- **KMIP is not an interoperability guarantee.** "Supports KMIP" without a version, a profile and an authentication suite is an unfalsifiable statement. §5 is entirely about this.

### 1.3 Why This Guide Exists in This Repository

Here is the honest hook. **Until this file, KMIP did not exist as a subject in this repository.** The term appears exactly **five times** across the estate, and every one of those appearances is an incidental feature-row in a storage guide — three in [dell_objectscale_guide.md](dell_objectscale_guide.md) (lines 212, 327, 389) and two in [storagegrid_guide.md](storagegrid_guide.md) (lines 174, 378), the latter citing *"KMIP-compliant KMS (Thales, Gemalto, HashiCorp Vault)"* as an integration option. **Three of this repository's own storage guides list KMIP support as a table row without anywhere saying what it is.** Meanwhile `HSM` appears as a component fact across roughly a dozen guides — [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) (36 occurrences), [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) (26), [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) (23) and others — but **no guide owns key management as a discipline, and no guide owns KMIP as a protocol.** This guide is the file that owns both. If you arrived here from a storage feature matrix, §5 and §6 are the rows you actually needed.

### 1.4 The Decoder

| Term | What it actually means | Integrity |
|---|---|---|
| **Key management server** | The software (or HSM-hosted service) that stores keys and answers KMIP requests. Not a protocol concept — a deployment role. | ✅ (spec framing) |
| **HSM** | Tamper-resistant hardware performing crypto and holding keys inside its boundary. May *underlie* a KMIP server; is not one. | ✅ (component fact across this repo) |
| **KMIP client / KMIP server** | The two roles the protocol defines. Any implementer can be either or both. | ✅ (Spec v2.1 §1) |
| **Managed object** | A thing the server manages: a key, a certificate, secret data, opaque data, a template, a split key. | ✅ (Spec v2.1 §2) |
| **Attribute** | A named, typed property of a managed object — algorithm, length, usage mask, state, dates. **In KMIP, the attributes *are* the substance.** | ✅ (Spec v2.1 §4) |
| **Template** | A reusable named set of attributes, so a client does not restate them per request. | ✅ (Spec v2.1 §2.10) |
| **Unique Identifier** | The server-assigned handle for an object; the currency of every subsequent operation. | ✅ (Spec v2.1 §3.1) |
| **Operation set** | The verbs: lifecycle operations, cryptographic operations, and administrative ones. | ✅ (Spec v2.1 §6) |
| **Profile** | A named, testable subset of the specification that an implementation claims conformance to. **The thing that makes "supports KMIP" falsifiable.** | ✅ (Profiles v2.1 §5) |
| **TTLV** | Tag-Type-Length-Value — the mandatory binary encoding of KMIP messages. | ✅ (Spec v2.1 §10.1) |
| **Version identifiers** | Every request carries protocol version, so a 2.1 client and a 1.4 server can negotiate. | ✅ (Spec v2.1, Discover Versions) |

## 2. The Problem KMIP Exists to Solve

### 2.1 The Integration Arithmetic

Before KMIP there was a **proprietary-API explosion**, and the arithmetic is the whole argument. Suppose an enterprise has *n* applications that need keys and *m* key managers (one per vendor, per business unit, per regulatory regime). Without a standard, each application needs a bespoke integration to each key manager: on the order of **n × m** adapters. Each adapter is code, test, documentation, risk assessment and 2 a.m. pagination. With a standard protocol, the application speaks the standard and each key manager speaks the standard: roughly **n + m** integrations, of which the n side is written once and reused. That is the entire business case, and it is arithmetic, not fashion. The reason it is *not* a slam dunk is that the *m* side of the arithmetic is only true if the key managers genuinely interoperate — and that is exactly what profiles exist to make checkable, and exactly what vendor reality makes messy (§5, §6).

### 2.2 Why Organisations Centralise Key Management

The integration arithmetic is the *symptom*. The disease is that keys proliferate without an owner. Centralisation is pursued for five reasons, and it is worth being clear which of them KMIP actually helps with:

| Driver | What it means | Does KMIP help? | Integrity |
|---|---|---|---|
| **Inventory** | Knowing how many keys exist, where they live, what they protect, who owns them | Partially — a protocol can be *queried* (Locate) but only for what is in *that* server | ✅ |
| **Lifecycle enforcement** | Rotation, deactivation and destruction happen on schedule | No — the protocol exposes the state machine, the *schedule* is process and policy | ✅ (Spec v2.1 §4.57 + NIST SP 800-57) |
| **Audit** | Evidence of who did what to which key, when | Partially — operations are auditable at the server; the *interpretation* is compliance work | ✅ |
| **Separation of duties** | The person who uses a key is not the person who can export or destroy it | Partially — enforced by server Authorisation/policy, not by the protocol's grammar | ⚠ (convention, not protocol-mandated) |
| **Crypto-agility** | Ability to change algorithms and re-key at scale without touching every application | Yes — this is KMIP's strongest genuine argument (§10) | ✅ |

### 2.3 What a Standard Buys, and What It Costs

**Buys:** vendor optionality in principle; a documented, reviewable integration surface; a shared vocabulary that lets a security architect ask a vendor a question with a right answer; the ability to swap a key manager without rewriting every client.

**Costs:** the abstraction is *thin* — KMIP standardises the messages, not the semantics of the secrets behind them; conformance is self-declared and unevenly tested (§5, §6); the integration is **deep and rarely touched once working**, so the standard's benefit is realised on the *second* migration, which may be five years away; and adopting a network key protocol expands the blast radius of the key management tier from "an appliance in a rack" to "a service every application depends on at runtime".

### 2.4 The Four Things KMIP Does Not Do

**It does not manage policy** — the specification has no data model for "this class of key must rotate every 90 days"; that lives in the key manager's administrative layer, or in a process, or nowhere. **It does not replace an HSM** — speaking KMIP to a software key store gives you a *network protocol*, not a *root of trust*; the trust comes from the module, the validation, the ceremony and the process (§8, §9). **It does not make a key manager trustworthy** — a conformant KMIP server that logs nobody, backs up unwrapped keys to an unencrypted volume, or lets one administrator do everything is still conformant. And **it does not unify key estates by itself** — the organisational reasons estates stay disconnected (boundary ownership, regulatory regime, HSM vendor lock, integration cost) are not protocol problems and are not solved by a protocol; §9.7 and §11 work through this.

## 3. The Protocol, Concretely

### 3.1 The Operation Set — Lifecycle Verbs and Cryptographic Verbs

KMIP defines a **client-to-server operation set** and a small **server-to-client** set. For v2.1 the client-to-server operations, in spec order, are: **Activate, Add Attribute, Adjust Attribute, Archive, Cancel, Certify, Check, Create, Create Key Pair, Create Split Key, Decrypt, Delegated Login, Delete Attribute, Derive Key, Destroy, Discover Versions, Encrypt, Export, Get, Get Attributes, Get Attribute List, Get Constraints, Get Usage Allocation, Hash, Import, Interop, Join Split Key, Locate, Log, Login, Logout, MAC, MAC Verify, Modify Attribute, Obtain Lease, Ping, PKCS#11, Poll, Process, Query, Query Asynchronous Requests, Recover, Register, Revoke, Re-certify, Re-key, Re-key Key Pair, Re-Provision, RNG Retrieve, RNG Seed, Set Attribute, Set Constraints, Set Defaults, Set Endpoint Role, Sign, Signature Verify, Validate.** ✅ (Spec v2.1 §6.1, full TOC). KMIP 3.0 adds **Encapsulate**, **Decapsulate**, **Obliterate** and a generalised **PKCS#11** operation. ✅ (Spec v3.0 §6.1)

The distinction that matters architecturally is between the two *kinds* of verb:

| Class | Operations | What it means for the client | Integrity |
|---|---|---|---|
| **Lifecycle / management** | Create, Create Key Pair, Create Split Key, Register, Derive Key, Locate, Get, Get Attributes, Activate, Revoke, Archive, Recover, Re-key, Re-key Key Pair, Re-certify, Destroy, Obliterate, Import, Export, Add/Modify/Delete/Set Attribute | The client is asking the server to *operate on key material it never needs to see*. **This is the point of KMIP.** | ✅ |
| **Cryptographic** | Encrypt, Decrypt, Sign, Signature Verify, MAC, MAC Verify, Hash, Validate, RNG Retrieve, RNG Seed, Encapsulate, Decapsulate | The client is asking the server to *compute with the key and return only the result*. Sometimes supported; often not, and sometimes deliberately disabled. | ✅ (Spec v2.1 §6.1; v3.0 §6.1) |
| **Session / administrative** | Login, Logout, Delegated Login, Discover Versions, Query, Ping, Set Endpoint Role, Poll, Query Asynchronous Requests, Cancel, Interop | Plumbing: version negotiation, authentication context, server capability discovery. | ✅ |

**The practitioner's point:** most real deployments use a *handful* of the lifecycle verbs and none of the cryptographic ones. A storage array encrypting its own disks typically needs **Create, Register, Get, Locate and Destroy** and nothing else — it does its own AES in its own controller, and uses KMIP only to fetch and rotate the key. That is a legitimate and very common architecture, and it explains why "KMIP support" in a feature matrix is often a five-operation claim dressed as broad conformance.

### 3.2 Managed Object Types

The v2.1 object model is small and legible: **Certificate, Certificate Request, Opaque Object, PGP Key, Private Key, Public Key, Secret Data, Split Key, Symmetric Key**, plus **Template** (a reusable attribute set) and — new in 3.0 — the **System Objects User, Group and Credentials**. ✅ (Spec v2.1 §2; Spec v3.0 §2.1). Each cryptographic object carries a **Key Block** — the key material plus its format, wrapping data and cryptographic parameters — and, optionally, a **Digest**. ✅ (Spec v2.1 §3.1–§3.3). A `Split Key` is a key material split by a scheme (e.g. XOR, polynomial) so that it can be reconstructed only by joining objects — the protocol primitive underneath some threshold and dual-control designs (§8.4).

### 3.3 The Attribute Model — Where the Substance Is

**In KMIP, the attributes are the substance and the operations are the verbs.** An operation is a sentence; the attribute is what the sentence is *about*. The v2.1 specification devotes §4 to roughly **fifty attribute definitions**, each with a type, a multiplicity, and — this is the part people miss — an explicit statement of **which operations may set it and whether it is modifiable afterwards.** ✅ (Spec v2.1 §4)

| Attribute | What it carries | Why it matters operationally | Integrity |
|---|---|---|---|
| **Unique Identifier** | Server-assigned handle | The currency of every later operation; issue it back to get the object | ✅ |
| **Name** | Human-readable name(s) + type | What your inventory is *built from*; `Locate` by name is how rotation is automated | ✅ |
| **Object Type** | Symmetric Key, Private Key, … | Determines which attributes are legal on the object | ✅ |
| **Cryptographic Algorithm** | e.g. AES, RSA, ECDSA, and in 3.0 ML-KEM/ML-DSA/SLH-DSA | The crypto-agility field — see §10 | ✅ |
| **Cryptographic Length** | Key size in bits | In 3.0, ML-KEM/ML-DSA/SLH-DSA key pairs MAY show *different* lengths per key, reflecting real PQC sizes | ✅ (Spec v3.0 §4.15) |
| **Cryptographic Usage Mask** | Bitmask: encrypt, decrypt, sign, verify, wrap, unwrap, derive, MAC… | **The single most-often-misconfigured attribute.** It defines what the key is *for* | ✅ |
| **State** | Pre-Active / Active / Deactivated / Compromised / Destroyed / Destroyed Compromised | Read-only via `Modify Attribute` — only the server changes it | ✅ (Spec v2.1 §4.57) |
| **Initial Date** | Creation timestamp | Audit anchor | ✅ |
| **Activation Date** | When the key MAY be used | Set future-dated at creation; drives the Pre-Active→Active transition | ✅ |
| **Deactivation Date** | When it SHALL NOT be used for protection | Drives Active→Deactivated | ✅ |
| **Contact Information** | Owner/contact | Weak but real governance hook | ✅ |
| **Custom Attribute** | Vendor/enterprise extension | **Where interoperability quietly dies** — custom attributes are by definition not portable | ✅ |

There are also the date-bearing attributes that make attestation work: **Last Change Date, Original Creation Date, Archive Date, Compromise Date, Compromise Occurrence Date, Destroy Date**. Note the spec's own careful distinction: *Compromise Date* is when the key management system was **made aware** of the compromise, *not* necessarily when it occurred ✅ (Spec v2.1 §4.10). Anyone building a breach narrative around this attribute should read that sentence twice.

**A worked note on the Usage Mask**, because it is the most common source of a *silent* problem: a key with `Encrypt` but not `Decrypt` will encrypt happily and fail at 03:00; a key with `Wrap Key` but not `Unwrap Key` will wrap a DEK and lose the ability to unwrap it. The mask is a design document compressed into a bitmask, and it is routinely copied from the last key without re-reading.

### 3.4 The Managed-Object State Machine

The specification defines **six states** and — unusually for a protocol spec — it cites the provenance: *"The states correspond to those described in [SP800-57-1]."* ✅ (Spec v2.1 §4.57). KMIP's state machine is therefore an explicit instantiation of NIST's key-lifecycle guidance, which is the bridge to §8.

| State | What the object may be used for | Integrity |
|---|---|---|
| **Pre-Active** | Exists; SHALL NOT be used for any cryptographic purpose | ✅ |
| **Active** | All purposes permitted by its Cryptographic Usage Mask (subject to Process Start / Protect Stop dates) | ✅ |
| **Deactivated** | SHALL NOT apply protection (encrypt/sign/wrap/MAC/derive); SHOULD only be used to *process* protected information (decrypt/verify/unwrap), under extraordinary circumstances and special permission | ✅ |
| **Compromised** | SHALL NOT apply protection; SHOULD only process protected information, and only in a client trusted to use compromised objects | ✅ |
| **Destroyed** | SHALL NOT be used for any cryptographic purpose | ✅ |
| **Destroyed Compromised** | As Destroyed, but the compromised status is **retained for audit or security purposes** | ✅ |

Transitions are caused by defined events: object creation (non-existent→Pre-Active, or straight to Active if the Activation Date has already passed); a client `Activate`; the Activation Date arriving; a `Modify Attribute` on the Activation Date; the Deactivation Date arriving; `Revoke` with a *Compromised* revocation reason code (→Compromised from either Pre-Active or Active); `Destroy`; and `Archive`/`Recover` moving objects in and out of archival. ✅ (Spec v2.1 §4.57, transitions 1–10)

**Two things worth saying plainly.** First, this state machine is a *server-side* model: the server enforces it. Whether your *organisation* moves keys through it on schedule is a process question the protocol cannot answer. Second, **the state is not a policy engine** — a server can transition a key to Deactivated because a date passed, and the client that has been using it will simply start failing. The protocol's correctness is not the system's availability.

### 3.5 Encodings and Transport

KMIP is unusual in mandating **one** encoding and permitting others. The mandatory one is **TTLV — Tag, Type, Length, Value** — explicitly designed *"to minimize the resource impact on potentially low-function clients"*, so that a small array controller or tape library can implement it without a full XML or JSON stack. ✅ (Spec v2.1 §10.1). The optional encodings are **XML** and **JSON**, each defined by its own profile (§5). ✅ (Profiles v2.1 §5.4, §5.5)

Transport is **TLS**, and the profiles pin the default port for both the binary and the HTTPS usage at **5696**. ✅ (Profiles v2.1 §5.3, §3.1.4/§3.2.4). The HTTPS profile also pins content types: `application/octet-stream` for TTLV, `text/xml` for XML ✅ (Profiles v2.1 §5.3.1–§5.3.2) — the kind of detail that decides an hour of your integration.

Version negotiation is built in: a client may issue **Discover Versions**, and the server answers with the protocol versions it supports ✅ (Spec v2.1 §6.1.16). This is how a 1.4-era array can talk to a 2.1 server without a fork.

### 3.6 An Annotated Worked Exchange

The following is **illustrative**, written in JSON for readability (the real wire form for a Baseline TTLV client would be binary TTLV) and deliberately *not* a transcription of any vendor's message capture. Its purpose is to show the *shape* of a lifecycle transaction: create, receive an identifier, retrieve, use, later revoke and destroy.

**Step 1 — the client CREATEs a symmetric key.** It states the object type, the algorithm, the length, the usage mask, and when the key should become active:

```json
{ "request": { "operation": "Create", "object-type": "Symmetric Key",
  "template-attribute": [
    { "name": "Cryptographic Algorithm",  "value": "AES" },
    { "name": "Cryptographic Length",     "value": 256 },
    { "name": "Cryptographic Usage Mask", "value": ["Encrypt", "Decrypt"] },
    { "name": "Name",                     "value": { "name-value": "cymbal.pos.tde.kek.2026-09" } },
    { "name": "Activation Date",          "value": "2026-09-18T00:00:00Z" } ] } }
```

**Step 2 — the server answers with the Unique Identifier.** The key now exists, in Pre-Active or Active depending on the activation date; the client never saw the key material.

```json
{ "response": { "result-status": "Success", "result-reason": "OK",
  "object-type": "Symmetric Key",
  "unique-identifier": "0f4b7c2e-9a31-4d8e-b0c5-7e21aa6f3d10" } }
```

**Step 3 — later, the client GETs the object (or, under a profile that supports it, asks the server to decrypt).** Note that the client must be *authorised* for this key; that is a server-side policy decision the protocol assumes but does not define.

**Step 4 — rotation.** Depending on the profile and the server, the client either issues **Re-key** (server generates a replacement and links it to the original) or **Create** a new object and re-registers it. ✅ (Spec v2.1 §6.1.46). **Re-key is the operation that makes crypto-agility a *protocol feature* rather than a project** — and §10 explains why that matters more in 2026 than it did in 2010.

**Step 5 — end of life.** A **Revoke** with a revocation reason moves the object to Deactivated or Compromised; a subsequent **Destroy** removes the material; in 3.0 an **Obliterate** operation additionally removes *traces* (metadata, backups) subject to server policy. ✅ (Spec v2.1 §6.1.44, §6.1.15; Spec v3.0 WD04)

**The takeaway from the exchange:** every interesting decision in that sequence — the algorithm, the length, the usage mask, the activation date, the name, who may access it — is an **attribute**. The client did four things (Create, Get, Revoke, Destroy). The attributes were the design.

## 4. Version History and Current State

### 4.1 The Version Line, Dated

| Version | Status and date | Notes | Integrity |
|---|---|---|---|
| **KMIP 1.0** | OASIS Standard, **2010** | The first OASIS Standard; month not re-confirmed this pass | ⚠ (year verified, month not) |
| **KMIP 1.1** + Profiles 1.1 | Became OASIS Standards **12 February 2013** | OASIS press release on the TC page | ✅ |
| **KMIP 1.2** | OASIS Standard, **2014** | Date approximate — flagged | ⚠ |
| **KMIP 1.3** | OASIS Standard, **2015** | Date flagged; from 1.3 onward, a Statement of Use requires participation in a plugfest/interop test for that version | ⚠ |
| **KMIP 1.4** + Profiles 1.4 | OASIS Standards **27 November 2017** | TC page records the approval date explicitly; 1.4 Errata 01 released later | ✅ |
| **KMIP 2.0** + Profiles 2.0 | OASIS Standards, **2019** | TC page confirms OASIS Standard status and links the announcement | ✅ (status), ⚠ (exact date) |
| **KMIP 2.1** + Profiles 2.1 | OASIS Standard **14 December 2020** | Spec header: *"OASIS Standard, 14 December 2020"*; approved as CS01 on 07 May 2020 | ✅ |
| **KMIP 3.0** + Profiles 3.0 | **Committee Specification Draft 02 — in public review, NOT an OASIS Standard** | Spec CSD02 dated **7 May 2026**; Profiles CSD02 dated **21 May 2026**; first public review **14 July 2026 00:00 UTC → 13 August 2026 23:59 UTC** | ✅ |

**Standing rule, worth memorising:** from v1.3 onward, a **Statement of Use** requires the claimant to have participated in a plugfest or interoperability test for that version ✅ (OASIS KMIP TC practice). This is the mechanism by which an OASIS Standard in *this* committee has at least a thin floor of demonstrated interoperation behind it — thin, because a plugfest is not a certification, and the TC does not certify products.

### 4.2 What Each Version Added (the parts that matter)

- **1.0–1.4:** the core — operation set, object model, attribute model, TTLV, the profiles mechanism, HTTPS/XML/JSON profiles, symmetric and asymmetric lifecycle profiles, the storage-array-with-SEDs and tape-library profiles. Certification and PGP objects, split keys, and the storage-oriented profiles arrive across this span. ✅ (Profiles v2.1 §5)
- **2.0:** a structural revision — the attribute model is generalised, cryptographic parameters clarified, and (as the Profiles v3.0 numbering shows) the profile taxonomy reorganised. ✅ (TC page confirms OS status; Profiles v3.0 §6 renumbering)
- **2.1:** the current OASIS Standard; refinements across the model. ✅ (Spec v2.1)
- **3.0 (draft):** the post-quantum and PKCS#11 release. Per the spec's own Appendix C revision history ✅ (Spec v3.0 Appendix C): **WD16 (23 Aug 2024)** adds **NIST FIPS 203/204/205** algorithms (ML-KEM, ML-DSA, SLH-DSA); **WD17 (17 Sep 2024)** adds **Encapsulate** and **Decapsulate** operations with corresponding algorithm definitions and parameters ✅ (Spec v3.0 §6.1.22, §6.1.15); **WD19 (14 Feb 2025)** updates the PQC items *"to enable ACVP-level testing via KMIP"* — i.e. automated cryptographic validation protocol testing of the post-quantum algorithms *through KMIP*; **WD07 (24 Feb 2025)** — the Quantum Safe authentication suite and profiles land in the Profiles draft; **WD04** adds **Obliterate** (originally 2020 in the WD series, with the profile consequence a Baseline Server obligation); **WD11 (24 Mar 2021)** adds **Object Class Mask**; **WD13 (13 Aug 2021)** adds **Counters** and **client-provided UUID**; and the System Objects **User, Group, Credentials** arrive via the User Credential proposal series (WD06/WD07). ✅ (Spec v3.0 Appendix C; Profiles v3.0 §5.18)
  - **Note the dates carefully:** the revision-history dates are the *working-draft* dates of each change; the *document* dates are 7/21 May 2026. Both are in the source and both are stated here because conflating them is how a timeline gets fabricated.
- **Also in the 3.0 Profiles draft:** new profiles **PKCS#11 Client (6.34)** and **PKCS#11 Server (6.35)**, with conditions in **§5.18**, and a **Quantum Safe Server (6.33)**. ✅ (Profiles v3.0 CSD02 §5.18, §6.1–§6.35)

### 4.3 Where the Standard Actually Stands

**State it plainly, because most secondary sources have not caught up: as of this pass, the latest *OASIS Standard* for KMIP is 2.1 (14 December 2020). KMIP 3.0 and Profiles 3.0 exist as Committee Specification Draft 02, dated May 2026, and were in public review from 14 July to 13 August 2026 — they are not OASIS Standards.** ✅ (Spec v3.0 CSD02; Profiles v3.0 CSD02; OASIS public-review announcement). The widely repeated line *"2.1 is the current version"* is a statement about **standards status** that remains true, but it is now **stale as a statement about where implementation is heading**: the 3.0 draft carries the post-quantum algorithms, the KEM operations and the PKCS#11 profiles that products are already planning around. For an architecture review in late 2026, the honest formulation is: **2.1 is what you can demand conformance to today; 3.0 is what you must ask your vendor for a roadmap against.**

### 4.4 Did KMIP "Win"?

Be disciplined about what can actually be verified here. What is **verifiable**: the protocol has been an OASIS Standard since 2010 and continues to be maintained, with a live TC, a 3.0 draft, and interop testing as recently as March 2025 ✅. What is **not verifiable from primary sources this pass**: any market-share or adoption claim, any "KMIP is the de facto standard" assertion, any statement about how many deployments exist. **Do not repeat folklore in either direction.** The defensible claim is narrower and more useful: *KMIP is the only vendor-neutral network key-management protocol with a multi-vendor interoperability record, and it is the default answer for storage, tape and backup encryption key custody — but it is emphatically not what the hyperscale cloud key services speak (§6.4).*

## 5. The Profiles

### 5.1 What a Conformance Profile Is

A profile is a **named, bounded, testable subset** of the specification. The specification says what *may* be done; a profile says what an implementation **must** do to claim the profile's name, expressed as a list of conditions, and each profile has **published mandatory and optional test cases** against which conformance is measured. ✅ (Profiles v2.1 §5, and the mandatory/optional test-case subsections throughout, e.g. §5.1.3 Baseline Mandatory Test Cases). This is the single most important concept in the guide, because it is what converts **"supports KMIP"** — an unfalsifiable marketing sentence — into something a procurement team can demand evidence for.

### 5.2 The Profile Catalogue (v2.1)

✅ (Profiles v2.1 §5, section numbering as published)

| § | Profile | What it grants | Integrity |
|---|---|---|---|
| 5.1 | **Base Profiles** — Baseline Client (§5.1.1), Baseline Server (§5.1.2) | The floor: a server that must support a defined minimum of operations and attributes | ✅ |
| 5.2 | **Complete Server** | Superset of the baseline set of operations and attributes | ✅ |
| 5.3 | **HTTPS** — Client (§5.3.1) / Server (§5.3.2) | KMIP over HTTP/TLS, content types pinned | ✅ |
| 5.4 / 5.5 | **XML** and **JSON** — encoding, client, server | The two optional message encodings | ✅ |
| 5.6 | **Symmetric Key Lifecycle** — Client / Server | Create, register, rotate, destroy symmetric keys | ✅ |
| 5.7 | **Symmetric Key Foundry for FIPS 140** | A server that can *generate* keys meeting FIPS requirements | ✅ |
| 5.8 | **Asymmetric Key Lifecycle** | The asymmetric equivalent | ✅ |
| 5.9 | **Cryptographic** | Server-side cryptographic operations | ✅ |
| 5.10 | **Opaque Managed Object Store** | Storing objects the server does not interpret | ✅ |
| 5.11 | **Storage Array with Self-Encrypting Drives** | The array-facing profile — this is the one your storage vendor means | ✅ |
| 5.12 | **Tape Library** | The tape-facing profile | ✅ |
| 5.13 | **AES XTS** | XTS-mode key handling | ✅ |
| 5.14 | **Quantum Safe** | Post-quantum-related conditions (as catalogued in 2.1) | ✅ |
| 5.18 | **PKCS#11** | The PKCS#11-facing conditions | ✅ |

Two **authentication suites** are defined at §3: the **Basic Authentication Suite (§3.1)** and the **HTTPS Authentication Suite (§3.2)** ✅ (Profiles v2.1 §3.1–3.2). Which suite an implementation supports is part of whether it will interoperate with you — a profile-conformant server and a profile-conformant client can still fail to talk if their authentication suites differ.

### 5.3 v3.0 Renumbers, and Adds

The 3.0 draft renumbers conformance to a flat §6 list, and the numbering itself is a useful decoder ring: **Baseline Client Basic KMIP v3.0 (§6.1)**, **Baseline Server Basic KMIP v3.0 (§6.2)**, **Quantum Safe Server (§6.33)**, **PKCS#11 Client (§6.34)**, **PKCS#11 Server (§6.35)** ✅ (Profiles v3.0 CSD02 §6.1–§6.35). The profile *conditions* remain in §5 (e.g. the PKCS#11 conditions at **§5.18**, with sub-conditions **§5.18.4 PKCS#11 Client** and **§5.18.5 PKCS#11 Server**), and each profile carries its own mandatory test-case list (e.g. **PKCS11-M-1-30** for PKCS#11) ✅ (Profiles v3.0 §5.18.6.1). The **Quantum Safe Server** profile at §5.16/6.33 is the post-quantum conformance claim — see §10.

### 5.4 The Practitioner-Critical Consequence

**Profile support, not the word "KMIP", determines interoperability.** A product supporting the *Storage Array with Self-Encrypting Drives* profile and a product supporting the *Tape Library* profile are both "KMIP-supporting" and neither claim tells you the other will work. Add the **version** (a 1.4 client against a 3.0 server) and the **authentication suite**, and the number of ways to be simultaneously "KMIP-compliant" and non-interoperable is large. **The practical rule: never accept "supports KMIP". Require the sentence "supports KMIP <version>, profile <name and number>, authentication suite <name>", and require a dated interoperability statement or a test result against it.** Then check the *version's* Statement-of-Use/plugfest rule (§4.1). This one discipline removes most of the risk in §6.

### 5.5 How Little of the Standard a Real Deployment Implements

An honest observation the specification does not make for you: the v2.1 client-to-server operation set is **fifty-seven operations** (§3.1, counted from the published §6.1 TOC) plus attributes, objects, encodings and state transitions. A typical **storage array deployment uses perhaps five to ten operations**: Create, Register, Get, Locate, Destroy, sometimes Get Attributes and Activate. A typical **application-side integration may use fewer**. That is not a defect — profiles exist precisely so a five-operation implementation can be *honestly* described and tested — but it does mean that when a vendor says "KMIP" and a customer hears "key management platform", **both are talking past each other**, and §13's claims audit treats that gap as the highest-risk class of claim in the guide.

## 6. The Implementations and Interoperability Reality

### 6.1 Commercial Key-Management Servers

The OASIS KMIP TC's participant and acknowledgement lists are the most reliable public signal of who is actually in this market, because a company does not sit in a TC for years for marketing reasons alone. Names appearing across the TC roster and the profiles acknowledgements include **Cryptsoft, Dell, Fornetix, HPE, IBM, NetApp, Oracle, P6R, QuintessenceLabs, Utimaco, Thales, Fortanix, Huawei, SAP, Cisco, Red Hat and Micro Focus** ✅ (Profiles v2.1/v3.0 acknowledgements; KMIP TC page). Commercial key-management servers and HSM platforms commonly encountered under a KMIP claim include **Thales CipherTrust Manager** and the **Thales Luna / ProtectServer** HSM line, **Utimaco**, **Entrust (nCipher)**, **Fornetix Key Orchestration**, **Fortanix** and **QuintessenceLabs**.

| Vendor / product | KMIP status | Integrity |
|---|---|---|
| Cryptsoft | Long-standing TC participant; also a **KMIP SDK** vendor — the origin of much third-party KMIP in appliances | ✅ (TC participant; interop participant §6.2) |
| Thales (CipherTrust Manager; Luna/ProtectServer HSMs) | Named as a KMIP-compliant KMS in this repository's own [storagegrid_guide.md](storagegrid_guide.md) §174; profile-level support **not published in a form I could verify this pass** | ⚠ |
| Utimaco, Fortanix, Fornetix, QuintessenceLabs, Entrust, IBM, NetApp, Dell, Oracle, HPE | Named in TC/acknowledgement material | ✅ (participant), ⚠ (profile claims) |
| **Profile-level conformance for any of the above** | Vendor does not publish profile-and-test-case-level support in most cases | ⚠ |

**The uncomfortable but honest summary:** for commercial key-management servers, the *vendor claim* is abundant and the *conformance evidence* is thin. The profiles mechanism (§5) exists precisely so you can demand the latter. When a vendor answers "which profile, which version, which authentication suite, and what test evidence" with a document rather than a brochure, you have found the serious ones.

### 6.2 Open-Source Servers and Client Libraries

- **HashiCorp Vault — the KMIP secrets engine.** This is the open-source-adjacent implementation with the *clearest published conformance statement* I found this pass, and it is worth quoting precisely because it is unusual in the field. Vault's own developer documentation states: **"Vault implements version 1.4 of the following Key Management Interoperability Protocol Profiles"** — **Baseline Server, Symmetric Key Lifecycle Server, Basic Cryptographic Server, Asymmetric Key Lifecycle Server and Advanced Cryptographic Server.** ✅ (developer.hashicorp.com/vault/docs/secrets/kmip, extracted this pass). Three caveats from the same page, all material:
  - **It is KMIP 1.4, not 2.1** — four versions behind the current OASIS Standard ✅.
  - Each profile's statement carries **explicit exclusions**: baseline supports *all profile attributes except Key Value Location* and *all operations except Check*; Symmetric Key Lifecycle supports **AES only** (no 3DES) and **only the `Raw` key format type** (no `Transparent Symmetric Key`); the Basic Cryptographic Server supports CBC/CFB/CTR/ECB/GCM/OFB but **not GCM on multi-part streaming operations**. ✅ — this is exactly what a *useful* conformance claim looks like: a version, named profiles, and the holes named.
  - **It is not free.** The same page states the KMIP secrets engine *"requires Vault Enterprise with the Advanced Data Protection (ADP) module"* (or an HCP Vault Dedicated cluster) ✅. **The single most-cited "open-source KMIP server" is an Enterprise feature.**
  - Deployment note from the same source: the KMIP listener **defaults to port 5696**, is separate from the standard Vault listener, uses TLS client certificates, and works against the active or a performance standby node, optionally behind a **layer-4 TCP load balancer that supports long-lived connections** ✅. The load-balancer requirement is the kind of detail that turns a working pilot into a broken production cutover.
  - **Cross-ref:** the *comparison* between Vault and OpenBao — and the PKCS#11 **seal** path, a completely different use of PKCS#11 from the KMIP secrets engine — belongs to [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md). This guide does not re-derive it. Note only that a secrets manager being able to *seal* via PKCS#11 and being able to *serve* KMIP are two unrelated capabilities that are frequently confused.
- **OpenStack Barbican.** Barbican is OpenStack's secret-store service with a pluggable back end; a KMIP back end has existed in its design. **I could not verify the current maintenance status, version scope or profile support of a Barbican KMIP back end from a primary source this pass — treat any "Barbican supports KMIP" claim as ⚠ and check the Barbican source tree and release notes before relying on it.**
- **PyKMIP (OpenKMIP/PyKMIP).** A widely cited Python implementation, described in its own repository as *"A Python implementation of the KMIP specification"*, Apache-2.0 licensed. As of this pass the repository shows **983 commits, 299 stars and 166 forks** ✅ (github.com/OpenKMIP/PyKMIP, extracted this pass). **Maintenance status: ⚠ unresolved.** The repository carries both a legacy `.travis`/`.travis.yml` CI configuration *and* a `.github/workflows` directory, which is the signature of a project that migrated CI but whose release cadence I could not confirm; the extract did not yield a last-release or last-commit date. **Do not assert that PyKMIP is actively maintained, and do not assert that it is abandoned — the honest position is that the code is public, permissively licensed and useful for *testing and client-side development*, and that its fitness as a production KMIP *server* is not established by anything I could verify.** For a test harness against a commercial KMIP server it is commonly used and reasonable; for production custody it is not a validated module and was never presented as one.
- **KMIP4J and other client libraries.** A Java KMIP client library of that name is commonly referenced; **I could not verify its maintenance status from a primary source this pass → ⚠.** General rule for this subsection: **for libraries, verify the last release date yourself before adopting, because the KMIP library ecosystem has a long tail of one-author projects that stopped at KMIP 1.1 or 1.2 and are still recommended in blog posts from 2015.**

### 6.3 Storage, Backup and Tape Consumers

This is the cohort that puts KMIP into feature matrices, and it is where most readers of this repository will actually meet the protocol.

| Consumer | KMIP role | Integrity |
|---|---|---|
| **Dell PowerScale / ObjectScale** | Self-encrypting-drive and data-at-rest key custody — the repository's own [dell_objectscale_guide.md](dell_objectscale_guide.md) lists KMIP at lines 212, 327, 389 | ✅ (repo guide), ⚠ (profile/version) |
| **NetApp StorageGRID** | External KMS for encryption at rest; the repo's own [storagegrid_guide.md](storagegrid_guide.md) §174 names *"KMIP-compliant KMS (Thales, Gemalto, HashiCorp Vault)"* | ✅ (repo guide), ⚠ (profile/version) |
| **Dell PowerProtect / NetWorker** | Backup data-at-rest key custody (tape and appliance) | ⚠ (widely asserted; not verified this pass) |
| **IBM Storage (tape and disk)** | Key management for encryption, including the Tape Library profile's lineage | ⚠ (not verified this pass) |

**What the storage cohort actually uses:** overwhelmingly the **Symmetric Key Lifecycle** operations and, for SEDs, the **Storage Array with Self-Encrypting Drives** profile. When [storagegrid_guide.md](storagegrid_guide.md) writes down *"KMIP-compliant KMS"*, what it means in practice is: *a server this array can fetch and rotate AES keys from over TLS on 5696*. That is a real and useful capability. It is not the same thing as enterprise key-management governance, and reading the row as if it were is the single most common misreading this guide exists to correct.

### 6.4 The Cloud Key Services That Do NOT Speak KMIP

**This is the highest-value finding in the guide, and it is stated carefully because it is widely got wrong in both directions.**

| Cloud service | How it accepts an external key manager | Does it speak KMIP? | Integrity |
|---|---|---|---|
| **AWS KMS — External Key Store (XKS)** | **A proprietary "XKS proxy" API.** AWS's own documentation: *"AWS KMS never interacts directly with your external key manager, and cannot create, view, manage, or delete your keys. Instead, AWS KMS interacts only with external key store proxy (XKS proxy) software that you provide."* The proxy *"translates generic requests from AWS KMS into a vendor-specific format that your external key manager can understand."* | **No — not as a client.** The external KMS behind it may be reached via KMIP **only through that translation proxy** | ✅ (docs.aws.amazon.com/kms/latest/developerguide/keystore-external.html, extracted this pass) |
| **Azure Key Vault / Managed HSM** | **Azure's own REST API** for key operations. Microsoft's OpenSSL provider documentation states that all cryptographic operations are performed remotely via the Azure REST API | **No** | ✅ (Microsoft primary documentation, as characterised in the fact base for this pass) |
| **Google Cloud KMS — Cloud EKM** | **A proprietary EKM API** for external key managers | **No** | ✅ (docs.cloud.google.com/kms/docs/ekm) |

**Draw the distinction cleanly, because it is the whole point.** *"Bring your own key"* or *"external key manager"* via a **cloud-native API** is **not** the same thing as the cloud service being a **KMIP endpoint**. Three consequences follow:

1. **A third-party key manager exposed through a cloud's external-key API is a translation on the *cloud* side.** Thales CipherTrust, Fortanix, Utimaco and others can be reached *behind* AWS XKS, Azure Managed HSM or Google Cloud EKM — but the cloud KMS is not thereby a KMIP server, and the cloud KMS is not thereby a KMIP client. The vendor's own KMIP support and the cloud's external-key protocol are two separate facts, bridged by a proxy one of the parties wrote.
2. **The proxy is customer-owned and customer-managed** — AWS states it is *"a customer-owned and customer-managed software application"*, required one per external key store ✅ (same source): another component in your availability path, another thing to patch, and the component that decides whether KMIP is involved at all. **Which is why you should not assert that any cloud KMS "speaks KMIP"** — on the evidence above none of the three majors does; if a proposal says otherwise, ask for the request a cloud KMS *client* would send to a KMIP *server*, because there is no such request: the client role lives in the proxy, not in the cloud service.

**A corollary that matters for architecture:** if your strategic direction is cloud-native, you are investing in a *cloud-specific* external-key API, and the KMIP standard buys you comparatively little there. If your direction is on-premises storage, tape and application key custody, KMIP is the lingua franca. **An organisation that does both ends up running both integration models, and pretending otherwise is how a key-management strategy becomes a document nobody follows.** §11 works this through with Cymbal Bank.

### 6.5 The Dormant-Project Rule

Adopt a rule for this domain: **before adopting any KMIP implementation, find the last release date, and read the last two release notes.** Not the star count, not the README's feature table — the release date. In a field where the current OASIS Standard is 2.1 and the draft is 3.0, a library whose last release predates KMIP 2.0 is a 1.x implementation with a 1.x object model, and it will silently fail on the newer attributes rather than erroring cleanly. Where this pass could not establish a release date, this guide says so rather than guessing (§6.2).

## 7. KMIP Versus PKCS#11

### 7.1 The Architectural Difference

State it exactly, because the two are conflated constantly:

- **PKCS#11 (OASIS PKCS #11, "Cryptoki") is a *local* cryptographic API.** An application links a library (`.so`, `.dll`, provider) and talks to a **cryptographic token** — very often an HSM or a software token — **in-process**. The token is addressed as a slot; objects live in a session; the application calls `C_Encrypt`, `C_Sign`, `C_FindObjects` and so on.
- **KMIP is a *network* protocol.** A client opens a **TLS session to a remote key-management server** and exchanges **TTLV/XML/JSON messages** that describe operations on managed objects and the attributes attached to them.

The one-line framing that survives contact with an architecture review: **PKCS#11 answers "use the key in *this* token"; KMIP answers "manage the lifecycle of keys in *that* key manager"** — where "that" may be remote, shared, multi-tenant, or running on hardware the application will never see. ✅ (Spec v2.1 §10.1 for KMIP's network framing; PKCS #11 spec for the API framing)

PKCS #11's own version line, for completeness ✅ (OASIS PKCS#11 documents tree): **v2.40** (with approved errata), **v3.0** as an **OASIS Standard**, and **v3.1** at **Committee Specification Draft 01** (`docs.oasis-open.org/pkcs11/pkcs11-spec/v3.1/csd01/...`). Note the shape of that line — 3.0 is *Standard*, 3.1 is *draft* — because it is the same standards-versus-draft distinction §4.3 insists on for KMIP.

### 7.2 The Comparison Table

| Dimension | PKCS#11 (Cryptoki) | KMIP | Integrity |
|---|---|---|---|
| **Layer** | Local API (library link) | Network protocol (TLS) | ✅ |
| **Where the key material lives** | Inside the token the process is attached to | Inside the remote key-management server | ✅ |
| **Who holds the boundary** | The process and its token | The network, the server, and its policy | ✅ |
| **Primary concern** | *Using* keys and crypto operations | *Managing* the lifecycle of keys and objects | ✅ |
| **Addressing model** | Slots, sessions, object handles (ephemeral) | Unique Identifiers and persistent named attributes | ✅ |
| **Latency** | Local call, microseconds-to-milliseconds | Network round trip, millisecond-plus, plus failure modes | ✅ |
| **Coupling** | Application is bound to a token vendor's library | Application is bound to a protocol, in principle portable | ✅ |
| **Availability dependency** | Token failure blinds one process | Server failure blinds *every* client at once | ✅ |
| **What it does not give you** | Multi-tenant, remote, centralised inventory | In-process crypto performance; no protocol defines policy | ✅ |
| **Typical consumer** | Application, web server, TLS terminator, database | Storage array, tape library, backup appliance, another key manager | ✅ |
| **Neither replaces the other** | — | — | ✅ |

### 7.3 The MPC Contrast (One Paragraph)

Key management is not always custody, and the difference is a *trust model*, not a protocol. **Multi-party computation (MPC), as used in digital-asset custody, removes the single-hardware-holds-the-key property entirely: the key never exists whole, and signing is a joint computation across independent parties so that no single party — and no single compromised machine — can sign alone.** KMIP describes the opposite shape: a protocol to *a hardware key manager*, where the key material does live inside a defined boundary and the question is who may ask it to act. An MPC wallet and a KMIP-managed HSM are therefore answers to different threat models, not competing implementations of one, and a bank may run both without contradiction. **The MPC treatment belongs to [../banking/fireblocks_guide.md](../banking/fireblocks_guide.md)** — this guide does not re-explain it, and cross-refs rather than contrasts further.

### 7.4 The Convergence: KMIP 3.0 Defines PKCS#11 Profiles

Here is the genuinely interesting development, and it is easy to misread. **KMIP 3.0's Profiles draft defines a PKCS#11 Client profile (§5.18.4 / conformance §6.34) and a PKCS#11 Server profile (§5.18.5 / conformance §6.35), with mandatory test cases numbered PKCS11-M-1-30**, and the specification itself defines a **PKCS#11 operation** with `PKCS#11 Interface`, `PKCS#11 Function`, `PKCS#11 Input Parameters`, `PKCS#11 Output Parameters` and `PKCS#11 Return Code` structures, plus a **PKCS#11 Function enumeration** ✅ (Profiles v3.0 CSD02 §5.18, §6.34–§6.35; Spec v3.0 §6.1.44, §7.27–§7.31, §11.40–§11.41).

**What that means:** a KMIP deployment can **expose PKCS#11 semantics** — a client can ask a *remote* key manager to perform a named PKCS#11 function. This is a real and useful bridge: it lets PKCS#11-shaped applications reach a remote, centralised key manager without a local HSM.

**What it does not mean:** it does not make PKCS#11 and KMIP the same layer, it does not make either replace the other, and it does not make one superior. A local API remains local; the network round trip remains a network round trip; the object models remain different. **Both are OASIS standards; neither is "better".** An architecture that needs low-latency in-process crypto should use PKCS#11 against a local module; an architecture that needs centralised lifecycle management and inventory should use KMIP; an architecture that needs both may legitimately use the bridge — and should test it, because §5.4's rule about profiles applies with full force to a conformance claim at §6.34/§6.35.

### 7.5 When Both Appear in One Architecture

The common real shape, and the one worth recognising in the wild:

- An **application** uses **PKCS#11** against a local HSM for its own signing or TLS operations — because it needs the operation, in-process, now.
- The **same application** uses **KMIP** to a central key manager for the *key lifecycle* of the data it protects — because rotation, inventory and audit live centrally.
- A **storage array** uses **KMIP** only, and never touches PKCS#11, because it does its own crypto in silicon and needs only the key.
- A **secrets manager** (see [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md)) may use **PKCS#11 to seal its own root key** to an HSM *and* separately expose a **KMIP server** to third parties. Same product, two different uses of two different standards, and the seal path has nothing to do with the KMIP path.

**The design question that resolves most confusion:** *is this component asking to **use** a key, or asking to **manage** one?* Use → local API (PKCS#11). Manage → network protocol (KMIP). A component doing both is two integrations, and should be documented as two.

## 8. The Key-Management Discipline Underneath It

### 8.1 The Discipline KMIP Is a Transport For

The framing of this entire section, and the sentence to take away: **KMIP is the transport; no protocol enforces a lifecycle — only processes and audit do.** The specification hands you a state machine, a set of attributes and a vocabulary. It does not rotate your keys, does not notice a missed rotation, does not require dual control, and does not generate evidence an auditor will accept. All of that is **discipline**, and its authoritative anchor is not OASIS but **NIST**.

**The standards anchor:** **NIST SP 800-57 Part 1 Revision 5, *Recommendation for Key Management: Part 1 – General*, published May 2020** ✅ (csrc.nist.gov/pubs/sp/800/57/pt1/r5/final). This is where the *cryptoperiod* concept, the key-types taxonomy, the key hierarchy and the lifecycle states are actually specified — and note that **KMIP's own state machine explicitly cites SP 800-57-1 as its source** (§3.4) ✅ (Spec v2.1 §4.57). **Status note, flagged honestly: an initial public draft of SP 800-57 Part 1 Revision 6 was posted on 5 December 2025 with a comment period through 5 February 2026.** ⚠ — Rev. 6 remains a draft, and **Rev. 5 (May 2020) is the current published version**; do not cite Rev. 6 as a standard.

### 8.2 Lifecycle States and the Cryptoperiod

**The cryptoperiod** is a defined period of time during which a key is authorised for use, and it is the *only* mechanism that makes rotation a scheduled activity rather than an incident response ✅ (NIST SP 800-57 Pt 1 Rev 5). It is defined per key type and per use, because the right cryptoperiod for a session key is minutes and for a root signing key it may be years.

| KMIP state | Lifecycle phase (SP 800-57 vocabulary) | What the *process* must do | Integrity |
|---|---|---|---|
| **Pre-Active** | Pre-activation | Generate ahead of need; distribute; do not use | ✅ |
| **Active** | Active | Use within the cryptoperiod; monitor usage | ✅ |
| **Deactivated** | Deactivated / process-only | Stop applying protection; permit verification/decryption only | ✅ |
| **Compromised** | Compromised | Stop protection; investigate; re-key the population | ✅ |
| **Destroyed / Destroyed Compromised** | Destroyed | Remove material; retain audit evidence | ✅ |
| *(no KMIP state)* | **Post-operational / archive period** | Retain beyond use because old data must still be decrypted | ⚠ (this is *outside* the KMIP state machine's six states and lives in archival policy) |

**The row that catches people out is the last one.** A deactivated key is not a disposable key: data encrypted five years ago still needs it. The distinction between *deactivation* (stop protecting new data) and *destruction* (remove the key) is where organisations destroy keys they still need, and then discover it during a restoration test. KMIP gives you `Archive` and `Recover` operations for exactly this ✅ (Spec v2.1 §6.1.4, §6.1.42) — using them is a process decision.

### 8.3 The Key Hierarchy and the Envelope Model

Keys protect keys. The canonical hierarchy, in SP 800-57 vocabulary, is **root/master key → key-encrypting keys (KEKs) → data-encrypting keys (DEKs)**, with each layer's key protecting the layer beneath it ✅ (NIST SP 800-57 Pt 1 Rev 5, key types). This is **envelope encryption**, and its practical consequences for a KMIP deployment are concrete:

- **Only the top of the hierarchy needs the strongest custody.** The DEK can be wrapped and stored next to the data; the KEK can live in the key manager; the root key should be in hardware, and ideally under ceremony (§8.5). This is why a storage array using KMIP for its key-encrypting keys is a *sound* design, not a shortcut — the array's own data keys are wrapped by a key the array must fetch.
- **A hierarchy gives you rotation economics.** Re-keying a DEK forces re-encryption of data; re-keying a KEK requires only re-wrapping DEKs. **Agility lives at the KEK and root layers, not the DEK layer**, and a design that only allows DEK rotation is a design that cannot migrate an algorithm cheaply (§10).
- **Two-person integrity becomes expressible.** If the root key is itself wrapped under a split key or a threshold scheme — which KMIP can express with a **Split Key** object and `Create Split Key` / `Join Split Key` ✅ (Spec v2.1 §2.8, §6.1.10, §6.1.27) — no single custodian holds a usable root. Whether anyone actually uses it that way is the process question.

### 8.4 Separation of Duties, Split-Knowledge and Dual-Control

Three related controls, frequently written down as if they were the same thing, and distinguished here deliberately:

- **Separation of duties** — the roles *key custodian*, *key user*, *key auditor* and *security administrator* are held by different people. In KMIP terms: the authorisation layer of the server decides who may `Get`, who may `Revoke`, and who may `Destroy`; the protocol provides the operations, not the policy. **The strictest reading — that a custodian who can use a key must not also be able to export it — is enforced by server configuration, and this is a *convention* rather than something the specification mandates.** ⚠
- **Split-knowledge** — the credentials or key shares needed to activate a key are divided among individuals such that no single person can reconstruct them, classically **M-of-N**. KMIP's **Split Key** object model is the protocol primitive; the *M-of-N* and *who holds which share* decisions are governance. ⚠ (mechanism is standard; the use is convention)
- **Dual-control** — two authorised people must be physically or logically present for a sensitive operation (root key loading, activation at ceremony). This is the control audits actually test in payment environments (§9.5), and it is **a real regulatory expectation, not merely a convention** — though the specific requirement numbering needs care (§9.3). ⚠→✅ depending on the regime

**Say plainly what KMIP does and does not contribute here:** KMIP can *express* a split key and can *log* an operation; it cannot make two humans stand in the same room, and no protocol can. The gap between "the system supports M-of-N" and "the institution actually requires M-of-N and can evidence it" is the entire content of the control.

### 8.5 The Key Ceremony as Governance

**A key ceremony is a governance event, not a technical operation.** It is a scripted, witnessed, minuted, evidenced process for generating, loading, backing up or destroying key material of the highest sensitivity — classically a root key or a CA signing key. What makes it a ceremony is not the crypto; it is the **evidence**:

| Element | What it provides | Integrity |
|---|---|---|
| A written **script** | Reproducibility; no improvised steps; reviewable in advance | ⚠ (convention/best practice) |
| Named **witnesses** independent of the operators | Attestation that the script was followed | ⚠ (convention) |
| **Dual-control** throughout | No single-actor exposure | ⚠ → ✅ per regime (§9) |
| **Audit records** from the HSM and the key manager | Machine evidence to complement human attestation | ⚠ (convention) |
| **Signed attestation** by participants, retained | What an auditor can actually read months later | ⚠ (convention) |
| **Sealed backups** of the wrapped key shares, with their own custody rules | Recovery capability that does not weaken custody | ⚠ (convention) |

**The distinguishing test:** a key ceremony that produced no durable, independent, signed record **did not happen**, whatever was physically done in the room. A KMIP deployment makes this *easier* — the server logs the `Create`, the `Activate` and the lineage — but the log is supporting evidence for the ceremony, not a substitute for it. **This is the point at which key management stops being engineering and becomes assurance.**

### 8.6 Audit and Attestation

What an auditor will ask, in the shape they will ask it, and where KMIP helps:

| Question | Where the answer lives | Does KMIP help? | Integrity |
|---|---|---|---|
| What keys exist, of what type, protecting what? | Key manager inventory, built from `Name`/`Object Type`/custom attributes | ✅ (Locate/Get) — but only for *that* server's objects | ✅ |
| Who accessed which key, when, and what did they do? | Server audit log of operations | ✅ (operations are auditable) | ✅ |
| Is the key material in a validated module? | FIPS 140-3 validation certificate and the module's security policy | ❌ (nothing to do with KMIP) | ✅ |
| Were a key's lifecycle transitions actually performed on schedule? | Rotation job logs, dated `Re-key`/`Revoke`/`Destroy` records, and the *absence* of gaps | ⚠ (protocol records the transition; nothing forces the schedule) | ⚠ |
| Was dual control observed? | Ceremony records, HSM audit, physical access logs | ⚠ (governance, not protocol) | ⚠ |
| Are the keys inventoried across *all* estates? | Nothing, usually — see §9.7 | ❌ | ✅ |

**The honest summary of this section:** everything in the left column is a **process artifact**. KMIP makes some of them *easier to produce* and none of them *automatic*. If you take one sentence from §8 into your next review, take this one: **a KMIP deployment is a protocol implementation; a key-management *programme* is an inventory, a set of cryptoperiods, a role model, a ceremony practice and an audit trail — and the protocol is the smallest part of it.**

## 9. The Regulated-Payments and Banking Angle

### 9.1 Where Key Management Becomes an Obligation

In a bank, key management is not a security *choice*; it is a compliance *requirement*, and the requirements are specific in a way that general IT security frameworks are not. Four regimes matter, and they matter differently:

| Regime | What it governs | Where it bites in a key estate | Integrity |
|---|---|---|---|
| **FIPS 140-3** (US/Canada module validation) | The *cryptographic module* — does this device do crypto correctly and securely, at a validated level? | Determines which HSMs and which key managers are *acceptable* for regulated workloads | ✅ |
| **PCI PIN Security Requirements** (PCI SSC) | PIN management, key management for PIN encryption/translation, HSM controls | Governs the payment-HSM estate and its key ceremonies | ✅ (standard exists) / ⚠ (requirement numbers — §9.3) |
| **PCI DSS** | Cardholder-data environment, including key-management requirements for stored data | Governs the keys protecting cardholder data at rest and in transit | ✅ |
| **PCI 3DS / PCI TSP** and the various scheme-mandated key-management rules | 3-D Secure, token service providers | Niche but strict; keys must live in specified module types | ⚠ (not verified this pass) |

**The structural point, and the one that reframes the whole guide:** in a regulated institution, **key material for payments, signing and encryption must often live in dedicated tamper-resistant hardware *because the regulation says so*, not because the threat model demands it.** A well-designed software key manager with strong operational controls might be *technically* adequate for a given workload and *prohibited* for that workload. Protocol choice follows the module choice, not the other way around — which is why "can we replace the HSM with a KMIP server?" is usually the wrong question, and "which components in this estate are regulatory-obliged to be hardware, and what protocol do they speak?" is the right one.

### 9.2 The Payment-HSM Estate

This is a bounded cross-reference, not a re-derivation: the payment-HSM estate is owned elsewhere in this repository. **The SWIFT HSM and CSP context belongs to [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md); the NETS payment-HSM estate belongs to [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md).** Both guides treat the HSM as a *component fact* — the thing that holds keys and performs PIN block and MAC operations. Neither treats key management as a discipline, and neither needs to; that is this guide's remit.

What a payment-HSM estate looks like from *this* guide's angle: it is a **separate key estate**, with its own modules, its own ceremonies, its own vendors, its own audit regime and its own change-control constraints — and it typically speaks **not KMIP but the HSM vendor's native API plus PKCS#11**, plus scheme-specific protocols. That matters because it is the single largest reason the "one key estate" ambition fails (§9.7).

### 9.3 PIN Security and the Card-Data Standards — Handle the Numbers Carefully

**What is verifiable this pass:** **PCI PIN Security Requirements v3.1** exists and is published by the PCI Security Standards Council ✅ (PCI SSC). **Dual-control and split-knowledge are genuine PCI PIN controls** covering sensitive key operations, and the PIN-security standard is where the payment industry's key-ceremony expectations are actually written down ✅ (as a control area, confirmed).

**What is NOT verified, and must be flagged:** a third-party summary (governancedocs.com) states that PCI PIN v3.1 contains *"7 control objectives, 33 requirements, 145 sub-requirements"*. **I could not confirm those counts against the PCI SSC PDF this pass. Do NOT state those numbers as fact.** ⚠ **The disciplined position: name the standard and version, describe the control (dual-control and split-knowledge for sensitive key operations), cite by name, and verify the requirement number against the PCI SSC document itself before quoting one in an audit response.** Quoting a requirement number from a secondary source into a regulator-facing document is a small error with a large blast radius, and it is entirely avoidable.

**The same discipline applies to PCI DSS's key-management requirements.** The *substance* — key generation in a secure environment, secure key distribution, key changes at defined cryptoperiods, split knowledge and dual control for manual clear-text key operations, key destruction when no longer needed, and documented key-management procedures — is stable and well known. ✅ (substance) / ⚠ (requirement numbers, version-specific wording). **Cite the substance with the standard's name and version, and cite the number only from the source.**

### 9.4 The Module-Validation Regime: What FIPS 140-3 Does and Does Not Guarantee

**FIPS 140-3 is the current US and Canadian standard for cryptographic-module validation** ✅ (csrc.nist.gov/pubs/fips/140-3/final), superseding FIPS 140-2. It validates a **module**, at a **security level from 1 to 4**, against a published security policy. What that means, stated precisely because it is the most commonly over-read certificate in the industry:

| What FIPS 140-3 validation **does** establish | What it **does not** establish | Integrity |
|---|---|---|
| That *this module version* meets the requirements for *this level* | That the **system** built around the module is secure | ✅ |
| That the module's cryptographic algorithms and self-tests behave as specified | That the **deployment** is correct — key handling, config, network exposure, IAM | ✅ |
| That the module resists specified physical/logical attack classes at higher levels | That the **key lifecycle** is managed — rotation, destruction, inventory, ceremony | ✅ |
| That someone independent (the lab, and CMVP) reviewed the evidence | That **KMIP**, or any protocol, is involved at all | ✅ |

**Three practical consequences for a key-management programme:**

1. **A validated module behind an unvalidated *system* is the normal state of affairs, and it is not necessarily wrong** — but the residual risk lives in the system, and it must be assessed there. The certificate is a floor, not a roof.
2. **Validation is version-bound and level-bound.** "FIPS 140-3 validated" without the module name, version and level is an incomplete claim, and a firmware upgrade can move you off the validated version. This is a real operational obligation, not a pedantic one.
3. **KMIP is orthogonal to validation.** A KMIP-conformant server backed by a FIPS 140-3 Level 3 HSM and a KMIP-conformant server backed by a file are *both* KMIP-conformant, and the certificate is what distinguishes them — not the protocol.

### 9.5 What an Auditor Actually Tests

The controls an auditor will sample in a key-management review, and where the evidence comes from. This is the practical shape of §8:

| Control under test | The evidence they will ask for | KMIP's contribution | Integrity |
|---|---|---|---|
| **Dual control on sensitive key operations** | Ceremony records, HSM audit logs showing two distinct authenticators, physical access logs | ⚠ (governance; the HSM logs, not KMIP) | ✅ (control is real) |
| **Split knowledge** | Key-component custody records; who holds which share; M-of-N assignments | ⚠ (Split Key object exists; custody is process) | ⚠ |
| **Key inventory completeness** | A register of keys, their purpose, owner and cryptoperiod | ✅ partially — KMIP `Locate`/`Get` for one server | ✅ |
| **Rotation on the documented cryptoperiod** | Dated rotation evidence with no unexplained gaps | ⚠ (the operation is logged; the schedule is not enforced) | ⚠ |
| **Key destruction with retained audit** | Destruction records; and the `Destroyed Compromised` state retaining compromise status | ✅ (state exists, purpose is audit) | ✅ |
| **Retirement of superseded algorithms** | Evidence that a defined algorithm set is enforced, not merely preferred | ⚠ (attribute enforcement depends on server policy) | ⚠ |
| **Roles and least privilege on the key manager** | Role definitions; who can `Export`; who can `Destroy` | ⚠ (server authorisation, not protocol) | ⚠ |

**The pattern in that table is the guide's thesis in miniature:** the *machine* contributes the operations and the dated records; the *institution* contributes the schedule, the roles, the ceremony and the evidence. An auditor is testing the second column.

### 9.6 Hardware Because the Regulation Says So

Restating §9.1 in its sharpest form, because it changes procurement behaviour:

- **The module obligation precedes the protocol decision.** If a workload's regulation requires a validated, tamper-responsive module, then the key manager for that workload is whatever wraps that module — and KMIP may or may not be its interface.
- **The threat model and the compliance model are not the same model, and both must be satisfied.** A threat-model-first engineer will reach for the cheapest sufficient control; a compliance-first auditor will reach for the regulated control. In a bank, **the auditor's model is a constraint, not an opinion**, and the engineering conversation should stop trying to win it and start designing inside it.
- **This has a cost consequence that should be stated openly in any business case:** hardware roots of trust, validated modules, ceremony processes and independent witnesses are expensive, and they are expensive *because the regulation requires a specific, evidenced control*, not because the cryptography is hard. §11.8 puts illustrative numbers on this for Cymbal Bank.

### 9.7 The Disconnected Key Estates Problem

This is the cross-domain observation that explains why KMIP's promise is undersold in practice, and it is worth stating as a standalone finding.

**One institution usually runs several disconnected key estates, with no shared inventory.** The pattern is remarkably consistent across banks:

| Estate | Typical key holder | Typical interface | Who owns it |
|---|---|---|---|
| **Payments / PIN** | Payment HSMs | Vendor API + PKCS#11 + scheme protocols | Payments engineering / card services |
| **SWIFT / interbank messaging** | Dedicated HSMs (see [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md)) | Vendor API + CSP | Financial-messaging operations |
| **General IT / PKI** | CA HSMs, sometimes soft CAs | PKCS#11, CA APIs | Infrastructure / security |
| **Storage & backup at rest** | External KMS, or the array's own store | **KMIP** | Storage / infrastructure |
| **Cloud workloads** | Cloud KMS, with a cloud-native external-key API | **Not KMIP** (§6.4) | Cloud platform |
| **Code signing & release integrity** | Signing HSMs, code-signing services | PKCS#11 / vendor tooling | Engineering / supply chain |
| **Digital-asset custody** | MPC or native-custody wallets | Wallet APIs | **A different trust model entirely** (§7.3) |

**There is no shared inventory across those columns.** Ask a large bank for a list of every cryptographic key it depends on and you will get seven lists, from seven teams, in seven formats, three of which are "we'd have to check with the vendor". **That is the actual state of the art, and it is the finding that makes every "unify key management with KMIP" slide deck look naive.**

**Why KMIP rarely unifies them — four reasons, in descending order of force:**

1. **Boundary ownership beats protocol elegance.** Each estate has a team, a budget line, a change-control regime and a regulator or scheme relationship. Nobody's incentive is to hand their key custody to a shared service that will then become their outage.
2. **Regulatory regimes differ and do not compose.** A payments key and a general-IT key are not governed by the same rules, and a unified estate must satisfy the *strictest* applicable control for every workload it absorbs. Consolidation therefore means *raising* the bar on the general-IT keys, which is expensive and slow, not lowering the payments bar, which is not permitted.
3. **HSM vendor lock is real and physical.** The payment HSM's keys often cannot be exported in usable form, by design. That is a *feature* of the module, and it forecloses the tidy migration the slide proposes.
4. **KMIP integration is deep and is rarely touched once working.** A storage array's KMIP integration, once certified and in production, is not something anyone re-opens. The standard's benefit is realised on the *second* migration — and the second migration is often a decade away, by which time the integration is folklore and the engineer who built it has left. **This is the quiet reason standards under-deliver in this domain: the switching cost is paid up front and the benefit is collected by a future you cannot invoice.**

**The defensible conclusion:** KMIP can unify key *interfaces* where the estates genuinely share a protocol need — storage, tape, backup and application lifecycle across a mixed vendor set is a real win. It does **not** unify the *inventory*, the *governance*, or the *module obligation*, and any programme claiming it will should be asked what happens to the payment estate. §11 walks this through.

## 10. Crypto-Agility and the Post-Quantum Problem

### 10.1 Why This Is a Key-Management Problem

**Migration to new algorithms is largely a key-management exercise, not a coding exercise.** Change the algorithm and the code still runs; what changes is the *key* — its type, its size, its wrapping, its storage format, its backing hardware, and its relationship to every other key in the hierarchy. Concretely, an algorithm migration requires:

- **Algorithm identifiers** that can express the new algorithm (KMIP's `Cryptographic Algorithm` attribute, extended in 3.0 — §10.2).
- **Key sizes and formats** the store and the wrapping layer can carry — and note that post-quantum keys are large: the 3.0 spec explicitly handles the case where the public and private keys of an ML-KEM/ML-DSA/SLH-DSA pair have **different `Cryptographic Length` values**, *"reflecting the natural sizes of the underlying mathematical material"* ✅ (Spec v3.0 §4.15).
- **An inventory** that tells you where the affected keys actually are — the step that fails first, because §9.7 says it does not exist.
- **The ability to re-key at scale** — which is precisely what `Re-key`, `Re-key Key Pair` and `Derive Key` are for ✅ (Spec v2.1 §6.1.46, §6.1.47, §6.1.14).
- **Hardware that can perform the new algorithm** — and this is where crypto-agility gets physically hard (§10.5).

**This is KMIP's strongest genuine argument, and it is worth stating without hype:** a protocol that expresses algorithms as attributes, exposes re-key and derive operations, and separates key lifecycle from key use is *architecturally* the right shape for algorithm migration. Whether your deployment is agile depends on whether anyone exercised that shape — see §12's "agility claim never exercised".

### 10.2 The Standards, By Name and Date

**NIST finalised the first three post-quantum cryptography standards in August 2024** ✅:

| Standard | Algorithm | Role | Integrity |
|---|---|---|---|
| **FIPS 203** | **ML-KEM** (Module-Lattice-Based Key-Encapsulation Mechanism) | Key establishment / KEM | ✅ (csrc.nist.gov/pubs/fips/203/final) |
| **FIPS 204** | **ML-DSA** (Module-Lattice-Based Digital Signature Algorithm) | Digital signatures | ✅ (csrc.nist.gov/pubs/fips/204/final) |
| **FIPS 205** | **SLH-DSA** (Stateless Hash-Based Digital Signature Algorithm) | Digital signatures (hash-based) | ✅ |

**KMIP 3.0's response** ✅ (Spec v3.0 Appendix C and §6.1):

| Change | Working draft and date | What it enables |
|---|---|---|
| FIPS 203/204/205 algorithm definitions added | **WD16, 23 August 2024** | Naming ML-KEM, ML-DSA and SLH-DSA as KMIP algorithms — the precondition for managing them |
| **Encapsulate** and **Decapsulate** operations added | **WD17, 17 September 2024** | The KEM operations a post-quantum key exchange actually needs; defined at §6.1.22 and §6.1.15 with their own parameter and error tables |
| Quantum Safe authentication suite and Quantum Safe profiles | **WD07, 24 February 2025** | A conformance claim for post-quantum-capable KMIP endpoints; **Quantum Safe Server at §6.33 / §5.16** |
| PQC items updated *"to enable AVCP-level testing via KMIP"* | **WD19, 14 February 2025** | Automated Cryptographic Validation Protocol testing of post-quantum algorithms **through KMIP** — i.e. you can *prove* the algorithms behave correctly, over the protocol, the same way FIPS 140-3 module validation tests algorithms |
| **PKCS#11 Client / Server profiles** | Profiles CSD02 §5.18, §6.34–§6.35 | A KMIP endpoint that can present PKCS#11 semantics (see §7.4) |

### 10.3 Harvest-Now-Decrypt-Later, Honestly

**State its evidence status carefully, in both directions, because this is where the field's marketing is loudest.**

- **The threat model:** an adversary records encrypted traffic now, and decrypts it later once a cryptographically relevant quantum computer exists. For data with a long confidentiality requirement — a decade or more — the *effective* protection of today's traffic depends on the migration happening before that machine exists.
- **Evidence status, stated honestly:** this is **a plausible and widely-held threat model with limited public confirmed instances**. ⚠ Migration urgency is a *risk-management judgement* made under uncertainty, not an observation. There is no public, verified case of a large-scale harvest-now-decrypt-later programme attributed to a specific actor that this pass could confirm.
- **Timelines are forecasts, not findings.** Any statement of the form "quantum computers will break RSA by year X" is a **FORECAST**, produced by an expert or a vendor, and should be labelled as such in any document that carries it. ⚠ **Do not launder a forecast into a finding by stripping the hedge.**
- **What *is* defensible without any timeline at all:** the migration takes years, the inventory step fails first, the affected hardware cannot be upgraded in place, and the work is cheaper started than finished. **Those four propositions justify beginning the work without asserting any quantum date** — and that is the honest business case.

### 10.4 What Crypto-Agility Concretely Requires

Strip away the marketing and agility is seven specific capabilities. Test your estate against all seven; most fail at least three:

1. **A complete inventory** of keys with algorithm, length and purpose — across *all* estates (§9.7 says this is the hardest). ⚠
2. **Algorithm as a first-class, changeable attribute**, not a compile-time constant. KMIP supports this ✅ (`Cryptographic Algorithm`).
3. **A hierarchy that supports re-wrapping without re-encrypting data** — the KEK/DEK separation (§8.3). ⚠ (design-dependent)
4. **Re-key operations at scale**, exercised in anger, not merely available ✅ (`Re-key`, `Re-key Key Pair`).
5. **Hardware that supports the target algorithms**, or a migration path to hardware that does (§10.5). ⚠
6. **Protocol-level algorithm negotiation** so that a client and server can agree on what they both support — the Quantum Safe profiles and authentication suite (§10.2). ⚠ (draft)
7. **A tested cutover runbook** — the thing that turns the other six from capabilities into an executed migration. ⚠ (process)

**Note what is missing from that list: any requirement to know the quantum timeline.** Agility is valuable regardless of when the threat arrives, because the same capabilities are what you need for *any* algorithm retirement — including the entirely terrestrial ones (3DES→AES, SHA-1→SHA-256, RSA-2048→RSA-4096, ECDSA curve changes). **The strongest argument for crypto-agility is that you have already needed it several times, historically, for reasons that had nothing to do with quantum computing.**

### 10.5 The Hardware That Cannot Be Upgraded In Place

The honest constraint, and the reason post-quantum migration is not a software release:

- **HSMs are validated as modules at specific versions** (§9.4). A module that does not implement ML-KEM cannot be made to implement ML-KEM by a configuration change; it needs new firmware — which moves it off its validation — or new hardware.
- **Key material inside a tamper-resistant module is often deliberately non-exportable.** That is the *point* of the module, and it means the migration is not "copy the keys to the new box"; it is "generate new keys on the new box and re-establish trust", which is a ceremony (§8.5) and therefore a project.
- **Wrapped key backups, smart cards, key components and offline shares all carry the old algorithm** and all need re-issuing.
- **The KMIP estate is the *easier* half of this problem**, because at least the protocol can name the new algorithms and express the new lengths (§10.2). The hardware estate is where the timeline actually lives.

**The practical conclusion for a regulated institution:** the post-quantum migration is a **key-management programme with a hardware-replacement sub-project inside it**, and the highest-value thing you can do in 2026 is not to buy PQC-capable hardware — it is to **build the inventory, name the cryptoperiods, and exercise a re-key at scale**. That work is worth doing for its own sake, and it is the precondition for every PQC timeline anyone forecasts. ⚠ (recommendation, not a standard)

## 11. The Cymbal Bank Worked Example

> **Every figure and finding in this section is fictional and illustrative.** Cymbal Bank is this repository's only bank persona. Nothing here is a benchmark, a survey result, or a claim about any real institution. The *shape* of the exercise is the deliverable.

### 11.1 The Trigger and the Estate

**The trigger (illustrative).** A Cymbal Bank internal audit finding lands on the architecture desk: *"The bank cannot produce a complete inventory of cryptographic keys in use across its estate, and cannot demonstrate that key rotation occurs on a documented cryptoperiod for any estate other than payments."* It is not a breach finding. It is a *governance* finding, which is worse — because there is no incident to point at and no immediate fix to ship.

**The estate, as discovered (illustrative, and deliberately untidy):**

| Estate | Approx. keys (illustrative) | Interface today | Owner | Rotation evidence |
|---|---|---|---|---|
| Payment HSMs (PIN, MAC) | ~4,800 | Vendor API + PKCS#11 | Payments engineering | ✅ documented, audited, ceremonial |
| SWIFT messaging HSMs | ~900 | Vendor API + CSP | Financial-messaging ops | ✅ documented |
| Internal PKI (CA + issuing) | ~1,200 | PKCS#11 + CA API | Infrastructure security | ⚠ partial |
| Storage / backup at rest | ~15,000 | **mixed: some KMIP, some array-local** | Storage | ⚠ mostly unverified |
| Cloud workloads | ~9,000 | **cloud KMS, cloud-native external-key API** | Cloud platform | ⚠ provider-managed, no bank-side inventory |
| Code signing / release | ~60 | PKCS#11 + build tooling | Engineering | ⚠ partial |
| Total | **~31,000** | **seven models** | **six teams** | **two of seven complete** |

**Read the table again for what it is:** ~31,000 keys, seven interfaces, six owners, and complete rotation evidence in two estates. That is the finding, and it is the honest starting condition of nearly every institution that has not run this exercise.

### 11.2 Inventory Before Protocol

**The first recommendation, and the one that has nothing to do with KMIP: the inventory precedes the protocol.** KMIP cannot tell you what keys exist outside its own server (§8.6, §9.7), so a KMIP programme that starts before an inventory starts from a false floor. The sequence:

1. **Enumerate by estate, not by key.** You cannot list 31,000 keys; you *can* list the seven estates, and within each, the key *types* and their purpose. That gets you to 90% of the governance value in weeks rather than years.
2. **For each key type, record:** purpose, owner, algorithm and length, cryptoperiod (or the absence of one), backing module, interface, and backup/escrow arrangement. This is the register the auditor asked for, and it is owner-supplied, not tool-extracted.
3. **Then** ask where a *protocol* could reduce the cost of maintaining that register.

**Why this ordering matters:** every failed key-management programme this guide can describe started with a technology selection and then tried to fit the estate to it. The register is the cheap part and the part the regulator can actually read.

### 11.3 Protocol-Fit Analysis, Estate by Estate

The core analytical work: **where is KMIP the right tool, where is it the wrong tool, and where is it irrelevant?** Note that two of the seven estates are explicitly *not* KMIP candidates, and saying so plainly is the value of the exercise.

| Estate | KMIP fit | Reasoning | Verdict |
|---|---|---|---|
| **Storage / backup at rest** | ✅ **Strong** | ~15,000 keys, already partly KMIP, mixed array vendors, at-rest encryption with no application coupling — the textbook case (§5, §6.3) | **Consolidate onto KMIP** |
| **General IT / PKI** | ✅ **Good, scoped** | Private keys can be KMIP-managed and centrally inventoried; CA operations remain with the CA. The `Asymmetric Key Lifecycle` profile is the relevant claim | **Adopt for lifecycle, not for CA operations** |
| **Code signing** | ⚠ **Partial** | Key *lifecycle* fits (few keys, high sensitivity, real governance need); the *signing* operation is latency-sensitive and build-pipeline-coupled, and will stay on PKCS#11 against a local module | **Hybrid: KMIP lifecycle + PKCS#11 use** (§7.5) |
| **Cloud workloads** | ❌ **Wrong tool** | The cloud KMS does not speak KMIP and uses a cloud-native external-key API (§6.4). Forcing KMIP here means running the cloud's own translation path or a proxy, for no governance gain | **Out of scope — manage via cloud-native controls** |
| **Payment HSMs** | ❌ **Wrong tool** | Regulatory module obligations (§9.1, §9.4), scheme-specific protocols, veteran HSM APIs, and keys that are non-exportable by design. KMIP would be an additional protocol on top of a working, audited estate | **Out of scope — leave it alone** |
| **SWIFT messaging HSMs** | ❌ **Wrong tool** | Same reasoning, and owned by a different team with a different regulator relationship — see [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) | **Out of scope** |
| **Digital-asset custody** | ❌ **Not applicable** | MPC — a different trust model, not a KMIP client (§7.3); owned by [../banking/fireblocks_guide.md](../banking/fireblocks_guide.md) | **Out of scope** |

**The headline of the analysis, and the finding worth taking away: KMIP is the right protocol for two of seven estates, a hybrid candidate for one, and the wrong tool for four.** A programme that had begun with "let us unify key management on KMIP" would have spent its first year failing to absorb the payments estate — the largest and most sensitive column — and would have been stopped. **Scoping KMIP to where it fits is what makes it adoptable.**

### 11.4 Profile Requirements

With the scope narrowed to three estates, the conformance demand becomes specific for the first time in the programme's life. This is §5.4 applied. The illustrative procurement sentence Cymbal Bank writes:

> *"The proposed key-management server SHALL declare, in writing: the KMIP protocol version(s) supported; the conformance profile(s) claimed by number and name; the authentication suite(s) supported; and any attribute, operation, algorithm or key-format-type exclusions within each claimed profile — with a dated interoperability test result or Statement of Use covering the claimed version."*

And the demand is then made concrete against the actual consumers:

| Consumer (illustrative) | Profile it needs | Why |
|---|---|---|
| Storage arrays (mixed vendors) | **Storage Array with Self-Encrypting Drives**; **Symmetric Key Lifecycle** | SED key custody and AES key rotation |
| Backup appliances | **Symmetric Key Lifecycle**; **Baseline Server** | Data-at-rest key fetch and rotation |
| Application integrations (internal PKI) | **Asymmetric Key Lifecycle**; **Cryptographic** (for server-side operations where desired) | Private-key lifecycle and certification |
| Optional cryptographic offload | **Cryptographic** profile | Only where in-process latency permits — assessed per integration |
| Post-quantum readiness (planned) | **Quantum Safe Server** (3.0 draft) | Forward requirement, not a 2026 acceptance criterion — flagged as draft (§4.3) |

**Note that the v2.1 profile names and the v3.0 numbering differ (§5.3), so the procurement sentence must specify *which* version's profile taxonomy it is naming.** This is exactly the kind of ambiguity that produces a two-month argument with a vendor after signature, and it is avoided by writing the version in the sentence.

### 11.5 Interoperability Testing Before Commitment

**The rule: test the actual client against the actual server, in the actual profile, before signing, and test it with the *rotation* operation, not just Create/Get.** The illustrative test plan:

| Test | What it proves | If it fails |
|---|---|---|
| Version negotiation (`Discover Versions`) | The endpoints agree on a protocol version at all | Negotiate down or reconsider |
| Profile operation set — the operations the array actually calls | The array's five to ten operations exist and behave | Reject the bid or scope down |
| **Rotation in anger — `Re-key` / re-register of a live key, with the array continuing to read data** | The lifecycle actually works end to end | **This is the test that matters and the one most often skipped.** A pilot that only creates and fetches keys has tested nothing about rotation |
| Authentication suite exchange | The suites intersect | Fix config, or accept the risk explicitly |
| **Failure behaviour — kill the key server mid-operation** | What the array does when key fetch fails: continues on cached keys, or stops I/O? | Sizing the availability dependency — **the single most important result, and it belongs in the resilience register, not just the security one** |
| Custom-attribute portability | Whether any vendor extension leaked into the integration | Re-work the integration or accept lock-in consciously |

**The fifth row deserves emphasis in any real programme:** a key-management server is now a **runtime dependency of every client that fetches keys on demand**. Whether a client caches keys (and for how long), and what it does when the fetch fails, determines whether a five-minute key-server outage is a monitoring blip or a storage outage. **This is an availability question wearing a cryptography costume, and it should be tested as one** — the discipline of deliberately injecting that failure belongs to [chaos_engineering_guide.md](chaos_engineering_guide.md), and the *blast-radius reasoning* is the same.

### 11.6 Governance and Operational Implications

| Implication | Illustrative Cymbal position | Integrity |
|---|---|---|
| **New single point of dependency** | The consolidated key manager becomes a Tier-1 service with its own RTO/RPO and DR test | ⚠ (design decision) |
| **Role model** | Custodian / operator / auditor / admin roles defined and separated in the server, then *evidenced* quarterly | ⚠ (convention, per §8.4) |
| **Ceremony scope** | Root keys under ceremony; DEK rotation automated. **Ceremony is not applied to 15,000 storage keys** — that would be unaffordable and would fail | ⚠ (proportionality judgement) |
| **Cryptoperiod register** | Every key type gets a documented cryptoperiod; the register is the artifact the auditor reads | ✅ (the SP 800-57 concept) |
| **Compromise playbook** | `Revoke`-with-compromised-reason, re-key the population, retain `Destroyed Compromised` evidence | ✅ (state model supports it) |
| **Cloud estate exclusion** | Documented explicitly, with the cloud-native controls that cover it, so the exclusion is *visible* rather than an oversight | ⚠ |
| **Payment estate exclusion** | Documented explicitly, referencing the existing audited controls | ⚠ |

**The deliberate inclusion of two exclusions in the governance document is a design choice worth copying.** An estate map that silently omits the payments and cloud estates reads as an oversight to an auditor and as ignorance to a regulator. An estate map that names them, and explains why they are governed differently, reads as competence.

### 11.7 Migration Sequencing and Risk

**Sequence by risk, not by size, and never migrate two things at once.** The illustrative five-wave plan:

| Wave | Scope | Why this order | Rollback |
|---|---|---|---|
| **1. Inventory + register** | All seven estates; no technology change | The prerequisite, and it is genuinely useful on its own | N/A |
| **2. Pilot** | One non-critical storage array; one backup appliance; full rotation test | Proves the profile claim and the failure behaviour on something that can break | Revert to array-local keys |
| **3. Storage consolidation** | Remaining arrays and backup, wave by wave | The strong-fit estate; amortises the integration work | Per-array, during a maintenance window |
| **4. PKI lifecycle** | Private-key lifecycle only; CA operations untouched | Smaller blast radius if the role model is wrong | Keep the old store read-only |
| **5. Code signing (hybrid)** | Lifecycle only; signing stays on PKCS#11 | Lowest operational coupling, highest sensitivity | Revert to previous signing module |

**Risks, with the mitigation that actually works:**

| Risk | Why it is real | Mitigation |
|---|---|---|
| **The key server becomes a single point of failure for I/O** | Clients fetch keys at runtime (§11.5) | Test the failure path *first*; cache policy as a first-class design decision; include in DR tests |
| **Profile claim turns out narrower than needed** | §5.4 — the word "KMIP" hides a five-operation implementation | Demand the profile sentence *and* the exclusion list before signature |
| **Vendor lock-in via custom attributes** | §3.3 — custom attributes are by definition non-portable | Forbid vendor-proprietary attributes in the integration; test with a second server |
| **The migration is irreversible in practice** | Once arrays depend on the server, decommissioning it means re-keying everything | Accept explicitly, or sandbox the pilot long enough to be sure. **Do not pretend a rollback plan exists beyond the pilot** — §12 covers this |
| **The inventory drifts immediately** | Keys are created by systems, not by the register | Automate register updates from server queries (`Locate`/`Get Attributes`) where the estate is KMIP; accept manual for the rest |

### 11.8 Illustrative Cost Comparison

> **All figures below are fictional and illustrative.** They are a *shape* for a business case — order-of-magnitude and relative — not a quotation, not a benchmark, and not derived from any real procurement.

| Cost element (illustrative, 3-year) | Option A — status quo (per-estate key stores) | Option B — consolidate storage + PKI onto KMIP |
|---|---|---|
| Key-management servers / licences | ~SGD 140k (fragmented, some bundled with arrays) | ~SGD 320k (consolidated, enterprise tier) |
| Integration & professional services | ~SGD 60k (incremental, unmanaged) | ~SGD 480k (up-front; the big line — *and the one that is one-off*) |
| **Conformance & interoperability testing** | ~SGD 10k | ~SGD 90k (explicitly budgeted — §11.5) |
| Operations (FTE, ~0.6 → ~1.4) | ~SGD 420k | ~SGD 320k after consolidation (later waves) |
| Ceremony & audit effort | ~SGD 150k | ~SGD 180k (broader scope, more evidence) |
| Training | negligible | ~SGD 60k |
| **3-year total** | **~SGD 780k** | **~SGD 1.45M** |
| **Inventory completeness** | 2 of 7 estates | 4 of 7 estates (payments, SWIFT, cloud excluded by design) |
| **Rotation evidence** | partial in 2 estates | complete in 4 estates |
| **Vendor optionality** | low | medium-high in the two consolidated estates |

**Read the comparison honestly, because it does not favour the standard:** **Option B costs roughly 1.9× Option A over three years.** The business case is *not* "KMIP saves money" — on a three-year view it does not, for this estate, because the integration cost is front-loaded and the operational savings arrive in waves four and five. **The business case is: it is the only option that produces a complete rotation-evidence story in four estates and a defensible architecture for the next algorithm migration, and the alternative is a permanent audit exception.** Whether that is worth SGD ~670k is a governance decision, not an engineering one, and it should be made by the people who own the audit finding.

### 11.9 The Recommendation, and What Could Not Be Established

**The illustrative recommendation:**
1. **Do the inventory now**, independently of any protocol decision — it is the deliverable the audit finding actually asks for.
2. **Adopt KMIP for storage/backup consolidation and PKI key lifecycle**, on a narrowed scope that explicitly excludes the payment, messaging, cloud and digital-asset estates.
3. **Make profile-level conformance an acceptance criterion**, with rotation and failure behaviour tested before signature (§11.5).
4. **Do not claim crypto-agility until a re-key at scale has been exercised** (§12).
5. **Report the two exclusions and the cost honestly** to the audit committee, on the reasoning in §9.7: KMIP unifies interfaces, not inventory, governance or module obligation.

**What could not be established from vendor documentation, and therefore went into the programme as open items** (this list is the honest residue of the exercise, and any real version of it will look similar):
- **Profile-level conformance claims for most commercial key-management servers**, stated as a version/profile/exclusion list (§6.1).
- **The failure and caching behaviour of specific array firmware** when its key server is unreachable — not published, and only discoverable by test.
- **Whether any vendor's KMIP implementation supports the specific rotation semantics** Cymbal Bank's cryptoperiod register requires — an integration test, not a datasheet.
- **The maintenance status of at least one candidate open-source component** (§6.2) — which is why open-source was not on the critical path for the consolidation.
- **Any statement of the post-quantum readiness of the hardware in the payment estate** — which is why §10.5's work is scoped as a separate programme.

## 12. The Gotchas and Anti-Patterns

Nine, in the symptom/cause/guardrail format. Each of these is a failure this guide has either documented above or watched happen in the pattern of the literature.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **"KMIP-compatible" read as "interchangeable"** | A server and a client both claim KMIP support and fail to interoperate on the third operation | The word "KMIP" carries no version, profile or authentication suite (§5.4) | Demand the sentence: *version + profile by number + authentication suite + exclusions*; test before signature |
| 2 | **Assuming a version/profile that was never tested** | Works in pilot, fails in production on an operation nobody exercised — typically rotation | The pilot tested Create/Get; the profile claim was never validated against the operations production uses (§11.5) | Test the operations the **cryptoperiod register** requires, including `Re-key`, not the ones that are easy to demo |
| 3 | **Treating the protocol as the lifecycle** | A KMIP server is deployed and the audit finding about rotation evidence is unchanged a year later | Confusing the state machine with the schedule (§8.1) — the protocol has states, not policies | Separate the *programme* deliverable (cryptoperiods, register, evidence) from the *technology* deliverable (the server) |
| 4 | **A single key estate with no inventory or owner** | Nobody can answer "how many keys do we have?" — and the answer when forced is contradicted by the next team asked | Keys are created by systems; no register is maintained; ownership is assumed rather than assigned (§9.7) | The register of §11.2, owned by a named function, updated on key creation |
| 5 | **A key ceremony with no evidence** | The ceremony happened; six months later nobody can produce a record of who attended or what was loaded | Treating the ceremony as a technical operation rather than a governance event (§8.5) | Script, independent witnesses, signed attestation, retained; machine audit as corroboration, not substitute |
| 6 | **Conflating a module's validation with the system's security** | "We're FIPS 140-3 validated, so we're secure" | Over-reading the certificate (§9.4) — it validates a module at a version and level, not a deployment | State the claim precisely: *module name, version, level*; assess the system separately |
| 7 | **The irreversible migration** | The rollback plan exists on paper and cannot be executed, because 15,000 keys now live in the new store and the old one was decommissioned | Migration sequenced without a reversibility checkpoint (§11.7) | Keep the previous store read-capable until the last wave; treat "irreversible" as an explicit accepted risk, not an oversight |
| 8 | **The agility claim never exercised** | "We can migrate algorithms easily" — and the first real migration takes nine months because nobody had performed a re-key across estates | Crypto-agility asserted as an architectural property rather than demonstrated as a capability (§10.4) | Exercise a re-key at scale **annually, as a drill**, on a real estate. If it has never been done, it does not exist |
| 9 | **Proprietary attributes leaking into the integration** | A migration to a second vendor requires re-integration from scratch | Custom attributes used for load-bearing semantics (§3.3) | Forbid vendor-proprietary attributes in the interface contract; run a second server in a lab |

**Two cross-references rather than re-derivations, because the sibling guides own them:** the *failure-mode testing* discipline behind #2 and #7 — deliberately breaking the key path to observe what happens — is the method of [chaos_engineering_guide.md](chaos_engineering_guide.md), and the *secrets-manager* patterns behind #4 and #9 (dynamic secrets, leases, seal/unseal, the PKCS#11 seal path) belong to [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md). This guide does not re-derive either.

## 13. The Claims Audit

Every factual claim of consequence in this guide, with its source quality and the date of the finding. **Highest-risk classes: version history, profile contents, each implementation's status, every cloud-service claim, and every regulatory requirement number.**

| Claim | Status | Source (quality) | Date |
|---|---|---|---|
| KMIP 1.1 + Profiles 1.1 became OASIS Standards **12 Feb 2013**; KMIP 1.4 + Profiles 1.4 **27 Nov 2017** | ✅ | OASIS press releases and announcement text, KMIP TC page (primary) | 2013/2017 |
| KMIP 2.1 + Profiles 2.1 = OASIS Standard, **14 December 2020**; CS01 07 May 2020. KMIP 2.0 + Profiles 2.0 = OASIS Standards (2019) | ✅ 2.1 dates / ✅ 2.0 status, ⚠ 2.0 exact date | Spec v2.1 header; TC page + announcement link (primary) | 2019/2020 |
| KMIP **3.0 CSD02 dated 7 May 2026**; Profiles CSD02 **21 May 2026**; public review **14 Jul–13 Aug 2026**; **not** an OASIS Standard | ✅ | Spec/Profiles v3.0 CSD02; OASIS invitation-to-comment page (primary) | 2026 |
| KMIP 1.0 = OASIS Standard 2010; 1.2 = 2014; 1.3 = 2015 | ⚠ years flagged, exact months not re-confirmed | TC documents tree (primary, partial) | 2010–2015 |
| From v1.3, a Statement of Use requires plugfest/interop participation | ✅ as stated in the fact base | OASIS KMIP TC practice (primary) | 2026 pass |
| v2.1 client-to-server operation set = the 57 operations listed in §3.1 | ✅ | Spec v2.1 §6.1 TOC (primary) — counted | 2026 pass |
| Additions: FIPS 203/204/205 (WD16, 23 Aug 2024); Encapsulate/Decapsulate (WD17, 17 Sep 2024); ACVP-level PQC testing (WD19, 14 Feb 2025); Object Class Mask (WD11); Counters + client UUID (WD13) | ✅ | Spec v3.0 Appendix C revision history (primary) | 2024–2025 |
| Quantum Safe profiles + auth suite (WD07, 24 Feb 2025); Quantum Safe Server profile §5.16/§6.33 | ✅ | Profiles v3.0 CSD02 (primary) | 2025–2026 |
| PKCS#11 Client/Server profiles §5.18.4/§5.18.5, conformance §6.34/§6.35, tests PKCS11-M-1-30 | ✅ | Profiles v3.0 CSD02 (primary) | 2026 |
| System objects User/Group/Credentials added in 3.0; Obliterate added; Lease Time mandatory | ✅ | Spec v3.0 Appendix C; §2.1 (primary) | 2026 |
| Six managed-object states; **spec explicitly cites SP 800-57-1** as their source | ✅ | Spec v2.1 §4.57 (primary) | 2020 pass |
| TTLV is the mandatory encoding; XML and JSON via profiles; default port **5696**; content types pinned | ✅ | Spec v2.1 §10.1; Profiles v2.1 §5.3 (primary) | 2026 pass |
| v2.1 profile catalogue as listed in §5.2 (Baseline → PKCS#11 §5.18); auth suites §3.1/§3.2 | ✅ | Profiles v2.1 §3–§5 (primary) | 2026 pass |
| **Vault KMIP secrets engine implements KMIP 1.4** profiles: Baseline Server, Symmetric Key Lifecycle Server, Basic Cryptographic Server, Asymmetric Key Lifecycle Server, Advanced Cryptographic Server — with named exclusions; **requires Vault Enterprise + ADP**; listener defaults to 5696 | ✅ | developer.hashicorp.com/vault/docs/secrets/kmip (vendor primary, extracted) | 2026 pass |
| PyKMIP: 983 commits, 299 stars, 166 forks, Apache-2.0 ✅ — but **maintenance/release status unresolved ⚠**; KMIP4J maintenance and any Barbican KMIP back-end status also **not verified ⚠** | ⚠ mixed | github.com/OpenKMIP/PyKMIP (repo, extracted); no last-release date obtained | 2026 pass |
| **AWS KMS does not speak KMIP**: XKS uses a proxy that *"translates generic requests from AWS KMS into a vendor-specific format"*; *"AWS KMS never interacts directly with your external key manager"*; proxy is customer-owned, one per key store | ✅ | docs.aws.amazon.com KMS Developer Guide (vendor primary, extracted) | 2026 pass |
| Azure Key Vault / Managed HSM uses Azure REST; Google Cloud KMS Cloud EKM uses a proprietary EKM API — **neither speaks KMIP** | ✅ | Microsoft and Google primary docs, as characterised in the fact base | 2026 pass |
| KMIP 3.0 PQC interop test: **1452 tests**, **>1,000,000 messages**, **3–7 March 2025**, participants **Cryptsoft, NetApp, P6R**; **the TC does not independently verify results** | ✅ | kmip-interop.org/background.html (interop site, primary for the event) | 2025 |
| NIST PQC standards finalised **Aug 2024**: FIPS 203 (ML-KEM), 204 (ML-DSA), 205 (SLH-DSA) | ✅ | NIST/CSRC (primary) | 2024 |
| NIST SP 800-57 Pt 1 Rev 5 published **May 2020**; Rev 6 initial public draft posted **5 Dec 2025**, comments to **5 Feb 2026**, still a draft | ✅ Rev 5 / ⚠ Rev 6 status | CSRC (primary) | 2020/2025 |
| FIPS 140-3 is the current module-validation standard; levels 1–4; validates a module, not a system | ✅ | CSRC FIPS 140-3 (primary) | 2026 pass |
| PCI PIN Security Requirements **v3.1** exists; dual-control and split-knowledge are genuine PCI PIN controls | ✅ (existence and control) | PCI SSC (primary) | 2026 pass |
| *"7 control objectives, 33 requirements, 145 sub-requirements"* in PCI PIN v3.1 | ⚠ **NOT CONFIRMED** — third-party summary only; **do not cite** | governancedocs.com (third-party) | 2026 pass |
| PKCS #11: v2.40, v3.0 OASIS Standard, v3.1 at CSD01 | ✅ | OASIS PKCS#11 documents tree (primary) | 2026 pass |
| KMIP TC participants / profiles-acknowledgement companies | ✅ as listed | Profiles acknowledgements; TC page (primary) | 2026 pass |
| Profile-level conformance for most commercial KMIP servers | ⚠ not published in verifiable form | — | 2026 pass |
| Storage/backup consumer KMIP support (ObjectScale, StorageGRID) | ✅ as this repo's own guides state / ⚠ profile-level | dell_objectscale_guide.md; storagegrid_guide.md | repo pass |
| PowerProtect, IBM Storage KMIP specifics | ⚠ not verified | — | 2026 pass |
| Market share / adoption share of KMIP | ❌ no verified basis found — **not asserted** | — | 2026 pass |
| Harvest-now-decrypt-later as a *confirmed* large-scale campaign | ⚠ plausible threat model, **limited public confirmed instances** — not asserted as observed | — | 2026 pass |
| Any quantum-cracking timeline | ⚠ **FORECAST**, not a finding — labelled as such throughout | industry forecasters | 2026 pass |
| All Cymbal Bank figures (key counts, SGD costs, FTE) | ⚠ **explicitly fictional and illustrative** | this guide | 2026 pass |
| Every `web_search` used this pass | ⚠ returned empty result sets on some queries (rate-limited); **an empty search is not evidence of absence** | — | 2026 pass |

## 14. What Could Not Be Verified

Stated plainly, in the spirit of the rest of this guide — this is the honest residue.

1. **The KMIP 3.0 approval timeline.** CSD02 is confirmed (7/21 May 2026) and the public review window is confirmed (14 July–13 August 2026). **Whether the TC will advance 3.0 to Committee Specification, to a vote, and on what schedule, is not something I could establish** — no public roadmap was found this pass. ⚠
2. **Exact dates for KMIP 1.0 (month), 1.2 and 1.3, and for the 2.0 OASIS Standard approval.** Years are confirmed from the OASIS documents tree; the specific dates are flagged rather than guessed. This guide does not invent a date. ⚠
3. **Profile-level, test-case-level conformance for every commercial key-management server.** Vault publishes a model statement (§6.2); Thales, Utimaco, Entrust, Fornetix, Fortanix, QuintessenceLabs and others do not publish profile-plus-exclusion statements in a form reachable this pass. **This is the single largest evidence gap in the field**, and it is the reason §5.4's procurement sentence exists. ⚠
4. **The maintenance status of PyKMIP, KMIP4J and any OpenStack Barbican KMIP back end.** Repository metadata for PyKMIP is confirmed (commits/stars/forks); **last release dates were not obtained**, and no release cadence is asserted. Do not read this as abandonment — it is an absence of evidence. ⚠
5. **PCI PIN Security Requirements v3.1's requirement counts** (`7 / 33 / 145` from a third-party summary) — **not confirmed against the PCI SSC PDF; deliberately not asserted.** Any requirement number quoted in an audit response must come from the PCI SSC document. ⚠
6. **Any load-bearing market-adoption figure for KMIP.** Empty `web_search` results this pass, and no primary adoption survey was reached. **An empty search is not evidence of absence, and no adoption claim is made in either direction.** ⚠
7. **The concrete caching and failure behaviour of array and backup firmware when the key server is unreachable.** Not published by any vendor reached this pass, and — as §11.5 argues — only discoverable by test. This is the item that most often decides whether a KMIP project is safe. ⚠
8. **Whether the KMIP TC's own conformance test cases are publicly reusable as a certification suite** (`kmip-interop.org` and the profiles' mandatory test-case lists exist; **the licensing and process for third-party use were not established**), and **the post-quantum capability of specific HSMs** or any vendor firmware roadmap — which is why §10.5 is framed as a programme rather than a product recommendation. ⚠
9. **The `web_search` tool's reliability this pass.** Several queries returned **zero results**; the substantive work was done from cached primary OASIS documents, vendor primary pages, and `web_extract` against primary URLs. **Readers should treat the `⚠` items above as the places to re-verify, not as settled negatives.** ⚠

## 15. Glossary

| Term | Definition |
|---|---|
| **Attribute** | A named, typed property of a KMIP managed object (algorithm, length, usage mask, state, dates). In KMIP, the attributes are the substance and the operations are the verbs. |
| **Baseline Client / Server** | The floor profiles: the minimum operations and attributes a conformant KMIP endpoint must support. |
| **Cryptoperiod** | The defined period during which a key is authorised for use — the concept that makes rotation scheduled (NIST SP 800-57 Pt 1 Rev 5). |
| **DEK / KEK / envelope encryption** | Data-encrypting key and key-encrypting key — the lower and middle layers of the hierarchy in which each layer's key protects the one beneath (§8.3). |
| **Dual control** | Requiring two authorised persons for a sensitive key operation. |
| **FIPS 140-3** | The current US/Canadian cryptographic-module validation standard; validates a module at a level (1–4). |
| **HSM** | Hardware security module — tamper-resistant hardware holding keys and performing crypto inside its boundary. |
| **KMIP** | Key Management Interoperability Protocol — the OASIS standard network protocol this guide is about. |
| **Managed object** | A thing the KMIP server manages: a key, certificate, secret data, opaque data, template or split key. |
| **MPC** | Multi-party computation — a custody trust model in which no single party holds the whole key (see [../banking/fireblocks_guide.md](../banking/fireblocks_guide.md)). |
| **Obliterate** | A KMIP 3.0 operation removing key traces (metadata, backups) beyond simple destruction, subject to server policy. |
| **PKCS#11** | The OASIS *local* cryptographic token API — an application links it and uses keys in-process. |
| **Profile** | A named, bounded, testable subset of the KMIP specification, with published mandatory and optional test cases. |
| **Split key** | A KMIP object type representing key material divided by a scheme (e.g. XOR, polynomial) and reconstructable only by joining. |
| **TTLV** | Tag-Type-Length-Value — KMIP's mandatory binary encoding, designed for low-function clients. |
| **Unique Identifier** | The server-assigned handle by which an object is addressed in every subsequent operation. |

## 16. Cross-References and Further Reading

**Inside this repository — the guides whose boundaries this one respects:**

- [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) — **owns** the secrets-manager comparison and the PKCS#11 *seal* path. Note that Vault's KMIP *secrets engine* is a different capability from its PKCS#11 seal; §6.2 here gives the KMIP-side conformance detail and defers the comparison.
- [chaos_engineering_guide.md](chaos_engineering_guide.md) — **owns** the deliberate-failure method this guide invokes in §11.5 (testing what a client does when its key server disappears).
- [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) — **owns** the SWIFT HSM and CSP context.
- [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) — **owns** the NETS payment-HSM estate.
- [../banking/fireblocks_guide.md](../banking/fireblocks_guide.md) — **owns** MPC key management; §7.3 here contrasts the trust model in one paragraph and defers.
- [../banking/tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) and [../banking/fapi_financial_grade_api_guide.md](../banking/fapi_financial_grade_api_guide.md) — **own** their private-key requirements.
- [oracle_database_guide.md](oracle_database_guide.md) (TDE key custody), [container_certificates_guide.md](container_certificates_guide.md), [w3c_dids_guide.md](w3c_dids_guide.md) — **own** their certificate and key-material slices.
- [dell_objectscale_guide.md](dell_objectscale_guide.md) and [storagegrid_guide.md](storagegrid_guide.md) — the consumers that put KMIP into this repository as a feature row; §1.3 and §6.3 here give what the row means.

**Primary sources, for the reader who wants to check any of this:**

- *KMIP Specification v2.1* (OASIS Standard, 14 Dec 2020) and *KMIP Profiles v2.1* — `docs.oasis-open.org/kmip/kmip-spec/v2.1/os/` and `/kmip-profiles/v2.1/os/`.
- *KMIP Specification v3.0* CSD02 (7 May 2026) and *KMIP Profiles v3.0* CSD02 (21 May 2026) — the drafts carrying the post-quantum and PKCS#11 work; plus the OASIS KMIP TC home page for membership and version announcements.
- `kmip-interop.org` (the March 2025 PQC interoperability test record); NIST SP 800-57 Part 1 Rev. 5 (May 2020); NIST FIPS 140-3; FIPS 203/204/205.
- AWS KMS Developer Guide, *External key stores* (the XKS proxy architecture, and the evidence AWS KMS does not speak KMIP); HashiCorp Vault documentation, *KMIP secrets engine* (a rare published profile-level conformance statement).

## 17. Closing Summary

**KMIP is a network protocol for asking a key-management server to create, store, find, use, rotate and destroy cryptographic objects — and that sentence is the whole of what it guarantees.** Everything else that a key-management programme needs is elsewhere: the state machine is in the specification and the *schedule* is in a register; the conformance is in a profile and the *version, profile and authentication suite* are what make "supports KMIP" a claim anyone can test; the module validation is FIPS 140-3 and it validates a module rather than a system; the inventory is a document no protocol can write for you; and the ceremony is a governance event whose only proof is a signed record.

The three findings this guide would most want carried forward: **the current OASIS Standard is 2.1 while 3.0 sits in public review carrying the post-quantum work** — so demand 2.1 conformance today and a 3.0 roadmap in writing; **the major cloud key services do not speak KMIP at all**, reaching external key managers through their own proprietary APIs and a proxy you own; and **KMIP is the right protocol for storage, tape, backup and application key lifecycle, and the wrong tool for the payment-HSM estate, the cloud estate and MPC custody** — which is a scoping conclusion, not a criticism. One institution will keep running several disconnected key estates whatever protocol it adopts, because the boundaries are organisational and regulatory, not technical.

Test the rotation before you sign. Exercise the re-key before you claim agility. Write down the exclusions. And remember that in every architecture this guide describes, the interesting question is never which protocol — it is **who holds the key.**
