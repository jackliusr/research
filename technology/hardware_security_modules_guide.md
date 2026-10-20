# HSM: The Refusal Is the Product - A Comprehensive Guide

**The Device Deep-Dive - What a Hardware Security Module Physically Is and Why It Refuses, the Four Device Classes From PCIe Card to Cloud Tenant, the FIPS 140-3 Ladder Stated Level by Level in the NIST Wording, the 21 September 2026 Transition and the Three Separate Approval Regimes, the Device Interfaces as Interfaces, the Key Ceremony as a Physical Procedure, the Payment-HSM Mechanics From ISO 9564 PIN Blocks to DUKPT, the Cloud Tenancy Model, the Operations the Refusal Makes Hard, the Failure Modes, a Cymbal Bank Worked Example, the Anti-Patterns, the Claims Audit, and What Could Not Be Verified**

> **Author:** Jack Liu Shurui, Solution Architect
> **Series:** Technology / Cryptographic Infrastructure - the device at the bottom of the key hierarchy, and the only component in the estate that is designed to say no.
> **Audience:** Security architects and platform engineers who have to rack, cluster, patch and recover these devices; payments engineers who meet the HSM only as the thing that answers a PIN translate in under ten milliseconds; and - the audience this guide is really written for - the architect in a regulated institution who has to turn "we have an HSM" into a procurement decision, a ceremony script and an operational runbook that survive an audit.
> **Context:** This guide **OWNS** the device and its operation: what an HSM physically is and why it refuses, the device classes, the validation regime stated level by level plus the procurement question it settles, the device interfaces as interfaces, the key ceremony as a **physical procedure** at the device, the payment-HSM mechanics, the cloud tenancy model, operations (availability, DR, backup, firmware, monitoring) and the failure modes. It deliberately does **NOT** re-derive, and cites by section rather than repeating: the KMIP protocol and the protocol comparison (the KMIP guide, section 7); the key-lifecycle discipline - cryptoperiods, lifecycle states, key hierarchy and envelope model, separation of duties (section 8, and 8.3-8.4 specifically); the key ceremony **as a governance and evidence event** (section 8.5); the FIPS module-versus-system argument **in principle** (section 9.4); the PIN-standards carefulness about requirement numbers (section 9.3); and the disconnected-key-estates finding (section 9.7). Where this guide touches any of those, it names the section and moves on. If a paragraph here would sit unchanged in the KMIP guide, it does not belong here.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** October 2026
> **Companion guides (sibling, same folder - plain filenames):** [kmip_key_management_guide.md](kmip_key_management_guide.md) - **THE companion guide**, owning the protocol, the lifecycle discipline, the ceremony as evidence, the module-versus-system argument in principle and the PIN-standards care; this guide defers to it by section throughout. [cloud_providers_guide.md](cloud_providers_guide.md) - the cloud platform comparison; section 8 here gives only the HSM tenancy model and does not re-compare providers. [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) - the secrets-manager comparison and the PKCS#11 *seal* path, which this guide mentions once and does not re-derive.
> **Companion guides (banking/, prefix `../banking/`):** [swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) - **owns** the SWIFT HSM and CSP context; [nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) - **owns** the NETS payment-HSM estate; [payments_hub_guide.md](../banking/payments_hub_guide.md) and [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md) - the surrounding payment and market-infrastructure context.
> **Primary Sources for this pass:** NIST CMVP, *FIPS 140-3 Standards* page (`csrc.nist.gov/projects/cryptographic-module-validation-program/fips-140-3-standards`, page footer "Updated August 21, 2026", read this pass); NIST, *FIPS 140-3* publication page (`csrc.nist.gov/pubs/fips/140-3/final`, published 22 March 2019); NIST CMVP, *FIPS 140-3 Transition Effort* page (`csrc.nist.gov/projects/fips-140-3-transition-effort`, page footer "Updated April 13, 2026", transition table "Updated 06-02-2021"); NIST, *FIPS 140-2* full text (`nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf`, issued 25 May 2001, change notices to 3 December 2002) - used only for the free, public statement of the four-level ladder, since FIPS 140-3 delegates the normative requirement text to the paywalled ISO/IEC 19790:2012; NIST SP 800-140 (March 2020, Kim Schaffer) and its parts 800-140A to 800-140F, cited by name; PCI SSC, "PCI SSC Publishes PCI PTS HSM v5.0", blog post by Alicia Malone, 18 May 2026, and the PTS HSM topic page carrying the same date; PCI SSC, "Just Published: PCI Key Management and Operations (KMO) Standard v1.0", blog post, 14 September 2026, and the KMO standards page; ISO 9564-1:2017, fourth edition 2017-11, ISO/TC 68/SC 2 - front matter and table of contents read from the ISO preview, **structure only, mechanics flagged as restated**; AWS Payment Cryptography User Guide, *Industry terminology* (`docs.aws.amazon.com/payment-cryptography/latest/userguide/terminology.html`, read this pass); AWS CloudHSM User Guide, *Introduction* (`docs.aws.amazon.com/cloudhsm/latest/userguide/introduction.html`); Microsoft Learn, *Azure Key Vault Managed HSM overview* (`learn.microsoft.com/en-us/azure/key-vault/managed-hsm/overview`, page "Last updated on 10/14/2026"); Google Cloud, *Cloud HSM* (`cloud.google.com/kms/docs/hsm`, page "Last updated 2026-10-13 UTC"); SoftHSMv2 repository (`github.com/softhsm/SoftHSMv2`, README read this pass); Regulation (EU) 2022/2554 (DORA) via EUR-Lex CELEX 32022R2554 and the ESMA DORA page; Thales payShield 10K product page (`cpl.thalesgroup.com`) - read for **product shape only**, its marketing claims are handled in section 14 and are not repeated as fact.
> **Scope note on verification:** Every ✅ in this guide is tied to a named source read on the date shown. Every ⚠ marks something I could not reach or could only read as a secondary restatement - most importantly the **mechanics** of the ISO 9564 PIN block formats, which sit behind a paywall, and the **clause numbers** of the PCI KMO standard, whose PDF would not fetch this pass. Where a claim is a vendor's statement about its own product, it is labelled as such and is not treated as an independent finding. An empty search is not evidence of absence, and no negative claim in this guide rests on one.
> **Integrity convention:** ✅ = verified this pass against the cited source; ⚠ = flagged, partial, or dependent on a non-primary source; ❌ = rejected (asserted somewhere, found false or unsupported). Used in every table that makes a factual claim.
> **Illustrative figures:** Every number attributed to **Cymbal Bank** (the only institution in this guide, and a fictional one) is **explicitly fictional and illustrative** - a worked shape for a design review, not a benchmark, not a survey result, and not a claim about any real institution. No market share, ranking, customer count or price figure appears anywhere in this guide.

**How this guide is organised.** Section 1 defines the device in one paragraph, hands you the decoder table, states the thesis the whole guide defends, and declares the boundary against the KMIP guide. Section 2 is the argument that carries the rest: the five refusals that make an HSM an HSM, each argued as a security property and each with its cost named. Section 3 is the device classes and the requirement each one actually answers. Section 4 is the validation regime - the four FIPS 140-3 levels in the NIST wording, what a certificate does and does not cover, the 21 September 2026 transition, Common Criteria and PCI PTS HSM as **separate** approvals, and the procurement question this settles. Section 5 is the interfaces as interfaces: slot, session, object, provider, key store provider, host command. Section 6 is the ceremony as a physical procedure. Section 7 is the payment HSM, the section that will be read hardest. Section 8 is the cloud tenancy model and where the refusal sits when somebody else operates the device. Section 9 is operations - availability, DR, backup, firmware, monitoring. Section 10 is the regulatory angle: PCI DSS, PCI PIN, PCI PTS HSM, PCI KMO, DORA. Section 11 is the failure modes. Section 12 is the Cymbal Bank worked example. Section 13 is the anti-patterns. Section 14 separates what is certain from what is vendor positioning. Section 15 is the claims audit and the honest residue. Section 16 is the glossary, the cross-references and the closing.
**Source discipline, applied throughout:** this is a field where **the certificate is public, the security policy is public, the mechanical behaviour is not, and the marketing is very loud**. Every claim is tagged as one of three things - a **PRIMARY DOCUMENT** (a NIST publication, a PCI SSC publication, a vendor's own product documentation for its own product), a **VENDOR CLAIM ABOUT ITS OWN PRODUCT** (marketing until independently corroborated), or a **THIRD-PARTY RESTATEMENT** (an explainer, a standards summary, a blog - useful for leads, not for facts). The distinction matters most in sections 7, 14 and 15.

### Table of Contents

1. [The Overview, the Decoder and the Thesis](#1-the-overview-the-decoder-and-the-thesis) - 1.1 the one-paragraph definition | 1.2 the decoder table | 1.3 the thesis | 1.4 the boundary, by name and section | 1.5 what this guide refuses to do
2. [What an HSM Is, and the Refusal That Defines It](#2-what-an-hsm-is-and-the-refusal-that-defines-it) - 2.1 tamper-responsive design | 2.2 key non-extractability | 2.3 the rate limit | 2.4 role separation at the device | 2.5 the device audit log | 2.6 the cost of each refusal | 2.7 where the thesis strains
3. [The Device Classes](#3-the-device-classes) - 3.1 the network-attached general-purpose appliance | 3.2 the PCIe card and the embedded module | 3.3 the payment HSM | 3.4 the cloud HSM | 3.5 the software emulator and SoftHSM v2 | 3.6 matching the class to the requirement
4. [The Validation Regime](#4-the-validation-regime) - 4.1 what FIPS 140-3 is, with dates | 4.2 the four levels in NIST's wording | 4.3 what a certificate covers and does not | 4.4 the 140-2 to 140-3 transition and 21 September 2026 | 4.5 Common Criteria, named | 4.6 PCI PTS HSM as a separate device approval | 4.7 the procurement question | 4.8 a module is not a deployment
5. [The Device Interfaces](#5-the-device-interfaces) - 5.1 what an interface to a device actually exposes | 5.2 PKCS#11 | 5.3 JCE/JCA | 5.4 CNG and the KSP | 5.5 vendor-native host commands | 5.6 the integration properties that decide outages | 5.7 the boundary against the protocol comparison
6. [The Key Ceremony](#6-the-key-ceremony) - 6.1 why a ceremony exists at all | 6.2 what physically happens | 6.3 who must be present and where they stand | 6.4 split knowledge and dual control at the device | 6.5 key components and key-loading devices | 6.6 the ceremony record | 6.7 why it is the most under-documented process in the discipline
7. [The Payment HSM](#7-the-payment-hsm) - 7.1 what makes a payment HSM a different machine | 7.2 PIN blocks and ISO 9564-1:2017 | 7.3 translation, and why the device is in the path | 7.4 DUKPT and the per-transaction key | 7.5 MAC generation and verification | 7.6 the key hierarchy that the scheme dictates | 7.7 the operational model: latency, HA and the shared LMK | 7.8 PCI PTS HSM v5.0 and what changed
8. [The Cloud HSM Model](#8-the-cloud-hsm-model) - 8.1 what is actually rented | 8.2 where the boundary of the refusal sits | 8.3 tenancy and separation | 8.4 the three providers, as documented, with dates | 8.5 what changes about the control argument | 8.6 PCI KMO and the as-a-service validation
9. [The Operations](#9-the-operations) - 9.1 availability and clustering | 9.2 the DR problem the refusal creates | 9.3 backup and restore of a non-exportable key | 9.4 firmware and patch | 9.5 monitoring a device that will not tell you what it is doing | 9.6 the runbook items nobody writes
10. [The Banking and Regulated Angle](#10-the-banking-and-regulated-angle) - 10.1 PCI DSS, by substance | 10.2 PCI PIN | 10.3 PCI PTS HSM v5.0 | 10.4 PCI KMO v1.0 | 10.5 DORA and the third-party device | 10.6 a validated module is not a compliant deployment
11. [The Failure Modes](#11-the-failure-modes) - six, each with symptom, cause and guardrail
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example) - 12.1 the estate | 12.2 the failure the thesis predicts | 12.3 the decision the discipline changed | 12.4 what it cost, illustratively
13. [The Anti-Patterns](#13-the-anti-patterns) - nine, each with symptom, cause and guardrail
14. [The State of Practice](#14-the-state-of-practice) - 14.1 what is certain | 14.2 what is vendor positioning | 14.3 the shape of the marketing | 14.4 what a buyer can actually rely on
15. [The Claims Audit](#15-the-claims-audit) - the table, then what could not be verified
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

## 1. The Overview, the Decoder and the Thesis

### 1.1 The One-Paragraph Definition

**An HSM is a physically protected, tamper-responsive computing device whose entire purpose is to generate, hold and use cryptographic key material inside a boundary that the machine which calls it cannot read across.** It does not export the key; it exports *the result of using the key*. It exposes a narrow, authenticated interface - encrypt this, translate that PIN block, sign this digest, verify this MAC, wrap that key for backup - and it enforces, in hardware rather than in configuration, that the material behind that interface is not readable, not copyable, and destroyed if the enclosure is opened. It is validated as a **module** at a defined level, and it is operated as an **appliance**: it has an owner, a role model, an audit log, a firmware version that matters, a backup regime that requires a quorum, and a physical location that somebody has to be allowed into. Everything else in this guide is a consequence of one of those clauses. ✅ (NIST FIPS 140-3 for the module framing and the requirement areas; the operational clauses are argued in sections 2 and 9)

### 1.2 The Decoder

The words in this field are used interchangeably by people who should know better, and the confusion is expensive because the terms sit at different layers. Seven terms, defined once, used consistently afterwards.

| Term | What it actually denotes | What it is often confused with | Integrity |
|---|---|---|---|
| **The HSM** | The physical device: a tamper-responsive enclosure containing a cryptographic processor, key storage, sensors, a real-time clock, an audit log and one or more host interfaces. It refuses. | "The crypto layer", "the key server", "the vault" - all of which are software roles that may be backed by an HSM, or may not | ✅ |
| **The module** | The **validated unit** in FIPS 140 terms: a named hardware-plus-firmware combination, at a version, evaluated against a security policy. One physical appliance may contain one module; one module may span a card and its firmware only. | The appliance, the product, the deployment - none of which the certificate covers | ✅ |
| **The token** | The PKCS#11 abstraction for the thing that holds objects: what an application "opens" and authenticates to. On a real device it maps to a partition; in a software implementation it maps to a directory. | The physical device - one device may present many tokens | ✅ |
| **The slot** | The PKCS#11 addressing primitive: a numbered (or labelled) location at which a token may be present. A slot with no token is a legal, and common, state. | A physical slot in a chassis - the collision of vocabulary is pure coincidence and causes real support tickets | ✅ |
| **The partition** | The device-side tenancy unit: an isolated key store with its own security officer, its own access control and its own objects, carved out of a shared physical device. This is what makes one appliance serve several applications, or several customers. | A VLAN, a namespace, a database schema - all logical; a partition is enforced by the device's own firmware | ✅ |
| **The ceremony** | The scripted, witnessed, dual-controlled **physical procedure** by which key material of the highest sensitivity is generated, loaded, backed up or destroyed at the device. | A change ticket, a deployment, a key rotation job - none of which require two people in a room | ✅ |
| **The HSM-as-a-service** | A tenancy model in which a provider owns, operates, patches and physically secures the device, and the customer holds an administrative identity inside it. The refusal still happens; the question is who is allowed to overrule it. | A managed key service with a software backend; a KMS with an HSM behind it that the customer never addresses directly | ✅ |

Two of those rows deserve a sentence more. **The module is not the appliance**: a vendor may ship a chassis whose validated module is the cryptographic card inside it, which means the chassis management controller, the network stack and the surrounding Linux are outside the certificate - a distinction that section 4.3 turns into a procurement question. And **the partition is the unit of blame**: when two applications share a device and one of them exhausts the operation rate limit, the other one experiences an outage caused by a partition it does not own, which is failure mode five in section 11. ⚠ (partitioning is universal in the class; the specific enforcement mechanism is vendor documentation and was not read for every vendor this pass)

### 1.3 The Thesis

**An HSM is the only component in the estate whose job is to refuse.**

Everything else in the architecture is measured by what it will do for you: the database returns rows, the load balancer forwards connections, the message broker delivers. The HSM is measured by what it will **refuse**. The key does not come out. The operation is rate-limited, so an attacker with a session cannot simply guess. The device zeroizes its key store when the enclosure is opened, so a stolen appliance is a brick rather than a breach. The security officer cannot use the keys they created, and the application cannot create the keys it uses. A backup cannot be taken without a quorum of custodians physically present. Those are not limitations that better engineering will eventually remove; they are the product.

The thesis is not a slogan, because it predicts things, and a claim that predicts nothing is decoration. It predicts that **every operational complaint about HSMs is a complaint that the refusal is working**: "we cannot export the key to the new platform" (correct - that is the property), "the DR site cannot be brought up without flying three people there" (correct - that is the property), "the vendor's cloud service will not let us hold the backup" (correct - that is the negotiation), "the patch requires a re-validation because it changes the validated version" (correct - that is the property applied to firmware). It also predicts where the refusal gets **bought out rather than removed**: the cloud tenancy model in section 8 is precisely a negotiation about who is permitted to hold the ability to refuse, and a customer who has not read the tenancy documentation has not had that negotiation.

And it predicts the failure mode that recurs most: **organisations route around the refusal instead of paying for it**. The key gets copied to a software store "for the DR test". The emulator goes into production because the real device was three months out on delivery. The ceremony happens once, undocumented, and the only two people who knew the split are both gone. Section 11 is that list, and section 13 is the same list stated as anti-patterns, because the difference between a failure mode and an anti-pattern is only whether it has happened to you yet.

Where the evidence complicates the thesis, this guide says so. Section 2.7 is the honest counter-argument: a device that refuses everything is also a device that cannot be recovered, and the refusal is only a security property when the surrounding process is funded well enough to absorb its cost. Section 7.7 shows a case where the refusal is deliberately weakened for availability - an HA group sharing a common master key - and argues that the weakening is correct and must be documented rather than denied.

### 1.4 The Boundary, by Name and Section

This guide is the sibling of [kmip_key_management_guide.md](kmip_key_management_guide.md), and the split is deliberate because the two subjects fail in different ways. **The KMIP guide owns the protocol and the discipline; this guide owns the device and its operation.** Concretely, and by section:

| Subject | Owner | How this guide treats it | Integrity |
|---|---|---|---|
| The KMIP protocol, its operations, attributes, profiles and version line | The KMIP guide, section 7 (and its sections 3 to 5) | Named once in section 5.7 as the alternative interface; **no protocol comparison table is reproduced here** | ✅ |
| The key-lifecycle discipline: cryptoperiod, lifecycle states, key hierarchy and envelope model, separation of duties | The KMIP guide, section 8, and 8.1-8.4 specifically | Section 2.4 covers role separation **at the device** (who may do what on this appliance); the discipline itself is not re-derived | ✅ |
| The key ceremony **as a governance and evidence event** | The KMIP guide, section 8.5 - including its evidence table and its test that a ceremony producing no durable, independent, signed record did not happen | Section 6 takes the **mechanics**: what is done, in what order, by whom, standing where, with which components. The evidence argument is cited, not restated | ✅ |
| The FIPS 140-3 module-versus-system argument **in principle** | The KMIP guide, section 9.4 | Section 4 states the four levels precisely with the NIST wording, adds the transition dates and the three separate approval regimes, and answers the procurement question. Section 4.8 cites 9.4 rather than re-arguing it | ✅ |
| The PIN-standards carefulness: name the standard, describe the control, do not quote an unverified requirement number | The KMIP guide, section 9.3 | Section 7 cites **ISO** clause numbers (which I read in the preview) and section 10 names PCI standards by version without quoting requirement numbers at all | ✅ |
| The disconnected-key-estates finding | The KMIP guide, section 9.7 | Referenced in section 11 as the reason a backup quorum cannot be reassembled; not re-argued | ✅ |
| **The device: physics, classes, validation detail, interfaces, ceremony mechanics, payment mechanics, cloud tenancy, operations, failure modes** | **This guide** | - | ✅ |

The practical test I applied while writing: if a paragraph would sit unchanged in the KMIP guide, it was cut. That is why there is no lifecycle-state table here, no KMIP-versus-PKCS#11 comparison, and no re-telling of the ceremony's evidence argument - only the parts that happen **at the device**, in a room, with people in it.

### 1.5 What This Guide Refuses to Do

Three refusals of its own, in the spirit of the subject.

- **It will not rank devices or vendors.** Thales, Utimaco, Entrust, AWS CloudHSM, Azure Managed HSM, Google Cloud HSM and SoftHSM are named because they are the objects of specific documented facts. None is called better, none is scored, and section 14 explains why the available evidence could not support a ranking even if I wanted to publish one.
- **It will not quote a requirement number it did not read.** The ISO clause numbers in section 7 come from the standard's own table of contents read in preview. The PCI standards in section 10 are named by version with their control areas described in substance, because the requirement-level PDFs did not all fetch this pass - which section 15 records as a limitation rather than papering over.
- **It will not treat a vendor's description of its own internals as a finding.** "Tamper-responsive to level 4", "zeroization on intrusion", "secure room" - where these come from a product page rather than a validation document, they are labelled as vendor claims, and section 15 has a row for exactly this class.

## 2. What an HSM Is, and the Refusal That Defines It

### 2.1 Tamper-Responsive Design

The distinction that matters is between **tamper-evident** and **tamper-responsive**, and it maps onto the FIPS levels in section 4.2 without being identical to them.

- **Tamper-evident** means you can tell afterwards that somebody opened it: seals, coatings, pick-resistant locks on covers. This is the Level 2 posture, and its security value is entirely *after the fact* - it converts an undetected compromise into a detected one, which is useful for investigation and useless for prevention.
- **Tamper-responsive** means the device reacts: sensors for intrusion, temperature, voltage and sometimes light or pressure, wired to a circuit whose only job is to **zeroize** the key store when a threshold is crossed. This is the Level 3 posture and the Level 4 posture at greater rigour, and its security value is *preventive in the only way that matters* - the attacker who opens the box gets a device full of zeroes and no key.

The design consequence is worth stating because it explains a hundred operational annoyances. A tamper-responsive device must have a **battery**, because the sensor circuit has to work when the power is off; the battery has a service life, and replacing it is a controlled procedure that on some devices must be done with the power on and on others cannot be done at all without destroying the module. The device must have a **tamper log** that survives the zeroization, because knowing that somebody tried is nearly as valuable as knowing they failed. And the enclosure must be **opened only under a defined procedure**, which is why the physical access control around a payment HSM is part of the security argument and not a facilities afterthought. ✅ (FIPS 140-2 sections 1.2-1.4 for the level wording; the battery and service-life consequences are general properties of the class and are vendor-documented, ⚠ for any specific model)

**Why this is a security property and not a limitation:** the alternative is a device that an attacker with five minutes and a screwdriver can turn into a key-extraction exercise. Every hour of physical-access paperwork you endure is an hour the threat model has already accounted for.

**What it costs:** a device you cannot open is a device you cannot repair in place. Field service is a swap, not a fix. Spares must be stocked, and a spare is not a spare until it has been through its own initialisation - which includes its own ceremony, which is section 6 again.

### 2.2 Key Non-Extractability

**The key is generated inside the boundary and never crosses it in plaintext.** That single sentence generates most of this guide's later difficulty, so it is worth being precise about what "never" excludes:

- **It excludes wrapped export.** A key can leave encrypted under another key, in a form that is only usable inside a device that holds the wrapping key. This is how backup works (section 9.3), how key exchange between devices works, and how a migration between estates is *supposed* to work. The wrapping is itself done inside the module, and the export is usually gated by a role and often by a quorum.
- **It excludes the split-key case.** Where a master key is loaded as components under dual control, the components exist outside the device on physical media under separate custody - but no single component is the key, and the reconstruction happens inside the device. This is section 6.
- **It does not exclude side channels.** Non-extractability is a statement about the interface, not about physics. A device can leak through timing, power or electromagnetic emission, which is why FIPS 140-3 has a requirement area for non-invasive security and another for mitigation of other attacks ✅ (FIPS 140-3 abstract, requirement areas) - and why the certificate's level is a statement about evaluated attack classes, not a guarantee of indistinguishability from an ideal black box.

**Why this is a security property:** it collapses the threat model. If the key cannot be read, then the compromise you have to worry about is not "someone copied the key and now operates undetectably forever"; it is "someone used the device while they had access", which is bounded in time, logged by the device, and detectable by rate anomaly. Non-extractability converts an unbounded secret compromise into a bounded operational one.

**What it costs:** you cannot move the key to a new platform, so the platform decision is sticky for the life of the key. You cannot test a restore without a device. You cannot hand a copy to a partner, an acquirer, or an acquiring bank without a key-exchange protocol and a device on both ends. And you cannot answer "what is the key?" to anybody, including a regulator, including a court order, including yourself at 3 a.m. during an incident - you can only answer "what did the key do?", which is what the audit log is for.

### 2.3 The Rate Limit

Every HSM class imposes a ceiling on operations per unit time, and it is almost always experienced as a performance problem rather than a control. It is a control.

Consider what the rate limit buys. A PIN-verification HSM that will answer an unlimited number of guesses per second is a device that turns a stolen PIN block and a captured card into a search problem; the rate limit is the difference between a 10,000-guess space being searched in seconds and being searched in months. An HSM fronting a signing service that answers unlimited requests is a device that will happily sign an attacker's way through a token space. The ceiling is set by the device's cryptographic throughput, but its *purpose* is to bound the number of adversarial attempts per unit time that the key can be made to participate in.

The second, less discussed function: **the rate limit bounds the blast radius of a stolen application credential.** If an application's PKCS#11 PIN is compromised, the attacker inherits the application's operation budget - not the device's total capacity, because the partition's limits are enforced per partition. On a well-configured device the partition ceiling is an order of magnitude below the device ceiling, which is exactly the design intent: one compromised application degrades its neighbours rather than taking the device. ⚠ (per-partition rate limiting is documented behaviour on the appliances in this class; the specific numbers are vendor documentation and were not read for every vendor this pass)

**Why this is a security property:** it is the only control in the estate that makes an *online* guessing attack expensive in wall-clock time regardless of the attacker's hardware. Everything else - key length, algorithm choice - makes each guess expensive in compute; the rate limit makes the *sequence* expensive in time, and time is the one resource an attacker cannot buy more of.

**What it costs:** capacity planning becomes a cryptographic exercise, and it is done badly almost everywhere. The device is sized for peak transaction rate at launch. Three years later the same device serves four more applications, the peak has doubled, and the first symptom is not an error - it is **latency**, because the device queues. Section 9.5 is how you see it coming, and section 11's fourth failure mode is what happens when you do not.

### 2.4 Role Separation at the Device

A real device has a role model enforced in firmware, and it is stricter than most application role models because the roles are designed to be **mutually suspicious**:

- The **security officer** creates and deletes partitions, sets policy, and initialises the device. The security officer cannot use the keys.
- The **user** (or crypto officer, depending on vendor vocabulary) uses keys within a partition and cannot create the partition or change its policy.
- The **auditor** reads the log and changes nothing.
- On payment devices there is additionally a **master key** custodian role bound to the ceremony, and the component holders are people, not accounts.

The property that matters is the **separation**, not the naming: **whoever can create a key must not be the identity that can use it, and whoever can use it must not be able to export it.** When all three are the same identity - which is the default in every software emulator and in far too many real deployments - the role model is theatre, and the audit log records a single actor doing everything, which tells you nothing about whether the process was followed.

**Why this is a security property:** it means that compromising the application does not give you the ability to mint a new key with the application's own privileges, and compromising the administrator does not give you the ability to use the keys you administer. The two compromises are different attacks, and the device forces an attacker to mount both.

**What it costs:** onboarding is a two-person, two-ticket process forever. Every new application means a new partition, a new security-officer action, and a handover of credentials through a channel that is itself controlled. In practice this is the control most often collapsed "for agility", and section 13's first anti-pattern is exactly that collapse.

### 2.5 The Device Audit Log

The device keeps its own log, inside the boundary, of what was asked of it: authentication successes and failures, key generation and deletion, policy changes, ceremony events, tamper events, and - on some devices - the operations themselves without the key material. It is **append-only in intent**, it is often **hash-chained or signed** so that a deleted entry is detectable, and it is **exported** to a collector because the device's own storage is small and circular.

Three properties make it different from an application log:

1. **It is written by the component that did the work**, not by the component that asked. An application can lie about what it did; the device's record is the device's own testimony.
2. **It survives the application.** Rebuilding the application from scratch does not erase the device's history of what that application's partition did.
3. **It is evidence of refusal as well as action.** Failed authentications, rejected rate limits, refused exports and tamper events are in the same log as the successes, which is precisely the half that a normal application log omits.

**Why this is a security property:** it is the only record in the estate that is produced by a component with no incentive to flatter the application, and at the highest FIPS levels the device is required to detect and respond to physical attack - so the log's integrity claim is stronger than "we wrote it down".

**What it costs:** log volume, and a nasty operational detail: the device's clock. A device whose real-time clock drifts, or which loses time on battery replacement, produces evidence with unreliable timestamps, and an auditor who finds a two-minute gap between the device log and the collector's receipt time will ask about it. Section 9.5 is the monitoring consequence; section 6.6 is the ceremony-record consequence.

### 2.6 The Cost of Each Refusal

| The refusal | The security property it buys | What it costs, concretely | Integrity |
|---|---|---|---|
| **The key does not come out** | An unbounded secret compromise becomes a bounded operational one | Platform stickiness for the key's life; no portable copy; no "just restore it on the new kit" | ✅ |
| **The device zeroizes when opened** | A stolen appliance is a brick, not a breach | Field service is swap-only; battery service life becomes a scheduled risk; the enclosure procedure is part of the security argument | ✅ |
| **Operations are rate-limited** | Online guessing becomes expensive in wall-clock time; a stolen credential inherits a budget, not the device | Capacity planning is a cryptographic exercise; the first symptom of overload is latency, not error | ✅ |
| **Roles are separated** | Compromising the app does not let you mint keys; compromising the admin does not let you use them | Every onboarding is two people and two tickets, forever | ✅ |
| **There is an internal, tamper-evident log** | The only testimony produced by a component with no incentive to flatter the caller | Log volume; clock integrity; an export path that must itself be secured | ✅ |

**The pattern in that table is the guide's argument in miniature:** every row is a control that a reasonable operations team would describe as an obstacle, and every row is a control that a reasonable attacker would describe as the reason the attack is not worth doing.

### 2.7 Where the Thesis Strains

Three honest complications, because a thesis that is never tested is a slogan.

**First, the refusal is only a security property if the process can absorb its cost.** A bank that installs a Level 3 payment HSM and then staffs one custodian, no backup custodian, no documented component custody and no rehearsal has bought the refusal and cannot operate it. The device will refuse correctly, and the business will route around it - usually by writing the components down somewhere convenient. The control is the device **plus** the funded process, and the process is the expensive half.

**Second, availability and refusal are in genuine tension, and the tension is resolved by design, not by principle.** A single device that refuses to export its key is a single point of failure for every transaction that needs it. The industry answer is an HA group in which every member shares the same master key material - which means the master key exists in more than one enclosure, which is a deliberate weakening. Section 7.7 argues that this is correct, and that the honest way to hold it is to say so in the design document rather than to pretend the refusal is absolute.

**Third, the cloud model relocates the refusal rather than removing it.** When a provider operates the device, the customer still cannot export the key - but the provider's staff can perform physical and administrative actions the customer cannot observe in detail. Whether that is a weakening depends entirely on what the tenancy documentation says and what the attestation covers, which is section 8, and the honest answer for most customers is "less weakening than you fear, more than the marketing implies".

## 3. The Device Classes

Four classes, and the useful discipline is to ask **which requirement each one answers** rather than which one is better. They are not interchangeable, and the most common procurement error in this field is buying the class that is convenient rather than the class the requirement names.

### 3.1 The Network-Attached General-Purpose Appliance

A 1U or 2U chassis with one or more cryptographic cards inside, redundant power, redundant network ports, and a management interface. It serves many hosts over the network, usually through PKCS#11 via a vendor client library that proxies to the device, or through the vendor's own protocol, or both. It holds TLS keys, signing keys, database encryption keys, code-signing keys and general application keys; it does not do PIN processing.

**The requirement it answers:** many applications, one key estate, one audit trail, one physical security boundary, and a key store that no single host administrator can read. It is the class that makes "the key lives in hardware" true for a general-purpose estate without putting a card in every server.

**The operational shape:** network latency on every operation (milliseconds, not microseconds), a client library on every host that must be version-matched to the device firmware, an HA pair or group for availability, and a backup regime that involves a quorum. Sections 9.2 and 9.3 are almost entirely about this class.

**The failure that is specific to it:** the client library becomes a single point of integration failure. A host that patches its OS and picks up a new library version can lose the ability to talk to a device whose firmware has not moved, and the symptom is an application that cannot start rather than a device that is down. Section 5.6 is the integration consequence.

### 3.2 The PCIe Card and the Embedded Module

A card in the server's own chassis, or a module soldered to a board. It performs the same cryptographic functions as the appliance, with two structural differences: **the latency is a local bus transaction rather than a network round trip**, and **the physical security boundary ends at the host's chassis lid**.

**The requirement it answers:** throughput and latency where the network hop is unacceptable, and a key boundary in environments where a separate appliance is not deployable - an edge location, a branch, a self-contained appliance product that ships with an HSM inside it.

**The honest limitation, and it is the one that gets glossed:** the card's tamper response protects the card, but the host's administrator has physical access to the host. A compromised host administrator can attack the card's interface, its driver, and its firmware update path. This is not a reason to avoid the class - it is a reason to be precise about what the physical-security claim covers, and to notice that the certification level of a card is a claim about the card.

**Where it appears in practice:** as the cryptographic engine inside a network appliance (so the appliance class in 3.1 is often a card plus a chassis plus a network stack), and as an in-server accelerator for high-volume signing or TLS termination.

### 3.3 The Payment HSM

A different machine with a different instruction set, because its job is not "hold a key and sign" but "participate in a card transaction inside a latency budget, using key material organised the way the card schemes require". PIN blocks, PIN translation between zones, DUKPT-derived per-transaction keys, MAC generation and verification over financial messages, CVV and PIN offset generation, EMV personalisation support.

**The requirement it answers:** the card networks' and the PCI standards' requirement that PIN and key material for card processing be handled inside an approved device, in a defined hierarchy, with dual control at every manual key operation. The class is defined less by its hardware than by its **approval** (PCI PTS HSM) and by its **key hierarchy** (section 7.6).

**Why it gets its own section:** section 7 exists because the payment HSM is where the refusal is hardest to operate - the device is in the authorisation path, an outage stops card payments, and the key hierarchy is dictated by parties outside the bank. It is also the section most likely to be read by someone who has to make a decision this quarter, so it is written to be actionable rather than architectural.

### 3.4 The Cloud HSM

The same functions, rented. The provider owns the chassis, the room, the power, the firmware update and the physical access; the customer holds an administrative identity inside a tenancy and speaks a standard interface to it. Section 8 is the full treatment and answers the only question that matters: **where does the refusal sit when the operator is not you?**

**The requirement it answers:** a key boundary in hardware for workloads that already live in a cloud account, without a data-centre build; and, increasingly, a regulatory requirement that the key material be in a validated module even when everything else is managed.

### 3.5 The Software Emulator, and SoftHSM v2

SoftHSM v2 is a software implementation of PKCS#11, used for development, testing, CI and protocol conformance work. It is well made, it is honest about what it is, and it is the single best illustration in this guide of a device that **does not refuse**.

The sentence that makes the point, from its own documentation: **"All of the tokens and their objects are stored in the location given by softhsm2.conf. Backup can thus be done as a regular file copy."** ✅ (SoftHSMv2 README, read this pass)

Read that again as a security statement. The entire backup and disaster-recovery problem that section 9.2 spends pages on **does not exist** for this implementation, because the key material is a file. There is no quorum, no ceremony, no wrapping key, no component custody, no device to restore into. Every operational cost this guide attributes to the refusal is absent, and so is every security property: the key is readable by anyone who can read the file, the object store has no tamper response, the rate limit is the host's CPU, the roles are whatever the file permissions say, and "restore to a new platform" is `cp`.

**Where it is legitimate:** unit and integration tests, CI pipelines that need a PKCS#11 surface, protocol conformance work, and training. These are real uses and the tool is good at them.

**Where it is a finding:** in production, in a regulated workload, or anywhere the phrase "we will migrate to a real HSM later" appears in a design document with no date. The emulator is not a cheaper HSM; it is a different security model that happens to share an API. Its presence in a production key path is failure mode six in section 11.

### 3.6 Matching the Class to the Requirement

| Requirement | Class that answers it | Class that does not, and why | Integrity |
|---|---|---|---|
| One key estate, many applications, one audit trail | Network-attached general-purpose | Cloud HSM, unless the estate is already in that cloud; a card per host multiplies the audit problem | ✅ |
| Sub-millisecond signing at high volume inside one server | PCIe card or embedded module | Network appliance - the hop is milliseconds and the queue is shared | ✅ |
| PIN translation and MAC inside a card authorisation budget | Payment HSM | General-purpose appliance - it lacks the PIN block and DUKPT instruction set and, critically, the approval | ✅ |
| A hardware key boundary for a cloud-native workload | Cloud HSM | On-premises appliance - the latency and the network path usually kill it | ✅ |
| Development, CI, conformance testing | SoftHSM v2 | Any real device - the cost and the ceremony make it unusable for ephemeral test environments | ✅ |
| "The regulation says the key must be in a validated module" | Any class whose **module** carries the certificate at the required level and version | Any class where you cannot produce the certificate for the firmware you will run - see section 4.7 | ✅ |

The last row is the one that gets missed. The class is a deployment decision; the **certificate** is a module decision; and a device can be the right class and the wrong certificate, which is a conversation about section 4 rather than section 3.

## 4. The Validation Regime

### 4.1 What FIPS 140-3 Is, With Dates

**FIPS 140-3 is the current US and Canadian standard for cryptographic-module validation.** It was published on **22 March 2019**, superseding FIPS 140-2 (which was issued 25 May 2001, with change notices to 3 December 2002) ✅ (NIST FIPS 140-3 publication page; FIPS 140-2 front matter). It "became effective September 22, 2019, permitting CMVP to begin accepting validation submissions under the new scheme beginning September 2020" ✅ (NIST CMVP, FIPS 140-3 Standards page, footer "Updated August 21, 2026").

The structural fact that people miss, and it explains why the requirement text is hard to read: **"Rather than encompassing the module requirements directly, FIPS 140-3 references ISO/IEC 19790. The testing for these requirements will be in accordance with ISO/IEC 24759."** ✅ (same page). So the normative requirements live in **ISO/IEC 19790** (published August 2015, folding in the 2012 edition and a 2015 corrigendum) and the test requirements in **ISO/IEC 24759** (published March 2017), and the American modifications to those test requirements are in **NIST SP 800-140** (March 2020, Kim Schaffer) and its parts 800-140A through 800-140F, which "modifies the test (TE) and vendor (VE) evidence requirements of ISO/IEC 24759" ✅ (SP 800-140 abstract).

The consequence for a reader trying to check a claim: **the FIPS 140-3 document is short, and the requirement it points at is paywalled.** That is why section 4.2 quotes FIPS 140-2 for the level wording - it is superseded, but it is the free public text that states the ladder, and FIPS 140-3 carries the same four-level architecture while delegating the normative text to ISO/IEC 19790:2012. This guide says that explicitly rather than pretending the free document is the current normative one. ✅

The requirement areas FIPS 140-3 names, because they are the checklist a security policy is written against: module specification; interfaces; roles, services and authentication; software/firmware security; operating environment; physical security; non-invasive security; sensitive security parameter management; self-tests; life-cycle assurance; mitigation of other attacks ✅ (FIPS 140-3 abstract). Note that **physical security is one of eleven**, which is a useful corrective to the habit of treating "it's an HSM" as a synonym for "it's secure".

### 4.2 The Four Levels, in NIST's Wording

The standard "provides four increasing, qualitative levels of security" ✅ (FIPS 140-3 abstract). The level-by-level physical-security wording below is quoted from FIPS 140-2 sections 1.1 to 1.4, which remains the free public statement of the ladder; the same architecture is carried by FIPS 140-3 with the normative text in ISO/IEC 19790.

| Level | Physical security, in the standard's words | Authentication and interface consequences | Integrity |
|---|---|---|---|
| **Level 1** | "No specific physical security mechanisms are required ... beyond the basic requirement for production-grade components." | Role-based authentication is not required beyond the basic; the module may be a software module on a general-purpose computer | ✅ (FIPS 140-2 s.1.1) |
| **Level 2** | Adds "tamper-evidence, which includes the use of tamper-evident coatings or seals or for pick-resistant locks on removable covers or doors" | Role-based authentication required; the operating environment must be a controlled one at the OS level | ✅ (FIPS 140-2 s.1.2) |
| **Level 3** | "attempts to prevent the intruder from gaining access to CSPs ... mechanisms are intended to have a high probability of detecting and responding to attempts at physical access, use or modification ... may include the use of strong enclosures and tamper detection/response circuitry that zeroizes all plaintext CSPs when the removable covers/doors ... are opened" | Identity-based authentication required; plaintext CSP entry and output must occur on physically or logically separated ports, or via a trusted path | ✅ (FIPS 140-2 s.1.3) |
| **Level 4** | "a complete envelope of protection ... detecting and responding to all unauthorized attempts at physical access. Penetration ... from any direction has a very high probability of being detected, resulting in the immediate zeroization of all plaintext CSPs ... useful for operation in physically unprotected environments" | As Level 3, plus environmental failure protection (voltage and temperature) or rigorous environmental failure testing | ✅ (FIPS 140-2 s.1.4) |

Four things to notice in that table, because they are the four things buyers get wrong:

1. **The levels are qualitative, not a score.** "Increasing" does not mean Level 4 is "twice" Level 2; it means the requirement set is a superset with qualitatively stronger claims in specific areas.
2. **Level 2 is tamper-*evident*, Level 3 is tamper-*responsive*.** The single word difference is the difference between "you will know" and "it will erase itself", and it is the most commonly misquoted fact in vendor material.
3. **Level 3 is where the identity-based authentication and the trusted path arrive**, which is why Level 3 is the normal floor for a payment or general-purpose appliance rather than a stretch goal.
4. **Level 4 is about the physical environment, not about better cryptography.** Its distinguishing requirement is the complete envelope and the environmental failure protection - it is the level you need when the device sits somewhere you do not control. The cryptographic algorithms are the same at every level.

### 4.3 What a Certificate Covers and Does Not

| What the certificate establishes | What it does not establish | Integrity |
|---|---|---|
| That **this module**, at **this hardware and firmware version**, meets the requirements for **this level** | That the **appliance** around the module is secure - chassis management, network stack, host OS | ✅ |
| That the module's algorithms and self-tests behave as specified, and that the CAVP algorithm certificates it points at exist | That the algorithms are the ones your application actually uses, or that your protocol negotiates them | ✅ |
| That the module resists the specified attack classes at that level, as evaluated | That the deployment's configuration, key handling or network exposure is correct | ✅ |
| That an independent laboratory and CMVP reviewed the vendor's evidence against the security policy | That the key lifecycle is managed, the ceremony happened, or the operators are trained | ✅ |
| That a **security policy document** exists and is public, describing the services, roles and CSPs | That anybody in your organisation has read it | ✅ |

The five rows on the right are not hypothetical risks; they are the normal state of affairs. A validated module inside an unvalidated system is not a scandal, it is Tuesday - the residual risk simply lives in the system and must be assessed there. The KMIP guide, section 9.4, argues that distinction in principle; this section's contribution is the **version-binding** consequence, which is operational and bites:

**A firmware update can move you off the validated version.** The certificate names a version. If you apply a patch that changes the firmware version and the vendor has not re-validated, you are running an unvalidated module - possibly with the same cryptography, possibly not, and definitely without a certificate that matches what is on the tin. This is not a pedantic point; it is the reason section 9.4 treats firmware as a change-control event with a validation check in it, and the reason section 4.7 makes "does the certificate cover the version I will actually run" the first procurement question.

### 4.4 The Transition, and 21 September 2026

The transition from FIPS 140-2 to FIPS 140-3 has a hard date and a soft consequence, and they are routinely conflated.

The hard date, from the CMVP transition page: **"FIPS 140-2 modules can remain active for 5 years after validation or until September 21, 2026, when the FIPS 140-2 validations will be moved to the historical list."** ✅ (NIST CMVP, FIPS 140-3 Transition Effort page, footer "Updated April 13, 2026"; transition schedule table "Updated 06-02-2021")

The soft consequence, from the same page: **"Even on the historical list, CMVP supports the purchase and use of these modules for existing systems."** ✅ And the procurement guidance that follows from it: CMVP **"recommends purchasers consider all modules that appear on the Validated Modules Search Page ... regardless of whether the modules are validated against FIPS 140-2 or FIPS 140-3."** ✅

So the honest reading, as of October 2026, is three sentences. **A 140-2 certificate is not revoked on 21 September 2026; it is moved to a historical list.** **The modules remain usable and supported for existing systems.** **What changes is the answer to "is this a current validation?"** - which is the question a procurement reviewer, an auditor and a cloud provider's marketing page all ask differently.

Two further facts on the timeline, because they explain why 140-3 took so long to populate: FIPS 140-3 was approved **22 March 2019** and effective **22 September 2019**; the SP 800-140x documents that modify the ISO test requirements were published **20 March 2020**; and CMVP began accepting submissions under the new scheme in **September 2020** ✅ (transition schedule table). A validation regime that started accepting submissions in late 2020 had, by late 2026, six years to populate - and the population is uneven by product class, which is exactly why the three cloud providers in section 8.4 document different levels.

**What this guide will not do:** state how many certificates moved on 21 September 2026. The CMVP page read this pass still used future tense, so I could not confirm the move had completed or count the certificates affected. Section 15 records that as an unverified item rather than guessing at a number.

### 4.5 Common Criteria, Named

**Common Criteria (ISO/IEC 15408) is a separate evaluation regime with its own assurance levels, the EALs, and it is not a substitute for FIPS 140-3 nor a synonym for it.** It evaluates a **Target of Evaluation** against a **Security Target**, at an **Evaluation Assurance Level** from EAL1 to EAL7, and it is the regime under which smart cards, payment terminals and many HSMs are separately evaluated.

The concrete link worth knowing, because it is genuinely useful in a design review: **Common Criteria EAL levels appear inside the FIPS text itself, as an operating-system evaluation dependency.** FIPS 140-2's operating environment requirements reference EAL2, EAL3 and EAL4 at Levels 2, 3 and 4 respectively - that is, a module claiming a level may satisfy its OS requirement by running on an operating system evaluated to the corresponding EAL. ✅ (FIPS 140-2, operating environment requirements, sections 1.2-1.4)

That fact does two things. It shows that the two regimes are **interlocking rather than competing** - CC is one of the ways a module demonstrates a FIPS requirement. And it warns against the common procurement move of substituting one for the other: a CC certificate at some EAL is a statement about a protection profile and an assurance package, and it does not tell you the module's FIPS level, which is what most payment and government workloads actually name.

**What this guide does not do:** claim any specific product's CC status. I did not read a CC certificate this pass, and a claim about a named vendor's EAL would be exactly the kind of unverified assertion section 15 exists to catch.

### 4.6 PCI PTS HSM as a Separate Device Approval

**PCI PTS HSM is a payment-device approval, run by the PCI Security Standards Council, and it is separate from FIPS 140.** It is not a level of FIPS, it is not awarded by NIST, and a device can hold one without the other - though in practice a payment HSM holds both, because the two regimes answer different questions.

- **FIPS 140-3 answers:** is this cryptographic module correctly implemented and physically robust at level N?
- **PCI PTS HSM answers:** is this device fit to hold and use payment key material in a card-processing estate, under the approval regime the card industry recognises?

**PCI PTS HSM v5.0 was published on 18 May 2026** ✅ (PCI SSC blog, "PCI SSC Publishes PCI PTS HSM v5.0", posted 18 May 2026 by Alicia Malone; the PTS HSM topic page carries the same date). It is described as a "major revision ... from version 4.0 to version 5.0", addressing "modern cryptographic practices, cloud and multi-tenant deployments, and emerging threats such as post-quantum risks" ✅ (same). Section 7.8 covers what changed and why each change matters to an operator; section 10.3 places it in the regulatory stack. The point to carry out of section 4 is structural: **there are three separate approval regimes in play** - FIPS 140-3/CMVP for the module, Common Criteria for a broader evaluation, and PCI PTS HSM (plus PCI KMO for operations, section 8.6) for the payment device and its operation - and collapsing them into one mental category is how procurement teams end up asking a vendor for "the FIPS certificate" when what the scheme requires is a PTS HSM approval, or vice versa.

### 4.7 The Procurement Question This Actually Settles

The question a validation regime settles is **not** "is it secure". It settles a narrower and much more useful question:

> **Which named module, at which version, at which level, holds a current certificate, and does the firmware I am about to run appear on it?**

That question decomposes into six checks, and each one is a document you can ask for:

| Check | The document that answers it | The answer that should worry you | Integrity |
|---|---|---|---|
| **Module name and version** | The certificate, and the vendor's security policy | The certificate names a version the vendor no longer ships, or a version you cannot identify from the part number | ✅ |
| **Level, per requirement area** | The certificate plus the security policy's level table | A single headline level with no per-area breakdown - the physical security level and the overall level are not always the same claim | ✅ |
| **Firmware you will run** | The certificate's version field, against the vendor's release notes | A patch released after the certificate, applied by default | ✅ |
| **Algorithm coverage** | The CAVP certificates the module's security policy references | Algorithms you need that are not in the validated set - common at the edges (a specific curve, a specific mode) | ✅ |
| **Approval for the payment use** | The PCI PTS HSM listing, separately | A FIPS certificate offered as the answer to a PTS question | ✅ |
| **The security policy itself** | The public security policy document | A vendor who cannot produce it - the certificate is not the whole evidence, the policy is where the roles and services are described | ✅ |

**The discipline in one sentence:** ask for the certificate **and** the security policy, check the version field against your build, and check the payment approval separately. Everything else in this guide's section 4 is context for those three moves.

### 4.8 A Module Is Not a Deployment

Stated once here, and then cited rather than repeated, because the KMIP guide owns the argument in principle (section 9.4) and this guide owns the device: **FIPS 140 validates a module, not a deployment.**

What that means at the device, specifically, in this guide's terms:

- The certificate does not know whether the partition's security officer and the application's user are the same person. That is section 2.4.
- The certificate does not know whether the ceremony happened, or whether the components were written down. That is section 6.
- The certificate does not know whether the backup quorum can be reassembled. That is section 9.3.
- The certificate does not know whether the rate limit was raised to make a latency number go green. That is section 2.3.
- The certificate does not know whether the device in the rack is the device in the certificate - a swap during a repair, a refurbished unit, a spare that was never re-initialised. That is section 11.

**And the reverse is also true, which is the part the sceptics miss:** a deployment can be excellent and the module still matters, because the module is the part that holds the key when every process control has failed. The certificate is a floor, not a roof - and a floor is worth having.

## 5. The Device Interfaces

### 5.1 What an Interface to a Device Actually Exposes

An HSM does not expose "cryptography". It exposes a **small, opinionated surface**: somewhere to authenticate, somewhere to hold objects, a way to name an object, a way to say what to do with it, and a way to find out what happened. Every interface in this section is a different arrangement of those five things, and the differences that cause outages are almost never about algorithms - they are about **naming, lifetime, and what the interface does when the device is not there**.

| Surface | What it is on the device | What it is on the host | Integrity |
|---|---|---|---|
| **Authentication** | A partition identity with a PIN or credential, checked in firmware | A configuration file, an environment variable, a secrets-manager fetch, or a hardcoded string in a properties file | ✅ |
| **Object space** | The partition's key store, with per-object attributes | Handles, aliases or labels that the application maps to device objects | ✅ |
| **Naming** | Persistent attributes on the object (label, ID, class) | Application-side identifiers that must be kept in sync with the device | ✅ |
| **Operation** | A mechanism - a specific algorithm-mode combination the device implements | A call into a library that translates it into a device command | ✅ |
| **Observation** | The device audit log and per-partition statistics | Whatever the client library chooses to expose, plus the log export | ✅ |

The pattern worth internalising: **the device's model is persistent and the host's model is ephemeral.** A key object exists on the device until somebody deletes it; the session that used it exists for milliseconds. Every integration bug in this section is a confusion between those two lifetimes.

### 5.2 PKCS#11

The dominant interface for general-purpose devices, and the one most people mean when they say "the HSM API". Its vocabulary is the decoder table in section 1.2, and the four terms that matter operationally:

- **Slot.** An address at which a token may be present. An application enumerates slots, finds the one whose token matches its expectation, and opens a session. The classic production bug is **slot-number drift**: a device reboots, a partition is added, and slot 3 is now a different partition - an application pinned to a slot number is now talking to the wrong key store, or failing to initialise. Pin to the **token label**, not the slot index.
- **Session.** A logged-in context, with a lifetime. Sessions are a resource: the device has a finite number, they leak when applications crash without closing them, and a leak presents as "the device is full" rather than "the application is broken". Pooling is mandatory at scale, and section 5.6 has the trap in pooling.
- **Object.** A key, certificate or data object inside the token, with a **handle** that is valid only for the life of the session. Handles are not names; an application that caches a handle across sessions is caching a dangling pointer into the device.
- **Attribute.** The persistent facts about an object: class, key type, length, label, ID, whether it is sensitive, whether it is extractable, whether it is a token object (persists) or a session object (disappears). **`CKA_EXTRACTABLE` and `CKA_SENSITIVE` are the device's refusal expressed as attributes**, and their defaults are worth reading in the vendor's documentation rather than assuming.

**The architectural detail that decides your availability model:** on most network-attached appliances, the PKCS#11 library on the host is **not** talking to the device directly - it is a vendor client that proxies over TCP to the device's own service, with a local configuration file naming the device, the partition and the credentials. That proxy is where timeouts, retries, session pooling and failover behaviour live, and it is the component that turns a device failure into an application behaviour. Section 5.6 is about it.

**Where PKCS#11 ends:** it is a **use** interface. It can create objects, use them and delete them; it has no concept of a lifecycle, a cryptoperiod, an inventory across devices, or a policy that outlives a session. That is the gap the KMIP guide's section 7 describes as the "use versus manage" distinction, and this guide does not re-argue it.

### 5.3 JCE and JCA

The Java Cryptography Architecture and its extension, the Java Cryptography Extension, reach an HSM through a **provider**: a class that implements the JCA/JCE service provider interface and translates `Cipher`, `Signature`, `Mac`, `KeyGenerator` and `KeyStore` calls into device operations. The HSM vendor ships the provider; the application registers it by name and, in most cases, by position in the provider list, and then either names it explicitly in a `getInstance` call or relies on the list order.

Four consequences that are specific to this stack:

- **Provider position is load-bearing.** An application that says `Cipher.getInstance("AES/CBC/PKCS5Padding")` without naming the provider gets whichever provider is first in the list, which may be the software provider - so the operation succeeds, the tests pass, and the key is being used in software. This is the single most common way an HSM ends up bypassed without anyone deciding to bypass it.
- **Key objects are opaque handles.** A `PrivateKey` or `SecretKey` obtained from the device's `KeyStore` may hold a reference rather than material, which is exactly what you want - and which means serialising it produces something that cannot be reconstructed elsewhere, by design.
- **The KeyStore is the naming layer.** JCE applications typically address keys by alias in a `KeyStore` that the provider implements against the device's partition. Alias management becomes key management, and an alias renamed during a deployment is a production incident.
- **Threading and session models are the provider's business, and undocumented in the general case.** Two threads sharing one `Cipher` instance, or a provider that serialises all operations behind one session, produces latency behaviour that looks like a device problem and is a provider problem.

### 5.4 CNG and the Key Storage Provider

On Windows, the modern interface is **CNG (Cryptography Next Generation)**, and the device-facing piece is a **Key Storage Provider (KSP)** - the successor to the older Cryptographic Service Provider (CSP) model. A KSP implements key creation, import, export (where permitted), signing, encryption and deletion against a key store, and the operating system's own consumers - IIS, SQL Server, Active Directory Certificate Services, Schannel - reach the HSM through it without knowing what is underneath.

The properties that matter operationally:

- **The KSP is a machine-wide registration**, not an application configuration. Installing or upgrading it affects every consumer on the host, and an upgrade that changes behaviour changes it for all of them at once.
- **Key names live in a machine-scoped namespace**, which means two applications that both want a key called "tls-prod" collide, and the collision is silent.
- **Permissions are the OS's**, layered on top of the device's partition roles - so the effective permission on a key is the intersection of an ACL and a device role, and diagnosing "access denied" requires reading both.
- **The older CSP model still exists** and some line-of-business software only speaks it, which is why Windows estates often run both interfaces against the same device and treat them as one integration. They are two.

### 5.5 Vendor-Native Host Commands

Payment HSMs, and some general-purpose devices, expose a **host command interface**: a message-oriented protocol in which the host sends a command code plus a data block and the device returns a response code plus a data block. It is older than PKCS#11, it is not standardised across vendors, and it is the interface that payment applications actually use - because the commands are the payment operations (translate a PIN block, generate a MAC, derive a key, verify a CVV) rather than generic cryptographic primitives.

The practical characteristics, and they are why this interface survives:

- **It is message-shaped, not call-shaped.** The device receives a whole request, so the operation and its parameters are one atomic unit - which maps well onto a financial message that already has a structure.
- **Response codes are the error model.** A two- or four-character response code carries the outcome, including the security-relevant failures (invalid key, key not found, MAC failure, PIN block format not permitted). Application error handling is a lookup table over those codes, and a table that treats every non-success as "retry" is how a security failure becomes an outage.
- **The interface is a protocol surface with its own security.** Host commands can be authenticated and encrypted on some devices, and on others they are protected only by the network. A payment HSM reachable on a flat network with an unauthenticated host command port is a device whose refusal applies to the key and not to the operation.
- **Versioning is by device firmware, not by protocol.** A new command appears when the firmware supports it, and an application that sends it to an older device gets a response code, not a negotiation failure.

### 5.6 The Integration Properties That Decide Outages

Five properties, none of them about cryptography, all of them about whether the estate stays up.

| Property | The failure it produces | The guardrail | Integrity |
|---|---|---|---|
| **Client library and firmware version match** | After a host patch, the application cannot connect at all, or connects and fails on one operation | Treat library and firmware as one versioned unit; test the pair before either moves | ⚠ (vendor-documented in general; specific compatibility matrices not read this pass) |
| **Session pooling** | Sessions leak on crash; the pool is exhausted; new requests queue; latency climbs before anything errors | Pool with a hard cap, a health check that returns a session, and an alert on pool saturation | ✅ |
| **Timeout and retry semantics** | A slow device is retried three times, tripling the load on a device that was merely busy, converting latency into an outage | Retry only on idempotent operations, with jitter and a budget; never retry a failed MAC or a failed PIN translate blindly | ✅ |
| **Failover behaviour of the client** | The client fails over to a standby that has the partition but not the key, or has the key but not the policy | Failover is tested with a real key operation, not a TCP connect | ✅ |
| **Fork safety** | A forked worker inherits an open session and two processes share one device session, corrupting state | Initialise after fork; never inherit a provider across a fork | ✅ |

**The single most valuable test in this list** is the fourth: kill the active device, and make a real cryptographic call. A TCP health check that goes green on the standby proves the network works; it proves nothing about whether the standby can serve the key, which is the thing the application needs. This is the deliberate-failure method that section 9.1 turns into a quarterly drill.

### 5.7 The Boundary Against the Protocol Comparison

This section describes each interface **as an interface to a device** - what it exposes, how it names things, and how it fails. The comparison of PKCS#11 against KMIP as *architectures* - the local-API-versus-network-protocol argument, the comparison table, the MPC contrast, and the KMIP 3.0 PKCS#11 profiles - belongs to the KMIP guide, section 7, and is not reproduced here. The one sentence worth carrying across: **PKCS#11 answers "use the key in this token"; KMIP answers "manage the lifecycle of keys in that key manager".** If your question is which one to integrate, read section 7 there; if your question is what the device does when you call it, you are in the right section.

## 6. The Key Ceremony

### 6.1 Why a Ceremony Exists at All

Start from the mechanical fact, because the ceremony is usually presented as a compliance ritual and it is actually a forced move.

**You cannot export a backup of the master key in usable form.** The device will not give it to you. The only two places the key can exist are (a) inside the device, and (b) as **components** - separate physical items, each useless alone, which must be combined inside a device to reconstruct it. Therefore the only way to create a second copy, to recover from a device loss, or to bring a new device into an HA group is to physically handle those components and load them, under conditions where no single person can walk away with the whole key. That is a ceremony. It exists because of section 2.2, and its shape is dictated by the physics rather than by a standard.

The second mechanical fact: **the data the device protects is often irreplaceable in a way that makes recovery non-optional.** A payment HSM's master key protects every zone key beneath it; lose the master key and every terminal under it must be re-injected. A CA's signing key loss is a trust-anchor event. So the ceremony is not only about preventing a single person from knowing the key - it is equally about **producing a recoverable copy without producing a readable one**.

### 6.2 What Physically Happens

The sequence varies by vendor and by device class, but the shape is stable, and this is the shape to write a runbook against.

1. **Preparation, days before.** Devices identified by serial number. Components ordered or retrieved from custody, with their own chain-of-custody record. Room booked and access list agreed. Script reviewed and signed off by someone who will not be in the room. A second, independent copy of the script in the room.
2. **Room setup.** Devices powered, connected only to what they need, console access established, and the room swept for recording devices. Phones and cameras out. On a payment device, the device is typically not on the production network at this stage.
3. **Initialisation.** The security officer initialises the device or the partition, sets the policy, and establishes the identities that the ceremony will use. This is done before the key material exists.
4. **Component generation and entry.** The master key is either generated by the device and split into components, or entered as components that the device combines. Components are handled by different custodians, each of whom knows only their own.
5. **Verification.** The device produces a **check value** - a value derived from the key by a one-way function - and every custodian independently confirms it matches the check value recorded at the previous ceremony or printed on the component envelope. This is the step that catches a mis-entered component before it becomes a production outage three months later.
6. **Loading into the estate.** The same key material is loaded into every device that must share it - the HA group, the DR device, the standby - each load being its own dual-controlled act with its own check-value verification.
7. **Backup.** The components are sealed into tamper-evident bags, each custodian signs across the seal, and the bags go to separate physical custody. The custody register is updated in the room.
8. **Test.** A real cryptographic operation is performed against the newly keyed device and its result verified against a known-good value - a test vector, or an operation performed before the ceremony on the old device.
9. **Close.** The log is exported, the room is cleared, the device is put on the network, and the record is assembled and signed.

**The step people skip is 5**, the check value. It is the only cheap verification in the whole procedure, and skipping it means the first evidence of a bad component is a production failure - at which point you are performing an emergency ceremony with no rehearsal and no witnesses who were expecting to be there.

### 6.3 Who Must Be Present, and Where They Stand

The physical arrangement is not decoration; it is the control.

- **Two or more custodians**, each holding a component or a credential, none of whom holds a complete set. They stand so that neither can see the other's entry, and so that the console or key-loading device is visible to a witness.
- **An independent witness**, whose job is to attest that the script was followed. "Independent" means not a custodian, not the operator, and not somebody whose performance review depends on the ceremony finishing on time.
- **An auditor or compliance observer** on ceremonies with a regulatory dimension, who takes the record away rather than signing it in the room.
- **A vendor field engineer**, sometimes, for the initial device commissioning - and this is the role most often unmanaged. The engineer has administrative access to the device and is not your employee. The control is that the engineer's actions are performed under observation, logged, and covered by the device's own audit trail, and that the engineer does not hold a component.
- **Nobody else.** Every additional person in the room is an additional person who could memorise a component. The guest list is a security control.

**The rule that makes the arrangement work:** at no point is a single person alone with an unsealed component or an authenticated administrative session. That rule is checkable by reading the room log against the device log, which is why both exist.

### 6.4 Split Knowledge and Dual Control at the Device

The two controls are different and are constantly conflated - the KMIP guide distinguishes them in section 8.4, and this section only states what they mean **at the device**.

- **Split knowledge** means the key material is divided so that no component is usable alone. At the device, this is expressed as components: N physical items, of which M are needed. The device combines them internally; the split never exists in readable form outside.
- **Dual control** means two people must act for a sensitive operation to complete. At the device, this is expressed as two authenticated identities - two smart cards, two passwords, two console logins - required for the operation to proceed, and logged as two distinct actors.

You can have either without the other, and both are weaker alone. Split knowledge with one custodian who happens to hold all components is dual control in name. Dual control with both people knowing the same complete key is split knowledge in name. **The device can enforce the mechanism; only the process can enforce the separation** - which is why the custody register, not the device, is the artifact an auditor spends time on.

### 6.5 Key Components and Key-Loading Devices

The physical kit, because it is the part that is never written down:

- **Smart cards** as component carriers, usually in pairs, with the card's own PIN as a second factor - so a stolen card is not a stolen component.
- **A key-loading device (KLD)**, a small tamper-responsive unit that holds a component and presents it to the HSM in a controlled way, so that the custodian never reads the component aloud. On payment estates the KLD is itself a controlled item with its own custody.
- **Component envelopes and tamper-evident bags**, numbered, signed across the seal, and stored in separate safes - the separation being the point, since two components in one safe is one component for anyone who can open the safe.
- **The custody register**, which is the actual control artifact: who holds which component, where, since when, transferred to whom, witnessed by whom.

**The failure that this kit is designed to prevent is the photograph.** A component written on paper, photographed, or typed into a laptop "for safety" converts split knowledge into split theatre, and it is the most common finding in a payment-HSM review. The guardrail is procedural and blunt: components are never transcribed, and the register records the *location*, never the value.

### 6.6 The Ceremony Record

The record is the deliverable. A ceremony that happened and produced no record did not happen - the KMIP guide, section 8.5, makes that argument as a governance matter; here is what the document must physically contain to satisfy it.

| Element | Why it must be there | Integrity |
|---|---|---|
| **Script, versioned, with the version actually used** | Proves what was supposed to happen, and that the version in the room was the approved one | ✅ |
| **Date, time, location, room** | Places the event; allows cross-reference to physical access logs | ✅ |
| **Device serial numbers and firmware versions** | Ties the ceremony to specific hardware, which is what a later audit will check | ✅ |
| **Names and roles of every person present** | The attestation set; also the guest-list control in 6.3 | ✅ |
| **Component identifiers and custody transfers** | The chain from generation to safe, with no gap | ✅ |
| **Check values, recorded, and confirmed by each custodian** | The only technical evidence that the components combined to the intended key | ✅ |
| **Test-operation result** | Evidence the device actually works with the key, before it went into production | ✅ |
| **Device log export, hash-stamped** | Machine corroboration of the human record - the part that is hard to fabricate | ✅ |
| **Signatures of custodians and witness** | What an auditor reads months later | ✅ |
| **Storage location of the sealed components, and of the record itself** | A record in the same safe as the components is one failure away from being no record | ✅ |

Two details that separate a good record from a compliant-looking one: **the device log export must be hash-stamped at export and stored separately from the human record**, so that a later alteration of one is detectable against the other; and **the record must be retrievable by somebody who was not in the room**, in a system the payments team does not administer. A ceremony record that only the custodian can find is a record that will not survive the custodian's departure.

### 6.7 Why It Is the Most Under-Documented Process in the Discipline

Five reasons, and they compound:

1. **It happens twice a decade, not twice a sprint.** Nobody's muscle memory holds it, and the runbook is written by someone who has done it once, under supervision.
2. **It is vendor-specific in its details and generic in its principles**, so the vendor's procedure document is the only accurate source - and it is usually a PDF attached to an engagement letter rather than a maintained document.
3. **It is performed by senior people who are not documentation's natural allies.** The security officer and the custodians are experienced; the ceremony works because they know what they are doing; and that knowledge is exactly what is not written down.
4. **It is uncomfortable to rehearse.** A rehearsal consumes components, device time and two senior people's day, and produces nothing visible. So it is skipped, and the first full run is the production run.
5. **The evidence is diffuse.** The record spans a script, a room log, a device log, a custody register and a set of signatures, held by different functions. Nobody owns the whole artifact, so nobody notices when one part is missing.

**The guardrail, and it is the same one the KMIP guide reaches from the governance side:** write the runbook from the *next* ceremony backwards - assume the person running it has never seen one and the person who wrote it has left. Then rehearse the parts that can be rehearsed without consuming components: the room setup, the console steps, the check-value verification, the log export, the record assembly. A ceremony rehearsed on a lab device on a Tuesday is worth more than a ceremony script reviewed by a committee.

## 7. The Payment HSM

### 7.1 What Makes a Payment HSM a Different Machine

A general-purpose HSM holds a key and performs an operation. A payment HSM does that **inside a card transaction, in a queue of other transactions, against a key hierarchy that somebody else designed, with an approval that somebody else granted**. Four differences, and each one changes the operational model:

- **The instruction set is payment-shaped.** Translate a PIN block from one zone key to another. Generate or verify a MAC over a financial message. Derive a per-transaction key from a terminal's injected key. Verify a CVV. Generate a PIN offset. These are single host commands, not compositions of generic primitives.
- **The key hierarchy is dictated by the card schemes**, not by the bank's key-management policy. Section 7.6.
- **The device is in the authorisation path**, so its latency is part of the transaction budget and its availability is part of the payment service. Section 7.7.
- **The approval is a payment approval** - PCI PTS HSM - not a general cryptographic-module validation. Section 7.8.

### 7.2 PIN Blocks, and ISO 9564-1:2017

**ISO 9564-1:2017**, "Financial services - Personal Identification Number (PIN) management and security - Part 1: Basic principles and requirements for PINs in card-based systems", is the fourth edition, dated 2017-11, from ISO/TC 68/SC 2, and it "cancels and replaces the third edition (ISO 9564-1:2011)" while incorporating Amd 1:2015 ✅ (front matter read from the ISO preview). The clause structure below is read from the standard's own table of contents; the **format mechanics** are restated from secondary sources, because the normative text is paywalled and I did not read it.

A **PIN block** is the PIN, padded and combined with account data, encrypted under a key, so that it can travel from the point of entry to the point of verification without ever existing in clear outside a security module. The standard's structure:

| Clause | Subject | Integrity |
|---|---|---|
| **9.3** | "Compact PIN block formats" | ✅ (TOC) |
| **9.3.1** | Construction, and the assignment of format values | ✅ (TOC) |
| **9.3.2 - 9.3.5** | Formats 0, 1, 2 and 3 respectively | ✅ (TOC) |
| **9.3.6** | Usage restrictions on the compact formats | ✅ (TOC) |
| **9.4** | "Extended PIN blocks" | ✅ (TOC) |
| **9.4.2** | Format 4 | ✅ (TOC) |
| **9.5** | "PIN block format translation restrictions" | ✅ (TOC) |
| **9.6** | Journalizing of transactions containing PIN data | ✅ (TOC) |
| **Annex A** (normative) | Destruction of sensitive data | ✅ (TOC) |
| **5.3** | PIN entry device characteristics | ✅ (TOC) |
| **6.1.1** | PIN processing systems - "host application software driving host security modules" | ✅ (TOC) |
| **6.2** | PIN encipherment | ✅ (TOC) |
| **7** | Online versus offline PIN verification | ✅ (TOC) |

The formats themselves, **restated from secondary sources and flagged as such**: Format 0 combines the PIN field with the primary account number by an exclusive-or, producing a 64-bit block encrypted under a double-length 3DES key, and is the classic acquirer-side format. Format 1 uses the same construction but with a field that is not the PAN, for environments where the PAN is unavailable. Format 2 is a variant of Format 0 with the account field right-justified and zero-filled. Format 3 is **PAN-independent** - the block carries the PIN and padding only - which is why it is used where the block must not be bound to an account. Format 4 is the **AES-based extended block**, with a 128-bit block, introduced for AES estates. ⚠ **All five descriptions are third-party restatements; I did not read the ISO text, and any implementation decision should be taken from the standard itself.**

Two operational consequences that hold regardless of the mechanics:

- **The format is part of the interface contract, not an implementation detail.** A zone that expects Format 0 and receives Format 3 produces a wrong-PIN result that looks like a customer error, and it is discovered by customers. The format value travels with the block; clause 9.3.1's assignment of format values exists precisely so that the receiver knows what it got.
- **Clause 9.5 exists because translation between formats is not always permitted**, and the restriction is a security control rather than an interoperability nicety. Accepting a block in a weak format and translating it into a strong one launders the weakness; the standard constrains which translations are allowed, and a device configured to permit all of them is a device configured to launder.

### 7.3 Translation, and Why the Device Is in the Path

The PIN is encrypted under a key shared with the acquirer at the point of entry, and must be verified by the issuer, who shares a different key. Neither party shares a key with the other, and neither is permitted to see the PIN. So somebody in the middle must **translate**: decrypt under the incoming key, re-encrypt under the outgoing key, both operations inside a security module, with the clear PIN existing only inside the device's protected memory for the duration of a single instruction.

That is the whole reason a payment HSM sits in the authorisation path. It is not there to store keys - a general-purpose device could do that - it is there because **the translation must happen somewhere, and the only acceptable somewhere is inside a boundary that neither the acquirer nor the issuer controls**. The device holds the zone keys on both sides, performs the translate as one host command, and returns a block encrypted under the outgoing key. At no point does the host application see a clear PIN, and at no point does either zone key leave the device.

The consequence for the architecture is unavoidable: **every authorisation message that needs a PIN translation makes a network round trip to the HSM, inside the transaction's latency budget.** Section 7.7 is what that does to the design.

### 7.4 DUKPT, and the Per-Transaction Key

The problem DUKPT solves is an arithmetic one. If every terminal shares a key with the host, then compromising one terminal compromises every transaction that terminal ever sent, and re-keying means touching every terminal. If every terminal has its own key, then the host must hold one key per terminal - tens or hundreds of thousands of keys - and every terminal addition is a key-injection ceremony.

**DUKPT - derived unique key per transaction - removes both problems by making the terminal's key a derivation rather than an injection.** The vocabulary, from AWS Payment Cryptography's own terminology page (a vendor's primary statement about the terms it implements) ✅:

- **BDK** - base derivation key, "commonly used as part of PCI PIN and PCI P2PE DUKPT process", key block type `TR31_B0_BASE_DERIVATION_KEY`. It is the one key the host holds.
- **IPEK** - initial PIN encryption key, derived from the BDK. "No transactions are processed on this key." It is injected once into the terminal, and it is the seed for everything the terminal will ever do.
- **The derivation counter**, carried in the transaction, advances per transaction, so each transaction's key is derived from the IPEK and the counter rather than shared.
- **X9.24-1:2009 versus X9.24-1:2017** - "The derivation method for creating an IPEK was defined in X9.24-1:2009. When an AES BDK is used, X9.24-1:2017 is the applicable standard and IPEK is replaced by Initial Key (IK)." ✅ (same page)

The detail that is more interesting than it looks: **"IPEK is a misnomer as this key can also be used to derive data encryption and mac keys"** ✅ (same page). The same mechanism derives the PIN key, the data key and the MAC key for a transaction, which is why DUKPT is not a PIN feature but the terminal estate's whole key-derivation scheme.

**Why it exists, stated as the design property:** one key per transaction, derived rather than distributed, so that a compromised terminal yields only the transactions that terminal processed after the compromise - and the derivation counter makes even those bounded, since the host can refuse counters that go backwards. **Without per-terminal key injection**, which is the operational property that made the scheme adoptable at scale.

**What it costs:** the host must track, per terminal, the highest counter seen - and that state is security-critical. A host that accepts a replayed counter accepts a replayed transaction; a host that loses the counter state must re-inject the terminal. And the scheme's security depends on the BDK, which is the single key whose compromise is catastrophic for the whole terminal estate - which is why the BDK lives in the payment HSM, under the LMK, and why its ceremony is the one in section 6.

### 7.5 MAC Generation and Verification

A financial message carries a **MAC** - a message authentication code over the message, computed with a key shared between the sender and the receiver, so that the receiver can tell the message was not altered and came from somebody holding the key. The payment HSM generates it on the outbound side and verifies it on the inbound side; the key never leaves the device, and the algorithm, padding and key selection are **specified by the card scheme**, not chosen by the bank.

What is verifiable this pass, and what is not:

- ✅ **The shape:** MAC over the transaction or ISO 8583 message, using a scheme-specified algorithm, padding convention and key, with the network verifying on receipt and the MAC key resident in the HSM.
- ✅ **The key type:** the AWS terminology page names `TR31_C0_CARD_VERIFICATION_KEY` for the CVK and describes TR-31 key blocks as binding attributes to key data, noting that **"TR-31 has been superseded by ANSI X9.143-2022"** ✅ (AWS Payment Cryptography terminology page).
- ⚠ **The exact algorithm, padding and field selection per scheme** are in scheme specifications that are partner-gated, and I did not read them. A bank's implementation is correct or not according to documents it can obtain and I cannot. This is recorded in section 15 rather than guessed at.

**The design property worth naming:** MAC verification is a **refusal** in its purest form. The device returns a boolean - verified or not - and no key material, no intermediate value, no explanation. The application cannot debug a MAC failure by inspecting anything, which is exactly the point, and exactly why MAC failures are among the hardest production incidents to diagnose: the only lever is the key and the algorithm, both of which are correct until proven otherwise.

### 7.6 The Key Hierarchy That the Scheme Dictates

A general estate's hierarchy is designed by the bank (KMIP guide, section 8.3). A payment estate's hierarchy is **specified by the schemes and the PCI standards**, and the bank implements rather than designs it. The layers, using the terminology the industry and the AWS page share ✅:

| Layer | Key | What it protects | Integrity |
|---|---|---|---|
| **Device master** | **LMK** - local master key | Every key stored in the device, in a vendor-specific variant scheme. Never leaves, never backed up in clear; its components are the ceremony in section 6 | ✅ |
| **Zone** | **ZMK** - zone master key, between two institutions | The keys exchanged between acquirer, processor and issuer | ✅ |
| **Zone PIN** | **ZPK** - zone PIN key | PIN blocks in transit between zones; the target of the translation in 7.3 | ✅ |
| **Terminal** | **TPK** - terminal PIN key, and **AWK** - acquirer working key | PIN blocks from terminal to acquirer. AWS notes AWK was historically 3DES, key block type `TR31_P0_PIN_ENCRYPTION_KEY` | ✅ |
| **Derivation** | **BDK** | The DUKPT seed for the whole terminal estate (7.4) | ✅ |
| **Verification** | **PVK** (PIN Verification Key), **CVK** (Card Verification Key) | PIN offset verification; CVV/CVC generation and verification | ✅ |
| **Personalisation** | **IMK / CMK** | EMV card personalisation | ✅ |

**The structural difference from a general estate, and the point of this subsection:** in a general estate, one key can serve one purpose and the hierarchy is a design choice. In a payment estate, **the purpose of a key is encoded in the key itself** through **variant-per-purpose derivation** - the same underlying key material is used with a variant constant to produce a PIN key, a MAC key or a data key, so that a key used for the wrong purpose produces a wrong result rather than a working operation. That is why the hierarchy is not just a naming convention: using a ZPK as a MAC key does not merely violate policy, it produces garbage, which is a *stronger* control than an access check because it fails closed at the arithmetic level.

The second structural difference: **key exchange between institutions uses key blocks, not raw keys.** TR-31 (now ANSI X9.143-2022) binds attributes - algorithm, usage, mode, exportability - to the key data, so that a key cannot be reinterpreted as a different type in transit ✅ (AWS terminology page for TR-31 and its supersession). TR-34, "an implementation of ANSI X9.24-2", does the same for asymmetric distribution. A payment estate that exchanges raw key material is a finding, regardless of how good its cryptography is.

### 7.7 The Operational Model: Latency, HA and the Shared LMK

Three properties, and they interact.

**Latency.** The device is in the authorisation path, so its service time is inside the transaction budget. A PIN translate is a single host command and the device's own service time is small - but the round trip, the queue and the client library are not, and at peak the queue is the dominant term. This is why payment HSMs are sized on **transactions per second at peak with headroom**, and why the capacity conversation is about the queue rather than about the cryptographic throughput on the datasheet. Section 2.3's rate limit is the same number seen from the security side.

**Availability.** An outage of the device stops card authorisation for every transaction that needs a PIN translation. Not degrades - stops, unless the application has a fallback, and a fallback that approves transactions without PIN verification is a fraud control failure dressed as resilience. So the device is deployed as an **HA group**: two or more devices, each able to serve any transaction, behind a client-side or network-side distribution.

**The shared LMK, which is the interesting part.** For any device in the group to serve any transaction, every device must be able to decrypt the same zone keys - which means **every device in the group shares the same LMK**. That is a deliberate weakening of the refusal: the master key exists in more than one enclosure, so the tamper response protects one copy at a time rather than the only copy.

**Why it is nevertheless correct:** the alternative is that a device failure is a payment outage, and the availability requirement is not negotiable. The design response is to make the *group* the security boundary rather than the device - the enclosures are in the same controlled space, under the same physical access regime, with the same tamper monitoring, and the LMK ceremony keys all of them in one procedure with a check value verified on each (section 6.2, step 6). **The honest way to hold this is to write it down**: the design document should say that the LMK is shared across N devices and that the physical security claim is a claim about the room, not about each box. A design that shares the LMK and then describes each device as an independent root of trust is documenting a property it does not have.

### 7.8 PCI PTS HSM v5.0 and What Changed

**PCI PTS HSM v5.0 was published on 18 May 2026**, a "major revision ... from version 4.0 to version 5.0", addressing "modern cryptographic practices, cloud and multi-tenant deployments, and emerging threats such as post-quantum risks" ✅ (PCI SSC blog, 18 May 2026). The changes that matter to an operator, as stated in the announcement:

| Change | What it means at the device | Integrity |
|---|---|---|
| **Device security keys** (firmware authentication, tamper and storage keys) "must now use cryptography with an effective key strength of at least 128 bits, and TDES is no longer permitted for device security purposes" | The keys protecting the device itself can no longer be 3DES. An estate with older firmware has a defined migration problem, and the migration is a firmware change - section 9.4 | ✅ |
| **EC-SDSA support** | An elliptic-curve signature option for device security functions, aligning the device with modern algorithm policy | ✅ |
| **New evaluation modules** for "Key-Transfer Functionality, Remote Administration, and HSM Solution Security" | Three areas that were previously assessed implicitly are now assessed explicitly - notably **remote administration**, which is how most estates are actually operated | ✅ |
| **Multi-tenant and HSM-as-a-Service controls**, "such as tenant key erasure and strict isolation between tenants" | The standard now has something to say about the cloud model in section 8, and the two requirements it names - erasure on exit, isolation in operation - are exactly the two a tenant should be asking about | ✅ |
| **Vulnerability management** moved into the lifecycle security modules | Patching is now part of the approval rather than an operational afterthought | ✅ |
| **Laboratories now perform source code review** for certain requirements | The evidence base behind the approval got deeper, and the review timeline for a new product got longer | ✅ |

**The reading that matters commercially, stated without ranking anything:** v5.0 makes the device's *own* key material subject to a modern algorithm floor, makes *remote administration* an assessed function, and makes *multi-tenancy* an assessed property. Those three together mean that a payment HSM approval now says something about how the device is operated, not only about how it is built - which is a change in kind, and it is the change that makes section 8's cloud model assessable at all.

## 8. The Cloud HSM Model

### 8.1 What Is Actually Rented

Four things, and being precise about which ones you get is the whole of this section's value:

1. **A device you do not own**, in a room you will never enter, on a network you do not control, with firmware you do not choose the timing of.
2. **An administrative identity inside it** - a security officer in your own tenancy, with your own users and your own keys, whose authentication is managed by the provider's control plane and, in some offerings, deliberately not by the provider's IAM.
3. **An interface**, which is usually the standard one (PKCS#11, JCE, CNG/KSP) proxied to the device, so that an application can be migrated with a configuration change rather than a rewrite.
4. **A tenancy boundary** - a partition or a dedicated cluster - whose isolation properties are documented by the provider and are the thing you should actually read.

**What you do not rent is the physical refusal.** You rent its *effects*: your key does not come out, your operations are rate-limited, the device wipes itself if tampered with. But the *decisions* that surround the refusal - who may open the enclosure, who may load firmware, who may read the audit log at the hypervisor level - are the provider's, and your visibility into them is whatever the provider's attestation and audit documentation gives you.

### 8.2 Where the Boundary of the Refusal Sits

The honest formulation, and the one to carry into a design review: **in a cloud HSM the refusal is still enforced by hardware, but the authority to overrule it has moved to an organisation you do not employ.**

Three consequences follow, and they are not the ones the marketing discusses:

- **Your key cannot be extracted by you, and the provider states it cannot be extracted by them.** What you can verify is the statement, the certification, and the audit evidence. Whether the provider's staff *could* in principle extract it is a question about their internal controls, which is exactly the class of claim section 15 lists as unverifiable.
- **The availability of the refusal is the provider's SLA.** A device that is unavailable refuses everything, including your legitimate operations. Your DR plan is now partly a contract.
- **The audit log has two halves.** The half you can read - your tenancy's operations - and the half you cannot - the provider's administrative actions on the host, the hypervisor, the firmware. The second half is where a tenancy compromise would live, and it is covered by the provider's own controls, not yours.

**What changes about the control argument:** in an on-premises estate, "the key is in a validated module" and "nobody unauthorised can touch the module" are two claims you can verify with the same visit. In a cloud estate they are two claims with different evidence: the first is a certificate you can read, and the second is an attestation from a party whose interests are aligned with yours but not identical. Both are legitimate; only one is auditable by you directly.

### 8.3 Tenancy and Separation

Three models, and the vocabulary is not standardised across providers:

| Model | What is separated | The property you get | The property you do not | Integrity |
|---|---|---|---|---|
| **Dedicated device** (single-tenant cluster) | The whole device, including its firmware and its audit log | Isolation at the hardware level; your security officer is the only one | Cost, and a smaller pool of capacity - you own the peak | ✅ |
| **Partition on a shared device** | A key store and a role set inside a device that hosts other partitions | Cost, elasticity, and the same interface | The physical device's fault domain, and its rate limit - a noisy neighbour is a real outage vector | ✅ |
| **Logical tenancy over a managed service** | An identity and a key namespace | Manageability, integration with the provider's IAM | Any visibility of the device at all; the refusal is entirely the provider's | ✅ |

**The separation property that matters most, and it is the one to ask about in writing:** what happens to your key material when your tenancy ends? The answer should be "it is destroyed, and the destruction is evidenced", and it is now a named requirement rather than a courtesy - PCI PTS HSM v5.0's multi-tenant controls name "tenant key erasure and strict isolation between tenants" ✅ (PCI SSC, 18 May 2026). A provider that cannot describe its erasure procedure in a document you can attach to a contract has not answered the question.

Second: **the security domain.** Azure's documentation, for example, states that "each HSM cluster uses a separate customer-specific security domain that cryptographically isolates each customer's HSM cluster" ✅ (Microsoft Learn, page "Last updated on 10/14/2026"). That is a specific, checkable architectural claim, and it is the level of specificity you should be asking every provider for - not "we isolate tenants", but "here is the mechanism and here is what it cryptographically separates".

### 8.4 The Three Providers, as Documented, With Dates

These are the providers' own statements about their own services, read on the dates shown. They are **documented validation levels**, not independent findings, and the differences between them are a fact about the documentation as of October 2026 - **not a ranking**.

| Provider | What the documentation says | Validation level documented | Date read | Integrity |
|---|---|---|---|---|
| **AWS CloudHSM** | "general purpose, single tenant, and either FIPS 140-2 level-3 or FIPS 140-3 level-3 validated for clusters in FIPS mode"; clusters run in FIPS and non-FIPS modes; "your data plane is end-to-end (E2E) encrypted and not visible to AWS, you control your own user management (outside of IAM roles)"; "The trade off for this control is you have more responsibility than if you used a managed AWS service"; documented migration path for applications using PKCS#11, JCE, CNG, KSP | **140-2 L3 or 140-3 L3**, depending on cluster mode | October 2026 | ✅ |
| **Azure Key Vault Managed HSM** | "fully managed, highly available, single-tenant, standards-compliant cloud service ... using FIPS 140-3 Level 3 validated HSMs"; page note: "We have updated our HSM fleet to a FIPS 140-3 level 3 validated firmware for both Azure Key Vault Managed HSM and Azure Key Vault Premium"; partitions migrate to healthy nodes on hardware failure | **140-3 L3** | Page "Last updated on 10/14/2026" | ✅ |
| **Google Cloud HSM** | "cluster of FIPS 140-2 Level 3 certified HSMs"; Cloud KMS as the front end; also offers a "Bare Metal Rack HSM" for single-tenancy; notes that asymmetric key operations in Cloud HSM "may incur a noticeably greater latency" than software keys | **140-2 L3** | Page "Last updated 2026-10-13 UTC" | ✅ |

**The observation, stated without a conclusion:** as of October 2026 the documented validation level **differs** across the three - one documents 140-3 Level 3, one documents either 140-2 or 140-3 Level 3 depending on mode, and one documents 140-2 Level 3. That is a difference in what is **documented**, which is not the same as a difference in what is **true** of the hardware, and it is certainly not a difference in security. It is a difference in the evidence a procurement reviewer can attach to a file today.

**Why this is the best illustration in the guide of section 4.3's argument:** a reader who chose a provider on the marketing phrase "FIPS 140 Level 3 validated" would conclude the three were equivalent. A reader who read the certificate and the page date would conclude that one of them will need a re-validation conversation sooner than the others. The marketing sentence is true in all three cases and useful in none.

**And the point that follows for a tenant:** the level is the provider's claim about a module the provider operates. Whether *your* use of it is validated is a question about your configuration, your key handling and your tenancy - which is section 4.8, applied to a device you cannot see.

### 8.5 What Changes About the Control Argument

Four shifts, and they are the substance of any cloud-HSM design review.

- **From "we control the room" to "we can evidence the provider's control of the room".** The evidence is a certification, an attestation report, a contract schedule and an audit right. If the contract has no audit right, the control argument is a hope.
- **From "the key cannot leave" to "the key cannot leave, and neither can the backup".** In several cloud models the backup is the provider's, wrapped under the provider's key, restorable only into that provider's service. That is a strong refusal and a real lock-in, and both facts belong in the same sentence.
- **From "our DR plan" to "our DR plan plus their DR plan".** A regional failure is now a scenario you do not control, and the recovery time is bounded by the provider's architecture rather than yours. The question to ask is not "is it highly available" but "what is the recovery point and time for *my* keys, in *my* tenancy, when their region fails".
- **From "the HSM team" to "the platform team plus a contract".** The role that used to own the device now owns a relationship. That is a real capability change, and it is the one that most often goes unfunded.

### 8.6 PCI KMO and the As-a-Service Validation

**PCI KMO - the Key Management and Operations Standard v1.0 - was published on 14 September 2026** ✅ (PCI SSC blog, "Just Published: PCI Key Management and Operations (KMO) Standard v1.0", 14 September 2026). It "defines security requirements, test requirements, and guidance for entities involved in the operation and management of systems that use cryptographic keys", covers the key lifecycle "from generation and conveyance, through loading and use, to archive, retirement, and destruction", and - the clause that matters for this section - **"includes the validation of remote and as-a-service based HSM implementations"** ✅ (same).

Three further facts from the announcement: it is "created in alignment with the recently published PCI HSM v5 requirements"; it is intended as "a single source for requirements covering key management operations" with an "assess-once-use-many" modular approach; and its initial focus is keys securing PIN and P2PE data, with a KMO Assessors programme and Listings "Coming Soon" as of the page read ✅.

**Why this is the most consequential regulatory development in this guide's subject area, stated plainly:** until now, the approval regime assessed the **device** (PTS HSM) and the **institution's PIN controls** (PCI PIN), and the cloud model sat awkwardly between them - the device was approved, the operator was not the entity being assessed, and the tenant's evidence chain had a gap where the provider's operations should be. KMO closes that gap by assessing **key management operations** as a subject in its own right, including when those operations are performed remotely or as a service. Section 10.4 places it in the regulatory stack; section 15 records that I could not read the requirement document itself.

## 9. The Operations

### 9.1 Availability and Clustering

An HSM is a single point of failure by construction: one enclosure, one power supply chain, one firmware, one set of keys. The industry answer is a **group**, and the group has three properties worth designing deliberately.

- **Members share key material.** For any member to serve any request, each must hold the same keys - which means the same master key, loaded by the same ceremony, verified by the same check value on every device (section 6.2, step 6). This is the weakening that section 7.7 argues is correct and must be documented.
- **Members are active-active behind a distribution layer.** Either the client library balances across a device list, or a network load balancer does. The first is simpler and fails with the application; the second survives application restarts and is a component you must itself make highly available.
- **The health check must be a cryptographic operation.** A TCP connect proves the device is listening. A real operation - a sign, a translate, a key lookup in the partition - proves the device can serve *this* key. Every HA design review should ask what the health check does, and "it pings the management port" is the answer that precedes an outage.

**The subtlety that catches mature estates:** the group's capacity is the group's, not the sum of the members', because the distribution layer's health check removes a member at the first timeout - and a member that is merely slow is removed, concentrating load on the rest, which makes them slow, which removes them too. This is a cascade, and it is prevented by a health check with hysteresis and a load balancer that degrades rather than evicts.

### 9.2 The DR Problem the Refusal Creates

Here is the sentence that makes the DR problem concrete: **you cannot copy the key to the DR site.** There is no `rsync` for an HSM. So a DR strategy is a choice among four options, and each has a different recovery time and a different security cost.

| Strategy | How it works | RTO shape | What it costs | Integrity |
|---|---|---|---|---|
| **Keyed DR device** | A device at the DR site already holds the same keys, loaded during the original ceremony | Minutes - the device is warm | A second device, keyed and maintained, plus a ceremony that covered both sites; two copies of the master key, which must be documented | ✅ |
| **Cold spare plus components** | A spare device on site, unkeyed; recovery requires the component ceremony | Hours, and it is bounded by how fast the custodians can be assembled | Components must be at (or retrievable to) the DR site, and the quorum must be reachable out of hours | ✅ |
| **Device backup and restore** | A wrapped backup restored into a replacement device, under a quorum | Hours, bounded by procurement if no spare exists | The backup key's own custody, and the fact that restore requires the same device family and a compatible firmware (9.3) | ✅ |
| **Provider-managed DR** | The cloud provider's own replication inside its tenancy model | The provider's SLA | No control of the mechanism; the backup may be restorable only into that provider's service (section 8.5) | ✅ |

**The finding that matters, and it is the one the thesis predicts:** the RTO of an HSM-dependent service is usually not bounded by technology. It is bounded by **how long it takes to get two authorised people, the components, and a device into the same room**. An organisation that writes "RTO: 4 hours" in a DR plan without having measured a component ceremony has written a number it cannot meet, and the gap is discovered during the test - or during the incident. Measure the ceremony, then write the RTO.

### 9.3 Backup and Restore of a Non-Exportable Key

There are two distinct things called "backup" and they are constantly confused:

- **Backup of the configuration** - partitions, policies, users, object attributes. This is ordinary data, it can be exported, and it should be, because rebuilding a device from scratch by hand is error-prone and slow.
- **Backup of the key material** - a wrapped blob produced by the device, encrypted under a **backup key** that is itself derived from components under a quorum, and restorable only into a device of a compatible type and firmware.

The properties of the second, which is the one that matters:

- **The backup is useless without the quorum.** That is the point: the blob can sit on a file share, in a vault, on tape, and it is not a key. What makes it recoverable is M-of-N custodians presenting their components, which means **the backup's availability and the quorum's availability are two separate risks**, and organisations routinely protect the first and lose the second.
- **Restore is a ceremony, not a restore.** It has a script, witnesses, a check value and a record, exactly like generation - and it is the ceremony that gets the least rehearsal, because it only happens after a failure and by then the people who wrote the script have moved on.
- **The restore target must be compatible.** Same device family, compatible firmware, and a partition configured to accept the backup. A backup taken on firmware N and restored on firmware N+2 is a change-control event, and vendors differ on whether it is permitted.
- **The backup key must not live with the backup.** A wrapped blob and its unwrapping components in the same safe is one burglary away from no backup at all - the same argument as section 6.6's record storage.

**The guardrail, stated as a test rather than a policy:** restore into a lab device, from the real backup, using the real quorum, on a scheduled date, and verify a real operation with a known-good check value. If that test has not run in the last twelve months, you do not have a backup; you have a file.

### 9.4 Firmware and Patch

Firmware is where the refusal meets change management, and the collision has three distinct failure surfaces.

**Validation binding.** The certificate names a firmware version (section 4.3). Applying a vendor patch that has not been validated moves the module off its certificate. The operational rule that follows: **firmware versions are approved as a set with their certificate**, and a patch that is not on the approved list is a change that requires a validation decision, not a maintenance window.

**Key destruction on update.** On some devices a firmware update preserves keys; on others, a major version update requires re-initialisation, which destroys the key store and requires a ceremony to restore. This is not a detail that can be discovered during the window - it is a question to the vendor, in writing, per firmware version, and the answer belongs in the runbook.

**Approval-version binding.** PCI PTS HSM v5.0 removed TDES for device security purposes and set a 128-bit effective-strength floor for device security keys (section 7.8). A device whose firmware cannot meet that floor has a **date** attached to it, not a preference: the estate migrates, or it holds an approval position it can no longer defend. This is the concrete example of a standards change that turns into a firmware programme.

**The failure mode in both directions.** Estates that never patch accumulate unpatched vulnerabilities in a device that sits on the network and holds the keys. Estates that patch eagerly break their validation and, occasionally, their keys. The middle path is unglamorous: a staging device of the same model, a validated-version matrix, and a change record that names the certificate.

### 9.5 Monitoring a Device That Will Not Tell You What It Is Doing

The joke is accurate: the device will not tell you what it is doing, because telling you would be an information leak. What it will tell you is **counters and events**, and the discipline is to instrument the things it will say rather than wishing for the things it will not.

| What to collect | Why it matters | The failure it catches | Integrity |
|---|---|---|---|
| **Operation rate and latency, per partition** | The queue is the early warning; latency rises before errors appear | The rate-limit-as-outage (section 11, failure four) | ✅ |
| **Authentication failures** | A burst is either a misconfigured client or an attack | Credential misuse; a stuck retry loop from a bad password | ✅ |
| **Tamper and environmental events** | The device's own testimony that something physical happened | Enclosure intrusion, temperature, voltage | ✅ |
| **Battery status and service life** | The tamper circuit depends on it | An unannounced loss of tamper response | ⚠ (vendor-specific reporting; not every model exposes it) |
| **Device clock and NTP status** | Timestamps are the evidence value of every other event | Audit records that cannot be relied on in an investigation | ✅ |
| **Firmware version, per device** | Drift from the validated set is invisible otherwise | A device patched outside change control | ✅ |
| **Log export success** | An unexported log is a log that will be lost | The circular buffer overwriting the incident | ✅ |
| **Session and connection counts** | Leaks present as exhaustion, which presents as outage | The integration bug in section 5.6 | ✅ |

**The one metric people forget is the log export itself.** The device's internal log is small and circular; if the export path breaks silently, the device keeps working perfectly and the evidence disappears backwards in time. Monitor the *absence* of exports, not just their failure.

### 9.6 The Runbook Items Nobody Writes

Six items, each of which is cheap now and expensive later:

1. **Spares, keyed or unkeyed, with a defined keying procedure.** A spare in a cupboard is a device, not a replacement.
2. **Battery service dates**, per device, with the procedure for replacement - including whether it can be done without destroying the module.
3. **Custodian succession.** Components held by people who have left, or by two people who both report to the same manager, is a quorum on paper only.
4. **Capacity headroom as a policy**, not a hope: the peak transaction rate the device was sized for, the current peak, and the review date.
5. **The device inventory with serial numbers, firmware and certificate references**, so that the next audit can be answered from a document rather than a site visit.
6. **The list of what depends on the device**, which nobody owns because it crosses teams - and which is the first thing needed when the device fails.

## 10. The Banking and Regulated Angle

### 10.1 PCI DSS, by Substance

**PCI DSS v4.0.1 is the current version** ✅ (PCI SSC document library, observed). What it requires of a key estate is stable across versions and can be stated in substance without quoting a requirement number - which is the discipline this guide applies throughout, for the reason the KMIP guide sets out in its section 9.3: a requirement number quoted from a secondary source into a regulator-facing document is a small error with a large blast radius.

The substance, in the order an assessor will walk it:

- **Keys are generated in a secure environment**, and for the strongest cases, inside the device that will use them.
- **Key distribution protects the key** - split knowledge and dual control for manual clear-text operations, wrapped or encrypted distribution otherwise.
- **Keys are changed at defined cryptoperiods**, and the cryptoperiod is documented per key type.
- **Keys are destroyed when no longer needed**, with the destruction recorded.
- **Key-management procedures are documented**, which is the clause that turns every other clause into an auditable obligation.

**What PCI DSS does not do:** it does not approve a device, it does not validate a module, and it does not tell you which HSM to buy. It states outcomes, and the device is one way to achieve them - which is why "we are PCI DSS compliant" and "our module is FIPS 140-3 validated" are two different sentences that are often spoken as one.

### 10.2 PCI PIN

**PCI PIN Security Requirements v3.1 exists and is published by the PCI SSC** ✅. Its subject is PIN management and the key management that surrounds PIN encryption, translation and verification - which makes it the standard closest to the payment-HSM estate in section 7.

**Dual control and split knowledge are genuine PCI PIN controls** covering sensitive key operations, and they are a real regulatory expectation rather than a convention - which is why section 6.4 treats the custody register as the audit artifact. ✅ (control area confirmed) / ⚠ (requirement numbers not verified this pass, and deliberately not quoted - see section 15)

The device-level consequence: the PIN standard is the reason the payment HSM's ceremony is not optional, and the reason the components are audited. An estate that has the device and not the ceremony has satisfied neither.

### 10.3 PCI PTS HSM v5.0

Stated as a regulatory fact rather than a technical one: **PCI PTS HSM is the approval regime for the payment device**, published at v5.0 on 18 May 2026 (section 7.8 has the changes). For a bank, its significance is that the *device* is approved by a body the card industry recognises, separately from the module being validated by NIST - and a procurement file that contains only a FIPS certificate is a file with a gap in it.

The v5.0 changes that a compliance function should care about, in one line each: the device's own keys now have a 128-bit floor and no TDES; remote administration is an assessed function; multi-tenancy is an assessed property with named controls; and vulnerability management is inside the approval. Two of those four are about **how the device is operated**, which is new.

### 10.4 PCI KMO v1.0

**PCI KMO Standard v1.0 was published on 14 September 2026** ✅ (PCI SSC blog, 14 September 2026). Its subject is not the device and not the institution's PIN handling but **key management operations as such** - "security requirements, test requirements, and guidance for entities involved in the operation and management of systems that use cryptographic keys", across the lifecycle "from generation and conveyance, through loading and use, to archive, retirement, and destruction", including "the validation of remote and as-a-service based HSM implementations" ✅ (same).

Three things follow for a bank:

- **The operational subject now has a standard.** The ceremony (section 6), the backup and restore (section 9.3) and the cloud tenancy (section 8) are all "key management operations", and this is the document that assesses them.
- **"Assess-once-use-many" is the commercial promise**, and its modular structure is designed so that a service provider's assessment can be reused by its customers - which is the only way an as-a-service model can be assessed at all without every customer repeating the work.
- **Alignment with PTS HSM v5.0 is explicit** ✅ (same), which means the device approval and the operations approval are designed to compose rather than to be read separately.

**What I could not do:** read the requirement document. The PDFs did not fetch this pass, so no clause numbers are quoted anywhere in this guide, and section 15 records it.

### 10.5 DORA and the Third-Party Device

**DORA - Regulation (EU) 2022/2554 of the European Parliament and of the Council of 14 December 2022 on digital operational resilience for the financial sector - "entered into force on 16 January 2023 and will apply as of 17 January 2025"** ✅ (ESMA DORA page, read this pass; EUR-Lex CELEX 32022R2554 confirms the citation and date).

Two obligations land directly on an HSM estate, and they land hardest when the device is a cloud service:

- **ICT third-party risk management.** A cloud HSM provider is an ICT third-party provider; the tenancy is a contractual arrangement that must be risk-assessed, contractually documented, and monitored. Section 8.5's "audit right" is not a nice-to-have under this regime - it is the mechanism by which the obligation is discharged.
- **The register of information.** DORA requires a register of ICT contractual arrangements, and a provider-operated HSM estate is exactly the kind of arrangement that must appear on it - which means the device inventory in section 9.6 is now a regulatory artifact as well as a good idea.

**No penalty figures.** The Regulation's sanction regime is not something this guide will characterise without reading the text, and a number quoted from memory into a compliance document is precisely the failure mode section 10.1 warns about.

### 10.6 A Validated Module Is Not a Compliant Deployment

The sentence, once, in the strongest form: **every regime in this section assesses something narrower than your deployment, and the gap between them is where findings live.**

| Regime | What it assesses | What it does not assess | Integrity |
|---|---|---|---|
| **FIPS 140-3 / CMVP** | A module, at a version, at a level | The appliance, the configuration, the keys, the operators | ✅ |
| **Common Criteria** | A target of evaluation against a security target, at an EAL | The deployment, or the FIPS level | ✅ |
| **PCI PTS HSM v5.0** | A payment device, and now some operational functions | Your ceremony, your quorum, your inventory | ✅ |
| **PCI PIN v3.1** | PIN management controls at the institution | The device's design, or the cloud tenancy | ✅ |
| **PCI KMO v1.0** | Key management operations, including as-a-service | The rest of your estate | ✅ |
| **PCI DSS v4.0.1** | The cardholder-data environment, by outcome | Which device, which level, which ceremony | ✅ |
| **DORA** | Operational resilience, including third-party arrangements | The cryptography, or the module | ✅ |

**The practical consequence:** a compliance file that answers all seven rows is a file that answers nothing about whether the key can be recovered on a Sunday. That question is answered by section 9.3's restore test, and no regime in the table requires you to run it.

## 11. The Failure Modes

Six, each with the symptom you will actually observe, the cause underneath, and the guardrail that would have prevented it. They are ordered by how often I would expect to meet them, and every one of them is a case of the refusal being routed around rather than paid for.

| # | Failure | Symptom | Cause | Guardrail | Integrity |
|---|---|---|---|---|---|
| 1 | **An HSM in a rack nobody owns** | The device is up, serving, and no team claims it; the network team thinks security owns it, security thinks the application does | Procured for a project that closed; no service owner assigned at handover; monitoring goes to a distribution list that no longer exists | Every device in the inventory has a named service owner, a monitoring target and a review date; ownership is a field in the CMDB, not a convention | ✅ |
| 2 | **A backup whose quorum cannot be reassembled** | Restore is attempted and two of the three custodians are uncontactable, or one is at a different employer | Custody tracked the *components*, not the *people*; no succession planning; the ceremony recorded names but never a deputy | Custodian register with named deputies, reviewed annually, tested by assembling the quorum as a drill (section 9.3) | ✅ |
| 3 | **A ceremony performed once and never documented** | Six months later nobody can produce the check value, the participant list or the component identifiers | The ceremony was treated as a technical operation (KMIP guide, section 8.5); the record was assembled informally and never stored | The record checklist in section 6.6 becomes a gate: the device does not go into production until the record exists and is filed outside the payments team | ✅ |
| 4 | **A rate limit that becomes an outage** | Latency climbs, then requests time out; the device shows healthy CPU and no errors | Capacity sized at launch for one application; four more applications added; the queue, not the CPU, is the constraint (section 2.3) | Per-partition rate monitoring with an alert at a defined fraction of the ceiling; a capacity review tied to onboarding, not to incidents | ✅ |
| 5 | **A cloud HSM whose tenancy was never verified** | The tenancy is in use, the keys are there, and nobody can say what the provider's staff can do to the device | The service was adopted on the strength of a marketing page and a validation badge; the tenancy and erasure documentation was never read (section 8.3) | Read the tenancy documentation before the first key; record the security-domain mechanism, the erasure procedure and the audit rights in the design | ✅ |
| 6 | **A software emulator that entered production** | The key estate is functioning, cost nothing, and the "migrate to hardware" ticket has been open for two years | The emulator was used for the pilot; the hardware was out on delivery; the migration had no date (section 3.5) | Any PKCS#11 endpoint in a production path is inventoried with its type; an emulator in production is a finding with a dated remediation, not a backlog item | ✅ |

**The pattern, stated once:** in every row the device behaved correctly. It refused, it logged, it rate-limited, it required a quorum. What failed was the human arrangement around the refusal - ownership, succession, documentation, capacity, verification - and the device had no way to notice, because noticing would have meant trusting the humans, which is the one thing it is built not to do.

**The seventh failure, which is not in the table because it is not a failure of the device at all:** the organisation that reads this list, recognises all six, and does nothing, because none of them has happened yet. Section 13 is the same list from the other direction.

## 12. The Cymbal Bank Worked Example

**Everything in this section is fictional and illustrative.** Cymbal Bank is the only institution in this guide and it does not exist. The figures are shapes for a design review, not benchmarks, and no statement here should be read as a claim about any real institution, vendor or market.

### 12.1 The Estate

Cymbal Bank, illustratively, runs three key estates that matter to this story: a payment estate of four network-attached payment HSMs in an HA group at the primary site, sharing one LMK; a general-purpose estate of two appliances serving TLS and database encryption for about forty applications; and a cloud estate where two business units have started using a provider's managed HSM service for new workloads. There is one security officer for the payment estate, one for the general estate, and a documented ceremony that has been performed twice in six years.

The trigger for the review is mundane: the payment HSM hardware reaches end of service life, the vendor's replacement model is a different chassis, and the migration plan says "restore from backup". Which is when somebody asks the question this guide exists to answer.

### 12.2 The Failure the Thesis Predicts

The plan assumed the key could be moved. It cannot: the LMK is not exportable, the backup is wrapped under a backup key whose components are held by three custodians, and one of those custodians left the bank fourteen months ago. The succession was never recorded because the ceremony document lists **names**, not **roles with deputies** - which is failure mode two in section 11, and it arrived on schedule.

So the migration that was scoped as a hardware refresh became, in the first week, a question about whether the bank can reconstruct a quorum. The answer, illustratively, was yes - two of the three components were available and the scheme in use was 2-of-3 - but the *discovery* took four days, because the custody register was a spreadsheet maintained by the person who had left, and the fourth day was spent establishing that the spreadsheet was current.

**The thesis, tested:** the device refused to export the key, exactly as designed. Every hour of that four-day delay was the refusal working. The alternative - a device that would hand over its master key to an administrator with a change ticket - would have made the migration a two-hour job and would have made the bank's entire PIN estate one compromised admin account away from disclosure. The cost was real, and it was the cost of the property.

### 12.3 The Decision the Discipline Changed

The review had been framed as a hardware procurement: which replacement model, at which level, at which cost. Section 4.7 changed the question. The team stopped asking "is the replacement FIPS validated" and started asking **"which module, at which version, at which level, and does the firmware we will actually run appear on the certificate"** - which surfaced, illustratively, that the vendor's shipping firmware was one release ahead of the validated version, and that the release in the certificate would not be available for the delivery date.

That is not a scandal; it is a sequencing problem with a known answer. The decision the discipline changed was to **accept delivery on the validated firmware and schedule the update as a separate, validated change** - rather than taking the newer firmware because it was what shipped, and discovering the validation gap during the audit eight months later.

The second decision, and the more consequential one: the ceremony was rewritten from the *next* ceremony backwards (section 6.7), with named roles and deputies instead of names, a check-value verification step that had been implicit, and a restore test added to the annual calendar. The rewrite cost, illustratively, three days of a security architect's time and one rehearsal on a lab device. The failure it addresses cost four days of a migration and could have cost the quorum.

### 12.4 What It Cost, Illustratively

The hardware refresh was the small number. The ceremony rewrite, the rehearsal, the restore test and the deputy register were each individually cheaper than the refresh and together were the larger line, because they are **recurring** - a rehearsal every year, a register review every year, a restore test every year, forever. That is what a security property costs when it is a property of a process rather than of a box: the box is bought once and the property is rented annually.

**And that is the finding to carry into a business case.** The device is a capital purchase with a depreciation schedule. The refusal is an operating expense with no end date, and the organisation that funds the first and not the second has bought a device it cannot operate - which is failure mode one, arriving quietly, in a rack nobody owns.

**Ending on the thesis:** Cymbal Bank's migration was slower, more expensive and more annoying than the plan, and every minute of that was the product working. The question the review should have asked at the start was not "what does the new device cost" but **"who holds the refusal, and can we assemble them on a Sunday"** - which is the same question as section 8's cloud tenancy, asked about a room instead of a contract.

## 13. The Anti-Patterns

Nine, each stated as the thing you will hear in a design review, and each with the mechanism that makes it wrong and the guardrail that prevents it. They are the failure modes of section 11 seen from the other direction - the decision that creates the failure, rather than the failure itself.

| # | Anti-pattern | Symptom | Cause | Guardrail | Integrity |
|---|---|---|---|---|---|
| 1 | **"We will use one identity for the security officer and the application"** | Onboarding is fast; the audit log shows one actor doing everything; nobody can say who created a key | Role separation treated as an obstacle to agility (section 2.4) | Two identities, always, with the application unable to create its own key; the separation is a gate in the onboarding checklist | ✅ |
| 2 | **"We will export the key to the new platform to save the migration"** | A wrapped key appears somewhere it should not, or the whole migration is blocked at the last step | The refusal was not costed into the plan (section 2.2); "migration" was scoped as data movement | Plan the migration as a **re-key or a keyed-device ceremony**, not a copy; if a key genuinely must move, it moves wrapped, into a device, under a role | ✅ |
| 3 | **"The emulator is fine for now, it speaks PKCS#11"** | Production traffic on SoftHSM; a "migrate to hardware" ticket older than the application | Development tool adopted for a pilot, then promoted by inertia (section 3.5) | Inventory every PKCS#11 endpoint by type; an emulator in a production path is a dated finding, not a backlog item | ✅ |
| 4 | **"It is FIPS 140 validated, so we are compliant"** | The certificate exists, names a firmware version nobody checked, and the deployment has never been assessed | The certificate over-read as a statement about the system (section 4.3) | State the claim precisely - module, version, level - and assess the deployment separately (section 10.6) | ✅ |
| 5 | **"The vendor's engineer will do the ceremony"** | The device is keyed, the components are in a vendor's possession or unknown, and the bank cannot re-key without a purchase order | Commissioning treated as a vendor deliverable rather than a bank-controlled event (section 6.3) | The vendor may perform, the bank must witness and must hold the components; the record is the bank's, and it names who stood where | ✅ |
| 6 | **"We rehearse the DR plan annually"** (and the HSM step is a TCP ping) | The plan passes every year and fails on the first real invocation | The test measured reachability, not the ability to serve a key (section 9.1) | The DR test performs a real cryptographic operation with a known-good check value against the DR device, and measures the quorum assembly time | ✅ |
| 7 | **"We will patch the firmware in the next window"** | The device runs an unvalidated version for eighteen months, or an unpatched vulnerability for longer | Firmware treated as ordinary maintenance, not as a validation event (section 9.4) | A validated-version matrix, a staging device, and a change record that names the certificate | ✅ |
| 8 | **"The cloud provider handles the HSM, so the control is theirs"** | Nobody can state the tenancy model, the erasure procedure or the audit rights | The service was adopted on a marketing page and a badge (section 8.2) | Read the tenancy documentation before the first key; record isolation mechanism, erasure, audit rights and DR in the design | ✅ |
| 9 | **"The ceremony is a one-off, we will document it later"** | The device has been in production for two years and the record was never assembled | The ceremony treated as an operational task rather than an evidence event (KMIP guide, section 8.5) | The record checklist in section 6.6 is a gate on production, and the record is stored outside the team that performed the ceremony | ✅ |

**The common structure, in one sentence:** every anti-pattern above is a decision that **buys availability, speed or cost with a piece of the refusal**, taken by somebody who did not know that was the trade - and the guardrail is always the same move, which is to make the trade visible in a document that survives the person who made it.

## 14. The State of Practice

### 14.1 What Is Certain

Four things, and they are the only four this guide will assert without qualification:

- **The validation regime is documented and dated.** FIPS 140-3, its ISO references, the four levels, the 21 September 2026 historical-list date and the CMVP's own guidance to consider both 140-2 and 140-3 modules are all primary, public and checkable (section 4). ✅
- **The payment approvals are dated.** PCI PTS HSM v5.0 (18 May 2026) and PCI KMO v1.0 (14 September 2026) are published, and their announced changes are stated in the announcements (sections 7.8, 8.6, 10.3, 10.4). ✅
- **The cloud providers document different validation levels**, as of October 2026, on pages carrying their own last-updated dates (section 8.4). That is a fact about documentation, and it is the most useful single fact in this guide for a procurement reviewer. ✅
- **The operational properties the refusal creates** - no exportable copy, ceremony-bound recovery, quorum-bound restore, firmware-version-bound validation - follow mechanically from the device's design and are visible in any competent implementation. ✅

### 14.2 What Is Vendor Positioning

Stated as a category, because the individual claims do not deserve repetition:

- **Superlatives about being first, best or most-deployed.** No independent basis was found this pass, and none is asserted here.
- **Peer-review scores and ranking badges on product pages.** These are aggregated user opinions, which is a different kind of evidence from a certificate, and they are not comparable across products with different review populations.
- **"Tamper-proof", "unbreakable", "military-grade".** Not terms any validation regime uses. The regimes use tamper-evident and tamper-responsive, and the difference between those two words is the whole of section 2.1.
- **Claims about internal design** - the specific sensor arrangement, the zeroization circuit, the key storage technology. These are the vendor's own description of its own product, are usually not in the security policy in that level of detail, and are exactly the class of claim the integrity convention marks ⚠.
- **Any statement that a validation level makes a deployment secure.** Section 4.8 and section 10.6 exist because this one is both ubiquitous and false in a way that matters.

### 14.3 The Shape of the Marketing

Worth describing, because recognising the shape is a skill. A payment-HSM product page in this pass carried, alongside the factual product description - remote configuration and monitoring, hot-swappable components, redundant ports, no-touch operation - a ranking badge and an aggregated peer-review score. The factual half is useful: those are checkable product properties, and "no-touch operation" in particular is a real operational claim about whether the device can be administered without physical access, which matters for section 9.

The other half is not evidence of anything, and its presence next to the factual half is the thing to notice. **A page that mixes specification with ranking is a page where the reader must separate them**, and the separation is easy: specifications can be tested, rankings cannot.

### 14.4 What a Buyer Can Actually Rely On

Four artifacts, in descending order of usefulness, and none of them is a web page:

1. **The certificate and the security policy.** The certificate gives module, version, level and the CAVP algorithm references; the policy gives the roles, services and CSPs. Together they are the only complete technical statement about the module.
2. **The payment approval listing**, when the use is payment. Separate regime, separate document (section 4.6).
3. **The vendor's operational documentation for the operations you will perform** - the ceremony procedure, the backup and restore procedure, the firmware compatibility matrix. These are checkable by reading them against your own runbook, and their gaps are the gaps you will discover in production.
4. **A reference you can telephone**, with a specific question: "show me your last restore test, and tell me who held the components." That question is unanswerable by marketing and answerable by an operator, which is what makes it useful.

**The closing judgement of this section, stated as an opinion because it is one:** the evidence available to a buyer is better than the evidence available for most enterprise software - a certificate, a policy, an approval and an audit log are more than most categories offer - and it is still not enough to rank two competent devices against each other. Which is why every ranking claim in this field, in either direction, should be read as marketing rather than as analysis.

## 15. The Claims Audit

Every factual claim of consequence in this guide, with its source quality and the date of the finding. **Highest-risk classes: the level-by-level wording (where the free text is superseded), the ISO mechanics (paywalled), the PCI requirement numbers (not read), and every cloud-provider statement (vendor documentation about its own service).**

| Claim | Status | Source (quality) | Date |
|---|---|---|---|
| FIPS 140-3 published **22 March 2019**; supersedes FIPS 140-2 (issued 25 May 2001) | ✅ | NIST FIPS 140-3 publication page (primary) | 2026 pass |
| FIPS 140-3 "became effective September 22, 2019", submissions accepted from September 2020 | ✅ | NIST CMVP, FIPS 140-3 Standards page, footer "Updated August 21, 2026" (primary) | 2026 pass |
| FIPS 140-3 "references ISO/IEC 19790"; testing per **ISO/IEC 24759** | ✅ | Same CMVP page (primary, direct quotation) | 2026 pass |
| ISO/IEC 19790 published August 2015 (2012 edition + 2015 corrigendum); ISO/IEC 24759 published March 2017 | ✅ | CMVP standards table (primary) | 2026 pass |
| NIST SP 800-140 (March 2020, Kim Schaffer) modifies the TE/VE evidence requirements of ISO/IEC 24759; parts 800-140A to F exist | ✅ | NIST SP 800-140 abstract (primary) | 2026 pass |
| FIPS 140-3 "provides four increasing, qualitative levels of security", with eleven named requirement areas | ✅ | FIPS 140-3 abstract (primary) | 2026 pass |
| The four-level physical-security wording quoted in section 4.2 | ✅ **as text** / ⚠ **as current normative text** - FIPS 140-2 is superseded; FIPS 140-3 delegates the normative requirements to ISO/IEC 19790 (paywalled) | FIPS 140-2 sections 1.1-1.4 (primary, free, superseded) | 2001/2002, read 2026 |
| "FIPS 140-2 modules can remain active for 5 years after validation or until **September 21, 2026**, when the FIPS 140-2 validations will be moved to the historical list" | ✅ | NIST CMVP, FIPS 140-3 Transition Effort page, footer "Updated April 13, 2026" (primary) | 2026 pass |
| "Even on the historical list, CMVP supports the purchase and use of these modules for existing systems" | ✅ | Same page (primary) | 2026 pass |
| CMVP recommends purchasers consider all modules on the Validated Modules Search Page regardless of 140-2 or 140-3 | ✅ | Same page (primary) | 2026 pass |
| **Whether the 21 September 2026 move completed, and how many certificates it affected** | ⚠ **NOT CONFIRMED** - the page read still used future tense | CMVP page (primary, incomplete) | 2026 pass |
| Common Criteria EAL2/EAL3/EAL4 appear in FIPS 140-2's operating-environment requirements at Levels 2/3/4 | ✅ | FIPS 140-2, operating environment requirements (primary) | 2026 pass |
| Any named HSM's Common Criteria evaluation status | ⚠ not read this pass - **not asserted** | - | 2026 pass |
| PCI PTS HSM **v5.0 published 18 May 2026**; "major revision ... from version 4.0 to version 5.0" | ✅ | PCI SSC blog by Alicia Malone, 18 May 2026; PTS HSM topic page (primary) | 2026 pass |
| v5.0: device security keys at least 128-bit effective strength, **TDES no longer permitted** for device security; EC-SDSA support; new modules for Key-Transfer Functionality, Remote Administration and HSM Solution Security; tenant key erasure and strict tenant isolation; vulnerability management in lifecycle modules; lab source code review | ✅ | Same announcement (primary, direct quotation) | 2026 pass |
| PCI KMO **v1.0 published 14 September 2026**; covers generation to destruction; **includes validation of remote and as-a-service HSM implementations**; aligned with PTS HSM v5; Listings "Coming Soon" | ✅ | PCI SSC blog, 14 September 2026; KMO standards page (primary) | 2026 pass |
| PCI KMO requirement clause numbers, counts or structure | ⚠ **NOT READ** - the PDFs would not fetch this pass; **no clause number is quoted anywhere in this guide** | - | 2026 pass |
| PCI DSS **v4.0.1** is the current version | ✅ | PCI SSC document library (primary, observed) | 2026 pass |
| PCI DSS and PCI PIN requirement numbers | ⚠ **deliberately not quoted** - see the KMIP guide, section 9.3 | - | 2026 pass |
| Dual control and split knowledge are genuine PCI PIN controls | ✅ (control area) | PCI SSC (primary) | 2026 pass |
| ISO 9564-1:2017, fourth edition 2017-11, ISO/TC 68/SC 2, cancels and replaces the 2011 third edition, incorporates Amd 1:2015 | ✅ | ISO preview front matter (primary) | 2026 pass |
| ISO 9564-1:2017 clause structure: 9.3 compact formats, 9.3.1-9.3.6, 9.4 extended, 9.4.2 Format 4, 9.5 translation restrictions, 9.6 journalizing, Annex A, 5.3, 6.1.1, 6.2, clause 7 | ✅ | ISO preview table of contents (primary) | 2026 pass |
| The **mechanics** of PIN block Formats 0, 1, 2, 3 and 4 (PAN exclusive-or, PAN-independence, AES 128-bit block) | ⚠ **RESTATED FROM SECONDARY SOURCES** - the normative text is paywalled and was not read | third-party restatement | 2026 pass |
| BDK, IPEK, no transactions processed on the IPEK, X9.24-1:2009 for IPEK and X9.24-1:2017 with **Initial Key (IK)** for an AES BDK | ✅ | AWS Payment Cryptography, *Industry terminology* (vendor primary for its own terms) | 2026 pass |
| "IPEK is a misnomer as this key can also be used to derive data encryption and mac keys" | ✅ | Same page (direct quotation) | 2026 pass |
| TR-31 binds attributes to key data and **"has been superseded by ANSI X9.143-2022"**; TR-34 is "an implementation of ANSI X9.24-2"; AWK historically 3DES, `TR31_P0_PIN_ENCRYPTION_KEY`; CVK `TR31_C0_CARD_VERIFICATION_KEY` | ✅ | Same page (vendor primary) | 2026 pass |
| The exact MAC algorithm, padding and field selection per card scheme | ⚠ **NOT VERIFIED** - scheme specifications are partner-gated | - | 2026 pass |
| AWS CloudHSM: general purpose, single tenant, "either FIPS 140-2 level-3 or FIPS 140-3 level-3 validated for clusters in FIPS mode"; data plane E2E encrypted and not visible to AWS; user management outside IAM roles; more customer responsibility; PKCS#11/JCE/CNG/KSP migration path | ✅ | AWS CloudHSM User Guide, *Introduction* (vendor primary) | 2026 pass |
| Azure Managed HSM: single tenant, "using FIPS 140-3 Level 3 validated HSMs"; fleet updated to 140-3 L3 firmware for Managed HSM and Key Vault Premium; separate customer-specific security domain per cluster; partitions migrate on hardware failure | ✅ | Microsoft Learn, *Managed HSM overview*, page "Last updated on 10/14/2026" (vendor primary) | 2026 pass |
| Google Cloud HSM: "cluster of FIPS 140-2 Level 3 certified HSMs"; Cloud KMS front end; Bare Metal Rack HSM for single-tenancy; asymmetric operations may incur noticeably greater latency | ✅ | Google Cloud, *Cloud HSM*, page "Last updated 2026-10-13 UTC" (vendor primary) | 2026 pass |
| The **documented** validation level differs across the three providers as of October 2026 | ✅ (observation about documentation) | The three pages above (vendor primary each) | 2026 pass |
| SoftHSM v2 stores all tokens and objects in the location given by `softhsm2.conf`, so "backup can thus be done as a regular file copy" | ✅ | `github.com/softhsm/SoftHSMv2` README (primary, direct quotation) | 2026 pass |
| DORA: Regulation (EU) 2022/2554 of 14 December 2022, in force **16 January 2023**, applies **17 January 2025** | ✅ | EUR-Lex CELEX 32022R2554; ESMA DORA page (primary) | 2026 pass |
| Thales payShield 10K product shape: remote configuration and monitoring, hot-swappable components, redundant ports, no-touch operation | ⚠ **VENDOR CLAIM ABOUT ITS OWN PRODUCT** - product shape only; the ranking badge and peer-review score on the same page are **not** repeated as fact | `cpl.thalesgroup.com` (vendor marketing) | 2026 pass |
| Any vendor's internal tamper design, sensor arrangement or zeroization mechanism | ❌ **NOT ACCEPTED AS A FINDING** - vendor description of its own internals; the level is what is evaluated, not the mechanism | vendor material | 2026 pass |
| Market share, ranking, customer counts or price for any HSM vendor or product | ❌ **no verified basis found - nothing is asserted; no such figure is printed anywhere in this guide** | - | 2026 pass |
| All Cymbal Bank figures, device counts, timelines and costs | ⚠ **explicitly fictional and illustrative** | this guide | 2026 pass |

### What Could Not Be Verified

Stated plainly, in the spirit of the rest of this guide - this is the honest residue, and these are the places to re-verify rather than to treat as settled.

1. **The mechanics of the ISO 9564 PIN block formats.** I read the standard's front matter and table of contents from the ISO preview, which fixes the **clause structure** - 9.3.1 to 9.3.6, 9.4.2, 9.5 - but the normative text is paywalled. Every description of how Format 0 combines the PIN with the account number, how Format 3 is PAN-independent and how Format 4 uses an AES block is **restated from secondary sources and marked as such in section 7.2**. Any implementation decision, and any audit response, should be taken from the standard itself. ⚠
2. **PCI KMO v1.0's requirement clauses.** The announcement is verified - publication date, scope, lifecycle coverage, the as-a-service clause, the alignment with PTS HSM v5.0 - but the two requirement documents on `docs-prv.pcisecuritystandards.org` would not fetch this pass. **No clause number from KMO appears anywhere in this guide**, and section 8.6 and 10.4 are written so that none is needed. ⚠
3. **Whether the FIPS 140-2 historical-list move completed on 21 September 2026, and how many certificates it affected.** The date is verified from the CMVP transition page, but the page read this pass still used the future tense ("will be moved"), and I found no post-move count. Anyone citing "the number of 140-2 certificates moved to historical" should get it from the Validated Modules Search Page on the day they cite it. ⚠
4. **Scheme-specific MAC algorithm, padding and field selection.** The card schemes specify these, the specifications are partner-gated, and I did not read them. Section 7.5 therefore states the *shape* of MAC generation (message, scheme-specified algorithm and key, key resident in the device) and explicitly does not name an algorithm per scheme. ⚠
5. **Any vendor's claim about its own internal design** - the tamper sensor arrangement, the zeroization circuit, the key storage technology, the "secure room" controls around a cloud region. These are unfalsifiable from outside, are usually not in the security policy at the claimed level of detail, and are recorded in this guide as vendor claims rather than as findings. The evaluated level is the checkable statement; the mechanism is not. ⚠
6. **The reliability of the search tooling this pass.** Several queries returned empty or narrow result sets, and the substantive work was done from primary URLs extracted directly - NIST CSRC pages, PCI SSC announcements, the AWS/Microsoft/Google documentation, the SoftHSM repository, EUR-Lex. **An empty search is not evidence of absence**, and no negative claim in this guide rests on one. ⚠

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

| Term | Definition |
|---|---|
| **HSM** | Hardware security module - a tamper-responsive device that generates, holds and uses key material inside a boundary the calling host cannot read across, and exports the result of an operation rather than the key (section 1.1). |
| **Tamper-responsive** | A design that *reacts* to physical intrusion by zeroizing its key store, as distinct from tamper-evident, which only leaves evidence afterwards (section 2.1). |
| **FIPS 140-3** | The current US/Canadian cryptographic-module validation standard, published 22 March 2019, effective 22 September 2019, with four qualitative levels and normative requirements delegated to ISO/IEC 19790 (section 4.1). |
| **PKCS#11** | The OASIS local cryptographic token API - slots, sessions, objects, attributes, mechanisms - through which an application uses keys in a device (section 5.2). |
| **Key ceremony** | The scripted, witnessed, dual-controlled physical procedure by which high-sensitivity key material is generated, loaded, backed up or destroyed at a device; it exists because the key cannot be exported (section 6). |
| **Quorum** | The M-of-N set of custodians whose components must be combined for a sensitive operation - typically a restore or a master-key load - to proceed (sections 6.4 and 9.3). |
| **DUKPT** | Derived unique key per transaction: a scheme in which each transaction's key is derived from a terminal's initial key and a counter, so that no per-transaction key is ever distributed and a compromised terminal yields only its own transactions (section 7.4). |
| **PIN block** | The PIN, padded and combined with account data, encrypted under a key so it can travel from entry to verification without existing in clear outside a security module; ISO 9564-1:2017 defines the formats and the translation restrictions (section 7.2). |
| **Partition** | The device-side tenancy unit: an isolated key store with its own security officer, access control and objects, carved out of a shared physical device (sections 1.2 and 3.1). |
| **CloudHSM** | A cloud-provider-operated HSM tenancy - single-tenant or partitioned - in which the refusal is still enforced by hardware but the authority around it sits with the provider (section 8). |

### 16.2 Cross-References

**Inside this repository - the guides whose boundaries this one respects:**

- [kmip_key_management_guide.md](kmip_key_management_guide.md) - **owns** the KMIP protocol (section 7), the key-lifecycle discipline (section 8), the ceremony as a governance and evidence event (section 8.5), the module-versus-system argument in principle (section 9.4), the PIN-standards carefulness (section 9.3) and the disconnected-key-estates finding (section 9.7). This guide defers to each by section and re-derives none of them.
- [cloud_providers_guide.md](cloud_providers_guide.md) - **owns** the cloud platform comparison; section 8 here gives only the HSM tenancy model and does not compare providers.
- [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) - **owns** the secrets-manager comparison and the PKCS#11 *seal* path, which is the other way an application puts a key behind a device boundary.
- [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) - **owns** the SWIFT HSM and CSP context.
- [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) - **owns** the NETS payment-HSM estate; section 7 here gives the mechanics that estate runs on.
- [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) and [../banking/financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md) - **own** the surrounding payment-flow and market-infrastructure context in which an in-path device sits.
- [container_certificates_guide.md](container_certificates_guide.md) and [oracle_database_guide.md](oracle_database_guide.md) - **own** the certificate and TDE key-material slices that may or may not be backed by a device.

**Primary sources, for the reader who wants to check any of this:** the NIST CMVP FIPS 140-3 Standards page and the FIPS 140-3 Transition Effort page; the FIPS 140-3 publication page and the FIPS 140-2 full text (for the level wording); NIST SP 800-140 and its parts; the PCI SSC announcements for PTS HSM v5.0 (18 May 2026) and KMO v1.0 (14 September 2026); ISO 9564-1:2017 (structure only, from the preview); the AWS Payment Cryptography terminology page, the AWS CloudHSM introduction, the Azure Managed HSM overview and the Google Cloud HSM page, each read on the date recorded in section 15; the SoftHSMv2 repository; and EUR-Lex for Regulation (EU) 2022/2554.

### 16.3 Closing Summary

**An HSM is a device that holds a key you cannot have, and every operational difficulty in this guide is that sentence being enforced.**

The findings worth carrying forward, in the order they will bite: **the certificate names a module, a version and a level, and none of those is your deployment** - so ask for the certificate and the security policy, check the version against the build, and check the payment approval separately, because FIPS 140-3, Common Criteria, PCI PTS HSM and PCI KMO are four different approvals answering four different questions. **The 21 September 2026 date moves FIPS 140-2 certificates to a historical list without revoking them**, so the question is not "is it still valid" but "is it current", and the answer differs between the three cloud providers as documented in October 2026. **The ceremony exists because the key cannot be exported**, which means the recovery plan is a physical procedure with a measurable duration, and an RTO written without measuring that procedure is a number nobody can meet. **The cloud model does not remove the refusal; it relocates the authority around it**, which is why the tenancy documentation, the erasure procedure and the audit rights are the three documents that decide whether the control argument survives.

And the thesis, tested and found to hold with two honest complications. The refusal is a security property, not a limitation - the key that will not leave, the rate that will not be exceeded, the enclosure that wipes itself, the roles that cannot be merged, the log that cannot be edited. But it is only a property when the process around it is funded: a ceremony nobody rehearses, a quorum nobody can assemble and a spare nobody keyed are all ways of buying the device and not the property. The second complication is that availability sometimes requires weakening the refusal deliberately - an HA group sharing one master key is the standard example - and the honest response is to write the weakening down, not to deny it.

Test the restore. Measure the ceremony. Read the certificate's version field. And keep asking the only question that survives every architecture in this guide: who holds the refusal, and can you assemble them on a Sunday?

**An HSM is the only component in the estate whose job is to be unhelpful.**
