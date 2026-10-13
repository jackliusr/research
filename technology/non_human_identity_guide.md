# Non-Human Identity: The Discipline Nobody Owns

*Every integration, every automation, every agent, every certificate and every API key is an identity — and unlike a person, none of them ever resigns, none of them changes team, none of them is caught by an offboarding checklist, and almost none of them has a named owner. This guide treats non-human identity as a **managed asset class** rather than as a collection of mechanisms: it is about the inventory problem, the lifecycle and the orphan, the ownership question, the identity-versus-secret distinction, governance and privilege, and the agent-as-non-human-identity explosion. It owns the **discipline**; the **mechanisms** — SPIFFE and SPIRE, mTLS, OIDC, secrets managers, certificate automation — are owned by the sibling guides it cross-references by name, and it does not re-explain a single one of them. Its thesis is one sentence: the hard problem is not giving a machine an identity, it is being able to list them.*

**Jack Liu Shurui, Solution Architect**

> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** October 2026  
> **Companion guides (technology/, same folder):** [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) — the parent discipline: the CISA five-pillar model, NIST SP 800-207 doctrine and the estate migration plan. Its §4.2 Identity-pillar row names *"service identities and workload identity (SPIFFE/SPIRE) for machine-to-machine"* in a single cell; that row is this guide's evidence that the *control* is named while non-human identity is not treated as a *subject*. [distributed_auth_guide.md](distributed_auth_guide.md) — the mechanisms: authn vs authz (§1), the auth mechanisms (§2), session vs token (§3), token validation and JWKS (§5), RBAC/ABAC/ReBAC (§6), policy engines and OPA (§7), **service-to-service including SPIFFE §8.1, SPIRE §8.2, mesh mTLS §8.3 and the S2S table §8.4** (§8), SSO (§9), and the microservices worked example (§11). This is the guide this one most risks duplicating, so it is cross-referenced for every mechanism rather than restated. [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) — the mainframe credential problems: §5 the identity bridge, §6 privileged access and the named identity, §10 the audit-side answer. [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) — §9 owns the shared database credential, named there as "a control-bypass, not a missing control". [container_certificates_guide.md](container_certificates_guide.md) — §8 rotation strategy, §9 cert-manager, §10 mTLS between containers, §12 monitoring certificate expiry. [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) — the secrets-management context and the vault boundary. [kmip_key_management_guide.md](kmip_key_management_guide.md) — the key-management discipline and the HSM/KMIP layer. [audit_as_code_guide.md](audit_as_code_guide.md), [data_governance_guide.md](data_governance_guide.md) and [api_governance_guide.md](api_governance_guide.md) — the audit, data-ownership and API-estate governance frames this guide cross-references for the regulated angle rather than re-deriving. [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) — §5 RBAC and admission, §8 secrets and supply chain, §10 certificate management and audit. [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) and [power_platform_cicd_guide.md](power_platform_cicd_guide.md) — §7 "Authentication: Service Principals, Application Users, and Secrets" is the citizen-developer case of the same problem. [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) — §6 "The Identity Question: AI User, Dynamic User, and Impersonation", §7 evidence and segregation of duties, §12 the operating model of ownership, review and retirement. [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) — a venture-forecast entry that carries the widely circulated non-human-to-human identity ratio; that guide already flags the figure as unverified and uncited, and this guide prints no ratio at all.  
> **Companion guides (technology/ai_llm/, prefix `ai_llm/`):** [enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) — §4.6 security in depth, §4.7 governance, §6.2 identity integration. [agent_experience_guide.md](ai_llm/agent_experience_guide.md) — the precedent for this guide's honesty about a newly-coined term, and §9–§10 on the security and liability of the material an agent consumes. [agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) — §2 the four postures, §9 banking and authority. [agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) — §2 the isolation spectrum and §10 tool-execution patterns, the containment half of the agent identity problem. [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) — the OWASP LLM Top 10, including LLM08 excessive agency.  
> **Companion guides (banking/, prefix `../banking/`):** [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md), [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md), [regtech_guide.md](../banking/regtech_guide.md), [risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md) and [risk_management_models_guide.md](../banking/risk_management_models_guide.md) — the risk, register and governance frames §12 cross-references for the accountability question.  
> **Method note:** this pass had live web access in **October 2026**. Verification used **direct page extraction of primary URLs** — the OWASP Non-Human Identities Top 10 project page and its 2025 list index, the Cloud Security Alliance whitepaper page, spiffe.io documentation, the AWS EKS IAM-roles-for-service-accounts page, Google Cloud Workload Identity Federation, Microsoft Entra Workload Identity Federation, the NIST CSRC page for SP 800-207, and the Kubernetes service-account documentation. `web_search` was **intermittently empty on this host**; where a search returned nothing usable that is recorded as a **tool limitation, not as evidence of absence**. Several vendor and analyst pages would not extract — the Gartner document `gartner.com/en/documents/6178123`, the `nhimg.org` page carrying the "who coined the term" claim, and the IETF datatracker pages for RFC 8705 and RFC 8693 — and each is recorded as ❌ rather than filled in from memory. **Every illustrative figure is labelled illustrative and attached to the fictional Cymbal Bank scenario only; Cymbal Bank is the sole bank persona and the sole institution in any worked example.** No vendor is ranked, sized, compared or recommended. **No machine-to-human identity ratio, credential-sprawl percentage, breach statistic, market size or growth rate is printed anywhere in this guide**; §15 records that omission explicitly.

---

**Purpose.** The enterprise has spent a decade governing *people*. It has a joiner-mover-leaver process, an identity provider, a directory, an access-review cadence, and an offboarding checklist that closes a person's accounts the day they leave. None of that machinery was built for a subject that is created by a system, has no manager, never takes leave, is never promoted, and is never removed by an exit interview. And yet the estate now runs on those subjects: service accounts, service principals, workload identities, integration credentials, API keys, machine certificates, OAuth clients, bots, RPA accounts and AI agents. The mechanisms for each have been written down — in this repository most of them live in [distributed_auth_guide.md](distributed_auth_guide.md), [container_certificates_guide.md](container_certificates_guide.md) and [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) — but the **discipline** that treats them as a single managed asset class has not. This guide writes that discipline. It is deliberately the *governance* half and not the *engineering* half: it explains what a non-human identity is as an asset, why the estate sprawls, why the orphan is the central failure case, why an identity and a secret are different objects, how to govern the estate, and what the arrival of the AI agent changes. It does **not** re-explain a mechanism. Where a mechanism is needed, the reader is sent, by a pointer to a named section, into the sibling that already owns it.

**Verification-markers convention.** ✅ = verified this pass against a named primary source, with the source dated and the retrieval date given as October 2026. ⚠ = vendor claim, approximate, or a single secondary source. ⚠-knowledge = well-established industry knowledge not re-verified this pass. ❌ = could not be verified at source; every ❌ is recorded honestly in §16. The consolidated ledger is the Claims Audit (§15).

**How this guide is organised.** §1 is the overview: the decoder, the honest provenance of the term, the thesis and the boundary, declared by name against the four adjacent guides. §2 is what a non-human identity is — the definition, the types a real estate contains, the human-versus-machine axis and the things that are *not* NHIs. §3 is the sprawl and the inventory problem. §4 is the lifecycle and the orphan — the central failure case. §5 is the identity-versus-secret distinction, the guide's conceptual core. §6 maps the trust mechanisms without restating them. §7 is the governance of an NHI estate. §8 is the agent as a non-human identity. §9 is the product categories, structurally. §10 is the assessment question set. §11 is the failure modes. §12 is the regulated and banking angle. §13 is the Cymbal Bank worked example. §14 is the anti-patterns. §15 is the claims audit. §16 records what could not be verified, holds the glossary and the cross-references, and closes. Cross-references follow repo convention: same-directory guides by plain filename, `ai_llm/...` for the AI sub-folder, `../banking/...` and `../management/...` otherwise.

---

**Table of Contents**

1. [The Overview: the Decoder, the Provenance, the Thesis and the Boundary](#1-the-overview-the-decoder-the-provenance-the-thesis-and-the-boundary)
   - 1.1 [The Decoder](#11-the-decoder)
   - 1.2 [The Provenance, Part One: the Application-Security Framing (OWASP, 2025)](#12-the-provenance-part-one-the-application-security-framing-owasp-2025)
   - 1.3 [The Provenance, Part Two: Machine Identity (CSA, 2023) and the Vendor Coinage](#13-the-provenance-part-two-machine-identity-csa-2023-and-the-vendor-coinage)
   - 1.4 [The Provenance, Part Three: the Market Label and What a Standards Body Has Not Said](#14-the-provenance-part-three-the-market-label-and-what-a-standards-body-has-not-said)
   - 1.5 [The Thesis](#15-the-thesis)
   - 1.6 [The Boundary — What This Guide Does Not Own](#16-the-boundary--what-this-guide-does-not-own)
2. [What a Non-Human Identity Is](#2-what-a-non-human-identity-is)
   - 2.1 [The Definition](#21-the-definition)
   - 2.2 [The Types a Real Estate Contains](#22-the-types-a-real-estate-contains)
   - 2.3 [The Human-Versus-Machine Axis](#23-the-human-versus-machine-axis)
   - 2.4 [What Is Not a Non-Human Identity](#24-what-is-not-a-non-human-identity)
3. [The Sprawl and the Inventory Problem](#3-the-sprawl-and-the-inventory-problem)
   - 3.1 [How the Estate Grows](#31-how-the-estate-grows)
   - 3.2 [The Asymmetry](#32-the-asymmetry)
   - 3.3 [Why Growth Outpaces the Controls](#33-why-growth-outpaces-the-controls)
   - 3.4 [The Central Finding — You Cannot Govern What You Cannot Enumerate](#34-the-central-finding--you-cannot-govern-what-you-cannot-enumerate)
4. [The Lifecycle and the Orphan](#4-the-lifecycle-and-the-orphan)
   - 4.1 [The Lifecycle](#41-the-lifecycle)
   - 4.2 [The Orphan](#42-the-orphan)
   - 4.3 [The Ownership Question](#43-the-ownership-question)
   - 4.4 [The Offboarding Framing, Dated](#44-the-offboarding-framing-dated)
5. [The Identity-Versus-Secret Distinction](#5-the-identity-versus-secret-distinction)
   - 5.1 [The Distinction](#51-the-distinction)
   - 5.2 [The Contrast Table](#52-the-contrast-table)
   - 5.3 [The Three Consequences](#53-the-three-consequences)
   - 5.4 [The Test to Apply](#54-the-test-to-apply)
6. [The Trust Mechanics, Mapped Not Restated](#6-the-trust-mechanics-mapped-not-restated)
   - 6.1 [The Mapping Table](#61-the-mapping-table)
   - 6.2 [What the Mechanism Gives the Inventory](#62-what-the-mechanism-gives-the-inventory)
7. [The Governance of an NHI Estate](#7-the-governance-of-an-nhi-estate)
   - 7.1 [The Ownership Model](#71-the-ownership-model)
   - 7.2 [The Policy — Registration Before Issuance](#72-the-policy--registration-before-issuance)
   - 7.3 [The Privilege Question](#73-the-privilege-question)
   - 7.4 [Segregation of Duties](#74-segregation-of-duties)
   - 7.5 [The Review Cadence](#75-the-review-cadence)
   - 7.6 [The Audit Question](#76-the-audit-question)
8. [The Agent as a Non-Human Identity](#8-the-agent-as-a-non-human-identity)
   - 8.1 [What an Agent Is, in Identity Terms](#81-what-an-agent-is-in-identity-terms)
   - 8.2 [The Four Assumptions It Breaks](#82-the-four-assumptions-it-breaks)
   - 8.3 [The Concrete Instance Already in This Repository](#83-the-concrete-instance-already-in-this-repository)
   - 8.4 [The Design Rule](#84-the-design-rule)
9. [The Product Categories, Structurally](#9-the-product-categories-structurally)
   - 9.1 [The Four Categories](#91-the-four-categories)
   - 9.2 [What Each Structurally Cannot Do](#92-what-each-structurally-cannot-do)
10. [The Assessment Question Set](#10-the-assessment-question-set)
    - 10.1 [The Five Required Questions](#101-the-five-required-questions)
    - 10.2 [The Additional Questions](#102-the-additional-questions)
    - 10.3 [The Response Shape and the Scoring](#103-the-response-shape-and-the-scoring)
11. [The Failure Modes](#11-the-failure-modes)
    - 11.1 [The Table](#111-the-table)
    - 11.2 [The Modes, in Prose](#112-the-modes-in-prose)
12. [The Regulated and Banking Angle](#12-the-regulated-and-banking-angle)
    - 12.1 [The Accountability Question](#121-the-accountability-question)
    - 12.2 [What Can and Cannot Be Asserted](#122-what-can-and-cannot-be-asserted)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
    - 13.1 [The Estate and the Inventory](#131-the-estate-and-the-inventory)
    - 13.2 [The One Decision the Inventory Changed](#132-the-one-decision-the-inventory-changed)
    - 13.3 [The Thesis, Restated](#133-the-thesis-restated)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified](#161-what-could-not-be-verified)
    - 16.2 [The Glossary](#162-the-glossary)
    - 16.3 [The Cross-References](#163-the-cross-references)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview: the Decoder, the Provenance, the Thesis and the Boundary

### 1.1 The Decoder

The vocabulary in this subject is unstable, and part of the honest case in §1.2–1.4 is that the instability is itself the finding. These are the definitions this guide uses. Where a definition is a standards body's, it is attributed and dated; where it is this guide's own framing, it is written as a definition and not dressed as a quotation.

| Term | What it means in this guide |
|---|---|
| **Non-human identity (NHI)** | A subject in a system that is not a person and that can be authenticated, authorised and audited: it can be granted access and its actions can be attributed to *it*. An identity is a **subject**, not a credential (§5). |
| **Workload identity** | The identity of a running piece of software — a service, a container, a job, a function. The scope of SPIFFE, whose documentation defines a *workload* as "a single piece of software, deployed with a particular configuration for a single purpose" ✅ (spiffe.io, *SPIFFE Concepts*, extracted October 2026). Mechanism owned by [distributed_auth_guide.md](distributed_auth_guide.md) §8.1. |
| **Service account** | A platform-level account that a service, job or automation runs as: the Kubernetes `ServiceAccount`, the Windows or Active Directory service account, the database login an application uses. Tests for whether Kubernetes itself treats it as a subject: `kubectl get serviceaccounts` lists them; the identity is `spec.serviceAccountName` ✅ (Kubernetes docs, extracted October 2026). |
| **Service principal** | The directory object that *is* an application's identity in a cloud directory: the Microsoft Entra "app registration"/service principal, the cloud IAM principal a workload authenticates as. Distinct from the secret that authenticates *to* it (§5). |
| **API key** | A bearer credential — a long string that grants whatever the server has decided that string means. It is a **credential**, and frequently has only a weak or implicit subject behind it (§5). |
| **Secret** | A credential: the material a subject presents to prove it is that subject — a password, an API key, a private key, a client secret. A secret is not an identity (§5). |
| **Certificate** | A signed statement binding a public key to a name — the carrier of an identity in X.509 form, and the credential that proves it in mTLS. Lifecycle owned by [container_certificates_guide.md](container_certificates_guide.md). |
| **Token** | A time-bounded assertion presented for access (a JWT, an OIDC ID token, an access token, a projected service-account token). Validated per [distributed_auth_guide.md](distributed_auth_guide.md) §5. |
| **Attestation** | Platform evidence that a workload *is* what it claims, checked by an issuing system before a credential is handed over — SPIRE's node and workload attestation. The property that lets an identity be established without a bootstrap secret ([distributed_auth_guide.md](distributed_auth_guide.md) §8.2). |
| **Rotation** | Replacing a credential on a schedule or event. Rotation acts on a **secret**; it does not change the **identity** (§5.3, i). |
| **Orphan** | An identity, or the credential of an identity, that has outlived its purpose, its owner, or the human who once used it — and that nothing in the enterprise still manages. The central failure case of this guide (§4.2). |
| **Privilege** | What an identity is permitted to do. The default state of an NHI's privilege is **over-privilege**, not least privilege (§7.3). |

### 1.2 The Provenance, Part One: the Application-Security Framing (OWASP, 2025)

The most formal recent articulation of the term this repository could verify is the **OWASP Non-Human Identities Top 10**, which its own project page describes as **"An OWASP incubator project"** ✅ (owasp.org/www-project-non-human-identities-top-10/, extracted October 2026). The **2025** list is published, with a documentation index at `/2025/` ✅. Two entries are load-bearing for this guide:

- **NHI1:2025 — Improper Offboarding**, which OWASP's own page defines verbatim as *"the inadequate deactivation or removal of non-human identities (NHIs) such as service accounts and access keys when they are no longer needed"*, adding that unmonitored and deprecated services may remain and their NHIs *"can be exploited by attackers to gain unauthorized access"* ✅.
- **NHI2:2025 — Secret Leakage**, covering the leakage of *"API keys, tokens, encryption keys, and certificates to unsanctioned data stores throughout the software development lifecycle"* — including hard-coded into source or stored in plain-text configuration ✅.

The framing sentence matters too, and it is worth quoting exactly because it shows what the *category* is being defined around: *"Production software environments are composed of a large number of applications which need to be identified, resulting in 'non-human identities' or NHI."* ✅ — that is, OWASP's definition is an *application-identity* definition. Note also that the list is broader than offboarding and leakage: it also names **NHI5:2025 Overprivileged NHI**, **NHI9:2025 NHI Reuse** and **NHI10:2025 Human Use of NHI** (the last describing exactly the shared-account failure this guide develops in §2.4 and §11) ✅. Two observations this guide will not overstate: first, **an application-security body's number-one non-human-identity risk is offboarding** — which is the orphan, and which supports making the orphan central here; and second, the OWASP project is a nonprofit foundation's project **staffed by vendor personnel** — its listed project leaders are from Astrix Security and Torch Security ✅ — which is consistent with, and does not by itself prove, the vendor-origin account in §1.3.

### 1.3 The Provenance, Part Two: Machine Identity (CSA, 2023) and the Vendor Coinage

Before "non-human identity", the standards-adjacent label was **machine identity**, and the term of art was **machine-to-machine**. The Cloud Security Alliance whitepaper **"Machine Identity in Cybersecurity and IAM"**, **released 09/19/2023**, defines machine identities as **"anything other than human"** and enumerates them as *"device identities, digital identities, and workload identities"* ✅ (cloudsecurityalliance.org/artifacts/machine-identity-in-cybersecurity-and-iam, extracted October 2026). So there are two labels in circulation, and the older one — machine identity, dated 2023 — is the one closer to a body of standards work. "Non-human identity" is the newer.

Where did the newer label come from? The honest answer is **a vendor**, and the source for that is weak in a way this guide must say out loud. A vendor-adjacent site, `nhimg.org`, claims that **Astrix Security "can be credited as the first to coin the term 'Non-Human Identity'"**, dating it to the **RSA Innovation Sandbox competition in May 2023**. That page **would not extract this pass (❌)** and the claim is a **single self-interested secondary source**, so it is recorded as **⚠** and not asserted. What *can* be verified is the commercial consequence: **Astrix Security now presents itself under a banner reading "Astrix is now part of Cisco"**, and its site (extracted October 2026) markets "Identity Security for AI Agents & NHIs" ✅. The category a vendor coined in 2023 has consolidated into a large security vendor — which is evidence about the *market*, and no evidence at all about the *discipline*.

### 1.4 The Provenance, Part Three: the Market Label and What a Standards Body Has Not Said

Two further provenance facts, stated with the same restraint.

First, **Gartner's IAM taxonomy** is *reported* to place NHI at the top of a hierarchy with machine identity as a subset, and *reported* to observe that "NHI" is often used as a buzzword by vendors offering workload-IAM capabilities. The primary Gartner document (`gartner.com/en/documents/6178123`) **did not extract this pass (❌)**, so this guide cites it only as **⚠ reported second-hand by vendors** — by an explainer the parent pass dated September 2, 2026, which names the Gartner document by URL — and puts the primary document in §16.

Second, and more important than either vendor claim: **no standards body this guide could verify has defined "non-human identity" as a term of art.** NIST SP 800-207 (August 2020) does not use the term; its published abstract speaks of moving defences to *"focus on users, assets, and resources"*, and of authentication and authorization *"both subject and device"* being *"discrete functions performed before a session to an enterprise resource is established"* ✅ (NIST CSRC, SP 800-207, DOI 10.6028/NIST.SP.800-207, extracted October 2026). NIST's account management control (SP 800-53 AC-2) is written around **accounts and users** — ⚠-knowledge, not re-verified this pass. This guide therefore makes a claim it will not extend: **it does not claim NIST, or any regulator, has defined non-human identity.**

**What the provenance establishes, stated plainly:** *non-human identity is a recently-emerged, vendor-driven framing that an application-security body (OWASP, 2025) and a cloud-security body (CSA, 2023, under the older label "machine identity") have begun to formalise — not a settled standard.* This guide presents itself as a **framing**, the way [agent_experience_guide.md](ai_llm/agent_experience_guide.md) found its own term newly coined in 2025 and said so rather than manufacturing a canon. A reader who prefers "machine identity" loses nothing by substituting it throughout; a reader who needs a standard to point at does not yet have one.

### 1.5 The Thesis

The thesis is one sentence, and it is deliberately narrower than "identity is important":

> **Giving a machine an identity is easy. Being able to list every identity a machine holds — and who owns it — is the hard problem, and almost no estate can do it.**

The first half is why the subject looks solved: every cloud hands out workload identities, every platform issues tokens, every vault stores secrets. The second half is why the subject is not solved: those identities are created by *whoever needed one*, in *whatever system required it*, under *no shared register*, and the only lifecycle event that reliably fires is *none*. §3 develops the inventory problem, §4 the orphan, §7 the governance, and §13 shows one decision that changes the moment a list exists. The guide's closing line returns to the same point.

A reader who wants the thesis in the negative — what happens if the discipline is not taken up — can hold one image: an estate whose every credential is freshly rotated, whose every workload has a short-lived SVID, whose every certificate auto-renews, and that still cannot answer the question "can you list every identity that can act on this system?". That estate has done the easy half perfectly and the hard half not at all, and nothing in the easy half will ever substitute for the hard one.

### 1.6 The Boundary — What This Guide Does Not Own

This guide owns the **discipline** of non-human identity as a managed asset class. It does **not** own any **mechanism**, and it re-explains none. The boundary is declared by name against four adjacent guides, and every mechanism a reader might expect here is deliberately pointed, not restated:

- **[zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) owns the CISA five-pillar model and NIST SP 800-207 doctrine.** This guide cites its §4.2 Identity-pillar row — which names *"service identities and workload identity (SPIFFE/SPIRE) for machine-to-machine"* in a single cell ✅ — only as evidence that the control is *named* while non-human identity is *not treated as a subject*. It does not restate the pillars or the doctrine.
- **[distributed_auth_guide.md](distributed_auth_guide.md) owns the authentication and authorisation mechanisms**: authn vs authz (§1), the mechanisms including OIDC, JWT, mTLS, WebAuthn, SAML and OAuth (§2), session vs token (§3), token validation and JWKS (§5), RBAC/ABAC/ReBAC (§6), policy engines and OPA (§7), **service-to-service including SPIFFE (§8.1), SPIRE (§8.2), mesh mTLS (§8.3) and the S2S table (§8.4)** — which already lists *static API keys as "the baseline to migrate away from"* and OAuth client-credentials with mTLS (RFC 8705) — (§8), SSO (§9) and the microservices worked example (§11). **This is the most likely duplicate of anything in this guide, and every mechanism in §6 points there.**
- **[zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) owns the mainframe credential problems**: §5 the identity bridge, §6 privileged access and vaulting under the named identity, and §10 the audit-side answer.
- **[zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §9 owns the shared database credential**, named there as *"a control-bypass, not a missing control"*, and §10 the shared transfer identity.

Beyond those four, this guide cross-references and does not duplicate: [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) (§5 RBAC and admission, §8 secrets and supply chain, §10 certificate management and audit); [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) and [power_platform_cicd_guide.md](power_platform_cicd_guide.md) (§7 "Authentication: Service Principals, Application Users, and Secrets"); the key and certificate material in [container_certificates_guide.md](container_certificates_guide.md) (§8 rotation strategy, §9 cert-manager, §10 mTLS between containers, §12 monitoring certificate expiry), [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) (the secrets-management context) and [kmip_key_management_guide.md](kmip_key_management_guide.md) (the key-management and HSM layer); the agent material in [ai_llm/enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md), [ai_llm/agent_experience_guide.md](ai_llm/agent_experience_guide.md), [ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md), [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md), [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) (§6 the identity question, §7 evidence and segregation of duties, §12 the operating model) and [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md); and the regulated, audit and governance material in [audit_as_code_guide.md](audit_as_code_guide.md), [data_governance_guide.md](data_governance_guide.md), [api_governance_guide.md](api_governance_guide.md), and in `../banking/` and `../management/`. Every filename above was verified to exist this pass before being cited.

---

## 2. What a Non-Human Identity Is

### 2.1 The Definition

A **non-human identity is a subject in a system, other than a person, that can be authenticated, granted permissions, and have its actions attributed to it.** Three words in that sentence carry the weight:

- **Subject** — it is the thing access is granted *to* and the thing an audit log names as the actor. An identity is a subject, not a credential (§5).
- **Authenticated** — it can prove it is itself, by some mechanism. The mechanism is not the identity; the identity is the thing the mechanism establishes.
- **Attributed** — when it acts, the record of the action names it. This is the property the shared account destroys (§11) and the property the orphan makes meaningless (§4).

Two boundaries of this definition are worth stating at once. First, it is deliberately **broader than "a service"**: an AI agent, a bot, an RPA account and a device certificate are all subjects by this definition, and the estate contains all of them (§2.2, §8). Second, it is deliberately **narrower than "a secret"**: a credential with no subject behind it is not an identity, it is an exposure (§2.4).

### 2.2 The Types a Real Estate Contains

A real estate does not contain one kind of non-human identity; it contains a dozen, created by different teams in different systems with different names for the same thing. The types below are the ones an inventory must be prepared to receive — and the reason a single tool rarely covers them all is that no single system creates them all.

| Type | Where it lives | Created by | The identifier it is known by |
|---|---|---|---|
| **Service accounts** | The OS, the database, the Kubernetes namespace, the application server | An administrator, a bootstrap script, a deployment template | An account name; on Kubernetes, `metadata.name` of a `ServiceAccount` |
| **Service principals / app registrations** | The cloud directory (e.g. an Entra app registration or service principal) | A developer or platform team, often in minutes | An application (client) ID in the directory |
| **Workload identities** | The orchestration platform or the identity fabric | The platform itself, from attestation or a binding | A SPIFFE ID, a federated principal, a role binding |
| **Device and machine certificates** | PKI, the device, the service host | A PKI or a certificate-automation controller | A certificate serial and subject/SAN |
| **API keys and integration credentials** | A SaaS tenant, an API gateway, a config store, a partner portal | A human in a web console, most often | An opaque key string, frequently with a label only |
| **OAuth clients** | An authorisation server or IdP | A developer registering an application | A client ID (with a client secret or a certificate) |
| **SSH keys** | A server's `authorized_keys`, a jump host, a CI runner | Whoever needed shell access | A public-key fingerprint |
| **Bots and chat integrations** | A collaboration platform | A user or admin, self-service | A bot/app ID and a token |
| **RPA accounts** | The RPA platform and the legacy systems it drives | An automation team | A user account, often a *human-shaped* login |
| **AI agents** | The agent platform, or a workflow engine | A developer, a business user, a vendor platform | An agent identity — or, too often, *a borrowed human or service account* (§8) |

The list is not a taxonomy to be memorised; it is a warning that **the same logical identity is known by a different name in every system that touches it**, which is exactly what defeats a naive inventory (§3).

### 2.3 The Human-Versus-Machine Axis

It is tempting to say non-human identities are "just like users that aren't people". They are not, and the differences are the whole reason the human governance machinery does not transfer. What actually differs:

- **No lifecycle event.** A person has joiner, mover and leaver events that fire from HR and the directory. An NHI has a creation event and, if it is lucky, a deletion. There is no HR system for a service account; the *only* reliable lifecycle trigger is the one a human chooses to build.
- **No MFA.** A person can be challenged with a second factor. An NHI that authenticates with a static secret has one factor by construction, and the "factor" is a string that sits in a config file, an environment variable, or a CI secret store.
- **No manager.** A person has someone accountable for them by organisational structure. An NHI is accountable to whoever created it — and that person may have left, changed team, or forgotten.
- **No departure.** A person leaving triggers offboarding. A machine does not resign; nothing fires when an NHI's purpose ends, because no system knows the purpose ended.
- **Created by a system, not a joiner.** A person's account is created by a provisioning process that answers to HR. An NHI is often created by *the thing that needed it* — a script, a template, a developer in a console — with no gate at all.

The consequence is stated once here and developed in §3 and §4: **every control the enterprise built for people depends on a lifecycle signal that non-human identities do not produce.**

The difference is easiest to hold as a single side-by-side, because it is the same row set the human governance model is built on and the same row set the NHI estate is missing:

| Lifecycle row | A person (human identity) | A non-human identity |
|---|---|---|
| **Joiner event** | Fires from HR; an account is provisioned through a gate | Does not exist; creation is self-service and unapproved (§3.1) |
| **Mover event** | Fires; entitlements are re-scoped when the role changes | Does not exist; access drifts with the integration, not the team |
| **Leaver event** | Fires; the person's accounts are closed | Does not exist; the identity outlives its creator (§4.2) |
| **Second factor** | MFA or a passkey is available and enforced | Usually a single static secret, by construction |
| **Accountable to** | A manager, by the org chart | The creator — who may have left, changed team, or forgotten |
| **Review trigger** | Periodic access recertification | None, by default |
| **Retirement trigger** | The leaver event | A human decision, if anyone still remembers the purpose |

Read down the right-hand column and the guide's problem is nearly visible: every mechanism of accountability that the left-hand column supplies for free is, on the right, either absent or would have to be built deliberately.

### 2.4 What Is Not a Non-Human Identity

The term is new enough that it is routinely over-applied. Four things get called non-human identities and are not:

- **A human's shared login.** When three operators use one account, or a team shares a service account password, the *subject* is still a person — several of them. This is a broken human identity, not an NHI. OWASP names the mirror image of it (a human using an NHI for manual work) as **NHI10:2025 Human Use of NHI** ✅, and names a team sharing an identity as **NHI9:2025 NHI Reuse** ✅. The diagnosis is different and so is the fix (§11).
- **A hardware key or a token device.** An MFA device is a factor a *person* presents. It is not a subject; it is not authenticated *as* anything on its own.
- **A plaintext secret with no subject behind it.** A password in a config file, an API key with no owner, a private key in a repository — these are **credential exposures**, and OWASP's NHI2:2025 Secret Leakage covers them ✅. But a secret is not an identity (§5). Treating a leaked key as "an NHI problem" hides the harder question: *which subject was that key the credential for, and does that subject still need to exist?*
- **A role or a permission.** A role is a bundle of permissions; a permission is a decision about what may be done. Neither is a subject. An NHI may *hold* roles and *have* permissions; it is not one. Confusing the two produces the inventory that lists permissions and believes it has listed identities.

---

## 3. The Sprawl and the Inventory Problem

### 3.1 How the Estate Grows

Non-human identities are not created in one place; they are created in dozens, by whoever needed one at the time. Every one of the following adds identities, and none of them asks a central authority first:

- **Every integration** — each API connection to a partner, a SaaS platform, a payment rail, a data provider — arrives with an API key, an OAuth client, or a service account.
- **Every automation** — every scheduled job, every CI/CD pipeline, every iPaaS flow — runs as *something*, and that something is an identity.
- **Every agent** — every AI agent, every bot, every RPA robot — needs authority to act, and that authority arrives as an identity or, worse, as a borrowed human or service account (§8).
- **Every platform migration** — a move to Kubernetes, a move to the cloud, a new data platform — mints new workload identities and leaves the old ones behind.
- **Every proof of concept** — a pilot that needed a service account "just to test" and was never decommissioned (§11, the fourth mode).

The estate is therefore a *diffuse creation surface*: the entities that create NHIs far exceed the entities that could govern them, and most creation happens with no approval step at all.

### 3.2 The Asymmetry

The single most important structural fact in this guide is an asymmetry between two activities:

> **Creating a non-human identity takes minutes and is unapproved by default. Enumerating, owning, reviewing and retiring one takes coordination, and is nobody's job by default.**

A developer can create a cloud service principal in a console, generate an API key in a SaaS tenant, or bind a workload identity to a Kubernetes service account, in minutes, usually without leaving their chair and usually without a ticket. There is no "joiner" process to satisfy, no manager to ask, no HR record to match against. The creation is *self-service* because that is what made the platform pleasant to build on. Meanwhile the counter-activity — knowing how many exist, who owns each, which are orphaned, which share a credential — requires that someone *chose* to build an inventory, and that the people who create identities *chose* to record them. Neither is the default.

### 3.3 Why Growth Outpaces the Controls

Four reasons the controls fall behind, none of them about carelessness:

- **The control was built for people.** The identity provider, the directory, the access review and the offboarding checklist all key off a lifecycle signal that NHIs do not emit (§2.3, §4.1). A control fed by the wrong signal does not scale down; it fails to apply.
- **The creation surface is not the governance surface.** Identities are created in cloud consoles, CI systems, SaaS tenants and code. Governance lives in the identity provider and the risk function. The two sets overlap only where someone wired them together, which is rarely.
- **The mechanisms are modern; the register is not.** A workload-identity fabric issues short-lived SVIDs the estate can trust ([distributed_auth_guide.md](distributed_auth_guide.md) §8.1–§8.2), but nothing about issuing an SVID tells the enterprise that the *workload* exists, who owns it, or when its purpose ended. Modern mechanisms automate the *credential*; they do not inventory the *subject* (§6.2).
- **Nobody owns the category.** The platform team owns the platform, the security team owns the controls, the application team owns the app — but the non-human identities that span them belong to no one's job description. This is the ownership question of §4.3 and §7.1, and it is the root of the problem rather than a symptom of it.

There is a temptation to read the four reasons above as a maturity gap — a control that simply has not been built yet. The more accurate reading is a **category gap**: the enterprise's identity governance was designed to answer questions about people, and non-human identity asks different questions. "Who is this person's manager?" has no answer for a service account. "When did they last change role?" has no answer for a workload identity. "Does their access still match their job?" has no answer for an integration whose job was defined by the code that needed it. The mechanisms of §6 are modern; the questions above are not answered by them, because no mechanism answers a question it was never built to ask. The answer has to be a register and a set of owners, which is what §4, §7 and §10 build.

### 3.4 The Central Finding — You Cannot Govern What You Cannot Enumerate

The finding is the thesis in problem form, and it is worth stating as bluntly as the estate deserves:

> **You cannot govern what you cannot enumerate. The first deliverable of a non-human identity programme is a list — not a tool, not a policy, not a vault.**

Everything in governance — ownership, least privilege, review, revocation, segregation of duties — is an operation on a *named set*. If the set is unknown, each of those operations degrades to a guess: you cannot assign an owner to an identity you have not found, cannot right-size privilege on a subject you cannot name, and cannot revoke what you do not know exists. The inventory is not a preliminary to the work; it *is* the work's precondition, and it is the one deliverable that no mechanism arrives with.

**On the numbers this section does not contain.** The literature on this subject is dense with ratios and percentages — how many machine identities there are relative to humans, how fast the estate grows, what share of credentials are unrotated. This guide prints **none of them**, because the most-circulated figure is uncited even in this repository's own source for it: [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) records the widely circulated non-human-to-human identity ratio, attributed to a venture guest essay, as **"⚠ unverified — no source given in the entry"**, and flags it as uncited in its claims audit. A ratio without a dated primary source is a number pretending to be a fact, and a discipline built on an invented number is worse than one built on none. §15 records this omission as an explicit row.
---

## 4. The Lifecycle and the Orphan

### 4.1 The Lifecycle

Every non-human identity has, in principle, a lifecycle with five states, and the enterprise manages between one and three of them.

1. **Creation** — the identity comes into existence and is granted permissions. This happens, per §3.1, in a diffuse creation surface, usually self-service, usually with no approval. It is the *only* state that reliably happens.
2. **Use** — the identity authenticates and acts. This is the state the mechanisms are built for (§6) and the state the estate is comfortable with.
3. **Rotation** — the credential is replaced. This is the state that secrets-management and certificate-automation tooling automates ([openbao_vs_vault_guide.md](openbao_vs_vault_guide.md), [container_certificates_guide.md](container_certificates_guide.md) §8–§9).
4. **Revocation** — the identity's access is withdrawn. This happens *reactively*, on incident, and almost never proactively.
5. **Retirement** — the identity is deleted because its purpose ended. **This is the state almost nothing manages.**

The reason is structural, and it is the same reason §2.3 gives: there is no lifecycle *signal* for states 4 and 5. A person's account is retired because a leaver event fires; an NHI's is retired only if a human decided to retire it, remembered that it existed, and found every place it was used. Rotation (state 3) is the state the *tooling* automates, and it is the state that feels like lifecycle management without being it — which is the root of the rotation-theatre failure in §11.

### 4.2 The Orphan

> **An orphan is a non-human identity, or the credential of one, that has outlived its purpose, its owner, or the human who once used it — and that nothing in the enterprise still manages.**

The orphan is the central failure case of this guide because it is the *expected* steady state, not an accident. Every NHI becomes an orphan unless a process prevents it, because the two events that would retire it — the purpose ending and the creator leaving — generate no signal. Varieties of orphan the estate should expect:

- **The departed contractor's credential.** A contractor is given a service account, an API key or an SSH key to do a job; the contract ends; the human is offboarded (a joiner-mover-leaver event *did* fire for the person), and the *identity they created* is not offboarded, because offboarding checks the person's accounts, not the accounts the person made. The credential still works.
- **The identity behind a retired system.** A decommission removes the system but not every identity that depended on it. The identity now points at something that no longer exists, or at something that still exists and still trusts it.
- **The proof-of-concept identity that graduated silently.** A pilot ran as a service account "just for the test"; the pilot became production; the identity was never formalised, owned, or scoped.
- **The integration with no partner left.** A connection to a third party that moved off the platform, leaving an API key and a trust relationship behind.
- **The credential whose owner changed team.** The person still works here, but stopped thinking about the identity years ago, and no review ever asked them.

**Why the orphan is worse than a missing control.** A missing control is a gap; an orphan is an *active subject* with access that nobody is watching and nobody will ever ask about. It does not appear in an access review, because reviews are run against a list, and the orphan is not on the list — that is what makes it an orphan. It is, in the terms the neighbouring guides use, a **control-bypass rather than a missing control** ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §9 names the shared database credential the same way): its existence makes the controls *above* it untrue, because the review, the entitlement report and the segregation-of-duties check all silently assume the list is complete.

### 4.3 The Ownership Question

Beside the orphan sits the question that makes it possible: **who owns a non-human identity?**

For a person, the answer is structural — a manager, an org chart, an HR record. For an NHI, the honest default is **nobody**. The identity was created by whoever needed it; that person may have moved on; the team that runs the system may not know the identity exists; the platform team owns the platform but not the accounts on it. So the identity drifts into a state where there is no one to *ask* whether it is still needed, because there was never a name against it. This is why §7.1 makes **naming an owner at creation** the first governance rule and not a later refinement: ownership is the *only* mechanism that generates a retirement signal, because the owner is the one human who can answer "is this still needed?" — and if no one is asked, the answer is never given.

There is a second-order effect that makes the ownership gap worse over time, not better. The longer an identity exists without an owner, the less anyone knows about why it exists: the creator has moved on, the integration has been rewritten, and the purpose is now inferred from what the identity can *reach* rather than from why it was granted that reach. The absence of ownership is therefore not a stable state that can be tidied up later at leisure — it actively degrades the information needed to tidy it. This is the strongest argument for naming an owner **at creation** (§7.1) rather than in a later review: creation is the one moment the purpose is known with certainty, and every subsequent moment is worse.

### 4.4 The Offboarding Framing, Dated

The industry framing now attaches a name to this failure. **OWASP NHI1:2025 — Improper Offboarding** is, per OWASP's own definition, *"the inadequate deactivation or removal of non-human identities (NHIs) such as service accounts and access keys when they are no longer needed"*, with the consequence that deprecated services and their NHIs *"can be exploited by attackers to gain unauthorized access to sensitive systems and data"* ✅ (owasp.org/www-project-non-human-identities-top-10, extracted October 2026). Two things about that entry deserve stating. First, it is **ranked first** in an application-security body's 2025 list — the orphan is not a footnote in the industry's own framing, it is the headline. And second, its diagnosis is exactly the one this guide reaches from the mechanism side: **offboarding is hard for non-human identities because the offboarding process was designed for humans and humans are offboarded at the *person* level, not at the *identity they created* level.** OWASP's own list also names the mirror failure, **NHI10:2025 Human Use of NHI** — humans doing manual work on a shared or machine identity — which is the reason an orphan is often *also* a shared account, and which the failure-modes table of §11 carries.

---

## 5. The Identity-Versus-Secret Distinction

### 5.1 The Distinction

The conceptual core of this guide is one sentence, and almost every machine-credential failure traces back to violating it:

> **An identity is a subject. A secret is a credential.** They are different objects, and the routine conflation of the two is the root cause of most non-human-identity failure.

A **subject** is the thing access is granted to and the thing an audit log names as the actor. A **credential** is the material a subject presents to prove it is that subject. The subject is the *who*; the credential is the *proof*. You can change the credential without touching the subject (rotate a key); you can remove the subject without touching any credential (delete the account, and every credential it had becomes inert because there is no longer a subject for it to authenticate *as*). The two operations fix different problems, and treating them as one operation is the fault.

The distinction is not pedantry; it has a diagnosis attached to it. A leaked secret is a *credential* problem, and its fix is credential-side: rotate, revoke, move to short-lived issuance. An orphan is a *subject* problem, and its fix is subject-side: find the identity, decide whether it should exist, and delete it if it should not. An estate that responds to every non-human-identity incident by rotating keys has been treating a subject problem as a credential problem, which is why the incidents recur — the subject is still there, holding the next credential.

### 5.2 The Contrast Table

| Dimension | **Identity (a subject)** | **Secret (a credential)** |
|---|---|---|
| **What it answers** | *Who is this, and what may it do?* | *Can this prove it is that who?* |
| **What it proves** | Nothing on its own — it is a decision point about access | That the presenter holds material matching the subject's registration |
| **What invents it** | A provisioning process, a directory, a platform — a governance act | A key-generation function, a console button, a script — a technical act |
| **What rotating it does** | Nothing — you do not rotate a subject; you change its permissions or retire it | Replaces the proof; the subject is unchanged (§5.3, i) |
| **What revoking it does** | Removes the actor: the subject can no longer authenticate as itself, and every credential it held becomes inert | Removes one path: the subject survives and can be issued a new credential |
| **What it is an instance of** | A governed asset with an owner and a lifecycle | A piece of sensitive material with a lifetime |
| **Where losing it hurts** | Everywhere — it is the actor in every log line | Wherever it was stored; a leak is an exposure, not an actor |
| **Named in a review as** | A line item requiring an owner (§7.1) | Rarely — reviews list identities, not keys |

### 5.3 The Three Consequences

The distinction does three jobs in this guide, each stated as a consequence.

**(i) A secrets-management tool is not a non-human identity inventory.** A vault ([openbao_vs_vault_guide.md](openbao_vs_vault_guide.md)) stores and issues *credentials*; a certificate controller ([container_certificates_guide.md](container_certificates_guide.md) §9) rotates *certificates*. Both are excellent at what they do — the credential side of the estate — and neither is an inventory of *subjects*. A vault can tell you which secrets it holds; it cannot tell you which identities exist, because an identity that authenticates to the vault is exactly the thing the vault *does not own*. The common error is to point at a vault or a secrets scanner and say "non-human identity is covered". The vault reduces credential exposure; it does not answer "list every identity that can act on this system" (§10.1). The two are complements, and only one of them is a list of subjects.

**(ii) An API key in a config file and a workload identity that can prove what it is are different objects.** An API key is *a credential with, at best, a weak subject behind it*: the server trusts a string, and the string's meaning is whatever the server decided. A workload identity — a SPIFFE ID, a federated principal, a bound service-account token — is *a subject that can prove what it is* from platform evidence ([distributed_auth_guide.md](distributed_auth_guide.md) §8.1–§8.2). The API key can be copied and used from anywhere, by anything; the workload identity is tied to *a workload the platform can attest*, so a copy is not portable to another workload. This is why [distributed_auth_guide.md](distributed_auth_guide.md) §8.4 lists *static API keys as "the baseline to migrate away from"*. The migration is a **subject-side** improvement: the estate moves from trusting a string to trusting a provable subject, and that is a different — and better-shaped — object than a shorter-lived key.

**(iii) "We rotate our keys" answers a different question from "can you list every identity that can act on this system."** The first is a *credential* assurance: the proof is fresh. The second is a *subject* assurance: the set of actors is known. An estate can rotate every key on schedule and still be unable to answer the second question, and hearing the first answer tender to the second is the most common conflation this guide exists to correct. **Rotation is not inventory.**

### 5.4 The Test to Apply

One test resolves the conflation under pressure. When someone says an identity has been "fixed", ask which object changed:

- **Did the credential change?** (rotated, shortened, moved to a vault) — then a *secrets* problem was addressed.
- **Did the subject change?** (owned, scoped, reviewed, retired) — then an *identity* problem was addressed.

If the answer is the first and the intent was the second, the fix has not landed; the subject is still there, and it will be handed the replacement credential in due course.

The test is worth applying to the *reporting* an estate produces, not only to individual fixes. A dashboard that counts credentials rotated, vault-managed secrets and expired certificates is reporting on the credential half of §5.2's table; a dashboard that counts identities listed, identities owned and orphans closed is reporting on the subject half. An estate that reports only the first has a metrics set that is structurally incapable of showing whether the discipline of §3.4 exists — which is why the reporting change, not the mechanism, is the one that moves the thesis.

---

## 6. The Trust Mechanics, Mapped Not Restated

### 6.1 The Mapping Table

This guide does not restate a single mechanism. It maps, for each mechanism the estate is likely to meet, **what it proves** and **which sibling guide owns the explanation**, with an explicit pointer. The purpose of mapping rather than restating is to make the *inventory* argument of §6.2 and to keep this guide out of the territory that [distributed_auth_guide.md](distributed_auth_guide.md), [container_certificates_guide.md](container_certificates_guide.md) and [kmip_key_management_guide.md](kmip_key_management_guide.md) already cover.

| Mechanism | What it proves | Who owns the explanation here |
|---|---|---|
| **mTLS (mutual TLS)** | That both peers hold a private key for a certificate the other trusts — a transport-layer identity, verified in the handshake | go to [distributed_auth_guide.md](distributed_auth_guide.md) §2.6; the mesh form (sidecar mTLS between workloads) is §8.3 |
| **Workload attestation** | That a workload *is* what its platform says it is (namespace, service account, node, labels), checked before a credential is issued — no bootstrap secret | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.2 |
| **SPIFFE / SPIRE SVIDs** | That a workload is the holder of a specific SPIFFE ID (`spiffe://trust-domain/path`), via a short-lived X.509-SVID or JWT-SVID ✅ (spiffe.io, extracted October 2026) | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.1 |
| **Cloud workload-identity federation — AWS (IRSA / EKS pod identity)** | That a pod's Kubernetes identity is trusted by an external system: EKS hosts a public OIDC discovery endpoint with the signing keys so systems such as IAM can validate `ProjectedServiceAccountToken` JWTs ✅ (AWS EKS docs, extracted October 2026) | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.4 (the S2S mechanism table); provider context in [cloud_providers_guide.md](cloud_providers_guide.md) §3 |
| **Cloud workload-identity federation — Google Cloud (WIF)** | That an external workload's federated identity (from an OIDC/SAML provider, X.509 cert, or another cloud) maps to a principal that may be granted IAM roles — "eliminates the maintenance and security burden associated with service account keys", following OAuth 2.0 token exchange ✅ (GCP docs, page last updated 2026-10-09) | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.4; provider context in [cloud_providers_guide.md](cloud_providers_guide.md) §5 |
| **Cloud workload-identity federation — Microsoft Entra (WIF and managed identities)** | That a workload gets Entra access tokens without managing secrets: an app registration or user-assigned managed identity trusts an external IdP's tokens, per the client-credentials flow with a federated credential ✅ (Microsoft Learn, page last updated 04/09/2025) | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.4; provider context in [cloud_providers_guide.md](cloud_providers_guide.md) §4 |
| **Kubernetes bound service-account tokens** | That a token was issued to a specific ServiceAccount and pod; obtained via the `TokenRequest` API and mounted by a projected volume, with bounded lifetime and automatic invalidation when the pod is deleted ✅ (Kubernetes docs, extracted October 2026) | go to [distributed_auth_guide.md](distributed_auth_guide.md) §5 (token validation); platform detail in [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) §5 |
| **OAuth client credentials + mTLS (RFC 8705)** | That a client is an authorised *client* (not a user), with the token optionally bound to a client certificate | go to [distributed_auth_guide.md](distributed_auth_guide.md) §8.4 |
| **HSM / KMS-signed certificates** | That a certificate was signed by a key held in a hardware or managed key store, so the signing authority is not itself a file on disk | go to [kmip_key_management_guide.md](kmip_key_management_guide.md) §8 (the key-management discipline); the certificate lifecycle is [container_certificates_guide.md](container_certificates_guide.md) §8–§9 |

### 6.2 What the Mechanism Gives the Inventory

Every mechanism in the table above shares one property, and it is the property that matters for this guide:

> **Attestation makes an identity provable. It does not make an identity enumerable.**

A workload-identity fabric (SPIFFE/SPIRE), a cloud federation, or a bound-token mechanism can *prove* that a workload is what it claims and can *issue* a short-lived credential without a stored secret. That is a large improvement over a static key — it shrinks the compromise window and removes the bootstrap secret from the estate, exactly as the sibling guides establish ([distributed_auth_guide.md](distributed_auth_guide.md) §8.2, §8.4). But none of those mechanisms, by itself, produces the thing §3.4 says is the precondition of governance: **a list of the subjects, their owners, and their purposes.** A SPIRE server will attest a workload that asks it for an SVID; it will not tell you that a workload you forgot about exists, and it has no view of the API key a team created in a SaaS tenant last year.

This is the reason §6 is a *mapping* section and not a mechanisms section. The mechanisms make each individual identity trustworthy; the **discipline** — the inventory, the owner, the review, the retirement — is what makes the *estate* governable, and no mechanism arrives carrying it.

---

## 7. The Governance of an NHI Estate

### 7.1 The Ownership Model

The first and load-bearing rule of governing an NHI estate is the answer to §4.3:

> **Every non-human identity is named to an owner at creation, or it is unowned — and an unowned identity is already an orphan in waiting.**

Ownership is not a label; it is the mechanism that generates the retirement signal. The owner is the one human who can answer the only question that retires an identity — *is this still needed?* — and if no one is named, the question is never asked. Three properties of a workable ownership model:

- **Named at creation, not assigned later.** If ownership is added in a review months afterwards, the identity has already spent its life unowned, and the owner may not be the right person — they are merely the first person asked.
- **A role, or a person, that can actually answer.** "The platform team" is not an owner; a named role in a named team is. The residual-risk convention the neighbouring guides use — a **named accountable role** rather than a team — is the model ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §12.6, §13.4).
- **Ownership transferable on team change.** Because the human mover event *does* fire, an ownership register can piggyback on it: when the owner's team changes, the identity's owner field is something the mover process can reassign — which is the one place the human lifecycle machinery can be made to serve the NHI estate.

### 7.2 The Policy — Registration Before Issuance

Ownership only works if it is attached at the moment of creation, and the way to attach it is a policy that sounds bureaucratic and is not:

> **Registration before issuance.** An identity is added to the register — with an owner, a purpose and a system — *before or as* it is issued, not after.

The policy exists because creation is the one moment a human is present and knows the purpose (§3.1–§3.2). Every hour after creation, that knowledge decays: the creator moves on, the purpose changes, the context is lost. "Registration before issuance" is therefore not red tape; it is *capturing the only information that will ever be cheaply available*. It is the non-human-identity analogue of the provisioning gate that human accounts get for free. Where a mechanism can enforce it — a platform that requires a labelled, owner-annotated service account, or a federation model that will only bind a declared identity — the policy can be partly mechanised; where it cannot, it is a process control, and the honest position is that a process control on a self-service creation surface is weak, which is why §7.5's review exists as the backstop.

### 7.3 The Privilege Question

The second governance rule is counter-intuitive and matters more than it sounds:

> **Over-privilege is the default state of a non-human identity, not an exception. Least privilege is a project, not a starting point.**

The reason is structural, and it is one of the differences in §2.3 made operational. A human who is granted too much finds it mildly inconvenient and asks for less, or a manager notices, or a review trims it — because a human *experiences* their own entitlement. A machine does not. It cannot ask for less, it does not complain, and it will use every permission it holds, because the integration was written to work, not to be minimal. When an integration is granted access, the grant is usually whatever made it work — "read and write to the bucket", "contributor on the subscription", "admin on the tenant for setup" — because the person wiring it up optimises for the integration succeeding, not for the privilege being minimal. So the estate drifts, not through malice, toward maximal privilege, and the *absence* of a complaint is read as evidence that the privilege is right.

The governance consequence is that least privilege for NHIs must be **engineered in, then verified**, not assumed. This is why OWASP names **NHI5:2025 Overprivileged NHI** as its own 2025 entry ✅ — the industry framing agrees that the over-privileged identity is a named risk, not an edge case. And it is why the machine holding excessive privilege is dangerous in a specific way: its compromise is not bounded by intent. A service account with "make it work" scope, once stolen, grants everything that scope allows, to whoever stole it.

The practical consequence is that an over-privilege review for NHIs cannot rely on the identity asking for less, because it will never ask. The review has to be driven from the other direction: from the **purpose** of the identity (what is this integration for?), to the **minimal scope** that purpose implies, to the **current grant** — and the gap between the last two is the finding. This is why §10.1's fourth question pairs "which are over-privileged" with a demand for a stated purpose: without the purpose there is no baseline, and every grant looks justified by the sole fact that the integration runs.

### 7.4 Segregation of Duties

Segregation of duties is a human-control concept: two people, two roles, so that no single person can both perform and conceal an act. A machine breaks it in a way the human control does not anticipate: **one identity can hold both sides of a control and there is no "second person" to notice.** The concrete forms:

- **The integration that both raises and approves.** A workflow identity that creates a record and has the authority to approve the same record performs the whole transaction, and the segregation the control diagram assumes exists between two humans collapses into one subject.
- **The agent that both drafts and commits.** [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §7.4 states the finding directly — segregation of duties does not survive an agent that can both raise and approve — and it is the clearest instance in this repository of a machine holding both sides.
- **The shared account the whole team uses.** When several humans act as one subject, the *reverse* failure appears: the subject holds everything, and no individual holds anything, so accountability is inverted (OWASP NHI9:2025 NHI Reuse ✅, and the shared-account mode in §11).

The design rule is the same in all three: **segregation for NHIs must be expressed as *distinct identities with distinct authority*, not as distinct people.** If a single identity can perform both sides of a control, the control does not exist, however the diagram is drawn.

### 7.5 The Review Cadence

A register decays. The review is what keeps it worth having, and its shape follows from the ownership model rather than from a compliance template:

- **Review against the owner, not the system.** The governing question is not "does this system still exist?" but "does the named owner still assert that this identity is needed?" — the question only an owner can answer.
- **Review the credential *and* the subject.** A review that asks "are your keys rotated?" is a credential review (§5.3, iii); the review this guide needs also asks "should this identity exist at all?"
- **Review the orphan class explicitly.** Because orphans are by definition off the list, the review must include a standing hunt for identities with no owner, no recent use, and no live purpose — the discovery activity that §10.1's third question drives.
- **Cadence tied to consequence, not to a uniform interval.** An identity that can move money and one that can read a public document do not need the same review frequency; both need to be *reviewed*, and the register should say which is which.

### 7.6 The Audit Question

What can an auditor actually be shown? This is the sharpest practical test of an NHI programme, and it is the one the shared-account case fails hardest (§11, mode 2; §12.1). The honest answer has three parts:

1. **The register.** A list of the identities, with owners and purposes — the deliverable §3.4 names as the precondition. An auditor can read it and test against it.
2. **The evidence of review.** Who asserted, when, that each identity is still needed — the output of §7.5.
3. **The honest gap.** Which identities the enterprise *cannot* enumerate, and why — the accounts in systems it cannot query, the SaaS tenants no one audited, the legacy estate ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md)). An auditor is testing the *disclosure* as much as the control; the unstated gap is the finding, exactly as the neighbouring guide puts it for the legacy estate ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §13.4).

The audit question of §12.1 then follows from part 2: when the register is complete and the shared-account problem is absent, each action in the log names a subject that can be traced to an owner. When it is not, the log records *that* something happened and cannot say *who decided it*.
---

## 8. The Agent as a Non-Human Identity

### 8.1 What an Agent Is, in Identity Terms

An **AI agent** is, for the purposes of this guide, a non-human identity that *behaves like a user*. That is the whole definition the identity discipline needs, and it is deliberately minimal: the guide does not attempt to define agency, autonomy or intelligence — those are owned elsewhere in the repository — it defines only what an agent is *as a subject in an access system*. It is a subject that authenticates, holds permissions, acts, and is (or should be) named in a log. Everything difficult about it follows from the gap between "behaves like a user" and "is not a user".

This section is short and its claims are deliberately bounded. It cross-references the repository's agent guides by real filename and asserts **no capability they do not establish**: the platform architecture in [ai_llm/enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) (§4.6 security, §4.7 governance, §6.2 identity integration), the design material in [ai_llm/agent_experience_guide.md](ai_llm/agent_experience_guide.md) (§9 security, §10 liability and authority — open questions there, not answers), the decision posture in [ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) (§2 the four postures, §9 banking and authority), the containment half in [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) (§2 isolation spectrum, §10 tool-execution patterns), and the risk catalogue in [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) (where excessive agency is a named LLM risk, §10). Where those guides mark a question open, this guide marks it open too and does not resolve it from memory.

### 8.2 The Four Assumptions It Breaks

An agent is a non-human identity that breaks four assumptions a human-shaped identity system relies on. Each is a real, structural break, not a maturity gap:

- **It can act continuously.** A person works a shift and sleeps; the human lifecycle assumes periods of non-activity during which access can be reviewed, revoked, or left alone. An agent can act at machine speed, continuously, and across more systems than any one person would touch — which means the *window* between "identity is created" and "identity has done something consequential" is tiny, and every review cadence built on human timescales is too slow.
- **It can be delegated authority.** An agent frequently does not authenticate *as itself*; it acts *on behalf of* a human principal, holding authority delegated from that person. This is the problem [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §6 is built around — the AI-user / dynamic-user / impersonation question — and it is the hardest part of the identity problem, because a delegated authority has two subjects (the agent and the principal) and only one of them is a person.
- **It can create further identities.** An agent with enough authority can provision service accounts, register applications, or bind workload identities — that is, it can *manufacture more non-human identities*, which is the §3.1 creation-surface problem given a new, automated engine. This is also why the *write* scope of an agent capable of changing identity configuration deserves a different approval standard than its read scope, the point [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §8.5 makes about an agent that can write to a system of record.
- **Its authority arrives as a delegation from a human and outlives the delegation.** The authority was granted because a person decided the agent should act for them. That person may change role, leave, or revoke the intent — and the agent's *standing* credential does not notice, because it is not tied to the delegating person's lifecycle any more than a service account is tied to its creator's (§4.2). The delegation is the *reason* for the authority; the authority is *not bound* to the delegation unless someone engineered it that way.

### 8.3 The Concrete Instance Already in This Repository

This is not a hypothetical future. The repository already contains the worked instance of the agent-as-NHI question, and this guide defers to it rather than re-deriving it:

- **[servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §6, "The Identity Question: AI User, Dynamic User, and Impersonation"** — the actual design question of what identity an agent runs as, across the documented run-as models, and §6.5's finding on shared accounts.
- **The same guide's §7, "The Evidence Question"**, including §7.4's direct finding that **segregation of duties does not survive an agent that can both raise and approve** — the §7.4 problem in this guide, in concrete form.
- **The same guide's §12, "The Operating Model: Ownership, Review, and Retirement"** — §12.2 the ownership model, §12.3 how it is reviewed, §12.5 how it is retired. This is the agent-specific instance of exactly the §7 governance model above, and it already exists; a reader who needs the agent case in full should read §12 there.

The reason to name these rather than restate them is the boundary rule of §1.6 and the honesty rule of §16: the repository has already done the agent-identity work in the guide that owns the platform, and duplicating it here would be the failure mode this guide's own §1.6 forbids.

### 8.4 The Design Rule

The rule this guide adds, and the only one it asserts for the agent case, is a direct application of §5:

> **Give the agent an identity, not a borrowed one, and bound its authority to the delegation that justifies it.**

That is two requirements, and both are subject-side. *An identity, not a borrowed one*: the agent should be a registered subject with a named owner, not a human's credentials and not a shared service account — because a borrowed identity destroys attribution (§12.1) and cannot be reviewed on its own terms. *Bounded to the delegation*: the agent's authority should be limited to what the delegating principal could have done and to the purpose the delegation was for, so that when the delegation ends — the person changes role, the project ends — the authority can be withdrawn as a subject operation rather than chased down as an unrotated credential. Everything about *how* to execute that rule technically lives in the agent, sandboxing and decision-policy guides above; this guide asserts only the identity-shaped requirement, because the identity-shaped requirement is the one that determines whether the agent can ever be reviewed, revoked or retired at all.

---

## 9. The Product Categories, Structurally

### 9.1 The Four Categories

This section is deliberately short, and deliberately structural. It describes the **categories** of product an enterprise will meet when it goes looking for NHI capability — what each category is architecturally good at, and what it structurally *cannot* do. It does **not** rank the categories, size any market, name a comparison, or recommend a vendor. Category archetypes are named only as the repository's other guides already name them (with the citation), and no vendor is compared to another.

| Category | What it is | Architecturally good at |
|---|---|---|
| **Secrets managers / vaults** | The store-and-issue layer for credentials — the discipline [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) covers, whose §1 is literally "Secrets-Management Context" | Centralising credential storage, controlling issuance, dynamic and short-lived credentials, audit of *access to secrets* |
| **Certificate lifecycle management** | The issue-and-rotate layer for X.509 material — the discipline [container_certificates_guide.md](container_certificates_guide.md) §8–§9 covers (rotation strategy, cert-manager) | Automated issuance and rotation of certificates, expiry monitoring ([container_certificates_guide.md](container_certificates_guide.md) §12), removal of long-lived certificate secrets |
| **Workload-identity platforms** | The attest-and-issue layer for workload identity — the SPIFFE/SPIRE family ([distributed_auth_guide.md](distributed_auth_guide.md) §8.1–§8.2), the cloud federation models ([distributed_auth_guide.md](distributed_auth_guide.md) §8.4), and the mesh that carries it ([distributed_auth_guide.md](distributed_auth_guide.md) §8.3) | Giving workloads provable, short-lived identities with no stored secret; removing static keys from the service-to-service path |
| **NHI governance / posture (emerging)** | The discovery-and-governance layer: find identities across systems, attach ownership, flag over-privilege and orphaning, drive review | *Addressing the inventory problem* — the one category whose subject is the estate of subjects rather than a mechanism |

### 9.2 What Each Structurally Cannot Do

The structural limits matter more than the capabilities, because they are where the category boundaries get mistaken for coverage:

- **A secrets manager cannot be an inventory of subjects.** It holds and issues *credentials*; the identities that authenticate *to* it are, by construction, outside it (§5.3, i). It reduces credential exposure; it does not enumerate the estate's subjects.
- **A certificate manager cannot see the identity behind the certificate at the level of ownership.** It will tell you a certificate exists, when it expires, and when it rotated; it will not tell you which human is accountable for the workload that holds it, or whether that workload should still exist.
- **A workload-identity platform cannot inventory what is outside it.** It can attest and issue for the workloads it knows; it cannot see the API keys created in SaaS tenants, the SSH keys on hosts, the service accounts in an untracked directory, or the identity created last week in a console it does not govern (§6.2).
- **Even an NHI-governance tool cannot govern what it cannot reach.** An inventory built by a tool is bounded by the connectors the tool has and the permissions it was granted — which is to say, the tool addresses the inventory problem *for the systems it can see*. The structural limit is the same one §3.4 names for the enterprise itself: enumeration is bounded by reach, and the honest deliverable includes the systems not reached (§7.6, part 3).

The structural conclusion: **the categories are complements, not substitutes.** A vault, a certificate manager and a workload-identity platform modernise three different mechanisms; none of them, alone or together, is the register §3.4 says is the precondition. That gap is precisely the space the fourth category names — and this guide takes no position on whether an enterprise should buy it, build it, or spread the work across existing registers.

One more structural observation, and it is architectural rather than commercial: these categories fail in the same *direction*, not in different ones. Each sees the slice of the estate it was connected to and reports on that slice, and a reader who takes any one slice as the whole gets a confident, partial picture. The structural answer is not to pick the category that sees the most, but to **treat every category's output as one source feeding a single register**, with the systems each category cannot see named as gaps. The register, not the tool, is the artefact that has to be complete — which is the same point §3.4 makes, and the reason the categories are complements rather than rivals.

---

## 10. The Assessment Question Set

### 10.1 The Five Required Questions

This is the guide's practical contribution: five questions a reader can carry to their own estate. Five are required, because between them they test the whole thesis — enumeration, ownership, orphaning, privilege, sharing and review. For each: what it asks, why the answer is usually unavailable, and what a usable answer would look like.

**Q1. Can you list every non-human identity that can act on this system?**
- *What it asks:* enumeration — the precondition of §3.4, for one system.
- *Why it is usually unavailable:* identities are created across many systems without a shared register (§3.1); the list, if it exists, lives in the ticket system, the cloud console, the IdP and someone's memory, and no single query joins them. The most common honest answer is "we can list the ones we know about".
- *What a usable answer looks like:* a dated list, per system, of every subject that can authenticate to it — service accounts, principals, workload identities, keys, certificates, tokens — with the source system each was drawn from and the systems *not* yet drawn from, named as gaps.

**Q2. Can you say who owns each of them?**
- *What it asks:* the ownership model of §7.1, tested per identity.
- *Why it is usually unavailable:* ownership was never captured at creation (§7.1, §7.2), so most entries have no owner or a stale one; "the platform team" appears where a named role should be.
- *What a usable answer looks like:* a named accountable role against each identity, with an owner-coverage figure the enterprise is willing to state (e.g. "of the identities we can list, this share has a current named owner"), and the unowned remainder carried explicitly as work, not hidden in a total.

**Q3. Which are orphaned?**
- *What it asks:* the §4.2 failure case, and the question OWASP NHI1:2025 makes the industry's first ✅.
- *Why it is usually unavailable:* orphans are by definition off the list (§4.2) — you find them by hunting, not by reviewing, and hunting requires the inventory of Q1 to be a starting point rather than an answer. The tell-tale signals — no owner, no recent use, purpose unknown to any current team — are not collected by default.
- *What a usable answer looks like:* an active, dated orphan-hunt result: identities with no owner, no use in a stated window, and no team willing to claim the purpose — paired with a disposition (retire, adopt an owner, or accept in writing with a review date).

**Q4. Which are over-privileged?**
- *What it asks:* the privilege question of §7.3, against the default state it describes.
- *Why it is usually unavailable:* "over-privileged" requires knowing what an identity *needs*, and the recorded grant is "whatever made the integration work" (§7.3). Without a purpose statement per identity, there is no baseline to compare the grant against, so every grant looks justified by the fact that it works.
- *What a usable answer looks like:* for the identities that can act on high-consequence systems, a stated purpose and the minimal scope that would satisfy it, with the current grant measured against it — and the over-scope named, not implied.

**Q5. Which share a credential?**
- *What it asks:* the shared-account failure (§2.4, §11, mode 2) — the one that destroys attribution.
- *Why it is usually unavailable:* shared credentials are invisible in a per-account view, because the *account* is one subject and only the *usage pattern* reveals that several actors use it; and a legible-looking estate will still contain a team login, a shared service account, or a machine identity driven by humans.
- *What a usable answer looks like:* a list of identities where the actor is ambiguous — a shared team account, a service account used interactively, a machine identity whose usage cannot be attributed to one process — with a plan to split each into distinct identities (§7.4).

### 10.2 The Additional Questions

Beyond the five, four questions that sharpen the same ground:

- **Which have ever been reviewed, and when?** (Tests §7.5; the answer is usually "never", or "on creation only".)
- **Which were created without approval, and how would you know?** (Tests §3.2; if the creation surface has no gate, the honest answer is "all of them, and we cannot tell".)
- **Which exist in systems you cannot query?** (Tests reach and the §7.6 gap; this is where the legacy and mainframe estates appear, cross-referenced to [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) and [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) rather than re-derived.)
- **Which non-human identity can create another non-human identity?** (Tests the agent/automation creation engine of §8.2, and the write-scope question [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §8.5 raises.)
- **Which identities are reachable from outside the estate?** (Tests reach of exposure: an identity whose credential is held by a third party, a partner or a SaaS tenant is reachable by someone the enterprise does not manage — the same external-reachability signal the legacy-estate guide uses to rank its priorities, [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §11.2.)
- **Which identities have a credential that does not expire?** (Tests the long-lived-credential condition OWASP names as **NHI7:2025 Long-Lived Secrets** ✅ — a credential with no end date is one that a compromise can use without a time limit.)

### 10.3 The Response Shape and the Scoring

The questions are designed to be answered *honestly*, and the honesty is easier to reach if the expected shape of an answer is fixed in advance.

- **A usable answer names its gaps.** "We can list subjects in these nine systems; we cannot yet enumerate these four; here is the plan." A total presented without its gaps is not an answer, it is a confidence.
- **A usable answer is dated.** An inventory is a state, not a truth; it is true as of a date and decays (§7.5).
- **A usable answer is qualified by confidence.** "Confirmed", "reported", "unknown" are three different statuses and should be shown as such, in the manner of the verification marks this repository uses.
- **Score the estate, not the tool.** A simple response-shape scoring — for each of Q1–Q5, is the answer (a) full and dated, (b) partial with named gaps, or (c) unavailable — produces a profile that is more useful than a single number, because it shows *which* dimension is the binding constraint. An estate that can enumerate but cannot name owners (Q1 yes, Q2 no) has a governance problem, not a discovery problem; an estate that can do neither has §3.4's precondition unmet, and no amount of mechanism will help until it is met.

The profile is easiest to read as a grid, one row per required question and one column per answer quality:

| Dimension | Full and dated | Partial, with named gaps | Unavailable |
|---|---|---|---|
| **Q1 Enumeration** | A dated list per system, with sources named | Known subjects listed; unreached systems named as gaps | No list; "we can say we have some" |
| **Q2 Ownership** | Every identity has a current named role | Coverage figure stated; the remainder carried as work | "The platform team" or blank |
| **Q3 Orphaning** | A dated orphan hunt with dispositions | Signals collected, dispositions outstanding | No hunt; orphans found only by accident |
| **Q4 Over-privilege** | Purpose and minimal scope stated per high-consequence identity | Purpose known for some; grants unmeasured | Grant is "whatever made it work" |
| **Q5 Shared credentials** | Ambiguous-actor identities listed and split | Some found; usage not attributed | Not measured; attribution assumed |

Read the grid down the right-hand column and the estate's shape is visible without a single invented number: an estate whose gaps cluster in Q1 has a discovery problem; one whose gaps cluster in Q2, Q3 and Q5 has a governance problem — and the second is the more common, because discovery can be bought and governance cannot.

---

## 11. The Failure Modes

### 11.1 The Table

Each failure mode is stated as **symptom → cause → guardrail**, so a review can use it as a checklist. Every guardrail points at the section that develops the fix. The five modes the task of this guide requires are the first five; the remainder are the ones the estate meets most often beside them.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **The orphaned credential of a departed contractor** | A service account, API key or SSH key still works months after the contractor's engagement ended and the person was offboarded | Offboarding fires at the *person* level and checks the person's accounts, not the identities the person created; the created identity has no owner and no retirement signal (§4.2, §4.4) | Name an owner at creation (§7.1–§7.2); hunt the orphan class explicitly in review (§7.5); OWASP NHI1:2025 is the dated framing (§4.4) |
| **2** | **The shared service account that makes an audit trail meaningless** | Several people or processes act as one subject; the log records the account, and the account is not a person | One identity reused across actors (OWASP NHI9:2025 ✅), or a machine identity driven manually (OWASP NHI10:2025 ✅) | Split into distinct identities per actor/process (§7.4); treat human use of a machine identity as a finding, not a convenience |
| **3** | **Rotation theatre** | Every inventoried secret rotates on schedule; the uninventoried estate grows; incidents keep happening | "We rotate our keys" applied as the whole answer to "list every identity that can act" — a credential assurance substituted for a subject assurance (§5.3, iii) | Pair every rotation program with an enumeration program (§3.4); measure progress by subjects listed and owned, not only by credentials rotated |
| **4** | **The proof-of-concept identity now running in production** | A service account created to test a pilot is still in production, unowned and over-scoped | A pilot's identity was never formalised when the pilot graduated; creation was self-service and unapproved (§3.1, §3.2) | A graduation gate: promoting a pilot to production re-registers its identities with an owner and a minimal scope (§7.1–§7.3) |
| **5** | **The agent granted a standing credential** | An AI agent holds a long-lived credential or a shared account, with authority that outlives the human delegation that justified it | The agent was given a borrowed identity rather than its own, and a standing secret rather than delegation-bound authority (§8.2, §8.4) | Give the agent its own identity and bound authority to the delegation; see [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §6, §12 |
| **6** | **The inventory built once** | A one-off discovery produces a list; the list is stale within a quarter; the estate believes it is covered | Creation continues after the snapshot; nothing re-runs the discovery or feeds new creations into the register (§7.2, §7.5) | Make registration part of creation (§7.2) and review a standing activity (§7.5); treat the register as a feed, not a report |
| **7** | **The vault mistaken for coverage** | A secrets-manager deployment is presented as evidence that non-human identity is handled | A credential store is read as a subject inventory (§5.3, i; §9.2) | Keep the two separate in reporting: secrets management *and* identity inventory are both reported, as two different statuses |
| **8** | **The over-privileged integration accepted as required** | An integration holds far more access than its function needs; nobody questions it because it works | Over-privilege is the default (§7.3); no purpose statement exists to measure the grant against | Require a purpose and a minimal scope per identity (§7.3; §10.1 Q4); OWASP NHI5:2025 Overprivileged NHI is the dated framing ✅ |

### 11.2 The Modes, in Prose

The table is a checklist; three of the modes deserve a paragraph, because their causes are the guide's argument compressed.

**On the orphaned credential (mode 1).** This mode is the guide's chapter-1 failure case, and its cause is worth restating in full because it is *not* negligence: the enterprise's offboarding process is a good process that was built for a different subject. It asks "does this person still work here?" and closes the *person's* accounts. It does not ask "what did this person create?", because until non-human identity is treated as a class, nothing records that a given human ever created a given machine identity. The fix is therefore not a better offboarding checklist; it is the ownership and registration model of §7.1–§7.2, which is the only thing that makes "what did this person own?" an answerable question at offboarding time.

**On the shared service account (mode 2).** This mode is the one that quietly voids the enterprise's audit trail, and it is the reason §12.1 can say the log records *that* something happened but not *who decided it*. A shared account is legible — it exists on the list, it has a name, it is "managed" in the sense that someone rotates its password — and it is exactly the identity a per-account control cannot fix, because the *account* is fine and the *usage* is the problem. It is also the failure OWASP confronts twice, from both directions: reuse across actors (NHI9:2025) and human manual use (NHI10:2025) ✅. The guardrail is not a control on the account; it is the splitting of the account into distinct identities (§7.4), which is a governance act, not a technical one.

**On rotation theatre (mode 3).** This is the mode that separates a mature-looking programme from a governable one. Rotation is visible, automatable, reportable and reassuring; enumeration is slow, unglamorous and often reveals that the estate is larger than anyone said. When a programme drifts toward the reassuring half, the estate ends up in the state the thesis names: machinery that proves the *credentials it knows about* are fresh, over a set of *subjects it cannot list*. The guardrail in the table — measure progress in subjects listed and owned, not only in credentials rotated — is the operational form of §5.3 (iii), and it is the single most useful reporting change an NHI programme can make.

**On the proof-of-concept identity (mode 4).** This mode is the guide's argument about *creation without registration* (mode 6) wearing a pilot's clothes. Nothing about the pilot was wrong; the failure is that the moment a pilot graduates to production is precisely the moment its identities stop being disposable and start being infrastructure — and nothing in the graduation changes how the identity was registered, scoped, or owned, because the identity was created to make an experiment work, not to be least-privilege, and no separate event re-scoped it. The guardrail is a **graduation gate**: promoting a pilot to production is a governance event that re-registers the pilot's identities with an owner and a minimal scope, so that the promotion that makes them infrastructure is the same act that makes them governed (§7.1–§7.3).

**On the agent granted a standing credential (mode 5).** This mode is the newest and the most consequential, because the agent is the §3.1 creation surface given an automated engine: an agent that can provision a service account can grow the unlisted estate faster than any team of developers. Its root is the §5 conflation seen from the identity side — the agent was given a *standing* credential (a secret, held indefinitely) rather than a *delegation-bounded* identity, which makes its authority outlive the human decision that justified it (§8.2). The guardrail is the §8.4 rule, and it is a governance rule rather than a mechanism: the agent gets its own registered, owned identity, and its authority is bounded to the delegation, so that when the delegation ends the authority can be withdrawn as a subject operation instead of chased down as an unrotated key. The concrete instance already exists in this repository ([servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §6, §12), which is where the agent case is developed in full.

---

## 12. The Regulated and Banking Angle

### 12.1 The Accountability Question

The regulated interest in non-human identity is not, in the first instance, a compliance question. It is an **accountability and evidence** question, and it is easiest to state as a scene:

> When a shared machine identity performs an action, the audit log records **that the action happened**. It does not record **who decided it should**. The action is attributed to a subject — and if that subject is a shared service account used by four people and a scheduler, the attribution names none of them.

That is the whole regulated angle, and it is why modes 1 and 2 of §11 matter most to a bank. A regulator, an auditor or an incident review asks a question that is fundamentally about **attribution**: *who is accountable for this action?* A per-person identity answers it. A shared machine identity, or an orphaned one nobody owns, does not — the log line is true and useless at the same time, because it names a subject that is not a decider. The shared database credential that [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §9 calls *"a control-bypass, not a missing control"* is the same problem seen from the data side: the control above it (the application's role logic, the report's masking, the audit trail) is silently untrue because the shared credential lets actors past it.

Three consequences for the regulated reader, all of them accountability-shaped rather than rule-shaped:

- **Segregation of duties depends on distinct subjects.** The way a control is expressed to a supervisor or an auditor is that no one actor can do both halves of a sensitive transaction. Machine identities collapse that if one identity holds both halves (§7.4) — and the repository already contains the direct finding for agents ([servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §7.4).
- **Evidence quality depends on attribution.** An audit trail is only as good as the actor it names. The governance frame for the *evidence* — what a system of record must capture, and how review is evidenced — is owned by the repository's audit and data-governance material, not re-derived here: [audit_as_code_guide.md](audit_as_code_guide.md) (evidence management, §8; the maturity model, §11) and [data_governance_guide.md](data_governance_guide.md) (§4 governance organisation and operating model, §10 the banking and regulatory context).
- **Residual exposure is an owned register item, not an omission.** Where an identity cannot be scoped, owned or retired — the accounts in an estate the enterprise cannot fully query — the honest treatment is the one the neighbouring guides use: name it, give it a compensating control with its limits stated, a **named accountable role**, a review date, and a **written acceptance** ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §12.6). The register frames this guide cross-references: [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md), [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md) and [risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md), with the governance-reporting framing in [regtech_guide.md](../banking/regtech_guide.md) and [risk_management_models_guide.md](../banking/risk_management_models_guide.md).

### 12.2 What Can and Cannot Be Asserted

This guide asserts **no jurisdiction's rule**. It does not claim, and has not verified a dated source for, any specific obligation under MAS, DORA, PSD2, SWIFT, Basel, SOX or any other regime. The reason is the reason §1.4 gives about the term itself: a compliance claim without a dated, named source is exactly the kind of fabrication this repository's conventions forbid. What this guide *does* assert is the accountability question in the abstract — and the abstract form is genuinely the safer one to reason with, because the accountability requirement is stable across regimes while any particular rule's wording and applicability is not.

The practical position, stated plainly: **treat non-human identity as an evidence and accountability control, and let the compliance mapping be done against the actual, current rule text by the function that owns it.** The repository has already done that mapping for its own regulated material in the sibling guides' established ledgers ([zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §8.2 for the regulatory frame; [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) §13 for the regulated OpenShift angle), and this guide cross-references those rather than restating a mandate it has not verified.

There is a second reason to leave the compliance mapping with the function that owns it: the non-human identity population is **younger than the rulebook**. Most of the identities in a modern estate — cloud workload identities, federated principals, agent identities — did not exist in the form they take now when many operative rules were written, so a direct mapping is frequently a stretch rather than a lookup. Framing the obligation as accountability and evidence (what §12.1 does) is both more honest and more durable: it is the property a regulator is actually testing for, and it survives the next re-write of a rule that never mentioned machines.

---

## 13. The Cymbal Bank Worked Example

*Fictional and explicitly illustrative. Cymbal Bank is the sole bank persona and the only institution in this example. Every figure below is illustrative; no real institution, system, vendor or count is described.*

### 13.1 The Estate and the Inventory

Cymbal Bank has the estate most institutions have, and it had never listed it as one thing. The identities it lives on were created across a dozen systems by a dozen teams, and the only place any of them were "recorded" was the system that issued them. So Cymbal ran the assessment question set of §10, not as a compliance exercise but to answer one question: *what can act on our systems?* The discovery is best described by what it found, because the findings are the failures of §11.

- **Sprawl, as expected.** Identities were found in the cloud tenant, the Kubernetes clusters, a SaaS integration estate, the on-premise service accounts, the database logins, the CI/CD pipelines, and a set of API keys in the connector estate ([api_governance_guide.md](api_governance_guide.md) covers the API-estate governance frame Cymbal applied). The registry that was supposed to be authoritative for all of them was authoritative for none of them in full: it held the identities the last migration had carried over, and nothing created since.
- **Orphans, in quantity.** The clearest group was credential material created by, or for, people who had since left — including contractor engagements that ended with the *person* offboarded and the *identity they created* untouched (mode 1). A second group were the identities of two systems that had been decommissioned, whose service accounts still existed and were still trusted by a system that had not been decommissioned (§4.2).
- **Shared credentials, the ones that would matter at audit.** Two service accounts had been used interactively by operations teams — a human-use-of-NHI pattern (mode 2) — and one database credential was shared between an application, a reporting job, and an analyst tool, the exact shared-credential shape [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §9 names as a control-bypass rather than a missing control.
- **Proof-of-concept lineages.** Several identities traced to pilots that had "become production" without ever being re-registered (mode 4), and were consequently over-scoped, because a pilot is scoped to *work*, not to *least privilege* (§7.3).
- **No owners.** For the majority of the identities found, no one interviewed could say who owned them, which is the answer §4.3 predicts and the finding that made the rest of the programme possible: once Cymbal knew ownership was absent, it stopped treating the problem as a discovery problem and rebuilt the register as an ownership register with a discovery feed (§7.1, §7.5).

None of these findings required a tool Cymbal did not have, and none of them required a policy that did not already exist on paper. They required the list. That is the point of the example.

Cymbal also recorded the limits of its own inventory, in the shape §7.6 part 3 requires. Two systems — a mainframe-adjacent estate ([zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md)) and a set of legacy platform accounts ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md)) — could not be fully enumerated from the systems Cymbal could query; rather than smooth this into the totals, it was carried as a stated gap with a named owner and a plan. The findings were then reported as a profile rather than a single number (§10.3): Cymbal could enumerate most of the estate but could not yet name owners for the majority of it, which told it the binding constraint was governance, not discovery — and that is the constraint the following decision tested.

### 13.2 The One Decision the Inventory Changed

A worked example earns its place by showing a decision that came out *differently* because the inventory existed — not a list of what was found, but one choice that would otherwise have been made wrongly. For Cymbal, the decision was an integration it had already budgeted and designed, and the inventory cancelled it (well, rewrote it — the cancellation is the illustration).

The proposal was this. A new internal consumer needed programmatic access to the APIs of an existing system. The prepared answer was the standard one: provision a new integration identity, and build the small proxy service that would own it, because the standard pattern for "something needs to call our APIs" is "give it an identity and a service to hold it". The work was scoped, costed, and on the next quarter's list.

Then the inventory arrived with its second-order finding: **the identity that would be provisioned already existed, several times over.** The discovery found service accounts created for earlier versions of exactly this need — one from a pilot that had graduated silently (mode 4), one belonging to a decommissioned system whose replacement had been folded into the same estate (§4.2), each already scoped to the API surface the new consumer needed, each now orphaned, each holding a credential that still worked. Cymbal had been about to build a *fourth* identity for a purpose that three others already served and none owned.

The decision the inventory changed was therefore not "don't build the integration" — the consumer genuinely needed access. It was that the integration was **re-pointed onto the existing identities, adopted into ownership, scoped and reviewed, instead of creating a new one beside them.** The budgeted proxy service was dropped; the work became a governance task (adopt, scope, review three existing identities, retire the two the consumer's use case did not need) rather than a build. Three consequences the Cymbal team drew from it:

- **The inventory's value was not in what it prevented being created; it was in what it revealed already existed.** The proposal was not unsafe; it was *redundant*, and redundancy in the identity estate is how the estate grows into the state §3.4 describes.
- **Had the new identity been provisioned as planned, it would have orphaned exactly as the other three had** — because the reason the other three were orphaned, a good offboarding process for people and no offboarding at all for identities (§4.1, §4.4), would not have been changed by adding a fourth.
- **The decision was a decision, and it needed the list to make it.** No individual interview, no system owner's memory, and no tool pointed at any single system would have surfaced that three satisfying identities already existed across three different systems; only the enumerated estate did.

This is the shape of every decision an inventory changes, and it is why §3.4 makes the list the first deliverable rather than a preliminary: the estate's next mistake is not always a risk, it is often a duplicate, and a duplicate is invisible until someone can see the whole set.

### 13.3 The Thesis, Restated

Cymbal's cycle is the guide in miniature. It did not begin by buying a mechanism or writing a policy: it began by producing the one thing the estate had never had — **a list of the subjects that can act on its systems, with owners against them** — and the list, by existing, changed an integration decision the institution would otherwise have got wrong. Everything after that was ordinary governance applied to a named set: owner the unowned, scope the over-privileged, split the shared, retire the orphaned, and stand the review up so the register stays worth having (§7). None of it needed a new mechanism; all of it needed the enumeration the mechanisms do not provide (§6.2). The example ends where the thesis begins: the difficulty was never giving a machine an identity.

---

## 14. The Anti-Patterns

Each anti-pattern is stated as **symptom → cause → guardrail**, consistent with the sibling guides, so a design review can use the table as a checklist. Every guardrail points at the section that develops the fix.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| **1** | **Treating non-human identity as a secrets-management problem.** | A vault and a secrets scanner are deployed; the estate still cannot say which identities can act on a system | The identity-versus-secret conflation (§5.1); a credential mechanism read as a subject inventory (§5.3, i) | Keep the two statuses separate in reporting (§9.2, §11.2); pair every credential program with an enumeration program (§3.4) |
| **2** | **Rotating what is inventoried while the estate grows unlisted.** | Rotation reports are green; new identities are created daily with no register entry | Creation is self-service and unapproved (§3.2); the reassuring half of the work is done and the unglamorous half is not (§11.2, rotation theatre) | Registration before issuance (§7.2); measure progress in subjects listed and owned (§11.2) |
| **3** | **Assuming the modern mechanism solved the governance problem.** | Workload identities or federation are in place, so the estate believes non-human identity is handled | Attestation makes an identity *provable*, not *enumerable* (§6.2); a mechanism read as a discipline | Keep the register as a separate deliverable from the mechanism (§3.4, §6.2) |
| **4** | **Naming a team, not a role, as the owner.** | "The platform team owns it" appears against identities; no individual can answer whether the identity is needed | Ownership treated as a label rather than the retirement signal (§7.1); the §4.3 default of nobody | Name an accountable role per identity at creation (§7.1); mirror the residual-risk register's named-role convention ([zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) §12.6) |
| **5** | **Letting one identity hold both sides of a control.** | A workflow identity can both raise and approve; segregation is asserted on the diagram and absent in the system | Segregation expressed as distinct *people* rather than distinct *identities* (§7.4); the machine holding both halves ([servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §7.4) | Express segregation as distinct identities with distinct authority (§7.4) |
| **6** | **Giving an agent a borrowed identity.** | An agent runs as a human's credentials or a shared service account | The agent assigned a convenient identity rather than its own (§8.2, §8.4); attribution destroyed by borrowing | Give the agent its own registered, owned identity, and bound its authority to the delegation ([servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) §6, §12) |
| **7** | **Building the inventory once and calling it done.** | A discovery snapshot is presented as coverage; it is stale within a quarter | The register treated as a report rather than a feed (§11, mode 6); review not stood up (§7.5) | Make registration part of creation and review a standing activity (§7.2, §7.5) |
| **8** | **Calling a permission an identity, and an identity a permission.** | The "identity inventory" is a list of roles or entitlements; the subjects are not in it | The category confusion of §2.4; a role is not a subject | Inventory subjects, then map their roles; a role or a permission is never a line item of the identity register (§2.4) |
| **9** | **Reading a green rotation report as a clean estate.** | The credential reports look healthy; the orphaned and shared identities are untouched | Reporting measures the credential half of the estate (§5.3, iii); orphans are off the list by construction (§4.2) | Report orphan-hunt and owner-coverage alongside rotation (§7.5, §10.1 Q3); the three are different questions |
| **10** | **Letting the audit-log question wait.** | The log records actions but cannot name who decided them, and the estate finds this out at audit or incident | Attribution was never treated as a design requirement; shared and orphaned identities survive (§12.1) | Design for attribution: distinct subjects per actor, an owner per subject, and the evidence question answered as a control (§7.6, §12.1) |

The thread through all ten is the thesis: **the failure is almost never a missing mechanism — it is a missing list, a missing owner, or a distinction not made.** The estate can issue every credential the mechanisms allow and still fail, because the mechanisms were never the hard part.
---

## 15. The Claims Audit

Verified this pass = checked against the source named, extracted in **October 2026**, unless a source date is given. The convention follows the repo: ✅ verified this pass against a named primary source with its date; ⚠ vendor claim / approximate / single secondary source; ⚠-knowledge well-established industry knowledge not re-verified this pass; ❌ could not be verified at source (see §16). The **Kind** column names what sort of source it is — standards-body publication, primary specification, vendor documentation, vendor site, secondary report, or this repository's own internal material.

| # | Claim | Status | Source, with date and kind |
|---|---|---|---|
| 1 | The **OWASP Non-Human Identities Top 10** describes itself as **"An OWASP incubator project"**, and the **2025** list is published with a documentation index | ✅ | owasp.org/www-project-non-human-identities-top-10/ and /2025/, extracted October 2026 — standards-adjacent nonprofit (OWASP) project page |
| 2 | **NHI1:2025 — Improper Offboarding** is defined as *"the inadequate deactivation or removal of non-human identities (NHIs) such as service accounts and access keys when they are no longer needed"*, and deprecated services' NHIs *"can be exploited by attackers to gain unauthorized access"* | ✅ | OWASP NHI Top 10 project page, extracted October 2026 — nonprofit project page, quoted verbatim (2025 list) |
| 3 | OWASP's framing sentence: *"Production software environments are composed of a large number of applications which need to be identified, resulting in 'non-human identities' or NHI."* | ✅ | Same page, extracted October 2026 — nonprofit project page, quoted verbatim |
| 4 | The list also names **NHI5:2025 Overprivileged NHI**, **NHI9:2025 NHI Reuse** and **NHI10:2025 Human Use of NHI** | ✅ | Same page, extracted October 2026 — nonprofit project page |
| 5 | The OWASP project's listed project leaders are from **Astrix Security** and **Torch Security** (vendor personnel on a nonprofit project) | ✅ | Same page, extracted October 2026 — nonprofit project page, personnel listing |
| 6 | The **Cloud Security Alliance** whitepaper **"Machine Identity in Cybersecurity and IAM"** was **released 09/19/2023**, and defines machine identities as **"anything other than human"**, enumerating *"device identities, digital identities, and workload identities"* | ✅ | cloudsecurityalliance.org/artifacts/machine-identity-in-cybersecurity-and-iam, released 09/19/2023, extracted October 2026 — standards-body (cloud-security nonprofit) publication |
| 7 | A vendor-adjacent site (**nhimg.org**) claims **Astrix Security** "can be credited as the first to coin the term 'Non-Human Identity'", dating it to the **RSA Innovation Sandbox, May 2023** | ⚠ (❌ at source) | nhimg.org — **page would not extract this pass (❌)**; single self-interested secondary source; recorded as ⚠, not asserted |
| 8 | **Astrix Security** presents itself under a banner **"Astrix is now part of Cisco"** and markets "Identity Security for AI Agents & NHIs" | ✅ | astrix.security, extracted October 2026 — vendor site (banner links a Cisco news post) |
| 9 | **Gartner's IAM taxonomy** reportedly places NHI above machine identity in a hierarchy and reportedly calls "NHI" a vendor buzzword | ⚠ (❌ at source) | Reported second-hand by a vendor explainer dated **September 2, 2026** that names the Gartner document by URL; **the Gartner document (gartner.com/en/documents/6178123) did not extract this pass (❌)** — secondary report only, primary in §16 |
| 10 | **NIST SP 800-207** (Zero Trust Architecture) was published **August 2020**, DOI 10.6028/NIST.SP.800-207; its abstract frames defence around *"users, assets, and resources"* and states that authentication and authorization *"both subject and device"* are *"discrete functions performed before a session"*; it does not use the term "non-human identity" | ✅ | csrc.nist.gov/pubs/sp/800/207/final, **Date Published: August 2020**, extracted October 2026 — standards-body (NIST) publication |
| 11 | NIST SP 800-53 account management (AC-2) is written around **accounts and users** rather than non-human subjects | ⚠-knowledge | Established knowledge of the control's framing; **not re-verified at source this pass** |
| 12 | A **SPIFFE ID** is a URI of the form `spiffe://trust-domain/workload-identifier`; an **SVID** proves it in **X.509** or **JWT** form; a **WIT-SVID** specification also exists in the spec index | ✅ | spiffe.io/docs/latest/spiffe-about/spiffe-concepts/ and the specs index, extracted October 2026 — primary specification / project documentation |
| 13 | SPIFFE defines a **workload** as *"a single piece of software, deployed with a particular configuration for a single purpose"*, and the Workload API issues short-lived, auto-rotated key material without a co-deployed secret | ✅ | spiffe.io/docs/latest/spiffe-about/spiffe-concepts/, extracted October 2026 — primary specification documentation, quoted |
| 14 | **AWS EKS hosts a public OIDC discovery endpoint** for each cluster containing the signing keys for `ProjectedServiceAccountToken` JWTs, "so external systems, such as IAM, can validate and accept the OIDC tokens that are issued by Kubernetes"; the signing private key rotates every 7 days; legacy service-account tokens do not expire | ✅ | docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html, extracted October 2026 — vendor (AWS) documentation, quoted |
| 15 | **Google Cloud Workload Identity Federation** provides access to Google Cloud resources "by using federated identities instead of a service account key", says it "eliminates the maintenance and security burden associated with service account keys", and follows the **OAuth 2.0 token exchange** specification | ✅ | cloud.google.com/iam/docs/workload-identity-federation, page **Last updated 2026-10-09**, extracted October 2026 — vendor (Google Cloud) documentation |
| 16 | **Microsoft Entra workload identity federation** lets an app registration or user-assigned managed identity trust an external IdP's tokens, and "eliminates the maintenance burden of manually managing credentials and eliminates the risk of leaking secrets or having certificates expire" | ✅ | learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation, page **Last updated on 04/09/2025**, extracted October 2026 — vendor (Microsoft) documentation |
| 17 | **Kubernetes bound service-account tokens** are obtained via the `TokenRequest` API and mounted by a projected volume; they have bounded lifetimes and "are automatically invalidated when the Pod they are mounted into is deleted" | ✅ | kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/, extracted October 2026 — primary project documentation |
| 18 | The Identity-pillar row of the zero-trust guide names *"service identities and workload identity (SPIFFE/SPIRE) for machine-to-machine"* within a single table cell | ✅ | [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §4.2 — this repository, internal source |
| 19 | This repository's own a16z guide records the widely circulated non-human-to-human identity ratio, attributed to a venture guest essay, as **"⚠ unverified — no source given in the entry"** and flags it as uncited in its claims audit | ✅ | [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) claims-audit row 16 and §15 — this repository, internal source |
| 20 | **No machine-to-human identity ratio is printed in this guide** — nor any credential-sprawl percentage, breach statistic, market size, growth rate or vendor count for the non-human identity category | ⚠ (by design) | Reason: the most-circulated figure is uncited, including in this repository's own [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) (row 19 above), and no dated primary source stating a ratio was located this pass. A ratio without a dated primary source is a number pretending to be a fact, so this guide prints none |
| 21 | **Conclusion:** "non-human identity" is a recently-emerged, vendor-driven framing that an application-security body (OWASP, 2025) and a cloud-security body (CSA, 2023, under the older label "machine identity") have begun to formalise — **not a settled standard** | ⚠ | This guide's assessment, supported by rows 1–11 above and the absence of a verified standards definition — an assessment, stated as such |

**The honesty note.** Every mechanism claim in this guide is traceable above, with its source, its date and its kind, and every mechanism is cross-referenced to the sibling that owns it rather than re-derived. The ✅ rows are the primary extractions; the ⚠ and ⚠-knowledge rows are exactly the ones where a claim rests on a vendor, a secondary source, or established practice rather than a dated primary document; the single ❌-bearing rows (7 and 9) are the vendor and analyst claims that could not be reached at source and are recorded in §16 rather than papered over. **Rows 20 and 21 are the two that keep the guide honest about what it is:** a framing that prints no invented number, and that says plainly it is a framing.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

The following could not be confirmed against a primary source during this pass, and are recorded honestly rather than asserted:

- **The Gartner IAM taxonomy document** (`gartner.com/en/documents/6178123`) — **❌**. `web_extract` returned a scraper failure (all engines failed; likely blocking or authentication), not page content. Any statement about Gartner's NHI-versus-machine-identity hierarchy, or about its reported observation that "NHI" is used as a vendor buzzword, therefore rests only on **second-hand vendor reporting** (an explainer dated September 2, 2026 that names the document by URL) and is cited **⚠, never as a Gartner position**. A reader relying on the taxonomy should reach the primary document directly.
- **The claim that Astrix Security "coined the term 'Non-Human Identity'"** at the RSA Innovation Sandbox in May 2023 — **❌ at source**. The page carrying the claim (`nhimg.org`) would not extract, and it is a single **self-interested** source. This guide records the claim as **⚠ and does not assert it**; what it *does* verify is the commercial outcome (Astrix now presents as part of Cisco, row 8) and the existence of the term in a nonprofit project's 2025 framing (rows 1–5). Whether the term was coined by that one vendor, or emerged from the market generally, is **not established** here.
- **RFC 8705 (OAuth 2.0 Mutual-TLS) and RFC 8693 (OAuth 2.0 Token Exchange)** — **❌ this pass**. The IETF datatracker pages would not extract (scraper failure). These mechanisms are **cross-referenced, not re-derived**: [distributed_auth_guide.md](distributed_auth_guide.md) §8.4 already carries OAuth client-credentials with mTLS (RFC 8705) in its S2S table, and Google Cloud's own documentation (row 15) states its Workload Identity Federation follows the OAuth 2.0 token exchange specification. This guide asserts nothing about an RFC it could not read this pass.
- **NIST SP 800-53 AC-2's exact wording** around accounts and users — **⚠-knowledge**, not re-verified at source this pass (row 11). Do not quote the control text; the framing observation is all that is claimed.
- **Any regulator's or jurisdiction's rule bearing on non-human identity** — **not verified, and deliberately not asserted**. No obligation under MAS, DORA, PSD2, SWIFT, Basel, SOX or any other regime is claimed in this guide (§12.2). The regulatory frames the repository *has* established live in the sibling guides' ledgers and are cross-referenced, not restated.
- **Any prevalence figure for orphaned, shared or over-privileged non-human identities** — **no figure is asserted anywhere in this guide**, by design (§3.4, §15 row 20). The failure modes and the assessed patterns are stated as the common practitioner shape with the explicit qualification that **practice varies by institution and estate era**.
- **`web_search` was intermittently empty on this host** during this pass — several queries returned nothing usable. This is recorded as a **tool limitation**, never as evidence of absence: a page not found by search is not a page that does not exist. Where a search was empty and a claim mattered, this guide either used `web_extract` on a known primary URL or recorded the claim as ❌.

### 16.2 The Glossary

- **Non-human identity (NHI)** — a subject in a system, other than a person, that can be authenticated, granted permissions, and have actions attributed to it. An identity is a **subject**, not a credential (§2.1, §5).
- **Machine identity** — the older, standards-adjacent label for the same population; the Cloud Security Alliance defines it as "anything other than human", subsuming device, digital and workload identities (CSA, 2023) (§1.3).
- **Workload identity** — the identity of a running piece of software; SPIFFE's territory (`spiffe://trust-domain/path`) (§2.2; mechanism in [distributed_auth_guide.md](distributed_auth_guide.md) §8.1).
- **Service account** — a platform-level account a service, job or automation runs as (Kubernetes `ServiceAccount`, Windows/AD service account, database login) (§2.2).
- **Service principal** — the directory object that *is* an application's identity in a cloud directory (e.g. an Entra app registration/service principal) (§2.2).
- **API key** — a bearer credential; a **credential**, frequently with only a weak or implicit subject behind it (§5.3, ii).
- **Secret** — a credential: the material a subject presents to prove it is that subject. A secret is not an identity (§5.1).
- **Certificate** — a signed statement binding a public key to a name; the carrier of an X.509 identity ([container_certificates_guide.md](container_certificates_guide.md)).
- **Token** — a time-bounded assertion presented for access (JWT, access token, projected service-account token) (§6.1).
- **Attestation** — platform evidence that a workload is what it claims, checked before a credential is issued — the property that lets an identity be established without a bootstrap secret ([distributed_auth_guide.md](distributed_auth_guide.md) §8.2).
- **Rotation** — replacing a credential on a schedule or event. Rotation acts on a **secret**; it does not change the **identity** (§5.3, i).
- **Orphan** — a non-human identity, or the credential of one, that has outlived its purpose, its owner, or the human who once used it, and that nothing still manages (§4.2).
- **Privilege** — what an identity is permitted to do; the default state of an NHI's privilege is **over-privilege**, not least privilege (§7.3).
- **OIDC (OpenID Connect)** — the identity layer on top of OAuth 2.0; owned by [distributed_auth_guide.md](distributed_auth_guide.md) §2.4.
- **SPIFFE** — the Secure Production Identity Framework For Everyone, a CNCF standard for workload identity (SPIFFE IDs and SVIDs); owned by [distributed_auth_guide.md](distributed_auth_guide.md) §8.1. **SPIRE** is its reference implementation (§8.2).
- **SVID (SPIFFE Verifiable Identity Document)** — the document with which a workload proves its identity, in X.509 or JWT form (§6.1; [distributed_auth_guide.md](distributed_auth_guide.md) §8.1).
- **WIF (Workload Identity Federation)** — a cloud's model for granting access to external workloads via federated identities instead of long-lived keys (AWS, Google Cloud, Microsoft Entra) (§6.1).
- **HSM (Hardware Security Module)** — a hardware or managed key store that holds signing keys; the layer that keeps a signing authority from being a file on disk ([kmip_key_management_guide.md](kmip_key_management_guide.md)).
- **CMDB / identity inventory** — the register of record. A CMDB is a *configuration* register and an identity inventory is a *subject* register; the two are related and not the same, and a CMDB alone does not answer §10.1's questions (§3.4, §7.6).
- **JIT (just-in-time access)** — granting access only when needed, for only as long as needed; the contrast to the standing credential that §11 mode 5 identifies on an agent.
- **Least privilege** — access limited to what a function needs; for non-human identities this is a project rather than a default state (§7.3).

### 16.3 The Cross-References

Because this guide is the discipline and the neighbouring guides are the mechanisms, the reader's next stop depends on the question:

- **The parent discipline — zero trust, the CISA pillars, NIST SP 800-207 and the estate migration plan** — [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) (§1–§5, §7, §9.3); its §4.2 Identity row is the evidence cited in §1.6 and §15 row 18.
- **Every mechanism — authn/authz, OIDC, JWT, mTLS, session vs token, token validation, RBAC/ABAC/ReBAC, OPA, the S2S models including SPIFFE/SPIRE, and the microservices worked example** — [distributed_auth_guide.md](distributed_auth_guide.md) (§1–§3, §5–§8, §11). This is the guide this one most risks duplicating, and §6 of this guide does not restate it.
- **The mainframe credential problems — the identity bridge, vaulting under the named identity, and the audit-side answer** — [zero_trust_mainframe_guide.md](zero_trust_mainframe_guide.md) (§5, §6, §10).
- **The shared database credential and the shared transfer identity** — [zero_trust_legacy_estate_guide.md](zero_trust_legacy_estate_guide.md) (§9, §10); its §12.6 named-role residual-risk convention is the one §7.1 and §12.1 adopt.
- **Certificate and key material** — [container_certificates_guide.md](container_certificates_guide.md) (§8 rotation, §9 cert-manager, §10 mTLS between containers, §12 expiry monitoring); [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) (the secrets-management context); [kmip_key_management_guide.md](kmip_key_management_guide.md) (the key-management and HSM layer).
- **The Kubernetes and OpenShift side of workload identity** — [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) (§5 RBAC and admission, §8 secrets and supply chain, §10 certificate management and audit); provider context in [cloud_providers_guide.md](cloud_providers_guide.md) (§3 AWS, §4 Azure, §5 GCP).
- **The citizen-developer case of the same problem** — [power_platform_cicd_guide.md](power_platform_cicd_guide.md) §7 ("Authentication: Service Principals, Application Users, and Secrets") and [microsoft_power_platform_guide.md](microsoft_power_platform_guide.md) (§9 governance and ALM).
- **The agent case in full** — [servicenow_agentic_platform_guide.md](servicenow_agentic_platform_guide.md) (§6 the identity question, §7 evidence and segregation of duties, §12 the operating model) and [ai_llm/enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) (§4.6–§4.7, §6.2), with [ai_llm/agent_experience_guide.md](ai_llm/agent_experience_guide.md) (§9–§10), [ai_llm/agent_decision_policy_guide.md](ai_llm/agent_decision_policy_guide.md) (§2, §9), [ai_llm/agent_sandboxing_strategies_guide.md](ai_llm/agent_sandboxing_strategies_guide.md) (§2, §10) and [llm_development_risks_security_guide.md](llm_development_risks_security_guide.md) (§10, excessive agency).
- **The regulated, audit and governance frames** — [audit_as_code_guide.md](audit_as_code_guide.md) (§8 evidence management, §11 maturity), [api_governance_guide.md](api_governance_guide.md) (§6 registry and catalog, §8 security and observability, §9 governance roles, §10 banking), [data_governance_guide.md](data_governance_guide.md) (§4 governance organisation, §10 banking and regulatory context), and in the banking folder [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md), [ai_risk_register_guide.md](../banking/ai_risk_register_guide.md), [regtech_guide.md](../banking/regtech_guide.md), [risk_data_aggregation_guide.md](../banking/risk_data_aggregation_guide.md) and [risk_management_models_guide.md](../banking/risk_management_models_guide.md).
- **The repository precedent for owning a newly-coined term honestly** — [ai_llm/agent_experience_guide.md](ai_llm/agent_experience_guide.md) §1 (its own term established or rejected) and [a16z_big_ideas_2026_guide.md](a16z_big_ideas_2026_guide.md) (the unverified-ratio ledger this guide's §15 row 20 defers to).

### 16.4 The Closing Summary

This guide set out to write the discipline that the mechanisms leave out, and it arrived at a thesis in two halves.

The first half is that **non-human identity is a real, large and under-governed asset class, and the reason it is under-governed is structural, not cultural.** The estate is built on subjects — service accounts, service principals, workload identities, certificates, API keys, tokens, bots, RPA accounts, agents — that were created by whoever needed them, in whatever system required them, with no joiner-mover-leaver signal, no manager, no departure, and no default owner (§2.3, §3.1). The consequence is the asymmetry of §3.2 — creation is self-service and unapproved while enumeration, ownership and retirement are nobody's job — and it produces the estate's expected steady state: the orphan (§4.2), the shared credential (§11, mode 2), and the rotation theatre that freshens the credentials it knows about while the set of subjects it does not know about grows (§11, mode 3). And the term itself is honest to state: **non-human identity is a recently-emerged, vendor-driven framing** that a nonprofit application-security body and a cloud-security body have begun to formalise under two different labels, and this guide presents itself as a framing rather than a canon (§1.2–§1.4).

The second half is that **the discipline is a list, an owner, and a distinction** — none of which any mechanism supplies. You cannot govern what you cannot enumerate (§3.4); an identity is a subject and a secret is a credential, and rotating one does nothing for the other (§5); attestation makes an identity provable, not enumerable, so the mechanisms can modernise every credential in the estate and still leave it unable to name its actors (§6.2). Governance is therefore the answer, and it is a governance of named things: an owner per identity at creation, registration before issuance, least privilege engineered rather than assumed, segregation expressed as distinct identities rather than distinct people, a review that hunts the orphan class, and an audit answer that reads out the register and volunteers the gaps (§7). The agent is the newest and sharpest case of the same discipline — an identity that behaves like a user, and must be given its own identity and an authority bounded to the delegation that justifies it, or it will break attribution exactly as the orphan and the shared account already do (§8). And the Cymbal Bank example shows the payoff as a single decision: the list, by existing, revealed that a new identity the institution was about to build was a fourth copy of three it already had and did not own, and changed the decision from a build to a governance task (§13.2).

It closes on the sentence it opened with, and it is the whole argument.

**the problem is not that machines have identities; it is that nobody can list them.**
