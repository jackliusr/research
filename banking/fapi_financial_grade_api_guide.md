# Financial-grade API (FAPI): The Security Profile for High-Value Financial APIs — A Comprehensive Guide

**The OpenID Foundation's FAPI security profiles — what they are, the attacker model they answer to, the mechanisms they mandate, the exact status of each version, the certification programme, and the regulators who have adopted them — written from the specification text outward, with a fictional Cymbal Bank worked example.**

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Banking Domain / Open Banking, Open Finance and High-Value API Security — the FAPI *standard* itself: its threat model, its 1.0 and 2.0 profiles, the OAuth 2.0/IETF mechanisms it composes, the OpenID Foundation conformance and certification programme, the regulatory regimes that mandate or imply it, and what an institution actually has to build. The worked example is a fictional bank, Cymbal Bank, opening account and payment APIs to third-party providers.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the banking cluster):** [The Open Bank Project (OBP)](openbankproject_guide.md) — the *platform* genre: TESOBE's open-source banking-API stack, its API, its PSD2/XS2A mapping, deployments and the open-source business model. That guide owns the PLATFORM; this guide owns the STANDARD (cross-ref its §2.4 and §4) · [Oracle Banking Microservices Architecture](oracle_banking_microservices_architecture_guide.md) — the source of this guide's framing quote (its §9.3) and the API-gateway/token-validation view of the same problem (cross-ref §10) · [Mastercard](mastercard_guide.md), [Visa](visa_guide.md), [Adyen](adyen_guide.md) — the scheme and acquiring sides of money movement that FAPI-secured APIs sit beside · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore regulatory overlay and the Cymbal Bank persona conventions (cross-ref §9.4) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — the repo's requirements-matrix and verification-ledger conventions.
> **Companion guides (other folders):** [Distributed Auth](../technology/distributed_auth_guide.md) — OAuth 2.0/OpenID Connect foundations; this guide assumes that material rather than repeating it · [API Governance](../technology/api_governance_guide.md) — the API-lifecycle and consumer-governance discipline that binds the *non-security* half of an open-banking estate · [Cybersecurity](../technology/cybersecurity_guide.md) and [Security by Design](../technology/security_by_design_guide.md) — the wider control environment an authorisation server lives in · [Enterprise Middleware & Integration Platforms](../technology/enterprise_middleware_integration_platform_guide.md) · [Payment Rails](payment_rails_guide.md) — clearing/settlement for the payment leg a PISP initiates.

**Verification convention used throughout:** ✅ = verified at primary source in this research pass (specification text on openid.net, IETF RFC text, an OpenID Foundation programme page, a regulator or scheme page — each dated in §13); ⚠ = flagged (secondary source, vendor claim, single-source, or approximate — treat as unconfirmed); unmarked = structural/industry knowledge presented as such. Every RFC number, spec version, publication date and certification status quoted here was read off a primary page in this pass; nothing of that kind is cited from memory. The claims audit in §13 dates each item and names the source, and §14 records everything that could not be established.

**How to read this guide:** §1 frames what FAPI is and what it is not. §2 is the intellectual core — the attacker model and the threat→requirement mapping. §3 explains why a competent OAuth 2.0 deployment was not considered sufficient. §4 and §5 are the two profile generations, stated precisely with dates and status. §6 walks the mechanisms one by one with their correct identifiers. §7 is the crypto and key-management reality. §8 is certification and conformance. §9 is the regulatory footprint, regime by regime. §10 is what a bank actually has to build and run. §11 is the critique and the limits. §12 is the fictional Cymbal Bank worked example. §13–§17 close with the claims audit, the explicit negative findings, the glossary, cross-references and the summary.

---

## Table of Contents

1. [Overview and Framing — What FAPI Is, and What It Is Not](#1-overview-and-framing--what-fapi-is-and-what-it-is-not)
2. [The Threat Model — Threats, Attackers, and the Requirement That Defeats Each](#2-the-threat-model--threats-attackers-and-the-requirement-that-defeats-each)
3. [Why Plain OAuth 2.0 Is Not Enough](#3-why-plain-oauth-20-is-not-enough)
4. [FAPI 1.0 — Part 1 Baseline and Part 2 Advanced](#4-fapi-10--part-1-baseline-and-part-2-advanced)
5. [FAPI 2.0 — Security Profile, Message Signing, and What Was Simplified](#5-fapi-20--security-profile-message-signing-and-what-was-simplified)
6. [The Mechanisms, One by One](#6-the-mechanisms-one-by-one)
7. [Cryptographic and Key-Management Reality](#7-cryptographic-and-key-management-reality)
8. [Certification and Conformance](#8-certification-and-conformance)
9. [The Regulatory Footprint, Regime by Regime](#9-the-regulatory-footprint-regime-by-regime)
10. [The Implementation Reality for a Bank](#10-the-implementation-reality-for-a-bank)
11. [The Critique and the Limits](#11-the-critique-and-the-limits)
12. [Cymbal Bank Worked Example — A FAPI-Mandated Market](#12-cymbal-bank-worked-example--a-fapi-mandated-market)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary — Including the Identifier Decoder](#15-glossary--including-the-identifier-decoder)
16. [Cross-References and Further Reading](#16-cross-references-and-further-reading)
17. [Closing Summary](#17-closing-summary)

---

## 1. Overview and Framing — What FAPI Is, and What It Is Not

### 1.1 The One-Paragraph Answer

FAPI — the **Financial-grade API**, now styled by the OpenID Foundation as just **FAPI** — is a *security profile*. It is not a new protocol. It takes the OAuth 2.0 authorisation framework (RFC 6749) and OpenID Connect Core 1.0 and constrains them: it removes optionality, forbids the weaker choices, mandates the stronger ones, and specifies exactly how a client, an authorisation server and a resource server must behave when the resources behind the API are account data and money movement. ✅ The FAPI Working Group's own framing is blunt about the scope: *"The FAPI WG does not work on data models or standards for financial or other APIs. These are ecosystem specific."* ✅ ([openid.net/wg/fapi/](https://openid.net/wg/fapi/), read 17 September 2026).

That single sentence is the whole boundary. FAPI tells you how to secure the channel, the consent grant and the token. It does not tell you what an account looks like, what a payment request contains, or what "consent" means commercially. Those belong to the business standard layered on top — the UK Open Banking Read/Write API standards, Australia's Consumer Data Standards, Brazil's Open Finance developer standards, the Financial Data Exchange (FDX) API in North America, the Berlin Group NextGenPSD2 specification, STET in France. Each is a *data and business* standard; FAPI is the *security* standard many of them adopt underneath.

### 1.2 What FAPI Is NOT

| FAPI is not… | Because… |
|---|---|
| **An API specification** | It defines no resources, endpoints, payloads or data models. It profiles OAuth 2.0/OIDC flows, not REST resources. Endpoint catalogues belong to the ecosystem standard (§9). |
| **A business standard** | It says nothing about pricing, liability, dispute handling, SLAs or consent longevity. Liability sits in the scheme rulebook and in regulation. |
| **A protocol** | It invents no wire protocol and no message format. Every mechanism it mandates is an IETF RFC or an OpenID Foundation specification it points at (§6). |
| **A regulator** | Adoption is voluntary at the standard level and compulsory only where a regulator or scheme makes it so (§9). FAPI has no enforcement power of any kind. |
| **A security audit** | Passing the OpenID Foundation's conformance suite proves protocol conformance — not that an implementation is free of vulnerabilities, and not that the business process behind it is sound (§8, §11). |
| **A complete security architecture** | The FAPI 2.0 Attacker Model explicitly places TLS integrity, JWKS key distribution, the user's device and browser, and end-user identity proofing *outside* its scope (§2.4, §11.4). ✅ |
| **Static** | It is in active generational transition: FAPI 1.0 is final and maintained but frozen; FAPI 2.0's core profile and message-signing profile are final; ecosystems are migrating at different speeds (§4, §5). |

### 1.3 The Anchor Quote

The repo's only prior substantive mention of FAPI is a single line inside an Oracle product guide, and it is the right place to start because it states the *relationship* correctly — an API standard with a security profile under it:

> "**Open Banking UK** — the CMA open banking API standards (account and payment APIs with FAPI security profile)." — [oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md), line 688

Read it again: the *API standards* are the subject; the *FAPI security profile* is a qualifier. That is exactly the layering this guide unpacks — CMA Order→Open Banking Limited standards own the API; OpenID Foundation FAPI owns the security posture. The same Oracle section lists PSD2 AISP/PISP flows and Singapore's SGFinDex approach in the same breath, which is a fair map of the regulatory footprint in §9.

### 1.4 Why a Bank Should Care

- **It is a precondition of market access.** In the UK, Australia, Brazil and others, participating in the open banking / open finance scheme means meeting the scheme's security profile, which is FAPI-derived (§9). You do not opt out of the *standard*; you choose whether to be certified or merely compliant.
- **It is a substitute for credential sharing.** FAPI's own introduction frames the alternative it displaces: screen scraping, which "accesses user's data and functions by impersonating a user through password sharing," producing "a brittle, inefficient, and insecure practice [that] creates security vulnerabilities which require financial institutions to allow what appears to be an automated attack against their applications." ✅ Every shift from scraping to consent-API moves risk from the customer's shared password to a scoped, revocable, sender-constrained token.
- **It is a checklist a supervisor can inspect.** The WG presents the specifications as "Clear point-by-point specifications that implementers can use as a 'check list'" plus "Exhaustive conformance tests." ✅ Normative checklist plus testable evidence is what makes FAPI auditable in a way that "we do OAuth properly" is not.
- **Its scope has outgrown banking.** The WG renamed the work from "Financial-grade API" to "FAPI" because "the specification is appropriate for many high-value use-cases requiring a more secure model beyond just financial services" ✅, and FAPI 2.0 says it is "designed to be universally applicable for protecting APIs exposing high-value and sensitive (personal and other) data, for example, in e-health and e-government applications" ✅. A bank implementing it is building a reusable high-assurance API security posture.

### 1.5 The Vocabulary Decoder

| Term | Expansion | One-line meaning | Spec anchor |
|---|---|---|---|
| **FAPI** | Financial-grade API (now just "FAPI") | The OpenID Foundation's high-security OAuth 2.0 profile family | OIDF FAPI WG ✅ |
| **ASPSP** | Account Servicing Payment Service Provider | The bank that holds the account (EU/PSD2 vocabulary) | PSD2 vocabulary, not an OIDF term |
| **TPP** | Third-Party Provider | Umbrella for AISP/PISP (EU/PSD2 vocabulary) | PSD2 vocabulary |
| **AISP** | Account Information Service Provider | Reads account data with consent | PSD2 |
| **PISP** | Payment Initiation Service Provider | Initiates a payment order with consent | PSD2 |
| **PAR** | Pushed Authorization Requests | Push the authorization request to the AS over a direct, authenticated call instead of the front channel | **RFC 9126** ✅ |
| **JAR** | JWT-Secured Authorization Request | Signed (optionally encrypted) request object carried as `request`/`request_uri` | **RFC 9101** ✅ |
| **JARM** | JWT Secured Authorization Response Mode | The authorization *response* returned as a signed JWT | **OpenID Foundation specification — not an IETF RFC** ✅ (§6.6) |
| **mTLS** | Mutual TLS | Both sides present certificates; authenticates the client and can bind the token to a certificate | **RFC 8705** ✅ |
| **DPoP** | Demonstrating Proof of Possession | Application-layer sender-constraining of tokens to a client-held key | **RFC 9449** ✅ |
| **RAR** | Rich Authorization Requests | Structured `authorization_details` for fine-grained consent | **RFC 9396** ✅ |
| **CIBA** | Client-Initiated Backchannel Authentication | Decoupled authentication: the client starts the flow, the user approves out-of-band | **OIDC CIBA Core 1.0** ✅ |
| **SCA** | Strong Customer Authentication | Two independent authentication factors (EU/PSD2 concept) | PSD2 + EBA RTS on SCA & CSC |
| **Conformance Suite** | OpenID Foundation Conformance Suite | The open-source test tool that produces certification evidence | [gitlab.com/openid/conformance-suite](https://gitlab.com/openid/conformance-suite) |

**Boundary declaration.** This guide owns the FAPI standard: its threat model, its 1.0 and 2.0 versions and profiles, the mechanisms it mandates with their identifiers, the certification programme, and the regulatory footprint. It does **not** own: the Open Bank Project platform ([openbankproject_guide.md](openbankproject_guide.md)), general OAuth 2.0/OIDC tutorial material ([../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md)), API lifecycle governance ([../technology/api_governance_guide.md](../technology/api_governance_guide.md)), payment clearing mechanics ([payment_rails_guide.md](payment_rails_guide.md)), or scheme/acquirer economics ([adyen_guide.md](adyen_guide.md), [mastercard_guide.md](mastercard_guide.md)). Where those are touched it is to point at them, not to re-derive them. Restated in §16.

---

## 2. The Threat Model — Threats, Attackers, and the Requirement That Defeats Each

### 2.1 The Attacker Model Is a Separate, Normative Artefact

FAPI 2.0 introduced a document the 1.0 generation did not have as a standalone artefact: the **FAPI 2.0 Attacker Model** — published **22 February 2025**, status **Final**, author D. Fett (Authlete) ✅ ([openid.net/specs/fapi-attacker-model-2_0-final.html](https://openid.net/specs/fapi-attacker-model-2_0-final.html)).

Why it exists matters more than its contents list. The Attacker Model states its own rationale: *"The security requirements are expressed through security goals and attacker models. From these requirements, the security mechanisms utilized in the Security Profile are derived."* ✅ That is the inversion at the heart of FAPI 2.0 and the thing vendor material most often misses. FAPI 2.0 is not a feature list assembled from experience; it is a specification *derived* from formally stated goals and an adversary model, then proven against it — the profile "has been proved by formal analysis to meet the stated attacker model" ✅.

### 2.2 The Security Goals (Four, and Only Four)

✅ (§5 of the Attacker Model.) **Authorization:** "**no attacker can access protected resources** other than their own" — met if no attacker can obtain and *use* an access token for resources that are not theirs. **Authentication** (where OIDC is used): "**no attacker is able to log in at a client under the identity of another user.**" **Session integrity, authentication half:** "**no attacker is able to force a user to be logged in under the identity of the attacker.**" **Session integrity, authorization half:** "**no attacker is able to force a user to use resources of the attacker.**" An implementer should test every design decision against these four, not against a vibe.

### 2.3 The Attacker Classes

The Attacker Model deliberately declines to enumerate concrete threats: *"This model does intentionally not define concrete threats… In order to not overlook any potential attacks, FAPI 2.0 therefore aims not to address concrete, narrow threats, but to exclude any attacks conceivable for the attacker types listed here."* ✅ FAPI does not defend against a *list*; it defends against *classes of adversary*, so threats nobody has thought of yet are also covered. ✅ (§7 of the Attacker Model):

| ID | Attacker | Capability |
|---|---|---|
| **A1** | Web attacker | Controls endpoints on the internet; participates in flows as a normal user; uses arbitrary tools on its own endpoints; can send links to honest users. Cannot intercept others' messages or break crypto; **cannot play a legitimate authorization server** (see A1a). |
| **A1a** | Web attacker participating as an authorization server | A1 plus the ability to act as an AS — including replaying messages received from honest ASes and redirecting users to honest AS endpoints. This is the rogue-AS capability that makes mix-up and issuer-confusion defence necessary. |
| **A2** | Network attacker | Controls the network (rogue Wi-Fi, compromised node): can intercept, block and tamper with others' messages, but cannot break crypto. The spec notes most A2-exclusive attacks are defeated by TLS. |
| **A3a** | Attacker at the authorization endpoint — read authorization request | A1 plus the ability to *read* the front-channel authorization request from the browser (mobile URL-scheme registration, browser history, XSS on the AS, TLS-intercepting antivirus). Read-only: an attacker able to read the authorization *response* is not modelled, because current browser technology would undermine most redirect-based protocols. |
| **A4** | Attacker at the token endpoint | A misconfigured-token-endpoint-URL model inherited from FAPI 1.0, **explicitly marked as no longer relevant in FAPI 2.0** ✅ because the token endpoint address must come from authoritative, protected OAuth metadata. Kept "for informative purposes only." |
| **A5** | Attacker at the resource server — read resource requests | A1 plus the ability to read requests after the RS processed them (e.g. TLS-intercepting proxy logs on the RS side). Reduced relative to the analysis version. |

§9 of the Attacker Model gives a reconciliation table between the analysis numbering and the published document: A1, A1a, A2, A3a carry over unchanged; the analysis's A3b is **removed**; analysis A5 maps to document A4; analysis A7 maps to A5 with reduced capabilities; analysis A8 is **removed** ✅ — with the assertion that the analysis results remain valid for the updated model.

### 2.4 What the Attacker Model Explicitly Puts Out of Scope

The honest half of §2, and the part vendor decks omit. ✅ The model assumes correct operation of, and therefore does not defend against failures in: **TLS** ("data integrity and confidentiality are ensured", correct public keys used, private keys unknown to attackers — with the warning that even application-level signing depends on TLS for key distribution, so a major TLS flaw is fatal to everyone); **JWKS / key distribution** ("encryption and signature verification keys of uncompromised parties are retrieved from the correct endpoints" — if key distribution is subverted, the model does not save you); **browsers and devices** (the resource owner's device and browser "are considered not compromised"); and **identity and session management** (end-user identity proofing, authentication and IAM on client or AS are out of scope; clients must protect sessions from each other and validate returned identity attributes against their own requirements).

§8 adds further limitations ✅: secrets are assumed unguessable (weak RNG is out of scope); FAPI "do[es] not prescribe, for example, end-user authentication mechanisms, firewall setups, software development practices, or security aspects of internal architectures"; "Real-world implementations… sometimes deviate from the specified and formally analyzed behavior and contain security vulnerabilit[ies]"; and new browser/technology behaviour can create new vulnerabilities over time. Deployment-level threat modelling is required *in addition to* FAPI.

### 2.5 Threat → Requirement: The Mapping That Matters

The concrete threat names live not in the Attacker Model but in FAPI 1.0 Part 2's security considerations and the FAPI 2.0 Security Profile's §6. This table is the practical core of the guide: threat, the FAPI requirement that defeats it, and the identifier to cite.

| Threat (source) | What the attacker does | FAPI requirement that defeats it | Mechanism |
|---|---|---|---|
| **Authorization request tampering / parameter injection** ✅ (P2 §8.4.2) | Modifies the authorization request in the front channel — injects parameters, changes scope/amount/`redirect_uri` | Request must be integrity-protected end-to-end: a signed request object (**1.0 Advanced**), or the request never travels the front channel at all (**2.0**: PAR) | **RFC 9101** (1.0) / **RFC 9126** PAR (2.0, mandatory) |
| **Authorization response tampering / authorization code injection** ✅ (P2 §8.4.3; P2 Introduction names "authorization response tampering including code injection, state injection") | Injects an attacker's authorization code into a victim client's flow, or swaps response parameters | Signed response (**JARM**, 1.0 Advanced); PKCE binding the code to the client's verifier; `iss` to identify the AS; per-session CSRF binding in 2.0 | **JARM (OIDF spec)**, **RFC 7636** PKCE, **RFC 9207** `iss`, `state` |
| **IdP mix-up attack** ✅ (P2 §8.3.3) | A malicious AS makes the client send the code (or a client assertion) to the wrong AS | `iss` in the authorization response that the client MUST verify (**RFC 9207**, mandatory in FAPI 2.0); PKCE so the code is useless to the wrong party; in 2.0, PAR + metadata from an authoritative source so the AS set is not attacker-controllable | **RFC 9207**, **RFC 7636**, **RFC 9126**, **RFC 8414** |
| **Client credential and authorization code phishing at the token endpoint** ✅ (P2 §8.3.2) | Social-engineers the developer into believing the token endpoint changed, then replays the code/secret | `private_key_jwt` or mTLS client authentication (no shared secret to phish); token endpoint address from signed metadata; PKCE single-use code binding; short code lifetime (2.0 mandates a **maximum 60-second** code lifetime ✅) | OIDC Core §9, **RFC 8705**, **RFC 8414**, **RFC 7636** |
| **Access token phishing / token theft and replay** ✅ (P2 §8.3.5) | Steals or phishes a bearer token and replays it | Sender-constrained tokens bound to a key the attacker does not hold — mTLS certificate-bound (**RFC 8705**) or DPoP-bound (**RFC 9449**). Mandatory in FAPI 2.0 ✅ | **RFC 8705** / **RFC 9449** |
| **Cuckoo's Token Attack / unauthorized token injection** ✅ (P2 §8.3.5, citing [FAPISEC]) | Tricks a client under the attacker's control into using a stolen token; requires a rogue AS or misconfigured token endpoint | Combination defence: sender-constrained tokens + issuer identification + authoritative metadata (closing the misconfiguration route) | **RFC 9207**, **RFC 8414**, token binding |
| **Session swapping / forced use of the attacker's resources (CSRF class)** ✅ (AM2 §5.4) | Forces the victim's client session to operate against the attacker's account, or logs the victim in as the attacker | Per-session `state`/nonce handling; PKCE; authorization code bound to the DPoP key when DPoP is used (**RFC 9449 §10.1**, mandatory in 2.0 ✅); no front-channel token exposure | **RFC 7636**, **RFC 9449** §10.1 |
| **Browser swapping / redirection abuse** ✅ (SP2 §6.5) | Intercepts a redirection and forwards the URL to a victim (phishing page, e-mail, QR code) | PKCE and PAR reduce the front channel's value; the profile documents the residual browser-swapping class in its security considerations; signing does not eliminate it | **RFC 7636**, **RFC 9126**; residual risk documented |
| **DPoP proof replay** ✅ (SP2 §6.2) | Replays captured DPoP proofs | Short-lived DPoP nonces; `jti`-based replay prevention; signed resource requests prevent replay of an *altered* request | **RFC 9449** §8/§11.1; FAPI 2.0 Message Signing |
| **Injection of stolen access tokens** ✅ (SP2 §6.3) | Presents a stolen token to a resource server | Sender-constraining verified at the RS: the certificate thumbprint (`cnf`/`x5t#S256`) or the DPoP key binding is checked before the token is honoured | **RFC 8705** §3, **RFC 9449** |
| **Consent tampering / scope escalation** | Alters what the user consented to (amount, payee, account, duration) between consent capture and API access | Request integrity + PAR (the request is immutable once pushed) + RAR `authorization_details` for explicit structured consent + grant management for lifecycle. FAPI guarantees the *security* of the request; the *content* of consent is the ecosystem's business (§9, §11.4) | **RFC 9126**, **RFC 9101**, **RFC 9396**, Grant Management |
| **Redirect manipulation / open redirector abuse** ✅ (SP2 §5.3.2.1(7)) | Uses an open redirector on the AS or client to exfiltrate codes/tokens | AS "shall not expose open redirectors"; exactly matching redirect URIs in 1.0 Advanced; PAR + JARM reduce front-channel trust | **RFC 9700** §4.11 (BCP 240); 1.0 Part 2 §5.2.2 |
| **Network eavesdrop / MITM on token or resource calls** | Reads or tampers with server-to-server traffic | TLS 1.2+ mandatory; BCP 195 cipher suites; **RFC 9525** server certificate check; DNSSEC/CAA recommended; mTLS ecosystems encouraged | BCP 195, **RFC 9525**, **RFC 8659**, **RFC 8705** |

Three rules follow for a design review. **(1)** Every row must name the artefact that carries the defence — "we use PKCE" is not a control unless you can point at the code that rejects a mismatched verifier. **(2)** Rows that depend on the ecosystem are marked as such: consent tampering, the central risk in money movement, is only partly a FAPI problem. **(3)** The out-of-scope list in §2.4 is a to-do list for someone else.

---

## 3. Why Plain OAuth 2.0 Is Not Enough

### 3.1 What a Competent OAuth 2.0 Deployment Already Gives You

Be fair to OAuth 2.0 before explaining why FAPI exists. RFC 6749 (October 2012) plus its ecosystem already provides delegated authorization without password sharing; scoped access tokens; a redirect-based authorization code flow; refresh tokens; a revocation path; authenticated client registration; and, with RFC 7636 (PKCE, September 2015), defence against authorization-code interception for public clients. A well-run OAuth 2.0 deployment is not insecure by default. The FAPI specifications themselves say their audience is "parties who already have a certified implementation of OpenID Connect and want to achieve a higher level of security" ✅ — FAPI is a step up from a working deployment, not a replacement for a broken one.

### 3.2 The Residual Exposures

The exposures that remain are structural, not implementation sloppiness. RFC 9101's abstract states the authorization-request problem exactly, and it is the cleanest primary-source statement of why the front channel is the weak point ✅:

> "The authorization request in OAuth 2.0 described in RFC 6749 utilizes query parameter serialization… While it is easy to implement, it means that a) the communication through the user agents is not integrity protected and thus, the parameters can be tainted, b) the source of the communication is not authenticated, and c) the communication through the user agents can be monitored." — RFC 9101, Abstract ✅

| Residual exposure in plain OAuth 2.0 | Why it matters when the resource is money |
|---|---|
| **Bearer tokens are possession-based** (RFC 6750) | Anyone who obtains the token string can use it. Phishing, log leakage or over-broad log capture converts directly into account access; the token has no cryptographic link to the client that requested it. |
| **The front channel is not integrity-protected** | The authorization request travels through the user's browser. Parameters — scope, amount, payee, `redirect_uri` — can be tampered with in transit through the user agent, which no amount of edge TLS fixes. |
| **The authorization response is likewise unprotected** | Code injection and response parameter injection are live attack classes (FAPI 1.0 Part 2 lists them ✅). |
| **Redirect flexibility** | Open redirectors, loose redirect-URI matching and wildcard registration hand an attacker a code-exfiltration path. RFC 9700 §4.11 tells implementers not to expose open redirectors; FAPI 2.0 makes that normative ✅. |
| **Weak client authentication remains legal** | RFC 6749 permits shared-secret client authentication and even the resource owner password credentials grant. FAPI 2.0 forbids the password grant outright and permits only `private_key_jwt` or mTLS ✅. |
| **Endpoint discovery can be attacker-controllable** | If the client learns the token endpoint from configuration rather than protected metadata, the misconfigured-token-endpoint attack (AM2's A4) works. FAPI 2.0 closes this by requiring metadata from an authoritative source over a secure channel ✅, and AM2 marks A4 obsolete for that reason ✅. |
| **Optionality itself** | OAuth 2.0 is a *framework* that intentionally leaves choices open; two compliant implementations can differ in ways that break interoperability and occasionally security. The WG's value proposition is exactly this: "The standards produced by the FAPI WG contain much less optionality than the general OAuth 2.0 framework and require implementers to use modern security best practices." ✅ |
| **Screen scraping as the status quo** | The alternative FAPI was built to displace: aggregators storing user passwords, "a brittle, inefficient, and insecure practice [that] creates security vulnerabilities which require financial institutions to allow what appears to be an automated attack against their applications." ✅ |

### 3.3 Why the Sector Profiled Rather Than Replaced

Three reasons, all visible in the primary text. **(1) OAuth 2.0's authors left the hardening to profiles** — RFC 6749 is a framework precisely so security-critical sectors can narrow it, and RFC 9700 (BCP 240, January 2025) is the IETF's own best-current-practice, which FAPI 2.0 explicitly "follows" ✅. **(2) A separate protocol would have lost the ecosystem** — banks already had OpenID Connect deployments, vendor products and integration teams, and the FAPI specs say implementers "are encouraged to build on top of existing OpenID Connect and/or OAuth 2.0 implementations instead of embarking on a 'from scratch' implementation" ✅. **(3) Profiling makes the requirement testable** — the WG's stated benefits are "Clear point-by-point specifications that implementers can use as a 'check list'" and "Exhaustive conformance tests" ✅; a profile with numbered `shall` clauses can be tested by a machine and audited by a supervisor, a general security posture cannot.

### 3.4 The Honest Contrast: the EU's eIDAS/QWAC Route

There is a second way to solve the same problem, frequently conflated with FAPI. In Europe, PSD2 (Directive (EU) 2015/2366) combined with the EBA's Regulatory Technical Standards on strong customer authentication and common and secure open standards of communication (the "RTS on SCA & CSC") took a *regulatory* route: TPPs are licensed and supervised, and their identification is anchored in eIDAS qualified certificates — the QWAC (Qualified Website Authentication Certificate) — with mutual TLS using such certificates for the XS2A interface. ⚠ **This paragraph reflects the well-documented shape of the EU regime; the specific RTS article numbers and QWAC mechanics were NOT re-verified at EBA/EUR-Lex primary sources in this pass — see §14.** Treat it as structural knowledge, not a verified quotation.

| Dimension | EU / PSD2 + eIDAS route | FAPI route |
|---|---|---|
| **Who authenticates whom** | Regulator-supervised identity: TPP identity proved by an eIDAS-qualified certificate issued under a supervised trust-service regime | Ecosystem-supervised identity: the scheme's directory/trust framework registers clients, key material and certificates; FAPI assumes this exists but does not define it |
| **Certificate quality** | Qualified (QWAC) under eIDAS — a legal category with a supervisory backstop | Any certificate the ecosystem trusts; FAPI 2.0 says MTLS ecosystems "should provide the trust list of the certificate authorities" ✅ — the trust list is an ecosystem artefact |
| **Request integrity** | Achieved through the certified channel and the regulatory identity of the caller | Achieved cryptographically: JAR (1.0 Advanced) or PAR (2.0) |
| **What FAPI adds** | — | Sender-constrained tokens, PAR, PKCE everywhere, `iss` issuer identification, a formal attacker model |
| **What the EU route adds** | Legal identity and supervision of the counterparty, a liability regime, a licence requirement | — |

**These are complementary, not competing.** A European ASPSP implementing PSD2 today often runs eIDAS-based mTLS for TPP identification *and* a FAPI-derived profile for the OAuth layer — the two stack, which is precisely what the UK does when its Directory issues "PSD2… compliant certificates" for FAPI's transport and message-signing needs ✅. FAPI's contribution is not identity assurance for the institution; it is cryptographic assurance for the flow. That is why the identity-assurance tie-in (§6.9) is a separate, still-maturing layer rather than part of the core profile.

---

## 4. FAPI 1.0 — Part 1 Baseline and Part 2 Advanced

### 4.1 The Shape of the 1.0 Generation

FAPI 1.0 is **two documents**, and the split is by *what the API does*, not by how strong the security is:

| Document | Exact title | Purpose | Inherent risk |
|---|---|---|---|
| **Part 1** | *Financial-grade API Security Profile 1.0 - Part 1: Baseline* ✅ | "a baseline security profile of OAuth that is suitable for protecting APIs with a **moderate** inherent risk" ✅ | Read-only data access |
| **Part 2** | *Financial-grade API Security Profile 1.0 - Part 2: Advanced* ✅ | "an advanced security profile of OAuth that is suitable to be used for protecting APIs with **high** inherent risk. Examples include APIs that give access to highly sensitive data or that can be used to trigger financial transactions (e.g., payment initiation)" ✅ | Read *and* write, incl. payments |

Part 1 is explicit that it is the weaker of the two ✅: "this profile does not provide non-repudiation (signing of authorization requests and responses) and sender-constrained access tokens. If such features or a higher level of security is desired, the use of [Part 2: Advanced] is recommended." Historically the parts carried the descriptive names **Read-Only API Security Profile** (Part 1) and **Read and Write API Security Profile** (Part 2) — those names survive in the FAPI 1.0-era JARM draft, which listed the FAPI parts as "Part 1: Read-Only API Security Profile; Part 2: Read and Write API Security Profile; Client Initiated Backchannel Authentication Profile" ✅. Both naming schemes are in circulation; the *final* published titles use Baseline and Advanced.

### 4.2 Publication History and Current Status

✅ Verified on the OpenID Foundation's own FAPI Working Group page (read 17 September 2026):

| Milestone | Date |
|---|---|
| First Implementers Draft | **July 2017** |
| Second Implementers Draft | **October 2018** |
| Conformance testing launched | **April 2019** |
| **Final** | **March 2021** |

The WG page states: *"The final version of the 1.0 specifications were published in March 2021."* ✅ Note the order of events: the conformance-testing programme (April 2019) preceded final publication by roughly two years — in FAPI-land the tests came first and ecosystems certified against drafts.

**Current status (September 2026):** ✅ **Final, maintained, frozen, still widely deployed — not superseded, not withdrawn.** *"Will FAPI 1.0 be maintained"* → *"Yes, if there are any security issues, or major interoperability issues found with FAPI 1.0 the working group is likely to update the FAPI 1.0 specs. However the specs have been used in production in multiple ecosystems for some time, so the working group does not expect many (if any) changes to FAPI 1.0."* ✅ And *"No new features are planned to be added to FAPI 1.0."* ✅

**Negative finding, stated as such:** the individual spec pages for Part 1 and Part 2 do not print a per-part publication date in the header block, and this pass did not locate an OpenID Foundation page giving separate Part-1 and Part-2 dates. The March 2021 date is verified for "the 1.0 specifications" collectively ✅ and is independently corroborated by the Australian CDR normative references, which date both parts March 2021 ✅ — but that is still not a per-part OIDF statement.

### 4.3 Part 1 Baseline — What It Mandates

✅ (FAPI 1.0 Part 1 §5.2.2, read at source.)

| Area | Baseline requirement ✅ |
|---|---|
| **Client types** | AS shall support confidential clients; *should* support public clients |
| **Client authentication** | mTLS for OAuth client authentication (**RFC 8705** §2), **or** `client_secret_jwt` / `private_key_jwt` (OIDC Core §9); symmetric secrets permitted if they meet OIDC §16.19 |
| **Key sizes** | RSA **≥ 2048 bits**; elliptic curve **≥ 160 bits** |
| **PKCE** | **Shall** require **RFC 7636** with **`S256`** as the code challenge method |
| **Redirect URIs** | Pre-registered; `redirect_uri` required in the request; its value must **exactly match** a pre-registered value; must use the **https** scheme |
| **Replay** | AS shall reject an authorization code that has been previously used |
| **Token entropy** | Non-guessable access tokens, codes and refresh tokens with sufficient entropy per RFC 6749 §10.10 |
| **Access token lifetime** | Servers *should* issue access tokens with a lifetime of **under 10 minutes** unless the tokens are sender-constrained |
| **Discovery** | Shall support OpenID Connect Discovery (OIDD); *may* support RFC 8414; **shall not** distribute discovery metadata by any other means |
| **User authentication & consent** | Shall require user authentication to an appropriate Level of Assurance for the operations to be authorized; shall require explicit user approval to authorize the requested scope if not previously authorized |
| **Grant transparency** | Shall return the granted scopes with the access token if the request travelled the front channel without integrity protection — deliberately, so "clients [can] detect when the authorization request was modified to include different scopes" ✅ |
| **CSRF / state** | Client requesting `openid` → AS shall require `nonce`; client not requesting `openid` → AS shall require `state` |
| **Public clients** | Shall support PKCE with `S256`; separate and distinct redirect URI per authorization server; store the redirect URI in the user agent's session and compare; effective CSRF protection; only use AS metadata from the well-known endpoint |
| **What Baseline explicitly lacks** ✅ | **No request object (JAR) requirement — the authorization request has no message integrity protection at all** ("request tampering and parameter injection are possible"), and **no sender-constrained access tokens** |

That last row is the whole reason Part 2 exists.

### 4.4 Part 2 Advanced — What It Adds

✅ Reading FAPI 1.0 Part 2 §5.2.2–§5.2.3 at source:

| Area | Advanced requirement ✅ |
|---|---|
| **Inherits Baseline** | AS shall support Part 1 §5.2.2 provisions — **except** Part 1 §5.2.2-7 (mandatory PKCE) is not required |
| **Request object (JAR)** | AS **shall require** a JWS-signed JWT request object, by value (`request`) or by reference (`request_uri`) — **RFC 9101** |
| **Permitted response types** | AS shall require either **`code id_token`**, or **`code` with `response_mode=jwt`** (JARM). Nothing else |
| **Sender-constrained tokens** | AS **shall only issue sender-constrained access tokens** and **shall support mTLS** (**RFC 8705**). The spec's NOTE: *"MTLS is currently the only mechanism for sender-constrained access tokens that has been widely deployed. Future versions of this specification are likely to allow other mechanisms"* ✅ |
| **Parameter provenance** | AS **shall only use the parameters included in the signed request object** — anything outside the signed object is disregarded |
| **Request object claims** | `aud` shall be (or contain) the OP's Issuer Identifier URL; `exp` present with a lifetime **no longer than 60 minutes** after `nbf`; `nbf` no longer than 60 minutes in the past |
| **Client authentication** | Advanced **overrides** Baseline: only `tls_client_auth` or `self_signed_tls_client_auth` (**RFC 8705** §2) **or** `private_key_jwt` (OIDC Core §9). `client_secret_jwt` — permitted in Baseline — is out |
| **Public clients** | AS **shall not support public clients** |
| **PAR** | AS *may* support PAR; if PAR is supported, PKCE with `S256` is required (§5.2.2-18) |
| **ID Token as detached signature** | When `code id_token` is used: AS shall support OIDC and signed ID Tokens; **shall return the ID Token as a detached signature to the authorization response**; **shall include `s_hash`** (state hash) in the ID Token when the client supplied `state` |
| **JARM** | When `code` is used with `response_mode=jwt`, the AS **shall create JWT-secured authorization responses** per JARM §4.3 ✅ |
| **Client-side duties** | The confidential client shall send **all** parameters inside the signed request object; when *not* using PAR it shall additionally send duplicate `response_type`, `client_id` and `scope` in OAuth 2.0 request syntax per OIDC §6.1; `aud` = OP issuer; `exp` ≤ 60 min; `nbf` present; PKCE `S256` if using PAR |

### 4.5 What Advanced Costs an Implementer — the Honest Note

Advanced is *four cryptographic mechanisms deep before a single API call happens*. To complete one authorization in FAPI 1.0 Advanced an implementation must (1) **generate and sign a request object** carrying `aud`, `exp`, `nbf` and every request parameter, with the ±60-minute windowing the AS enforces ✅; (2) **transmit it by value or reference** and duplicate three parameters in plain OAuth syntax when PAR is not used ✅; (3) **receive either an `id_token` used as a detached signature with `s_hash`** validating the `state`, or **a JARM JWT-secured response** ✅; and (4) **authenticate with a client certificate or `private_key_jwt`**, then present a **certificate-bound access token** on every resource call ✅.

That is the complexity the working group later unwound, and the evidence is in the primary text. FAPI 2.0's §5.5 table — the WG's own diff — describes the 1.0 defences as "**BCM principles, defences based on particular threats**" and 2.0 as "**attacker model, security goals, best practices from the OAuth Security BCP**" for "clearer design guideline, suitability for formal analysis" ✅. In other words, the WG concluded that 1.0 Advanced had been assembled *threat by threat* rather than derived from a model. The practical costs that followed (**⚠ attributed** — this is the widely-argued implementer position, consistent with the WG's own "simpler to implement" objective, stated as an attributed assessment rather than a spec quotation): **signing key custody for every client** (JAR moves a private key into the client estate, so every TPP needs an HSM or equivalent, rotation discipline and JWKS publication — the single largest barrier to entry in the 1.0 Advanced model); **`s_hash` and detached-signature ID tokens** as an unusual construct with a narrow failure mode; **two response-type paths** meaning two AS implementations and two conformance paths for the same intent; **client-authentication asymmetry** (`client_secret_jwt` legal at Baseline, forbidden at Advanced); and the **`x-fapi-*` headers**, ecosystem conventions that read as though they were part of the profile — FAPI 2.0 explicitly **moved them out** into a separate Implementation and Deployment Advice document as "not relevant to the core of the security profile" ✅.

---

## 5. FAPI 2.0 — Security Profile, Message Signing, and What Was Simplified

### 5.1 The Three Documents and Their Exact Status

This is where secondary sources are most often wrong, so each is stated with the publication line read off its own page ✅ (all read 17 September 2026):

| Document | Published | Status | URL |
|---|---|---|---|
| **FAPI 2.0 Security Profile** | **22 February 2025** | **Final** ✅ | [openid.net/specs/fapi-security-profile-2_0-final.html](https://openid.net/specs/fapi-security-profile-2_0-final.html) |
| **FAPI 2.0 Attacker Model** | **22 February 2025** | **Final** ✅ | [openid.net/specs/fapi-attacker-model-2_0-final.html](https://openid.net/specs/fapi-attacker-model-2_0-final.html) |
| **FAPI 2.0 Message Signing** | **25 September 2025** | **Final** ✅ | [openid.net/specs/fapi-message-signing-2_0-final.html](https://openid.net/specs/fapi-message-signing-2_0-final.html) |

Cross-checks and honest notes. The WG's news banner announces **"FAPI 2.0 is now approved as FINAL specification (2025-02-19)"** ✅, approving the Security Profile and Attacker Model together; the individual pages print **22 February 2025**. Both are correct for what they measure (approval vs publication); this guide uses the printed publication date. **Message Signing is Final as of 25 September 2025 — later than the Security Profile, and this is the fact most often missed**; it went through Implementers Draft 1 in January 2024 ✅ before reaching Final. ⚠ **One inconsistency, flagged rather than smoothed over:** the Message Signing page's header reads "Status: Final, Published: 25 September 2025", but its body retains the draft boilerplate *"This document is not an OIDF International Standard. It is distributed for review and comment."* The same boilerplate does **not** appear on the Security Profile or Attacker Model pages. Most likely an editorial carry-over — but the artefact-level contradiction is real and is recorded, not resolved by assertion. ⚠ **Conformance tests for FAPI 2.0 were launched in March 2023** ✅ — *before* the specifications were Final — and the first FAPI 2.0 self-certifications were published, with the WG congratulating **Authlete, Cloudentity, ConnectID, Ping Identity and Raidiam** ✅ and noting ConnectID in Australia "adopted FAPI 2.0 for their ecosystem and funded the FAPI 2.0 conformance test suite development."

### 5.2 Is There a "FAPI 2.0 Baseline"?

Yes — and the artefact structure and the level structure are different things. **Artefact structure:** one core specification, *FAPI 2.0 Security Profile*, plus *FAPI 2.0 Message Signing* as an add-on. There is no document titled "FAPI 2.0 Baseline" or "FAPI 2.0 Advanced". **Level structure:** the WG describes two compliance levels aligned to protection levels: *"Both FAPI 1.0 as well as FAPI 2.0 define two compliance levels, but the FAPI 2.0 levels are aligned with different protection levels (baseline vs advanced) rather than API access modes (read vs read-write) in FAPI 1.0. The baseline level aims to be secure against all threats captured in the security threat model, the advanced level adds non-repudiation."* ✅ So in practice: **FAPI 2.0 Baseline = the Security Profile; FAPI 2.0 Advanced = Security Profile + Message Signing.** The WG's own comparison ✅:

| | Security level | Non-repudiation |
|---|---|---|
| **FAPI 1.0 Baseline** | Medium | None |
| **FAPI 1.0 Advanced / FAPI 2.0 Baseline** | High | Limited |
| **FAPI 2.0 Advanced** | High | Comprehensive |

That table is the most useful sentence in the whole comparison literature: **FAPI 2.0 Baseline protects against a similar attacker model to FAPI 1.0 Advanced** ✅ while being simpler to implement. An institution on 1.0 Advanced gets comparable security from 2.0 Baseline and only goes to 2.0 Advanced if it needs non-repudiation across all exchanges.

### 5.3 What FAPI 2.0 Actually Depends On — and the OAuth 2.1 Question

The Security Profile's normative references are narrower and more concrete than the "OAuth 2.1" shorthand that circulates ✅: **RFC 6749** (OAuth 2.0, October 2012), **RFC 6750** (Bearer Token Usage), **RFC 7636** (PKCE, September 2015), **RFC 8705** (mTLS + certificate-bound tokens, February 2020), **RFC 9449** (DPoP, September 2023), **RFC 9126** (PAR, September 2021), **RFC 8414** (AS Metadata, June 2018), **RFC 9207** (Issuer Identification, March 2022), **OpenID Connect Core 1.0** inc. errata set 1, **RFC 9700 / BCP 240** (OAuth Security BCP, January 2025 — which FAPI 2.0 "follows the recommendations in" ✅), **RFC 8725 / BCP 225** (JWT BCP), plus **RFC 9525**, **BCP 195**, **RFC 8659** and **RFC 6797** for transport.

**FAPI 2.0 does *not* normatively depend on OAuth 2.1.** It depends on **RFC 6749 plus RFC 9700**. OAuth 2.1 consolidates the security BCP and the widely-used extensions into a single document; as of this pass it remains an **Internet-Draft — `draft-ietf-oauth-v2-1`, most recently at revision `-16`** — and ⚠ **was not verified as a published RFC**. The IETF datatracker refused to serve content to this pass's extraction tooling (§14), so the revision number and draft-not-RFC status rest on search-result metadata plus a secondary source stating it "has not been published as a final RFC as of July 2026." **Treat "is OAuth 2.1 an RFC yet?" as ⚠ flagged and re-check the datatracker before citing it.** What is safe: an institution implementing FAPI 2.0 is implementing the numbered RFCs above, not "OAuth 2.1".

### 5.4 What FAPI 2.0 Mandates

✅ Read from the Security Profile's §5.2–§5.4.

**Transport.** TLS-protected endpoints only, **TLS 1.2 or later**, following **BCP 195**, with a server certificate check per **RFC 9525**; DNSSEC *should* be used. Server-to-server endpoints may only use BCP 195 *recommended* cipher suites (browser-facing endpoints may use any BCP 195 *allowed* suite); browser-facing endpoints must prevent TLS stripping (HSTS preload) and **must not** support CORS on the authorization endpoint ✅. Implementors are expected to become compliant with new BCP 195 versions within **12 months** ✅. MTLS ecosystems should provide the CA trust list; a new `use_mtls_endpoint_aliases` client metadata parameter is introduced ✅.

**Authorization server.** Shall **reject** the resource owner password credentials grant; shall **only support confidential clients** (the WG "is not currently aware of any mechanisms that would allow public clients to be secured to the same degree", so public clients are out of scope ✅); shall **only issue sender-constrained access tokens**, using **mTLS (RFC 8705)** or **DPoP (RFC 9449)** ✅; shall authenticate clients only via **mTLS** (RFC 8705 §2) or **`private_key_jwt`** (OIDC Core §9) ✅; shall not expose open redirectors; shall accept only its own issuer identifier in the `aud` of client assertions; shall **not use refresh token rotation except in extraordinary circumstances**; shall issue authorization codes with a **maximum lifetime of 60 seconds** ✅; if using DPoP *may* use server-provided nonces but **shall** support "Authorization Code Binding to DPoP Key" (RFC 9449 §10.1) ✅; and, for clock skew, **shall accept** `iat`/`nbf` between 0 and 10 seconds in the future and **shall reject** anything beyond 60 seconds in the future ✅.

**Client.** Shall use the authorization code grant; **shall use PAR (RFC 9126)**; **shall use PKCE with `S256`**; shall generate the PKCE challenge per request and bind it to the client and user agent; **shall check the `iss` parameter** per **RFC 9207** "to prevent mix-up attacks" ✅; **shall send only `client_id` and `request_uri` to the authorization endpoint** ✅ (a striking simplification of the front channel — every other parameter goes in the pushed request); shall obtain the issuer URL from an authoritative source over a secure channel and verify it matches the metadata `issuer`; *should* keep `nonce` values ≤ **64 characters** for interoperability; *should* request least privilege.

**Resource server.** Shall accept tokens **in the HTTP header only** — **shall not** accept them as query parameters ✅; shall verify validity, integrity, expiration and **revocation status**, and that the authorization represented is sufficient for the requested resource; **shall support and verify sender-constrained tokens** via mTLS or DPoP ✅.

**Cryptography.** JWTs must adhere to **RFC 8725** and use **`PS256`, `ES256`, or `EdDSA` (Ed25519 variant)** — and **never `none`** ✅ (the spec notes there was no fully-specified IANA registration for "EdDSA using Ed25519" at time of writing). RSA keys **≥ 2048 bits**; EC keys **≥ 224 bits**; non-user-handled credentials **≥ 128 bits of entropy** ✅. JWKS endpoints over TLS only; *should not* use `x5u`/`jku`; *should not* serve duplicate `kid`s; verifiers **shall** disambiguate same-`kid` keys using `alg`, `use`, `kty` or `crv` ✅.

### 5.5 What FAPI 2.0 Deliberately Simplified or Dropped — the Analytical Core

The WG's own diff table, reproduced because it is the most illuminating artefact in the FAPI corpus ✅ (Security Profile §5.5, "Main differences to FAPI 1.0"):

| FAPI 1.0 - Part 2: Advanced | FAPI 2.0 | Reason given by the WG |
|---|---|---|
| JAR | **PAR** | "integrity protection and compatibility improvements for authorization requests" |
| JARM | **only code in response** | "the authorization response is reduced to only contain the authorization code, obsoleting the need for integrity protection" |
| BCM principles, defences based on particular threats | **attacker model, security goals, best practices from the OAuth Security BCP** | "clearer design guideline, suitability for formal analysis" |
| `s_hash` | **PKCE** | "protection provided by `state` (in particular against CSRF) is now provided by PKCE; `state` integrity is partially protected by PAR" |
| pre-registered redirect URIs | **redirect URIs in PAR** | "pre-registration is not required with client authentication and PAR" |
| response types `code id_token` or `code` | **response type `code`** | "no ID token in front-channel (privacy improvement); nonce/signature check can be skipped by clients, PKCE cannot (security improvement)" |
| ID Token as detached signature | **PKCE** | "ID token does not need to serve as a detached signature" |
| potentially encrypted ID Tokens in the front channel | **No ID Tokens in the front channel (therefore no encryption required)** | "ID Tokens are only exchanged in the back channel and as such do not need to be encrypted" |
| `nbf` & `exp` claims in request object | **`request_uri` has limited lifetime** | "Prevents pre-generation of requests" |
| `x-fapi-*` headers | **Moved to Implementation and Deployment Advice document** | "Not relevant to the core of the security profile" |
| MTLS for sender-constrained access tokens | **MTLS or DPoP** | "Due to the lack of the tight integration with the TLS layer, DPoP can be easier to deploy in some scenarios" |

Read down the left column: Advanced accumulated *four* distinct integrity-protection constructs — JAR for the request, JARM for the response, ID-token-as-detached-signature with `s_hash` for the state, and mTLS for the token. FAPI 2.0 replaced that pile with **PAR + PKCE + one response type + a choice of token binding**. The reasoning is explicit and it is *not* "the old way was insecure": several mechanisms were doing overlapping jobs, redundancy across a front channel creates implementation surface rather than defence in depth, and a construct you must implement correctly to be safe is itself a vulnerability surface. The WG says the 2.0 work was driven by "an analysis of various open banking implementations, the recommendations of the latest OAuth Security BCP, and a comprehensive security threat model" ✅ — i.e. by watching what implementers actually got wrong. **What 2.0 did NOT drop is the security goal:** Baseline targets "a similar attacker model as FAPI 1.0 Advanced" ✅, and sender-constraining remains mandatory. What was traded away is **non-repudiation**, which 2.0 unbundles into the separate Message Signing profile rather than baking into the core flow.

### 5.6 Migration and Coexistence for an Institution Already on 1.0

The WG's answers, verbatim ✅. **Backward compatibility: none, formally** — *"There are similarities between FAPI 1.0 and FAPI 2.0 (e.g., response type code + PKCE in FAPI 1.0/read and FAPI 2.0/baseline) but the scope is different so there is no full backwards compatibility."* **Coexistence on one AS: plausible** — *"This may be possible. FAPI 1.0 Advanced allows the use of PAR and PKCE. These are both required by FAPI 2.0."* ✅ **Whether to migrate: ecosystem-specific** — *"Should all FAPI 1.0 ecosystems migrate to FAPI 2.0? … this is an ecosystem specific decision"* ✅, with the WG noting that vendor support, maturity, and 2.0's coverage of "complex authorization requests and grant lifecycle management" are the deciding factors ✅.

**A practical migration shape** (derived from the above, unmarked, not asserted by any spec): for an institution running 1.0 Advanced, add **PAR** and **PKCE** to the existing server first — both are already permitted by 1.0 Advanced ✅ — then make **DPoP** available alongside mTLS, then expose a FAPI 2.0 conformance profile per client configuration. That is a per-client profile selection, not a big-bang cutover, which is why "can one AS do both?" matters more in practice than "which is better?". The hardest migration costs are dropping `code id_token` for clients that depended on front-channel ID tokens, and re-registering redirect URIs where pre-registration was the only control, since 2.0 moves that to PAR. ⚠ **No OpenID Foundation-published migration guide was located in this pass** (§14).

---

## 6. The Mechanisms, One by One

Each mechanism below states its **identifier** (verified at source in this pass), the **problem it solves**, **where it is mandatory vs optional**, and the **operational cost**. The identifiers are the part most commonly mis-cited in vendor material.

### 6.1 Pushed Authorization Requests (PAR) — RFC 9126 ✅

*OAuth 2.0 Pushed Authorization Requests*, **RFC 9126**, Standards Track, **September 2021** ✅. **Problem:** the authorization request's journey through the user's browser — the client POSTs the parameters directly to the AS's PAR endpoint over an authenticated TLS channel and receives a `request_uri`, so the browser carries only `client_id` and `request_uri`. **Buys:** integrity protection for every request parameter before any front channel is involved; the AS sees the request before the user does, so it can validate and log it. **Mandatory vs optional:** **mandatory in FAPI 2.0** ✅ ("shall use pushed authorization requests"; "shall only send `client_id` and `request_uri`"); **optional in FAPI 1.0 Advanced** ✅ (the AS "may support" PAR — and if it does, PKCE `S256` becomes mandatory ✅). **Cost:** an extra AS round-trip; short-lived pushed-request state; a new security-critical endpoint needing authentication and rate limiting. The compensation is a pre-flight validation hook where a hostile request can be rejected before the customer sees a consent screen.

### 6.2 JWT-Secured Authorization Request / signed request objects (JAR) — RFC 9101 ✅

*The OAuth 2.0 Authorization Framework: JWT-Secured Authorization Request (JAR)*, **RFC 9101**, Standards Track, **August 2021** ✅. **Problem:** the same front-channel integrity problem as PAR, solved cryptographically — parameters travel as a JWS-signed JWT, by value in `request` or by reference in `request_uri` ✅, attaining "the integrity, source authentication, and confidentiality properties of the authorization request" ✅. **Buys:** source authentication (the AS can prove *which client* assembled the request), parameter integrity, optional confidentiality via JWE, and — per RFC 9101 ✅ — the ability to have requests "prepared by a trusted third party so that a client application cannot request more permissions than previously agreed upon." **Mandatory vs optional:** **mandatory in FAPI 1.0 Advanced** ✅; **superseded by PAR in FAPI 2.0** ✅ and not required there. **Cost:** every client needs a signing key, an HSM or equivalent, rotation discipline and JWKS publication — the single largest barrier to entry in the 1.0 Advanced model, and a principal reason the WG replaced it with PAR. It is not *forbidden* in 2.0, and the two compose cleanly (a signed object pushed via PAR).

### 6.3 Mutual TLS and certificate-bound access tokens — RFC 8705 ✅

*OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens*, **RFC 8705**, Standards Track, **February 2020** ✅. **Problem, two in one:** as **client authentication** (§2) it replaces a shared secret with a client certificate — nothing to phish; as **sender-constraining** (§3) the issued token carries a confirmation claim (`cnf`) binding it to the certificate's thumbprint (`x5t#S256`), so a stolen token is useless without the private key and a completed TLS handshake. **Variants used by FAPI:** `tls_client_auth` and `self_signed_tls_client_auth` ✅. **Mandatory vs optional:** **mandatory in FAPI 1.0 Advanced** ✅ ("shall only issue sender-constrained access tokens; shall support MTLS"); in **FAPI 2.0** sender-constraining remains mandatory but mTLS is **one of two permitted mechanisms** ✅ and one of the two permitted client-auth methods ✅. **Cost:** the highest of any mechanism here — certificate issuance, custody, renewal, revocation and trust-list distribution are ecosystem-scale problems, which is why FAPI 2.0 says MTLS ecosystems "should provide the trust list of the certificate authorities" ✅ and introduces endpoint aliasing ✅. Certificate expiry is the classic production outage (§7.5).

### 6.4 Demonstrating Proof of Possession (DPoP) — RFC 9449 ✅

*OAuth 2.0 Demonstrating Proof of Possession (DPoP)*, **RFC 9449**, Standards Track, **September 2023** ✅. **Problem:** the same token-theft problem as mTLS, at the application layer — the client generates an asymmetric key pair, sends a signed DPoP proof JWT with each request, and the token is bound to that key's public thumbprint. **Mandatory vs optional:** optional in FAPI 1.0 (which permits only mTLS); **one of the two permitted mechanisms in FAPI 2.0** ✅. **DPoP vs mTLS — the honest comparison.** The WG's own rationale for adding DPoP: "Due to the lack of the tight integration with the TLS layer, DPoP can be easier to deploy in some scenarios" ✅.

| Dimension | mTLS (RFC 8705) | DPoP (RFC 9449) |
|---|---|---|
| **Where the binding lives** | TLS handshake — transport layer | Per-request signed header — application layer |
| **What the attacker must steal** | Token *and* private key *and* ability to complete the handshake | Token *and* private key |
| **Certificate infrastructure** | Required: CA trust lists, issuance, renewal, revocation ✅ | Not required — no PKI, no trust list |
| **TLS termination** | Breaks under TLS offload unless handled carefully | Survives TLS termination — the proof is an HTTP header |
| **Replay surface** | Out of scope of the mechanism itself | Explicit — FAPI 2.0 documents proof replay and prescribes nonces, `jti` replay prevention, and signed requests against altered-request replay ✅ |
| **Ecosystem precedent** | The deployed choice in FAPI 1.0-era ecosystems (UK, Brazil, AU) ✅ | Newer; adopted in FAPI 2.0-era schemes |
| **Operational cost** | High fixed cost (PKI), low per-request cost | Low fixed cost, higher per-request cost, and new failure modes (clock skew, nonce handling, `jti` cache) |
| **Non-repudiation** | Partial | Partial — comprehensive non-repudiation needs the Message Signing profile |

**Heuristic:** mTLS where the ecosystem already operates a CA and trust list for TPPs (the UK/Brazil/AU pattern); DPoP where it does not, or where TLS is terminated before the application. In either case a FAPI 2.0 resource server must support "one or both" ✅.

### 6.5 Rich Authorization Requests (RAR) — RFC 9396 ✅

*OAuth 2.0 Rich Authorization Requests*, **RFC 9396**, Standards Track, **May 2023** ✅. **Problem:** `scope` cannot express "transfer 45 euros to Merchant A" ✅ — the RFC's own example ✅. RAR introduces the `authorization_details` parameter, a JSON array of typed objects each with a mandatory `type` field determining the object's allowable contents ✅. **Buys:** consent that can be described, transmitted, enforced and audited at the granularity money movement requires — amount, currency, creditor, accounts and permitted actions (`initiate`, `status`, `cancel` in the RFC's payment example ✅) — plus common cross-API fields (locations, actions, datatypes, identifier, privileges) ✅. **Mandatory vs optional:** **optional in both FAPI generations**; the WG lists it among the extensions that "can be implemented on top of OAuth 2.0 implementations" ✅, demonstrated in the yes QES scheme and supported by Authlete ✅. It becomes effectively mandatory at the **ecosystem** level, because a scheme wanting fine-grained payment consent must pick something and RAR is the standardised option. **Cost:** the AS must render `authorization_details` into a human-readable consent screen (a UI problem, not a crypto one); the RS must enforce amount/payee against the token on every call, not merely check a scope string; and each ecosystem defines its own `type` values, so RAR alone delivers no cross-scheme interoperability.

### 6.6 JWT-Secured Authorization Response Mode (JARM) — an OpenID Foundation specification, NOT an IETF RFC ✅

**This is the most frequently mis-cited identifier in the field, so it is stated flatly: JARM is an OpenID Foundation (FAPI Working Group) specification. It is not an RFC.** The page title is *"Draft-02: Financial-grade API: JWT Secured Authorization Response Mode for OAuth 2.0 (JARM)"* ✅; it defines the JWT response modes `query.jwt`, `fragment.jwt`, `form_post.jwt` and `jwt` ✅ and registers OIDC client metadata parameters (`authorization_signed_response_alg`, `authorization_encrypted_response_alg`, `authorization_encrypted_response_enc`) and AS metadata parameters (`authorization_signing_alg_values_supported`, plus encryption equivalents) in the IANA registries established by RFC 8414 ✅ — but that registry activity is the *consequence* of an OIDF specification, not an IETF publication. **What it solves:** the authorization *response* is as unprotected as the request in plain OAuth; JARM returns the response parameters — including the code — inside a signed (optionally encrypted) JWT, restoring integrity, sender authentication, audience restriction and protections against replay, credential leakage and mix-up ✅. **Mandatory vs optional:** required in **FAPI 1.0 Advanced** when `code` is used with `response_mode=jwt` ✅; **obsolete in FAPI 2.0** ✅, where the response is reduced to the code alone, judged as "obsoleting the need for integrity protection" ✅. **Honest note:** JARM still exists and is referenced by 1.0 Advanced and by deployed ecosystems, so it cannot be ignored — but a FAPI 2.0-only architecture should not implement it, and a document citing JARM as "RFC 9xxx" is wrong about its provenance.

### 6.7 Client-Initiated Backchannel Authentication (CIBA) — OpenID Foundation specification ✅

*OpenID Connect Client-Initiated Backchannel Authentication Flow - Core 1.0* ✅, dated **1 September 2021** in the FAPI 2.0 Security Profile's normative reference list ✅; there is also a **FAPI-CIBA profile** with its own certification stream ✅. **Problem:** decoupled authentication — the user interacts with the client at a **Consumption Device** but authenticates and consents at a different **Authentication Device**, with direct relying-party-to-OP communication and **no browser redirect** ✅; the OP is given a user identifier and the client obtains tokens by poll, ping or push ✅. **Buys:** FAPI applicability to flows with no redirect at all — call-centre initiated payments, point-of-sale or ATM scenarios, corporate/machine-adjacent cases. **Mandatory vs optional:** not part of either core profile — an **optional companion specification** ✅; where a scheme mandates CIBA for a channel it becomes mandatory *by ecosystem*. **Cost:** a second, structurally different authorization endpoint; `user_code` and consumption/authentication device binding management; and security now depends heavily on how the user authenticates on the authentication device — which is **explicitly out of scope** of the FAPI attacker model ✅.

### 6.8 Grant Management — an OpenID Foundation draft ✅

*Grant Management for OAuth 2.0 (Draft)*, OIDF FAPI Working Group, **Internet-Draft `oauth-v2-grant-management-03`, published 9 May 2023** ✅; status **draft**. Note the rename — the canonical URL is now `oauth-v2-grant-management.html`, with the older `fapi-grant-management.html` redirecting to it ✅ — evidence the document has outgrown a FAPI-only scope. **Problem, in the draft's own words** ✅: OAuth has no explicit representation of a grant, so clients "cannot explicitly manage grants, e.g. query the status or revoke a grant that is no longer needed", with status only implicitly communicated by a successful refresh or a 401/403. It cites the two workarounds already deployed: the UK's `DELETE /account-access-consents/{ConsentId}` and Australia's `cdr_arrangement_id` with `POST /arrangements/revoke` ✅. **Buys:** a standardised `grant_id` plus a **Grant Management API** — creation and update via an authorization request, status query and revocation via the new API ✅ — and a useful distinction between **grant** (a set of permissions, an AS-managed resource) and **consent** (a legal concept that may produce a grant, and which carries "legal, audit, reporting, archiving and non-repudiation requirements" ✅). **Mandatory vs optional:** optional, and still a draft ✅ — but its relevance is disproportionate to its status, since every mature open-banking scheme has hand-rolled a version of it. ⚠ No later revision was verified in this pass. **Cost:** a new authenticated API surface on the AS; durable, queryable, auditable grant records; and reconciliation between grant state and the ecosystem's consent record.

### 6.9 The Identity-Assurance Tie-In — OpenID Connect for Identity Assurance ✅

*OpenID Connect for Identity Assurance 1.0 incorporating errata set 1*, OpenID Foundation **eKYC-IDA** Working Group, **published 1 July 2026**, status **Final** ✅ — a separate working group and specification family from FAPI, but the natural companion. **Problem:** `acr` tells a relying party what *authentication* happened, not how trustworthy a *claim* is. The spec's own illustration ✅: "the assurance an OP typically will be able to give for an e-mail address will be 'self-asserted' or 'verified'. The family name of an end-user, in contrast, might have been verified in accordance with the respective anti-money laundering law by showing an ID card to a trained employee of the OP operator." **Buys:** the ability to communicate how well a person was identified alongside the authorization decision — the missing half of the identity story for AML compliance, access to sensitive data, risk mitigation and fraud prevention ✅. **Mandatory vs optional:** **not part of FAPI** — a distinct specification ✅ whose own scope is honest: it defines the technical tools, while "legal aspects (including liability), trust frameworks, or commercial agreements are out of scope. It is up to the particular deployment to complement the technical solution based on this document with the respective definitions ('the rules')." **Cost:** if implemented, it must be implemented *truthfully* — the assurance metadata describes real verification processes, so the specification forces documentation of the KYC pipeline, not merely of the API.

**Section summary.** The identifier table in **§15 Part B** is the compact decoder; mandatory/optional status across the three profiles is stated in each entry above and summarised in the §5.5 diff table. Two rules to carry forward: **JARM is an OIDF specification, not an RFC**, and **everything mandatory in FAPI 2.0 is an RFC** (PAR, PKCE, mTLS/DPoP, `iss`, metadata).

---

## 7. Cryptographic and Key-Management Reality

Every mechanism in §6 has a cryptographic tail, and the tail is where FAPI deployments actually fail — not in the happy path, but in a key rotated without being published early enough, a client certificate that expired on a Saturday, or a JWKS cache that served a stale key for twenty minutes.

### 7.1 Permitted Signing Algorithms

✅ (FAPI 2.0 Security Profile §5.4.1.)

| Requirement | Value |
|---|---|
| **Permitted JWS algorithms** | **`PS256`, `ES256`, or `EdDSA` (using the `Ed25519` variant)** — nothing else ✅ |
| **`none`** | **"not use or accept"** ✅ |
| **JWT handling** | Must adhere to **RFC 8725** (JWT BCP) ✅ |
| **RSA / EC key length** | RSA minimum **2048 bits**; EC minimum **224 bits** ✅ (FAPI 1.0 Baseline allowed EC ≥ 160 bits ✅ — note the tightening between generations) |
| **Credential entropy** | Access tokens, refresh tokens, authorization codes and other non-user-handled credentials: **≥ 128 bits** ✅ |

Two observations. The list is **short on purpose**, and it excludes `RS256` — still ubiquitous in older OpenID Connect deployments — so an institution whose existing OP signs with `RS256` has real work to do and cannot simply "turn on FAPI". And the spec **admits its own gap** ✅: a note records that at time of writing no fully-specified IANA registration existed for "`EdDSA` using the `Ed25519` variant", with the caveat that such an algorithm may be used if registered later. In practice `PS256` and `ES256` are the universally-implementable choices today (⚠ this guide's recommendation, not a spec clause).

### 7.2 Key Storage and Rotation

FAPI does not prescribe how keys are stored — the Attacker Model assumes "secrets are created such that attackers cannot guess them" and puts weak random number generators outside its scope ✅ — but the profile's security considerations address the key-compromise consequence directly ✅:

> "**Stateful credentials:** It is recommended that implementers consider the trade-offs between stateful and stateless credentials, such as access tokens. In the event of a key compromise, the use of stateless tokens signed by the compromised key could enable an attacker to forge tokens. This risk can be mitigated if all tokens are stateful… However, stateless tokens offer significant advantages… They can be parsed and validated by resource servers directly, without further authorization server involvement." ✅ (Security Profile, Security Considerations)

That is the sharpest operational advice in the profile, and it is a genuine trade-off rather than a recommendation: a self-contained JWT access token is fast and decentralised, but a compromised signing key mints valid tokens until the key is revoked *and* every resource server has refreshed its JWKS cache. The mitigation is short token lifetimes plus revocation-aware validation — FAPI 2.0 requires the RS to "verify the validity, integrity, expiration and **revocation status** of access tokens" ✅.

| Element | Requirement / good practice | Source |
|---|---|---|
| **Asymmetric keys only** for `private_key_jwt`; client holds the private key, AS the public key | | SP2 §5.3.2.1 ✅ |
| **JWKS publication** | AS **strongly recommended** to use `jwks_uri`; clients recommended to use `jwks_uri` **or** the `jwks` parameter via **RFC 7591/7592** | SP2 §5.4.2 ✅ |
| **JWKS endpoint** | TLS only; *should not* use JOSE `x5u`/`jku`; *should not* serve duplicate `kid`s | SP2 §5.4.2 ✅ |
| **Same-`kid` disambiguation** | Verifiers **shall** consider `kty`, `use`, `alg`, `crv` when selecting a verification key | SP2 §5.4.3 ✅ |
| **Rotation practice** | New key published **before** it starts signing; old key kept published until every token it signed has expired; never reuse a `kid` | Structural (unmarked) — the spec constrains endpoints, not process |
| **Algorithm pinning** | Reject tokens whose `alg` is outside the permitted set; never let the `alg` header select a key from an unconstrained set | **RFC 8725** ✅ |
| **Refresh tokens** | AS **shall not** use refresh token rotation "except in extraordinary circumstances" ✅ — removing a class of race conditions but also removing the reuse-detection signal rotation normally provides | SP2 §5.3.2.1 ✅ |

### 7.3 mTLS Certificate Lifecycle

For an mTLS-based deployment, certificate management *is* the channel. **(1) Trust-list distribution** — FAPI 2.0: "MTLS ecosystems **should** provide the trust list of the certificate authorities" ✅; in a national scheme this is a directory (Open Banking Directory, Brazil's directory, the CDR register), i.e. an ecosystem artefact, not a FAPI one. **(2) Two variants, different burdens** — `tls_client_auth` means the AS issues the certificate, `self_signed_tls_client_auth` means the client self-signs and the *directory* vouches for the key ✅; the self-signed variant shifts trust entirely onto the registration process. **(3) Endpoint aliasing** — an AS may publish `mtls_endpoint_aliases` (**RFC 8705** §5); clients **shall** honour the `use_mtls_endpoint_aliases` metadata if present ✅ (a parameter introduced by FAPI 2.0). **(4) The binding depends on the handshake** — a certificate-bound token is validated against the certificate presented on that connection ✅, so every component in the path (load balancer, service mesh, API gateway) must either pass the client certificate through or be the sole terminator and perform the binding check itself; TLS offload that discards the client certificate breaks the mechanism. **(5) Revocation** must be genuinely reachable — a stale CRL or an uncached OCSP response is a live risk.

### 7.4 JWKS Distribution and Caching

The failure arithmetic is not intuitive. If an AS rotates its signing key while a resource server caches the JWKS for a fixed window (say 24 hours), then a token signed by the new key fails verification on that RS for up to 24 hours — or the RS fails open, which is worse. The controls FAPI-adjacent deployments converge on: **cache on `kid` miss, not on a timer alone** (refetch when a presented token's `kid` is unknown, subject to rate limiting — unmarked, this is deployed practice rather than a spec clause); **respect `Cache-Control`/`max-age`** and keep the document small; **never trust `jku`/`x5u`** — FAPI says servers should not *use* them ✅, and a verifier that follows an attacker-supplied key URL has handed over the channel, a classic JWT vulnerability class RFC 8725 warns about; and **publish-then-sign** as the mandatory rotation sequence.

### 7.5 Clock Skew, Nonces, and the Failure Modes That Actually Happen

**Clock skew, precisely** ✅ (SP2 §5.3.2.1): a server **shall accept** JWTs with an `iat` or `nbf` between **0 and 10 seconds in the future**, and **shall reject** anything **greater than 60 seconds** in the future. The spec's own guidance to a client whose device clock is unreliable ✅: "a client should consider using the HTTP date header returned from the server to synchronize its own clock when generating client assertions" — a NOTE that exists because `private_key_jwt` failing on a skewed clock is a real deployment failure. The three timings interact in a way that catches implementers out:

| Timing rule | Value | Interaction |
|---|---|---|
| `iat`/`nbf` in the future, accepted | 0–10 s ✅ | A client 11 s fast is rejected immediately |
| `iat`/`nbf` in the future, rejected | > 60 s ✅ | A client 61 s fast is rejected |
| **Authorization code maximum lifetime** | **60 s** ✅ | A client 61 s slow may fail to redeem a code in time *even though the code is legitimate* |

**Nonces.** `nonce` is required by FAPI 1.0 Baseline when the client requests `openid` scope ✅; FAPI 2.0 retains OIDC support and advises `nonce` values ≤ **64 characters** "to aid interoperability" ✅ — a mundane constraint that exists because some implementations had length limits. DPoP adds its own mechanism: an AS "may use the server provided nonce mechanism" (**RFC 9449** §8) ✅, and the profile's security considerations address DPoP proof replay with nonces and `jti`-based replay prevention ✅.

**The operational failure modes, in the order they take channels down:**

| Failure mode | What breaks | Mitigation |
|---|---|---|
| **Expired client certificate** | Every request from that TPP fails at once — total channel outage for that client | Automated expiry monitoring well ahead of renewal; certificate inventory as a first-class asset; directory reminder integration |
| **Stale public key in a JWKS cache** | Verification fails (or fails open) after rotation | Refetch on unknown `kid`; publish-then-sign; monitor verification failures by `kid` |
| **Client clock skew** | `private_key_jwt` rejected; codes expire before redemption | NTP discipline; HTTP `Date` synchronisation per the spec's NOTE ✅ |
| **TLS offload discarding the client certificate** | Certificate-bound tokens can no longer be validated | Enforce the binding where the certificate is visible; test the full proxy path, not just the origin |
| **Duplicate `kid` across two keys** | Verifier picks the wrong key and rejects a valid signature | Don't reuse `kid`s ✅; if it happens, disambiguate on `kty`/`use`/`alg`/`crv` ✅ |
| **Refresh token rotation race** | Concurrent refreshes invalidate each other | FAPI 2.0 prohibits rotation except in extraordinary circumstances ✅ |
| **Authorization code consumed twice** | Second redemption correctly rejected, but the user sees a failure | AS **shall** reject a reused code (1.0 Baseline) ✅; the client must not blindly retry |

**The transferable lesson.** The hardest-won insight of the FAPI generations is not about an algorithm: **each additional mandatory cryptographic mechanism is itself a new failure surface.** FAPI 1.0 Advanced required four (JAR, JARM, detached-signature ID token with `s_hash`, mTLS) ✅; FAPI 2.0 requires fewer and reuses them harder ✅. The security properties held; the operational surface shrank. Prefer the profile whose mandatory mechanisms you can instrument, monitor and rotate — not the one with the longest list.

---

## 8. Certification and Conformance

### 8.1 What the Certification Programme Is

The OpenID Foundation runs a two-stage programme ✅ ([openid.net/certification/](https://openid.net/certification/)): **conformance testing**, free and open to anyone — "The free suite of conformance tests are available to anyone and enables developers to assess how well their products and implementations conform to our most mature, stable standards and profiles. This helps ensure interoperability and ensures, for example, that the security features of a given standard have been built correctly" ✅ (the suite is open source at [gitlab.com/openid/conformance-suite](https://gitlab.com/openid/conformance-suite) and runs at [certification.openid.net](https://www.certification.openid.net/)); and **certification**, the paid published step — "Implementers may want to certify conformance to a standard. This is particularly relevant if a company wishes to signal to its customers that a product is safe, secure, and capable of interoperating with others. Sometimes, certification is a pre-requisite to participating in an ecosystem (such as an open finance ecosystem)." ✅ **Self-Certification** lets implementers run the tests, submit, and use the **"OpenID Certified"** mark ✅. **Independent Certification** — the OIDF "is developing a network of third-parties who will act as **Authorized Auditors** and **Testing Service Providers**… This page will be updated as those services mature" ✅ — ⚠ i.e. **in progress at the time of this pass**, not fully established.

### 8.2 Which FAPI Certifications Exist Today ✅

Verified from the OpenID Foundation's "All Certified Implementations" index (read 17 September 2026); the programme is organised by **role** and by **ecosystem profile**:

| Certification family | Listings present ✅ |
|---|---|
| **FAPI1-Advanced OPs** | General FAPI1 Adv. OP & Profiles; **UK Open Banking**; **Australia CDR**; **Brazil Open Banking**; **Brazil Open Finance (FAPI-BR v2)**; **Brazil Open Insurance**; **Brazil Open Insurance (FAPI-BR v2)**; **KSA Open Banking**; **Second Implementers Draft (including UK Open Banking)** |
| **FAPI CIBA OPs** | FAPI CIBA OP & Profiles |
| **FAPI1-Advanced RPs** | General FAPI1 Adv. RP & Profiles; Brazil Open Banking; Brazil Open Finance (FAPI-BR v2); Brazil Open Insurance; Brazil Open Insurance (FAPI-BR v2); KSA Open Banking; Second Implementers Draft |
| **FAPI 2.0 OPs** | **FAPI 2.0 OP Security Profile Final & Message Signing Final**; FAPI 2.0 OP Security Profile Second Implementer's Draft & Message Signing First Implementer's Draft; **Australia FAPI 2.0 OP ConnectID (Final and Implementer's Draft)**; **CBUAE FAPI 2.0 OP Message Signing (Final and ID1)**; **KSA Open Banking FAPI 2.0 Message Signing OP** |
| **FAPI 2.0 RPs** | The same set mirrored: Security Profile Final & Message Signing Final; Second ID / First ID pairing; Australia ConnectID (Final and ID); CBUAE Message Signing (Final and ID1); KSA Open Banking Message Signing |

**What the naming tells you.** **Every FAPI 2.0 certification is a paired artefact** — "Security Profile Final **&** Message Signing Final" ✅; there is no listing for a Security-Profile-only FAPI 2.0 certification, so ⚠ **whether a "FAPI 2.0 baseline-only" certification is available was not established** (§14). **Ecosystem profiles are first-class certifications** — UK Open Banking, Australia CDR, Brazil (banking, finance, insurance, in two versions), KSA, CBUAE and ConnectID each have their own listing pages ✅; this is how the ecosystem-mandated standards in §9 get tested. **Second Implementers Draft certification still exists** ✅ — a reminder that this ecosystem certified against drafts for years and that some certifications in the wild are against pre-final revisions. And **certification is per role**: an OP certification and an RP certification are distinct, so a bank acting as both AS and RS may need both while a TPP needs RP certification.

### 8.3 What the Conformance Suite Tests

The suite exercises the protocol — flows, request/response structures, parameter handling, error responses — for the profile in question. The FAPI-specific test pages the OIDF publishes are ✅: "How to run conformance tests for FAPI 2 and FAPI1-Advanced OPs", "How to run conformance tests for FAPI-CIBA OPs", "How to run conformance tests for FAPI RPs", plus per-ecosystem overviews (Open Banking Brazil OP and RP; Australian ConnectID FAPI RP payment instructions) ✅. ⚠ The individual test-module inventory was not enumerated in this pass; the suite source is the authoritative list.

### 8.4 What Passing Does NOT Prove

Certification proves **protocol conformance**, and the OpenID Foundation's own language supports the caveats. **(1) Not absence of vulnerabilities** — the Attacker Model says it directly ✅: "Real-world implementations, of course, sometimes deviate from the specified and formally analyzed behavior and contain security vulnerabilities on various levels", and FAPI 2.0 carries a section on "Incomplete or Incorrect Implementations of the Specifications" ✅. **(2) Not the security of the business process** — nothing tests whether the consent screen told the truth, whether an over-broad scope was granted, whether the resource side enforces the consented amount, or whether the bank's fraud controls work. **(3) Not the surrounding infrastructure** — TLS configuration, WAF, rate limiting, DDoS resilience, monitoring, incident response and key custody are all outside the suite, and mostly outside the Attacker Model's scope ✅. **(4) Not a security audit** — certification is not penetration testing, code review or an assurance report (SOC 2, ISO 27001); the OIDF's own framing is that tests ensure "the security features of a given standard have been built **correctly**" ✅ — correctly, not securely in context. **(5) Not continued conformance** — a certification is a snapshot against a profile version; change the implementation or the profile revision and the snapshot no longer describes the system. **(6) Not interoperability with a specific counterparty** — conformance maximises the odds; only testing against the actual counterparty proves it.

### 8.5 A Related and Often-Confused Point, and What Was and Was Not Verified

✅ The WG states explicitly that "The FAPI and OpenID Connect certifications are orthogonal. In order to pass OpenID Connect certifications servers would have to support various lower security methods (like `client_secret_basic` for client authentication) that would generally not be enabled in FAPI compliant servers." ✅ Put plainly: **a FAPI-certified authorisation server will typically *fail* parts of an OpenID Connect certification, and that is correct behaviour.** A procurement checklist requiring "OpenID Connect certified" as a proxy for "FAPI capable" tests the wrong thing. Ask for the FAPI certification profile by name, for the correct role (OP vs RP), and for the ecosystem profile if one applies.

✅ **Verified as published by the OIDF:** the WG names **Authlete, Cloudentity, ConnectID, Ping Identity and Raidiam** as having "achieved compliance with the current FAPI 2.0 certifications" ✅, and reports **15 UK banks holding 31 FAPI certifications across 16 deployments** ✅ including Barclays (Barclays OB TIAA), Cater Allen, Coutts, First Direct, HSBC RBWM, HSBC Business, ICICI Bank UK, Marks and Spencer, NatWest, RBS, Sainsbury's Bank, TSB (CA API Gateway 9.4), Ulster Bank, Vanquis, Hargreaves Lansdown Savings and WSO2 (UK) ✅, adding that "Most of the CMA9 have certified and OIDF anticipates OBIE requiring CMA9 to recertify annually" ✅. ⚠ **Not verified in this pass:** the *current* contents of each per-implementation certification listing page (§14).

---

## 9. The Regulatory Footprint, Regime by Regime

The distinction that matters throughout — and that vendor material routinely blurs — is three-way: **what the FAPI specification mandates**, **what a regulator or scheme mandates**, and **what a vendor claims**. For each regime: the instrument, the mandated standard, and whether FAPI is **named** or merely **implied** by an industry standard built on it.

### 9.1 United Kingdom ✅

**Framework:** open banking was created by the **CMA (Competition and Markets Authority) Order** requiring the nine largest banks (the "CMA9") to implement an open banking standard; the standard and directory were built and operated by the **Open Banking Implementation Entity (OBIE)**, subsequently **Open Banking Limited (OBL)**. **Instrument:** the standards at [standards.openbanking.org.uk](https://standards.openbanking.org.uk/) ✅. **Is FAPI named? YES — named and versioned:** "The Open Banking API standard has adopted **FAPI 1** as the security profile. FAPI is a highly secured OAuth profile provided by the Open ID Foundation." ✅ **Which revision:** **v4 of the Open Banking API Standard implements "the final release of the FAPI 1 Advanced specification"**, decided "by a vote at the Technical Design Authority" ✅; v3 had used **FAPI 1 Implementers Draft 2**, which "was deprecated with the final release of FAPI 1 Advanced in 2021 and support/certification services removed in **December 2024**" ✅. **Certificates:** "FAPI introduces new security features over and above OAuth which require the use of certificates for both transport layer and message signing. The **Open Banking Directory** provides **PSD2 (and UK Payment Services Regulations 2017) compliant certificates** for both purposes." ✅ — the EU eIDAS/QWAC route and the FAPI route converging in UK practice. **JARM: explicitly out of scope** ✅ ("JARM was not adopted in the original UK security profile, nor mandated in FAPI 1 Advanced and is out of scope for UK implementations"). **CIBA: optional** for ASPSPs in the UK ✅. **Ecosystem specifics:** an intent ID pre-generated via `POST /{API-TYPE}-Consents` using the **client credentials grant** ✅; ecosystem `acr` values `urn:openbanking:psd2:sca` and `urn:openbanking:psd2:ca`, the former referencing the **FCA's SCA-RTS** ✅; a **model bank** TPPs are advised to test against ✅. **Clock skew:** ✅ OBL notes "The FAPI 1 spec is **silent** on the approach participants should take to clock skew. Participants should determine what level of clock skew they are willing to accept and publish details on their developer portal," proposing the FAPI 2 values as a starting point — an unusually candid admission that a specification gap had to be filled locally. **Regime verdict: FAPI is NAMED, at a specific version, with ecosystem additions.**

### 9.2 Australia — the Consumer Data Right ✅

**Framework:** the **Consumer Data Right (CDR)**, a statutory data-sharing regime applied first to banking, then energy; the standards site is the *CDR Data Standards* published by the **Data Standards Chair** ✅. **Instrument:** [consumerdatastandardsaustralia.github.io/standards](https://consumerdatastandardsaustralia.github.io/standards/) ✅. **Is FAPI named? YES — in the normative references and in the security profile's own statement of basis.** ✅ The normative-reference table lists **`[FAPI-1.0-Baseline]`** *Financial-grade API Security Profile 1.0 - Part 1: Baseline* (**March 2021**), **`[FAPI-1.0-Advanced]`** *Part 2: Advanced* (**March 2021**) and **`[JARM]`** *Financial-grade API: JWT Secured Authorization Response Mode for OAuth 2.0* (**October 2020**) ✅, and the security profile states: "This information security profile **builds upon the foundations of the Financial-grade API Advanced Profile**" — i.e. a *derivative* of FAPI 1.0 Advanced inside an instrument that also names FAPI directly. **Ecosystem conventions:** the CDR standards mandate the **`x-fapi-*` headers** — `x-fapi-interaction-id` (mandatory on authenticated APIs, played back in the response), `x-fapi-auth-date` (required for all resource calls) and `x-fapi-customer-ip-address` ✅ — precisely the headers FAPI 2.0 later **moved out** of the core profile as "not relevant" ✅, a neat illustration of how ecosystem profiles accumulate what the security profile deliberately does not own. **Two facts for the claims audit:** the Australian standards independently date **both FAPI 1.0 parts to March 2021** ✅, corroborating the OIDF WG from a second non-OIDF source; and the AU JARM reference is dated **October 2020** ✅, which is the draft revision rather than a final publication date.

### 9.3 Brazil — Open Finance (Banco Central do Brasil)

Brazil's **open banking** programme, extended to **open finance** and — per the certification listings — **open insurance**, runs under the **Banco Central do Brasil (BCB)** with the CMN ✅ (institution names verified in the OIDF certification taxonomy; the specific BCB resolution numbers were **not** verified — §14). **Is FAPI named? YES — structurally, via dedicated OpenID Foundation certification profiles:** separate listings exist for **"FAPI1 OP – Brazil Open Banking"**, **"FAPI1 OP – Brazil Open Finance (FAPI-BR v2)"**, **"FAPI1 OP – Brazil Open Insurance"** and **"FAPI1 OP – Brazil Open Insurance (FAPI-BR v2)"**, with the mirrored RP listings ✅. The existence of a name — **FAPI-BR v2** — is itself the finding: Brazil operates a **national FAPI profile with its own version number**, which the OpenID Foundation certifies against. **Practical consequence:** a Brazilian implementation cannot simply "be FAPI 1.0 Advanced"; it must implement **FAPI-BR v2** as certified ✅. The former `openbanking-brasil.github.io` developer area was **deactivated on 21 July 2022** and redirected to the current Open Finance developer portal ✅. **Verdict: FAPI is NAMED, nationalised and versioned**; ⚠ the BCB/CMN instrument numbers and the FAPI-BR v2 text were not read at source.

### 9.4 India, Singapore and the EU — Three Regimes Where FAPI Is Not (Verifiably) Named ⚠

These are grouped because they share a pattern: substantial data-sharing regimes whose accessible public primary material, in this pass, did **not** name FAPI. Stating that honestly is more useful than implying a mandate.

**India — the Account Aggregator framework (RBI).** India's consent-based regime is the **Account Aggregator (AA)** framework under the **Reserve Bank of India**, with technical standards developed by **ReBIT** and an industry body (Sahamati) coordinating the ecosystem; the model centres on a signed **consent artefact** brokered by a licensed AA. ⚠ **Explicit negative finding: whether the RBI/ReBIT technical standard names FAPI was NOT ESTABLISHED.** `rebit.org.in` failed (server error) and the Sahamati documentation endpoint was blocked as a private/internal address (§14). **Do not report "India uses FAPI" without establishing it at RBI/ReBIT sources.**

**Singapore — SGFinDex and the MAS playbook.** ✅ **Verified at MAS:** *"SGFinDex is a digital public infrastructure that uses national identity and centrally managed online consent system for secure access to financial data from private and public sectors"*; it is *"a joint initiative by the Monetary Authority of Singapore (MAS) and the Government Technology Agency of Singapore (GovTech)… built on Singapore's National Digital Identity (Singpass), developed by the public sector in collaboration with The Association of Banks in Singapore (ABS), the Life Insurance Association (LIA Singapore), and 15 participating financial institutions."* It is described as a *"Centralised gateway for efficient data sharing based on common data and API standards."* ✅ **FAPI is not named on that page**, and neither is OAuth. ⚠ Whether the SGFinDex technical standards reference FAPI internally **was not established**; MAS's "Finance-as-a-Service: API Playbook" ⚠ was identified but not read at source. Architecturally this is the contrarian design — national identity, a centrally operated consent gateway and a curated participant list, solving data-sharing security by **narrowing the participant set** rather than mandating a security profile on open interfaces. That is a legitimate design, and it is not FAPI.

**The European Union — PSD2, the EBA RTS and FIDA.** **PSD2**, Directive (EU) 2015/2366, made AIS and PIS regulated activities and obliged ASPSPs to serve TPPs; the **EBA RTS on strong customer authentication and common and secure open standards of communication** ("RTS on SCA & CSC") supplies the operational security requirements — mutual TLS for TPP identification using **eIDAS qualified certificates (QWACs)**, plus dedicated/fallback interface rules. The Commission's follow-on **Financial Data Access (FIDA)** proposal is the open-finance successor direction. ⚠ **Verification status: UNVERIFIED AT SOURCE IN THIS PASS.** The EBA URL returned "Page not found" and two EUR-Lex extraction attempts failed; the directive number, the delegated-regulation number, the QWAC requirement, the interface rules and FIDA's COM number and stage are therefore **structural knowledge, flagged, not asserted** (§14). ⚠ **Is FAPI named in any EU instrument? NOT ESTABLISHED.** What can be said: the EU route does the *identity* work regulatorily and leaves the *protocol* work to ecosystem standards — which is why the UK, drawing on the same certificates, runs FAPI on top of them ✅.

### 9.5 The United States — FDX, and the CFPB's Section 1033 Rule

**The FDX liaison ✅.** The FAPI WG page records a formal liaison with the **Financial Data Exchange (FDX)**: "The FDX/OIDF agreement clarifies FDX's usage of OIDF trademarks. The Liaison Agreement describes the common interests of the two organizations and how they might work together," and states that "The **Financial Data Exchange** is also working closely with the FAPI WG to implement the specs in North America." ✅ So in the US, **FAPI is reached through an industry standard, not through a statute.**

**Section 1033 — the current status, stated precisely.** This is the fact most likely to be stale in secondary sources, so it is split between what was verified at primary source and what rests on secondary reporting.

✅ **Verified at primary source (consumerfinance.gov):** the rule exists in the Code of Federal Regulations as **12 CFR Part 1033 — "Personal Financial Data Rights; Industry Standard-Setting"**, marked **"Most recently amended Jan. 17, 2025"**, described as *"This final rule establishes minimum attributes a standard-setting body must possess to receive CFPB recognition and to issue consensus standards when the full rule is finalized."* ✅ The codified text of **§ 1033.101 (Authority, purpose, and organization)** is published as **"the current regulation"** ✅ and sets out the purpose: requiring data providers to make covered data available "to consumers and authorized third parties, upon request… in an electronic form usable by consumers and authorized third parties", plus "obligations of third parties that would access covered data on a consumer's behalf, including limitations on their collection, use, and retention of covered data" ✅. The structure (Subparts A–D plus an Appendix A for standard-setting-body recognition) is published ✅.

⚠ **Flagged — secondary reporting, internally consistent across the sources read in this pass but NOT verified at consumerfinance.gov or a court docket:**

- The rule was **finalised in October 2024** with a **phased compliance schedule running 2026–2030**, the largest banks and data providers facing the earliest deadlines, **starting April 2026**.
- **Forcht Bank, N.A., the Bank Policy Institute and the Kentucky Bankers Association sued the CFPB the same day the rule was finalised**, arguing it exceeded the Bureau's statutory authority under Dodd-Frank.
- In **March 2025** the parties agreed to **pause the case and toll the rule's compliance deadlines** while a newly-led CFPB reconsidered; the Bureau's chief legal officer subsequently told the court the Bureau now viewed its rule as unlawful and that it should be set aside.
- On **29 October 2025** the **US District Court for the Eastern District of Kentucky granted a preliminary injunction** barring enforcement while the Bureau reconsiders. The case is on appeal as **Forcht Bank, N.A. v. CFPB** before the **Sixth Circuit**, with the **appeal stayed pending the outcome of the rulemaking** and joint status reports filed roughly every six weeks.
- A **reconsideration proposal is with the White House** (per the reporting read in this pass).
- A separate legal commentary describes the federal rules as "codified in the Code of Federal Regulations but **effectively unenforceable**," with **states** bringing data-sharing into focus ⚠.

**The precise statement this guide will stand behind (as of 17 September 2026):** *The CFPB's Section 1033 Personal Financial Data Rights rule was finalised in October 2024 and remains codified at 12 CFR Part 1033 (most recently amended 17 January 2025). It is currently **not enforceable**: a federal court in Kentucky enjoined its enforcement on 29 October 2025 while the Bureau reconsiders it, the compliance deadlines have been tolled, and the appeal is stayed pending the rulemaking. A reconsideration proposal is reportedly awaiting White House review. Any US go-live plan built on the October 2024 timeline should be treated as unscheduled.* The codification is ✅; the litigation timeline is ⚠ and should be re-checked against the docket and consumerfinance.gov before being relied on.

### 9.6 The Regime Summary

| Regime | Instrument | Mandated standard | Is FAPI named? | Verified? |
|---|---|---|---|---|
| **UK** | Open Banking standards (CMA Order-derived; OBIE/Open Banking Limited) | **FAPI 1 Advanced (final)** since v4; v3 used FAPI 1 ID2 | **Yes, by version** ✅ | ✅ |
| **Australia** | CDR Data Standards (Data Standards Chair) | CDR security profile **built upon FAPI 1.0 Advanced**; JARM referenced | **Yes, in normative references** ✅ | ✅ |
| **Brazil** | BCB/CMN open finance framework ⚠ | **FAPI-BR v2** national profile, OIDF-certified | **Yes, nationalised and versioned** ✅ | ⚠ instrument only |
| **India** | RBI Account Aggregator framework; ReBIT technical standards | Consent-artefact model | **NOT ESTABLISHED** ⚠ | ⚠ negative finding |
| **Singapore** | MAS/GovTech SGFinDex; Singpass NDID | "Common data and API standards", centrally operated | **Not named on the MAS page read** ✅ | ✅ page, ⚠ internals |
| **EU** | PSD2 + EBA RTS on SCA & CSC; FIDA proposal | Regulatory identity (eIDAS/QWAC) + interface rules | **NOT ESTABLISHED** ⚠ | ⚠ unverified |
| **US** | FDX (industry); CFPB §1033 / 12 CFR Part 1033 (codified, enjoined) | FDX API + security model | **Via FDX liaison; not in statute** ✅ | ✅ liaison, ⚠ litigation |

---

## 10. The Implementation Reality for a Bank

### 10.1 Roles and the Trust Model

A bank in a FAPI ecosystem occupies **two roles at once**, and confusing them is the most common architectural error.

| Role | In OAuth terms | What the bank does |
|---|---|---|
| **Authorization server (AS)** | Authenticates the customer, obtains consent, issues tokens | Runs the authorization endpoint, PAR endpoint, token endpoint, JWKS and metadata; enforces PKCE, `iss`, DPoP/mTLS binding; manages the client registry |
| **Resource server (RS)** | Validates tokens, serves protected data | Validates the token's sender-constraining, scope/`authorization_details` and consent state on **every** call |
| **Consent record** | Not an OAuth role | The durable, auditable record of what the customer authorised, reconciled with the token's contents — the object the supervisor asks about |

Alongside sit the **third-party clients (TPPs)**: an AISP reads data, a PISP initiates payments. Each is a **confidential client** in FAPI 2.0 ✅ (public clients are out of scope ✅); FAPI 1.0 Advanced likewise forbids public clients ✅. **The key consequence:** the AS and RS are *separately* security-relevant and FAPI constrains both — the RS must verify validity, integrity, expiration and revocation status, must **not** accept tokens as query parameters, and **shall** verify sender-constraining ✅. In many banks these are different platforms (an IAM product for the AS, an API gateway for the RS) owned by different teams, which is where the FAPI requirement becomes an *integration* problem rather than a product-selection problem.

### 10.2 Architecture of a FAPI-Compliant Stack

A workable layering, with FAPI obligations attached to each layer (unmarked: a synthesis of §4–§7, not a prescribed architecture). **(1) Directory / trust framework** (ecosystem-provided) — TPP registration, client metadata, certificate or key publication, trust lists; the artefact FAPI 2.0 assumes exists ✅. **(2) Authorisation server** — discovery and AS metadata (RFC 8414/OIDD) ✅, authorization endpoint, **PAR endpoint** ✅, token endpoint (client auth: mTLS or `private_key_jwt` only) ✅, JWKS (`jwks_uri`, TLS-only, no `x5u`/`jku`) ✅, revocation and grant-management endpoints. **(3) Consent and SCA orchestration** — the bank's consent record, its SCA journey, and the mapping of `acr` (and RAR `authorization_details` where used ✅) onto what the customer actually agreed to. **(4) API gateway / resource servers** — token validation including binding, consent-state lookup, rate limiting, audit logging of who/what/when/consent. **(5) Key and certificate management** — HSMs or key stores, certificate lifecycle, JWKS publication pipeline. **(6) Observability and evidence** — token and consent events, error taxonomies, SLOs on the authorisation flow. **Where the binding is enforced matters most:** for mTLS the check needs the client certificate, so the gateway must terminate TLS with client auth and perform the binding itself, or pass the certificate through; for DPoP the proof is an HTTP header and survives TLS termination ✅ — a concrete advantage in banks with centralised TLS offload.

### 10.3 Consent and SCA Interplay

Three separable layers, often conflated. **Authentication strength (SCA)** — FAPI does not define it; the Attacker Model puts end-user identity proofing, authentication and IAM "out of scope" ✅. In the EU/UK this comes from PSD2 and the FCA's SCA-RTS, encoded in the UK in ecosystem `acr` values ✅; FAPI 1.0 Advanced requires only that the AS "authenticate the user to an appropriate Level of Assurance" ✅. **What was authorised** — the consent record; FAPI secures the *request* that expresses it (PAR/JAR) and the *grant* that results (token, grant-management `grant_id`), but the semantics of the consent object belong to the ecosystem standard. **Enforcement at the resource** — the RS must check that "the authorization represented by the access token is sufficient for the requested resource access" ✅, which for a PISP means checking amount, payee and account against the consent *per call*, not merely checking a scope. The design rule: **treat the consent record as the system of record and the token as a derived, expiring capability** — a token whose contents drift from the consent record is a compliance finding waiting to happen.

### 10.4 Client Onboarding and Registration

Sequence matters and FAPI implies one: register the client, establish its credentials (certificate, or JWKS + `private_key_jwt`), agree its permitted scopes / `authorization_details` types, then admit it to the channel. OpenID Connect **Dynamic Client Registration** (RFC 7591) and **Client Configuration** (RFC 7592) are referenced by FAPI 2.0's JWKS guidance ✅; in practice most schemes register through the ecosystem **directory** and the AS consumes that registry. FAPI 1.0 Advanced effectively requires registration before use for redirect URIs ✅; FAPI 2.0 moves that to PAR, where the AS validates `redirect_uri` on the pushed request ✅.

### 10.5 Monitoring and Audit Evidence a Supervisor Will Ask For

Derived from the requirements (unmarked: a synthesis), this is the evidence set that most directly maps to FAPI's normative clauses: **full authorisation-flow traces per transaction** (PAR push, authorization, token exchange, `iss` check) ✅; **token issue/refresh/revoke records with binding type** (mTLS thumbprint or DPoP key) ✅; **consent records** with the `authorization_details` or scope granted and their lifecycle ✅; **client-authentication success/failure by method and client** (to detect a client falling back to a weak method) ✅; **JWKS key inventory and rotation log, plus certificate inventory with expiry** ✅; **conformance run results per client profile and per release** (certification is a snapshot; supervisors want the current one) ✅; and **rejected-request telemetry** (bad `iss`, expired certificate, `kid` miss, code reuse) — the strongest evidence that a control exists is that it has visibly fired.

### 10.6 Build vs Buy — and the Landscape (Verified Names Only)

⚠ **A deliberate limitation.** This pass verified the **OpenID Foundation's certification listings by name** ✅ but read no vendor product documentation. The names below are therefore *the vendors appearing in OIDF certification listings*, not capability assessments; treat every product claim as unverified.

**Certified-in-the-listings vendors ✅:** **Authlete**, **Cloudentity**, **ConnectID**, **Ping Identity** and **Raidiam** are congratulated by the WG for FAPI 2.0 certification ✅. **WSO2 (UK)** and **TSB Bank PLC (CA API Gateway 9.4)** appear in the UK FAPI certification listings, as do bank-operated deployments (Barclays "Barclays OB TIAA", Cater Allen, Coutts, First Direct, HSBC RBWM, HSBC Business, ICICI Bank UK, M&S, NatWest, RBS, Sainsbury's Bank, Ulster Bank, Vanquis, Hargreaves Lansdown Savings) ✅. **Authlete** supports "PKCE, mTLS, PAR, RAR and the `iss` response parameter" per the WG's implementation list ✅, and the **yes QES Scheme Signing API** is reported as "implemented by three different authorization servers, uses PKCE, mTLS, PAR, RAR, `iss` authorization response parameter" ✅. **The "build on a certified product" route is ecosystem-recommended:** the UK standards say plainly, "As FAPI builds upon a number of existing specifications it is recommended that implementers adopt a **certified product**," pointing at the OIDF list ✅. **Open source:** the **OIDF Conformance Suite** is open source and free ✅ but is *test tooling*, not an authorisation server; the **Open Banking UK model bank** (`github.com/OpenBankingUK/OBL-ModelBank-Integration`) is a reference implementation TPPs are advised to test against ✅ — again a testing asset. ⚠ **The open-source authorisation-server landscape for FAPI specifically was not verified in this pass** (§14); naming products without checking their FAPI conformance would be exactly the kind of claim §13 exists to catch. **The honest build-vs-buy conclusion:** because FAPI's hardness is concentrated in key management, certificate lifecycle, conformance evidence and RS-side consent enforcement (§10.7), buying a **certified AS** and building the **consent / RS / evidence layers** around it matches where the difficulty actually sits. Buying the whole channel — including consent semantics — usually means buying a product that encodes an ecosystem you may not be in.

### 10.7 Where FAPI Compliance Is Genuinely Hard vs Merely Laborious

Offered as labelled judgement, consistent with the specification structure in §4–§7. **Genuinely hard (design and judgement):** (1) **key and certificate lifecycle at ecosystem scale**, including rotation without downtime and being able to prove months later which key signed what; (2) **enforcing consent at the resource, per call** — amount, payee, account, duration — rather than trusting a scope string; (3) **making the AS/RS split work across teams and platforms**, especially where TLS terminates centrally; (4) **choosing and defending a profile position** — 1.0 Advanced (what most ecosystems run) vs 2.0 Baseline (comparable security, less machinery) vs 2.0 Advanced (adds non-repudiation) ✅, which the WG treats as "an ecosystem specific decision" ✅ rather than a lookup; and (5) **deciding whether non-repudiation is worth Message Signing** for a given flow (§11.3). **Merely laborious (volume, not difficulty):** the PAR endpoint, PKCE, `iss` verification, metadata documents and algorithm allow-lists (each small and well-specified ✅); conformance runs and certification submissions per profile and per ecosystem ✅; the client registry, onboarding workflow and TPP support desk; audit logging and the §10.5 evidence pack; and clock-skew, nonce-length and code-lifetime tuning (§7.5) — fiddly but bounded and documented.

### 10.8 Operating Cost Shape

⚠ No verified cost figures exist for FAPI specifically, and none are invented here. The cost *shape* the requirements support (unmarked, structural): a **fixed** PKI and key-management cost (directory participation, HSMs, key ceremony, certificate authority); a **fixed** AS licensing-or-build cost; a **recurring** conformance-and-certification cost per profile and per revision ✅; a **recurring** TPP-onboarding and support cost scaling with the number of participants rather than transaction volume; and a **variable** run cost dominated by authorisation-flow volume plus the added PAR round-trip ✅. §12.8 attaches illustrative **fictional** figures to this shape.

---

## 11. The Critique and the Limits

Opinions in this section are **attributed to a position**, not asserted as fact.

### 11.1 Complexity: the Criticism and the Counter-Criticism

**The criticism (attributed: the implementer community).** FAPI 1.0 Advanced's mandatory stack — signed request object, JARM or detached-signature ID token with `s_hash`, mTLS-bound tokens, plus certificate infrastructure — imposed a cost that fell hardest on smaller participants. The strongest *evidence* is not the criticism but the WG's subsequent actions: FAPI 2.0's diff table replaces four overlapping integrity mechanisms with PAR + PKCE ✅; the WG's stated objectives include "simpler to implement (less requirement on message signing without reducing security)" ✅; and the WG lists "FAPI 2.0 requires less use of message signing which may make it easier to implement (especially for clients)" as a genuine reason to choose 2.0 ✅. Specifications do not undergo that kind of subtraction unless the complexity was felt. **The counter-criticism (attributed: the security-engineering position).** The complexity bought real properties *justified by explicit threats* — authorization request tampering, response tampering including code injection, state injection, and token request phishing ✅ (FAPI 1.0 Part 2's own introduction). Removing a mechanism moves risk to the compensating control (PAR's architectural protection instead of JAR's cryptographic one) and depends on that control being present. FAPI 2.0's answer is that PAR + PKCE + sender-constrained tokens + `iss` **are** those compensating controls and that the combination was formally analysed ✅ — but "fewer mechanisms is better" is a design *choice* supported by analysis, not a proven law. **This guide's reading (labelled as such):** the WG's diff table is the most credible document on either side precisely because it states what was removed *and why* ✅, and it does not claim the removed mechanisms were wrong — only that they were redundant with better alternatives. The WG explicitly declines the "more secure" framing: "The reason for work on the 2.0 draft is **not** a more secure specification than 1.0" ✅.

### 11.2 Remaining Interoperability Friction

**Ecosystem profiles fork the base specification:** Brazil's FAPI-BR v2 ✅, the UK's out-of-scope JARM and UK-specific `acr` values ✅, the CDR's mandatory `x-fapi-*` headers ✅ — all national or scheme-level deltas on the same underlying profile, so a vendor implementing "FAPI 2.0" has not thereby implemented any *particular* ecosystem. **The profile deliberately leaves choices open** — it "leave[s] a number of choices open to implementors, deployers and/or ecosystems" while warning that a conforming profile "shall not remove or override mandatory behaviors" ✅ — and those open choices (mTLS *or* DPoP; RAR or ecosystem-specific consent; optional CIBA ✅) are where interoperability breaks first. **Clock skew was not specified in FAPI 1 at all**, as Open Banking Limited states outright ✅, so an ecosystem had to publish its own values — a worked example of how a specification gap becomes a coordination cost in every ecosystem; FAPI 2.0 fixed it with explicit bounds ✅. And **`EdDSA` support is contingent on an unregistered identifier** ✅ — a small but real interop hazard.

### 11.3 Is the Message-Signing Tier Worth It? (the attributed debate)

**In favour (attributed: the WG's design and the FAPI 2.0 Advanced level).** Message Signing reached **Final on 25 September 2025** ✅ and "has been formally analysed… for its security and non-repudiation properties" ✅; the WG defines **FAPI 2.0 Advanced as Security Profile + Message Signing**, with "comprehensive" non-repudiation versus "limited" at baseline ✅; and the certification listings show ecosystems adopting it — **CBUAE FAPI 2.0 Message Signing** (OP and RP, Final and ID1) and **KSA Open Banking FAPI 2.0 Message Signing** ✅ — so at least two national ecosystems have made it a certification target. **Against (attributed: the simplicity argument).** Non-repudiation is expensive exactly where it is least convenient (per-message signing, key custody per client) and its benefit is usually **evidentiary rather than preventive** — it helps in a dispute or an audit, not in blocking an attack; the core profile already achieves its security goals without it ✅, and the WG's own simplification rationale cites "less requirement on message signing" as a *benefit* ✅, so an institution that already logs comprehensively may reasonably judge the marginal value low. **Where the decision turns (unmarked judgement):** does a dispute or regulatory process exist in which the bank must *prove* that a TPP sent a specific request, or vice versa? If yes, Message Signing is the mechanism; if no, it is cost without a consumer. The FAPI 1.0 Advanced ecosystem answered this differently — it embedded signing in the mandatory path — and paid the complexity bill.

### 11.4 What FAPI Does Not Address

| Not addressed | Evidence |
|---|---|
| **Business-logic abuse** — a client using valid access to do something legitimate-but-harmful (excessive scraping within consent, gaming a payment pattern) | No FAPI clause addresses request *purpose*; the Attacker Model's goal is that "no attacker can access protected resources other than their own" ✅ — a client acting within its own consent violates no FAPI security goal |
| **The authorised-but-malicious client** | FAPI authenticates and constrains the client; it does not evaluate its intent. Client supervision and onboarding are ecosystem business |
| **Fraud inside a valid consent** — APP fraud, coercion, a customer tricked into consenting | Out of scope: end-user authentication and identity proofing are explicitly excluded ✅ |
| **Data quality and availability obligations** — whether the data is right, or whether the API is up | Out of scope entirely; those are scheme/regulatory service-level obligations carried by the CDR and UK standards themselves |
| **Consent semantics** — what a consent object *means* | The ecosystem's business; FAPI secures the grant, not its meaning ✅ |
| **TLS integrity, JWKS distribution, browsers/devices, internal architecture** | Explicitly out of scope in the Attacker Model ✅ |
| **Implementation quality** | The Attacker Model: implementations "sometimes deviate… and contain security vulnerabilities" ✅; FAPI provides defence in depth, not correctness |

### 11.5 Open Questions as 2.0 Is Adopted

Genuinely open as of September 2026, framed as questions rather than predictions. **Will FAPI 1.0 ecosystems migrate, and when?** The WG calls it "an ecosystem specific decision" ✅ and plans no new 1.0 features ✅; whether the UK's FAPI 1 Advanced position ✅, Australia's FAPI 1.0 Advanced-derived profile ✅ and Brazil's FAPI-BR v2 ✅ move to 2.0 is unresolved here, and ⚠ no published ecosystem migration timetable was located. **Will a baseline-only FAPI 2.0 certification emerge?** Every FAPI 2.0 listing read pairs Security Profile with Message Signing ✅, yet the WG describes baseline and advanced as two *levels* ✅ — not reconciled. **Does DPoP displace mTLS in practice?** FAPI 2.0 makes DPoP a first-class option ✅ for deployment-simplicity reasons ✅, while the deployed ecosystems (UK, Brazil, AU) are mTLS-based ✅. **Do RAR and Grant Management become mandatory by ecosystem rather than by profile?** Both are optional in FAPI ✅ and both solve problems every mature scheme has hand-rolled ✅ — the national-profile vector (FAPI-BR style) is the likely route. **How does non-repudiation get demanded?** The CBUAE and KSA Message Signing certifications ✅ are the only clear ecosystem signals found. And **what happens to the unresolved mandates** — the US §1033 position (§9.5) and the EU's FIDA stage (§9.4) both bear on how much of the world's FAPI demand is regulatory versus commercial, and neither was resolved in this pass.

---

## 12. Cymbal Bank Worked Example — A FAPI-Mandated Market

> **Cymbal Bank is a FICTIONAL institution.** The market, the regulator's instrument, the volumes and all monetary figures below are **illustrative and invented** to walk the design end to end, and every figure is clearly labelled. Nothing here describes a real institution, and no real bank is used as the worked example. Where real standards bodies, specifications and regulators are named, they are named factually as the subject matter.

### 12.1 The Scenario

Cymbal Bank, Singapore, is a mid-size retail and SME bank in a market whose **fictional** regulator mandates a FAPI-based open-banking standard ("the Cymbal Market Open Finance Standard", an invented instrument loosely modelled on the real regimes in §9 — a named profile, a national directory, mandatory certification, a FAPI-derived security profile). Cymbal must expose **account information APIs** (AIS, read-only: balances, transactions, account details) and **payment initiation APIs** (PIS, read-write) to licensed third-party providers. This is the shape of the real problem: a bank acting as AS *and* RS, under a mandated security profile, with TPPs it does not control and consent it must prove.

### 12.2 Target Architecture and Role Split

| Component | FAPI obligation | Cymbal's choice |
|---|---|---|
| **Trust directory** (ecosystem-run) | Provides TPP registration, client metadata, trust list | Consume the directory; do not build a shadow registry ✅ |
| **Authorisation server** | Confidential clients only; mTLS **or** `private_key_jwt`; **sender-constrained tokens only**; PAR; PKCE `S256`; `iss`; code lifetime ≤ 60 s; no password grant ✅ | Buy a **certified** AS ✅ (the UK ecosystem's own advice ✅); configure per-client profiles |
| **Consent & SCA service** | Not FAPI-defined; `acr` / `authorization_details` carriage ✅ | Build: Cymbal's consent record is the system of record (§10.3) |
| **API gateway / resource servers** | Header-only tokens; verify binding, validity, revocation and authorization sufficiency ✅ | Build on the existing gateway; **enforce binding where the client certificate or DPoP proof is visible** ✅ |
| **Key & certificate management** | JWKS TLS-only, no `x5u`/`jku`, no duplicate `kid`; RSA ≥ 2048, EC ≥ 224; `PS256`/`ES256` ✅ | Build: HSM-backed keys, certificate lifecycle, publish-then-sign rotation |
| **Evidence & monitoring** | — | Build: the §10.5 evidence pack |

### 12.3 Chosen Profile and Mechanisms Actually Required

Cymbal selects **FAPI 2.0 Security Profile (Baseline level)** as its target, with **FAPI 1.0 Advanced** retained in parallel for TPPs already certified to that profile — the WG's coexistence answer ✅ makes this a per-client profile choice rather than a cutover. **Required:** PAR ✅, PKCE `S256` ✅, `iss` verification ✅, sender-constrained tokens with **mTLS (primary) and DPoP (secondary)** ✅, `private_key_jwt` as the alternative client authentication ✅, and AS metadata from an authoritative source ✅. **Deliberately not required at this stage:** **Message Signing** (no dispute process currently demands non-repudiation; §11.3), **CIBA** (no redirect-less channel in scope yet; §6.7), **Grant Management** (available for future scheme alignment; §6.8). **RAR** is evaluated and deferred: the Cymbal Market standard defines its own consent object, and RAR's benefit arrives only if the scheme adopts it (§6.5).

### 12.4 Consent and SCA Design

The consent record stores TPP identity, account set, data clusters (balances / transactions / details), payment parameters where applicable (amount, currency, creditor, reference), validity dates, revocation state, and the `acr` achieved at consent. SCA is performed once at consent for AIS (with re-authentication per the scheme's rules) and per payment for PIS. The **enforcement rule** is the important one: on every resource call the gateway checks the token's binding **and** re-reads consent state, comparing amount, payee and account against the token contents — never trusting the scope string alone ✅.

### 12.5 Key and Certificate Management Model

HSM-backed AS signing keys (**`PS256`** ✅); per-TPP certificates issued through the ecosystem directory (`tls_client_auth`), with `self_signed_tls_client_auth` accepted where the directory vouches for the key ✅; JWKS served over TLS with no `x5u`/`jku` ✅ and no duplicate `kid` ✅; publish-then-sign rotation with a refetch-on-unknown-`kid` policy on every verifier (§7.4); certificate expiry watched as a first-class alert (§7.5).

### 12.6 Onboarding and Monitoring of TPPs

**Onboarding:** directory registration → credential issuance → profile assignment (1.0 Advanced or 2.0) → sandbox conformance run → production enablement. **Monitoring:** per-TPP authorisation success rates, the client-authentication method actually used (to catch silent downgrade), token-binding failures, consent-revocation rates, and anomaly detection on data-fetch volume within a valid consent — because **volume abuse inside a valid consent is not a FAPI-detected event** (§11.4).

### 12.7 Conformance and Certification Path

Run the **OpenID Foundation Conformance Suite** (free and open ✅) against the AS as an **OP**, and against the resource-server path as an **RP** where Cymbal also consumes APIs ✅; expect to need **both** certifications if both roles apply ✅; certify against the **ecosystem profile** where the market standard has an OIDF certification profile (the UK / Brazil / Australia CDR / KSA / CBUAE precedent ✅). Re-certify on profile revision and after material change — a certification is a snapshot (§8.4).

### 12.8 Operating Cost and Team Shape — ILLUSTRATIVE FICTIONAL FIGURES

> ⚠ **All figures below are invented for illustration only.** They are not benchmarks, not survey data, and not derived from any real institution. Use them as a shape, not a number.

| Cost element | Illustrative fictional figure | Nature |
|---|---|---|
| Certified AS licence + support | ~USD 0.4–1.2 m / year | Fixed, recurring |
| PKI / HSM / certificate operations | ~USD 0.3–0.6 m / year | Fixed, recurring |
| Build (consent service, gateway enforcement, evidence) | ~USD 2.5–5 m one-off | Fixed, one-off |
| Conformance & certification | ~USD 50–150 k / profile / revision | Recurring per profile |
| TPP onboarding + support desk | ~USD 30–60 k per TPP per year | Scales with participants, not volume |
| Run cost per authorisation flow (incl. PAR round-trip) | sub-cent, volume-driven | Variable |

**Team shape (fictional illustration):** one platform owner for the AS; one for the RS/gateway enforcement; two to three engineers on the consent service; one PKI/key-management specialist (the hardest role to hire and to replace); one conformance/compliance engineer; plus a TPP-facing support function — roughly **10–15 people**, with the PKI and consent-enforcement roles carrying the real risk.

### 12.9 Failure Modes and Mitigations

| Failure mode (§7.5) | Cymbal mitigation |
|---|---|
| Expired TPP certificate breaks a channel | Certificate inventory and expiry alerting at 90/60/30 days; directory reminder integration |
| Stale JWKS after AS key rotation | Publish-then-sign; refetch on unknown `kid`; alert on verification failures by `kid` |
| Consent record drifts from token contents | Consent record as system of record; per-call comparison; reconciliation job and alerting |
| Volume abuse within a valid consent | Anomaly detection outside FAPI's scope (§11.4); per-TPP rate policy |
| Client clock skew rejecting `private_key_jwt` | NTP; HTTP `Date` synchronisation per the spec's NOTE ✅ |
| TLS offload discarding the client certificate | Enforce mTLS binding at the terminator; end-to-end test of the proxy path |

### 12.10 The Regulatory Evidence Cymbal Must Produce

Directory-conformant registration records; per-client certification evidence and current conformance results ✅; full authorisation-flow traces per transaction; token issue/refresh/revoke records with binding type ✅; consent records and their lifecycle ✅; JWKS key inventory and rotation log; certificate inventory with expiry; and rejected-request telemetry showing controls actually firing (§10.5).

### 12.11 Recommendation, and What Cymbal Could Not Establish

**Recommendation (fictional context, real reasoning):** adopt **FAPI 2.0 Security Profile** as the target; keep **1.0 Advanced** per-client for coexistence ✅; buy a **certified** authorisation server rather than building one; build the **consent service and the resource-side enforcement** in-house, because those are where the bank's regulatory exposure actually sits; and defer **Message Signing** until a dispute or regulatory process demands non-repudiation — while designing the key-management layer so it can be added without re-architecture. **What Cymbal could not establish:** whether the scheme will migrate its profile to FAPI 2.0, or on what timetable ⚠; whether the scheme will mandate Message Signing, RAR or Grant Management in a future revision ⚠; whether an OIDF **baseline-only FAPI 2.0 certification** exists ⚠; and how the scheme would respond to the still-unsettled US §1033 position if Cymbal operated in that market ⚠ (§9.5).

---

## 13. The Claims Audit

**✅ = verified at primary source in this pass (all reads dated 17 September 2026); ⚠ = flagged.** Sources: **[WG]** = openid.net/wg/fapi/ · **[SP2]** = FAPI 2.0 Security Profile · **[AM2]** = FAPI 2.0 Attacker Model · **[MS2]** = FAPI 2.0 Message Signing · **[P1]** / **[P2]** = FAPI 1.0 Part 1 / Part 2 · **[CERT]** = openid.net/certification/ + /all-certified-implementations/ · **[RFC]** = rfc-editor.org · **[OBL]** = standards.openbanking.org.uk · **[CDR]** = consumerdatastandardsaustralia.github.io/standards · **[MAS]** = mas.gov.sg · **[CFPB]** = consumerfinance.gov.

| # | Claim | Verdict | Source | Quality | Date |
|---|---|---|---|---|---|
| 1 | FAPI is a security profile of OAuth 2.0/OIDC; the WG "does not work on data models or standards for financial or other APIs. These are ecosystem specific" | ✅ | [WG] | Primary | 2026-09-17 |
| 2 | FAPI 1.0 parts: *Part 1: Baseline* = moderate risk / read-only; *Part 2: Advanced* = high risk / read-write incl. payment initiation | ✅ | [P1], [P2] | Primary spec | 2026-09-17 |
| 3 | FAPI 1.0 Final published **March 2021** (both parts) | ✅ | [WG] + [CDR] normative refs (both parts March 2021) | Primary, dual-source | 2026-09-17 |
| 4 | FAPI 1.0 history: first ID July 2017; second ID October 2018; conformance testing April 2019 | ✅ | [WG] | Primary | 2026-09-17 |
| 5 | FAPI 1.0 remains final, maintained, frozen; no new features planned | ✅ | [WG] | Primary | 2026-09-17 |
| 6 | Per-part P1 vs P2 publication dates individually | ⚠ — no per-part date on the spec pages | — | — | 2026-09-17 |
| 7 | Advanced mandates: JAR request object; `code id_token` or `code`+`response_mode=jwt`; sender-constrained tokens with mTLS; no public clients; `client_secret_jwt` excluded; request-object `aud` / `exp` ≤ 60 min / `nbf` | ✅ | [P2] §5.2.2–5.2.3 | Primary spec | 2026-09-17 |
| 8 | Baseline mandates PKCE `S256`, exact-match pre-registered https redirect URIs, code-reuse rejection, RSA ≥ 2048 / EC ≥ 160, AT lifetime < 10 min unless sender-constrained; explicitly has **no** request-integrity protection | ✅ | [P1] §5.2.2–5.2.4 | Primary spec | 2026-09-17 |
| 9 | **FAPI 2.0 Security Profile** — published **22 Feb 2025**, status **Final** | ✅ | [SP2] header | Primary spec | 2026-09-17 |
| 10 | **FAPI 2.0 Attacker Model** — published **22 Feb 2025**, status **Final** | ✅ | [AM2] header | Primary spec | 2026-09-17 |
| 11 | WG announces FAPI 2.0 approved FINAL **2025-02-19** | ✅ | [WG] news | Primary | 2026-09-17 |
| 12 | **FAPI 2.0 Message Signing** — published **25 Sep 2025**, status **Final** | ✅ | [MS2] header | Primary spec | 2026-09-17 |
| 13 | Message Signing page retains draft boilerplate despite Final status | ✅ observed | [MS2] body | Primary — **internal inconsistency** | 2026-09-17 |
| 14 | FAPI 2.0 levels: baseline (secure against the attacker model) and advanced (adds non-repudiation); baseline ≈ FAPI 1.0 Advanced security | ✅ | [WG] | Primary | 2026-09-17 |
| 15 | §5.5 diff table: JAR→PAR; JARM→code-only; `s_hash`→PKCE; pre-registered redirect URIs→redirect URIs in PAR; `code id_token`/`code`→`code`; detached-signature ID token→PKCE; `x-fapi-*` moved out; mTLS→mTLS or DPoP | ✅ | [SP2] §5.5 | Primary spec | 2026-09-17 |
| 16 | FAPI 2.0 mandates PAR, PKCE `S256`, `iss` check, sender-constrained tokens (mTLS or DPoP), confidential clients only, password grant rejected, no open redirectors, code lifetime ≤ 60 s, no refresh rotation "except in extraordinary circumstances", clock skew 0–10 s accept / >60 s reject | ✅ | [SP2] §5.3–5.4 | Primary spec | 2026-09-17 |
| 17 | Algorithms: `PS256`, `ES256`, `EdDSA` (Ed25519); no `none`; RSA ≥ 2048; EC ≥ 224; ≥ 128-bit credential entropy | ✅ | [SP2] §5.4.1 | Primary spec | 2026-09-17 |
| 18 | Attackers A1, A1a, A2, A3a, A4, A5; A4 obsolete in 2.0; goals = authorization, authentication, session integrity (both directions) | ✅ | [AM2] §5–§7 | Primary spec | 2026-09-17 |
| 19 | AM2 scope excludes TLS integrity, JWKS distribution, browsers/devices, identity & session management, weak RNG, internal architecture, implementation errors | ✅ | [AM2] §6, §8 | Primary spec | 2026-09-17 |
| 20 | PAR = **RFC 9126**, Sept 2021 | ✅ | [RFC]/[SP2] | Primary | 2026-09-17 |
| 21 | JAR = **RFC 9101**, Aug 2021 | ✅ | [RFC] | Primary | 2026-09-17 |
| 22 | mTLS + certificate-bound tokens = **RFC 8705**, Feb 2020 | ✅ | [SP2] | Primary | 2026-09-17 |
| 23 | DPoP = **RFC 9449**, Sept 2023 | ✅ | [SP2] | Primary | 2026-09-17 |
| 24 | RAR (`authorization_details`) = **RFC 9396**, May 2023 | ✅ | [RFC] | Primary | 2026-09-17 |
| 25 | **JARM is an OIDF specification, NOT an IETF RFC**; page titled "Draft-02…"; defines `query.jwt`/`fragment.jwt`/`form_post.jwt`/`jwt` | ✅ | openid.net/specs/openid-financial-api-jarm.html | Primary | 2026-09-17 |
| 26 | PKCE = **RFC 7636**, Sept 2015 | ✅ | [SP2] | Primary | 2026-09-17 |
| 27 | Issuer identification = **RFC 9207**, Mar 2022 | ✅ | [SP2] | Primary | 2026-09-17 |
| 28 | AS metadata = **RFC 8414**, June 2018 | ✅ | [SP2] | Primary | 2026-09-17 |
| 29 | OAuth Security BCP = **RFC 9700 / BCP 240**, Jan 2025 | ✅ | [SP2] | Primary | 2026-09-17 |
| 30 | JWT BCP = **RFC 8725 / BCP 225** | ✅ | [SP2] | Primary | 2026-09-17 |
| 31 | CIBA = *OIDC Client-Initiated Backchannel Authentication Flow - Core 1.0*, dated **1 Sept 2021** | ✅ | [SP2] refs | Primary | 2026-09-17 |
| 32 | Grant Management = OIDF draft **`oauth-v2-grant-management-03`, 9 May 2023**, status draft; renamed from `fapi-grant-management.html` | ✅ | openid.net/specs/oauth-v2-grant-management.html | Primary | 2026-09-17 |
| 33 | OIDC4IDA 1.0 incorporating **errata set 1** published **1 July 2026**, Final, eKYC-IDA WG | ✅ | openid.net/specs/openid-connect-4-identity-assurance-1_0.html | Primary | 2026-09-17 |
| 34 | **OAuth 2.1 is NOT a published RFC**; `draft-ietf-oauth-v2-1` at `-16`; FAPI 2.0 does **not** depend on it normatively | ⚠ — datatracker refused extraction; revision from search metadata | datatracker + secondary | Weak primary access | 2026-09-17 |
| 35 | Two-stage programme (free conformance testing; paid certification with the "OpenID Certified" mark); **independent certification via Authorized Auditors / Testing Service Providers still in development** | ✅ | [CERT] | Primary | 2026-09-17 |
| 36 | Certification families: FAPI1-Advanced OP (general, UK Open Banking, AU CDR, Brazil Open Banking / Open Finance FAPI-BR v2 / Open Insurance variants, KSA Open Banking, Second ID), FAPI CIBA OP, FAPI1-Advanced RP (mirrored), FAPI 2.0 OP/RP ("Security Profile Final & Message Signing Final" pairings, Australia ConnectID, CBUAE Message Signing, KSA Message Signing) | ✅ | [CERT] index | Primary | 2026-09-17 |
| 37 | FAPI 2.0 conformance tests launched **March 2023**; Authlete, Cloudentity, ConnectID, Ping Identity, Raidiam named | ✅ | [WG] | Primary | 2026-09-17 |
| 38 | UK: 15 banks / 31 FAPI certifications / 16 deployments; most CMA9 certified; annual recertification anticipated | ✅ | [WG] | Primary | 2026-09-17 |
| 39 | FAPI and OpenID Connect certifications are orthogonal | ✅ | [WG] | Primary | 2026-09-17 |
| 40 | UK adopts **FAPI 1**; v3 used FAPI 1 ID2 (deprecated, support/certification removed **Dec 2024**); v4 uses the **final FAPI 1 Advanced** per a Technical Design Authority vote; Directory issues PSD2/PSR 2017-compliant certificates; **JARM out of scope**; CIBA **optional**; `acr` values `urn:openbanking:psd2:sca` / `:ca`; FAPI 1 silent on clock skew | ✅ | [OBL] | Primary (scheme) | 2026-09-17 |
| 41 | AU CDR profile "builds upon the foundations of the Financial-grade API Advanced Profile"; normative refs date FAPI 1.0 P1 & P2 March 2021 and JARM October 2020; `x-fapi-*` headers mandated | ✅ | [CDR] | Primary (scheme) | 2026-09-17 |
| 42 | Brazil operates national FAPI profiles including **FAPI-BR v2**; old developer area deactivated **21 July 2022** | ✅ profile names / ⚠ BCB instrument | [CERT] + openbanking-brasil.github.io | Primary/partial | 2026-09-17 |
| 43 | SGFinDex is an MAS+GovTech infrastructure on Singpass with 15 participating FIs; **FAPI not named** on the MAS page | ✅ | [MAS] | Primary | 2026-09-17 |
| 44 | India RBI Account Aggregator / ReBIT — whether FAPI is named | ⚠ **NOT ESTABLISHED** — sources inaccessible | — | — | 2026-09-17 |
| 45 | EU PSD2 / EBA RTS on SCA & CSC / eIDAS QWAC / FIDA — instruments, mechanics, stage, and whether FAPI is named | ⚠ **UNVERIFIED** — EBA and EUR-Lex extraction failed | — | — | 2026-09-17 |
| 46 | OIDF–FDX liaison exists; FDX "working closely with the FAPI WG to implement the specs in North America" | ✅ | [WG] | Primary | 2026-09-17 |
| 47 | CFPB §1033 codified at **12 CFR Part 1033**, "most recently amended Jan. 17, 2025", published as "the current regulation" | ✅ | [CFPB] | Primary | 2026-09-17 |
| 48 | §1033: finalised Oct 2024; phased compliance 2026–2030 starting April 2026; **Forcht Bank, N.A. v. CFPB** (E.D. Ky.) preliminary injunction **29 Oct 2025**; deadlines tolled; appeal stayed pending rulemaking; reconsideration proposal with the White House | ⚠ — secondary reporting, not verified at docket or CFPB | fintechly / openbankingtracker / consumerfinancemonitor | Secondary | 2026-09-17 |
| 49 | Per-implementation certification listing rows (which vendor, which revision, Sept 2026) | ⚠ **NOT VERIFIED** — index verified, rows not opened | — | — | 2026-09-17 |
| 50 | Conformance-suite internal test-module inventory | ⚠ **NOT VERIFIED** | — | — | 2026-09-17 |
| 51 | Open-source authorisation-server landscape for FAPI | ⚠ **NOT VERIFIED** — only test tooling verified | — | — | 2026-09-17 |
| 52 | The anchor quote at oracle_banking_microservices_architecture_guide.md line 688 | ✅ | repo file, read in this pass | Primary (repo) | 2026-09-17 |

**Rejected claims (asserted in common secondary material, not repeated here):** that JARM is an IETF RFC; that FAPI 2.0 depends on OAuth 2.1; that "FAPI 2.0 is more secure than FAPI 1.0" (the WG explicitly declines this framing ✅); that a FAPI certification is a security audit; that any regulator outside those verified in §9 mandates FAPI; and that FAPI 1.0 has been superseded and retired.

---

## 14. What Could Not Be Verified

Explicit negative findings. Each is something a reader might reasonably expect this guide to state, and each is something this pass could **not** establish. **None of these gaps has been filled by guessing.**

1. **OAuth 2.1's publication status at primary source.** `datatracker.ietf.org` refused extraction for both the doc page and the HTML rendering; the revision number (`-16`) comes from search-result titles and the "not yet an RFC" conclusion from a secondary blog. Re-check the datatracker before citing OAuth 2.1's status. *Unaffected:* FAPI 2.0's normative dependencies were read directly from the spec ✅ and do not include OAuth 2.1.
2. **Per-part publication dates for FAPI 1.0 Part 1 and Part 2 separately.** The spec pages carry no header date; only the collective "March 2021" is verified ✅, corroborated by the Australian CDR normative references ✅ but still not a per-part OIDF statement.
3. **The contents of individual certification listing pages.** The certification **index** was read ✅; the per-implementation rows were **not** opened. Every vendor-certification claim in §10.6 is a *listing-name* claim, not a confirmed current status.
4. **The conformance suite's test-module inventory.** The suite's existence, licence and purpose were verified ✅; the specific FAPI tests were not enumerated. The suite source is the authoritative list.
5. **Whether a FAPI 2.0 baseline-only (Security Profile without Message Signing) certification exists.** Every FAPI 2.0 listing read pairs the two ✅; the WG's level description implies baseline should be certifiable alone ✅; the two facts were not reconciled. **Explicitly unresolved.**
6. **India's Account Aggregator technical standards and whether they name FAPI.** `rebit.org.in` failed (server error) and the Sahamati documentation endpoint was blocked as a private/internal address. **No RBI/ReBIT primary document was read.** Do not assert an Indian FAPI position.
7. **The EU's PSD2 / EBA RTS / FIDA instrument details.** The EBA URL returned "Page not found"; two EUR-Lex URLs failed. **No EU primary document was read.** Directive numbers, the delegated regulation, the QWAC mechanics and FIDA's current stage remain structural knowledge, flagged in §9.4.
8. **Whether the SGFinDex technical standards reference FAPI internally.** The public MAS page does not ✅; the technical specifications were not accessible. MAS's "Finance-as-a-Service: API Playbook" was identified ⚠ but not read.
9. **Brazil's specific BCB/CMN instrument and the text of the FAPI-BR v2 profile.** The national profile's existence is verified via OIDF certification listings ✅; its content and its regulatory instrument are not.
10. **The US §1033 litigation record at primary source.** No court docket, CFPB newsroom page or CFPB rulemaking page for the reconsideration was retrieved. The injunction date, docket and appeal status are **secondary** (§13 item 48). The rule's codification **is** ✅ primary.
11. **The open-source authorisation-server landscape for FAPI.** Only *test* tooling was verified (the OIDF conformance suite ✅ and the Open Banking UK model bank ✅). No open-source production authorisation server was verified as FAPI-conformant, and none is named.
12. **Vendor product capabilities.** No vendor documentation was read; vendor names in §10.6 appear only because they appear in OIDF certification listings ✅.
13. **Any published FAPI migration guide or ecosystem migration timetable.** The WG's coexistence answer was verified ✅; a formal migration guide was not found.
14. **Any verified cost, effort or staffing benchmark for FAPI.** No credible primary or authoritative figure was found; §10.8 and §12.8 give **shapes** and clearly-labelled **fictional** figures, never asserted as real.
15. **The FAPI 2.0 "Implementation and Deployment Advice" document** (named by the §5.5 diff table as the new home of the `x-fapi-*` headers ✅). Its location, contents and status were not established. **Explicitly unresolved** — it is named only as the destination of a moved requirement.
16. **JARM's final publication status.** The page read is titled **Draft-02** ✅ and carries draft boilerplate; whether a later final revision exists was not established. Cited here by name and provenance only, never by version or date of finality.

---

## 15. Glossary — Including the Identifier Decoder

**Part A — Concepts and roles**

| Term | Meaning |
|---|---|
| **FAPI** | Financial-grade API — now styled simply "FAPI" by the OpenID Foundation; a family of high-security OAuth 2.0 profiles. Not an API specification, not a business standard, not a protocol (§1.2) |
| **FAPI WG** | The OpenID Foundation working group that produces FAPI; chaired by Nat Sakimura per the WG page ✅ |
| **ASPSP** | Account Servicing Payment Service Provider — the bank holding the account (EU/UK/PSD2 vocabulary) |
| **TPP / AISP / PISP** | Third-Party Provider; Account Information Service Provider (reads data with consent); Payment Initiation Service Provider (initiates a payment order with consent) |
| **Data Holder / Data Recipient** | Australian CDR and FDX vocabulary for ASPSP and TPP respectively ✅ (as used in the Grant Management draft) |
| **AS / RS / OP / RP** | Authorisation Server; Resource Server; OpenID Provider (OIDC term for the AS); Relying Party (OIDC term for the client). AS/OP and RS/RP are the two roles a bank occupies (§10.1) |
| **SCA** | Strong Customer Authentication — an EU/PSD2 concept (two independent factors); **not defined by FAPI**, and end-user authentication is explicitly out of the Attacker Model's scope ✅ |
| **Consent vs Grant** | Consent is a legal concept; a grant is "an authorization created as a result of consent" — an AS-managed resource ✅ (Grant Management draft) |
| **Non-repudiation** | The property that a party can later be *proved* to have sent a message; the distinguishing feature of FAPI 2.0 "advanced" and of Message Signing ✅ |
| **Attacker Model** | The distinct FAPI 2.0 artefact stating security goals, attacker classes and limitations, from which the mechanisms are derived ✅ |
| **Conformance Suite** | The OpenID Foundation's free, open-source test tool producing certification evidence ✅ |

**Part B — The identifier decoder: every acronym with its correct RFC or specification**

| Identifier | Term | RFC / specification | Date |
|---|---|---|---|
| **PAR** | Pushed Authorization Requests | **RFC 9126** ✅ | September 2021 |
| **JAR** | JWT-Secured Authorization Request (request object) | **RFC 9101** ✅ | August 2021 |
| **JARM** | JWT Secured Authorization Response Mode | **OpenID Foundation specification — NOT an IETF RFC** ✅ | page read = *Draft-02*; finality not established ⚠ |
| **mTLS** | Mutual TLS client auth + certificate-bound access tokens | **RFC 8705** ✅ | February 2020 |
| **DPoP** | Demonstrating Proof of Possession | **RFC 9449** ✅ | September 2023 |
| **RAR** | Rich Authorization Requests (`authorization_details`) | **RFC 9396** ✅ | May 2023 |
| **PKCE** | Proof Key for Code Exchange | **RFC 7636** ✅ | September 2015 |
| **`iss`** | Authorization Server Issuer Identification | **RFC 9207** ✅ | March 2022 |
| **CIBA** | Client-Initiated Backchannel Authentication | **OpenID Connect CIBA Flow - Core 1.0** (OIDF) ✅ | 1 September 2021 |
| **Grant Management** | Grant Management for OAuth 2.0 | **OIDF draft `oauth-v2-grant-management-03`** ✅ | 9 May 2023 |
| **OIDC4IDA** | OpenID Connect for Identity Assurance 1.0 inc. errata set 1 | **OpenID Foundation (eKYC-IDA WG)** ✅ | errata-set-1 revision published 1 July 2026 |
| **OAuth Security BCP** | Best Current Practice for OAuth 2.0 Security | **RFC 9700 / BCP 240** ✅ | January 2025 |
| **JWT BCP** | JSON Web Token Best Current Practices | **RFC 8725 / BCP 225** ✅ | February 2020 |
| **AS Metadata** | OAuth 2.0 Authorization Server Metadata | **RFC 8414** ✅ | June 2018 |
| **DCR / Client Config** | Dynamic Client Registration / Client Configuration | **RFC 7591 / RFC 7592** ✅ (referenced by FAPI 2.0) | 2015 |
| **`cnf` / `x5t#S256`** | Token confirmation claim / certificate SHA-256 thumbprint | **RFC 8705** ✅ | February 2020 |
| **OAuth 2.0 core** | Authorization Framework / Bearer Token Usage | **RFC 6749 / RFC 6750** ✅ | October 2012 |
| **OIDC** | OpenID Connect Core 1.0 inc. errata set 1 | **OpenID Foundation** ✅ | 8 November 2014 |
| **OAuth 2.1** | The OAuth 2.1 Authorization Framework | **Internet-Draft `draft-ietf-oauth-v2-1-16` — NOT a published RFC** ⚠ | flagged |
| **Transport refs** | Secure Use of TLS (BCP 195) · Service Identity in TLS (**RFC 9525**) · CAA (**RFC 8659**) · HSTS (**RFC 6797**) · JWT access tokens (**RFC 9068**) | ✅ (all referenced by FAPI 2.0) | RFC 8659 = November 2019 |

---

## 16. Cross-References and Further Reading

**Boundary declaration (restated).** This guide owns the **FAPI standard itself** — its threat model, its 1.0 and 2.0 versions and profiles, the mechanisms it mandates with their identifiers, the certification programme and the regulatory footprint. It does **not** own and does not re-derive: the **Open Bank Project platform** ([openbankproject_guide.md](openbankproject_guide.md) — TESOBE's software, API, deployments and open-source model); **general OAuth 2.0/OIDC fundamentals** ([../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md)); **API lifecycle governance** ([../technology/api_governance_guide.md](../technology/api_governance_guide.md)); **payment clearing and settlement** ([payment_rails_guide.md](payment_rails_guide.md)); **scheme and acquiring economics** ([adyen_guide.md](adyen_guide.md), [mastercard_guide.md](mastercard_guide.md), [visa_guide.md](visa_guide.md)); **the Singapore regulatory overlay** ([mas_regulations_guidelines_guide.md](mas_regulations_guidelines_guide.md)); or **API-first banking architecture** ([oracle_banking_microservices_architecture_guide.md](oracle_banking_microservices_architecture_guide.md), whose §9.3 supplies this guide's framing quote).

**Primary specifications, listed properly.**

| Document | Publisher | Status / date | URL |
|---|---|---|---|
| Financial-grade API Security Profile 1.0 — Part 1: Baseline | OpenID Foundation | Final; March 2021 ✅ | [openid.net/specs/openid-financial-api-part-1-1_0-final.html](https://openid.net/specs/openid-financial-api-part-1-1_0-final.html) |
| Financial-grade API Security Profile 1.0 — Part 2: Advanced | OpenID Foundation | Final; March 2021 ✅ | [openid.net/specs/openid-financial-api-part-2-1_0-final.html](https://openid.net/specs/openid-financial-api-part-2-1_0-final.html) |
| FAPI 2.0 Security Profile | OpenID Foundation | Final; 22 February 2025 ✅ | [openid.net/specs/fapi-security-profile-2_0-final.html](https://openid.net/specs/fapi-security-profile-2_0-final.html) |
| FAPI 2.0 Attacker Model | OpenID Foundation | Final; 22 February 2025 ✅ | [openid.net/specs/fapi-attacker-model-2_0-final.html](https://openid.net/specs/fapi-attacker-model-2_0-final.html) |
| FAPI 2.0 Message Signing | OpenID Foundation | Final; 25 September 2025 ✅ | [openid.net/specs/fapi-message-signing-2_0-final.html](https://openid.net/specs/fapi-message-signing-2_0-final.html) |
| Financial-grade API: JWT Secured Authorization Response Mode (JARM) | OpenID Foundation | page read = Draft-02 ✅ / finality ⚠ | [openid.net/specs/openid-financial-api-jarm.html](https://openid.net/specs/openid-financial-api-jarm.html) |
| Grant Management for OAuth 2.0 | OpenID Foundation (FAPI WG) | Draft `-03`; 9 May 2023 ✅ | [openid.net/specs/oauth-v2-grant-management.html](https://openid.net/specs/oauth-v2-grant-management.html) |
| OpenID Connect CIBA Flow — Core 1.0 | OpenID Foundation | 1 September 2021 ✅ | [openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) |
| OpenID Connect for Identity Assurance 1.0 inc. errata set 1 | OpenID Foundation (eKYC-IDA) | Final; 1 July 2026 ✅ | [openid.net/specs/openid-connect-4-identity-assurance-1_0.html](https://openid.net/specs/openid-connect-4-identity-assurance-1_0.html) |
| FAPI Working Group (charter, FAQ, adoption references) | OpenID Foundation | live; read 17 Sep 2026 ✅ | [openid.net/wg/fapi/](https://openid.net/wg/fapi/) |
| OpenID Certification programme | OpenID Foundation | live ✅ | [openid.net/certification/](https://openid.net/certification/) · [all certified implementations](https://openid.net/certification/all-certified-implementations/) |
| OpenID Conformance Suite | OpenID Foundation (open source) | live ✅ | [gitlab.com/openid/conformance-suite](https://gitlab.com/openid/conformance-suite) · [certification.openid.net](https://www.certification.openid.net/) |
| **RFCs**: 6749, 6750, 6797, 7591, 7592, 7636, 8414, 8659, 8705, 8725, 9068, 9101, 9126, 9207, 9396, 9449, 9525, 9700, 9701 | IETF | see §15 Part B for each date ✅ | [rfc-editor.org](https://www.rfc-editor.org/) |
| OAuth 2.1 Authorization Framework | IETF | Internet-Draft `draft-ietf-oauth-v2-1-16`; **not a published RFC** ⚠ | [datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| Open Banking Standards — Security Profiles | Open Banking Limited | live ✅ | [standards.openbanking.org.uk/security-profiles/](https://standards.openbanking.org.uk/security-profiles/) |
| CDR Data Standards — Security Profile | Australian Data Standards Body | live ✅ | [consumerdatastandardsaustralia.github.io/standards](https://consumerdatastandardsaustralia.github.io/standards/) |
| SGFinDex | Monetary Authority of Singapore | live ✅ | [mas.gov.sg/development/fintech/sgfindex](https://www.mas.gov.sg/development/fintech/sgfindex) |
| Personal Financial Data Rights, 12 CFR Part 1033 | Consumer Financial Protection Bureau | codified; **enjoined** ⚠ | [consumerfinance.gov/rules-policy/regulations/1033/](https://www.consumerfinance.gov/rules-policy/regulations/1033/) |

**Sibling and related guides in this repo.** [The Open Bank Project (OBP)](openbankproject_guide.md) — the platform, not the standard · [Oracle Banking Microservices Architecture](oracle_banking_microservices_architecture_guide.md) — §9.3 the framing quote, §9.4 API security controls · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Singapore regime (cross-ref §9.4) · [Payment Rails](payment_rails_guide.md) — what happens after a PISP-initiated payment authorises · [Adyen](adyen_guide.md) / [Mastercard](mastercard_guide.md) / [Visa](visa_guide.md) — scheme and acquiring context · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — the repo's verification-ledger conventions · [../technology/distributed_auth_guide.md](../technology/distributed_auth_guide.md), [../technology/api_governance_guide.md](../technology/api_governance_guide.md), [../technology/cybersecurity_guide.md](../technology/cybersecurity_guide.md), [../technology/security_by_design_guide.md](../technology/security_by_design_guide.md), [../technology/enterprise_middleware_integration_platform_guide.md](../technology/enterprise_middleware_integration_platform_guide.md) — the adjacent disciplines.

---

## 17. Closing Summary

FAPI is a **security profile**, and holding that fact steady is most of the work. It constrains an OAuth 2.0 and OpenID Connect deployment for high-value APIs; it does not define an API, a business standard, a protocol, a regulatory mandate, or a security audit. The OpenID Foundation's own sentence — "The FAPI WG does not work on data models or standards for financial or other APIs. These are ecosystem specific" ✅ — is the boundary this guide has respected, and the UK, Australian, Brazilian and US regimes in §9 show exactly what sits on either side of it: the scheme owns the API and the consent semantics; FAPI owns the security posture.

The intellectual core is §2. FAPI 2.0 derives its mechanisms from a stated **attacker model** and four security goals ✅ rather than from a list of known threats ✅, which is why it can claim coverage of attacks nobody has enumerated — and why it explicitly publishes what it does *not* cover (TLS integrity, key distribution, the user's device, end-user authentication, internal architecture, implementation quality) ✅. Read the out-of-scope list as carefully as the requirements; a deployment that stops at "FAPI-compliant" has stopped early.

The generational story is a story about subtraction. FAPI 1.0 Advanced protected the front channel four times over — signed request object, JARM response, detached-signature ID token with `s_hash`, certificate-bound tokens ✅ — and the working group's own diff table shows 2.0 replacing that with PAR, PKCE, a code-only response and a choice of token binding ✅, for reasons it states plainly: redundancy across a browser-mediated channel bought implementation surface more than safety. Nothing was downgraded in the security goal; non-repudiation was unbundled into the Message Signing profile, final on 25 September 2025 ✅.

The practical reading for an institution is §10 and §12. The hard parts are key and certificate lifecycle, resource-side consent enforcement, and the AS/RS split across teams; the laborious parts are everything else. Certification proves conformance, not security, and the programme's independent-auditor tier is still being built ✅. And the regulatory picture is uneven in a way worth remembering: the UK names FAPI by version ✅, Australia builds on FAPI 1.0 Advanced in its normative references ✅, Brazil certifies a national FAPI-BR v2 ✅, Singapore solves the same problem with national identity and a central gateway instead ✅, India and the EU could not be verified in this pass ⚠, and the United States has an industry standard alongside a rule that is codified, enjoined and under reconsideration ✅⚠.

The one sentence to carry away is the one the whole profile exists to make true: in a FAPI deployment, a stolen token is not a stolen account, because the token is useless without the key it is bound to — the bound token.
