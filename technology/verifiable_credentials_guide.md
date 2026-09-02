# Verifiable Credentials: The Provable Claim

**The W3C verifiable-credentials standards and ecosystem — the data model and its version history (v1.0, 19 November 2019 → v2.0, 15 May 2025, both verified at w3.org), the verifiable-presentation wrapper, the DID layer (inherited from the Web3 sibling, not re-derived), the formats (JSON-LD, the v1.0 JWT conventions, SD-JWT), the signature and privacy machinery (Data Integrity suites, BBS+), the revocation/status mechanics, the ecosystem programmes, the use cases (KYC, national digital identity, education, eIDAS 2.0), and the Cymbal Bank worked example — the deep-dive on the provable claim that sibling guides cross-reference instead of re-deriving.**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research — the standards-and-ecosystem deep-dive on W3C Verifiable Credentials (VCs); the Web3 umbrella guide carries the DID/Web3-identity layer map, and the banking guides carry the Singpass/MyInfo onboarding and KYC/EDD context, so this guide cross-references those instead of re-deriving them
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (technology/, same folder):** [web3_technologies_guide.md](web3_technologies_guide.md) (the Web3 umbrella — carries the verified DID record: w3.org/TR/did-core, W3C Recommendation 19 July 2022, plus the identity/storage/oracles application-layer map; cross-ref for DIDs and Web3 identity) · [smart_contracts_guide.md](smart_contracts_guide.md) (the EIP/ERC standards process and the on-chain mechanics — the place where VC meets chains, e.g. revocation registries and on-chain KYC tokens) · [cybersecurity_guide.md](cybersecurity_guide.md) (threats and key management — the security context for issuer/holder keys and wallet security)
> **Companion guides (banking/, prefix `../banking/`):** [private_banking_guide.md](../banking/private_banking_guide.md) (KYC/EDD conventions for onboarding — the compliance baseline that credential-based KYC builds on) · [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (the SG/MAS regulatory regime — the posture any VC-based onboarding in Singapore must fit) · [fireblocks_guide.md](../banking/fireblocks_guide.md) (institutional wallet/custody mechanics — the wallet-security twin for §7) · [dbs_bank_guide.md](../banking/dbs_bank_guide.md), [uob_software_systems_guide.md](../banking/uob_software_systems_guide.md), [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [maribank_guide.md](../banking/maribank_guide.md), [trust_bank_guide.md](../banking/trust_bank_guide.md), [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) (the Singpass/MyInfo onboarding context in SG banking — cited in §9, not re-derived)

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, the Cymbal Bank lens, and the map of sibling relationships. Section 2 is the standards core, verified at w3.org: the VC Data Model v1.0 Recommendation (19 November 2019), the v1.1 and v2.0 steps (v2.0 Recommendation 15 May 2025), Verifiable Presentations, the DID layer (inherited ✅), the satellite specs (Bitstring Status List, Data Integrity and its cryptosuites, VC JSON Schema, vc-jose-cose), and the W3C process context (the Verifiable Credentials Working Group and the Credentials Community Group). Section 3 is the ecosystem: the issuer/holder/verifier trust triangle, the trust fabric (registries, trust anchors), and the ecosystem programmes (ToIP, OpenWallet Foundation). Sections 4–7 are the technology: formats (JSON-LD vs JWT conventions vs SD-JWT), signatures and privacy tech (LD-Proofs, Ed25519/ECDSA suites, BBS+), revocation/status, and protocols/flows including wallets (condensed — the institutional wallet mechanics live in the fireblocks guide). Sections 8–11 are the use cases: KYC/onboarding (cross-ref'd to the private-banking guide), digital identity (Singpass/MyInfo in SG context; the EUDI wallet), education/employment/health/travel credentials, and the eIDAS 2.0 regulatory case. Section 12 is the Cymbal Bank worked example (clearly marked design fiction). Section 13 is the claims audit, with a "What Could Not Be Verified" subsection. Section 14 is the glossary, the cross-reference map, and the closing summary. **Reading paths:** *Architecture:* §1 → §2 → §7 → §12. *Engineer:* §4 → §5 → §6 → §7. *Compliance:* §8 → §9 → §11 → §12. *In a hurry:* §1, §2, §11, §12, §14.

**Integrity convention.** Every factual claim in this guide carries one of three marks: **✅** verified this pass against a primary or authoritative source (w3.org, EUR-Lex, european-commission pages, gov.sg, the IETF/RFC editor — named in the claims audit, §13); **⚠** flagged — reported, approximate, single-sourced, fast-moving, contested, or not re-verifiable this pass; **❌** refuted or not found. Unmarked statements are domain-stable technical knowledge (what a signature is, what a bitstring is) rather than research claims. Where a sibling guide carries a verified fact, this guide marks it ✅ and cites the sibling instead of re-verifying from scratch — the repo convention is that sibling-verified facts are inherited.

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [Why a Bank Should Care: The Cymbal Bank Lens](#13-why-a-bank-should-care-the-cymbal-bank-lens)
   - 1.4 [The Relationship to the Sibling Guides](#14-the-relationship-to-the-sibling-guides)
2. [The Standards: The Data Model and Its Satellite Specs](#2-the-standards-the-data-model-and-its-satellite-specs)
   - 2.1 [The VC Data Model v1.0: W3C Recommendation, 19 November 2019](#21-the-vc-data-model-v10-w3c-recommendation-19-november-2019)
   - 2.2 [v1.1 and the v2.0 Evolution: W3C Recommendation, 15 May 2025](#22-v11-and-the-v20-evolution-w3c-recommendation-15-may-2025)
   - 2.3 [Verifiable Presentations](#23-verifiable-presentations)
   - 2.4 [The DID Layer: Inherited, Not Re-Derived](#24-the-did-layer-inherited-not-re-derived)
   - 2.5 [The Satellite Specs: Status, Proofs, Schemas, and Securing](#25-the-satellite-specs-status-proofs-schemas-and-securing)
   - 2.6 [The W3C Process Context: Working Group, Community Group, Registries](#26-the-w3c-process-context-working-group-community-group-registries)
3. [The Ecosystem: Roles, Trust Fabric, and Programmes](#3-the-ecosystem-roles-trust-fabric-and-programmes)
   - 3.1 [The Trust Triangle: Issuer, Holder, Verifier](#31-the-trust-triangle-issuer-holder-verifier)
   - 3.2 [The Trust Fabric: Registries, Trust Anchors, Governance](#32-the-trust-fabric-registries-trust-anchors-governance)
   - 3.3 [The Ecosystem Programmes](#33-the-ecosystem-programmes)
4. [The Formats: JSON-LD, JWT Conventions, and SD-JWT](#4-the-formats-json-ld-jwt-conventions-and-sd-jwt)
   - 4.1 [Model versus Serialization](#41-model-versus-serialization)
   - 4.2 [JSON-LD and Linked-Data Proofs](#42-json-ld-and-linked-data-proofs)
   - 4.3 [The v1.0 JWT-VC Conventions: the `vc` and `vp` Claims](#43-the-v10-jwt-vc-conventions-the-vc-and-vp-claims)
   - 4.4 [v2.0 Securing: vc-jose-cose and SD-JWT (RFC 9901)](#44-v20-securing-vc-jose-cose-and-sd-jwt-rfc-9901)
5. [The Signatures and Privacy Technology](#5-the-signatures-and-privacy-technology)
   - 5.1 [The Data Integrity Suite Family](#51-the-data-integrity-suite-family)
   - 5.2 [Ed25519 and ECDSA Suites: the Workhorses](#52-ed25519-and-ecdsa-suites-the-workhorses)
   - 5.3 [BBS+: Selective Disclosure and Zero-Knowledge](#53-bbs-selective-disclosure-and-zero-knowledge)
   - 5.4 [The Trade-Offs: Correlation, Privacy, Performance](#54-the-trade-offs-correlation-privacy-performance)
6. [Revocation and Status](#6-revocation-and-status)
   - 6.1 [Bitstring Status List v1.0: W3C Recommendation, 15 May 2025](#61-bitstring-status-list-v10-w3c-recommendation-15-may-2025)
   - 6.2 [Registries, On-Ledger Revocation, and the Alternatives](#62-registries-on-ledger-revocation-and-the-alternatives)
   - 6.3 [The Trade-Offs: Online versus Offline Verification](#63-the-trade-offs-online-versus-offline-verification)
7. [Protocols and Flows](#7-protocols-and-flows)
   - 7.1 [Issuance Flows](#71-issuance-flows)
   - 7.2 [Presentation: Holder Binding, Derivation, Challenge](#72-presentation-holder-binding-derivation-challenge)
   - 7.3 [Verification Flows](#73-verification-flows)
   - 7.4 [Wallets, Condensed](#74-wallets-condensed)
   - 7.5 [The Interoperability Realities](#75-the-interoperability-realities)
8. [Use Cases I — KYC and Onboarding](#8-use-cases-i--kyc-and-onboarding)
   - 8.1 [Credential-Based KYC](#81-credential-based-kyc)
   - 8.2 [The Regulated-Entity Verification Flows](#82-the-regulated-entity-verification-flows)
   - 8.3 [The Account-Opening Flow](#83-the-account-opening-flow)
9. [Use Cases II — Digital Identity: Singpass and the EUDI Wallet](#9-use-cases-ii--digital-identity-singpass-and-the-eudi-wallet)
   - 9.1 [Singpass and MyInfo: Singapore's National Digital Identity](#91-singpass-and-myinfo-singapores-national-digital-identity)
   - 9.2 [Singapore's Verifiable-Credential Plans](#92-singapores-verifiable-credential-plans)
   - 9.3 [The EU Digital Identity Wallet](#93-the-eu-digital-identity-wallet)
10. [Use Cases III — Education, Employment, Health, and Travel](#10-use-cases-iii--education-employment-health-and-travel)
    - 10.1 [Diplomas and Badges: Open Badges and Blockcerts](#101-diplomas-and-badges-open-badges-and-blockcerts)
    - 10.2 [The European Layer: Europass EDCs and EBSI/Europeum](#102-the-european-layer-europass-edcs-and-ebsieuropeum)
    - 10.3 [Employment, Health, and Travel Credentials](#103-employment-health-and-travel-credentials)
11. [Use Cases IV — Regulatory: eIDAS 2.0 and the EU Digital Identity Framework](#11-use-cases-iv--regulatory-eidas-20-and-the-eu-digital-identity-framework)
    - 11.1 [The Regulation, Verified](#111-the-regulation-verified)
    - 11.2 [What eIDAS 2.0 Mandates for the Wallet](#112-what-eidas-20-mandates-for-the-wallet)
    - 11.3 [The Implications for Banks](#113-the-implications-for-banks)
12. [The Cymbal Bank Worked Example: A Verifiable-Credential KYC Pilot](#12-the-cymbal-bank-worked-example-a-verifiable-credential-kyc-pilot)
    - 12.1 [The Design-Fiction Frame](#121-the-design-fiction-frame)
    - 12.2 [The Credential Design](#122-the-credential-design)
    - 12.3 [Issuance: Cymbal Bank as Issuer](#123-issuance-cymbal-bank-as-issuer)
    - 12.4 [Verification at Onboarding: The Corporate-Client Flow](#124-verification-at-onboarding-the-corporate-client-flow)
    - 12.5 [Architecture Notes and Interoperability Caveats](#125-architecture-notes-and-interoperability-caveats)
13. [The Claims Audit](#13-the-claims-audit)
    - 13.1 [The Verified-Facts Table](#131-the-verified-facts-table)
    - 13.2 [What Could Not Be Verified](#132-what-could-not-be-verified)
14. [Glossary, Cross-References and the Closing Summary](#14-glossary-cross-references-and-the-closing-summary)
    - 14.1 [The Glossary](#141-the-glossary)
    - 14.2 [The Cross-Reference Map](#142-the-cross-reference-map)
    - 14.3 [Primary Sources Used This Pass](#143-primary-sources-used-this-pass)
    - 14.4 [The Closing Summary](#144-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

**Verifiable Credentials (VCs)** are the W3C's standard way to express a cryptographically secure, privacy-respecting, machine-verifiable digital credential — a driver's licence, a diploma, a bank's KYC attestation — so that the person or organisation holding it can present it to any verifier **without calling the issuer back for every transaction**. The core W3C specification, the *Verifiable Credentials Data Model*, reached W3C Recommendation status on **19 November 2019** (v1.0), was updated as v1.1 on **3 March 2022**, and was replaced by **v2.0 on 15 May 2025** — all three dates verified this pass at w3.org (§2.1–§2.2). The model defines a three-party ecosystem of **issuers, holders, and verifiers** (verified in the v2.0 abstract), and a companion document defines **verifiable presentations**, the verifier-facing wrapper in which a holder assembles and presents one or more credentials (§2.3).

The one-line summary that captures the pitch: a VC turns a **claim** ("this client passed our KYC checks") into a **provable claim** — signed by the issuer, portable across verifiers, and revealing only what the holder chooses. That is the whole thesis of this guide and the reason the closing line echoes it: for a bank, the credential is not the interesting part; the **provability** is — the auditable, replayable, standards-shaped proof that a claim was made by a particular issuer about a particular subject at a particular time, without the issuer needing to be online.

For a bank, the honest summary is more layered. The W3C stack is real, ratified, and now unusually well timed: v2.0 and its satellite Recommendations all landed on **15 May 2025** (Data Integrity, Bitstring Status List, vc-jose-cose, the EdDSA/ECDSA cryptosuites — §2.5), the IETF published **SD-JWT as RFC 9901 in November 2025** (§4.4), and the EU's eIDAS 2.0 regulation (Regulation (EU) 2024/1183, in force since **20 May 2024**) obliges member states to make **EU Digital Identity Wallets** available to citizens by **the end of 2026** (§11.1). Yet the ecosystem remains fragmented in practice — JSON-LD versus JWT camps, multiple proof suites, wallet formats that do not all interoperate (§7.5) — and the most-cited national example in this guide's world, Singapore's Singpass, is a *centralised* national identity built on consent-based data sharing (MyInfo) rather than a decentralised VC network (§9.1). The rest of this guide maps the standards, the formats, the use cases, and the banking applications with that nuance kept firmly in view.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|---|---|---|
| VC Data Model v1.0 | W3C Recommendation, **19 November 2019** | ✅ §2.1 |
| VC Data Model v1.1 | W3C Recommendation, **3 March 2022** | ✅ §2.1 |
| VC Data Model v2.0 | W3C Recommendation, **15 May 2025** (published by the Verifiable Credentials Working Group) | ✅ §2.2 |
| Verifiable Presentations | Defined in the data model (v2.0 §4.13): aggregate one or more VCs; SHOULD be short-lived and bound to a verifier challenge; `type` MUST include `VerifiablePresentation` | ✅ §2.3 |
| DID v1.0 | W3C Recommendation **19 July 2022** (w3.org/TR/did-core) — inherited from the Web3 sibling | ✅ §2.4 |
| DID methods registered (W3C registry) | `did:web`, `did:key`, `did:ethr`, `did:sov`, `did:jwk` — all status "registered" in the W3C DID-methods registry (w3c.github.io/did-extensions/methods) | ✅ §2.4 |
| Bitstring Status List v1.0 | W3C Recommendation, **15 May 2025** (the spec formerly known as StatusList2021) | ✅ §6.1 |
| VC Data Integrity 1.0 | W3C Recommendation, **15 May 2025** | ✅ §5.1 |
| EdDSA / ECDSA cryptosuites | Both W3C Recommendations, **15 May 2025** | ✅ §5.2 |
| BBS cryptosuites (vc-di-bbs) | Still a Candidate Recommendation Draft (latest CRD 7 April 2026) — not yet a Recommendation | ✅/⚠ §5.3 |
| VC JSON Schema | Candidate Recommendation Draft (latest 4 February 2025) — not yet a Recommendation | ✅/⚠ §2.5 |
| vc-jose-cose | W3C Recommendation, **15 May 2025** — JOSE/COSE/SD-JWT securing for VCs | ✅ §4.4 |
| SD-JWT | **RFC 9901**, IETF Standards Track, **November 2025** | ✅ §4.4 |
| SD-JWT VC (draft-ietf-oauth-sd-jwt-vc) | Active IETF draft (expires 2027) — ⚠ not yet an RFC | ✅/⚠ §4.4 |
| eIDAS 2.0 | Regulation (EU) 2024/1183 of **11 April 2024**, OJ 30.4.2024, in force **20 May 2024**; amends eIDAS (Regulation (EU) No 910/2014) | ✅ §11.1 |
| EUDI wallet application | Wallet obligations run from the Article 5a(23) implementing acts (adopted 28 Nov 2024) + 24 months; Commission's framing: wallets by **end of 2026**; the widely quoted **21 November 2026** is the derived target date | ✅/⚠ §11.1 |
| Singpass | Singapore's national digital identity (Smart Nation strategic project), GovTech-built; app since 2018; MyInfo consent-based data pre-fill | ✅ §9.1 |
| EUROPEUM-EDIC | Commission decision **21 May 2024**; 9 member states; takes over governance of EBSI | ✅ §10.2 |
| v1.0 JWT-VC conventions | `vc`/`vp` JWT claims plus `exp`/`iss`/`nbf`/`jti`/`sub`/`aud` mappings (v1.0 §6.3.1) | ✅ §4.3 |

### 1.3 Why a Bank Should Care: The Cymbal Bank Lens

Cymbal Bank's interest in verifiable credentials is not the technology fashion — it is three structural pressures the standards happen to answer:

1. **KYC cost and reuse.** Onboarding a client means re-collecting and re-verifying identity and corporate documents at every relationship, every product, every jurisdiction. A KYC attestation issued once by a regulated entity and presented onward as a VC is the "portable KYC" that regulators and consortia have chased for a decade; the difference now is that the envelope is a ratified W3C standard rather than a proprietary token. Cross-ref [private_banking_guide.md](../banking/private_banking_guide.md) for the EDD conventions that credential-based KYC must still respect — the credential does not remove the compliance obligation, it changes how the evidence is carried (§8.1).
2. **The regulatory calendar.** eIDAS 2.0 obliges EU member states to issue EUDI Wallets by end-2026, and service providers that must identify customers "unequivocally" (a class that plainly includes banks) will be obliged to accept them for authentication (§11.2–§11.3). A Singapore-licensed bank serving EU clients, or any bank with an EU branch, will need wallet-acceptance capability on that timetable. In Singapore itself the relevant posture is MAS's, carried by [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md).
3. **Client experience and fraud.** Verified credentials reduce manual document handling, shorten account-opening time, and cut the attack surface of emailed PDFs and uploaded scans — but they also move risk to key management, wallet security, and status-check availability (§6.3, §7.5). The bank's cybersecurity posture (cross-ref [cybersecurity_guide.md](cybersecurity_guide.md)) and its institutional-wallet experience (cross-ref [fireblocks_guide.md](../banking/fireblocks_guide.md)) become the relevant security context.

The counterweight: adoption is uneven, the format wars are real (§7.5), and the flagship national schemes (Singpass, the EUDI wallet) are government-led rather than open-market — so the bank's realistic strategy is to build **acceptance capability** (verify well-known formats) before it builds **issuance volume**. The worked example in §12 scores this into a concrete pilot.

### 1.4 The Relationship to the Sibling Guides

This guide is the **standards-and-ecosystem deep-dive**; the siblings carry the adjacent mechanics, and the repo's convention is that their verified facts are inherited rather than re-derived:

- [web3_technologies_guide.md](web3_technologies_guide.md) is the Web3 umbrella: it carries the verified DID record (W3C Recommendation 19 July 2022) and maps the Web3 identity/storage/oracles application layer. This guide's §2.4 inherits the DID fact ✅ and does not re-derive DID mechanics; it adds only the DID-methods registry verification, which the umbrella does not carry.
- [smart_contracts_guide.md](smart_contracts_guide.md) carries the on-chain mechanics (EIP/ERC process, token standards, proxies). Where VC infrastructure touches a ledger — revocation registries anchored on-chain, or "on-chain KYC" tokens — that is a different beast from W3C VCs, and the distinction is drawn in §6.2.
- [private_banking_guide.md](../banking/private_banking_guide.md) carries KYC/EDD conventions; §8 builds credential-based KYC on top of that baseline without restating it. [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) carries the SG/MAS regime; §9 and §11 reference it for the regulatory posture.
- The SG banking cluster — [dbs_bank_guide.md](../banking/dbs_bank_guide.md), [uob_software_systems_guide.md](../banking/uob_software_systems_guide.md), [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [maribank_guide.md](../banking/maribank_guide.md), [trust_bank_guide.md](../banking/trust_bank_guide.md), [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) — carries the Singpass/MyInfo onboarding context as it appears in each bank's systems; §9 cites the cluster instead of re-describing bank-by-bank onboarding.
- [fireblocks_guide.md](../banking/fireblocks_guide.md) carries institutional wallet/custody mechanics; §7.4 condenses the wallet topic and points there.

Rule of thumb: if a fact is about *how the W3C data model, its formats, or its ecosystem works*, it lives here; if it is about *chains, KYC procedure, the MAS regime, or wallet custody*, it lives in the sibling.

## 2. The Standards: The Data Model and Its Satellite Specs

### 2.1 The VC Data Model v1.0: W3C Recommendation, 19 November 2019

The *Verifiable Credentials Data Model 1.0* — full title "Verifiable Credentials Data Model 1.0: Expressing verifiable information on the Web" — was published as a **W3C Recommendation on 19 November 2019** (verified at the dated Recommendation URL w3.org/TR/2019/REC-vc-data-model-20191119/). Its abstract states the programme: credentials are "part of our daily lives" (driver's licences, degrees, passports), and the specification provides "a mechanism to express these sorts of credentials on the Web in a way that is cryptographically secure, privacy respecting, and machine-verifiable." Editors were Manu Sporny, Grant Noble, Dave Longley, Daniel C. Burnett and Brent Zundel (ConsenSys, Digital Bazaar, Evernym affiliations visible on the document). ✅

The v1.0 model's core shapes, all verified in the Recommendation text this pass:

- A **credential** carries `issuer`, `issuanceDate` (and optionally `expirationDate`), a `credentialSubject` holding the claims, an optional `@context` (JSON-LD), and a `proof` or equivalent securing mechanism.
- The **trust model** is deliberately decentralised: the spec's conformance section notes that the ecosystem is application- and market-specific, and that the *issuer–holder–verifier* roles are defined by the use case, not hard-wired by the standard.
- **Proofs** were already polymorphic: at publication the Working Group had implementations using at least three mechanisms — JSON Web Tokens (RFC 7519) secured with JWS, Linked Data Signatures, and (per the surrounding text) additional proof formats. ✅
- The v1.0 spec defined a full **JWT serialization** with registered claims (`vc`, `vp` — §4.3 of this guide) that became the most widely deployed VC carrier in the industry's first wave.

A v1.1 revision followed on **3 March 2022** (verified: w3.org/TR/vc-data-model-1.1/ shows "W3C Recommendation 03 March 2022", with an implementation report covering fourteen implementations). v1.1 was a maintenance release clarifying the model (notably around `credentialSchema` and the proof model) rather than a redesign. ✅

### 2.2 v1.1 and the v2.0 Evolution: W3C Recommendation, 15 May 2025

**v2.0 is a W3C Recommendation dated 15 May 2025** (verified at w3.org/TR/vc-data-model-2.0/ and at the dated URL .../REC-vc-data-model-2.0-20250515/; the undated shortlink /TR/vc-data-model now serves v2.0). The Verifiable Credentials Working Group published it "as a Recommendation using the Recommendation track," and W3C recommends wide deployment. The v2.0 editorial team is Sporny, Ted Thibodeau Jr, Ivan Herman, Gabe Cohen and Michael B. Jones. ✅

The v2.0 changes that matter, verified in the Recommendation text and its satellite specs this pass:

- **A new conformance regime.** A conforming document MUST be a verifiable credential with media type `application/vc` or a verifiable presentation with media type `application/vp`, and MUST be secured by at least one securing mechanism as described in the spec's §4.12 (verified in the conformance section). v2.0 is deliberately agnostic about *which* securing mechanism — it defines the envelope, not the glue.
- **Securing mechanisms split into two families.** The spec distinguishes **embedded proofs** — the Data Integrity family (w3.org/TR/vc-data-integrity, itself a 15 May 2025 Recommendation), where the proof sits inside the credential graph — from **enveloping proofs**, where the credential is wrapped in an external token format, notably JOSE/COSE per the *Securing Verifiable Credentials using JOSE and COSE* spec (vc-jose-cose, also a 15 May 2025 Recommendation). The v2.0 figures verified this pass show both: an "enveloping proof based on VC-JOSE-COSE" and "an embedded proof based on Verifiable Credential Data Integrity 1.0".
- **The JWT serialization moved out of the data model.** Where v1.0 baked JWT encoding into the spec (§4.3), v2.0 leaves JWT/SD-JWT/COSE to vc-jose-cose (§4.4). This is the single most important migration fact for existing JWT-VC deployments.
- **New namespaces and media types.** v2.0 uses the new JSON-LD context namespace `https://www.w3.org/ns/credentials/v2` (verified in the spec's own presentation example) alongside `application/vc` / `application/vp` media types.
- **A JSON-LD 1.1-aligned processing model** and a large privacy/security section (data minimisation, bearer-credential warnings, aggregation-correlation cautions — §8.9–§8.13 of the spec, section titles verified in the text). The phrase "JSON-LD 1.1 alignment" is the community's shorthand for the v2 processing model; the normative anchor verified here is the v2 context and the RDF/default-graph treatment visible throughout the spec. ⚠ the shorthand itself is not a quoted phrase in the Recommendation.
- Satellite deliverables advanced in lockstep: Bitstring Status List v1.0, VC Data Integrity 1.0, the EdDSA/ECDSA cryptosuites, and vc-jose-cose are all Recommendations of the same 15 May 2025 date (§2.5), while VC JSON Schema and the BBS cryptosuites remain on the Candidate Recommendation track.

The practical read for a bank: **v1.0/v1.1 JWT-VC deployments are not broken, but the standard's centre of gravity has moved** — new work assumes the v2 data model, Data Integrity proofs or vc-jose-cose envelopes, and the v2 context. Any integration built in 2026 should target v2.0 with a deliberate decision about the securing mechanism (§7.5).

### 2.3 Verifiable Presentations

A **verifiable presentation (VP)** is the holder's wrapper: the object in which a holder presents one or more verifiable credentials to a verifier. The v2.0 spec defines it in §4.13 (verified in the Recommendation text this pass):

- Presentations "MAY be used to aggregate information from multiple verifiable credentials" — one presentation can carry several credentials about the same or related subjects via the `verifiableCredential` property.
- Presentations "SHOULD be extremely short-lived and bound to a challenge provided by a verifier" — the anti-replay property that separates a live presentation from a forwarded copy.
- The `type` property MUST be present and MUST include `VerifiablePresentation`; `id` and `holder` are optional (if `holder` is present it should dereference to machine-readable holder information; if absent, holder binding is delegated to the securing mechanism).
- The spec is explicit about the limits: unless the securing mechanism or protocol adds binding, "a verifier cannot generally assume that the verifiable presentation correlates with the presented verifiable credentials" — i.e., by default a VP does not prove the presenter is the credential's subject; **holder binding is an additional mechanism** (a signature over a challenge, a key-binding proof, a `cnf` claim), chosen per ecosystem.

v1.0 carried the same concept (§6.3.1 of v1.0 even defined the JWT `vp` claim). The v2.0 text verified this pass adds the modern framing: presentations may embed **enveloped verifiable credentials** (credentials wrapped in their securing envelope, referenced via `data:` URLs — verified in the v2.0 figures) rather than only inlined JSON-LD graphs.

### 2.4 The DID Layer: Inherited, Not Re-Derived

**Decentralized Identifiers (DIDs) v1.0 is a W3C Recommendation of 19 July 2022** (w3.org/TR/did-core) — this fact is ✅ inherited from [web3_technologies_guide.md](web3_technologies_guide.md) (§8.1 there, where it sits in the Web3 identity/storage/oracles layer map), and the w3.org/TR/did-core document and the W3C announcement of the ratification were both confirmed to exist this pass. Per repo convention this guide does not re-derive DID mechanics; the relevant cross-reference facts are:

- A DID is a URI whose controller holds the keys, decoupled from central registries — the identifier layer that VC `issuer`/`credentialSubject`/`holder` fields typically point at (web3 guide §8.1).
- **DID methods** define how a particular DID type is created/resolved. The W3C DID-methods registry (successor to the DID Specification Registries, now maintained as the "DID Methods" collection under w3c.github.io/did-extensions/methods/) lists hundreds of methods; the five most relevant to this guide are all **registered**, verified this pass against the registry's own JSON entries: `did:web` (spec: w3c-ccg.github.io/did-method-web), `did:key` (w3c-ccg.github.io/did-key-spec — purely generative, no registry), `did:ethr` (Ethereum, spec in the decentralized-identity/ethr-did-resolver repo), `did:sov` (Sovrin ledger), and `did:jwk` (ledger-agnostic, key expressed as a JWK). ✅
- Note the registry's own disclaimer (verified on the registry page): listing "does not act as an endorsement of any particular DID method" — registration is a discovery mechanism, not a quality mark.
- **DID Resolution** — the specification that would standardise resolving a DID to a DID document — did not advance to Recommendation with DID Core; it remains community-level work. ⚠ The CCG-hosted DID Resolution draft page was not reachable this pass, so its current home/status could not be re-verified; treat resolution as implemented per-method (the ethr-did-resolver, did:web HTTPS dereferencing) rather than per a single ratified resolution spec.

### 2.5 The Satellite Specs: Status, Proofs, Schemas, and Securing

Verified this pass at w3.org (statuses as of September 2026):

| Spec | Short name | Status (verified) |
|---|---|---|
| Bitstring Status List v1.0 | vc-bitstring-status-list | **W3C Recommendation, 15 May 2025** (renamed from StatusList2021 on the road to REC; first public working draft 27 April 2023) — §6.1 |
| Verifiable Credential Data Integrity 1.0 | vc-data-integrity | **W3C Recommendation, 15 May 2025** — the embedded-proof/Data Integrity framework — §5.1 |
| Data Integrity EdDSA Cryptosuites v1.0 | vc-di-eddsa | **W3C Recommendation, 15 May 2025** — §5.2 |
| Data Integrity ECDSA Cryptosuites v1.0 | vc-di-ecdsa | **W3C Recommendation, 15 May 2025** — §5.2 |
| Data Integrity BBS Cryptosuites v1.0 | vc-di-bbs | **Candidate Recommendation Draft** (latest 7 April 2026) — not yet a Recommendation — §5.3 |
| Securing VCs using JOSE and COSE | vc-jose-cose | **W3C Recommendation, 15 May 2025** — §4.4 |
| Verifiable Credentials JSON Schema Specification | vc-json-schema | **Candidate Recommendation Draft** (latest 4 February 2025) — not yet a Recommendation — §2.5 below |
| Verifiable Credentials Use Cases | vc-use-cases | W3C Note cited by both v1.0 and v2.0 as the use-case source document ✅ (referenced in the v2.0 introduction) |

**VC JSON Schema** (a W3C deliverable, CRD as of 4 February 2025) is the sanctioned way to describe VC shapes — the data model's answer to "what fields may this credential type carry?", used to validate credentials and to express the schemas that issuers publish for their credential types. Its CRD status means production users should treat it as stable-in-practice but not yet ratified. ⚠ its ecosystem weight is real but its adoption is younger than the data model's.

### 2.6 The W3C Process Context: Working Group, Community Group, Registries

- **The Verifiable Credentials Working Group** (w3.org/groups/wg/vc) is the Recommendation-track body that published the v1.0/v1.1/v2.0 data model and the 15 May 2025 satellite suite — verified on the spec pages themselves ("published by the Verifiable Credentials Working Group as a Recommendation"). Its predecessor, the Verifiable Claims Working Group, produced the use-cases Note the specs cite.
- **The W3C Credentials Community Group (CCG)** is the incubation layer where much of the ecosystem's vocabulary was born and where method specs such as `did:web` and `did:key` still live (verified: those specs are hosted under w3c-ccg.github.io). CCG artifacts referenced by the standards include the VC Extension Registry and Linked Data Proofs/CCG draft suites (e.g. the BBS+ 2020 linked-data proof draft — ⚠ that draft's page was not re-read this pass; its successor, the WG-track vc-di-bbs, is the actively maintained artefact, §5.3).
- **Registries** matter as the standard's "trust-free" coordination layer: the DID-methods registry (w3c.github.io/did-extensions/methods/, verified §2.4) and the W3C-maintained JSON-LD context/term registries for the v1/v2 credential vocabularies. Registration is lightweight and non-endorsing — which is precisely why it scales and why it must not be mistaken for governance (§3.2).

## 3. The Ecosystem: Roles, Trust Fabric, and Programmes

**This section maps the ground beyond the standards text: who the three parties are, what has to be true for a verifier to trust an issuer it has never met, and which industry programmes are building the wallet and governance layer.** The roles come straight from the data model's abstract (verified §2.2); the trust fabric and the programmes are where the W3C specifications deliberately stop and the ecosystem begins.

### 3.1 The Trust Triangle: Issuer, Holder, Verifier

**The issuer makes the claim; the holder carries it; the verifier checks it.** The v2.0 abstract (verified this pass) defines exactly these three roles, and they have been the model's spine since v1.0; the community has called the shape the "trust triangle" since the Verifiable Claims Working Group era (⚠ that exact phrase is community shorthand, not a quoted term in the Recommendation). For a bank the mapping is immediate:

- **Issuer** — the entity that asserts claims about a subject and signs them into a credential: a government issuing an identity credential, an employer issuing an employment-verification credential, or (in §12) a bank issuing a KYC-attestation credential to its own client. The issuer's two obligations are the claim's *quality* (it stands behind what it asserts) and the *key's custody* (the signing key must stay controlled and alive).
- **Holder** — the party that controls the credential — typically, in banking flows, the client. The holder stores the credential in a wallet, chooses what to present, and is the party whose consent drives privacy. v2.0's large privacy section (§8.9–§8.13, verified §2.2) exists mostly to protect this role: data minimisation, bearer-credential warnings, and aggregation-correlation cautions all assume a holder who should not be forced to over-share.
- **Verifier** — the party that receives a verifiable presentation and decides whether to accept it. The verifier's job decomposes into a mechanical check-list — schema, proof, status, issuer trust, holder binding (§7.3) — plus an *acceptance policy*: which issuers this bank trusts for which claims, at which assurance level. The policy is the verifier's own risk decision; the standards do not make it for you.

**One role does not imply another.** A credential about subject S may be issued by I, held by H (not necessarily S — a parent holding a child's credential, a compliance officer holding a corporate's), and checked by V. v2.0 is explicit that a presentation does not by itself prove the presenter is the credential's subject (§2.3) — binding holder to credential is a mechanism the flow must add (§7.2), which is precisely where replay and fraud live.

### 3.2 The Trust Fabric: Registries, Trust Anchors, Governance

**A verifier accepts a credential when three questions resolve: is the signature valid, is the issuer trustworthy for this claim, and is the credential still live?** The W3C standards answer the first (cryptographic suites, §5) and standardise the envelope of the third (status lists, §6); the second is where the trust fabric lives, and it is deliberately *not* standardised by the data model:

- **DID documents** give the issuer a stable identifier and publish the verification method (public key) that validates the signature. The mechanics are inherited from [web3_technologies_guide.md](web3_technologies_guide.md), not re-derived here (§2.4).
- **Registries** are the lightweight, non-endorsing coordination layer: the W3C DID-methods registry lists methods as "registered" (✅ verified §2.4) while carrying the disclaimer that listing "does not act as an endorsement of any particular DID method." Registration is discovery, not governance — the most common misreading of the ecosystem, and the audit (§13.1) flags it.
- **Trust anchors and trust lists** are where actual authority lives: a government's roster of accredited issuers, a sector authority's list of approved attestors, a consortium's agreed issuer registry. The EU's QEAA regime (§11.2) is this pattern at regulation scale — member states accredit qualified trust-service providers whose attestations relying parties are obliged to accept.
- **Governance frameworks** wrap the anchors in rules: who may issue which credential types, what assurance the issuance process must meet, how disputes and revocation are handled. The Trust over IP stack (below) exists largely to give these frameworks a shared vocabulary.

**The consequence for a bank:** accepting a VC is never only a cryptographic check — it is a policy decision about the issuer. That is why §12's worked example separates "proof checks" (mechanical) from "issuer-trust checks" (policy), and why the Cymbal Bank pilot keeps its own trust registry rather than assuming one.

### 3.3 The Ecosystem Programmes

Two Linux Foundation programmes dominate the neutral-industry layer of the ecosystem (both verified only to the depth the cached search material supported — see §13.2):

- **OpenWallet Foundation** — a Linux Foundation project **launched February 2023** (openwallet.foundation). ✅ its existence and launch month are verified from cached material; its mission is a neutral home for interoperable wallet software rather than a single vendor product. ⚠ anything beyond the launch (membership scale, specific codebases) was not re-verified this pass.
- **Trust over IP Foundation (ToIP)** — a Linux Foundation project **founded 2020** ✅, best known for its four-layer "trust over IP" stack separating the *network* layer (DIDs, transports), the *exchange* layer (credential protocols), the *governance* layer (frameworks and rules), and the *ecosystem* layer — the vocabulary that makes §3.2's fabric discussable across organisations. ⚠ the four-layer detail rests on cached snippets; treat the layer names as indicative rather than normative.

**Why a bank should track the programmes:** wallets are the distribution bottleneck of the whole model (§7.4), and wallet software is where the OpenWallet Foundation, the EUDI implementations (§9.3), and commercial vendors converge in practice. A wallet-acceptance roadmap should track the neutral programmes rather than any single vendor's format — the interoperability realities of §7.5 are exactly what those programmes exist to soften.

## 4. The Formats: JSON-LD, JWT Conventions, and SD-JWT

**The same credential can ride in several byte formats; the format is a transport decision, and the history of VC deployment is largely the history of that decision.** This section separates the model from its serializations, then walks the three that matter: JSON-LD (the data model's native dialect, with linked-data proofs), the v1.0 JWT conventions that powered the industry's first wave, and the v2.0-era securing stack (vc-jose-cose, SD-JWT as RFC 9901).

### 4.1 Model versus Serialization

**The data model is not a file format.** The *Verifiable Credentials Data Model* defines the conceptual credential — issuer, subject, claims, proof, validity — and, since v2.0, a conformance regime: a conforming document MUST be a verifiable credential with media type `application/vc` or a verifiable presentation with media type `application/vp`, and MUST be secured by at least one securing mechanism (v2.0 §4.12, verified §2.2). How those bytes are arranged is left to the serialization:

- **JSON-LD** — a linked-data graph, native to the model; typically secured with an embedded Data Integrity proof (§4.2, §5.1).
- **JWT (v1.0 conventions)** — the RFC 7519 token wrapping the credential in its `vc` claim (§4.3); the first-wave industrial format.
- **SD-JWT** — the selective-disclosure JWT of RFC 9901, with hashed claims and released disclosures (§4.4).
- **JOSE/COSE envelopes** — the v2.0-era family specified in vc-jose-cose, covering plain JWT, COSE (CBOR), and SD-JWT carriers (§4.4).

v2.0 frames all of this as two **securing families** (verified §2.2): **embedded proofs** (Data Integrity — the proof sits inside the credential graph) versus **enveloping proofs** (the credential is wrapped in an external token). Choosing a serialization is choosing a family, an ecosystem, and a set of verifier capabilities — not a purely cosmetic choice.

### 4.2 JSON-LD and Linked-Data Proofs

**JSON-LD is the data model's native dialect**: the `@context` maps every term to a global identifier (IRI), so claims stay machine-disambiguable across issuers. v2.0 uses the new JSON-LD context namespace `https://www.w3.org/ns/credentials/v2` (✅ verified §2.2), replacing the v1 context — a migration detail any JSON-LD deployment must track. A minimal, clearly illustrative credential:

```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2",
               "https://example.gov/ns/identity/v1"],
  "id": "urn:uuid:9c1e2f3a-0000-4000-8000-000000000001",
  "type": ["VerifiableCredential", "ProofOfAgeCredential"],
  "issuer": "did:web:example.gov",
  "validFrom": "2026-01-02T00:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6Mkexample0000000000000000000",
    "birthDate": "1990-04-12"
  }
}
```

*(Illustrative only — in production a Data Integrity proof of §5.1 would be attached, and the subject's `birthDate` would be the minimum necessary under the v2 privacy guidance.)*

**The proof lineage matters.** The first embedded-proof formats were the Credentials Community Group's Linked Data Proofs drafts (CCG is the incubation layer, §2.6); the WG-track successor is **Verifiable Credential Data Integrity 1.0**, a W3C Recommendation of 15 May 2025 (§5.1). JSON-LD + Data Integrity is the "native stack" that the W3C's own examples use, and the one with the richest semantics (an RDF graph that can be queried and selectively re-serialized).

### 4.3 The v1.0 JWT-VC Conventions: the `vc` and `vp` Claims

**The first industrial wave of verifiable credentials rode on JWTs — and v1.0 standardised exactly that.** Section 6.3.1 of the v1.0 Recommendation (verified §2.1) defined a JWT serialization in which the JWT is an ordinary RFC 7519 token whose payload carries the registered claim **`vc`** (holding the verifiable credential) or **`vp`** (holding the verifiable presentation), with the JWT's own registered claims mapped onto the credential's semantics: `exp` (expirationDate), `iss` (issuer), `nbf` (issuanceDate / not-before), `jti` (credential id), `sub` (credentialSubject id), and `aud` (intended verifier). ✅

**This is why so many existing integrations are "JWT-VCs".** The format was part of the ratified v1.0 spec, so the first wave of bank, government, and identity-provider pilots shipped it with a clean conscience — and those credentials are not broken today. But the centre of gravity moved: **v2.0 removed the JWT serialization from the data model and delegated JWT/SD-JWT/COSE to vc-jose-cose** (§2.2, §4.4). For a bank running a v1.0 JWT-VC integration, the migration question is not "does it still work?" (it does) but "does it sit on the standard's roadmap?" — and the answer is that new work assumes the v2 arrangement.

### 4.4 v2.0 Securing: vc-jose-cose and SD-JWT (RFC 9901)

**The v2.0 enveloping-proof family is specified in *Securing Verifiable Credentials using JOSE and COSE* (vc-jose-cose), a W3C Recommendation of 15 May 2025** (✅ §2.5): it defines how to secure credentials with plain JOSE (JWT), COSE (the CBOR equivalent), and SD-JWT, and it is the sanctioned home of what §4.3's conventions used to do.

**SD-JWT is now an IETF RFC.** **RFC 9901, published November 2025** (✅ verified against the RFC-editor text) defines Selective Disclosure JWT: the issuer replaces each disclosable claim with a salted digest — an `_sd` entry — inside the JWT payload, and releases the matching plaintext disclosures only at presentation time. The verifier re-hashes each disclosure and checks it against the digest set, so it learns only the released claims. Key binding is optional via the **`cnf`** claim — a public key the holder must prove possession of, which stops a copied token from being replayed by someone else. Structure, illustratively:

```
SD-JWT = JWT( payload: { "_sd": [ hash(d1), hash(d2), ... ], "cnf"?: {...}, ... } )
       + Disclosures [ d1, d2, ... ]           (released selectively at presentation)
       + optional KB-JWT                       (key-binding proof over cnf + challenge)
```

**The mapping layer is still a draft.** The SD-JWT VC profile (draft-ietf-oauth-sd-jwt-vc) that maps SD-JWT onto the VC data model is an active IETF draft, ⚠ not yet an RFC (✅/⚠ §1.2). ⚠ It is widely reported to be the format most EUDI-wallet technical work gravitates toward, but that pattern was not re-verified at a primary source this pass. The bank-relevant read: v1.0 JWT-VC (whole-credential, readable by any JWT library) and SD-JWT (selectively disclosable, privacy-preserving) answer different needs, and v2.0's vc-jose-cose supports both — the choice belongs to the ecosystem profile (§7.5).

## 5. The Signatures and Privacy Technology

**Underneath the data model sit the cryptographic suites that make a claim provable, and the choice of suite determines how much privacy the holder keeps.** This section covers the Data Integrity family (the v2.0 embedded-proof framework and its cryptosuites), the Ed25519/ECDSA workhorses, BBS+ selective disclosure and zero-knowledge proofs, and the trade-offs — correlation, privacy, performance — that decide which suite fits which banking use case.

### 5.1 The Data Integrity Suite Family

**Data Integrity is the v2.0 embedded-proof framework — a W3C Recommendation of 15 May 2025** (✅ §2.5). Where v1.0-era deployments bolted proofs on as an afterthought, Data Integrity standardises the proof itself: a `proof` node inside the credential graph (or attached to it) carrying the proof type, the cryptosuite, the verification method, the proof purpose, and a created timestamp. The family relation:

- **Verifiable Credential Data Integrity 1.0** (vc-data-integrity) — the framework: how proofs are created, added, and verified against a credential's canonical form (⚠ the canonicalisation details are normative machinery; this guide treats the framework's existence and role as the verified claim).
- **Cryptosuites** — the concrete signature algorithms bound into the framework, each a separate spec: the EdDSA suites (vc-di-eddsa), the ECDSA suites (vc-di-ecdsa), and the BBS suites (vc-di-bbs). A credential's proof states its `cryptosuite` by name (e.g. `eddsa-2022`), so a verifier knows exactly which algorithm to run.
- **The lineage** runs from the CCG-era Linked Data Proofs drafts (§2.6) through Data Integrity — the WG-track spec is the maintained artefact, and v2.0's own examples use it.

For a bank, the framework matters because it makes proof *introspectable*: a verifier can list which suites it accepts and reject the rest, which is exactly the acceptance-policy mechanism §3.2 asked for.

### 5.2 Ed25519 and ECDSA Suites: the Workhorses

**The overwhelming majority of deployed VCs are signed with Ed25519 or ECDSA — both now have ratified W3C cryptosuites:**

- **Data Integrity EdDSA Cryptosuites v1.0** (vc-di-eddsa) — **W3C Recommendation, 15 May 2025** ✅, Ed25519-based (the `eddsa-2022` suite in the §12 example).
- **Data Integrity ECDSA Cryptosuites v1.0** (vc-di-ecdsa) — **W3C Recommendation, 15 May 2025** ✅, covering the NIST P-256/P-384 curves commonly found in government and hardware-security modules.

**Why they are the workhorses:** Ed25519 and ECDSA key material is ubiquitous — in HSMs, secure elements, cloud KMS, and every modern TLS stack — so issuers can sign with hardware they already operate, and the signatures are small and fast to verify. The price is privacy-shaped: these are *whole-credential* signatures. Presenting the credential presents every signed claim, and every presentation carries the same signature, so a verifier (or a colluding set of verifiers) can correlate presentations of the same credential across services — the aggregation-correlation caution v2.0's privacy section warns about (verified §2.2). If selective disclosure is required, the suite must change.

### 5.3 BBS+: Selective Disclosure and Zero-Knowledge

**BBS+ is the privacy suite: a pairing-based signature scheme that lets a holder prove a subset of the signed attributes — or predicates over them ("over 18") — without revealing the rest or linking the presentations.** The mathematics is domain-stable and uncontested (pairing-based signatures with selective disclosure and zero-knowledge proofs). What is *not* finished is the standardisation status:

- **Data Integrity BBS Cryptosuites v1.0** (vc-di-bbs) — **Candidate Recommendation Draft, latest 7 April 2026** ⚠ — not yet a W3C Recommendation (✅/⚠ §1.2, §2.5). Production users should treat the suites as stable-in-practice but not ratified.
- The CCG-origin "BBS+ 2020" linked-data proof draft is the predecessor artefact (§2.6); the WG-track vc-di-bbs is the maintained successor.

**What BBS+ buys a bank's clients:** a corporate client can prove "our beneficial owners have all passed screening" from a signed attestation without disclosing who the beneficial owners are; an individual can prove age from an identity credential without revealing the birth date. **What it costs:** larger signatures, pairing-based verification (slower, less hardware support), and ecosystem adoption that still lags the workhorses. ⚠ deployment maturity is therefore uneven — treat BBS+ as a capability to *support in verification* long before issuing with it.

### 5.4 The Trade-Offs: Correlation, Privacy, Performance

**Suite choice is a three-way trade, and the right answer depends on the claim being carried:**

| Suite family | Selective disclosure | Presentation linkability | Verification cost | Status |
|---|---|---|---|---|
| Ed25519 (eddsa-2022) | None — whole credential | High — same signature every time | Very low | ✅ REC 15 May 2025 |
| ECDSA (P-256/P-384) | None — whole credential | High | Low | ✅ REC 15 May 2025 |
| BBS+ (vc-di-bbs) | Yes — subset + predicates | Low — derived proofs unlinkable | Higher (pairings) | ⚠ CRD (7 Apr 2026) |
| SD-JWT (RFC 9901) | Yes — hash-bound claims | Medium — `_sd` JWTs still carry the issuer signature | Low | ✅ RFC Nov 2025 (§4.4) |

**The design guidance that falls out:** use Ed25519/ECDSA where the whole credential is the unit of disclosure (a KYC attestation shown to one onboarding desk) and correlation risk is acceptable; use SD-JWT where claims must be peeled apart (identity attributes presented service-by-service); use BBS+ where *unlinkability across verifiers* is itself the requirement (a frequent traveller proving credentials at many checkpoints). The v2 data model supports all of them — which is the point: the provable claim should not dictate the privacy posture, the use case should.

## 6. Revocation and Status

**A credential's signature proves it was issued; it does not prove it is still valid.** Revocation and status are the machinery that lets an issuer retract a credential (a client exits, a licence lapses, a key is suspected compromised) and lets a verifier check before relying. The W3C answer is the Bitstring Status List; the ecosystem also runs older registry-based and on-ledger approaches, and the choice between them is really a choice about online versus offline verification.

### 6.1 Bitstring Status List v1.0: W3C Recommendation, 15 May 2025

**Bitstring Status List v1.0 is a W3C Recommendation of 15 May 2025** (✅ §2.5) — the spec formerly known as StatusList2021 (renamed on the road to Recommendation; first public working draft 27 April 2023). Its mechanism, verified in the Recommendation text:

- The issuer maintains a **statusListCredential** — which is itself a verifiable credential — whose payload is a bitstring (each bit encodes one credential's status, e.g. revoked or not).
- Each issued credential carries a **`credentialStatus`** property referencing its position: the status credential's URL (`statusListCredential`) plus an index (`statusListIndex`).
- To check status, a verifier fetches the status credential, verifies it (it is a VC — it carries the issuer's own proof), reads the bit at the index, and decides.

**Why the design is clever for privacy:** one status credential serves many issued credentials, so the status endpoint reveals only "this bitstring changed", not which credential was checked — and the verifier learns nothing about any other holder. **Why it needs care:** the status credential must be *fetchable*, which makes verification depend on the issuer's endpoint being online (or the verifier caching status — §6.3).

### 6.2 Registries, On-Ledger Revocation, and the Alternatives

**W3C status lists are not the only revocation machinery, and the alternatives confuse cleanly with VC infrastructure at the ledger edge:**

- **Registry-based revocation** — the issuer (or a consortium) operates a revocation registry listing revoked credential IDs; the verifier queries it. Simpler than bitstrings, but the query itself can reveal which credential is being checked — a correlation leak the bitstring design exists to avoid.
- **On-ledger revocation** — the registry is a smart contract or an anchored hash on a blockchain. The mechanics (contracts, token standards, the EIP/ERC process) live in [smart_contracts_guide.md](smart_contracts_guide.md); the VC-specific point is that anchoring status *hashes* on-chain buys tamper-evidence and decentralised availability at the price of gas, latency, and public visibility of the anchor activity.
- **On-chain KYC tokens are a different beast.** A tokenised "KYC done" badge living on a public ledger is not a W3C verifiable credential: the ledger makes the attestation visible to everyone, permanently, which inverts the VC privacy model (§5.4) and collides with data-minimisation expectations. Where the smart-contracts sibling discusses "on-chain KYC", read it as the token model; this guide's §12 follows the W3C VC model instead. The two interoperate at the *edge* (a VC's status anchored on-chain) rather than by identity of mechanism.
- **Short-lived credentials** — expire the credential quickly and skip revocation entirely (validFrom/validTo of hours or days). Popular for access and ticketing; impractical for multi-year KYC attestations.

### 6.3 The Trade-Offs: Online versus Offline Verification

**Status checking is the one step in verification that can force a network round-trip, and every design choice above is a bet on availability:**

| Approach | Freshness | Availability dependency | Privacy | Typical use |
|---|---|---|---|---|
| Bitstring Status List | Good — bit flips fast | Needs issuer endpoint (or cached copy) | High — no per-credential query leak | Ratified default (§6.1) |
| Registry query | Good | Needs registry endpoint; query reveals credential | Lower — query is linkable | Legacy/consortium |
| On-ledger anchor | Good | Needs chain access; censorship-resistant | Mixed — anchors public | High-assurance, cross-border |
| Short-lived credential | Perfect — no revocation needed | None after issuance | High | Access, sessions, tickets |

**The verifier's practical rules:** batch status checks where possible, cache status credentials with a freshness budget, treat "status endpoint unreachable" as a policy decision (reject, or accept with ⚠ flagged risk and compensating checks), and never let status availability become a single point of failure for a whole onboarding queue. §12.4's worked example puts the status check on the critical path and plans for the offline case explicitly.

## 7. Protocols and Flows

**The data model defines the objects; the flows move them between the three roles.** This section walks issuance, presentation (holder binding, derivation, challenges), verification, wallets (condensed — the custody mechanics live in the Fireblocks sibling), and the interoperability realities that still fragment the ecosystem in practice.

### 7.1 Issuance Flows

**Issuance is the moment the issuer converts a verified fact into a signed credential — and the moment the issuer's own KYC/EDD discipline matters most.** The canonical flow: the issuer authenticates the subject (to its own assurance level), assembles the claims, signs the credential with its chosen suite (§5), optionally registers it in a status list (§6), and delivers it to the holder's wallet. Delivery shapes vary:

- **Direct push** — the issuer sends the credential to a wallet URI or the holder pulls it via a deep link or QR (the common consumer pattern).
- **Delegated/mediated** — a trusted third party holds or forwards the credential on the issuer's behalf.
- **Key binding at issuance** — the credential's `credentialSubject.id` is the holder's DID, and the holder proves control of the corresponding key at issuance, so the credential is born holder-bound rather than bearer (§7.2).

⚠ The dominant *issuance protocol* in the field — OpenID for Verifiable Credential Issuance (OID4VCI) — is an OpenID Foundation specification, not a W3C deliverable, and was not re-verified at its source this pass; treat it as the ecosystem's de facto transport, with the W3C/IETF documents of §4–§5 as the content layer.

### 7.2 Presentation: Holder Binding, Derivation, Challenge

**Presentation is where the holder turns stored credentials into a verifier-facing wrapper — the verifiable presentation (VP) of §2.3.** Three properties do the security work:

- **Challenge binding.** v2.0 says presentations SHOULD be extremely short-lived and bound to a verifier challenge (§2.3, verified). The verifier mints a random challenge (plus a target `aud`); the holder's presentation is bound to it; a captured presentation cannot be replayed at a second verifier or a second login.
- **Holder binding.** Recall §2.3's warning: by default, a VP does not prove the presenter is the credential's subject. Binding is added by mechanism — a signature over the challenge using a key named in the credential or VP, an SD-JWT `cnf` key-binding proof (§4.4), or a BBS+ derived proof tied to the holder's key (§5.3). A bank should refuse bearer presentations for anything above the lowest risk tier.
- **Derivation.** With selective-disclosure formats the holder presents a *derived* form — SD-JWT disclosures peeled to the minimum (§4.4) or a BBS+ derived proof (§5.3) — so the verifier sees only what the flow requires.

### 7.3 Verification Flows

**Verification is a check-list, and the order matters:**

1. **Syntax and schema** — the credential parses; its type and shape match the expected credential definition (VC JSON Schema, ⚠ still CRD, §2.5).
2. **Proof** — the signature validates under the declared cryptosuite (§5.2); the verification method resolves via the issuer's DID document (§2.4, inherited).
3. **Status** — the credential is not revoked/expired (§6).
4. **Issuer trust** — the issuer is in the verifier's trust anchor set for this credential type (§3.2) — the policy step, not a crypto step.
5. **Holder binding** — the presenter proved control of the bound key (§7.2).

Only when all five pass does the claim enter the bank's own decisioning. §12.4 runs this list against a concrete onboarding; §8.2 notes the regulated-entity variant where steps 4–5 are also regulatory obligations.

### 7.4 Wallets, Condensed

**The wallet is where keys live, credentials rest, and presentations are built — and therefore where the ecosystem's risk concentrates.** For the institutional side — custody-grade key management, HSM integration, quorum controls, the segregation of signing authority — this guide condenses to a cross-ref: [fireblocks_guide.md](../banking/fireblocks_guide.md) carries the institutional wallet/custody mechanics and is the wallet-security twin for this section; do not re-derive them here. The VC-specific wallet facts this guide adds: a wallet holds (a) the holder's keys (the DID controller keys that sign presentations and key-binding proofs) and (b) the holder's credentials; wallet security is application security plus key management, and the threat model belongs to [cybersecurity_guide.md](cybersecurity_guide.md). Consumer wallet fragmentation is a first-order interoperability problem (§7.5); institutional wallets are fewer, and the OpenWallet Foundation (§3.3) exists to keep the software layer from fragmenting further.

### 7.5 The Interoperability Realities

**The honest headline of the VC ecosystem in 2026: the standards are ratified, the deployments are fragmented.** ⚠ The following realities are reported field conditions, assembled from the verified spec landscape rather than from a single survey:

- **The format camps.** JSON-LD/Data Integrity deployments and JWT/SD-JWT deployments verify each other's credentials only through bridging layers; vc-jose-cose (§4.4) now provides the sanctioned JWT-family home, but an ecosystem profile still has to pick a lane — or build both.
- **Multiple proof suites.** eddsa-2022, ECDSA suites, BBS+ (still CRD), plus legacy v1.0-JWT and LD-Proofs artifacts — a verifier's suite allow-list (§5.1) is a real configuration surface.
- **Wallet fragmentation.** Consumer wallets from governments (EUDI, Singpass-era apps), foundations (OpenWallet Foundation members), and vendors do not all speak one presentation protocol; the OID4VP/OID4VCI layer ⚠ (OpenID Foundation, not re-verified this pass) is the de facto glue, not a W3C standard.
- **Government profiles.** The EUDI Wallet (§11.2) and national schemes define their own profiles on top of the W3C/IETF base — a bank serving multiple jurisdictions verifies the same credential types under different profiles.

**The strategic consequence, restated from §1.3:** build acceptance capability across the ratified formats and profiles first, issue into the formats your own flows control, and let the profiles converge underneath you.

## 8. Use Cases I — KYC and Onboarding

**The use case that pays for the whole stack in banking: turning the outcome of one KYC process into a portable, verifiable attestation that the next desk, product, or institution can check without re-running the interview.** This section builds strictly on the compliance baseline carried by the banking siblings — the credential changes how evidence is carried, not what evidence the law requires.

### 8.1 Credential-Based KYC

**"Portable KYC" is the decade-old industry goal that VCs finally give a standard envelope.** The idea: a regulated entity completes KYC/EDD on a client and issues a KYC-attestation credential; the client stores it; the next relationship or product verifies it instead of re-collecting documents. What the credential adds is *provability and freshness* — a signed, status-checkable attestation (§6) from a named issuer (§3.2) — over the earlier proprietary-token attempts at the same idea.

**What the credential does NOT remove:** the receiving institution's own obligations. Cross-ref [private_banking_guide.md](../banking/private_banking_guide.md) for the KYC/EDD conventions that remain the compliance baseline — customer due diligence, beneficial-ownership look-through, screening, risk rating, and the record-keeping that goes with them — and [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) for the SG/MAS posture any VC-based onboarding in Singapore must fit. Do not re-derive either here; the worked example in §12.4 shows the division of labour (credential checks first, the bank's own EDD still runs).

### 8.2 The Regulated-Entity Verification Flows

**In the regulated world, the same institution plays both sides of the market:**

- **As issuer** — a bank or licensed entity that has completed KYC issues attestation credentials to clients (or, in a consortium model, to other members' clients). Issuance is itself a regulated act: the credential's claims must match the due-diligence record, and the issuer remains accountable for them.
- **As verifier** — the bank accepts government-issued or peer-issued credentials at onboarding. The verification check-list of §7.3 becomes a compliance control: schema, proof, status, and — decisively — *issuer trust* (is this issuer in the acceptable set for this claim type?) and *holder binding* (is the presenter the subject?). Under eIDAS 2.0, parts of this become obligation rather than choice for EU-facing flows (§11.3).
- **The assurance mismatch to manage:** a KYC attestation from another institution reflects *that* institution's process, not yours. Regulators generally require the relying party to remain responsible for the outcome — hence the "credential plus own EDD" pattern of §12.4 rather than credential-only onboarding, and why the private-banking sibling's conventions stay authoritative.

### 8.3 The Account-Opening Flow

**An account-opening flow with credential-based KYC, for a corporate client** (the §12 scenario in miniature; singpass-style national-ID variants are §9):

```
 Client wallet             Bank onboarding            Bank systems
     |                          |                          |
     |--- VP (KYC attestation) -->|                          |
     |   + challenge signature   |--- 1. schema check ------>|
     |                           |--- 2. proof verify ------>|
     |                           |--- 3. status check ------>|  (status list, §6)
     |                           |--- 4. issuer trust -------|  (trust registry, §3.2)
     |                           |--- 5. holder binding -----|
     |                           |                          |
     |                           |--- own EDD (look-through, |
     |                           |    screening, risk rate)-|>  (per private-banking guide)
     |<-- account opened, VC -----|                          |
     |      issuance offered      |                          |
```

The flow's economics: steps 1–5 are minutes of machine time; the human cost concentrates in the residual EDD, which is exactly where the private-banking sibling's conventions apply. §12 expands this diagram into the full worked example.

## 9. Use Cases II — Digital Identity: Singpass and the EUDI Wallet

**National digital identity is the biggest real-world deployment of "provable identity" — and the most instructive lesson in what VCs are and are not.** Singapore's Singpass is the flagship in this guide's home market; the EUDI Wallet is the regulatory-scale European counterpart. The contrast between them — centralised consent versus decentralised credentials — is the section's through-line.

### 9.1 Singpass and MyInfo: Singapore's National Digital Identity

**Singpass is Singapore's national digital identity — a Smart Nation strategic project built by GovTech, with the Singpass app in market since 2018** ✅ (verified against the cached GovTech page), providing secure login to government and private-sector digital services for citizens and businesses. **MyInfo** is the companion consent-based personal-data service that pre-fills forms with government-verified data — the citizen consents *per transaction* to release specific fields (name, NRIC, address, and so on) to the requesting service. ✅

**The distinction this guide insists on:** Singpass/MyInfo is a **centralised, consent-based** model — the government is the identity provider, data flows from the government's store to each service on the citizen's consent — **not a decentralised verifiable-credential network**. There is no holder-carried signed credential being verified offline; MyInfo is an API with consent, and the "verifier" is the requesting service talking to GovTech. This matters for architecture: Singpass gives Singapore world-class digital onboarding (see the SG banking cluster — [dbs_bank_guide.md](../banking/dbs_bank_guide.md), [uob_software_systems_guide.md](../banking/uob_software_systems_guide.md), [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [maribank_guide.md](../banking/maribank_guide.md), [trust_bank_guide.md](../banking/trust_bank_guide.md), [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) — which carry the onboarding context and are cited, not re-derived), but it is a different trust model from the W3C issuer/holder/verifier triangle of §3.1, and §12.5 draws the architectural consequence.

### 9.2 Singapore's Verifiable-Credential Plans

**Singapore has publicly signalled VC work without (so far) a Singpass-scale VC deployment.** ✅ GovTech announced verifiable-credential activity around the Singapore FinTech Festival 2022 period; cached snippets reference tech.gov.sg material and the docs.cdpi.dev "Verifiable Credentials" technical notes. ⚠ Specifics beyond "announced/piloted" — which credentials, which formats, which timeline — could not be confirmed at a primary page this pass; see "What Could Not Be Verified" (§13.2). The plausible trajectory, unverified: Singpass infrastructure evolving to *issue* rather than only *release* — a direction consistent with the EUDI model — while MyInfo-style consent APIs remain the workhorse for form pre-fill.

### 9.3 The EU Digital Identity Wallet

**The EU Digital Identity (EUDI) Wallet is the European counterpart — and it is being built on the VC/attribute-attestation stack this guide covers, under the eIDAS 2.0 regulation** (Regulation (EU) 2024/1183 — verified facts and dates in §11.1, which this subsection cross-refs rather than duplicates). The wallet is a member-state-provided app (§11.2) in which citizens hold identity data and qualified electronic attestations of attributes (QEAA) and present them to relying parties. For the purposes of §9, the comparison table:

| Dimension | Singpass/MyInfo (SG) | EUDI Wallet (EU) |
|---|---|---|
| Model | Centralised consent-based data release | Holder-held credentials/attestations, presented |
| Provider | GovTech (government) | Member states (mandated by regulation) |
| Legal basis | National programme | Regulation (EU) 2024/1183 (§11) |
| VC alignment | Not a decentralised VC network (§9.1) | QEAA/VC-aligned architecture in progress ⚠ |

The EUDI story continues in §11 — the regulation, the mandates, and what they mean for banks.

## 10. Use Cases III — Education, Employment, Health, and Travel

**Outside finance, the earliest and deepest VC deployments were in credentials that were already paper credentials: diplomas, certificates, licences, and passes.** This section covers the education standards (Open Badges, Blockcerts), the European learning-credential layer (Europass EDCs, EBSI/Europeum), and the employment/health/travel fringe — the last kept deliberately shallow because the verified-facts digest supports depth only where noted.

### 10.1 Diplomas and Badges: Open Badges and Blockcerts

**Open Badges is the 1EdTech standard for digital badges — the largest installed base of "verifiable" achievement credentials in the world.** ✅ its existence and role are domain-stable; the OB 3.0 line aligns badge credentials with the W3C VC data model — ⚠ exact conformance status was not re-verified this pass (cached material only). For a bank the relevance is indirect but real: employment and education verification is a KYC-adjacent input, and a candidate's degree credential from an Open-Badges issuer is verifiable the same way a KYC attestation is (§7.3).

**Blockcerts is the open standard for blockchain-anchored academic credentials, originated at the MIT Media Lab** ✅ (domain-stable; the MIT Media Lab origin is well established). Its pattern — hash the credential, anchor the hash on a blockchain, verify against the anchor — is the on-ledger family of §6.2 applied to diplomas: tamper-evident, issuer-independent verification, at the price of the public-anchor privacy trade-off noted there. ⚠ Blockcerts' current maintenance status was not re-verified this pass.

### 10.2 The European Layer: Europass EDCs and EBSI/Europeum

**The EU's learning-credential layer runs on two tracks that this guide keeps distinct:**

- **Europass / European Digital Credentials for Learning (EDC)** — the EU's credential framework for learning outcomes and qualifications, issued by education institutions and carried in the Europass ecosystem. ⚠ depth is limited here: the digest supports only a general statement, and the underlying infrastructure story is the EBSI/Europeum one below.
- **EBSI and EUROPEUM-EDIC** — the European Blockchain Services Infrastructure has been the EU's blockchain layer for cross-border use cases including learning credentials; its governance moved in 2024: **the European Commission's EUROPEUM-EDIC decision is dated 21 May 2024, with nine member states, taking over governance of EBSI** ✅ (verified §13.1). The name change matters for anyone integrating: the entity to track is now Europeum, not the older EBSI branding.

### 10.3 Employment, Health, and Travel Credentials

**The adjacent credential families, kept to their verified core:**

- **Employment credentials** — employment-verification and income credentials (an employer attesting "employed since, role, salary") are a natural issuer-holder-verifier fit and a direct input to credit and onboarding decisions; the pattern is the §8 one with the employer as issuer. ⚠ no specific scheme was verified this pass beyond the general pattern.
- **Health credentials** — the pandemic's EU Digital COVID Certificate demonstrated certificate-based health attestation at population scale, and vaccination/test credentials remain the canonical "verifiable health claim" example, but the W3C-VC mapping of health data is governed by medical-data rules this guide does not venture into. ⚠ historical framing only; not re-verified this pass.
- **Travel credentials** — digital travel credentials (the ICAO DTC work) sit at the intersection of national identity (§9) and the VC model; ⚠ the ICAO standardisation status was not verified this pass and is deliberately not claimed here.

The through-line for a bank: every one of these families is a *verification input* to onboarding and credit — and the §7.3 check-list plus the §3.2 trust fabric apply identically whether the claim is a diploma, a vaccination, or a KYC attestation.

## 11. Use Cases IV — Regulatory: eIDAS 2.0 and the EU Digital Identity Framework

**The regulatory case is the one with a calendar.** eIDAS 2.0 turns the EUDI Wallet from a technology project into a legal obligation with deadlines, and it obliges a specific class of relying parties — banks among them — to accept it. All dates in this section were verified against the EUR-Lex text of the regulation and the Commission's public framing (see §13.1).

### 11.1 The Regulation, Verified

**eIDAS 2.0 is Regulation (EU) 2024/1183 of 11 April 2024** (published in the Official Journal 30.4.2024), **in force since 20 May 2024**, amending the original eIDAS Regulation (EU) No 910/2014. ✅ Verified against the EUR-Lex text this pass. Its headline content: it establishes the **EU Digital Identity Wallet** framework — mandating EUDI wallets per member state — introduces **qualified electronic attestations of attributes (QEAA)** as the wallet's attribute currency, and imposes **relying-party acceptance obligations** on services that identify users by electronic means.

**The application dates deserve precision** ✅/⚠:

- Implementing acts under Article 5a(23) were adopted **28 November 2024** ✅; wallet obligations run from the implementing acts plus 24 months.
- The European Commission's public framing: wallets available to citizens by **the end of 2026** ✅ (verified against the cached Commission page).
- The widely quoted **21 November 2026** ⚠ is a *derived* target date (28 November 2024 + 24 months minus the drafting lag), not a date stated in the regulation itself — see "What Could Not Be Verified" (§13.2).

### 11.2 What eIDAS 2.0 Mandates for the Wallet

**The regulation's wallet architecture, in the parts a bank touches:**

- **Mandatory EUDI wallets** — each member state must make an EUDI Wallet available to its citizens and residents; the wallet holds identity data, keys, and QEAA.
- **QEAA** — qualified electronic attestations of attributes issued by qualified trust-service providers: a QEAA is, in VC terms, an attribute credential with a regulatory trust anchor (the member state's qualified list — the §3.2 trust-fabric pattern at regulation scale). QEAAs carry the same legal effect across member states as attested attributes.
- **Acceptance obligations** — relying parties that use electronic identification for user identification (a class that plainly includes banks opening accounts and authenticating customers) must accept the EUDI Wallet and QEAAs for that purpose; they may not discriminate against it in favour of proprietary methods.
- **Relying-party notification** — parties relying on the wallet for high-value services notify their member state, which is how the acceptance surface becomes auditable.

### 11.3 The Implications for Banks

**For a bank, eIDAS 2.0 lands as a capability deadline, not an option:**

- **Wallet-acceptance capability by the end-2026 window** — the §1.3 timetable: any bank with EU operations (or EU clients onboarded remotely) needs the ability to accept EUDI Wallet presentations and QEAA by the time wallets reach citizens; the derived 21 November 2026 date is the widely used planning anchor ⚠.
- **KYC interplay** — QEAA identity attributes feed the identification stage of onboarding, but whether a QEAA *alone* satisfies AML/KYC evidence requirements is a member-state and supervisory question; the compliance baseline of [private_banking_guide.md](../banking/private_banking_guide.md) and, for Singapore-licensed banks, [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) continues to govern. ⚠ the precise supervisory treatment of QEAA in AML/KYC was not verified this pass.
- **Architecture** — acceptance means the full §7.3 check-list against EUDI profiles (§7.5): proof suites, status, and the EU trust anchor set; the wallet's own security (keys, attestations) is a custody problem of the [fireblocks_guide.md](../banking/fireblocks_guide.md) kind for institutional holders.
- **The Singapore mirror** — none of this obliges SG-licensed banks directly, but a Singapore bank with EU clients will meet EUDI presentations at the onboarding desk; the MAS posture remains the local baseline, and §12's pilot is written to be jurisdiction-agnostic in its credential design.

## 12. The Cymbal Bank Worked Example: A Verifiable-Credential KYC Pilot

### 12.1 The Design-Fiction Frame

**This section is design fiction — clearly marked as such.** Cymbal Bank is the repository's only bank persona (per the integrity rules of the research repo), and the pilot described below is an illustrative architecture written to show how the verified standards of §2–§7 fit together in a real-shaped KYC flow. It is not a deployed system, not a product commitment, and not regulatory advice. Its job is to make the abstractions concrete: a credential, a presentation, a status check, and the division of labour between machine-verifiable claims and the bank's own EDD.

### 12.2 The Credential Design

**The pilot's core artefact is a KYC-attestation credential that Cymbal Bank issues to a corporate client after completing onboarding KYC/EDD.** Design decisions, mapped to the guide: the issuer identifier is a `did:web` (registered method, §2.4) so the issuer's DID document — and the verification key — is fetchable over HTTPS; the subject is a `did:key` (purely generative, §2.4) controlled by the client's wallet; the claims are the minimum necessary (data minimisation, §2.2); revocation is via Bitstring Status List (§6.1); the proof is Data Integrity `eddsa-2022` (§5.2). Illustrative JSON-LD:

```json
{
  "@context": ["https://www.w3.org/ns/credentials/v2",
               "https://issuer.cymbal.example/ns/kyc-attestation/v1"],
  "id": "urn:uuid:7f3a9c10-aaaa-4bbb-9ccc-00000000kyc1",
  "type": ["VerifiableCredential", "KycAttestationCredential"],
  "issuer": "did:web:issuer.cymbal.example",
  "validFrom": "2026-03-01T00:00:00Z",
  "validUntil": "2027-03-01T00:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6Mkclient0000000000000000000000",
    "legalName": "Meridian Trading Pte. Ltd.",
    "uen": "2026XXXXXXK",
    "beneficialOwnerDigest": "sha256:3f4a...", 
    "screeningStatus": "cleared",
    "eddTier": "standard"
  },
  "credentialStatus": {
    "type": "BitstringStatusListEntry",
    "statusPurpose": "revocation",
    "statusListIndex": "42",
    "statusListCredential": "https://issuer.cymbal.example/status/kyc-2026"
  },
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-2022",
    "verificationMethod": "did:web:issuer.cymbal.example#key-1",
    "proofPurpose": "assertionMethod",
    "created": "2026-03-01T00:00:05Z",
    "proofValue": "z4o... (signature)"
  }
}
```

*(Illustrative: identifiers are fictional; the `beneficialOwnerDigest` is a salted hash so the attestation proves "owners were looked through" without re-publishing the ownership register — the §5.3 privacy logic in whole-credential form.)*

### 12.3 Issuance: Cymbal Bank as Issuer

**Issuance happens once, at the end of a successful KYC/EDD engagement, and only on the client's request** (the client, not the bank, decides to hold the credential — the consent posture of §9.1's MyInfo comparison applies to banks too). The flow:

1. The client's wallet generates a `did:key` and proves control of it to the onboarding desk (key binding at issuance, §7.1).
2. Cymbal's KYC system, having completed CDD/EDD per [private_banking_guide.md](../banking/private_banking_guide.md), assembles the claim set above — note what is *not* in it: no full ownership register, no supporting documents, no screening detail. The credential asserts outcomes, not evidence.
3. The signing service — an HSM-held Ed25519 key behind the `did:web` document — signs the credential (`eddsa-2022`, §5.2) and registers index 42 in the status credential published at `issuer.cymbal.example/status/kyc-2026` (§6.1).
4. The credential is delivered to the client's wallet; the bank records the issuance in its own ledgers but keeps no copy of the holder's private key.

**The issuer-side controls worth naming:** the signing key never leaves the HSM (custody mechanics cross-ref [fireblocks_guide.md](../banking/fireblocks_guide.md)); the status list is refreshed on a schedule; and issuance is logged as a controlled event, because the bank remains accountable for the claims it signed (§8.2).

### 12.4 Verification at Onboarding: The Corporate-Client Flow

**The payoff scenario: the same client (or a client carrying a Cymbal-issued attestation) opens a second product, or a partner institution accepts the attestation.** The verifying onboarding desk runs the §7.3 check-list in order:

1. **Schema** — the credential's type and shape match the `KycAttestationCredential` definition (VC JSON Schema, ⚠ CRD status noted §2.5 — the pilot treats the schema as fixed by agreement).
2. **Proof** — the `eddsa-2022` signature validates against the key fetched from `did:web:issuer.cymbal.example` (§2.4 resolution via HTTPS — per-method, per §13.2's resolution caveat).
3. **Status** — the verifier fetches the status credential and reads bit 42 (§6.1); on fetch failure, the pilot's policy is *reject with retry*, not silent acceptance (§6.3).
4. **Issuer trust** — the issuer DID is on the bank's trust registry for this credential type (§3.2): the policy step, checked against Cymbal's own accept-list.
5. **Holder binding** — the client's presentation is a VP bound to the onboarding desk's random challenge and signed by the `did:key` named as the subject (§7.2) — a forwarded copy of the credential fails here.

**Then the human part:** the credential's outcome-level claims (screening cleared, EDD tier standard) enter the bank's decisioning, but the bank still performs its own residual EDD — look-through refreshed per the private-banking guide's conventions, sanctions screening against current lists, risk rating — because the credential proves what the *issuer* did, not what *this* bank is accountable for (§8.2). The credential compresses the document chase; it does not outsource the obligation.

### 12.5 Architecture Notes and Interoperability Caveats

**The pilot's architecture in five lines:** issuer DID + HSM signing service + status-list publisher on the issuing side; client wallet (keys + credentials) in the middle; verifier with schema store, suite allow-list, trust registry, and status fetcher on the accepting side. The caveats, from the verified landscape:

- **Singpass is the local elephant.** In Singapore, the dominant onboarding data path is MyInfo consent-based release (§9.1), not holder-presented VCs — so the pilot coexists with, rather than replaces, the MyInfo integration described in the SG banking siblings ([dbs_bank_guide.md](../banking/dbs_bank_guide.md), [trust_bank_guide.md](../banking/trust_bank_guide.md), [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md), et al.). A client holding a Cymbal attestation in a wallet is a different flow from a client whose form is pre-filled by MyInfo; the §8.3 diagram shows the former.
- **Format-profile risk.** The pilot's JSON-LD/Data Integrity stack is one lane of the §7.5 fragmentation; an EU-facing twin would likely verify SD-JWT/QEAA presentations under the EUDI profile (§11.2) — same check-list, different suites and trust anchors.
- **Status availability.** The whole model leans on the status endpoint (§6.3); the pilot plans for it as infrastructure with an SLA, not a convenience.
- **The honest scope:** this pilot proves *acceptance and issuance mechanics* for one credential type between known parties. Ecosystem-scale value — accepting a stranger's KYC attestation from an unfamiliar issuer — depends on the trust fabric of §3.2, which no single bank builds alone.

## 13. The Claims Audit

### 13.1 The Verified-Facts Table

**Every research claim in this guide, with its mark and source.** ✅ = verified this pass against the named primary/authoritative source; ⚠ = flagged (approximate, derived, single-sourced, or not re-verifiable); ❌ = refuted or not found. Sources in brackets refer to §14.3.

| # | Claim | Mark | Source |
|---|---|---|---|
| 1 | VC Data Model v1.0: W3C Recommendation 19 November 2019; JWT `vc`/`vp` conventions in §6.3.1 | ✅ | w3.org/TR/2019/REC-vc-data-model-20191119 |
| 2 | VC Data Model v1.1: W3C Recommendation 3 March 2022 | ✅ | w3.org/TR/vc-data-model-1.1 |
| 3 | VC Data Model v2.0: W3C Recommendation 15 May 2025; conformance regime; embedded vs enveloping proofs; JWT moved to vc-jose-cose; v2 context namespace | ✅ | w3.org/TR/vc-data-model-2.0 |
| 4 | Verifiable Presentations (v2.0 §4.13): aggregate VCs; SHOULD be short-lived and challenge-bound; holder binding not implied | ✅ | w3.org/TR/vc-data-model-2.0 |
| 5 | DID v1.0: W3C Recommendation 19 July 2022 | ✅ | inherited — web3_technologies_guide.md §8.1 |
| 6 | did:web / did:key / did:ethr / did:sov / did:jwk all "registered" in W3C DID-methods registry; registry non-endorsement disclaimer | ✅ | w3c.github.io/did-extensions/methods |
| 7 | DID Resolution not a Recommendation; CCG draft page unreachable this pass | ⚠ | §2.4; see §13.2 |
| 8 | Satellite suite of 15 May 2025: Bitstring Status List v1.0, Data Integrity 1.0, EdDSA + ECDSA cryptosuites, vc-jose-cose — all Recommendations | ✅ | w3.org TR pages (§2.5) |
| 9 | BBS cryptosuites (vc-di-bbs): CRD, latest 7 April 2026 — not yet a Recommendation | ⚠ | w3.org/TR/vc-di-bbs (§5.3) |
| 10 | VC JSON Schema: CRD, latest 4 February 2025 | ⚠ | w3.org/TR/vc-json-schema |
| 11 | SD-JWT: RFC 9901, IETF, November 2025; `_sd` digests + disclosures; optional `cnf` key binding | ✅ | RFC 9901 (§4.4) |
| 12 | SD-JWT VC (draft-ietf-oauth-sd-jwt-vc): active draft, not yet an RFC | ⚠ | IETF datatracker |
| 13 | eIDAS 2.0: Regulation (EU) 2024/1183 of 11 April 2024, OJ 30.4.2024, in force 20 May 2024; amends 910/2014; EUDI Wallet + QEAA | ✅ | EUR-Lex (§11.1) |
| 14 | Implementing acts 28 November 2024; obligations +24 months; Commission framing "end of 2026" | ✅ | EUR-Lex + EC digital-strategy page |
| 15 | Widely quoted 21 November 2026 is a derived date, not stated in the regulation | ⚠ | derived; §13.2 |
| 16 | Singpass: national digital identity, GovTech-built, app since 2018; MyInfo consent-based pre-fill | ✅ | GovTech (cached) (§9.1) |
| 17 | Singapore VC plans: GovTech announcement around Singapore FinTech Festival 2022; specifics unconfirmed | ⚠ | cached snippets (§9.2) |
| 18 | EUROPEUM-EDIC: Commission decision 21 May 2024; 9 member states; takes over EBSI governance | ✅ | EC decision (§10.2) |
| 19 | OpenWallet Foundation: Linux Foundation, launched February 2023 | ✅/⚠ | openwallet.foundation via cached snippets (§3.3) |
| 20 | Trust over IP Foundation: Linux Foundation, founded 2020; four-layer stack | ✅/⚠ | cached snippets (§3.3) |
| 21 | Open Badges OB 3.0 alignment with VC data model; Blockcerts MIT Media Lab origin | ⚠ | §10.1 |
| 22 | OID4VCI/OID4VP as de facto issuance/presentation protocols | ⚠ | OpenID Foundation, not re-verified this pass (§7.1) |

### 13.2 What Could Not Be Verified

**The honest ledger of what this pass could not confirm, despite the depth of the rest of the guide:**

- **DID Resolution status.** The CCG-hosted DID Resolution draft page was unreachable this pass; resolution is treated as per-method (did:web HTTPS dereferencing, did:key generation) rather than per a ratified resolution spec (§2.4).
- **BBS cryptosuite standardisation.** Data Integrity BBS Cryptosuites v1.0 is a Candidate Recommendation Draft (latest 7 April 2026), not yet a W3C Recommendation — the *math* (BBS+ selective disclosure and ZKPs) is uncontested, but the *status* is in motion (§5.3).
- **SD-JWT VC profile.** draft-ietf-oauth-sd-jwt-vc remains an active IETF draft, not an RFC (§4.4).
- **VC JSON Schema.** Still a Candidate Recommendation Draft (latest 4 February 2025); treated as stable-in-practice but unratified (§2.5).
- **Singapore VC specifics.** GovTech's verifiable-credential announcements around Singapore FinTech Festival 2022 could not be confirmed beyond cached snippets — which credentials, formats, or timelines were announced remains unverified (§9.2).
- **The derived EUDI date.** 21 November 2026 is derived from the 28 November 2024 implementing acts plus 24 months; the regulation itself states no such date, and the Commission's public framing is simply "the end of 2026" (§11.1).
- **Flagged ⚠ items carried through the guide:** the "trust triangle" and "JSON-LD 1.1 alignment" as community shorthand; OpenWallet Foundation and ToIP depth beyond their launches; Open Badges OB 3.0 conformance status; Blockcerts maintenance status; Europass EDC depth; OID4VCI/OID4VP protocol status; QEAA treatment under AML/KYC rules; EUDI-wallet gravitation toward SD-JWT; employment/health/travel scheme specifics; and the CCG "BBS+ 2020" predecessor page (not re-read this pass).

## 14. Glossary, Cross-References and the Closing Summary

### 14.1 The Glossary

| Term | Meaning |
|---|---|
| **Verifiable Credential (VC)** | A W3C-standard expression of claims, signed by an issuer, machine-verifiable (§2) |
| **Verifiable Presentation (VP)** | The holder's wrapper presenting one or more VCs to a verifier (§2.3) |
| **Issuer / Holder / Verifier** | The three trust-triangle roles: signer, controller, checker (§3.1) |
| **DID** | Decentralized Identifier — URI whose controller holds the keys (W3C REC 19 July 2022; inherited) |
| **DID method** | The scheme defining how a DID type is created/resolved (did:web, did:key, …) |
| **JSON-LD** | Linked-data serialization, native to the VC data model (§4.2) |
| **JWT-VC (v1.0)** | The v1.0 JWT serialization using `vc`/`vp` claims (§4.3) |
| **SD-JWT** | Selective Disclosure JWT — RFC 9901; hashed claims + released disclosures (§4.4) |
| **Data Integrity** | The W3C embedded-proof framework (REC 15 May 2025) with named cryptosuites (§5.1) |
| **Cryptosuite** | A named signature algorithm bound into Data Integrity (eddsa-2022, ECDSA, BBS+) |
| **BBS+** | Pairing-based signatures enabling selective disclosure and ZKPs (spec still CRD) (§5.3) |
| **Selective disclosure** | Presenting only a subset of a credential's claims (§5.3–§5.4) |
| **credentialStatus / status list** | The revocation/status mechanism — Bitstring Status List v1.0 (REC 15 May 2025) (§6) |
| **Trust anchor / trust registry** | The verifier's list of issuers it trusts for a claim type (§3.2) |
| **Holder binding** | The mechanism proving the presenter controls the credential (challenge + key) (§7.2) |
| **QEAA** | Qualified Electronic Attestation of Attributes under eIDAS 2.0 (§11) |
| **EUDI Wallet** | The EU Digital Identity Wallet mandated by Regulation (EU) 2024/1183 (§11) |
| **EBSI / Europeum** | EU blockchain infrastructure and its post-2024 governance body (§10.2) |

### 14.2 The Cross-Reference Map

**Sibling guides cited in this guide (relative links, same repo):**

- [web3_technologies_guide.md](web3_technologies_guide.md) — the Web3 umbrella; carries the verified DID record inherited in §2.4.
- [smart_contracts_guide.md](smart_contracts_guide.md) — the EIP/ERC process and on-chain mechanics; the ledger edge for §6.2 (on-ledger revocation; on-chain KYC tokens are a different beast).
- [cybersecurity_guide.md](cybersecurity_guide.md) — threats and key management; the security context for §7.4 wallets and issuer keys.
- [private_banking_guide.md](../banking/private_banking_guide.md) — KYC/EDD conventions; the compliance baseline for §8 and §12.4.
- [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) — the SG/MAS regime; the posture for §9 and §11.3 in Singapore.
- [fireblocks_guide.md](../banking/fireblocks_guide.md) — institutional wallet/custody mechanics; the wallet-security twin for §7.4 and §12.3.
- SG banking cluster: [dbs_bank_guide.md](../banking/dbs_bank_guide.md), [uob_software_systems_guide.md](../banking/uob_software_systems_guide.md), [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md), [maribank_guide.md](../banking/maribank_guide.md), [trust_bank_guide.md](../banking/trust_bank_guide.md), [singapore_fintech_payments_guide.md](../banking/singapore_fintech_payments_guide.md) — the Singpass/MyInfo onboarding context cited in §9.1, not re-derived.

### 14.3 Primary Sources Used This Pass

**Verified at their primary locations (this pass or inherited per repo convention):** the W3C Verifiable Credentials Data Model v1.0 (REC 19 Nov 2019), v1.1 (REC 3 Mar 2022) and v2.0 (REC 15 May 2025) — w3.org/TR; the satellite suite of 15 May 2025 (Bitstring Status List v1.0, Data Integrity 1.0, EdDSA/ECDSA cryptosuites, vc-jose-cose) and the CRD-track specs (vc-di-bbs, vc-json-schema) — w3.org/TR; the DID-methods registry — w3c.github.io/did-extensions/methods; RFC 9901 (SD-JWT) — rfc-editor.org; Regulation (EU) 2024/1183 — EUR-Lex (full text cached); the European Commission's digital-strategy page on the EUDI Wallet (cached); GovTech pages on Singpass/MyInfo (cached); the EUROPEUM-EDIC Commission decision; the W3C Verifiable Credentials Working Group pages. ⚠ Items whose sources were cached search snippets rather than full primary pages are flagged in §13.1–§13.2 (OpenWallet Foundation, ToIP, Singapore VC plans, Open Badges OB 3.0, Blockcerts).

### 14.4 The Closing Summary

**The guide in five lines:** the W3C data model is ratified and settled — v1.0 in 2019, v2.0 and its satellite suite on 15 May 2025 — and it now sits inside a working system of formats (JSON-LD, JWT conventions, SD-JWT as RFC 9901), signatures (Data Integrity's Ed25519/ECDSA workhorses, BBS+ still maturing), and status machinery (Bitstring Status List) that a bank can build against today. The use cases that pay are KYC portability, national digital identity, and the eIDAS 2.0 calendar, where the EUDI Wallet turns acceptance capability into a deadline. The ecosystem is still fragmented — format camps, proof suites, wallets, government profiles — so the strategy is acceptance-first, issuance-second, with the trust fabric of §3.2 doing the work no single bank can do alone. And throughout, one idea holds the stack together: a claim signed by an issuer, carried by a holder, checked by a verifier — the provable claim.
