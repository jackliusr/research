# Authentication and Authorisation in a Distributed System: The Identity Architecture Deep-Dive — A Comprehensive Guide

*A deep-dive on the identity architecture of distributed systems: the authentication-vs-authorisation distinction, the distributed problem, the authentication mechanisms (passwords → SAML → OAuth 2.0 → OIDC → JWT → mTLS → WebAuthn), sessions vs tokens, distributed session stores, token validation via JWKS, the authorisation models (RBAC, ABAC, ReBAC), policy engines (OPA), service-to-service identity (SPIFFE/SPIRE), SSO and enterprise patterns, the banking context (PSD2/SCA), and a worked microservices auth design.*

**Series:** System Design / Distributed Systems — Security (technology/)
**Audience:** Solution architects, platform engineers, security architects, API platform teams
**Cross-references:** [api_governance_guide.md](api_governance_guide.md) (API security toolbox §8.1 — OAuth 2.0/OIDC/JWT/mTLS governance, gateway runtime §7, Merlion Bank worked example §10.3 — this guide is the *dedicated identity-architecture* deep-dive behind that toolbox), [distributed_rate_limiter_guide.md](distributed_rate_limiter_guide.md) (gateway enforcement layer — authn/authz and rate limiting are sibling gateway controls), [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) (key/secret handling — the private keys behind signing, the client secrets behind OAuth), [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) (testing the identity layer), [adversarial_ml_attacks_guide.md](adversarial_ml_attacks_guide.md) (credential-stuffing/abuse vectors at scale), [mcp_discovery_guide.md](ai_llm/mcp_discovery_guide.md) and [mcp_framework_tools_guide.md](ai_llm/mcp_framework_tools_guide.md) (OAuth 2.x authorization for MCP servers — the identity layer applied to agent tooling), [enterprise_agentic_platform_architecture_guide.md](ai_llm/enterprise_agentic_platform_architecture_guide.md) (agent auth patterns), and the banking series under `../banking/` (PSD2/SCA context: e.g. `core_banking_systems_guide.md`, `payments_hub_guide.md`, `programmable_business_bank_guide.md`).

---

## Table of Contents

1. [Authn vs Authz: The Distinction](#1-authn-vs-authz-the-distinction)
2. [The Authentication Mechanisms](#2-the-authentication-mechanisms)
3. [Session vs Token](#3-session-vs-token)
4. [The Distributed Sessions](#4-the-distributed-sessions)
5. [Token Validation](#5-token-validation)
6. [The Authorisation Models](#6-the-authorisation-models)
7. [The Policy Engines](#7-the-policy-engines)
8. [Service-to-Service](#8-service-to-service)
9. [SSO and the Enterprise Patterns](#9-sso-and-the-enterprise-patterns)
10. [The Banking Context](#10-the-banking-context)
11. [Worked Example: A Microservices Auth Design](#11-worked-example-a-microservices-auth-design)
12. [The Summary: Auth in One Page](#12-the-summary-auth-in-one-page)
13. [Verification and Claims-Status](#13-verification-and-claims-status)
14. [Glossary](#14-glossary)

---

## 1. Authn vs Authz: The Distinction

### 1.1 The Distinction: Authentication (the Who)

**Authentication** — *authn* — answers the question **"who are you?"** It is the act of *proving* an identity claim: the system verifies that the caller is who they say they are. The verb is *verify the claimant*.

The canonical model is the **authentication factors** — the three categories of evidence a claimant can offer:

| Factor | Category | Examples |
|---|---|---|
| **Something you know** | Knowledge | Password, PIN, security answer |
| **Something you have** | Possession | Phone/SMS OTP, hardware token, authenticator app, smart card |
| **Something you are** | Inherence | Fingerprint, face, iris, voice |

Multi-factor authentication (MFA) requires **two or more factors from different categories** — the PSD2 Strong Customer Authentication (SCA) rule in EU banking uses exactly this three-factor model (see §10.2). The factor model is the frame that makes "password + SMS code" meaningfully stronger than "password + security question" (two knowledge factors — still one category).

Authentication establishes a **principal** — the identity that subsequent requests will be attributed to: a user (`sub: user-4821`), a machine (a service account), or a workload (an SVID, §8). Crucially, *authentication alone grants nothing*. It is the admission ticket to the authorisation conversation, not the permission itself.

### 1.2 Authorization (the What)

**Authorization** — *authz* — answers the question **"what are you allowed to do?"** It is the act of *deciding* whether an authenticated principal may perform a specific action on a specific resource, under the current conditions. The verb is *verify the permission*.

Every authorisation decision decomposes into the same shape:

```
Decision = f(subject, action, resource, context)
```

- **Subject** — the authenticated principal (user, service, workload) plus its attributes (roles, groups, clearance, organisation).
- **Action** — the operation: `read`, `write`, `delete`, `execute`, `transfer`, `approve`.
- **Resource** — the object: an account, a document, a payment order, an API endpoint.
- **Context** — the surrounding conditions: time of day, device, IP/geo, risk score, compliance flag, amount.

The decision is a yes/no (allow/deny) — but *who* computes it, and *where* it is enforced, are the design questions this guide spends most of its length on (§6, §7, §11). Two widely-used enforcement acronyms:

- **PEP** — Policy Enforcement Point: the component in the request path that *enforces* the decision (blocks or forwards).
- **PDP** — Policy Decision Point: the component that *computes* the decision (pure logic, no enforcement).

A classic failure mode is collapsing the two questions: systems that "authenticate" with a bearer token and then assume the caller may do *anything* the token's scopes say, without a per-request authorisation decision at the resource. That is how **BOLA / broken object-level authorization** happens — the #1 API flaw in OWASP's API Security Top 10 (API1:2023, cross-ref api_governance_guide.md §8.2). The OWASP list is essentially a taxonomy of what happens when authz is bolted on after authn instead of designed alongside it.

### 1.3 The Distributed Problem

In a single monolithic application, authn and authz are easy: one process holds the user database, the session, and the permission logic. The moment the system becomes **distributed** — many services, many nodes, many teams — three things break:

**The multi-service problem.** A user request now fans out across dozens of services (account service, payments service, cards service, ledger, notifications). Every one of those services must know *who* the caller is and *what* they may do — but each is a separate process, possibly a separate team's codebase. Either every service re-implements identity (nine authn schemes, zero consistency — the exact starting state of Merlion Bank in api_governance_guide.md §10.3), or identity is factored into a shared architecture (an identity provider + tokens + a policy layer — the subject of this guide). The second option is the only one that scales.

**The sessions problem.** Session state is by definition stateful. In a distributed system, the session that was created on gateway node 1 may be checked on gateway node 2, 3, or a backend service. Where does that state live? Sticky sessions, a shared store, or — the modern answer — nowhere, because stateless tokens carry the state instead (§3, §4).

**The keys problem.** Tokens and sessions are only as trustworthy as the cryptography behind them, and cryptography runs on keys. In a distributed system the keys that *sign* tokens, the keys that *verify* them, and the keys that identify *services* to each other are all distributed assets: they must be generated safely, stored safely (cross-ref [openbao_vs_vault_guide.md](openbao_vs_vault_guide.md) for the secrets-management layer), rotated on schedule, and revoked on compromise — across many machines, without ever leaking to the wrong one. Key distribution is the distributed problem's silent third leg: "the user authenticated" is only meaningful if the verification keys can be trusted (§5, §8).

The distributed problem in one line: **identity must be verified once, and its consequences carried everywhere — without every service re-implementing authentication, duplicating session state, or trusting keys it cannot verify.**

### 1.4 The Distinction Table

| Aspect | Authentication (authn) | Authorization (authz) |
|---|---|---|
| **Question** | Who are you? | What may you do? |
| **Verb** | Verify the claimant | Verify the permission |
| **Order** | First — establishes the principal | Second — consumes the principal |
| **Failure mode** | Impersonation: attacker proves a false identity | Over-permission: a real identity does what it shouldn't (BOLA, privilege escalation) |
| **Mechanisms** | Passwords, OTP, SAML, OIDC, mTLS, WebAuthn (§2) | RBAC, ABAC, ReBAC, policy engines (§6, §7) |
| **State** | Identity proof, sessions, credentials | Policies, roles, relationships, attributes |
| **Enforcement point** | Login flow, gateway authn, TLS/mTLS handshake | Per-request decision at the API/resource (PEP/PDP) |
| **Analogy** | Showing your ID at the door | What the ID entitles you to do inside |
| **Common error** | Assuming login = trust forever (no expiry, no re-auth) | Assuming authentication implies permission (the OWASP BOLA trap) |

---

## 2. The Authentication Mechanisms

A tour of the mechanisms, oldest to newest, with the verification status of each claim. The governance framing of this same toolbox (standards, use cases, governance notes) is in api_governance_guide.md §8.1 — this section is the *how it works* deep-dive.

### 2.1 Passwords

The oldest and still the most common mechanism: the claimant proves knowledge of a secret shared with the system.

- **Storage discipline (non-negotiable):** never store plaintext; store only salted, slow-hash digests — bcrypt, scrypt, Argon2id (the memory-hard KDFs). Fast hashes (MD5, SHA-1, plain SHA-256) are crackable at billions of guesses per second on commodity GPUs.
- **The catalog of password attacks:** credential stuffing (reusing breaches at scale — the adversarial-ML angle in adversarial_ml_attacks_guide.md), brute force (mitigated by rate limiting — cross-ref distributed_rate_limiter_guide.md), phishing (password entry on a fake page — passwords are *not* phishing-resistant, unlike WebAuthn, §2.7), rainbow tables (defeated by per-user salts), and password spraying (few passwords, many accounts).
- **The ecosystem answer:** password + MFA (a second factor, §1.1), breach-password screening (Have I Been Pwned's Pwned Passwords API), and progressive retirement in favour of passkeys (§2.7). OWASP guidance: length over composition complexity ("correct horse battery staple"), no forced periodic rotation for most accounts (⚠ this reverses older advice — current guidance is rotate on suspicion, not on a calendar).
- **In distributed systems:** passwords authenticate *to the identity provider*, never to individual services — services see the resulting token, not the password. A password that travels service-to-service is a design failure.

### 2.2 SAML — Security Assertion Markup Language (2005)

**SAML 2.0** ✅ — OASIS Standard, **March 2005**. The XML-based framework for exchanging *authentication and attribute assertions* between an **Identity Provider (IdP)** and a **Service Provider (SP)**.

- **The artifact:** a signed XML **assertion** — `AuthenticationStatement` (how/when the user authenticated), `AttributeStatement` (attributes: email, groups, entitlements), `AuthorizationDecisionStatement` (rarely used — SAML's authz part never caught on).
- **The flows:** **SP-initiated** (user hits the SP, is redirected to the IdP, returns with the assertion via HTTP POST or Redirect binding) and **IdP-initiated** (user starts at the IdP portal, is pushed into the SP with an unsolicited response). Metadata (entity IDs, signing certificates, endpoints) is exchanged out of band between IdP and SP.
- **The verdict:** enterprise-grade SSO for *browser* federation — the dominant pattern in large-enterprise and government identity federation (ADFS, Okta, Ping, Azure AD/Entra ID all speak it). XML is heavy, the bindings are clunky for APIs, and it does nothing for machine-to-machine auth. New API-era integrations default to OIDC (§2.4); SAML survives where the federation contract is already signed, and where regulators/industry schemes mandate it.

### 2.3 OAuth 2.0 — The Authorization Framework (2012)

**OAuth 2.0** ✅ — **RFC 6749, October 2012**, "The OAuth 2.0 Authorization Framework". Despite being *called* "authorization," OAuth 2.0 is really the protocol for **delegated access**: it lets a client obtain limited, scoped access to a resource *on behalf of* a resource owner (usually a user), without the client ever seeing the user's password. The name is the single most confusing fact in this guide — OAuth is the *conveyor belt* for authorization, and OIDC (§2.4) is the layer that makes it carry identity.

**The four roles:** resource owner (the user), client (the application), authorization server (AS — the issuer of tokens), resource server (RS — the API that enforces them).

**The flows** (grant types), and when each is used:

| Flow | RFC | Use case |
|---|---|---|
| **Authorization Code** | §4.1 of RFC 6749 | The workhorse: user-facing web/native apps. Client redirects the user to the AS, user consents, AS returns a code to the client, client exchanges code for tokens. |
| **Authorization Code + PKCE** | RFC 7636 (2015) | The same flow hardened for public clients (SPAs, mobile): a code verifier/challenge binds the code to the client, killing the code-interception attack. **The modern default for all user-facing apps.** |
| **Client Credentials** | §4.4 | Server-to-server: the client *is* the resource owner (a service acting for itself). The S2S workhorse — pair with mTLS (§2.6) or client assertion. |
| **Implicit** | §4.2 | Historically for SPAs (token returned directly from the authorize endpoint). **Deprecated** — RFC 8252-era guidance; tokens in the URL are a leak. Use auth-code + PKCE instead. |
| **Resource Owner Password Credentials (ROPC)** | §4.3 | Client collects the user's password directly. **Discouraged** — it reintroduces password-sharing the framework exists to avoid. |

**The token pair:** **access token** (short-lived, what the client presents to the API) and **refresh token** (long-lived, what the client uses to mint new access tokens at the AS — only issued to confidential clients in most deployments). **Scopes** are the authorisation vocabulary: `accounts:read`, `payments:initiate` — the API checks the scope, not a role (scope-as-authz is a coarse but standard first layer; §6 models go finer).

### 2.4 OIDC — OpenID Connect, the Identity Layer (2014)

**OpenID Connect (OIDC)** ✅ — OpenID Foundation **final specification, February 2014**; an identity layer *on top of* OAuth 2.0. OAuth gives you "the app may access these scopes"; OIDC adds "and here is *who* the user is, verifiably."

- **The ID Token:** a signed **JWT** (per OIDC Core, it must be a JWT) carrying standard identity claims: `sub` (subject — the stable user ID), `iss`, `aud`, `exp`, `iat`, plus profile claims (`name`, `email`, `email_verified`) and a `nonce` binding it to the login request (replay protection).
- **The UserInfo endpoint:** the client can additionally fetch fresh profile claims with the access token.
- **The discovery machinery:** `/.well-known/openid-configuration` publishes issuer, endpoints, and **JWKS URI** (→ §5.1) so any client/service can bootstrap verification without out-of-band config.
- **OIDC flows mirror OAuth grants:** Authorization Code Flow (with PKCE for public clients), Implicit (discouraged), and **Hybrid** (both code and token in one round).
- **The verdict:** OIDC is the default identity protocol for the API era — every major IdP (Azure AD/Entra ID, Okta, Auth0, Keycloak, Ping, AWS Cognito) speaks it natively. "Login with Google/GitHub/Apple" is OIDC (or its SAML sibling) under the hood.

### 2.5 JWT — The Token Format (2015)

**JSON Web Token (JWT)** ✅ — **RFC 7519, May 2015**. A compact, URL-safe, self-contained format for conveying claims between parties, signed (JWS) or encrypted (JWE).

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImtleS0yMDI0IiwidHlwIjoiSldUIn0.
eyJzdWIiOiJ1c2VyLTQ4MjEiLCJpc3MiOiJodHRwczovL2lkLm1lcmxpb24uYmFuayIs
ImF1ZCI6ImFjY291bnRzLWFwaSIsInNjb3BlIjoiYWNjb3VudHM6cmVhZCIsImV4cCI6
MTczNDQxMjAwMCwiaWF0IjoxNzM0NDA4NDAwfQ.
dGhpcy1pcy10aGUtc2lnbmF0dXJlLXBsYWNlaG9sZGVyLXRyaW1tZWQ
```

Three base64url segments — **header.payload.signature** — carrying the classic claims: `iss` (issuer), `sub` (subject), `aud` (audience), `exp`/`nbf`/`iat` (times), `jti` (unique ID — replay tracking), plus application claims (`scope`, `roles`, `branch`, `entitlements`). The signature (RS256/ES256/EdDSA, or HMAC) is what makes it verifiable without any server-side state — that property is the entire reason tokens exist in distributed systems (§3.2, §5). Governance rules from api_governance_guide.md §8.1: short expiry, strict `aud`/`iss` validation, **algorithm pinning** (never accept `alg: none` or symmetric algorithms for asymmetric-key tokens — the classic JWT confusion attacks), and no secrets in claims.

### 2.6 mTLS — Mutual TLS (certificate authentication)

**Mutual TLS (mTLS)** ✅ — TLS with *both* directions authenticated: the server presents its certificate *and* the client presents its own X.509 certificate, so the server cryptographically identifies the caller at the transport layer. The OAuth-flavoured standard is **RFC 8705 (2020)** — "OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens" — which lets clients authenticate to the AS with their TLS certificate and binds access tokens to the client certificate (a stolen token is useless without the cert — token *confusion* and *replay* resistance).

- **Strengths:** strong machine identity (crypto, not secrets), no bearer-token-in-header fragility, perfect for **service-to-service** (§8) and high-risk partner integrations (interbank, vendor systems).
- **Costs:** full PKI lifecycle (CA, issuance, renewal, revocation, distribution) — this is where Vault/OpenBao-style secret management earns its keep (cross-ref openbao_vs_vault_guide.md); cert rotation is an operational discipline, and you can't "revoke" a distributed cert instantly.
- **Where it sits:** as the transport identity under service meshes (Istio/Linkerd, §8.3), as client auth for confidential OAuth clients, and as the API-level authn for partner gateways.

### 2.7 WebAuthn — Phishing-Resistant Authentication (2019, passkeys)

**WebAuthn** ✅ — **W3C Recommendation, 4 March 2019** (Level 1; Level 2: 8 April 2021); core component of the **FIDO2** framework alongside CTAP (the authenticator transport protocol). It replaces "share a secret with the server" with **public-key cryptography per credential**: the authenticator (phone, security key, platform TPM) generates a key pair bound to the origin; the server keeps only the public key; authentication is a challenge signed by the private key.

- **Why it wins:** **phishing-resistant** — the credential is scoped to the exact origin, so a fake `merlion-bank.evil.com` page cannot harvest a usable credential (the one attack passwords and OTPs both lose to). No shared secrets to leak server-side; no credential stuffing; no replay.
- **Passkeys** ✅ — the FIDO Alliance's productisation (announced ~2022): discoverable WebAuthn credentials synchronised across the user's devices by platform vendors (iCloud Keychain, Google Password Manager), enabling passwordless sign-in and cross-device use. In a distributed system, passkeys authenticate the user *to the IdP* — the IdP then issues the session/tokens (§3).
- **The cost:** recovery flows when a device is lost, enterprise attestation/CA management for hardware keys, and user experience change management. But this is the direction of travel — every major platform vendor now ships passkey support, and banking regulators are watching (SCA's possession+inherence factors map naturally onto it, §10.2).

### 2.8 The Mechanisms Table

| Mechanism | Use case | Strengths | Weaknesses |
|---|---|---|---|
| **Passwords** | Universal baseline; user login | Ubiquitous, zero hardware, familiar | Phishable, stuffing/brute-force, server-side hash risk; must be salted + slow-hashed + MFA-augmented |
| **SAML 2.0** (2005) | Enterprise/legacy SSO federation, browser-based | Mature, ubiquitous IdP support, attribute assertions, regulated-sector trust | XML-heavy, browser-bound (no API/machine flows), complex metadata choreography |
| **OAuth 2.0** (RFC 6749, 2012) | Delegated *access* — scoped API authorization | Industry-standard grants (auth-code+PKCE, client-credentials), scopes, refresh tokens | Not identity — no user info by itself; flows are easy to misconfigure; implicit/ROPC deprecated |
| **OIDC** (2014) | Identity layer: *who* the user is + SSO | Verifiable ID token (JWT), discovery, UserInfo, built on OAuth's battle-tested grants | Requires OAuth 2.0 underneath; token size; IdP dependency for session state |
| **JWT** (RFC 7519, 2015) | Self-contained stateless tokens | Offline verification, no session store, portable claims, standard tooling | No revocation by default; must validate alg/aud/iss/exp rigorously; can't un-issue a leaked token |
| **mTLS** (RFC 8705, 2020) | Machine/service and high-risk partner auth | Cryptographic identity, no bearer secrets, certificate-bound tokens | PKI lifecycle burden, rotation/revocation lag, not user-facing |
| **WebAuthn** (W3C, 2019) | Passwordless user auth, passkeys | Phishing-resistant, no shared secrets, strong factors built-in (possession + inherence) | Recovery flows, device loss, platform/ecosystem dependence, attestation management |

---

## 3. Session vs Token

### 3.1 The Contrast: Stateful Sessions vs Stateless Tokens

The two great families of "how does the system remember that this caller is authenticated?" — and the choice between them is the first architectural fork of any identity design.

**Stateful sessions.** On login, the server creates a session record (user, expiry, metadata) in a server-side store and hands the client an opaque **session ID** — typically an HTTP cookie. Every subsequent request presents the ID; the server looks the record up, checks expiry, and knows who you are.

- The session ID is a *pointer*, not a claim: it carries zero meaning without the store.
- **Revocation is instant and trivial**: delete the record (logout, admin kill, compromise response) and the session dies.
- **The cost is state**: the store must exist, be fast, and be shared across every node that can receive requests (§4).
- The classic HTTP-cookie incarnation also brings the web's session attacks: fixation, hijacking (mitigated by `Secure`/`HttpOnly`/`SameSite` attributes), CSRF (mitigated with tokens/headers).

**Stateless tokens.** On login, the IdP mints a **JWT** that *is* the state: signed claims (subject, expiry, scopes) that any service can verify with the issuer's public key — no lookup, no shared store (§5).

- The token is a *claim*, not a pointer: verify the signature and the claims, and you know who the caller is and what they're scoped for.
- **Verification is distributed by construction**: every service, every gateway node, verifies offline against the same JWKS.
- **The cost is revocation**: a validly-signed token keeps being valid until `exp` — you cannot recall it. Mitigations: short expiry (minutes), refresh-token rotation, a revocation/denylist or introspection endpoint (RFC 7662) for high-stakes resources, and audience/scope discipline.

### 3.2 The Comparison (verified frame)

| Question | Session answer | Token answer |
|---|---|---|
| Where is the state? | Server-side store (pointer semantics) | Inside the token (claim semantics) |
| What does the client present? | Opaque session ID (cookie/header) | Signed JWT (bearer) |
| How does a service verify? | Lookup in the shared session store | Verify signature + claims against JWKS |
| Cost per request | One store round trip (or cache hit) | One signature verification (CPU, no I/O) |
| Revocation | Instant — delete the record | Hard — wait for `exp`, use denylist/introspection |
| Logout | Trivial server-side | Client-side discard; token remains valid until expiry (short `exp` is the mitigation) |
| Horizontal scale | Needs a shared, HA store (Redis — §4) | Naturally stateless; any node verifies |
| Server-side session data | Unlimited (cart, draft, MFA-in-progress) | Only what fits in claims (size limits, no secrets) |
| Where used | Traditional web apps, MFA staging, high-control contexts | APIs, microservices, S2S, mobile/SPA bearer auth |

**The hybrid reality:** production systems almost always use both — a *session* at the IdP/browser layer (login state, MFA step-up, SSO) and *tokens* at the API layer (what the services verify). The session is the stateful front door; the token is the stateless passport the distributed back end trusts. The banking worked example (§11) is exactly this hybrid.

---
## 4. The Distributed Sessions

### 4.1 The Session Stores (Redis)

When sessions must be shared across many nodes, the session state needs a home every node can reach. The standard answer is a **fast shared key-value store — Redis** ✅ (the same Redis the distributed rate limiter uses for its shared counters, cross-ref distributed_rate_limiter_guide.md §8). A session is a key (`sess:<id>`) with a JSON/msgpack value (user ID, expiry, attributes) and a **TTL** that makes sessions self-cleaning — the `EXPIRE` discipline that keeps the store from growing without bound.

Why Redis is the session-store default:

- **Sub-millisecond reads** — session lookup is on the hot path of every authenticated request.
- **Atomic operations** — `SET NX`, `INCR`, Lua scripts for check-and-act (session rotation, sliding expiry) in one round trip.
- **Native TTL** — expiry without a sweeper job; Redis deletes expired keys lazily/actively.
- **HA story** — Sentinel for failover, Cluster for horizontal scale and sharding (hash-tag `{sess:...}` keys for multi-key affinity, exactly as in the rate-limiter design).
- **Ops reality** — persistence must be tuned (RDB snapshots + AOF) because *session loss is a mass logout*; and the store is a single point of trust, so network isolation and auth (`requirepass`/ACLs) are non-negotiable.

Alternatives exist (Memcached — fast but no native TTL semantics and no persistence; a relational table — durable but slow on the hot path; Hazelcast/Infinispan — in-process distributed caches with a JVM home). Redis is the pragmatic default because it is *also* the rate-limiter store, the cache, and the queue — one operational surface.

### 4.2 The Session Design

Two canonical architectures for making sessions work across many nodes:

**Sticky sessions (session affinity).** The load balancer pins a client to one backend node (by cookie or IP hash) so its session always lives in that node's local memory.

- *Trade-off:* zero shared-state complexity — but the pin *is* the coupling: a node failure orphans every session on it (unless sessions replicate), node drains/rollouts log users out, and the balancer's affinity logic becomes part of your availability story. Load imbalance is structurally guaranteed (hot nodes accumulate). Acceptable for small fleets and low-availability-budget systems; a trap at scale.

**Shared session store (centralized).** All nodes read/write one store (Redis) — no affinity, any node serves any request, nodes are stateless and disposable.

- *Trade-off:* one more critical dependency on the request path (mitigated by Redis HA + local caching of hot sessions with bounded staleness), and a per-request round trip (sub-ms on a co-located LAN, but it is I/O). The resilience win — instant failover, horizontal scale, clean rolling deployments — is why the shared store is the enterprise default, and it composes with the *stateless token* story: use the store for what truly needs server-side state (MFA staging, high-value confirmations), tokens for everything else.

**Client-side sessions.** The extreme of statelessness: the session *is* an encrypted cookie (e.g., Rails encrypted cookies, JWT-as-cookie). No store at all.

- *Trade-off:* offline verification, but revocation is impossible until expiry, and the cookie must be signed *and* encrypted (it holds server-side secrets by construction). Used for lightweight state; not for high-control banking flows.

### 4.3 The Session Table

| Approach | Description | Trade-off |
|---|---|---|
| **Local (in-process)** | Session lives in one node's memory | Zero added latency/deps — but requests must hit the same node; node loss = session loss; no horizontal sharing |
| **Sticky sessions** | LB pins client to a node so local sessions work | No shared state — but affinity couples availability to nodes, drains/rollouts log users out, load imbalance is structural |
| **Shared store (Redis)** | All nodes read/write one session store with TTLs | Any node serves any request, instant revocation, HA via Sentinel/Cluster — but the store is a critical dependency + one hot-path round trip |
| **Client-side (encrypted cookie / JWT cookie)** | Session state lives in a signed+encrypted cookie | Stateless, no store — but no server-side revocation until expiry, payload size limits, must be signed *and* encrypted |

**Session hygiene (verified best practice):** rotate the session ID on privilege change (login, role change — defeats fixation); sliding vs absolute expiry (absolute wins for security, sliding for UX — banking defaults to absolute + short); idle timeout; bind sessions to risk context (device, IP) and force re-auth (step-up) on sensitive actions (payments → SCA, §10.2); and a global kill-switch (revoke all sessions for a user/tenant) — trivial with a shared store, painful with sticky sessions.

---

## 5. Token Validation

### 5.1 JWKS — JSON Web Key Set (RFC 7517)

**JSON Web Key (JWK) Set** ✅ — **RFC 7517 (2015)**. A JWKS is a JSON document — `{"keys": [...]}` — containing the **public keys** (RSA, EC, OKP) an issuer uses to sign tokens. The IdP publishes it at a well-known URL — for OIDC, `https://id.example.com/.well-known/jwks.json`, advertised by the discovery document (§2.4). Each key carries a **`kid`** (key ID) that the token's header references, so verification is: *look at the token's `kid` → find that key in the JWKS → verify the signature with it* — and key rotation becomes a *publish* operation, not a coordination ceremony (the new key appears in the JWKS; tokens signed with the old key verify until the old key is removed from the set, keeping overlap for in-flight tokens).

The JWKS is the answer to the **keys problem** from §1.3 at the token layer: trust is bootstrapped once (the JWKS URL itself, protected by HTTPS + the discovery document), and every verifying service — gateways, APIs, meshes — pulls the same set. Open-source JWT libraries consume JWKS natively (e.g., `jwks-rsa` for Node, PyJWKClient, Nimbus JOSE+JWT); mature gateways (Kong, Apigee, Envoy) ship JWKS-aware JWT validation plugins (cross-ref api_governance_guide.md §7).

### 5.2 The Validation Flow

The canonical **offline JWT validation** sequence — every service and gateway must execute it identically on every request:

```
1. Extract the token (Authorization: Bearer <token>)
2. Parse header.payload.signature; check `alg` against the allowlist   (pin RS256/ES256/EdDSA; reject `none`, reject HS* for asymmetric issuers)
3. Look up `kid` in the cached JWKS; if missing → refresh from the JWKS endpoint (and re-check)
4. Verify the signature with the key
5. Validate `exp` (reject expired) and `nbf`/`iat` (with a small clock-skew allowance, e.g. 30–60 s)
6. Validate `iss` equals the configured issuer
7. Validate `aud` contains this resource's audience (or that the audience is acceptable)
8. Check revocation, if enforced: denylist lookup or RFC 7662 introspection (high-stakes resources)
9. Extract claims (sub, scope, roles/attributes) → hand to the authorization layer (§6, §11)
10. Cache the verification result / key material per key-id; never trust a token without a signature check
```

**The failure modes this flow prevents:** `alg: none` tokens (unsigned — step 2), algorithm-confusion attacks (swapping RS256 for HS256 and signing with the public key — step 2's allowlist), expired tokens (step 5), tokens minted by a lookalike issuer (step 6), tokens minted for a *different* API being replayed here (step 7 — the audience check is the BOLA-of-tokens defence), and stale-key verification after rotation (step 3 + JWKS caching policy).

**Introspection (RFC 7662)** is the stateful counterpoint: the resource server asks the AS "is this token valid?" — enabling instant revocation at the cost of a round trip and AS load. The pragmatic split: *offline JWKS validation for the fast path* (every request), *introspection/denylist for the sensitive path* (payments, admin, high-value transfers). This mirrors the session-vs-token hybrid of §3.2: fast by default, stateful where it counts.

### 5.3 The Validation Table

| Aspect | Design | Example |
|---|---|---|
| **Key distribution** | JWKS endpoint (RFC 7517) + `kid`-based lookup; cache with refresh | `GET /oidc/jwks` returns RSA public keys; gateway caches 1h, refreshes on unknown `kid` |
| **Signature** | Asymmetric, algorithm allowlist (RS256/ES256/EdDSA) | Reject `alg: none`; never accept HS256 for an RSA-signing issuer |
| **Claims** | `exp`/`nbf` (+clock skew), `iss`, `aud` all mandatory | Token for `accounts-api` rejected at `payments-api` (`aud` mismatch) |
| **Lifetime** | Short `exp` (5–15 min access tokens) + refresh rotation | 10-minute access tokens; refresh token rotation with reuse detection |
| **Revocation** | Fast path: offline; sensitive path: denylist/introspection (RFC 7662) | Payments API introspects; read APIs verify offline |
| **Rotation** | Publish new key with new `kid`; overlap period; monitor `kid` usage | Monthly key rotation via Vault/OpenBao signing backend (openbao_vs_vault_guide.md) |
| **Observability** | Log validation failures by cause (expired/issuer/aud/signature) | Dashboard: failed-validation rate per service — the early-warning for token attacks |

---

## 6. The Authorisation Models

Authentication answers *who*; the authorisation models answer *what may they do* — and each model is a different answer to "how do we represent permission?" The three canonical families, oldest to newest:

### 6.1 RBAC — Role-Based Access Control (NIST)

**RBAC** ✅ — formalised by **NIST** (the reference model published as **ANSI INCITS 359-2004**, from the Sandhu/Ferraiolo NIST work). The model: **users are assigned roles; roles are assigned permissions; a user's permissions = the union of their roles' permissions**. `user → role → permission` is the whole ontology — three entities, two assignments.

- **The core concepts:** roles (e.g., `teller`, `payments-approver`, `auditor`), permissions (an operation on a resource: `initiate transfer`, `approve > SGD 50k`), sessions with activated-role constraints, and role hierarchies (senior role inherits junior's permissions).
- **Strengths:** legible, auditable ("why can this user do that?" → "because they hold role X"), cheap to administer at human scale, and the mental model every regulator's access review expects. It is the enterprise default and the baseline model in most IdPs' enterprise plans (Azure AD roles, Okta groups→roles).
- **Weaknesses:** **role explosion** — fine-grained needs multiply roles combinatorially (a role per branch × per product × per approval level); it is *static* — a role cannot express "only between 9am–6pm" or "only if the account balance is positive"; and it is **permission-centred, not relationship-centred** — "this user may see *these specific* accounts" is awkward (you end up with per-account roles, which is role explosion in its purest form).
- **NIST's own framing:** RBAC is the baseline; ABAC is the NIST-documented evolution for the cases RBAC cannot express (NIST SP 800-162 calls ABAC "an advanced method" that "evolves from RBAC").

### 6.2 ABAC — Attribute-Based Access Control (NIST)

**ABAC** ✅ — codified by **NIST SP 800-162** ("Guide to Attribute Based Access Control Definition and Considerations", **2014**). The model: an access decision is a **policy over attributes** of four categories — **subject** (user, role, clearance, org), **resource** (owner, classification, branch), **action** (read/write/transfer), and **environment/context** (time, location, device, risk score). A policy is a boolean rule combining attributes:

```
allow if subject.clearance >= resource.classification
      and subject.org == resource.org
      and action == "read"
      and environment.time within business-hours
      and environment.risk-score < 70
```

- **Strengths:** **dynamic and fine-grained** — the same user gets different decisions for different resources *and different moments* (amount, risk, time all live in the decision); fewer roles needed (policy replaces role taxonomy); scales to huge, heterogeneous populations (the attributes come from authoritative systems — HR, CRM, AML).
- **Weaknesses:** the **attribute dictionary must be governed** (who defines `risk-score`? who vouches for `clearance`?); policy sprawl and rule-conflict resolution need tooling; decisions are only as trustworthy as the attribute sources (attribute poisoning is an ABAC-specific attack); harder to audit than RBAC ("why did this decision happen?" needs the attribute snapshot, not a role name).
- **The pragmatic enterprise pattern:** **RBAC for coarse structure, ABAC for fine-grained rules** — roles remain the headline answer to "what is this user's job?", and ABAC policies refine it ("teller role, *and* branch matches, *and* amount under limit, *and* no AML flag"). This is the hybrid the banking worked example builds on (§11.2).

### 6.3 ReBAC — Relationship-Based Access Control (Zanzibar)

**ReBAC** ✅ — the relationship-centric model, canonicalised by **Google's Zanzibar paper (2019)**: *"Zanzibar: Google's Consistent, Global Authorization System"* (USENIX ATC 2019) — the system behind Google Drive, YouTube, Calendar, and Gmail's sharing models, serving billions of checks per day.

The core idea: **permission is a graph of relationships**, stored as **tuples**: `object#relation@user` — e.g. `doc:budget-2026#viewer@alice`, `folder:finance#member@team:finops`, `team:finops#member@alice`. A check ("can alice view doc:budget-2026?") is a **graph traversal** through the relationship graph, guided by a **namespace configuration** that declares how relations compose (`folder#viewer` includes `folder#member`, and `folder#member` includes members of nested teams...).

- **Why it exists:** the sharing models of modern products are *relationships* ("who has this document been shared with? who manages this folder?") — RBAC's roles and ABAC's attributes both express them badly (role explosion per object, or attribute plumbing for every share). ReBAC makes *"this specific user can do this specific thing to this specific object"* the native shape.
- **The Zanzibar design pillars (per the paper):** a global, replicated, strongly-consistent relationship store; tuple-based ACLs; namespace-config-driven expansion; and **consistency** — decisions respect the order of ACL changes (the paper's headline guarantee).
- **Open-source implementations:** **OpenFGA** (the open-source Zanzibar-style system), **SpiceDB** (AuthZed's Zanzibar-inspired database), **Ory Keto** — all implementing tuple stores + check/expand APIs.
- **Costs:** a new storage system to operate, tuple-fanout management (large groups, deep hierarchies), and the model requires *relationship data* to exist and be maintained — it does not replace RBAC/ABAC so much as add the *object-granular* dimension.

### 6.4 The Model Comparison

| Question | RBAC | ABAC | ReBAC |
|---|---|---|---|
| What is a permission? | A role's assigned permission | A policy over attributes | A relationship tuple in a graph |
| Granularity of decision | Coarse-to-medium (role-level) | Fine (attribute-level, per request) | **Per-object** (relationship-level) |
| Dynamic context (time, risk, amount)? | No — static by design | Yes — first-class | Indirect (via context attributes) |
| "Why this decision?" | Easy: role lookup | Needs attribute snapshot | Easy: tuple walk ("shared with X via folder Y") |
| The failure mode | Role explosion; static | Attribute governance; policy sprawl | Relationship data management; graph scale |
| Standards | NIST INCITS 359-2004 | NIST SP 800-162 (2014) | Zanzibar paper (Google, 2019); OpenFGA/SpiceDB ecosystem |
| Typical home | HR/enterprise systems, admin consoles | Payments, AML, fine-grained APIs | Collaboration/sharing products, multi-tenant SaaS |

### 6.5 The Models Table

| Model | Concept | Use case | Complexity |
|---|---|---|---|
| **RBAC** | Users → roles → permissions | Enterprise admin, coarse access tiers, regulatory access reviews | Low — the baseline every team should start from |
| **ABAC** | Boolean policies over subject/resource/action/environment attributes | Dynamic rules (amounts, risk, time, geography), fine-grained API control | Medium — attribute dictionary + policy governance needed |
| **ReBAC** | Relationship tuples (`object#relation@user`) + graph checks | Per-object sharing, multi-tenancy isolation, collaboration permissions | High — new store + model + data maintenance |

**Selection rule of thumb (practitioner consensus, flagged ⚠):** start with RBAC; add ABAC where decisions must be *dynamic*; add ReBAC where permissions are *per-object relationships*. The three compose — Merlion Bank's design (§11) uses all three deliberately: RBAC for staff roles, ABAC for amount/risk rules, ReBAC thinking for TPP account-permission scoping.

---

## 7. The Policy Engines

### 7.1 OPA — Open Policy Agent (CNCF)

**OPA** ✅ — the **Open Policy Agent**, the CNCF-graduated, general-purpose **policy engine** (created by Styra, first released **2016**; CNCF **graduated project**; OPA 1.0 shipped December 2024). Its one-sentence pitch: *decouple policy from application code* — instead of `if (user.role == "teller" && amount < 50000)` scattered through service code, policies live in **Rego**, OPA's declarative policy language, and services ask OPA for a **decision** at runtime.

- **The model:** the service sends a **query** (the input: subject, action, resource, context — the §1.2 shape) to OPA's decision API; OPA evaluates the loaded **policies** (Rego) against the input and its **data** (cached policy-relevant facts); the answer is an allow/deny (or a richer JSON — *which* records the caller may see, dynamic rules, etc.).
- **Deployment shapes:** as a **library** (Go), as a **daemon/sidecar** (co-located with the service — the low-latency pattern), or as a **central decision service**; integrated into gateways and meshes — **Envoy external authorization**, **Kubernetes Gatekeeper** (admission control), Kong/NGINX plugins. OPA in the wild: Kubernetes policy, API gateway authz, Terraform/CI policy (its original use), and — the point for this guide — **API authorisation decisions** in microservices.
- **Why it matters for identity architecture:** OPA is the standard home for the **authorisation policy** side of the house — the ABAC rules of §6.2 expressed as code, versioned in Git, reviewed like code, deployed like code, audited like code. It is the *PDP* that turns "policies as code" from a slogan into an artifact. (⚠ Practitioner consensus that OPA is the de-facto standard *policy engine*; it is not a full *authorisation service* — it does not store relationships or manage identity; pair it with an IdP for authn and a relationship store for ReBAC.)

### 7.2 The Policy Design: Policies as Code

The discipline that makes a policy engine trustworthy (verified pattern):

1. **Policies are code**: Rego files in Git, reviewed via the normal PR pipeline, tested with unit-style policy tests (`opa test`), versioned, and promoted through environments. A policy change is a code change with a paper trail — the security-review gate from api_governance_guide.md §5 applies to it.
2. **Decisions are data**: the engine returns a structured decision (allow/deny + reason + matched rule) that is logged — making "why was this allowed?" answerable (the audit requirement of §6.1's RBAC legibility, now for ABAC rules too).
3. **Fail closed**: on engine outage or evaluation error, deny (for security-critical paths) — the fail-open/fail-closed decision is a *policy* decision itself, made deliberately (cross-ref distributed_rate_limiter_guide.md §9.3 for the same debate on the rate-limiter side).
4. **Bundles and distribution**: policies are distributed to engines as signed **bundles** (OPA's bundle API) — the engine is a dumb evaluator, the control plane (Styra DAS, enterprise tooling, or plain CI + bundle server) owns the policy lifecycle.
5. **Performance budget**: a decision must be sub-millisecond on the hot path — co-locate the engine (sidecar), keep data in memory, and treat policy evaluation like any other latency line item.

A minimal Rego flavour (illustrative):

```rego
package bank.payments

# allow a teller to initiate a domestic transfer under 50k SGD
# unless the customer is flagged by AML
default allow := false

allow if {
    input.subject.role == "teller"
    input.action == "initiate_transfer"
    input.resource.type == "domestic"
    input.resource.amount < 50000
    not input.resource.customer.aml_flag
}
```

### 7.3 The Engine Table

| Engine | Description | Use |
|---|---|---|
| **OPA (Open Policy Agent)** | CNCF-graduated general-purpose policy engine; Rego language; decision API; bundles; Envoy/Gatekeeper integrations | The default PDP for API/microservices/Infra policy-as-code (Kubernetes admission, API authz, CI policy) |
| **OPA as sidecar vs central** | Sidecar = co-located engine, sub-ms decisions; central = shared decision service, simpler control | Sidecar for hot paths; central for cross-cutting policy + audit |
| **Gatekeeper (OPA for K8s)** | OPA as Kubernetes admission controller | Cluster policy: what may be deployed |
| **Envoy ext_authz** | Envoy's external authorization filter (OPA-Envoy plugin) | Per-request authz at the mesh/gateway data plane |
| **XACML (context)** | The older XML policy standard (2003-2013 era) | Legacy enterprise PDPs; the design ancestor of modern engines (⚠ legacy, largely superseded by OPA-style engines) |

---

## 8. Service-to-Service

Authentication so far has been about *people* (and the apps acting for them). But in a microservices architecture, the biggest authn population is **machines**: every service calling every other service. "Who is this request from?" between services is **service-to-service (S2S) identity** — and the era of shared-secret service accounts (a static API key in every config) is over. The modern answer is SPIFFE/SPIRE + mTLS.

### 8.1 SPIFFE (CNCF)

**SPIFFE** ✅ — **Secure Production Identity Framework For Everyone**, a **CNCF-graduated** open standard for **workload identity**. Its deliverables:

- **SPIFFE ID** — the standard identity *format* for workloads: `spiffe://trust-domain/path`, e.g. `spiffe://merlion.bank/ns/payments/sa/payments-service`. A workload's identity is a URL, not a secret.
- **SVID** — SPIFFE Verifiable Identity Document: the *credential* proving the SPIFFE ID — an **X.509-SVID** (a short-lived certificate with the SPIFFE ID in the SAN) or a **JWT-SVID** (a JWT with the SPIFFE ID as `sub`). Short-lived by design — hours, not months.
- **The point:** one portable, cryptographically verifiable identity scheme for every workload, on every platform — no per-platform service-account silos, no shared static secrets.

### 8.2 SPIRE (SPIRE)

**SPIRE** ✅ — the **SPIFFE Runtime Environment**, the CNCF-graduated reference *implementation* of SPIFFE: the **workload identity issuance system** that actually hands SVIDs to workloads. The magic is **attestation** — proving *what* a workload is from the platform itself, with **no bootstrap secret**:

- **Node attestation:** a server registers a node (a Kubernetes node, a VM) by verifying platform evidence (e.g., the node's join token or cloud instance identity).
- **Workload attestation:** when a workload (a pod, a container) asks for an SVID, SPIRE inspects the platform (kubelet metadata: namespace, service account, labels) and issues the SVID **only if the workload's platform identity matches its registered entry** — the workload *is* what its platform says it is.
- **Rotation and renewal:** SVIDs are short-lived and auto-renewed (hours), so compromise windows are small and there is no long-lived secret to steal — the operational payoff versus static service accounts.
- **Federation:** SPIFFE IDs federate across trust domains — `spiffe://merlion.bank` trusting `spiffe://swift-provider.net` — which is how cross-organization S2S trust gets built without shared secrets.

### 8.3 The mTLS Mesh

**The service mesh mTLS** ✅ — the enforcement story: in a service mesh (**Istio**, **Linkerd**, Consul), a **sidecar proxy** sits beside every workload, and all service-to-service traffic flows through the sidecars. The mesh provisions **mTLS between every pair of workloads** — certificates issued by the mesh CA (Istio's citadel/istiod, Linkerd's identity component), with **SPIFFE IDs as the certificate identities** (Istio issues SPIFFE-compatible certs; SPIFFE/SPIRE integrates as the CA). The result:

```
pod A ──(mTLS, cert: spiffe://merlion.bank/ns/payments/sa/payments-api)──▶ pod B
   │                                                                        │
  sidecar ── SPIFFE ID verified at handshake ──▶ sidecar                    │
   ▲                                                                        ▼
 SPIRE attests pod A's platform identity                        SPIRE attests pod B
```

- **What the mesh buys:** transport-layer identity and encryption *by default* — a workload's identity is verified in the TLS handshake, not by an application-level token the developer had to remember to check; certs rotate automatically; no code changes to "do authn" between services.
- **What it does not buy:** it is **authn, not authz** — the mesh proves *who* the caller workload is (and can enforce simple allow-lists of SPIFFE IDs), but *what the caller may do* (which endpoints, which data) is still the application's authorisation job (§6, §7). The classic error is treating mesh mTLS as if it were application authorization — it is the transport-level ID card, not the door policy.
- **East-west vs north-south:** the mesh secures **east-west** (service-to-service) traffic; the **API gateway** secures **north-south** (external clients → services) — the two layers compose (cross-ref api_governance_guide.md §7.4 "Gateway vs. Service Mesh"). The gateway terminates OIDC bearer tokens; the mesh terminates mTLS between workloads; both feed the same authorisation layer downstream.

### 8.4 The S2S Table

| Mechanism | Description | Use case |
|---|---|---|
| **SPIFFE ID** | Standard workload identity format: `spiffe://trust-domain/path` | A common identity *vocabulary* across platforms/clouds |
| **SVID (X.509 / JWT)** | The short-lived credential proving a SPIFFE ID | The thing workloads present; hours-long lifetime, auto-renewed |
| **SPIRE** | Attestation-based issuance: proves workload identity from platform evidence, no bootstrap secret | Automating SVID issuance for pods/VMs/containers |
| **Mesh mTLS (Istio/Linkerd)** | Sidecar-enforced mutual TLS between workloads, SPIFFE IDs in certs | Transport-level S2S authn + encryption, transparent to app code |
| **OAuth client credentials + mTLS (RFC 8705)** | Token-based S2S with certificate-bound access tokens | Token-using S2S where services call APIs *through* an AS (TPP integrations, partner APIs) |
| **Static API keys / shared secrets** | The legacy S2S identity | The baseline to migrate *away from* (no rotation, no attestation, leak-prone) |
## 9. SSO and the Enterprise Patterns

### 9.1 SSO — Single Sign-On

**Single sign-on (SSO)** ✅ — the enterprise identity pattern: the user authenticates **once**, and every application in the enterprise (or federation) accepts that login without asking again. The mechanics: all applications delegate authentication to a shared **identity provider**; the IdP holds the master session; each application receives a token/assertion proving the user authenticated there (SAML assertion, §2.2, or OIDC ID token, §2.4) and starts its own local session on that basis.

- **The user-visible contract:** one login for the morning; every app (email, HR, banking portal, vendor portal) says "you're already signed in."
- **The security contract:** authentication happens in *one* hardened place (the IdP's login page — MFA, passkeys, risk checks all live there once), and **every downstream app trusts the IdP's signed assertion** instead of re-implementing credential checks. One authentication, one audit trail, one place to enforce policy.
- **The protocols:** SAML 2.0 for legacy/browser federation, OIDC for modern apps, with the IdP as the common spoke. The enterprise reality is usually *both*: OIDC to the modern apps, SAML to the legacy portfolio, one IdP underneath (this is exactly what Okta/Azure AD/Entra ID/Keycloak do).
- **SSO vs. the distributed problem:** SSO is the *enterprise* answer to the multi-service problem of §1.3 — but note SSO is about **authn**; each application still performs its own **authz** (roles, permissions) against the identity the IdP vouches for. "SSO'd in" ≠ "allowed to do anything."

### 9.2 The IdP — Identity Provider

The **identity provider (IdP)** ✅ is the system of record for identity in the enterprise: the component that *authenticates* users, issues tokens/assertions, holds the user directory (or federates to it — Active Directory, HR systems), and manages the credential lifecycle (MFA enrollment, passkeys, password resets, deprovisioning).

**The IdP's job description:**

- **Authentication service** — login, MFA, passwordless (WebAuthn/passkeys, §2.7), step-up authn for sensitive actions.
- **Token/assertion issuance** — OIDC ID tokens + access tokens, SAML assertions, and the **JWKS** that makes them verifiable everywhere (§5.1).
- **Directory and lifecycle** — users, groups, attributes (the raw material of RBAC/ABAC, §6), plus the *scary* half: **deprovisioning** (when someone leaves, the IdP is where access dies — an identity architecture without a reliable offboarding path is an audit finding waiting to happen).
- **Federation hub** — trusting external IdPs (partner SSO, social login) and being trusted by downstream apps; the trust boundary of the whole enterprise identity graph.

**The vendor landscape (verified as categories):** cloud IdPs (Okta, Azure AD/Entra ID, Auth0, Ping Identity), self-hosted/open-source (Keycloak — the de-facto open-source IdP, Zitadel), legacy on-prem (ADFS, Oracle Access Manager, CA/ Broadcom). Banking reality: usually a hardened commercial IdP (or several — one for staff, one for customers) behind a lot of compliance paperwork.

**The IdP in the architecture:** one (or two: internal + customer-facing) IdPs, fronted by the gateway (§9.3), feeding every application's authn — and the *authorisation* layer (roles/policies, §6–§7) consuming the identity claims the IdP issues. IdP outage = nobody logs in anywhere, so IdP availability and failover are first-class architecture concerns.

### 9.3 The Gateway Auth

The **API gateway** is the enforcement chokepoint of the API platform: it sits in front of every API and performs the *mechanical* security controls — authentication of the presented credential, token validation, rate limiting, quotas — before the request reaches any service (cross-ref api_governance_guide.md §7 "API Gateway and Runtime Governance" and §8.1's governance table; the rate-limiter side is distributed_rate_limiter_guide.md).

**The gateway's identity duties:**

- **Terminate authn at the edge:** validate OAuth 2.0 bearer tokens / OIDC access tokens against the IdP's **JWKS** (§5) — signature, `exp`, `iss`, `aud`, scope — and reject before the backend sees anything. The gateway is the *PEP* for token validation.
- **Translate identity for the back end:** inject verified identity into the downstream call (a trusted header or a forwarded/transformed JWT with the verified claims — never blindly trusting client-supplied identity headers, the classic gateway bug).
- **Enforce coarse authz at the edge:** scopes, API-key-to-consumer mapping, per-endpoint access rules — the *coarse* authorisation layer (fine-grained authz stays in the services/OPA, §7).
- **Compose with the other edge controls:** rate limiting/quotas (distributed_rate_limiter_guide.md), IP/geo allow-lists, WAF — one request, one pass through the full control stack.

**The gateway as the SSO front door:** for APIs, the gateway is where the OIDC authorization-code flow terminates for browser/mobile clients; the user logs in at the IdP, the gateway receives the tokens, and downstream services receive verified identity — "authenticate once (at the IdP), authorize everywhere (services + OPA)" — the thesis of this guide (§12).

### 9.4 The Patterns Table

| Pattern | Description | Example |
|---|---|---|
| **SSO (OIDC)** | One IdP login; apps get ID tokens, verify via JWKS | Staff portal: login at IdP → HR, CRM, intranet all accept the OIDC session |
| **SSO (SAML)** | Legacy browser federation via XML assertions | Government/enterprise federation where SAML is the contracted protocol |
| **IdP-centric** | IdP is the single system of record for authn + lifecycle | Okta/Azure AD/Entra ID as the hub for all staff applications |
| **Gateway edge authn** | Gateway validates tokens at the edge; backends trust injected identity | Kong/Apigee JWT plugin validating against IdP JWKS before routing |
| **Edge coarse + service fine authz** | Gateway enforces scopes; services/OPA enforce per-resource rules | Payments API: gateway checks `payments:initiate` scope; OPA checks amount/risk/AML (Merlion Bank, §11) |
| **Federated B2B** | External IdP trust: partner's IdP vouches for partner staff | TPP access to open-banking APIs under PSD2 (cross-ref banking guides) |

---

## 10. The Banking Context

### 10.1 PSD2

**PSD2** ✅ — the EU's **Revised Payment Services Directive, Directive (EU) 2015/2366** — in force **13 January 2018** — the regulatory driver that forced European banks to open account data and payment initiation to licensed **Third-Party Providers (TPPs)** through APIs, under strict security and access rules. (Singapore's MAS takes a different, non-mandatory path — guidance + the Finance-as-a-Service API Playbook (2016) and Technology Risk Management guidelines — cross-ref api_governance_guide.md §10.1, which details both regimes; the banking-series guides under `../banking/` carry the institution-level view.)

PSD2's identity-relevant payload:

- **TPP roles:** **AISP** (Account Information Service Providers — read account data), **PISP** (Payment Initiation Service Providers — initiate payments), **CBPII** (Card-Based Payment Instrument Issuers — check funds). Each is a *licensed third-party principal* with its own identity — the *user* is a separate principal whose consent is scoped.
- **The consent model:** the user authorises a TPP to act *on their behalf* — the PSD2-flavoured OAuth 2.0 pattern: authorization code + explicit consent + **scopes tied to the specific accounts and purposes**, with consent *revocable* by the user — the ReBAC-style per-object scoping of §6.3 applied to bank accounts.
- **Secure communication:** TPP ↔ bank APIs must be authenticated *both ways* — the bank's API authenticates to the TPP and the TPP authenticates to the bank (mTLS or strong client auth — the S2S story of §8 in regulatory clothing; the EBA RTS mandates mutual authentication and, in practice, PSD2 APIs use mTLS + OAuth).

### 10.2 SCA — Strong Customer Authentication

**Strong Customer Authentication (SCA)** ✅ — the PSD2 security core, specified in the **RTS on SCA and secure communication (Commission Delegated Regulation (EU) 2018/389)**, applicable from **14 September 2019**. The rule: for accessing account data online and initiating electronic payments, the customer must be authenticated with **two or more independent factors from different categories** — knowledge (PIN/password), possession (phone/token/app), inherence (fingerprint/face) — per the §1.1 factor model. Two further SCA requirements:

- **Dynamic linking:** for remote payment transactions, the authentication must be *dynamically linked* to the specific transaction — amount and payee are bound into the authentication (the user sees "pay €47.20 to Merlion Ltd" *inside* the authentication step). This is SCA's answer to transaction fraud: the credential cannot be harvested for one transaction and replayed for another.
- **Independence of elements:** the RTS requires the factors to be *independent* — a compromised element in one category must not compromise another (EBA Q&A 2020_5619 addresses this directly; e.g., SMS-OTP-on-the-same-device as the app is a known weak spot).

**SCA exemptions (the RTS's escape hatches, risk-based):** low-value transactions (under €30, with counters), whitelisted beneficiaries, recurring/fixed-amount payments, secure corporate payment processes, and transaction-risk analysis (TRA) — the bank's own risk engine may waive SCA when the risk score is low. The bank's **authorisation layer** (§6, §7) is where these exemption rules live: ABAC-style policies over amount, risk, and counterparty — the worked example's OPA policies are precisely this machinery.

**SCA in identity-architecture terms:** SCA is *step-up authentication with transaction binding* — the "authenticate once, authorize everywhere" pattern with a regulated exception: for the highest-value actions, re-authenticate (step-up) *with the transaction's details bound into the proof*. The flow composes with everything in this guide: base session from the IdP, step-up via WebAuthn/OTP when the OPA policy says the action is high-risk, and the SCA proof recorded in the audit trail.

### 10.3 The Bank Table

| Requirement | Implementation | Notes |
|---|---|---|
| **Licensed TPP access (AISP/PISP/CBPII)** | OAuth 2.0 authorization code + consent + account-scoped scopes; mTLS between TPP and bank APIs | Per-object scoping = ReBAC-style tuples over accounts (the consent IS the relationship) |
| **Strong Customer Authentication** | Two independent factors (knowledge + possession/inherence) via the IdP's MFA/WebAuthn; dynamic linking for payments | RTS (EU) 2018/389; applicable since 14 Sep 2019 |
| **Step-up for high-risk actions** | OPA/ABAC policy flags the action (amount, risk, AML) → IdP step-up challenge | SCA exemptions (low value, TRA) are the same policy machinery, risk-based |
| **Mutual authentication (secure communication)** | mTLS certificates for TPP↔bank and bank↔bank channels | EBA RTS; cert lifecycle via PKI/Vault (openbao_vs_vault_guide.md) |
| **Consent revocation** | User-facing consent dashboard; relationship tuples deleted → access dies | Revocation *must* be effective, not cosmetic — the stateful-vs-token lesson of §3 applied |
| **Audit and evidence** | Every authn event, SCA proof, and authz decision logged with correlation IDs | Governance artifacts are regulatory artifacts (api_governance_guide.md §10.2) |
| **MAS (Singapore) context** | Enabling guidance, not PSD2: API Playbook (2016), TRM guidelines; SGFinDex consent-based sharing | Different regime, same architecture: IdP + consent + scoped tokens (cross-ref api_governance_guide.md §10.1) |

---

## 11. Worked Example: A Microservices Auth Design

### 11.1 The Scenario

*Merlion Bank* — the fictional Singapore bank from api_governance_guide.md §10.3 (consistent cross-ref) — has moved from monolith to microservices: **retail accounts, transfers/payments, cards, wealth, notifications**, plus a **TPP-facing open-banking API** (PSD2-style: AISP/PISP access with user consent). Requirements driving the identity design:

- **One login, many services:** a customer signs in once; every service must know who they are without re-authenticating ("authenticate once, authorize everywhere").
- **Fine-grained authorisation:** tellers have role limits; transfers are amount- and risk-gated; TPPs see *only consented accounts*; AML-flagged customers are restricted. RBAC alone cannot express this — ABAC policies are required (§6).
- **Regulatory:** SCA step-up for payments (two factors, dynamic linking), consent revocation for TPPs, full audit trail (§10).
- **Operations:** services scale independently; sessions must survive node churn; keys rotate without coordinated downtime (§4, §5).
- **S2S trust:** internal services call each other constantly; no static shared secrets (§8).

### 11.2 The Architecture: IdP + OIDC + JWKS + RBAC/ABAC

```
                    ┌──────────────────────────────────────────────────────┐
                    │                 MERLION BANK IDENTITY                 │
                    │                                                      │
   customer ──────▶ │  IdP (Keycloak-class)   ── OIDC ──▶  JWT + JWKS      │
   (browser/app)    │  • login / MFA / passkeys (WebAuthn)                 │
                    │  • SCA step-up (dynamic linking)                     │
                    │  • consent & TPP management (AISP/PISP)              │
                    └───────────────┬───────────────────┬──────────────────┘
                                    │                   │
                    OIDC code flow  │                   │  JWKS (public keys, RFC 7517)
                                    ▼                   ▼
                    ┌─────────────────────────────────────────────┐
                    │   API GATEWAY (Kong/Apigey-class)           │
                    │  • JWT validation (alg/exp/iss/aud, JWKS)   │  ◀── PEP #1 (token authn)
                    │  • rate limits & quotas                     │      (rate limiter guide §11)
                    │  • inject verified identity downstream      │
                    └──────────────┬──────────────┬───────────────┘
                                   │              │
                    east-west mTLS │              │ north-south requests
                    (SPIFFE/SPIRE) │              │ (verified JWT claims)
                                   ▼              ▼
        ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
        │  accounts-api    │   │  payments-api    │   │  cards-api       │
        │  (role checks)   │   │  (OPA decision)  │   │  (role checks)   │
        └────────┬─────────┘   └────────┬─────────┘   └────────┬─────────┘
                 │                      │                      │
                 └──────────┬───────────┴──────────┬───────────┘
                            │                      │
                    ┌───────▼────────┐      ┌──────▼────────┐
                    │   OPA (PDP)    │      │  Redis        │
                    │  Rego policies │      │  sessions +   │
                    │  • ABAC rules  │      │  rate keys    │
                    │  • SCA rules   │      │  (shared)     │
                    └────────────────┘      └───────────────┘
```

**The components, mapped to this guide:**

- **IdP** — authentication only (the who): login, MFA/passkeys, SCA step-up, consent records, TPP management. Issues OIDC ID tokens and OAuth access tokens (JWT, §2.4–2.5) and publishes the **JWKS** (§5.1).
- **Gateway** — the edge PEP: validates every JWT offline against the JWKS (alg pinning, `exp`, `iss`, `aud`, scope — §5.2), enforces rate limits (distributed_rate_limiter_guide.md), and injects the *verified* identity downstream. North-south security lives here.
- **Services** — consume verified identity claims; each enforces its own authorisation via **RBAC** (staff roles: teller/approver/auditor — coarse, legible, §6.1) and delegates the fine-grained rules to **OPA** (ABAC: amounts, risk scores, AML flags, SCA requirements — §6.2, §7.1).
- **OPA (PDP)** — the Rego policies from §7.2: `teller can initiate domestic transfers < SGD 50k unless AML-flagged`; `payments need SCA if amount > SGD 5k or risk > 40`; TPP consent checks (`account:123#consent@tpp-42` — a relationship tuple flavour, §6.3). Fail-closed on engine errors.
- **Redis** — shared session store for the browser session (IdP-fronted) and rate-limit counters (§4, distributed_rate_limiter_guide.md).
- **SPIFFE/SPIRE + mesh mTLS** — east-west: every service-to-service call is mTLS-authenticated with SPIFFE IDs, issued by SPIRE via platform attestation (§8). The mesh proves *which workload* called; OPA decides *what it may do*.

### 11.3 The Flow

**Customer-initiated transfer (the happy path):**

```
1.  Customer logs in at the IdP → MFA (knowledge + possession)
    → IdP starts the SSO session, issues OIDC ID token + access token (JWT)
2.  Gateway validates the JWT against the JWKS (alg/exp/iss/aud/scope) → passes
    → rate limiter: transfer API budget checked (Redis) → passes
3.  Gateway forwards verified identity (sub, scopes) to payments-api
4.  payments-api calls OPA with the decision input:
    { subject: teller-31, action: initiate_transfer,
      resource: { type: domestic, amount: SGD 12,000, aml_flag: false },
      context: { risk: 35, channel: branch } }
    → OPA evaluates Rego: allow (teller role ✓, amount < 50k ✓, no AML flag ✓)
      but… SCA rule fires: amount > SGD 5k ⇒ step-up required
5.  payments-api requests step-up via the IdP → customer confirms with
    possession factor + dynamic linking ("Pay SGD 12,000 to Merlion Ltd?")
    → SCA proof (transaction-bound) recorded
6.  payments-api executes the transfer; audit log records:
    authn (IdP), SCA proof (dynamic link), authz decision (OPA rule id), correlation id
```

**TPP flow (AISP reading consented accounts):** TPP redirects customer to the IdP → customer consents to *specific accounts* (relationship tuples: `acct:4821#consent@tpp-42`) → TPP receives code → exchanges for scoped access token (`accounts:read`) → gateway validates; cards-api/accounts-api ask OPA `can tpp-42 read acct:4821?` → consent tuple exists → allow. Customer revokes consent at the dashboard → tuple deleted → next check denies (§10.3).

### 11.4 The Lessons

- **Authenticate once, authorize everywhere.** The IdP authenticates; the gateway validates; the services authorize. Nobody downstream re-implements login, and nobody trusts a token without a signature check.
- **Separate the layers.** Authn (IdP/mTLS) ≠ token validation (gateway/JWKS) ≠ authorization (RBAC + OPA). Each layer has its own failure modes; mixing them (e.g., "the JWT scope is the authorization") is how BOLA happens.
- **Fast path, sensitive path.** Offline JWKS validation for every request; introspection/step-up for high-stakes ones. Revocation is a *design choice* (denylists, consent deletion, short `exp`), not an afterthought.
- **Machines are users too.** S2S identity is first-class: SPIFFE/SPIRE + mesh mTLS, no static secrets between services. Mesh mTLS proves *who*; OPA decides *what*.
- **Policies are code.** Rego in Git, reviewed, tested, versioned, fail-closed. The SCA exemption rules and the teller limits are the same policy machinery — one engine, one audit trail.
- **Regulation composes, not conflicts.** PSD2/SCA is step-up + consent + audit — a stricter instance of the same architecture, not a different one.
- **Start simple.** Merlion Bank's sequence (mirroring api_governance_guide.md §10.3): RBAC in services first; add the IdP + OIDC; add OPA when amount/risk rules multiply; add the mesh when S2S sprawl hurts. Each step forced by a measured problem.

---

## 12. The Summary: Auth in One Page

**Authentication and authorisation in a distributed system, in one page.**

- **The distinction:** **authn** — prove *who* you are (factors: knowledge/possession/inherence); **authz** — decide *what* you may do (subject × action × resource × context). Authn first, authz second, and never conflate them (§1).
- **The distributed problem:** many services must trust one identity without re-implementing login; session state must live somewhere shared; and keys must be distributed, rotated, and verified without leaking (§1.3).
- **The mechanisms:** passwords (salted slow hashes + MFA), SAML 2.0 (2005, enterprise XML federation), OAuth 2.0 (RFC 6749, 2012 — delegated *access*: auth-code+PKCE, client credentials), OIDC (2014 — the identity layer: ID tokens), JWT (RFC 7519, 2015 — the stateless claim format), mTLS (RFC 8705 — certificate machine identity), WebAuthn (W3C 2019 — phishing-resistant passkeys) (§2).
- **Sessions vs tokens:** stateful sessions = pointer + shared store, instant revocation; stateless tokens = self-contained claims, offline verification, hard revocation. Production = hybrid: IdP session at the front door, JWT passport in the back end (§3).
- **Distributed sessions:** Redis as the shared store (TTL, atomic ops, HA); sticky sessions as the anti-pattern at scale; client-side cookies as the stateless extreme (§4).
- **Token validation:** JWKS (RFC 7517) + `kid` lookup + algorithm pinning + `exp`/`iss`/`aud` — the same checks at every gateway and service; introspection (RFC 7662) for the sensitive path (§5).
- **The authorisation models:** RBAC (NIST INCITS 359-2004) for legible coarse roles; ABAC (NIST SP 800-162) for dynamic attribute rules; ReBAC (Google Zanzibar, 2019) for per-object relationships. Compose, don't choose (§6).
- **Policy engines:** OPA (CNCF) makes authorization *policies as code* — Rego in Git, sidecar PDPs, fail-closed, audited (§7).
- **Service-to-service:** SPIFFE/SPIRE (CNCF) give workloads verifiable IDs (`spiffe://…`) via attestation; mesh mTLS (Istio/Linkerd) enforces them at the transport layer. Authn for machines — authz still your job (§8).
- **SSO and enterprise patterns:** one IdP, one login, every app trusts the signed assertion; the gateway is the edge PEP (tokens, scopes, rate limits); edge coarse + service fine (§9).
- **Banking:** PSD2 + SCA (RTS 2018/389) = step-up authentication with transaction binding, mutual auth, consent-scoped TPP access, audit everywhere (§10).

**The final word: authenticate once, authorize everywhere.** The identity architecture of a distributed system is not a pile of tokens and protocols — it is a *separation of duties made structural*: one trusted place to authenticate (the IdP), one disciplined way to carry the proof (JWKS-verifiable tokens), one shared way to remember state (Redis), one codified way to decide (OPA + RBAC/ABAC/ReBAC), and one verifiable way for machines to trust each other (SPIFFE + mTLS). Get that skeleton right, and every new service is just another consumer of the same rails; get it wrong, and each service reinvents the identity wheel — nine authn schemes, zero consistency, one audit finding. Authenticate once, authorize everywhere, and treat revocation as a first-class citizen: identity is only as strong as the fastest way to take it away.

---

## 13. Verification and Claims-Status

✅ **Verified in this research pass (primary/secondary sources):**
- **OAuth 2.0** — RFC 6749, October 2012; the four grants (authorization code, implicit, client credentials, ROPC) and the token/scope model; **PKCE** — RFC 7636 (2015).
- **OIDC** — OpenID Foundation final specification (February 2014); identity layer on top of OAuth 2.0; ID token (JWT), UserInfo, discovery document.
- **JWT** — RFC 7519, May 2015; compact URL-safe claims, JWS/JWE, `iss/sub/aud/exp/nbf/iat` claims.
- **JWKS / JWK** — RFC 7517 (2015); JSON key-set structure, `kid`-based key selection.
- **SAML 2.0** — OASIS Standard, March 2005; XML assertions (authentication/attribute statements), SP-initiated and IdP-initiated flows.
- **WebAuthn** — W3C Recommendation 4 March 2019 (Level 1; Level 2, 8 April 2021); core of FIDO2 alongside CTAP; passkeys = FIDO Alliance productisation of discoverable credentials.
- **Zanzibar** — "Zanzibar: Google's Consistent, Global Authorization System," USENIX ATC 2019; relationship tuples, namespaces, global consistency; open-source descendants OpenFGA, SpiceDB, Ory Keto.
- **OPA** — Open Policy Agent: created by Styra, first released 2016, CNCF graduated project (first CNCF authorization-focused project to graduate); Rego declarative language; OPA 1.0 (December 2024); Envoy ext_authz and Kubernetes Gatekeeper integrations.
- **SPIFFE / SPIRE** — CNCF projects; SPIFFE ID format (`spiffe://trust-domain/path`), X.509-SVID / JWT-SVID; SPIRE does node + workload attestation with no bootstrap secret; Istio/Linkerd issue SPIFFE-compatible mTLS certificates.
- **RBAC** — NIST reference model, published as ANSI INCITS 359-2004; user→role→permission model.
- **ABAC** — NIST SP 800-162 (2014), "Guide to Attribute Based Access Control Definition and Considerations"; subject/resource/action/environment attributes.
- **mTLS / OAuth mTLS** — mutual TLS client-certificate authentication; RFC 8705 (2020) for OAuth client auth and certificate-bound access tokens.
- **PSD2 / SCA** — Directive (EU) 2015/2366 (in force 13 Jan 2018); RTS on SCA and secure communication, Commission Delegated Regulation (EU) 2018/389 (applicable 14 Sep 2019); two-of-three independent factors, dynamic linking, exemptions (incl. transaction risk analysis); EBA Q&A on independence of elements (2020_5619).
- **Redis as the session-store / shared-state default** — sub-ms reads, TTL, atomic ops, Sentinel/Cluster HA — consistent with Redis documentation and the rate-limiter architecture in distributed_rate_limiter_guide.md.
- **OWASP API Security Top 10 (2023)** — API1 Broken Object Level Authorization (BOLA) is the top API risk; consistent with api_governance_guide.md §8.2.
- **API gateway as token-validation PEP** — Kong/Apigee/AWS API Gateway JWT-validation patterns; consistent with api_governance_guide.md §7–§8.

⚠ **Flagged (inferred, approximate, no single standard, or single-source):**
- **"Passwords should not be force-rotated on a calendar"** — current NIST SP 800-63B guidance; older policies mandated rotation. The landscape shifted; treat as current-best-practice rather than eternal law.
- **Implicit flow "deprecated"** — RFC 6749 still lists it; the deprecation comes from OAuth 2.0 Security BCP (RFC 9700, 2025) and OIDC-era guidance. The direction is unambiguous; the word "deprecated" is a guidance status, not an RFC status. (Same framing as api_governance_guide.md §8.1.)
- **"The modern default is auth-code + PKCE for all user-facing apps"** — broad practitioner consensus (OAuth 2.0 Security BCP), but some confidential-client-only deployments still use plain auth-code without PKCE; PKCE is the safe default.
- **"OPA is the de-facto standard policy engine"** — strong practitioner consensus and CNCF graduation, but it is not a formal ISO/IETF standard; and OPA is a *policy engine*, not a full authorization service (no relationship store, no identity management).
- **"Istio issues SPIFFE-compatible identities"** — documented by Istio (spiffe:// trust domain); Linkerd uses its own identity scheme (mTLS with Kubernetes service accounts as the trust anchor). Treat mesh specifics as vendor-documented behavior.
- **PSD3/PSR successor status and any forward-looking regulatory statements** — status changes over time; flagged as such (see api_governance_guide.md §10.1).
- **Merlion Bank worked example and all capacity/design numbers** — entirely illustrative; consistent with the api_governance_guide.md §10.3 worked example.
- **XACML described as "legacy/superseded"** — XACML remains in use in enterprise estates; the claim is about *new* adoption favouring OPA-style engines, which is practitioner consensus rather than a standard verdict.
- **Passkey adoption and vendor support statements** — moving fast (platform sync ecosystems, FIDO Alliance roadmap); verify current vendor support before committing.

---

## 14. Glossary

- **Authentication (authn)** — Verifying *who* a caller claims to be: proving an identity claim via factors (knowledge, possession, inherence).
- **Authorization (authz)** — Deciding *what* an authenticated principal may do: a yes/no decision over subject, action, resource, and context.
- **Password** — The knowledge-factor secret; must be stored salted + slow-hashed (bcrypt/scrypt/Argon2id) and augmented with MFA.
- **SAML (Security Assertion Markup Language)** — OASIS standard (SAML 2.0, 2005) for exchanging signed XML authentication/attribute assertions between an IdP and service providers; enterprise/legacy SSO federation.
- **OAuth 2.0** — RFC 6749 (2012): the delegated-authorization framework; scoped access tokens via grants (authorization code + PKCE, client credentials, …); the conveyor belt for authorization, not identity.
- **RFC 6749** — The OAuth 2.0 Authorization Framework standard (October 2012).
- **OIDC (OpenID Connect)** — OpenID Foundation spec (2014): the identity layer on top of OAuth 2.0; adds the signed ID token, UserInfo, and discovery.
- **JWT (JSON Web Token)** — RFC 7519 (2015): compact, URL-safe, signed claims format (header.payload.signature); the stateless token format of the API era.
- **RFC 7519** — The JSON Web Token standard (May 2015).
- **mTLS (Mutual TLS)** — TLS with client certificates: both peers authenticate cryptographically at the transport layer; RFC 8705 (2020) for OAuth client auth + certificate-bound tokens.
- **WebAuthn** — W3C Recommendation (2019): browser/platform public-key authentication; core of FIDO2; phishing-resistant.
- **Passkey** — FIDO Alliance productisation of discoverable WebAuthn credentials, synchronised across devices; passwordless sign-in.
- **Session** — Server-side authenticated state referenced by an opaque session ID (cookie); stateful, instantly revocable.
- **Token** — A self-contained credential (typically JWT) carrying claims; stateless, offline-verifiable, hard to revoke before expiry.
- **Session store** — The shared server-side state holding sessions; Redis is the default (TTL, atomic ops, HA).
- **Redis** — In-memory key-value store; the standard shared store for sessions and rate-limit counters in distributed systems.
- **JWKS (JSON Web Key Set)** — RFC 7517: the JSON document of public signing keys, published by the IdP and used for offline token verification; key selection via `kid`.
- **RFC 7517** — The JSON Web Key (JWK) / JWK Set standard (2015).
- **RBAC (Role-Based Access Control)** — Users → roles → permissions; NIST/ANSI INCITS 359-2004; the legible enterprise baseline.
- **ABAC (Attribute-Based Access Control)** — Policies over subject/resource/action/environment attributes; NIST SP 800-162 (2014); dynamic fine-grained decisions.
- **ReBAC (Relationship-Based Access Control)** — Permissions as relationship tuples (`object#relation@user`) checked by graph traversal; canonicalised by Google's Zanzibar (2019).
- **Zanzibar** — Google's consistent, global authorization system (USENIX ATC 2019 paper); the reference architecture for ReBAC at scale.
- **OPA (Open Policy Agent)** — CNCF-graduated policy engine (Styra, 2016); declarative Rego policies; the standard policy-as-code PDP.
- **SPIFFE** — CNCF standard for workload identity: `spiffe://trust-domain/path` IDs and short-lived SVIDs.
- **SPIRE** — SPIFFE Runtime Environment: the reference implementation issuing SVIDs via node/workload attestation, no bootstrap secrets.
- **Service mesh** — The sidecar-proxy layer (Istio, Linkerd) providing mTLS, traffic control, and observability between workloads; enforces S2S authn, not authz.
- **SSO (Single Sign-On)** — One authentication at the IdP, accepted across all applications via signed assertions/tokens.
- **IdP (Identity Provider)** — The system of record for identity: authenticates users, issues tokens/assertions, manages directory/lifecycle/MFA; the trust root of the identity architecture.
- **Gateway** — The edge enforcement point (Kong, Apigee, AWS API Gateway, Azure APIM): token validation, scopes, rate limits; the PEP for north-south API traffic.
- **PSD2** — EU Directive (EU) 2015/2366 (in force 2018): opens account data and payment initiation to licensed TPPs under strict security rules.
- **SCA (Strong Customer Authentication)** — PSD2/RTS (EU) 2018/389 requirement: two independent factors from different categories, dynamic linking to the transaction, risk-based exemptions.
- **PEP / PDP** — Policy Enforcement Point (blocks/forwards in the request path) vs Policy Decision Point (computes the decision); the authz anatomy every design should name.

---

*End of guide. Feedback welcome via the repository.*
