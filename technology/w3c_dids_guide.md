# W3C DIDs — the persistent identifier

**The DID-Ecosystem Deep-Dive — the W3C DID Core 1.0 standard (Recommendation, 19 July 2022), the DID document data model, DID URLs and representations, the DID method ecosystem (did:key, did:web, did:jwk, did:ethr, did:sov, did:indy) and its registries, DID resolution and the Universal Resolver, DIDComm, the self-sovereign-identity stack, and a Cymbal Bank did:web issuance worked example — every research claim verified at the primary sources this pass.**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research · Identity Cluster — the Decentralized-Identifier deep-dive of the repository's Web3/identity stack; the sibling guides carry the surrounding mechanics that this guide cross-references instead of re-deriving
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [web3_technologies_guide.md](web3_technologies_guide.md) (the Web3 umbrella — its §8.1 maps DIDs and verifiable credentials into the broader owned-web stack; cross-ref for the ecosystem view) · [verifiable_credentials_guide.md](verifiable_credentials_guide.md) (the VC Data Model deep-dive — **expected sibling, not yet in the repo at the time of writing**; this guide forward-references it as `[verifiable_credentials_guide.md]` and does not re-derive the VC data model) · [smart_contracts_guide.md](smart_contracts_guide.md) (the EIP/ERC standards process and the Ethereum platform — cross-ref for the on-chain mechanics behind ledger-based DID methods such as did:ethr) · [distributed_auth_guide.md](distributed_auth_guide.md) (authn/authz in distributed systems — the identity-architecture twin that DID/VC plugs into) · [cybersecurity_guide.md](cybersecurity_guide.md) (audit practice and security context — cross-ref for the condensed §10) · [blockchain_technology_guide.md](blockchain_technology_guide.md) (ledger mechanics, for the verifiable-data-registry layer)
> **Companion guides (banking/, prefix `../banking/`):** [private_banking_guide.md](../banking/private_banking_guide.md) (the private-banking deep-dive — its §9.1 KYC/AML and §11 Cymbal Bank onboarding worked example carry the onboarding conventions this guide's §9.3/§11 cross-reference) · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the SG/MAS regulatory regime and the Cymbal Bank persona conventions) · [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) (institutional tokenization twin)
> **Related research folders:** the repo's [singapore/](../singapore/) directory currently holds four guides (sg_gdp_industry_distribution.md, singapore-government-securities-guide.md, singaporean_overseas_retirement.md, starhub_software_systems_guide.md) and **no digital-identity guide** — verified this pass, so no such cross-reference exists to make.

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, why a bank should care, and the explicit relationship to the sibling guides. Section 2 is the standard, verified at the primary source: W3C DID Core 1.0's Recommendation status and date, its editors, what it normatively defines, the core terminology, and the `did:method:method-specific-id` syntax. Section 3 is the DID document — the core properties (`id`, `controller`, `verificationMethod`, the five verification relationships, `service`), verification material, and a complete worked document. Section 4 is the DID URL — path, query, fragment, DID parameters, relative DID URLs. Section 5 is representations — JSON and JSON-LD as the two representations DID Core defines, and how production/consumption works. Section 6 is the DID method ecosystem — what a method is, the registries (the old community registry and its W3C Group-Note successor), and the major methods (did:key, did:web, did:jwk, did:ethr, did:sov, did:indy) each with its verified public facts and honest governance flags. Section 7 is what DID Core itself requires of methods — the method-syntax and CRUD-operation requirements. Section 8 is resolution and infrastructure — DID resolution and DID URL dereferencing, the resolution spec's governance journey (CCG draft → W3C Candidate Recommendation Draft), the Universal Resolver, and DIDComm. Section 9 is the self-sovereign-identity stack and use cases, cross-referencing the sibling guides rather than re-deriving them. Section 10 is the privacy/security overlay, condensed and cross-referenced. Section 11 is the Cymbal Bank worked example — a bank-issued `did:web` design, clearly marked design fiction built on the verified facts of the guide. Section 12 is the claims audit (✅/⚠/❌) with a "What Could Not Be Verified" subsection. Section 13 is the glossary. Section 14 is the cross-reference map and the closing summary. **Reading paths:** *Banking/architecture audience:* §1 → §2 → §9 → §10 → §11 → §14. *Engineer/developer:* §2 → §3 → §5 → §6 → §8 → §11. *Standards/compliance:* §2 → §6 → §7 → §8.3 → §12. *In a hurry:* §1, §6, §8, §11, §12, and the glossary.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a primary or authoritative source (named in the claims audit, §12.1); **⚠** flagged — reported, approximate, single-sourced, fast-moving, or contested (governance statuses of community-run infrastructure in particular move faster than documents); **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what a public key is, what TLS does) rather than research claims. Where a sibling guide carries a verified fact, this guide marks it ✅ and cites the sibling rather than re-verifying from scratch — the repo's convention is that sibling-verified facts are inherited.

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why a Bank Should Care: The Cymbal Bank Lens](#13-why-a-bank-should-care-the-cymbal-bank-lens)
   - 1.4 [How This Guide Relates to the Siblings](#14-how-this-guide-relates-to-the-siblings)
2. [The Standard: W3C DID Core 1.0](#2-the-standard-w3c-did-core-10)
   - 2.1 [The Recommendation, Verified](#21-the-recommendation-verified)
   - 2.2 [What the Standard Contains](#22-what-the-standard-contains)
   - 2.3 [The Terminology: DID, DID Document, DID URL, Subject and Controller](#23-the-terminology-did-did-document-did-url-subject-and-controller)
   - 2.4 [The Syntax: did:method:method-specific-id](#24-the-syntax-didmethodmethod-specific-id)
3. [The DID Document](#3-the-did-document)
   - 3.1 [The Core Properties](#31-the-core-properties)
   - 3.2 [Verification Methods and Verification Material](#32-verification-methods-and-verification-material)
   - 3.3 [Verification Relationships](#33-verification-relationships)
   - 3.4 [Services](#34-services)
   - 3.5 [A Complete DID Document Example](#35-a-complete-did-document-example)
4. [The DID URL](#4-the-did-url)
   - 4.1 [Path, Query, Fragment](#41-path-query-fragment)
   - 4.2 [DID Parameters](#42-did-parameters)
   - 4.3 [Relative DID URLs](#43-relative-did-urls)
5. [The Representations](#5-the-representations)
   - 5.1 [Production and Consumption](#51-production-and-consumption)
   - 5.2 [JSON: application/did+json](#52-json-applicationdidjson)
   - 5.3 [JSON-LD: application/did+ld+json](#53-json-ld-applicationdidldjson)
   - 5.4 [Other Representations](#54-other-representations)
6. [The DID Methods](#6-the-did-methods)
   - 6.1 [What a DID Method Is](#61-what-a-did-method-is)
   - 6.2 [The Registries: From Community Lists to W3C Group Notes](#62-the-registries-from-community-lists-to-w3c-group-notes)
   - 6.3 [The Major Methods Compared](#63-the-major-methods-compared)
   - 6.4 [did:key — The Ledgerless Public-Key Method](#64-didkey--the-ledgerless-public-key-method)
   - 6.5 [did:web — The Domain-Based Method](#65-didweb--the-domain-based-method)
   - 6.6 [did:jwk — The JWK Method](#66-didjwk--the-jwk-method)
   - 6.7 [did:ethr — The Ethereum ERC-1056 Method](#67-didethr--the-ethereum-erc-1056-method)
   - 6.8 [did:sov — The Sovrin Method, Governance Flagged](#68-didsov--the-sovrin-method-governance-flagged)
   - 6.9 [did:indy — The Hyperledger Indy Method, Governance Flagged](#69-didindy--the-hyperledger-indy-method-governance-flagged)
   - 6.10 [Choosing a Method](#610-choosing-a-method)
7. [The Method Requirements in DID Core](#7-the-method-requirements-in-did-core)
   - 7.1 [Method Syntax Requirements](#71-method-syntax-requirements)
   - 7.2 [Method Operations: Create, Read, Update, Deactivate](#72-method-operations-create-read-update-deactivate)
   - 7.3 [Security and Privacy Requirements](#73-security-and-privacy-requirements)
8. [Resolution and the Infrastructure](#8-resolution-and-the-infrastructure)
   - 8.1 [DID Resolution](#81-did-resolution)
   - 8.2 [DID URL Dereferencing](#82-did-url-dereferencing)
   - 8.3 [The Resolution Spec's Governance Journey](#83-the-resolution-specs-governance-journey)
   - 8.4 [The Universal Resolver](#84-the-universal-resolver)
   - 8.5 [DIDComm](#85-didcomm)
9. [The SSI Stack and the Use Cases](#9-the-ssi-stack-and-the-use-cases)
   - 9.1 [The Self-Sovereign Identity Model](#91-the-self-sovereign-identity-model)
   - 9.2 [The Verifiable-Credentials Sibling (Forward Reference)](#92-the-verifiable-credentials-sibling-forward-reference)
   - 9.3 [Banking and Identity Use Cases, Condensed](#93-banking-and-identity-use-cases-condensed)
   - 9.4 [The Singapore Context](#94-the-singapore-context)
10. [Privacy and Security Considerations](#10-privacy-and-security-considerations)
    - 10.1 [The Privacy Properties](#101-the-privacy-properties)
    - 10.2 [The Security Considerations](#102-the-security-considerations)
    - 10.3 [What This Means for a Bank](#103-what-this-means-for-a-bank)
11. [The Cymbal Bank Worked Example: A Bank-Issued DID](#11-the-cymbal-bank-worked-example-a-bank-issued-did)
    - 11.1 [The Design-Fiction Frame](#111-the-design-fiction-frame)
    - 11.2 [Choosing did:web for the Bank](#112-choosing-didweb-for-the-bank)
    - 11.3 [The Bank's DID Document](#113-the-banks-did-document)
    - 11.4 [The Verifiable-Credential Issuance Flow](#114-the-verifiable-credential-issuance-flow)
    - 11.5 [The Operational and Risk Overlay](#115-the-operational-and-risk-overlay)
12. [The Claims Audit](#12-the-claims-audit)
    - 12.1 [The Verified-Facts Table](#121-the-verified-facts-table)
    - 12.2 [What Could Not Be Verified](#122-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and the Closing Summary](#14-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**A Decentralized Identifier (DID) is a globally unique persistent identifier that does not require a centralized registration authority, is often generated and/or registered cryptographically, and resolves — through its DID method — to a DID document: a small data structure carrying the cryptographic keys (verification methods) and service endpoints that let the identifier's controller prove control and interact with the world.** The definition is the spec's own ✅ (verified at w3.org/TR/did-core, terminology section, this pass). The architecture in one line: **DIDs are URIs that associate a DID subject with a DID document, allowing trustable interactions associated with that subject** — the spec's abstract ✅. DIDs are designed so the controller can prove control over the identifier "without requiring permission from any other party", decoupled from centralized registries, identity providers, and certificate authorities ✅ (spec abstract).

Why "the persistent identifier"? Because the entire point of the DID layer is that the identifier outlives any single issuer, platform, wallet, certificate authority, or ledger vendor. A DID does not contain personal data and does not embed a pointer owned by a third party: it is a stable key under which cryptographic material and endpoints can rotate, be published, or be revoked, while the identifier itself stays constant. In the W3C stack the DID is the *identifier half* of verifiable identity; the credential half — the claims issued against the DID — is the Verifiable Credentials (VC) data model, covered by the sibling guide `[verifiable_credentials_guide.md]` (expected, not yet in the repo; §9.2).

The ecosystem has three governance layers, and it is essential to keep them separate:

1. **The W3C standards layer** — DID Core 1.0 is a **W3C Recommendation published 19 July 2022** ✅ (verified at w3.org/TR/did-core: status line "W3C Recommendation, 19 July 2022"; this version REC-did-core-20220719). The VC Data Model is likewise a W3C standard. This layer is stable, royalty-free, and endorsed by W3C for wide deployment ✅.
2. **The registry layer** — DID methods (the `key`, `web`, `ethr`… parts of the syntax) are defined in their own specifications and listed in registries. The old community-run registry pages are being superseded by W3C Group Notes published by the DID Working Group ("DID Extensions", "DID Methods") — Group Notes are *not* standards and do not endorse any method ✅ (verified at w3.org/TR/did-extensions-methods: "This registry does not act as an endorsement… by the W3C").
3. **The method-governance layer** — each method lives or dies with its own operator: `did:sov`'s governing foundation was **dissolved in 2025** with the ledger reduced to a read-only archive ✅ (verified at sovrin.org), while `did:indy` continues under the Linux Foundation Decentralized Trust ✅, and the ledgerless methods (`did:key`, `did:jwk`, `did:web`) have no registry at all to fail. Method governance is the part a bank must diligence; §6 and §12 carry the flags.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
| --- | --- | --- |
| W3C DID Core 1.0 status | W3C Recommendation, 19 July 2022 (REC-did-core-20220719) | ✅ w3.org/TR/did-core |
| Editors of DID Core 1.0 | Manu Sporny (Digital Bazaar), Amy Guy (Digital Bazaar), Markus Sabadello (Danube Tech), Drummond Reed (Evernym/Avast) | ✅ w3.org/TR/did-core (the expected list of six names proved wrong — Amy Guy is an editor; Dave Longley, Orie Steele and Christopher Allen are listed as *authors*; Grant Noble does not appear — §12.1) |
| DID syntax | `did:method-name:method-specific-id` (ABNF: `did = "did:" method-name ":" method-specific-id`) | ✅ DID Core §3.1 |
| DID document | A set of data describing the DID subject — verification methods, relationships, services; `id` is the only required property | ✅ DID Core §4–5 |
| Representations DID Core defines | JSON (`application/did+json`) and JSON-LD (`application/did+ld+json`) | ✅ DID Core §6.2–6.3 |
| DID methods registry (current) | "DID Methods" W3C Group Note, 12 August 2026 — ≈265 entries (machine-counted this pass); explicitly not an endorsement | ✅ w3.org/TR/did-extensions-methods |
| DID method registries (historical) | DID Specification Registries (W3C Note lineage) → republished as "Decentralized Identifier Extensions", Group Note 11 December 2025 | ✅ w3.org/TR/did-spec-registries (redirects), w3.org/TR/did-extensions |
| DID resolution spec | Former W3C CCG draft → now "DID Resolution v1", W3C Candidate Recommendation Draft, 28 August 2026 | ✅ w3.org/TR/did-resolution (⚠ governance note §8.3) |
| Universal Resolver | DIF project (decentralized-identity/universal-resolver); driver-based resolution across many methods; DIF-hosted instance at dev.uniresolver.io | ✅ github.com/decentralized-identity/universal-resolver |
| DIDComm | Open protocol for decentralized, DID-rooted messaging; didcomm.org + DIF DIDComm WG; **not** a W3C standard | ✅ didcomm.org, identity.foundation (⚠ v1/v2 version details §8.5) |
| Ecosystem scale at DID Core publication (July 2022) | 103 experimental DID method specifications, 32 experimental driver implementations, 46 implementations in the conformance test suite | ✅ DID Core status section |

### 1.3 Why a Bank Should Care: The Cymbal Bank Lens

For a bank (the repo's persona is **Cymbal Bank**, a MAS-licensed bank in Singapore — see [private_banking_guide.md](../banking/private_banking_guide.md) §11 and the [MAS guide](../banking/mas_regulations_guidelines_guide.md) for the persona conventions), DIDs matter for three reasons:

- **KYC reuse and client self-sovereignty.** A DID is the identifier a client controls; a bank-issued verifiable credential anchored to that DID ("this client passed Cymbal Bank's CDD") can be presented to other institutions without re-running the full onboarding — the onboarding conventions of [private_banking_guide.md](../banking/private_banking_guide.md) §9.1/§11 are the natural issuance workflow. This is the same conclusion the Web3 umbrella guide reaches in its §8.1 and its §11 Cymbal strategy assessment ✅ (inherited from [web3_technologies_guide.md](web3_technologies_guide.md)).
- **Institutional interoperability.** W3C-standard DIDs give the bank a standards-track identifier layer (as opposed to vendor-specific or per-network identifiers), which is what makes credentials portable across counterparties, wallets, and jurisdictions.
- **The governance trap is the real risk.** The *standard* is stable; the *methods* are a sprawl of community and foundation governance of wildly varying quality (a foundation dissolved, ledgers read-only, registries explicitly non-endorsing). A bank's DID strategy is therefore mostly a **method-selection and key-management** strategy, plus a **regulatory** question (AML/FATF linkage, PDPA data protection — cross-ref [cybersecurity_guide.md](cybersecurity_guide.md) §10 of this guide and the Web3 guide's §8.1 regulatory caveat ✅).

§11 works the bank-issued-DID design end to end under the only permitted bank persona, Cymbal Bank.

### 1.4 How This Guide Relates to the Siblings

- **[web3_technologies_guide.md](web3_technologies_guide.md)** is the umbrella: its §8.1 already verified the DID Core Recommendation date/status and the editor list this guide re-verifies in depth, and it maps identity into the owned-web stack. This guide is the DID-ecosystem deep-dive that §8.1's few paragraphs summarize.
- **`[verifiable_credentials_guide.md]`** (expected sibling, **not present in the repo when this guide was written** — checked this pass) will carry the VC Data Model: issuers, holders, verifiers, the trust triangle, and proof formats. This guide covers the DID as the identifier component and only sketches the VC flow (§9.2, §11.4); it does not re-derive the VC data model.
- **[smart_contracts_guide.md](smart_contracts_guide.md)** carries the Ethereum/EVM mechanics behind ledger-based methods such as `did:ethr` (ERC-1056 is a smart-contract registry) — cross-ref its EIP/ERC sections rather than re-deriving contract mechanics here.
- **[distributed_auth_guide.md](distributed_auth_guide.md)** is the classic identity-architecture twin: authn/authz, protocols, session mechanics. DIDs are the decentralised-identifier primitive that plugs into that architecture.
- **[cybersecurity_guide.md](cybersecurity_guide.md)**, [private_banking_guide.md](../banking/private_banking_guide.md), and the singapore/ research folder are cross-referenced in §9–§10 rather than re-derived. The singapore/ folder has **no digital-identity guide** (verified this pass — only the four files listed in the header), so no such reference is made.

## 2. The Standard: W3C DID Core 1.0

### 2.1 The Recommendation, Verified

**"Decentralized Identifiers (DIDs) v1.0 — Core architecture, data model, and representations" is a W3C Recommendation published on 19 July 2022** ✅, verified this pass at https://www.w3.org/TR/did-core/:

- Status line: **"W3C Recommendation — 19 July 2022"**; this version: `https://www.w3.org/TR/2022/REC-did-core-20220719/`; latest editor's draft at w3c.github.io/did-core; an errata exists; implementation report at w3c.github.io/did-test-suite ✅.
- **Editors:** Manu Sporny (Digital Bazaar), Amy Guy (Digital Bazaar), Markus Sabadello (Danube Tech), Drummond Reed (Evernym/Avast) ✅. **Authors:** Manu Sporny, Dave Longley (Digital Bazaar), Markus Sabadello, Drummond Reed, Orie Steele (Transmute), Christopher Allen (Blockchain Commons) ✅. ⚠ Note: the widely-copied "six editors" list (Reed, Sporny, Sabadello, Longley, Allen, Noble) is **not what the Recommendation page shows** — Amy Guy is an editor; Longley, Steele and Allen are authors; Grant Noble appears nowhere on the page. The discrepancy is recorded in §12.1.
- The document was produced by the **W3C Decentralized Identifier Working Group** on the Recommendation track; the status section says "W3C recommends the wide deployment of this specification as a standard for the Web" ✅ and notes, at publication time, **103 experimental DID method specifications, 32 experimental DID method driver implementations, and 46 implementations submitted to the conformance test suite** ✅.
- Related documents named by the Recommendation: *DID Use Cases and Requirements*, *DID Specification Registries*, and the *DID Core Implementation Report* ✅.

### 2.2 What the Standard Contains

DID Core 1.0 normatively specifies ✅ (verified across the Recommendation's sections, this pass):

1. **The DID syntax** (§3.1) — the generic `did` URI scheme as ABNF, which every DID must conform to.
2. **The DID URL syntax** (§3.2) — path, query, fragment, DID parameters, relative DID URLs.
3. **The data model** (§4) — DID documents as ordered maps of entries, with two classes of entries: *properties* (specified in §5) and *representation-specific entries* (such as `@context` in JSON-LD).
4. **The core properties** (§5) — `id`, `alsoKnownAs`, `controller`, `verificationMethod`, the five verification relationships (`authentication`, `assertionMethod`, `keyAgreement`, `capabilityInvocation`, `capabilityDelegation`), and `service`.
5. **The representations** (§6) — deterministic production/consumption rules for **JSON** and **JSON-LD**, with IANA media types `application/did+json` and `application/did+ld+json`.
6. **DID resolution and DID URL dereferencing** (§7) — the abstract function signatures `resolve()` / `resolveRepresentation()` / `dereference()`, their inputs and outputs.
7. **The requirements on DID method specifications** (§8) — method syntax rules and the Create/Read/Update/Deactivate operation definitions (§7 of this guide).
8. **Security and privacy considerations** (§9–§10) — including the famous guidance that DID documents should not carry personal data.

The spec is deliberately *method-agnostic*: "This specification does not presuppose any particular technology or cryptography" ✅ (§1) — a DID method may be built on a distributed ledger, a distributed filesystem, a database, DNS, or nothing at all.

### 2.3 The Terminology: DID, DID Document, DID URL, Subject and Controller

The definitions below are verified verbatim (paraphrased) from the DID Core terminology section ✅:

| Term | Verified definition (DID Core §2) |
| --- | --- |
| **DID** (decentralized identifier) | A globally unique persistent identifier that does not require a centralized registration authority and is often generated and/or registered cryptographically. Many — but not all — DID methods use DLT or another decentralized network. |
| **DID document** | A set of data describing the DID subject, including mechanisms (e.g. cryptographic public keys) the subject or a delegate can use to authenticate itself and prove its association with the DID. May exist in multiple representations. |
| **DID subject** | The entity identified by the DID and described by the DID document. "Anything can be a DID subject: person, group, organization, physical thing, digital thing, logical thing, etc." |
| **DID controller** | An entity that has the capability to make changes to a DID document. A DID might have more than one controller, denoted by the optional top-level `controller` property; a controller might be the subject itself. |
| **DID method** | A definition of how a specific DID method scheme is implemented: the method specification defines the operations by which DIDs and DID documents are created, resolved, updated, and deactivated. |
| **DID URL** | A DID plus optional path, query, and fragment — a network location identifier for a specific resource. |
| **Verification method** | A set of parameters that can be used, together with a process, to independently verify a proof (e.g., a public key verifying a signature). |
| **Verification relationship** | An expression of the relationship between the DID subject and a verification method (e.g., `authentication`). |
| **Service / service endpoint** | Means of communicating with the DID subject via one or more network addresses (e.g., discovery, messaging, VC-repository services). |
| **Verifiable data registry** | A system facilitating creation, verification, updating and/or deactivation of DIDs and DID documents — the method-specific backing store. |

**Subject vs controller is the concept that matters most for a bank.** The *subject* is *what the identifier is about*; the *controller* is *who may change the document*. The spec is explicit that the two may differ — a bank's institutional DID (subject = the bank's public identity) may list a recovery/operations DID as controller; a custody product's DID may have the bank as controller while the client is the subject of the credentials referencing it. Separating the two is also how key-recovery works: the `controller` value authorizes changes, which is distinct from `authentication` (§5.1.2 of the spec, verified ✅).

### 2.4 The Syntax: did:method:method-specific-id

A DID is "a simple text string consisting of three parts: 1) the `did` URI scheme identifier, 2) the identifier for the DID method, and 3) the DID method-specific identifier" ✅ (DID Core §1.1). The formal ABNF ✅ (§3.1):

```
did                = "did:" method-name ":" method-specific-id
method-name        = 1*method-char
method-char        = %x61-7A / DIGIT        ; lowercase a–z, 0–9
method-specific-id = *( *idchar ":" ) 1*idchar
idchar             = ALPHA / DIGIT / "." / "-" / "_" / pct-encoded
```

Examples (from the spec and real methods): `did:example:123456789abcdefghi`, `did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK`, `did:web:example.com`, `did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a`. Method names are lowercase; colons inside the method-specific-id are method-specific (hierarchical namespaces, e.g. `did:web:example.com:user:alice`) ✅.

## 3. The DID Document

The **DID document** is the resource a DID resolves to. It is a set of key/value entries (an ordered map) expressing what the DID subject is, what keys it controls, what those keys may be used for, and how to reach the subject's services. Everything in this section is verified against DID Core §4–§5 ✅.

### 3.1 The Core Properties

DID Core's normative property table (verified ✅) is:

| Property | Required? | Value constraints (verified) |
| --- | --- | --- |
| `id` | **yes** | A string conforming to the DID syntax (§3.1); denotes the DID of the subject when present in the topmost map |
| `alsoKnownAs` | no | A set of URIs asserted to identify the same subject (⚠ equivalence is only as strong as reciprocity — the spec advises independent verification) |
| `controller` | no | A DID string or set of DIDs authorized to make changes to this document |
| `verificationMethod` | no | A set of verification-method maps (each with `id`, `type`, `controller`, verification material) |
| `authentication` | no | A set of verification methods (embedded or referenced) used to authenticate the subject |
| `assertionMethod` | no | Verification methods used to express claims (e.g., issuing VCs) |
| `keyAgreement` | no | Verification methods used to derive encryption material for confidential communication |
| `capabilityInvocation` | no | Verification methods used to invoke cryptographic capabilities (e.g., updating the document) |
| `capabilityDelegation` | no | Verification methods used to delegate capabilities to other parties |
| `service` | no | A set of service maps (each with `id`, `type`, `serviceEndpoint`) |

Only `id` is required. The **subject** is expressed by the `id` property at the top level; the **controller** — an entity authorized to make changes — is expressed by the optional `controller` property, and the controller's own document "SHOULD contain verification relationships that explicitly permit" the relevant methods (verified ✅ §5.1.2). Verification methods may be *embedded* (full map inline) or *referenced* (a DID URL string pointing at a method in the same or another document) ✅ §5.2.2.

### 3.2 Verification Methods and Verification Material

A **verification method** (typically a public key) is expressed as a map with three required properties and one form of verification material ✅ (§5.2, verified):

- `id` — a DID URL (convention: `did:example:123#key-1`, fragment per key);
- `type` — a string naming exactly one verification-method type, SHOULD be registered in the DID registries (examples used across the spec and methods: `Ed25519VerificationKey2020`, `JsonWebKey2020`, `Multikey`, `X25519KeyAgreementKey2019`, `EcdsaSecp256k1RecoveryMethod2020`);
- `controller` — a DID string (a key cannot control itself; the key's controller must be explicit);
- **verification material** — DID Core limits interoperable expression to two properties ✅ (§5.2.1): `publicKeyJwk` (a JWK per RFC 7517 — MUST NOT contain private members such as `d`) and `publicKeyMultibase` (⚠ a multibase-encoded public key; the multibase spec itself "is not yet a standard and is subject to change"). A verification method MUST NOT carry both for the same material.

The `type` determines which cryptographic suite applies; suites such as *JSON Web Signature 2020* and *Ed25519 Signature 2020* are referenced by the spec (their definitions live in the registries) ✅.

### 3.3 Verification Relationships

A **verification relationship** states what a verification method may be used for. The spec is explicit that a key listed only under `authentication` cannot be used for key agreement — "it is up to a *verifier* to ascertain the validity of a verification attempt by checking that the verification method used is contained in the appropriate verification relationship property" ✅ (§5.3). The five relationships defined by DID Core (verified ✅):

| Relationship | Purpose (verified) | Typical use |
| --- | --- | --- |
| `authentication` | How the subject is expected to authenticate — login, challenge-response | Proving control of the DID |
| `assertionMethod` | How the subject expresses claims | **Issuing Verifiable Credentials** (the issuer's signing key) |
| `keyAgreement` | How entities generate encryption material to transmit confidential info to the subject | Establishing encrypted channels (e.g., X25519 key) |
| `capabilityInvocation` | Keys that may invoke a cryptographic capability — e.g., authorization to update the DID document | DID-document writes, protected-API access |
| `capabilityDelegation` | Keys that may delegate a capability to another party | Sub-delegating access |

Revocation is handled by document state, not by a relationship: "If a referenced verification method is not in the latest DID document… then that verification method is considered invalid or revoked" ✅ (§5.3); each method specifies its own revocation mechanics.

### 3.4 Services

`service` entries express ways of communicating with the subject ✅ (§5.4): each service map MUST contain `id` (a URI; no duplicate service ids), `type` (string or set, SHOULD be registered — e.g. `LinkedDomains`, DIDComm messaging endpoints), and `serviceEndpoint` (a URI string, map, or set of URIs). Service types are open-ended: discovery services, agent (DIDComm) services, and — the case a bank cares about — verifiable-credential repository/issuance services. The spec explicitly discourages publishing personal data (social accounts, personal emails) through services for privacy reasons ✅.

### 3.5 A Complete DID Document Example

A condensed, spec-conformant example (assembled from DID Core's own examples, §5; verified ✅ — structure only, keys are the spec's):

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "id": "did:example:123456789abcdefghi",
  "controller": "did:example:123456789abcdefghi",
  "verificationMethod": [{
    "id": "did:example:123456789abcdefghi#keys-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyMultibase": "zH3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
  }],
  "authentication": ["did:example:123456789abcdefghi#keys-1"],
  "assertionMethod": ["did:example:123456789abcdefghi#keys-1"],
  "service": [{
    "id": "did:example:123456789abcdefghi#linked-domain",
    "type": "LinkedDomains",
    "serviceEndpoint": "https://bar.example.com"
  }]
}
```

## 4. The DID URL

A **DID URL** is a network location identifier for a specific resource — it can retrieve representations of the subject, a verification method, a service, a part of the DID document, or other resources ✅ (DID Core §3.2). ABNF: `did-url = did path-abempty [ "?" query ] [ "#" fragment ]` ✅.

### 4.1 Path, Query, Fragment

Verified against the spec ✅:

- **Path** — identical to a generic URI path (`path-abempty`, RFC 3986 §3.3); semantics are method-defined. Example: `did:example:123456/path`.
- **Query** — identical to URI query syntax; elaborated into **DID parameters** (§4.2). Example: `did:example:123456?versionId=1`.
- **Fragment** — identical to URI fragment syntax; used as a **method-independent reference into the DID document** or to an external resource. This is how service endpoints and verification methods are pointed at: `did:example:123#public-key-0` (a verification method in the document), `did:example:123#agent` (a service in the document), and — via parameters — `did:example:123?service=agent&relativeRef=/credentials#degree` (a resource external to the document). Fragment semantics must be interpreted identically across representations for interoperability ✅.

### 4.2 DID Parameters

The query component carries **DID parameters**, which become part of the identifier of the resource ✅ (§3.2.1). The common, method-independent parameters the spec defines (verified ✅):

| Parameter | Meaning (verified) |
| --- | --- |
| `service` | Identifies a service from the DID document by service id |
| `relativeRef` | A relative URI reference to a resource at a service endpoint selected by the `service` parameter |
| `versionId` | A specific version of the DID document to resolve |
| `versionTime` | The DID document valid at a given (UTC) timestamp |
| `hl` | A resource hash (hashlink) adding integrity protection — non-normative |

Example: `did:example:123?versionTime=2021-05-10T17:00:00Z`. The spec's guidance: DID parameters say *what* resource is identified; resolution-option metadata says *how* it is resolved ✅.

### 4.3 Relative DID URLs

A relative DID URL is any URL value in a DID document that does not start with the full `did:<method>:<id>` prefix; it is resolved against the DID as base URI per RFC 3986 §5 ✅ (§3.2.2). Relative references like `"#key-1"` inside `authentication` resolve to `did:example:123456789abcdefghi#key-1` — the mechanism methods use to keep stored documents small, and the reason DID documents routinely reference their own keys and services by fragment only ✅.

## 5. The Representations

A DID document is a *data model*; a **representation** is a concrete serialization of it ✅ (DID Core §6). DID Core defines representations for **JSON and JSON-LD** ✅ — these are the two representations with normative production/consumption rules in DID Core 1.0 — and explicitly allows others (the spec names XML, YAML, CBOR as possibilities) provided each is properly specified with an IANA media type ✅.

### 5.1 Production and Consumption

Verified ✅ (§6.1): serializing the data model to a representation is **production**; parsing a representation back into the data model is **consumption**. Every representation MUST (i) define deterministic production and consumption rules for all data-model types, (ii) be uniquely associated with an IANA-registered media type, and (iii) define fragment-processing rules conformant with DID Core's; SHOULD register in the registries. Conforming producers/consumers serialize/deserialize *all* entries, including unknown ones, losslessly — that is how the data model stays representation-agnostic and conversion between representations is possible.

### 5.2 JSON: application/did+json

The JSON representation (§6.2, verified ✅) maps the data model's maps/lists/sets/datetimes/strings/numbers/booleans to JSON objects/arrays/strings/numbers/booleans with explicit rules (e.g., datetimes as XML-datetime strings normalized to UTC, no sub-second precision: `2020-12-20T19:17:47Z`). All document entries are serialized in the root JSON object; producers MUST declare media type `application/did+json`. Note: `@context` is *not* part of the JSON representation — it is a JSON-LD-only representation-specific entry.

### 5.3 JSON-LD: application/did+ld+json

The JSON-LD representation (§6.3, verified ✅) follows all JSON production rules **plus** the representation-specific `@context` entry, whose serialized value MUST be the string `https://www.w3.org/ns/did/v1` or an array whose first item is that string (further contexts — e.g. `https://w3id.org/security/suites/ed25519-2020/v1` — may follow). Producers MUST declare media type `application/did+ld+json`. JSON-LD gives the document linked-data semantics: terms map to IRIs via the context, which is what lets a verifier treat properties across issuers/methods uniformly. JSON-LD consumers SHOULD drop terms not defined by the context.

### 5.4 Other Representations

The spec's own diagram shows `application/did+cbor` alongside JSON and JSON-LD, and §6.1 permits further representations "such as XML or YAML" if properly specified (media type, fragment rules, lossless round-trip) ✅. The DID resolution spec (§8.1 of this guide) adds the practical layer: a resolver may return any conformant representation, negotiated via the `accept` option, with the media type reported in resolution metadata.

## 6. The DID Methods

### 6.1 What a DID Method Is

A **DID method** is "a definition of how a specific DID method scheme is implemented… the precise operations by which DIDs and DID documents are created, resolved, updated, and deactivated" ✅ (DID Core §2). The relationship between DID Core and a method specification is, per the spec itself, like the relationship between the IETF generic URI specification and a specific URI scheme such as `http` ✅ (§8): DID Core defines the generic syntax and data model; each method spec defines one `method-name` and how that name's identifiers and documents behave on a specific **verifiable data registry** — a ledger, a file system, DNS, a database, or nothing. "This specification does not presuppose any particular technology or cryptography" ✅ — which is why the method layer is where all the real variety (and governance risk) lives.

### 6.2 The Registries: From Community Lists to W3C Group Notes

**The registry situation changed materially between the July 2022 Recommendation and this pass (September 2026), and a guide must reflect the current state:**

- **DID Specification Registries (the W3C Note DID Core references).** DID Core's extensibility model points at the "DID Specification Registries" mechanism for registering properties, parameters, methods, and representations ✅ (§4.1, §8.1 — verified). That registry lineage is now published by the DID Working Group as **"Decentralized Identifier Extensions" (DID Extensions)**, a **W3C Group Note dated 11 December 2025** ✅ (verified at w3.org/TR/did-spec-registries, which this pass resolves to the Extensions Note; editors Manu Sporny 2017–present and Markus Sabadello 2020–until 2025-12-10, with former editors Orie Steele, Amy Guy, Ivan Herman). It enumerates three extension registries: **Property and Value Extensions, Resolution Extensions, and DID Methods** ✅. The registration process is normative (submissions MUST provide descriptions, machine-readable JSON-LD contexts, and MUST NOT infringe IP) ✅, and the Note's own status text warns the repository is "under active development" and implementers not involved with the DID Working Group should be cautious ✅.
- **The DID method registry (community era).** The historical community registry at `w3c-ccg.github.io/did-method-registry` — a W3C Credentials Community Group (CCG) page that DID Core-era material routinely cites — **returned HTTP 404 on every attempt this pass** ✅ (verified: the GitHub Pages site no longer serves the page; the same is true of the old CCG did:key and did-resolution pages, whose content moved to new homes noted in §6.4/§8.3). ⚠ The community registry is effectively superseded; treat any citation of it as historical.
- **The current registry: "DID Methods", a W3C Group Note of 12 August 2026** ✅ (verified at w3.org/TR/did-extensions-methods/): "This document serves as a collection of known DID Methods", listing ≈265 entries this pass (machine-counted) with links to each method's specification and contacts. Its disclaimer is the single most important governance sentence in the ecosystem: **"This registry does not act as an endorsement of any particular DID method or its underlying technologies by the W3C… It exists as a mechanism for developers to discover various DID methods"** ✅. A Group Note is endorsed by the DID Working Group but **not by W3C itself nor its Members**, and carries no patent-policy commitments ✅.

### 6.3 The Major Methods Compared

The six methods covered below are the ones this guide's audience will actually meet. Facts are verified against each method's own spec/registry entry this pass ✅ unless flagged ⚠:

| Method | Backing / registry (verified) | Ledger? | Update / Deactivate | Governance status this pass |
| --- | --- | --- | --- | --- |
| `did:key` | None — public key expanded into a DID document (CCG spec, v0.9) | No (generative) | Not possible (spec: cannot be updated or deactivated) | ✅ CCG draft, listed in the W3C method registry; no governance body needed |
| `did:web` | Domain name + HTTPS (`/.well-known/did.json`) (CCG spec, draft) | No (DNS/TLS) | Update = edit file; deactivate = remove file | ✅ CCG draft, listed in the W3C method registry; operator-governed |
| `did:jwk` | None — JWK deterministically expanded (spec in quartzjer/did-jwk) | No (generative) | Not supported (spec: Update/Deactivate "not supported") | ✅ listed in the W3C method registry; individual-maintained (Jeremie Miller) |
| `did:ethr` | Ethereum ERC-1056 smart-contract registry (uport-project/ethr-did-registry) | Yes (EVM chains) | Yes, via ERC-1056 `owner` | ✅ listed in the W3C method registry; DIF/uPort lineage, multi-network |
| `did:sov` | Sovrin MainNet (Hyperledger Indy-based public ledger) | Yes | Via ledger write roles | ⚠ **Sovrin Foundation dissolved 21 May 2025; MainNet is now a read-only archive under a caretaker** — §6.8 |
| `did:indy` | Hyperledger Indy ledgers (spec v1.0 Draft) | Yes | Yes (NYM writes) | ✅/⚠ spec is a Hyperledger (LF Decentralized Trust) draft, not W3C; ledger operators govern — §6.9 |

### 6.4 did:key — The Ledgerless Public-Key Method

**Verified public facts** (spec "The did:key Method v0.9", at w3c-ccg.github.io/did-key-spec ✅; registry entry: "Ledger-independent DID method based on public/private key pairs", contacts Manu Sporny, Dmitri Zagidulin, Dave Longley ✅):

- **No ledger, no registry, no lookups**: `did:key` is "a non-registry based DID Method based on expanding a cryptographic public key into a DID Document" ✅. The identifier *is* the key material: `did:key:z6MkhaXgBZDvotDkL5257faiztiGiC2QtKLGpbnnEGta2doK` — a multibase (base58-btc or base64url) value encoding the multicodec key-type prefix plus the raw public key bytes ✅.
- **Static and generative**: Create = generate a keypair and encode the public key; the DID document is derived deterministically (the key appears as `verificationMethod` under `authentication`, `assertionMethod`, `capabilityInvocation`, `capabilityDelegation`, with a derived X25519 key for `keyAgreement`) ✅. **Because the value is not stored in any registry, it cannot be updated or deactivated** ✅ — key rotation is impossible; a compromised `did:key` is dead.
- ⚠ Status notes: the spec is a CCG draft (v0.9), not a W3C standard-track document — though it is listed in the W3C DID-Methods Group Note. The spec's own guidance: suited to ephemeral sessions, development/testing, and bootstrap scenarios; "Use of did:key for long-lived use cases is only recommended when accompanied with high confidence in hardware isolation" ✅.

### 6.5 did:web — The Domain-Based Method

**Verified public facts** (spec "did:web Method Specification", CCG draft at w3c-ccg.github.io/did-method-web, developed in w3c-ccg/did-method-web ✅; registry entry: registry "Web", contacts Oliver Terbu, Mike Xu, Dmitri Zagidulin, Amy Guy ✅):

- **The method-specific identifier is a fully qualified domain name** (with optional path), and the DID document is served over HTTPS from that domain: the spec's target system is "the host (or domain)… resolved through the Domain Name System (DNS)", leveraging "a web domain's existing reputation" ✅.
- **Resolution is ordinary HTTPS**: replace `:` with `/` in the identifier, prepend `https://`, append `/.well-known` (when no path) and `/did.json`, then GET ✅. So `did:web:example.com` → `https://example.com/.well-known/did.json`, and `did:web:example.com:user:alice` → `https://example.com/user/alice/did.json`. The document's `id` must match the DID being resolved ✅. IPv4/IPv6 addresses are disallowed; a port must be percent-encoded (`did:web:example.com%3A3000`) ✅.
- **CRUD is file management**: Create = register the domain and publish `did.json`; Update = edit the file (version control + CI suggested for audit); Deactivate = remove the file ✅. There is intentionally **no HTTP API** for method operations — programmatic registration/management is left to each implementation ✅.
- ⚠ Status notes: CCG draft, not standards-track; listed in the W3C DID-Methods Group Note. The trust root is the web PKI (TLS + DNS), which is centralized by design — the spec's security section mandates strong TLS and recommends DNS-over-HTTPS (RFC 8484) during resolution to avoid tracking ✅. did:web's optional-path form proves control of a *file*, not of the whole domain ✅ (spec's own caveat).

### 6.6 did:jwk — The JWK Method

**Verified public facts** (spec.md in quartzjer/did-jwk, CC0-1.0 ✅; registry entry: "Ledger agnostic", contact Jeremie Miller ✅):

- "`did:jwk` is a deterministic transformation of a JWK into a DID Document" ✅. Format: `did:jwk:<base64url-value>` where the value is a base64url-encoded JWK ✅.
- The single key becomes verification method `#0` (the DID URL fragment is always `#0`; a JWK `kid` is not used as the reference) ✅; the DID document lists it under `assertionMethod`, `authentication`, `capabilityInvocation`, `capabilityDelegation`, and `keyAgreement` — with `keyAgreement` included only for non-"sig"-use keys ✅. Like did:key: **no registry; Update not supported; Deactivate not supported; key compromise = unusable identifier** ✅.
- ⚠ Status notes: maintained by Jeremie Miller (an early XMPP/Jabber figure) as an individual repo — the lightest governance of any method here; listed in the W3C method registry. Its security section notes the same JWK can serialize into many different identifiers (store the full `did:jwk:` URI) and that no 1:1 identifier↔key guarantee exists ✅.

### 6.7 did:ethr — The Ethereum ERC-1056 Method

**Verified public facts** (ETHR DID Method Specification, doc/did-method-spec.md in decentralized-identity/ethr-did-resolver, editor Mircea Nistor ✅; registry entry: registry "Ethereum", contact Mircea Nistor ✅; contract repo uport-project/ethr-did-registry, Apache-2.0 ✅):

- **ERC-1056 / uPort lineage**: the spec's abstract places the method on the Ethereum pattern "known as ERC1056", which "utilizes a smart contract for a lightweight identifier management system intended explicitly for off-chain usage" ✅. The registry contract is maintained in the **uport-project** GitHub organization ("Ethereum registry for ERC-1056 ethr did methods") ✅ — the uPort project's identity stack. The reference resolver implementation is decentralized-identity/ethr-did-resolver ✅. ⚠ The old standalone `ethr-did` library repository now 404s this pass (the resolver/spec repos remain authoritative); the ecosystem's `did:ethr` tooling also lives on under the DIF umbrella.
- **Identifier = Ethereum address or secp256k1 public key**: `did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a` (or with an explicit network: `did:ethr:mainnet:…`, `did:ethr:0x1:…`; mainnet is the default) ✅. Target systems: Ethereum Mainnet, Sepolia, Polygon, Gnosis Chain, and any EVM-compliant chain where ERC-1056 is deployed ✅.
- **Registration is implicit**: creating the DID requires no transaction — the address/keypair *is* the identifier; "no transaction fee for identifier creation; identifier creation is private" ✅. Key management, delegates, and attributes (service endpoints) are written to the ERC-1056 registry contract; the ERC-1056 `owner` of the identity performs Update/Delete, and deactivation is possible (setting owner to `0x0`) ✅.
- ⚠ Governance/status notes: `did:ethr` is listed in the W3C DID-Methods Group Note; the method spec is community-maintained (not W3C). On-chain data is public and immutable — the spec's privacy section is explicit that revoked attributes remain visible in historical event logs and that the method "does not require or encourage the disclosure of PII" ✅.

### 6.8 did:sov — The Sovrin Method, Governance Flagged

**Verified public facts** (registry entry: registry "Sovrin", contact Mike Lodder ✅; governance status verified at sovrin.org ✅):

- `did:sov` is the DID method of the **Sovrin MainNet**, a public permissioned ledger built on Hyperledger Indy, created by the Sovrin Foundation (Evernym's lineage) to anchor self-sovereign identity. ⚠ The canonical did:sov method specification URL (the sovrin-foundation GitHub Pages spec page) **404'd on every attempt this pass**; the W3C method registry's entry points to a spec-template page rather than a maintained spec — evidence of the method's archival state. Historical facts about the ledger's Indy basis and its write-role governance are inherited from the did:indy spec's Indy mechanics ✅.
- **The governance flag is now a documented fact, not speculation**: the **Sovrin Foundation was formally dissolved by the State of Utah on 21 May 2025** (announced on sovrin.org, article dated 20 January 2026, after a March 2025 shutdown warning) ✅. Per the Foundation's own wind-down notice: **Sovrin MainNet remains only as a "read-only archive"**; Trinsic acts as **Caretaker and sole Steward** of the ledger, which runs on a single virtual cloud server in read-only configuration; the network "could, in theory, be reactivated should there be sufficient interest, resources, and governance" ✅. Indyscan.io provides public browsing of the archived ledger ✅.
- **Practical meaning**: `did:sov` identifiers may still resolve against the archived ledger, but the method has **no active governance body, no active stewards consortium, and no write path** — ⚠ any production design that depends on did:sov issuance or updates should be treated as relying on dead infrastructure. This is the canonical case study for why method governance — not the W3C standard — is the DID ecosystem's real risk layer.

### 6.9 did:indy — The Hyperledger Indy Method, Governance Flagged

**Verified public facts** (Indy DID Method Specification v1.0 (Draft) at hyperledger.github.io/indy-did-method ✅; registry entry: "Any Hyperledger Indy Ledger", contact Stephen Curran ✅):

- The spec describes **Indy** as "a public ledger designed specifically and only for privacy-preserving self-sovereign identity… designed specifically to enable the use of verifiable credentials", and defines "how DIDs on an Indy ledger are managed and the operations for creating, reading, updating, and deleting DIDs" ✅. Ledger objects: **NYM** (the legacy Indy entity storing a DID — "writing a NYM… basically results in writing a DID to the ledger"), ATTRIB (deprecated for DID-doc service endpoints since did:indy allows them directly), SCHEMA/CRED_DEF/REV_REG_DEF (the AnonCreds issuance objects) ✅.
- **Editors** (eleven): Stephen Curran, Paul Bastian, Daniel Hardman, Char Howland, Christian Bormann, Dominic Wörner, Daniel Bluhm, Kyle Den Hartog, Artem Ivanov, Renata Toktar, Alexander Shcherbakov ✅. Latest draft is maintained at github.com/hyperledger-indy/indy-did-method ✅.
- **Governance status**: ⚠ the did:indy spec itself is a **draft (Version 1.0 Draft)** hosted under Hyperledger — now the **Linux Foundation Decentralized Trust (LFDT)**: the Hyperledger Indy project page is served under lfdecentralizedtrust.org branding and tags Indy **Graduated**, type "Distributed ledger software", "Verifiable Data Registry DLT" ✅ (verified this pass at hyperledger.org/projects/hyperledger-indy). Active work continues (e.g., Indy on Besu, cited in LFDT's Brazil digital-identity coverage ✅). The ledger operators (e.g., Indicio, IDunion-era networks, government deployments) govern each Indy network individually — did:indy is method-spec governance under LFDT, with per-network governance in the field. Listed in the W3C DID-Methods Group Note ✅.
- **Practical meaning**: did:indy is the *successor-oriented* Indy method (the did:sov community's forward path for Indy ledgers), but ⚠ a bank choosing it must evaluate *which* Indy network it points at and that network's operator, because "Any Hyperledger Indy Ledger" is the registry's own description of the target ✅.

### 6.10 Choosing a Method

The verified facts above reduce to a short selection rubric (design guidance, not a research claim):

- **Ledgerless, ephemeral, cheap**: `did:key` / `did:jwk` — no resolution infrastructure, no writes, no rotation, no revocation. Fine for session identifiers and test vectors; wrong for long-lived institutional identity.
- **Web-native institutional identity**: `did:web` — the DID document is a file the institution already knows how to protect (DNS + TLS + change management). This is the default choice for a bank's *own* corporate DID (§11); it is centralized by design.
- **On-chain identity**: `did:ethr` (EVM), `did:indy` (Indy networks) — where the requirement is a public, tamper-evident registry with rotation/deactivation and no reliance on a single domain operator. Cross-ref [smart_contracts_guide.md](smart_contracts_guide.md) for the contract-layer mechanics behind ERC-1056.
- **Avoid**: any method whose governance body has dissolved or whose registry is read-only (`did:sov` as of 2025 — §6.8), and any method whose spec is a personal repo with no maintenance commitment, unless the use case is throwaway.

## 7. The Method Requirements in DID Core

DID Core §8 ("Methods") is normative for method authors, and it is the *only* part of the W3C standard that governs the method layer ✅ (all requirements below verified against DID Core §8.1–§8.4):

### 7.1 Method Syntax Requirements

A method specification MUST: define exactly one method-specific DID scheme identified by exactly one method name (`method-name` rule); specify how to generate the `method-specific-id`; define sensitivity and normalization of that value; ensure uniqueness of the method-specific-id within the method and **global uniqueness of every DID it generates**; SHOULD be registered in the DID registries to reduce name conflicts; MAY define multiple method-specific-id formats; MAY restrict paths/queries/fragments more than the generic rules ✅.

### 7.2 Method Operations: Create, Read, Update, Deactivate

A method specification MUST define how **authorization** is performed for all operations, and MUST specify how a controller **creates** a DID and document, how a resolver **resolves** (reads) a document — including how the resolver verifies the authenticity of the response — what constitutes an **update** and how a controller updates (or state that updates are not possible), and how a controller **deactivates** (or state that deactivation is not possible) ✅. The spec notes the authority model is method-specific: a method might use the `controller` property, the `authentication` keys, the `capabilityInvocation` keys, or an out-of-band mechanism ✅. The generative methods (§6.4, §6.6) are exactly the "updates are not possible / deactivation is not possible" case the spec anticipates. The Read operation is what DID resolution invokes (§8.1) ✅.

### 7.3 Security and Privacy Requirements

Every method spec MUST document, per RFC 3552: the standard attack forms for its operations (eavesdropping, replay, message insertion, deletion, modification, denial of service, amplification, man-in-the-middle); residual risks; integrity protection and update authentication for all operations; the policy mechanism by which DIDs are proven uniquely assigned; method-specific endpoint authentication (including light-node/thin-client topology assumptions for DLTs) ✅. Privacy sections MUST discuss the applicable RFC 6973 subsections (surveillance, stored-data compromise, correlation, misattribution, secondary use, disclosure, exclusion…) ✅. These requirements are why every method page has a security/privacy section — and why a method *without* one is immediately suspect.

## 8. Resolution and the Infrastructure

### 8.1 DID Resolution

**DID resolution** is "the process that takes as its input a DID and a set of resolution options and returns a DID document in a conforming representation plus additional metadata", relying on the "Read" operation of the applicable DID method ✅ (DID Core §2, §7.1). DID Core defines the abstract function signatures — implementations are method-specific and out of scope for the core spec ✅:

```
resolve(did, resolutionOptions) → « didResolutionMetadata, didDocument, didDocumentMetadata »
resolveRepresentation(did, resolutionOptions) → « didResolutionMetadata, didDocumentStream, didDocumentMetadata »
```

Verified inputs/outputs ✅ (§7.1):

- **Inputs**: `did` (REQUIRED, conformant DID) and `resolutionOptions` (REQUIRED but may be empty; common option: `accept` — the caller's preferred representation media type, used only with `resolveRepresentation`).
- **Outputs**: `didResolutionMetadata` (data about the resolution process itself; REQUIRED; must contain `contentType` for `resolveRepresentation` and `error` on failure — defined error codes include `invalidDid`, `notFound`, `representationNotSupported`); `didDocument` (the data model — its `id` MUST match the resolved DID) or `didDocumentStream` (the byte stream in the negotiated representation); `didDocumentMetadata` (data about the document: `created`, `updated`, `deactivated`, `versionId`, `nextUpdate`, `equivalentId`, `canonicalId`…).
- Conforming resolvers MUST implement resolution for at least one method and return at least one conformant representation ✅.

The mental model for a bank: resolution is *the* lookup primitive — a verifier given a DID runs resolution (usually via a resolver library or the Universal Resolver, §8.4), obtains the DID document, and checks the signature or key-agreement material against the verification relationships the document publishes. No resolution, no verification: this is why §8.3's spec-governance journey matters.

### 8.2 DID URL Dereferencing

**DID URL dereferencing** is "the process that takes as its input a DID URL and a set of input metadata, and returns a resource" ✅ (DID Core §2, §7.2). The resource may be the DID document itself, a secondary resource inside it (a verification method or service, typically via fragment), or an external resource (via DID parameters). Signature ✅:

```
dereference(didUrl, dereferenceOptions) → « dereferencingMetadata, contentStream, contentMetadata »
```

Verified ✅: input `didUrl` is REQUIRED — and to dereference a fragment the *complete* DID URL including the fragment MUST be used; `dereferenceOptions` may be empty (common option: `accept`). Outputs: `dereferencingMetadata` (REQUIRED; `error` on failure, with codes `invalidDidUrl`, `notFound`), `contentStream` (the resource; empty on failure), `contentMetadata` (metadata about the content — if the content is a DID document this is the `didDocumentMetadata` structure). Dereferencing builds on resolution: the process first resolves the DID inside the DID URL, then applies path/query/fragment semantics ✅. Dereferencing a fragment is how `did:example:123#keys-1`-style references (§4) become actual key material.

⚠ One feature-level flag worth recording: in the current DID-Resolution v1 Candidate Recommendation Draft, the **DID URL dereferencing feature is marked "at risk"** — the Working Group states it "will likely be changed or removed… seeking feedback from implementers" ✅ (verified in the CR Draft's status section). Implementers (and this guide's readers) should not bet critical architecture on the *dereferencing* function's exact CR form; DID *resolution* is the stable core.

### 8.3 The Resolution Spec's Governance Journey

The task this guide inherits described DID resolution as "a CCG draft, NOT a W3C Recommendation" — true historically, and now out of date, so it is flagged precisely:

- **Origins**: DID Resolution was developed in the W3C Credentials Community Group by Markus Sabadello and Dmitri Zagidulin (the CCG-era draft, version 0.3, is the one DID Core's references point at) ✅.
- **Current state (verified this pass)**: the spec moved onto the W3C Recommendation track with the DID Working Group and is published as **"Decentralized Identifier Resolution (DID Resolution) v1" — W3C Candidate Recommendation Draft, 28 August 2026** ✅ (verified at w3.org/TR/did-resolution; this version CRD-did-resolution-1.0-20260828). Editors: Stephen Curran (Invited Expert) and Joe Andrieu; former editors Markus Sabadello (until 2025-12-10) and Dmitri Zagidulin; authors Sabadello and Zagidulin ✅. The editor's draft lives at w3c.github.io/did-resolution ✅; the old CCG pages at `w3c-ccg.github.io/did-resolution/` **404 this pass** ✅.
- ⚠ So the accurate status line is: **no longer merely a CCG draft — now a W3C Candidate Recommendation Draft on the track toward Recommendation, but not yet a W3C Recommendation** (and it requires two independent implementations per feature to exit CR ✅). Cite it as a work in progress.

### 8.4 The Universal Resolver

**Verified public facts** (github.com/decentralized-identity/universal-resolver, README and repo structure ✅; the universal-resolver.io marketing page itself was unreachable from this environment this pass — ⚠ flagged, the GitHub project page is authoritative):

- The **Universal Resolver** "resolves Decentralized Identifiers (DIDs) across many different DID methods, based on the W3C DID Core 1.0 and DID Resolution specifications" ✅. It is "a work item of the **DIF Identifiers & Discovery Working Group**" — a **Decentralized Identity Foundation (DIF) / community project, not a W3C standard** ⚠ ✅ (verified).
- **Driver architecture**: the resolver core (uni-resolver-core) dispatches each DID to a **driver** — a per-method adapter implementing that method's resolution logic. The repo ships `driver-http` plus per-method drivers; the README's own curl examples exercise did:jwk, did:key, did:web, did:ethr, did:sov, did:peer, did:btcr, did:ens, did:elem, did:kilt, and more ✅ — i.e., the driver catalog is what determines which methods a given deployment supports. ⚠ Driver coverage is community-maintained and uneven; check the driver list for the method you need rather than assuming universal coverage.
- **Deployment**: Apache-2.0 licensed; deployable locally via docker-compose; a DIF-hosted test instance runs at **dev.uniresolver.io**, with container images on Docker Hub (universalresolver) ✅; the project is also supported by NLnet/NGI0 PET under the EU's Next Generation Internet programme ✅.
- **Why it matters**: the Universal Resolver is the interoperability layer that turns "DIDs resolve" from a per-method library problem into a URL (`/1.0/identifiers/did:…`). It is also exactly where a bank's due diligence applies: it is *community infrastructure* — no W3C or standards-body SLA, no formal governance ✅/⚠.

### 8.5 DIDComm

**Verified public facts** (didcomm.org homepage and book.didcomm.org ✅; DIF DIDComm Working Group page ✅):

- **DIDComm** is "an open protocol for decentralized communication. It uses DIDs to establish confidential, ongoing connections, without the need for usernames and passwords" ✅ (didcomm.org/book). The site frames it as "powerful, high-trust, self-sovereign interactions over any transport", with a protocol registry for building blocks (messaging, credential exchange, etc.) ✅.
- **Governance/lineage**: developed by the **DIF DIDComm Working Group** (decentralized-identity/didcomm; chairs Sam Curren and Steve McCown ✅), with the spec published at didcomm.org and a community book + user group; related protocol work references Hyperledger Aries RFCs ✅. **⚠ DIDComm is not a W3C standard** — it is a DIF/Linux-Foundation-adjacent open protocol. ⚠ Version status: the ecosystem distinguishes DIDComm v1 (the earlier DIF-era messaging spec) from the DIDComm v2 spec family developed at didcomm.org; exact per-version ratification dates were not re-verified this pass (spec pages under didcomm.org 404'd on this pass's attempts) — treat "v1/v2" as a lineage statement, not a verified date claim.
- **Related protocols (kept brief)**: DIDComm messaging is the transport for agent-to-agent interactions; *presentation exchange* (DIF) and OpenID4VC (OIDF) are the surrounding credential-presentation protocols — the book explicitly notes DIDComm "can work alongside OpenID4VC" ✅. For a bank, DIDComm is the plausible channel over which a client wallet and the bank's agent exchange verifiable credentials and instructions; the exact protocol choice belongs to the VC sibling's territory `[verifiable_credentials_guide.md]`.

## 9. The SSI Stack and the Use Cases

### 9.1 The Self-Sovereign Identity Model

Self-sovereign identity (SSI) is the design philosophy that the individual or organization — not a platform, a government registry, or an identity provider — should hold and control the identifiers and credentials that make up its digital identity. The original ten-principles framing (Christopher Allen's 2016 "The Path to Self-Sovereign Identity" essay: existence, control, access, transparency, persistence, portability, interoperability, consent, minimization, protection) is the movement's canonical text ⚠ (the four-pillars variant — "the user controls their identifier, their data, their attestations, and their interactions" — is a popular restatement rather than a single citable primary source). This guide keeps the philosophical framing light and points at the engineering stack, where the terms are exact:

| SSI layer | W3C/DIF component (this guide's coverage) | Status |
| --- | --- | --- |
| Identifier | **DID** + DID document | W3C Recommendation, DID Core 1.0 — this guide, §2–§5 |
| Identifier transport | DID methods, resolution, Universal Resolver, DIDComm | §6–§8 (method specs community/foundation-governed) |
| Claims/attestations | **Verifiable Credentials** — issuer signs, holder stores, verifier checks | W3C standard; deep-dive lives in `[verifiable_credentials_guide.md]` (expected sibling — §9.2) |
| Discovery/ecosystem trust | Registries, trust registries, DIDComm protocol registry | §6.2, §8.5 |

The trust triangle that makes SSI work is a *VC-layer* concept (issuer → holder → verifier, with the signature checked against the issuer's DID document), so it is described fully in the VC sibling; §11.4 uses it only as the flow the bank's DID plugs into.

### 9.2 The Verifiable-Credentials Sibling (Forward Reference)

**`[verifiable_credentials_guide.md]` does not exist in the repo at the time of writing** — verified this pass (the technology/ folder has no such file). When it lands, it will carry the W3C VC Data Model (the standard's title is "Verifiable Credentials Data Model"; DID Core 1.0 itself cites VC-DATA-MODEL v1.1 in its terminology ✅) — the JSON/JSON-LD credential format, the `issuer`/`credentialSubject`/`proof` structure, issuance and presentation flows, and the surrounding protocols (OpenID4VC, presentation exchange). The DID is one component of that model: the *issuer* and *subject* fields reference DIDs, and the proofs verify against DID documents — which is why this guide exists as its sibling. Cross-reference it in detail once present; until then, §11.4's worked flow is deliberately sketched at the DID boundary and does not re-derive VC mechanics.

### 9.3 Banking and Identity Use Cases, Condensed

The banking use cases are covered in depth by the repo's banking guides; this guide only maps DIDs onto them (condensed, cross-referenced — not re-derived):

- **KYC/onboarding and KYC reuse** — the bank performs CDD/EDD once (the workflow lives in [private_banking_guide.md](../banking/private_banking_guide.md) §9.1 KYC/AML and §11's Cymbal Bank onboarding worked example ✅) and can later issue the client a verifiable credential referencing the client's DID ("CDD completed, PEP status, risk tier"). The Web3 umbrella guide's §8.1 and §11 already score this as the bank's DID/VC product lane ✅ (inherited from [web3_technologies_guide.md](web3_technologies_guide.md)).
- **Client self-sovereignty and portability** — the client's DID is not a bank account number: it is controlled by the client's wallet, so credentials (and the identity they prove) move with the client across institutions without re-verification. The counterparty risk is the pseudonymity/AML tension flagged in the Web3 guide: regulators (FATF travel rule, beneficial-ownership rules) require real-world linkage somewhere in the chain ✅ (inherited).
- **Institutional/corporate identity** — the bank itself (and its subsidiaries, funds, and products) gets a DID (§11): counterparties and clients can verify that a message, credential, or instruction genuinely originates from the bank. This is the *simplest* bank use case to deploy today because it does not depend on the consumer-wallet ecosystem maturing.
- **On-chain identity for tokenized assets** — where the bank operates in tokenized markets (cross-ref [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) and [smart_contracts_guide.md](smart_contracts_guide.md)), DIDs give wallets and contracts resolvable identities; ledger-based methods (did:ethr) bind the identifier to on-chain control (cross-ref §6.7).
- **⚠ Regulatory posture** — no MAS or SG digital-identity DID framework is asserted in this guide (none verified this pass; see §9.4). Data-protection treatment of DIDs/VCs falls under PDPA; issuance of credentials is "data governance, not a payment service" per the Web3 guide's MAS analysis ✅ (inherited from [web3_technologies_guide.md](web3_technologies_guide.md) §11.4).

### 9.4 The Singapore Context

The repo's [singapore/](../singapore/) research folder was checked this pass: it contains exactly four guides — `sg_gdp_industry_distribution.md`, `singapore-government-securities-guide.md`, `singaporean_overseas_retirement.md`, `starhub_software_systems_guide.md` — and **no digital-identity guide** ✅ (verified). The MAS regulatory posture for DIDs/VCs specifically is not re-derived here: the [MAS guide](../banking/mas_regulations_guidelines_guide.md) carries the persona conventions and the payment/DPT regime, and the Web3 guide carries the MAS-lens analysis of credential issuance ✅. No additional Singapore digital-identity source is therefore cross-referenced — the absence is noted rather than papered over.

## 10. Privacy and Security Considerations

Condensed from DID Core's own security (§9) and privacy (§10) considerations, the method privacy/security requirements (§7.3), and the repo's [cybersecurity_guide.md](cybersecurity_guide.md) (cross-ref for audit practice and the broader threat-model discipline). DID Core's considerations are explicitly advisory ("people using DIDs are advised to consider before deploying this technology in a production setting" ✅) but they encode the ecosystem's hard-won failure modes.

### 10.1 The Privacy Properties

Verified against DID Core §10 ✅:

- **Keep personal data out of DID documents.** "A DID and DID document do not inherently carry any personal data and it is strongly advised that non-public entities do not publish personal data in DID documents" ✅ (§9.2's binding note). A DID document should carry keys and service endpoints — nothing that identifies a human. The same applies to `service` entries, where publishing social accounts, personal websites, and email addresses is explicitly discouraged ✅ (§5.4).
- **Correlation risk.** DID documents are globally resolvable; reusing one DID across contexts lets observers correlate the subject's activities — DID Core devotes a subsection to "DID Document Correlation Risks" ✅. The mitigation is the design property from §1: "each entity can have as many DIDs as necessary to maintain their desired separation of identities, personas, and interactions" ✅ (spec introduction). The privacy requirements also mandate that method specs discuss RFC 6973 categories (surveillance, correlation, secondary use, disclosure, exclusion, etc.) ✅.
- **DIDs are not anonymous.** A DID proves *control of a key*, not an identity; pseudonymity survives only while nothing links the DID to a person. did:ethr's own privacy section makes the blockchain-specific version of this point: any on-chain link (KYC'd exchange deposit, ENS registration) "will compromise pseudonymity" ✅ (§6.7).

### 10.2 The Security Considerations

Condensed and verified against DID Core §9 ✅:

- **Proving control.** Authenticating as a DID subject means presenting a proof (signature) verifiable against a verification method listed under the appropriate relationship (§3.3) — "authenticate" is defined in the spec as proving "it controls a specific secret using one or more verification methods" ✅. Signed DID documents do *not* by themselves prove control: the spec is explicit that only the method's resolution process yields the correct document ✅ (§9.2 note).
- **Key management is the whole game.** There is no password reset and no helpdesk in DID land: the private keys are the control. DID Core devotes sections to key rotation (§9.7), revocation semantics (§9.8 — "revocation in trustless systems" is hard), and key-loss recovery via the controller relationship (§5.1.2 note on authorization vs authentication) ✅. For generative methods (did:key/did:jwk) compromise is terminal (§6.4/§6.6). For a bank this maps to HSM custody, quorum/dual-control signing, and documented rotation runbooks — standard practice per [cybersecurity_guide.md](cybersecurity_guide.md).
- **Infrastructure trust.** did:web trusts DNS + the web PKI (TLS) — the spec mandates strong TLS cipher configuration and recommends RFC 8484 DNS-over-HTTPS during resolution so the DNS resolver does not learn which DID is being looked up ✅ (§6.5). Ledger methods trust the ledger's consensus and the method's write-authorization model (§6.7–§6.9). Resolution infrastructure (Universal Resolver §8.4) is a centralization point to diligence — and per DID Core §9.1, implementers "need to bear in mind that there is no central authority to mandate which DID method specification is to be used with any specific DID method name" ✅.
- **Method-audit obligations.** §7.3's requirements (RFC 3552 attack coverage, residual risk, uniqueness policy, endpoint authentication) exist precisely so that a method spec can be read like a security review. A bank's method due diligence should treat a method spec *without* these sections as a red flag ✅/⚠.

### 10.3 What This Means for a Bank

Three takeaways, all grounded in the verified material above: (1) the *identifier layer* is privacy-safe by design (no PII in documents) while the *use layer* (credentials about people) is where PDPA-grade data protection applies — cross-ref the data-protection and outsourcing conventions of the [MAS guide](../banking/mas_regulations_guidelines_guide.md); (2) *key custody* replaces account recovery — the bank's DID operations need the same HSM/dual-control discipline as its payment signing, per [cybersecurity_guide.md](cybersecurity_guide.md); (3) *method and resolver governance* is an operational-risk item (the Sovrin dissolution, §6.8, is the existence proof) and should be reviewed like any third-party dependency.

## 11. The Cymbal Bank Worked Example: A Bank-Issued DID

### 11.1 The Design-Fiction Frame

Following the repository's worked-example conventions (as established in [web3_technologies_guide.md](web3_technologies_guide.md) §11 and [private_banking_guide.md](../banking/private_banking_guide.md) §11): this section is **design fiction** — an illustrative architecture built strictly from the verified facts of §§2–10, with example identifiers and endpoints that are deliberately fictional. The only bank persona permitted in this repository is **Cymbal Bank**, a MAS-licensed bank in Singapore (persona conventions per the [MAS guide](../banking/mas_regulations_guidelines_guide.md)); no other bank appears. The example domain `cymalbank.com` is fictional and used only to make the did:web mechanics concrete.

**The ask:** Cymbal Bank wants a corporate DID it can (a) publish as its institutional identity, (b) use to *sign verifiable credentials* it issues to private-banking clients (KYC/status credentials in the [private_banking_guide.md](../banking/private_banking_guide.md) §9.1/§11 flow), and (c) rotate safely without ever changing the identifier — the "persistent identifier" property that gives this guide its title.

### 11.2 Choosing did:web for the Bank

Why did:web (verified mechanics from §6.5) wins for Cymbal Bank's *corporate* DID:

- **The institution already operates the trust root.** did:web's security is DNS + TLS + file management — capabilities the bank's platform team runs daily. No new ledger vendor, no steward fees, no consensus to monitor.
- **Resolution works everywhere.** Any resolver or wallet that supports did:web (including the Universal Resolver's did:web driver, §8.4) fetches `https://cymalbank.com/.well-known/did.json` over ordinary HTTPS.
- **Rotation and recovery are change-management problems.** Update = publish a new did.json with a rotated key under `capabilityInvocation`, exactly the git/CI-controlled flow the did:web spec suggests ✅ (§6.5); deactivation = removing the file — a decision the bank can make unilaterally, unlike a ledger method's governance.
- **The trade-offs, accepted deliberately:** did:web is centralized (DNS/PKI) and its bare-domain form identifies the domain operator, not a specific legal entity — so Cymbal Bank's design uses a dedicated subdomain path and states the entity mapping in the document's own metadata and in the credentials it issues. For *client* DIDs (consumers/private clients) did:web is the wrong tool — clients should hold `did:key`/`did:jwk`-class identifiers in wallets, with the bank's credentials anchored to those (§11.4). ⚠ Design-fiction choices: the exact domain, entity DID, and key material below are illustrative; a real deployment would follow the same verified mechanics with real DNS, PKI, and HSM-issued keys.

### 11.3 The Bank's DID Document

Deployment target (following the did:web spec's resolution algorithm ✅): DID `did:web:cymalbank.com` resolves to the JSON file served at `https://cymalbank.com/.well-known/did.json`. Illustrative document (structure verified against DID Core §5 and the did:web spec's own examples ✅; keys fictional):

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/jws-2020/v1"
  ],
  "id": "did:web:cymalbank.com",
  "controller": "did:web:cymalbank.com",
  "verificationMethod": [
    {
      "id": "did:web:cymalbank.com#issuer-key-2026",
      "type": "JsonWebKey2020",
      "controller": "did:web:cymalbank.com",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "AaBcDeFgHiJkLmNoPqRsTuVwXyZ0123456789abcdefghij",
        "kid": "issuer-key-2026"
      }
    }
  ],
  "authentication": ["did:web:cymalbank.com#issuer-key-2026"],
  "assertionMethod": ["did:web:cymalbank.com#issuer-key-2026"],
  "capabilityInvocation": ["did:web:cymalbank.com#issuer-key-2026"],
  "service": [
    {
      "id": "did:web:cymalbank.com#vc-issuance",
      "type": "CredentialIssuanceService",
      "serviceEndpoint": "https://identity.cymalbank.com/vc/issuer"
    },
    {
      "id": "did:web:cymalbank.com#didcomm",
      "type": "DIDCommMessaging",
      "serviceEndpoint": "https://agent.cymalbank.com/didcomm"
    }
  ]
}
```

Design notes, each grounded in verified spec facts:

- **`assertionMethod` is the VC-signing key.** Per DID Core §5.3.2, `assertionMethod` is the relationship "used to specify how the DID subject is expected to express claims, such as for the purposes of issuing a Verifiable Credential" ✅ — so a verifier checking a Cymbal Bank-issued credential resolves this document and confirms the proof key sits under `assertionMethod` (§3.3).
- **JWK material** (`publicKeyJwk`, `JsonWebKey2020`) per DID Core §5.2.1 — public members only, `kid` as fragment convention ✅; Ed25519 chosen for the example (the same curve family the did:key examples use, §6.4).
- **Services** point at the bank's issuance and DIDComm endpoints (§3.4, §8.5). The DIDComm service makes the document addressable for agent-to-agent flows with client wallets.
- **Rotating the key** (2027 edition) means publishing a new did.json with `#issuer-key-2027` and moving the relationships to it — the *DID itself never changes*. That is the persistence property: `did:web:cymalbank.com` remains the stable identifier clients hold in their wallets and credentials reference, across every key rotation ✅ (update semantics, §6.5).

### 11.4 The Verifiable-Credential Issuance Flow

The flow is sketched at the DID boundary; the VC data-model mechanics themselves belong to `[verifiable_credentials_guide.md]` (expected sibling — §9.2):

1. **Client onboarding (existing process).** A private-banking client completes Cymbal Bank's CDD/EDD per [private_banking_guide.md](../banking/private_banking_guide.md) §9.1 and §11 — identity documents, source of wealth, structure register. Nothing about the DID changes this step; it is the bank's existing KYC engine ✅ (sibling-inherited).
2. **Client presents their DID.** The client's wallet holds a `did:key` identifier (or a wallet-managed method of the bank's supported list); the client authenticates to the bank's portal by proving control of that DID (a signature verifiable against the client's own DID document — §10.2's "proving control").
3. **Bank issues.** Cymbal Bank's issuance service (`did:web:cymalbank.com#vc-issuance`) signs a verifiable credential — subject = the client's DID, issuer = `did:web:cymalbank.com`, claims such as "CDD completed 2026-09, risk tier standard" — with the private key corresponding to the `#issuer-key-2026` verification method listed under `assertionMethod`.
4. **Client stores; verifier checks.** The client holds the credential in their wallet. Any third party (another bank, a custodian, a corporate-action agent) can verify it *without calling Cymbal Bank*: resolve `did:web:cymalbank.com` (via a resolver or the Universal Resolver, §8.4), take the `assertionMethod` key, check the credential's signature — the trust triangle of §9.1, with the DID document as the public verification anchor.
5. **AML linkage stays in the bank.** The pseudonymity caveat (§10.1): the verifiable credential proves the bank *issued* a status claim about that DID, and the bank retains the KYC file linking DID → legal identity for FATF/beneficial-ownership purposes — the regulatory overlay the Web3 guide flags ✅ (inherited).

### 11.5 The Operational and Risk Overlay

Condensed operational requirements that fall out of the verified facts:

- **Key custody:** the `#issuer-key-2026` private key lives in the bank's HSM with dual control; rotation is a scheduled, change-controlled operation (capabilityInvocation key controls document updates — §3.3, §7.2).
- **Document integrity:** did.json is served from the bank's CMS behind the same TLS/DNS controls as its public site, with the git/CI audit trail the did:web spec recommends ✅; a `LinkedDomains`-style service binding (§3.4) can tie the DID to the bank's verified web presence.
- **Governance diligence register:** method spec status (did:web is a CCG draft listed in the W3C method registry — §6.2/§6.5 ⚠), resolver dependency (Universal Resolver is DIF community infrastructure — §8.4 ⚠), and the no-endorsement registry disclaimer (§6.2 ✅) all go into the same third-party-risk file a bank keeps for any vendor.
- **Exit plan:** because the DID is a file the bank controls, the *identifier* can be migrated (new domain, new method) only by re-issuing — the persistence property cuts both ways: identifiers persist, so the bank should treat its DID as a long-lived public asset with the same care as its BIC/SWIFT identity.

## 12. The Claims Audit

### 12.1 The Verified-Facts Table

Every row was verified this pass against the primary source cited; ✅ = confirmed at the source, ⚠ = partial/flag, ❌ = not found/refuted.

| # | Claim | Status | Primary source verified this pass |
| --- | --- | --- | --- |
| 1 | DID Core 1.0 ("Decentralized Identifiers (DIDs) v1.0") is a W3C Recommendation published 19 July 2022 | ✅ | w3.org/TR/did-core (status line; this version REC-did-core-20220719) |
| 2 | DID Core editors: Sporny, Guy, Sabadello, Reed (Digital Bazaar, Danube Tech, Evernym/Avast affiliations) | ✅ | w3.org/TR/did-core (Editors block) |
| 3 | The expected six-editor list (Reed, Sporny, Sabadello, Longley, Allen, Noble) is NOT what the Recommendation shows | ✅ (correction) | w3.org/TR/did-core: Longley, Steele, Allen appear only under Authors; Noble absent; Guy is an editor |
| 4 | DID definition: globally unique persistent identifier, no centralized registration authority | ✅ | DID Core §2 terminology |
| 5 | DID syntax `did = "did:" method-name ":" method-specific-id` | ✅ | DID Core §3.1 ABNF |
| 6 | DID URL = DID + optional path/query/fragment; fragments reference document entries (services/keys) | ✅ | DID Core §3.2, examples 2–6 |
| 7 | DID subject vs DID controller definitions (subject = entity identified; controller = entity authorized to change the document) | ✅ | DID Core §2, §5.1.1–5.1.2 |
| 8 | DID document core properties: `id` required; `alsoKnownAs`, `controller`, `verificationMethod`, 5 relationships, `service` optional | ✅ | DID Core §5 property tables |
| 9 | Verification material limited to `publicKeyJwk` (no private members) and `publicKeyMultibase` | ✅ | DID Core §5.2.1 |
| 10 | Five verification relationships and their purposes (assertionMethod = VC issuance) | ✅ | DID Core §5.3.1–5.3.5 |
| 11 | Service entries need `id`, `type`, `serviceEndpoint` | ✅ | DID Core §5.4 |
| 12 | DID Core defines JSON and JSON-LD representations with media types application/did+json and application/did+ld+json; other representations permitted if properly specified | ✅ | DID Core §6 |
| 13 | Resolution signatures `resolve`/`resolveRepresentation` inputs (DID, options) and outputs (metadata, document/stream, document metadata) | ✅ | DID Core §7.1 |
| 14 | Dereferencing signature `dereference` inputs/outputs (metadata, contentStream, contentMetadata) | ✅ | DID Core §7.2 |
| 15 | Method-spec requirements: syntax rules, Create/Read/Update/Deactivate + authorization, RFC 3552 security + RFC 6973 privacy sections | ✅ | DID Core §8.1–8.4 |
| 16 | 103 experimental method specs / 32 drivers / 46 test-suite implementations at publication | ✅ | DID Core status section |
| 17 | DID Spec Registries lineage now published as "Decentralized Identifier Extensions" Group Note, 11 December 2025 | ✅ | w3.org/TR/did-spec-registries (resolves to the Extensions Note); w3.org/TR/did-extensions |
| 18 | "DID Methods" registry is a W3C Group Note, 12 August 2026, ≈265 entries, explicitly non-endorsing | ✅ | w3.org/TR/did-extensions-methods |
| 19 | Community registry w3c-ccg.github.io/did-method-registry now returns 404 | ✅ (this pass) | HTTP 404 on repeated fetches |
| 20 | did:key: CCG spec v0.9, generative, no ledger, cannot be updated/deactivated | ✅ | w3c-ccg.github.io/did-key-spec |
| 21 | did:web: domain-based, HTTPS did.json resolution, file-based CRUD, no HTTP API | ✅ | w3c-ccg.github.io/did-method-web (spec), github.com/w3c-ccg/did-method-web |
| 22 | did:jwk: JWK deterministically expanded; Update/Deactivate not supported; single `#0` key | ✅ | spec.md in github.com/quartzjer/did-jwk |
| 23 | did:ethr: ERC-1056 Ethereum registry; address/secp256k1 identifiers; implicit registration; uPort-lineage registry repo | ✅ | ethr-did-resolver did-method-spec.md; github.com/uport-project/ethr-did-registry |
| 24 | did:sov governance: Sovrin Foundation dissolved 21 May 2025; MainNet read-only archive under Trinsic caretaker | ✅ | sovrin.org (homepage banner + wind-down article, 20 January 2026) |
| 25 | did:sov canonical spec URL(s) dead this pass | ❌ (not found) | sovrin-foundation.github.io spec pages 404 |
| 26 | did:indy: spec v1.0 (Draft), 11 editors, CRUD on Indy ledgers | ✅ | hyperledger.github.io/indy-did-method |
| 27 | Indy project is a Graduated LF Decentralized Trust project ("Verifiable Data Registry DLT") | ✅ | hyperledger.org/projects/hyperledger-indy (served under LFDT branding) |
| 28 | DID Resolution: origins as CCG draft (Sabadello/Zagidulin) → now W3C Candidate Recommendation Draft, 28 August 2026 | ✅ | w3.org/TR/did-resolution (CRD-did-resolution-1.0-20260828); w3c-ccg.github.io/did-resolution 404 |
| 29 | DID URL dereferencing feature marked "at risk" in the DID Resolution CR Draft | ✅ | w3.org/TR/did-resolution status section |
| 30 | Universal Resolver: DIF work item, driver-based, dev.uniresolver.io test instance, Apache-2.0 | ✅ | github.com/decentralized-identity/universal-resolver |
| 31 | universal-resolver.io marketing page reachable | ❌ (blocked this pass) | fetch blocked (private-network guard) — GitHub repo used instead |
| 32 | DIDComm: open DID-based messaging protocol; DIF DIDComm WG; not a W3C standard | ✅ | didcomm.org; book.didcomm.org; identity.foundation/working-groups/did-comm.html |
| 33 | DIDComm v1/v2 specific version status/ratification dates | ⚠ | didcomm.org spec sub-pages 404'd this pass; lineage asserted only |
| 34 | VC sibling guide [verifiable_credentials_guide.md] exists in repo | ❌ (absent) | filesystem check of technology/ this pass |
| 35 | singapore/ folder has no digital-identity guide (4 files only) | ✅ | filesystem check this pass |
| 36 | Cymbal Bank persona (MAS-licensed, Singapore) conventions | ✅ (sibling-inherited) | private_banking_guide.md, mas_regulations_guidelines_guide.md, web3_technologies_guide.md |
| 37 | Web3 guide's DID facts (REC date, editors) consistent with this pass's primary verification | ✅ | cross-checked against w3.org/TR/did-core |

### 12.2 What Could Not Be Verified

Items flagged ⚠ or ❌ in this pass, collected here per the repo convention:

1. **The "six editors" list for DID Core (Reed, Sporny, Sabadello, Longley, Allen, Noble).** ❌ as stated — the Recommendation page lists four editors (Sporny, Guy, Sabadello, Reed) and six authors (Sporny, Longley, Sabadello, Reed, Steele, Allen); Grant Noble appears nowhere on the page. Corrected in §2.1; it is possible earlier drafts carried different editor lists, which was not investigated.
2. **did:sov method specification content.** ⚠ The W3C method registry entry links to a Sovrin Foundation spec page that 404'd on every attempt this pass; only the registry metadata (method name, registry "Sovrin", contact Mike Lodder), the Sovrin wind-down facts, and the did:indy spec's Indy mechanics were verifiable. The did:sov spec's detailed CRUD semantics could not be read at a primary source this pass.
3. **Sovrin ledger operational details beyond the Foundation's own notice.** ⚠ The read-only-archive state, Trinsic caretakership, and single-server hosting are as stated by sovrin.org (verified); no independent operational status check was possible.
4. **DIDComm v1 vs v2 version status and ratification dates.** ⚠ The didcomm.org spec sub-pages 404'd this pass; didcomm.org's homepage, the DIDComm Book, and the DIF WG page were verified, but exact per-version ratification facts were not. The guide therefore asserts the v1/v2 lineage only.
5. **Universal Resolver's live method-driver catalog.** ⚠ The repo README's examples (did:key, did:web, did:ethr, did:sov, did:peer, did:btcr, did:ens, did:elem, did:kilt, did:jwk…) were verified, but the current driver list of the DIF-hosted dev.uniresolver.io instance was not enumerable this pass (the universal-resolver.io marketing site was blocked from this environment, ❌).
6. **Hyperledger Indy per-network status.** ⚠ The did:indy spec draft status and Indy's LFDT Graduated project status were verified; the operational health of any specific Indy network was not.
7. **Current number of DID methods.** ⚠ ≈265 entries machine-counted in the 12 August 2026 Group Note this pass; the count drifts as entries are added, and a few rows may be non-method table rows. Treat as an order of magnitude.
8. **MAS/Singapore digital-identity DID framework.** ⚠ No SG/MAS DID-specific framework was verified this pass (none claimed); the singapore/ research folder contains no digital-identity guide (✅ absence verified). Any regulatory statements in §9.3/§10.3 are inherited from sibling guides, not re-verified here.
9. **SSI "four pillars" framing provenance.** ⚠ The ten-principles Allen-2016 essay is the canonical SSI text but was not re-fetched this pass; the "four pillars" phrasing is a popular restatement without a single primary source (§9.1).
10. **did:ethr library ecosystem.** ⚠ The standalone `ethr-did` repository 404'd this pass; the resolver/spec/registry repos (verified) are cited instead. Whether the library moved or was archived was not confirmed.
11. **The expected DID Core editor affiliations** in the research brief (Longley→Digital Bazaar ✅ as author; Allen→Blockchain Commons ✅ as author) — partially confirmed; the editor-role assignment itself is the unverified part (see item 1).

## 13. The Glossary

| Term | Definition (per DID Core unless noted) |
| --- | --- |
| **DID** | Decentralized Identifier — a globally unique persistent identifier with no centralized registration authority; syntax `did:method:method-specific-id` ✅ |
| **DID document** | The resolvable data structure describing the DID subject: keys, relationships, services ✅ |
| **DID subject** | The entity a DID identifies and its document describes — person, org, thing, data model ✅ |
| **DID controller** | Entity authorized to make changes to the DID document; may differ from the subject ✅ |
| **DID method** | The specification defining one `method-name`'s Create/Read/Update/Deactivate behavior on a verifiable data registry ✅ |
| **DID URL** | A DID plus optional path, query (DID parameters), and fragment; dereferences to a resource ✅ |
| **Verification method** | Parameters (typically a public key) used to verify proofs; expressed with `id`, `type`, `controller` and JWK/multibase material ✅ |
| **Verification relationship** | Declared purpose of a verification method: authentication, assertionMethod, keyAgreement, capabilityInvocation, capabilityDelegation ✅ |
| **Representation** | A concrete serialization of the DID document data model — JSON (`application/did+json`) and JSON-LD (`application/did+ld+json`) defined by DID Core ✅ |
| **DID resolution** | Input DID + options → DID document + metadata, via the method's Read operation ✅ |
| **DID URL dereferencing** | Input DID URL + options → resource (document, key, service, or external resource) + metadata; ⚠ marked "at risk" in the DID Resolution CR Draft |
| **Verifiable data registry** | The backing store a method uses to create/verify/update/deactivate DIDs — ledger, filesystem, DNS, or none ✅ |
| **Verifiable credential** | The W3C-standard signed claim format referencing DIDs; deep-dive in the expected sibling guide `[verifiable_credentials_guide.md]` |
| **DIDComm** | Open protocol for DID-rooted, transport-independent messaging (DIF; not W3C) ⚠ |
| **Universal Resolver** | DIF project resolving DIDs across methods via per-method drivers; community infrastructure ⚠ |
| **did:key / did:jwk** | Generative ledgerless methods: identifier derived from key material; no update/deactivate ✅ |
| **did:web** | Domain-based method: DID document served as did.json over HTTPS from the domain ✅ |
| **did:ethr** | Ethereum ERC-1056 method: address/secp256k1 identifiers with on-chain registry ✅ |
| **did:sov / did:indy** | Indy-ledger methods; did:sov's foundation dissolved 2025 (read-only archive); did:indy under LFDT ✅/⚠ |
| **SSI** | Self-sovereign identity — the model where the subject controls identifiers and credentials (§9.1) |

## 14. Cross-References and the Closing Summary

**Technology siblings (same folder):** [web3_technologies_guide.md](web3_technologies_guide.md) — the Web3 umbrella: §8.1 maps DIDs/VCs into the stack, §11 scores the bank's DID/VC product lane; this guide inherits its MAS-lens regulatory analysis. · `[verifiable_credentials_guide.md]` — the VC Data Model deep-dive, **expected but not yet in the repo when this guide was written** (§9.2): cross-reference it in detail once it lands; the DID is the identifier component of that model. · [smart_contracts_guide.md](smart_contracts_guide.md) — Ethereum/EVM and EIP/ERC mechanics behind did:ethr's ERC-1056 registry. · [distributed_auth_guide.md](distributed_auth_guide.md) — authn/authz architecture that DID/VC plugs into. · [cybersecurity_guide.md](cybersecurity_guide.md) — audit practice and threat modeling for §10's condensed considerations. · [blockchain_technology_guide.md](blockchain_technology_guide.md) — ledger mechanics for the verifiable-data-registry layer.

**Banking siblings (prefix `../banking/`):** [private_banking_guide.md](../banking/private_banking_guide.md) — §9.1 KYC/AML and §11's Cymbal Bank onboarding worked example (the issuance workflow this guide's §9.3/§11 map onto). · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) — MAS regime and Cymbal persona conventions. · [tokenized_assets_guide.md](../banking/tokenized_assets_guide.md) — institutional tokenization where on-chain DIDs apply. · **singapore/** research folder: four guides, no digital-identity guide (verified — absence noted, §9.4).

**The closing summary.** The W3C DID Core 1.0 Recommendation of 19 July 2022 standardized a deliberately thin layer: a URI syntax, a data model, two representations, and a set of method-authoring requirements — everything else in the DID ecosystem lives in method specifications, community registries, and foundation-governed infrastructure whose quality varies from "W3C Recommendation" to "dissolved foundation with a read-only ledger". The verified facts of this guide sort cleanly into three tiers: the **standard** (stable, ✅), the **registries** (real but explicitly non-endorsing, ✅/⚠), and the **methods** (diverse governance — diligence or die, ✅/⚠). For Cymbal Bank, the architecture writes itself from those facts: a `did:web` institutional DID as the stable, file-controlled anchor; an `assertionMethod` key under HSM custody signing client credentials; client-held wallet DIDs as the subjects; the Universal Resolver and DIDComm as the plumbing — all of it resting on one property the whole stack exists to deliver: an identifier that outlives keys, platforms, and vendors — the persistent identifier.
