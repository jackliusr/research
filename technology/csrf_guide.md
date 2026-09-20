# Cross-Site Request Forgery (CSRF) — The Web's Missing-Intent Vulnerability

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui, Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Web Application Security — the dedicated CSRF deep-dive. Where [distributed_auth_guide.md](distributed_auth_guide.md) covers **identity** (authentication, authorization, and the session-attack family — fixation, hijacking), [security_by_design_guide.md](security_by_design_guide.md) covers the **design-time discipline** and the weakness rankings, [threat_modeling_guide.md](threat_modeling_guide.md) covers the **threat-model method**, and [../banking/fapi_financial_grade_api_guide.md](../banking/fapi_financial_grade_api_guide.md) covers **OAuth/OIDC flow protections** (where the `state` parameter is the CSRF defence), this guide owns the one thing none of them do: the **missing-intent mechanism itself** — why the browser attaches a user's authority to a request the user never formed, what actually defends against it in 2026, where the defence still fails, and how to test that it works.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the specs and vendor engineering docs, not tutorials — the Fetch Metadata Request Headers specification (W3C Web Application Security Working Group, Working Draft 1 April 2025: https://w3c.github.io/webappsec-fetch-metadata/), the Cookies specification (draft-ietf-httpbis-rfc6265bis, rev. 22, December 2025: https://www.ietf.org/archive/id/draft-ietf-httpbis-rfc6265bis-22.txt), MDN (the CSRF glossary entry, the `Set-Cookie` reference), the Chromium SameSite launch timeline (chromium.org/updates/same-site), the Chromium blog, Mozilla Bugzilla (bugs 1617609 and 1812297 — see §4, these two bugs overturn a claim repeated in most published guides), WebKit Bugzilla (bugs 278353 and 279153), the OWASP CSRF Prevention Cheat Sheet, MITRE CWE-352, and the academic origin: Barth, Jackson & Mitchell, *Robust Defenses for Cross-Site Request Forgery*, ACM CCS '08, October 2008.
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — plain filenames):** [Distributed Auth](distributed_auth_guide.md) (**THE identity companion — session and cookie mechanics live THERE, not here; §1.5 draws the line**), [Security-by-Design](security_by_design_guide.md) (**THE design discipline — CWE-352 is cited THROUGH that guide's weakness canon, not re-derived here**), [Threat Modeling](threat_modeling_guide.md) (the modelling method — CSRF is a one-click/repudiation-flavoured threat there), [Cybersecurity](cybersecurity_guide.md), [Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md), [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) (the general security discipline — cross-referenced, not restated), [API Governance](api_governance_guide.md) (the API angle), [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) and [Test Orchestration](test_orchestration_guide.md) (**THE testing harnesses — §9 points at these rather than re-deriving a test rig**), [Chaos Engineering](chaos_engineering_guide.md) (failure injection — the adversarial analogue)
> **Companion guides (banking/, prefix `../banking/`):** [FAPI Financial-Grade API](../banking/fapi_financial_grade_api_guide.md) (**THE OAuth/OIDC companion — the `state` parameter is that guide's CSRF control; cross-ref §1.5 and §6**), [RegTech](../banking/regtech_guide.md) (the compliance machinery behind §10's evidence discussion), [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (the resilience taxonomy)
> **Verification:** this pass had live web access. Every specification quotation, browser version number and date below was checked against a primary source in September 2026 and is listed in the claims audit (§13). Items that could not be pinned down are flagged ⚠ and enumerated in §14.

---

**How to use this guide:** Section 1 is the framing — CSRF as a *missing-intent* bug rather than an injection bug, plus the vocabulary decoder. Section 2 is the mechanism, step by step, and the crux: why the same-origin policy does **not** stop the request. Section 3 is the defence stack in order of modern effectiveness — the section the guide is built around. Section 4 is the browser-level change and `SameSite` in depth. Section 5 is the token patterns and their real weaknesses. Section 6 is the API/SPA era. Section 7 separates the adjacent bugs exactly. Section 8 is the verified history. Section 9 is testing. Section 10 is the regulated-enterprise angle. Section 11 is the Cymbal Bank worked example (fictional). Section 12 is anti-patterns. Section 13 is the claims audit. Section 14 is what could not be verified. Section 15 is the glossary and cross-references. Section 16 is the closing summary. **Integrity convention:** ✅ = verified this pass against a named primary source with a date; ⚠ = flagged, partially verified, or unattributable to a primary source. Every cost figure in this guide is **ILLUSTRATIVE and fictional**.

---

## Table of Contents

1. [Overview, Definition and Decoder](#1-overview-definition-and-decoder)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [A Missing-Intent Bug, Not an Injection Bug](#12-a-missing-intent-bug-not-an-injection-bug)
   - 1.3 [The Confused Deputy, Named Honestly](#13-the-confused-deputy-named-honestly)
   - 1.4 [The Decoder — the Vocabulary You Need Before §2](#14-the-decoder--the-vocabulary-you-need-before-2)
   - 1.5 [The Boundary — What This Guide Owns, and What Its Siblings Own](#15-the-boundary--what-this-guide-owns-and-what-its-siblings-own)
2. [The Mechanism](#2-the-mechanism)
   - 2.1 [The Attack in Seven Steps](#21-the-attack-in-seven-steps)
   - 2.2 [The Crux — Why the Same-Origin Policy Does Not Stop the Request](#22-the-crux--why-the-same-origin-policy-does-not-stop-the-request)
   - 2.3 [The Credential-Attachment Rule — Why Cookies Are Ambient](#23-the-credential-attachment-rule--why-cookies-are-ambient)
   - 2.4 [Safe and Unsafe Methods — and the State-Changing GET](#24-safe-and-unsafe-methods--and-the-state-changing-get)
   - 2.5 [The Classic Vectors](#25-the-classic-vectors)
   - 2.6 [The Variants](#26-the-variants)
   - 2.7 [A Worked Attack Against a Multi-Step Flow](#27-a-worked-attack-against-a-multi-step-flow)
   - 2.8 [What the Attacker Can and Cannot Observe](#28-what-the-attacker-can-and-cannot-observe)
3. [The Defence Stack, in Order of Modern Effectiveness](#3-the-defence-stack-in-order-of-modern-effectiveness)
   - 3.1 [The Ordering, and Why It Is Contested](#31-the-ordering-and-why-it-is-contested)
   - 3.2 [Defence 1 — Fetch Metadata (`Sec-Fetch-Site` and Family)](#32-defence-1--fetch-metadata-sec-fetch-site-and-family)
   - 3.3 [Defence 2 — SameSite Cookies](#33-defence-2--samesite-cookies)
   - 3.4 [Defence 3 — Origin and Referer Validation](#34-defence-3--origin-and-referer-validation)
   - 3.5 [Defence 4 — The Token Patterns](#35-defence-4--the-token-patterns)
   - 3.6 [Defence 5 — Custom Headers and the Preflight Consequence](#36-defence-5--custom-headers-and-the-preflight-consequence)
   - 3.7 [Defence 6 — Re-authentication and Step-Up](#37-defence-6--re-authentication-and-step-up)
   - 3.8 [The Defence-in-Depth Argument, Stated Honestly](#38-the-defence-in-depth-argument-stated-honestly)
   - 3.9 [The Combination a Modern Application Should Actually Deploy](#39-the-combination-a-modern-application-should-actually-deploy)
4. [The Browser-Level Change and SameSite in Depth](#4-the-browser-level-change-and-samesite-in-depth)
   - 4.1 [What Changed, Precisely — Spec Value, First Browser, Version, Year](#41-what-changed-precisely--spec-value-first-browser-version-year)
   - 4.2 [How the Rollout Actually Proceeded](#42-how-the-rollout-actually-proceeded)
   - 4.3 [The "Lax+POST" Two-Minute Mitigation, and Whether It Still Exists](#43-the-laxpost-two-minute-mitigation-and-whether-it-still-exists)
   - 4.4 [What `SameSite=Lax` Actually Permits](#44-what-samesitelax-actually-permits)
   - 4.5 [The Three Values in Full, and the `None`+`Secure` Requirement](#45-the-three-values-in-full-and-the-nonesecure-requirement)
   - 4.6 [What Firefox and Safari Actually Do — the Correction](#46-what-firefox-and-safari-actually-do--the-correction)
   - 4.7 [Practical Misconfiguration Patterns](#47-practical-misconfiguration-patterns)
5. [The Token Patterns in Depth](#5-the-token-patterns-in-depth)
   - 5.1 [Why a Token Works at All](#51-why-a-token-works-at-all)
   - 5.2 [The Synchronizer Token, End to End](#52-the-synchronizer-token-end-to-end)
   - 5.3 [Per-Session Versus Per-Request — the Real Trade](#53-per-session-versus-per-request--the-real-trade)
   - 5.4 [The Double-Submit Cookie and Its Documented Exposure](#54-the-double-submit-cookie-and-its-documented-exposure)
   - 5.5 [The Signed / HMAC-Bound Variant — What It Actually Buys](#55-the-signed--hmac-bound-variant--what-it-actually-buys)
   - 5.6 [Where the Token Lives — Session, Cookie, Meta Tag, Form](#56-where-the-token-lives--session-cookie-meta-tag-form)
   - 5.7 [Stateless Architectures and the Honest Limits](#57-stateless-architectures-and-the-honest-limits)
6. [The API and Single-Page-Application Era](#6-the-api-and-single-page-application-era)
   - 6.1 [Why a Bearer Token in a Header Is Structurally Different](#61-why-a-bearer-token-in-a-header-is-structurally-different)
   - 6.2 [The CORS Interaction — a Read Restriction, Not a Send Restriction](#62-the-cors-interaction--a-read-restriction-not-a-send-restriction)
   - 6.3 [Same-Site Versus Cross-Site API Calls in an SPA](#63-same-site-versus-cross-site-api-calls-in-an-spa)
   - 6.4 [The Mobile and Hybrid Client Case](#64-the-mobile-and-hybrid-client-case)
   - 6.5 [The Honest Claim — Exposure Reduced by Changing the Credential Model](#65-the-honest-claim--exposure-reduced-by-changing-the-credential-model)
7. [The Adjacent Bugs, Separated Exactly](#7-the-adjacent-bugs-separated-exactly)
   - 7.1 [The Shared Substrate — the Same-Origin Policy](#71-the-shared-substrate--the-same-origin-policy)
   - 7.2 [CSRF vs XSS — the Asymmetry in Both Directions](#72-csrf-vs-xss--the-asymmetry-in-both-directions)
   - 7.3 [CSRF vs CORS](#73-csrf-vs-cors)
   - 7.4 [CSRF vs Clickjacking](#74-csrf-vs-clickjacking)
   - 7.5 [The Comparison Table, and the Misstatements It Corrects](#75-the-comparison-table-and-the-misstatements-it-corrects)
8. [The History, Condensed and Verified](#8-the-history-condensed-and-verified)
   - 8.1 [The Named Origin — 2001 to 2008](#81-the-named-origin--2001-to-2008)
   - 8.2 [The Defensive Practices That Emerged](#82-the-defensive-practices-that-emerged)
   - 8.3 [The Standards Work Behind the Cookie Attribute](#83-the-standards-work-behind-the-cookie-attribute)
   - 8.4 [The Standards Work Behind Fetch Metadata](#84-the-standards-work-behind-fetch-metadata)
   - 8.5 [How the Browser Default Change Moved the Baseline](#85-how-the-browser-default-change-moved-the-baseline)
   - 8.6 [Where the Early Record Is Genuinely Thin](#86-where-the-early-record-is-genuinely-thin)
9. [Testing and Verification](#9-testing-and-verification)
   - 9.1 [What a CSRF Test Is Actually Trying to Prove](#91-what-a-csrf-test-is-actually-trying-to-prove)
   - 9.2 [The Manual Checks That Matter](#92-the-manual-checks-that-matter)
   - 9.3 [Automated Scanning and its False Positives](#93-automated-scanning-and-its-false-positives)
   - 9.4 [The Test That Actually Matters — a Real Browser With a Real Session](#94-the-test-that-actually-matters--a-real-browser-with-a-real-session)
   - 9.5 [The Regression Test — Encoding the Defence So It Cannot Silently Rot](#95-the-regression-test--encoding-the-defence-so-it-cannot-silently-rot)
10. [The Regulated-Enterprise and Banking Angle](#10-the-regulated-enterprise-and-banking-angle)
    - 10.1 [Why This Class Matters More Than Its Modern Frequency Suggests](#101-why-this-class-matters-more-than-its-modern-frequency-suggests)
    - 10.2 [High-Value State-Changing Actions and the Step-Up Question](#102-high-value-state-changing-actions-and-the-step-up-question)
    - 10.3 [Mobile-Channel Differences](#103-mobile-channel-differences)
    - 10.4 [Fraud Monitoring as a Complement, Not a Substitution](#104-fraud-monitoring-as-a-complement-not-a-substitution)
    - 10.5 [The Evidence and Audit Angle — Tested, Not Asserted](#105-the-evidence-and-audit-angle--tested-not-asserted)
    - 10.6 [The Third-Party and Agency Question](#106-the-third-party-and-agency-question)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
    - 11.1 [The Scenario](#111-the-scenario)
    - 11.2 [Step 1 — the Endpoint Inventory](#112-step-1--the-endpoint-inventory)
    - 11.3 [Step 2 — Defence Selection per Endpoint Class](#113-step-2--defence-selection-per-endpoint-class)
    - 11.4 [Step 3 — the SameSite and Token Configuration](#114-step-3--the-samesite-and-token-configuration)
    - 11.5 [Step 4 — the Fetch Metadata Policy](#115-step-4--the-fetch-metadata-policy)
    - 11.6 [Step 5 — Migration Sequencing and Compatibility Risk](#116-step-5--migration-sequencing-and-compatibility-risk)
    - 11.7 [Step 6 — the Testing Plan](#117-step-6--the-testing-plan)
    - 11.8 [Step 7 — the Options Comparison (ILLUSTRATIVE figures)](#118-step-7--the-options-comparison-illustrative-figures)
    - 11.9 [The Recommendation, the Re-architecture, and the Accepted Residual](#119-the-recommendation-the-re-architecture-and-the-accepted-residual)
12. [Anti-Patterns and Misconfigurations](#12-anti-patterns-and-misconfigurations)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [The Glossary and Cross-References](#15-the-glossary-and-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. Overview, Definition and Decoder

### 1.1 The Short Answer

**Cross-Site Request Forgery is the exploitation of ambient authority.** A user is logged into a site. Their browser holds a credential — normally a session cookie — and the browser's rule is that this credential is attached to *any* request to that site, regardless of which page caused the request to be made. An attacker therefore does not need to steal the credential. They need only cause the victim's browser to issue a request. The server receives a request that carries fully valid authority, is structurally indistinguishable from a legitimate one, and performs whatever state change it was designed to perform. The user never intended it. The server cannot tell.

CWE-352 defines it precisely: *"The web application does not, or cannot, sufficiently verify whether a request was intentionally provided by the user who sent the request, which could have originated from an unauthorized actor."* ✅ (MITRE CWE-352, page last updated 30 April 2026). Note the wording — **intentionally provided by the user**. That is the whole bug. Not "maliciously crafted input", not "unvalidated parameter". *Intent* is the missing property.

The name follows: the request is *cross-site* (the attacker's page initiates it), it is a *request* (not a passive read), and it is a *forgery* of intent — the request itself is genuine, the intent behind it is fabricated. Microsoft's threat-modelling tradition calls it a **one-click attack**; it also travels as **XSRF**, **session riding**, and **sea surf** ✅ (alternate terms, CWE-352 and OWASP).

One consequence of that definition shapes everything that follows: **CSRF is a property of the credential model, not of a payload.** An application that authenticates with a credential the browser attaches automatically is exposed. An application that authenticates with a credential the *application* must attach explicitly is not. This is why, as §6 argues, a great deal of modern "CSRF has been solved" is really "a lot of modern APIs changed how they carry credentials" — and why the vulnerability survives wherever the cookie model survives.

### 1.2 A Missing-Intent Bug, Not an Injection Bug

The most useful framing in this whole guide is a taxonomy one. Every other famous web bug class is a *data* bug; CSRF is a *metadata* bug. That difference determines which defences can possibly work.

| Bug class | What the attacker supplies | What is broken | Defence locus |
|---|---|---|---|
| SQL injection (CWE-89) | **Hostile input** — a crafted string | Input is trusted as code | Input handling: parameterise, validate, escape |
| Cross-site scripting (CWE-79) | **Hostile code** — script that runs in your origin | Output is trusted as inert text | Output encoding, CSP, sanitisation |
| Cross-Site Request Forgery (CWE-352) | **Nothing at all** — no input reaches you | A *legitimate* request is accepted with unintended intent | Request *context*: prove the request came from your own UI |

Read the third row again. In a CSRF attack there is no malformed input to detect, no payload to escape, no signature to match. The request body is a well-formed statement of a legitimate operation. The parameters are valid. The authentication is valid. The only thing wrong with the request is *who caused it* — and that fact lives outside the request body, in metadata the browser attaches (or fails to attach).

This is why CSRF defences are all *provenance* checks rather than *sanitisation* checks. Every defence in §3 answers one question: **can I establish that this request originated from my own application's UI, in a context where I control what my own UI can be made to do?** Fetch Metadata headers answer it from the browser's own classification of the request. SameSite answers it by withholding the credential. Origin/Referer answer it from headers the page cannot forge. Tokens answer it by requiring a secret that only a same-origin page could have read. Step-up auth answers it by requiring the human to prove presence again. None of them inspect the payload, because the payload is not the problem.

### 1.3 The Confused Deputy, Named Honestly

CSRF is the canonical **confused deputy** (CWE-441, *Unintended Proxy or Intermediary*, one of the three component weaknesses of the CWE-352 composite ✅). The deputy is the web server; the authority it wields is the user's session; the confusion is that the server cannot distinguish a request the user *chose* from a request the user's browser *performed*.

The phrase matters architecturally because it points at the fix. A deputy is confused when it authorises an action it is asked to take without establishing that the *principal* wanted it. The remedies are the familiar access-control remedies: don't rely on ambient authority, require the principal to demonstrate intent per action, and record which principal actually caused the action. OWASP names the same thing in the same words: *"the most vulnerable component is the client-side JavaScript program because an attacker can use it to generate arbitrary asynchronous requests by manipulating the request endpoint and/or its parameters... it reintroduces the confused deputy flaw, that is, the server-side won't, again, be able to distinguish if the request was performed intentionally or not"* ✅ (OWASP CSRF Prevention Cheat Sheet, §"Dealing with Client-Side CSRF Attacks").

One further property of the confused deputy deserves emphasis because it defines the blast radius: **CSRF grants no privileges the victim does not already have.** The attacker does not escalate; they *borrow*. As CWE-352 puts it, *"Because the attacker has the identity of the victim, the scope of CSRF is limited only by the victim's privileges"* ✅. The corollary is that impact is a function of who the victim is and what their session can do — which is precisely why, in a retail banking context (§10), the same bug that is a nuisance on a forum is an account-takeover primitive on a payments page.

### 1.4 The Decoder — the Vocabulary You Need Before §2

The rest of the guide assumes these terms. Learn them here; they are used precisely and without re-definition from §2 onward.

| Term | Definition | Why it matters for CSRF |
|---|---|---|
| **Ambient authority** | Authority a system grants based on *who the caller is*, attached automatically and available to any code acting as that caller, without the caller choosing to present it. | The root cause. A session cookie is ambient authority: the browser presents it whether the human asked for the request or not. |
| **Same-origin policy (SOP)** | The browser rule that script from origin A may not *read* responses from origin B. Origin = scheme + host + port. | Prevents the attacker from reading the response. It does **not** prevent the request being sent — the single most important fact in §2. |
| **Credential-attachment rule** | The browser's rule for when cookies (and HTTP auth credentials, and client certs) are attached to a request: based on the *destination* and the request's cookie attributes, not on who initiated it. | Explains why the attacker needs no credential theft. The browser volunteers the credential. |
| **Safe method** | Per HTTP semantics, `GET`, `HEAD`, `OPTIONS`, `TRACE` — methods defined as retrieval-only, carrying no intended state change. | The `SameSite=Lax` exemption is keyed to method safety. A state-changing GET therefore escapes it. |
| **Unsafe method** | `POST`, `PUT`, `PATCH`, `DELETE` — methods that may change state. | `SameSite=Lax` blocks these on cross-site requests; the token requirement applies to them. |
| **Top-level navigation** | A request that changes the URL in the browser's address bar — a link click, a form submission, a `window.location` assignment. As opposed to a subresource fetch (`<img>`, `<script>`, `fetch()`). | The `Lax` allowance is scoped to top-level navigations specifically. Subresource requests never get the allowance. |
| **Synchronizer token** | A per-session (or per-request) secret generated server-side, embedded in the rendered page, and required back on each state-changing request, compared against the server's stored copy. | The classic defence. §5.2–5.3; §5.7 for its limits. |
| **Double-submit cookie** | A secret placed in a cookie *and* mirrored in a request parameter/header; the server accepts only if the two match. Stateless. | Weaker than it looks: §5.4 documents the cookie-injection exposure. |
| **`SameSite`** | A cookie attribute limiting whether the cookie is attached to cross-site requests. Values `Strict`, `Lax`, `None`; plus a spec-defined `Default` state. | The browser-level defence. §4 in full. |
| **`Origin`** | A request header, set by the browser not the page, giving the origin that initiated the request. | A provenance signal. Absent in some legitimate cases and set to `null` in others — §3.4. |
| **`Referer`** | A request header giving the URL (or, under modern defaults, only the origin) of the page that initiated the request. Suppressible by policy. | The fallback when `Origin` is absent. Weaker, because it is truncated and suppressible — §3.4. |
| **Fetch Metadata headers** | The `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User` request headers, set by the browser to describe *how* a request was made. | The structural defence. §3.2 for semantics, §8.4 for the standards work. |
| **Preflight** | The automatic `OPTIONS` request a browser sends before a *non-simple* cross-origin request, asking the server whether the real request is permitted. | The basis of the custom-header defence: a request carrying a non-simple header cannot be sent cross-site without a preflight the server can refuse — §3.6, §6.2. |

### 1.5 The Boundary — What This Guide Owns, and What Its Siblings Own

CSRF sits at the intersection of four topics that are frequently taught together and are therefore frequently muddled. The repository already covers three of them in depth; this guide covers the fourth and points at the others rather than re-deriving them.

| Topic | Owner in this repository | The line |
|---|---|---|
| **Session mechanics** — cookie attributes, session IDs, fixation, hijacking, rotation, expiry | [distributed_auth_guide.md](distributed_auth_guide.md) | That guide owns *how a session is established, stored and protected*. This guide owns *the fact that an established session is ambient*. **A session that is perfectly protected against fixation and hijacking is still fully CSRF-exposed.** CSRF is not a session bug; it is a consequence of the browser's credential-attachment rule meeting a server that does not check intent. |
| **Design-time discipline and weakness rankings** — OWASP Top 10, ASVS, CWE Top 25, secure-coding standards | [security_by_design_guide.md](security_by_design_guide.md) | That guide owns the rankings and the design principles. **CWE-352 is cited through it** rather than re-derived here; this guide takes the identifier, the mechanism and the countermeasures, and does not restate the SANS/MITRE Top 25 placement. |
| **Threat-modelling method** — STRIDE, DREAD, PASTA, attack trees | [threat_modeling_guide.md](threat_modeling_guide.md) | That guide owns the method. In STRIDE terms CSRF is *Spoofing* at the request layer combined with *Repudiation* (the user can credibly deny the action); this guide does not re-teach the taxonomy. |
| **OAuth/OIDC flow protections** — including the `state` parameter | [../banking/fapi_financial_grade_api_guide.md](../banking/fapi_financial_grade_api_guide.md) | That guide owns the OAuth flows. Understand the relationship precisely: **`state` is a CSRF defence for the *redirect/callback* leg of an authorisation flow** — it defeats an attacker who injects or replays an authorisation response into a victim's flow. It is not a defence for the resource endpoints behind the token. Do not conflate "we use `state`" with "we are protected". |
| **The general security discipline** — posture, frameworks, regulation | [cybersecurity_guide.md](cybersecurity_guide.md), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md), [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) | Those guides own the discipline. This guide is a single-vulnerability-class deep dive inside it. |
| **This guide** | — | The mechanism, the defence stack, the browser-level changes, the adjacent-bug distinctions, the API/SPA case, the testing, and the banking angle. |

One boundary deserves a sentence of its own, because it is the reason the confused-deputy framing is not merely rhetorical. **Zero-trust architecture does not automatically solve CSRF.** Zero trust removes implicit trust from *network position* and re-establishes it per request via explicit verification. But a cookie-bearing CSRF request arrives *already* explicitly verified as the user — that is the entire problem. Per-request authorisation does not help when the forgery *is* the authorised user's own authority, differently aimed. Zero trust helps with the identity and network halves; the intent half needs the defences in §3. This is exactly why CSRF survives every architectural fashion.

---

## 2. The Mechanism

### 2.1 The Attack in Seven Steps

Every CSRF attack, from the 2008 academic formulation to a 2026 bug-bounty report, is the same seven steps. Only steps 4 and 5 vary.

1. **The victim authenticates.** The user logs in to `https://bank.example` normally. The server sets a session cookie. The browser stores it, scoped to `bank.example`.
2. **The session stays live.** The user does not log out. The cookie remains valid. This is the normal state of a browser — most sessions are live most of the time.
3. **The victim visits the attacker's page.** Through any of the ordinary routes: a link in email or chat, a malicious advertisement, a page on a compromised legitimate site, an embedded widget, a QR code, a shortened URL.
4. **The attacker's page causes a request to `bank.example`.** It does not need the user to click a "Submit" button. The request can be triggered by a page load, a hidden form auto-submitted by script, an image or iframe whose `src` is the target URL, or a `fetch()`/`XMLHttpRequest` call.
5. **The browser attaches the session cookie.** Not because the attacker asked it to, and not because the user consented — because the browser's credential-attachment rule says that a request to `bank.example` carries `bank.example`'s cookies, and the cookie's own attributes permit it on this request. **The attacker never sees or handles the credential.**
6. **The server processes a well-formed, authenticated request.** It validates the session, finds it valid, checks the user's authorisation for the operation, finds it satisfied, and performs the state change.
7. **The attacker gets the effect, not the data.** The response goes to the victim's browser, where the attacker usually cannot read it — but the *state change* has happened on the server and is durable. The money moved; the email address changed; the password was reset; the payee was added.

Step 7 is what makes CSRF's impact asymmetrical and often delayed: the victim may not learn of the action for hours or days, and the attacker needs no channel back from the victim at all. An attack can be **fire-and-forget**.

### 2.2 The Crux — Why the Same-Origin Policy Does Not Stop the Request

This is the single most misunderstood point in the topic, and getting it wrong makes every subsequent defence look arbitrary. State it flatly:

> **The same-origin policy restricts the reading of responses. It does not restrict the sending of requests.**

The SOP exists to stop a page from origin A reading data belonging to origin B. It is a *confidentiality* control on responses. It has never been, and is not, a *request-origination* control. The browser will happily transmit a cross-site `POST` to `bank.example`, complete with cookies and a body; it will simply refuse to hand the *response body* back to the attacker's script if the response is not CORS-authorised. For a CSRF attack, the response is worthless anyway. The attacker wants the side effect.

This is why OWASP's own classical-CSRF walkthrough is careful to say of the `XMLHttpRequest`/`fetch` variant of the attack that *"this request will **not** be executed by modern web browsers thanks to same-origin policy restrictions"* ✅ — that statement is about **the specific case of a cross-origin `fetch`/XHR with a JSON body**, and it is true: modern browsers block *sending* certain cross-origin requests unless CORS permits it. It is emphatically **not** a general CSRF defence, and reading it as one is the origin of a great deal of bad advice. The forms of request that are **not** blocked are the ones that matter:

| Request form | Does SOP/CORS stop the request being sent? | Carries cookies? | Notes |
|---|---|---|---|
| Cross-origin `fetch()` with `Content-Type: application/json` | **Yes** — requires a successful preflight | If preflight passes and credentials are included | Blocked *by default* because a custom content type is "non-simple" |
| Cross-origin `<form method="POST">` submission | **No** — sent, unpreflighted | **Yes** | The classic vector. Form submission is a navigation, not a fetch. |
| Cross-origin top-level `GET` navigation (link click, `window.location`) | **No** | **Yes** (under `Lax` — see §4.4) | Exploitable whenever a `GET` changes state |
| Cross-origin subresource load — `<img>`, `<script>`, `<link>`, `<iframe>` | **No** | **Yes** for cookies whose `SameSite` permits | `<img src="…/transfer?amount=…">` — no script, no preflight |
| Cross-origin `fetch()` with `Content-Type: text/plain` (a "simple" request) | **No** — sent, unpreflighted | **Yes** if `credentials` set / enabled | The JSON-endpoint trap: a server that parses `text/plain` as JSON is CSRF-reachable — §2.6 |
| Cross-origin `fetch()` with a custom header | **Yes** — preflighted | If preflight passes | The basis of the custom-header defence — §3.6 |

The structural reason behind the table: the browser distinguishes **"simple" requests** — `GET`, `HEAD`, `POST` with one of `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` — from everything else. Simple requests are the ones the web has always been able to make cross-site (they are exactly the requests HTML forms and images and links have made since 1993); they are sent without preflight, and they carry cookies subject to the cookie's own rules. Everything that could not be made by 1993-era HTML — a custom header, a JSON body — is preflighted, and the server gets a veto. **CSRF lives entirely inside the simple-request space.** This is the fact that makes Fetch Metadata (§3.2) and custom-header requirements (§3.6) the modern structural defences, and it is the fact that a token still protects the simple requests the browser refuses to help you classify.

### 2.3 The Credential-Attachment Rule — Why Cookies Are Ambient

The browser's cookie-attachment decision is made from the *request* and the *cookie's own attributes*, never from the identity or intent of the code that caused the request. The inputs are:

1. **Destination host** — does the cookie's `Domain`/host-only scope match the request's host? (`__Host-` prefixed cookies must have been set with `Secure`, `Path=/` and **no** `Domain`, which is why they cannot be shared with or overwritten from subdomains ✅ — RFC 6265bis §5.4 / MDN.)
2. **Request path** — does the cookie's `Path` scope match?
3. **Scheme** — `Secure` cookies ride HTTPS only.
4. **Whether the request is "same-site" or "cross-site"** for the destination — the `SameSite` rule, §4.
5. **The API in use** — cookies set with `HttpOnly` are unavailable to script, but *are still sent on script-initiated requests*. MDN is explicit: a cookie created with `HttpOnly` *"will still be sent with JavaScript-initiated requests, for example, when calling `XMLHttpRequest.send()` or `fetch()`"* ✅.

Nothing in that list is "did the user mean this?" That is the whole vulnerability, expressed as an omission. And note item 5 carefully, because it is a recurring misunderstanding: **`HttpOnly` is an XSS control, not a CSRF control.** It stops script from *reading* the session cookie, which limits what an XSS attacker can exfiltrate; it does nothing to stop the browser from *sending* it, which is all CSRF needs.

The companion point is that the browser has other ambient credentials too. HTTP Basic/Digest credentials, TLS client certificates, and Windows Integrated Authentication (Kerberos/NTLM) are all attached by the browser under their own rules, and all are CSRF-relevant in intranet and API settings — which is why CWE-352's own *Applicable Platforms* notes that the pattern is not web-form-specific. The token patterns of §5 exist because *no browser rule* can prove intent for a credential that the browser volunteers.

### 2.4 Safe and Unsafe Methods — and the State-Changing GET

The method distinction is load-bearing for two of the six defences, so it is worth stating precisely rather than as folklore.

**Safely-defined** methods (`GET`, `HEAD`, `OPTIONS`, `TRACE`) are the ones HTTP semantics define as retrieval-only: *"a client can use them… without causing any state change on the server"* — and crucially, that safety is a *promise the application must keep*, not a property the framework enforces. The moment a server performs a state change in response to a `GET`, the method's safety is a lie, and every defence keyed to it silently fails:

- `SameSite=Lax` (the browser default in the browsers that implement it) permits the cookie on **cross-site top-level navigations that use a safe method** ✅ (RFC 6265bis §5.6.7.1). A state-changing `GET` is therefore a cookie-bearing, cross-site-triggerable, entirely unblocked operation under `Lax`.
- The custom-header defence (§3.6) does not apply, because a `<form>` or an `<img>` cannot be required to add a header — and requiring one on a `GET` endpoint that links must be able to reach is self-defeating.
- OWASP's guidance is unhedged: *"Do not use GET requests for state changing operations"*, and *"If for any reason you do it, protect those resources against CSRF"* ✅. Meanwhile SameSite's own limitations list names the failure mode as *"the single most common way `SameSite`-based defenses fail in practice"* ✅.

**Why state-changing `GET`s keep happening**, in rough order of frequency: (i) an "action link" in an email — "click here to confirm your email address" — where the designer wanted a link, not a form, and a form POST from an email client is unreliable; (ii) legacy RPC-over-HTTP interfaces where every operation is a `GET` with query parameters; (iii) a `fetch`-based SPA that happens to use `GET` for a mutating endpoint because the payload is small; (iv) an "unsubscribe" or "delete" link designed for one-click use from a newsletter; (v) a framework route that accepts both `GET` and `POST` on the same handler for developer convenience, without the author noticing that the `GET` leg mutates. All five are the same defect, and all five defeat the browser default.

**The test is mechanical and belongs in code review:** for every route handler that writes state — inserts, updates, deletes, sends, charges, enrols, revokes, approves — assert that `GET` and `HEAD` are not among its permitted methods. §12 lists this as the first anti-pattern; §11.2 makes it step one of the worked example.

### 2.5 The Classic Vectors

The vectors are few and old, because the underlying browser capability set is old. Each is worth knowing by shape, because each one defeats a *different* naive defence.

**The auto-submitted hidden form (the canonical `POST` vector).** The attacker's page contains a form whose `action` is the target endpoint, whose fields are the attacker's chosen values, and which is submitted by script on page load (or by a deceptive visible button). This defeats "we only accept `POST`, so the attacker can't make a link". OWASP addresses the misconception directly: *"Only accepting POST requests… The misconception is that since the attacker cannot construct a malicious link, a CSRF attack cannot be executed. Unfortunately, this logic is incorrect."* ✅

**The image tag.** `<img src="https://bank.example/action?param=value" width="0" height="0">`. No script, no form, invisible to the user — and it works even where a page-load submit would be blocked by a CSP that forbids inline script. OWASP's example: *"Or as a 0x0 fake image… Alice wouldn't see anything. However, the browser **will still** submit the request to bank.com without any visual indication that the transfer has taken place."* ✅

**The `fetch`/XHR vector.** Effective only where the request stays "simple" (no custom headers, a simple content type) or where the target's CORS policy is permissive enough to pass the preflight. This is the vector that makes permissive CORS dangerous — §7.3.

**The iframe.** Largely superseded by the image and form vectors for CSRF purposes, but still relevant to clickjacking — which is a *different bug* (§7.4). The confusion between them is common enough that it has its own entry in the comparison table.

**Speculative execution hints.** RFC 6265bis notes that *"Features like `<link rel='prerender'>` can be exploited to create 'same-site' requests without the risk of user detection."* ✅ Modern speculation-rules prefetching has the same property, and OWASP flags that *"prerender/prefetch and other speculative navigation may send `Sec-Fetch-*` values that don't match the final navigation"* ✅ — a genuine caveat on the Fetch Metadata defence, handled in §3.2.

### 2.6 The Variants

**Login CSRF.** The inversion. Instead of acting as the victim, the attacker forces the victim's browser to log in **as the attacker** — by forging a login form submission to the target with the attacker's credentials. The victim then unknowingly operates inside the attacker's account, and anything the victim enters (a card number, an address, a file to sync, a search query, a payment instruction) lands in a store the attacker can later read. OWASP: *"An attacker can use CSRF to obtain the victim's private data via a special form of the attack, known as login CSRF… the attacker can then log back into the account to view this data, along with the victim's activity history on the web application."* ✅ Note the two properties that make login CSRF important: the victim is **unauthenticated** at attack time, so defences that depend on a session (including the synchronizer token, which needs somewhere to store the secret) need care; and the impact is *data exfiltration*, which CSRF is normally said not to achieve. The defence is the same class of control applied to the pre-authentication endpoints — and it is why "CSRF protection" must be switched on for the login handler too, not just post-authentication pages.

**Logout CSRF.** Force the victim to log out. Impact is low in isolation and high in combination: it is a reliable step in a *session-fixation* or phishing chain, it can defeat a "log out before visiting another site" mitigation, and it can interrupt a transaction. Its real significance is defensive: it demonstrates that *any* state change is reachable, including ones the developer considered trivial, and it is usually the cheapest way to prove an application has no CSRF defence at all.

**JSON-endpoint CSRF.** Covered in §2.2 as a vector and again in §6.2 as a CORS problem. The failure mode is an API that requires `Content-Type: application/json` in its *documentation* but not in its *code*: if the server will parse a `text/plain` body as JSON, the request is simple, unpreflighted, and reachable from an attacker page. OWASP names both halves of the remedy — *"the server or API to disallow these simple content types"* ✅, and the companion requirement that the request actually be non-simple: a JSON-content-type requirement only helps if the server *rejects* the wrong content type rather than merely preferring the right one.

**Stored (or persistent) CSRF.** The attack payload is *stored on the target site itself* — in a profile field, a comment, an uploaded document, a support ticket — and fires when a victim views it. OWASP: *"Such vulnerabilities are called 'stored CSRF flaws'.\_ This can be accomplished by simply storing an IMG or IFRAME tag in a field that accepts HTML, or by a more complex cross-site scripting attack. If the attack can store a CSRF attack in the site, the severity of the attack is amplified."* ✅ Note that the richer form of stored CSRF depends on HTML-accepting input, which means stored CSRF frequently *co-exists with* an XSS-class flaw (§7.2) rather than being purely a CSRF issue. Severity rises because the victim is by definition authenticated on the site hosting the payload, so the "will the victim have a live session?" bet is no longer a bet.

**Client-side CSRF.** The modern variant, and the one that breaks the usual mental model. Instead of attacking the *server* with a forged request, the attacker manipulates the application's own JavaScript into issuing a state-changing request it was designed to issue. If the client code derives the request's method, endpoint or parameters from an attacker-controllable input — the URL fragment, `window.name`, `document.referrer`, a `postMessage` payload — then there is no forged request to detect: the request comes from the legitimate page, under the legitimate origin, carrying the legitimate token. OWASP: *"These variants of CSRF are particularly important as they can bypass some of the common anti-CSRF countermeasures like token-based mitigations and SameSite cookies."* ✅ The mitigations are structural rather than request-level: keep attacker-controllable inputs out of request-construction, validate strictly when they must feed it, and prefer a predefined table of safe request shapes that the input merely selects from ✅ (OWASP §"Client-Side CSRF Mitigation Techniques"). The academic source is Khodayari & Pellegrino, USENIX Security 2021 ✅ (cited by OWASP).

**Cross-site search / side-channel variants.** Not CSRF proper — no state change, no forged action — but adjacent and worth naming so the taxonomy stays clean: an attacker infers private data from *whether* a cross-site request was sent, or from response timing/length, without reading the response. The Fetch Metadata specification was written with this class in mind: it exists to let a server reject requests *a priori* because *"Being careful turns out to be hard in some cases ('simple' CSRF), and practically impossible in others (cross-site search, timing attacks, etc.)"* ✅. This is the reason the defence is a *server-side request-classification* mechanism rather than a cookie rule: it generalises beyond CSRF.

### 2.7 A Worked Attack Against a Multi-Step Flow

Multi-step flows are frequently claimed to be CSRF-proof. They are not, and OWASP says so plainly: *"Multi-Step transactions are not an adequate prevention of CSRF. As long as an attacker can predict or deduce each step of the completed transaction, then CSRF is possible."* ✅ The reason is that the "steps" are usually a *UI* construct backed by stateless or predictably-keyed server state; each step is an independent request and each is independently forgeable.

Consider a hypothetical (fictional) retail-banking flow to add a payee and then make a first payment — the shape is generic and deliberately not any real institution's:

- **Step 1.** `POST /payees` with `{name, accountNumber, bankCode}` → creates a *pending* payee, returns `payeeId`.
- **Step 2.** `GET /payees/{payeeId}/confirm` → renders a confirmation screen with the payee details and a `challengeId`.
- **Step 3.** `POST /payees/{payeeId}/confirm` → activates the payee.
- **Step 4.** `POST /payments` with `{payeeId, amount, fromAccount}` → moves money.

Now read it as an attacker. Steps 1 and 3 are simple form posts and forgeable outright if nothing checks provenance. Step 2 is a `GET` — if it has a side effect (marking the challenge viewed, or advancing the flow state), it is *itself* an attack surface. Step 4 requires the attacker to know `payeeId`, which is the only real obstacle — and it is not much of one: identifiers are frequently sequential or returned in the response to step 1, and the attacker's page *can* trigger step 1. If the attacker can observe any consequence of step 1 (for example, by causing step 1 to be sent and then inducing the victim to a page where the new payee is listed and the resulting DOM is measurable, or simply by guessing low-entropy IDs), the chain closes. **The multi-step nature bought the defender nothing**, because intent was never verified at any step; the steps only partitioned the operation.

Where multi-step flows *do* help is entirely different from what is claimed: they increase the number of observation points for **fraud monitoring** (§10.4) and they give a designer more places to insert a **step-up authentication** requirement (§3.7). Both are real controls. Neither is "the flow has multiple steps".

The honest generalisation: a defence is only as strong as the *weakest single request* in the flow. If any one leg is reachable and unauthorised-by-intent, the flow is forgeable; the number of legs is irrelevant.

### 2.8 What the Attacker Can and Cannot Observe

Getting this boundary right is what separates a CSRF risk assessment from a hand-wave, so it is worth stating as a list.

**The attacker can:** cause requests to be sent; cause them to carry the victim's cookies subject to cookie policy; cause a state change on the server; make the victim perform actions the victim's own account permits; learn *indirectly* whether the attack succeeded by observing a side effect on a page the victim later visits (or, in some configurations, by timing and response-length measurement); and choose *when* the attack fires, including in a victim-triggered fashion on a page they are likely to view while authenticated (stored CSRF, §2.6).

**The attacker cannot (in the classical model):** read the response body of a cross-origin request; read the victim's cookies; read the CSRF token from the victim's page (absent XSS, §7.2); learn the session ID; or choose the victim's method (the method is fixed by what the attacker's markup can produce — that is why the simple-request boundary of §2.2 matters).

**Three consequences of that boundary** carry directly into the defence design, and are the reason the defences look the way they do:

1. **Because the attacker cannot read the response, defences that rely on the response being secret are useless.** This is the precise reason the "secret cookie" idea fails — OWASP: *"Remember that all cookies, even the secret ones, will be submitted with every request."* ✅
2. **Because the attacker cannot read the token from the victim's page, a token that the server requires back works** — the attacker can force the request, but cannot supply the value the page would have supplied. This is the whole of §5.
3. **Because the attacker cannot choose the method freely, method-based classification is a real signal** — which is what both `SameSite`'s safety key and Fetch Metadata's `Sec-Fetch-Site`/`Mode`/`Dest` triplet exploit. But the same limitation is why the attacker's inability to add a custom header (without a preflight) is a *defence* (§3.6): the attacker's capability set is exactly the simple-request set, and every modern defence is a way of moving your endpoint's requirements outside it, or of asking the browser to classify the request for you.

---

## 3. The Defence Stack, in Order of Modern Effectiveness

### 3.1 The Ordering, and Why It Is Contested

There is no single authoritative ranking of CSRF defences, and any guide that presents one without qualification is overstating the state of the art. What follows is an *engineering* ordering — by structural strength, by how hard the defence is to bypass *given the current browser model*, and by how catastrophically it fails when misconfigured. It is defensible and it is useful for choosing controls; it is not a standard.

The ordering, with the one-line justification for each position:

| # | Defence | Why it sits here | Its characteristic failure |
|---|---|---|---|
| 1 | **Fetch Metadata (`Sec-Fetch-Site` family)** | Structural: the browser classifies the request from its own state, not from anything the page can influence; the policy is default-deny and lives above the application | Absent-header handling (legacy clients, some proxies) and speculative-loading edge cases |
| 2 | **`SameSite` cookies** | Structural, enforced by the browser before the request arrives; a browser default rather than an application change | The default only blocks *unsafe* methods on cross-site requests (so a state-changing `GET` survives); sibling-subdomain trust; non-universal browser enforcement (§4.6) |
| 3 | **`Origin` / `Referer` validation** | Server-side provenance check on browser-set headers the page cannot forge | `null` and absent values; truncated `Referer`; proxy interference; requiring careful target-origin resolution behind load balancers |
| 4 | **Token patterns (synchronizer, double-submit, signed)** | Strong against the classical model, requires per-request coordination; the *only* defence that also protects endpoints the browser has no reason to classify | Defeated wholesale by XSS; double-submit specifically defeated by cookie injection; per-session tokens weak against token leakage |
| 5 | **Custom-header requirement** | Elegant and cheap for API/SPA surfaces: moves the endpoint outside the simple-request space so a preflight is unavoidable | Does not apply to `<form>` or link-driven endpoints; depends on the CORS preflight actually being refused for unauthorised origins |
| 6 | **Re-authentication / step-up** | Not a request-provenance control at all, but a *human-presence* control; excellent last line for high-value actions | Usability cost; users habituate and approve reflexively; not applicable to machine-initiated endpoints |

Two honest caveats about this table. First, **defences 1–3 are all provenance checks on browser-supplied metadata, and they fail together when the metadata is missing** — which is why §3.8's defence-in-depth argument insists on pairing a metadata check with a token rather than stacking three metadata checks. Second, **the ordering is a statement about bypass difficulty, not about deployment effort**: in practice most applications get most of their benefit from defence 2 (often by changing nothing, since it is now a browser default in some browsers) and defence 4 (because their framework provides it). Ranking a defence first does not mean doing it first.

### 3.2 Defence 1 — Fetch Metadata (`Sec-Fetch-Site` and Family)

**The mechanism.** The browser attaches a small set of request headers describing *how the request was made*, from state the page cannot touch. The specification's stated purpose is precisely the CSRF and cross-site-leak problem: applications need *"to be very careful about the way those endpoints work in order to avoid abuse"*, and it *"would be helpful if servers could make more intelligent decisions about whether or not to respond to a given request based on the way it's made"* ✅ (Fetch Metadata Request Headers, W3C WebAppSec WG Working Draft, 1 April 2025). The illustrative example the spec itself uses is a server refusing a "Transfer all my money" endpoint referenced from an `<img>` tag ✅.

**The headers, their enumerable values, and what a server may conclude.** The four headers are structured fields; servers *"SHOULD ignore this header if it contains an invalid value"* for forward-compatibility ✅ (all four sections). A server may draw the following conclusions — and these are the conclusions the spec supports, not more:

| Header | Enumerable values (per spec) | What a server may conclude |
|---|---|---|
| `Sec-Fetch-Site` | `cross-site`, `same-origin`, `same-site`, `none` | The relationship between the request initiator's origin and the target's origin. This is the primary CSRF signal. |
| `Sec-Fetch-Mode` | `cors`, `navigate`, `no-cors`, `same-origin`, `websocket` | Whether the request is a navigation or a subresource/scripted fetch. `navigate` marks a top-level or nested navigation. |
| `Sec-Fetch-Dest` | the Fetch request destination set — including `document`, `empty` (scripted `fetch()`), `image`, `iframe`, `worker`, `script`, `style`, `object`, `embed`, `audio`, `video`, `font`, `track`, `manifest`, `report`, `xslt`, and others ✅ | What the response is going to be *used for*. A state-changing endpoint whose response is destined to be an `image` is a strong anomaly signal. |
| `Sec-Fetch-User` | a boolean; delivered **only** for navigation requests and **only when its value is `true`** ✅ | Whether a navigation was triggered by user activation. Its presence is the signal; its absence is not evidence of absence of a user. |

`Sec-Fetch-Site` is derivable *from the request* — it is `same-origin` by default, set to `none` for a navigation *"explicitly caused by a user's interaction with the user agent (by typing an address into the user agent directly, for example, or by clicking a bookmark, etc.)"*, and otherwise set to `cross-site`/`same-site` by walking the request's entire URL list ✅. Two specification details carry real operational weight:

- **Redirect chains degrade the classification, and the specification intends that.** *"The header's value will shift in the presence of cross-origin or cross-site redirection in order to mitigate confusion."* The spec's own worked example: a same-origin request redirected to a subdomain becomes `same-site`, then redirected to another registrable domain becomes `cross-site`, and if it then **redirects all the way back to the original origin, the value is still `cross-site`** ✅. So a defence keyed on `same-origin` must expect legitimate redirect chains to present as `cross-site` — a real and under-appreciated compatibility hazard.
- **`Sec-Fetch-User` is not a universal "did the user click?" oracle.** It is *"delivered only for navigation requests, and only when its value is `true`"* ✅, and the spec notes its own scope is limited by design. Absence therefore means "no user-activation navigation evidence", not "no user".

**The default-deny design, and why the absent-header case must be handled explicitly.** The correct policy shape is *allowlist by classification, deny by default*: permit `same-origin`; permit `none` only where a user-driven top-level navigation legitimately reaches the endpoint; treat `cross-site` as untrusted for state-changing actions; and treat `same-site` according to whether you trust sibling subdomains (the SameSite subdomain problem of §4.6 is the same problem, and the same judgement). OWASP's policy sketch says exactly this: *"Treat cross-site as untrusted for state-changing actions. By default, reject non-safe methods (POST / PUT / PATCH / DELETE) when `Sec-Fetch-Site: cross-site`"*; *"Allow `none` for user-driven top-level navigations (bookmarks, typed URLs, explicit form submits) where appropriate"*; and *"Treat `same-site` as allowed only if your threat model trusts sibling subdomains"* ✅.

The absent-header case is the part that is most often got wrong, and the specification's design intent is unambiguous: absence is **not** a licence to proceed. Because the headers are a *browser* feature, an attacker cannot add them for you — but absence also occurs for legitimate legacy clients, so the correct handling is a *deliberate* choice, stated and logged, not a silent fallthrough. OWASP frames the two options and recommends the first: *"Fail-safe (recommended for sensitive endpoints): treat absence as unknown and block the request"*, versus *"Fail-open (compatibility-first): fallback to standard origin verification, CSRF tokens, and/or require additional validation"* ✅. Either way, it must be explicit — the anti-pattern (§12) is a Fetch Metadata check written as `if (secFetchSite === 'cross-site') reject()` with no branch for `undefined`, which means the entire defence evaporates for any client that omits the header. **A default-deny policy written as a deny-list is not a default-deny policy.**

**State of browser support — and do not overstate it.** Deployment reality: the headers reached general availability across the major engines over a multi-year window. Chrome and Edge shipped the family in the Chrome 76–80 range (Chromium/Edge 76–80); Firefox from 90; Safari from 16.4 ✅ (browser-support data surfaced by MDN's compatibility tables, as reproduced in the specification's own "MDN" references — `Sec-Fetch-Mode`/`Sec-Fetch-Site`/`Sec-Fetch-User` from Chrome 76 / Edge 79 / Firefox 90 / Safari 16.4, and `Sec-Fetch-Dest` from Chrome 80 / Firefox 90 / Safari 16.4). OWASP's summary — *"supported in all modern browsers… since March 2023"*, with *"over 98% global coverage"* — is a reasonable working statement ✅ (OWASP cheat sheet, citing caniuse; the figure is OWASP's, and the guide does not independently re-measure it). **What that does not mean:** the residual ~2% is not uniformly distributed. It concentrates in older desktop browsers, embedded/kiosk browsers, some webviews, in-app browsers, and enterprise clients pinned to old engines — and in a retail-banking estate (§10), that residual is a *population of real customers*, not a rounding error. Two further operational caveats from OWASP that a deployment plan must carry: *"Intermediaries (proxies, gateways, load balancers) may remove or modify `Origin` and `Sec-*` headers"*, and speculative navigation may produce classifications that do not match the eventual navigation ✅. The roll-out advice that follows is standard and correct: log-only first, monitor user-agent coverage to size the fallback population, and document every endpoint exempted for intentional cross-origin use ✅.

**Failure modes, collected.** (i) Deny-list instead of default-deny → defence vanishes on absent headers. (ii) `same-site` treated as trusted without a subdomain threat model. (iii) `Vary: Sec-Fetch-Site` omitted on any cached response whose content depends on the classification — OWASP notes the `Vary` header *"does not impact CSRF defenses in any way"* but is needed to stop caches reusing a response generated for a different context ✅. (iv) Redirect chains presenting as `cross-site` for legitimate flows, generating either outages or (worse) exemptions. (v) Treated as sufficient alone: it is a *classification* control, so it cannot help with a request the browser classifies as same-origin — including, decisively, an XSS-forged request (§7.2).

### 3.3 Defence 2 — SameSite Cookies

**The mechanism.** `SameSite` is a cookie attribute; the browser decides whether to attach the cookie based on whether the request is same-site or cross-site, and how. The specification's operative language is terse and worth quoting because so much secondary commentary paraphrases it loosely: *"If the 'SameSite' attribute's value is 'Strict', the cookie will only be sent along with 'same-site' requests. If the value is 'Lax', the cookie will be sent with same-site requests, and with 'cross-site' top-level navigations, as described in Section 5.6.7.1. If the value is 'None', the cookie will be sent with same-site and cross-site requests."* ✅ (RFC 6265bis §4.1.2.7).

**Correct configuration, in one table.** Full semantics — including the browser default history and what `Lax` actually permits — are §4. Here, the configuration guidance:

| Cookie | Correct `SameSite` | Reasoning |
|---|---|---|
| Session / auth cookie | `Lax` minimum; `Strict` where the app tolerates inbound cross-site links landing logged-out; add `Secure`, `HttpOnly`, and the `__Host-` prefix | `Lax` blocks the unsafe-method cross-site cases that carry the classic attack; `__Host-` prevents a subdomain from overwriting it |
| CSRF token cookie (double-submit variant) | `Lax` or `Strict`; **not** `None`; **not** `HttpOnly` (the client must read it) | `None` re-opens the very exposure the token exists to close |
| Genuinely third-party / embed cookies | `None; Secure` — **only** for those, and only where the embed truly needs it | Global `SameSite=None` is anti-pattern #2 (§12) |
| Cookies on a shared registrable domain you do not fully control | `Strict`, `__Host-`, and treat the sibling-subdomain threat as live | See the subdomain problem below |

**Failure modes.** (i) **The state-changing `GET`** — the dominant real-world failure; §2.4. (ii) **Registrable-domain, not origin, scoping.** `SameSite` is evaluated against *site*, not origin, so a cookie set on `app.example.com` is "same-site" for a request from `anything.example.com`. OWASP states the consequence bluntly: *"If your application shares a registrable domain with content you do not fully control… a vulnerability or malicious actor on any of those sibling hosts can issue requests that the browser will treat as same-site. This also amplifies the impact of subdomain takeovers."* ✅ (iii) **Non-universal enforcement** — see §4.6, which is the guide's most important correction to received wisdom. (iv) **Top-level navigation and window-opening tricks** — an attacker who can induce a top-level navigation or open a window to your site can generate a request the browser treats as same-site; `Strict` blocks most of these at the cost of breaking legitimate inbound links ✅. (v) **Client-side CSRF is entirely unaffected**, because that attack never leaves the origin ✅.

**When `SameSite` alone may be defensible** — and this is a narrow exception, not a policy. OWASP lists the conditions: no shared registrable domain with anything you do not control; no safe-method endpoint performing a state change; the session cookie set `Strict` (or `Lax` *plus* `__Host-` plus a strict audit of every `GET` handler); Origin/Referer verification in place as defence in depth; and a conscious decision to exclude users on browsers that do not enforce `SameSite` ✅. Note that the last condition is where the §4.6 correction bites hardest: any estate sized against "all modern browsers enforce Lax by default" is underestimated for the browsers that do not.

### 3.4 Defence 3 — Origin and Referer Validation

**The mechanism.** Two request headers the page cannot set — they are on the browser's *forbidden header* list, *"meaning that only the browser can set them"* ✅ — carry the origin (or URL) of the page that caused the request. Verify that the source matches your own origin, and reject if not.

**What to check, exactly.** The check has a source half and a target half, and both have traps.

*Source half — the ordering and the fallbacks:* if `Origin` is present, verify it matches; the OWASP note on this is that *"Unlike the referer, the Origin header will be present in HTTP requests that originate from an HTTPS URL"* ✅. If `Origin` is **absent**, fall back to the hostname in `Referer` ✅. If **neither** is present, *"you can either accept or block the request. We recommend **blocking**."* ✅ — with the intermediate, pragmatic option of logging first and blocking once you have confidence, which is the same log-then-enforce pattern §3.2 recommends for Fetch Metadata.

*Match strength:* do the comparison against the **entire origin, including scheme and port**, with an anchored compare. The failure mode is a naive substring or suffix test: *"if your site is `example.org` make sure `example.org.attacker.com` does not pass your origin check"* ✅. This is a textbook CWE-346 (Origin Validation Error) implementation bug — and CWE-346 is, per MITRE, one of the component weaknesses required for CWE-352 to arise ✅ — so a sloppy Origin check is literally an instance of the weakness this defence exists to close.

*Target half, behind a proxy:* resolving "what is my own origin?" is harder than it looks. Do not trust the `Host` header if a proxy rewrites it; either configure the target origin explicitly server-side (*"the most secure approach as it's defined server side, so it is a trusted value"* ✅) or use `X-Forwarded-Host` to recover the client-facing host — while recognising that any appended proxy header is itself an input that a misconfigured edge can let a client spoof, which is why the configured-value option is preferred ✅.

**The `null` and absent cases.** These are the substance of this defence's weakness, and they need to be listed as cases rather than as a caveat, because they are what an attacker will probe:

| Case | `Origin` | `Referer` | Correct handling |
|---|---|---|---|
| Ordinary same-origin `POST`/`PUT`/`DELETE` | Sent (cross-origin always; same-origin typically for non-`GET`) | Usually sent, oriented to origin under modern referrer policy | Verify; allow |
| Cross-site `302`/`303` redirect chain | **May be omitted on the redirected request** ✅ (OWASP cites this as a case where `Origin` is *not* included) | Policy-dependent | Do not silently allow — this is the case that tempts teams into accepting `null` |
| Privacy contexts (some browsers/contexts) | Set to `null` ✅ (OWASP: *"there are some privacy contexts where Origin is set to 'null'"*) | Suppressed by policy | **Reject.** Anti-pattern #6 (§12) |
| Sandboxed iframe, `data:` / `file:` / some `blob:` contexts | `null` | Absent or `null` | **Reject** |
| Navigation from a bookmark, typed URL, or non-web context | May be absent | Absent | Legitimate. But legitimate *navigations* should be `GET`s — so if this reaches a state-changing endpoint, the endpoint is the bug, not the header |
| Referrer policy suppressing the referrer, or an HTTPS→HTTP downgrade | — | Absent / stripped | Reject, or fall back to another control |
| Intermediary (proxy, LB, privacy filter, embedded device) stripping the headers | Absent | Absent | Reject, and treat the strip as an infrastructure defect to fix — not as a reason to weaken policy |

**The `null`-acceptance temptation, and why the guide's position differs from common practice.** Accepting `null` origin is a widespread accommodation; OWASP describes it neutrally, noting the practical motivation (a small percentage of legitimate traffic) and that *"Please note that, attackers can exploit this"* ✅. This guide's position: **do not accept `null` for state-changing endpoints.** The whole purpose of the check is provenance, and `null` is the value that carries the *least* provenance — it is what an attacker's isolating context produces and what legitimate traffic produces only in edge cases. If your traffic data shows non-trivial `null` volume on a state-changing endpoint, the correct response is to find out why and fix that, because whatever is producing them has *less* provenance than a normal browser, not more. Accept `null` only on `GET` endpoints that are genuinely side-effect-free, and log the rate.

### 3.5 Defence 4 — The Token Patterns

**The mechanism.** The server issues a secret that is bound to the user's session, places it in the rendered page (a hidden form field, a response payload, a readable cookie, or a `meta` tag for the client to pick up), and requires it back on each state-changing request. An attacker who can force a cross-site request cannot supply the value, because reading it would require either reading a cross-origin response (blocked by SOP) or reading the victim's page (blocked absent XSS, §7.2).

That last clause is the honest boundary of the entire defence, and OWASP states it as a warning at the top of its guidance: **"Remember that Cross-Site Scripting (XSS) can defeat all CSRF mitigation techniques!"** ✅ Token patterns in depth — synchronizer, double-submit, signed/HMAC, storage, stateless architectures, and the per-session/per-request trade — are §5. The summary a reader needs here for defence selection:

| Pattern | Strength | Fatal-in-practice weakness |
|---|---|---|
| Synchronizer token (per-session) | Strong against classical CSRF; server compares to stored copy | Defeated by XSS; needs server-side state; a leaked token is replayable for the session's life |
| Synchronizer token (per-request) | Narrower replay window | Usability cost (back button, concurrent tabs); still defeated by XSS; must handle multi-tab and retry correctly |
| Naive double-submit cookie | Stateless, easy, scales across services | **Documented bypass:** if an attacker can write cookies on the target registrable domain — a vulnerable sibling subdomain, DNS takeover, or plaintext-HTTP cookie injection on a non-`__Host-` cookie — they can inject a matching pair and forge a valid request ✅ (OWASP explicitly *discourages* the naive form and marks it "for reference only") |
| Signed / HMAC-bound double-submit | Fixes the injection bypass: the server verifies it *issued* the token, not merely that two client-supplied values agree ✅ | Still defeated by XSS; key management becomes a real dependency |

The single most useful sentence for a design review is OWASP's, and it should be quoted in the ADR: the signed variant *"explicitly ties tokens to the user's authenticated session (e.g., session ID)"* and *"Simply signing tokens without session binding provides minimal protection and remains vulnerable to cookie injection attacks"* ✅. In other words: **signing without session-binding is not a fix**, and a team that "we implemented HMAC double-submit" without binding to the session has closed half the hole.

### 3.6 Defence 5 — Custom Headers and the Preflight Consequence

**The mechanism.** Require a header the attacker's markup cannot produce — `X-CSRF-Token`, or any arbitrary non-standard name — on state-changing endpoints. Because adding a custom header makes a request non-simple, a browser cannot send it cross-site without a successful CORS preflight. OWASP: *"This defense relies on the CORS preflight mechanism… All modern browsers designate requests with custom headers as 'to be preflighted'. When the API verifies that the custom header is there, you know that the request must have been preflighted if it came from a browser."* ✅ Note what is *not* required: *"No token is needed for this approach… no server state is introduced to track tokens"* ✅ — the header's presence, not its value, is the signal. (In practice most implementations put the CSRF token *in* the header, which gives both defences at once.)

**Why the preflight makes this work — and when it does not.** The preflight gives the target server a veto opportunity before the real request is sent: the browser asks with `OPTIONS`, and if the server does not answer with an `Access-Control-Allow-Origin` covering the attacker's origin, the browser never sends the request. Therefore the defence's integrity rests entirely on the server *not* permitting the attacker's origin. OWASP's warning is the exact failure mode: a CORS configuration that allows all subdomains by regex is dangerous, because *"If an attacker is able to take over a subdomain (not uncommon with cloud services) your CORS configuration would allow them to bypass the same origin policy and forge a request with your custom header."* ✅ And it requires that credentials be handled deliberately: *"Cookies are not set on cross-origin requests (CORS) by default. To enable cookies on an API, you will set `Access-Control-Allow-Credentials=true`. The browser will reject any response that includes `Access-Control-Allow-Origin=*` if credentials are allowed."* ✅

**The scope limit.** This defence simply does not exist for endpoints that must be reachable by an HTML form, an image, or a link. *"When a `<form>` tag is used to submit data, it sends a 'simple' request that browsers do not designate as 'to be preflighted'. These 'simple' requests introduce risk of CSRF because browsers permit them to be sent to any origin. If your application uses `<form>` tags to submit data anywhere in your client, you will still need to protect them with alternate approaches."* ✅ So the practical rule is: custom-header-plus-token for the JSON/SPA surface; the token pattern for the form surface; and never let the two share a handler without both. A resource that can be reached both ways is only as protected as its weakest route — a theme the worked example (§11) runs into directly.

### 3.7 Defence 6 — Re-authentication and Step-Up

**The mechanism.** For the highest-value actions, require the human to prove presence again at the moment of the action: re-enter the password, provide an OTP, approve on a registered device, or satisfy a biometric prompt. CWE-352 lists this as a mitigation in its own right: *"Identify especially dangerous operations. When the user performs a dangerous operation, send a separate confirmation request to ensure that the user intended to perform that operation."* ✅ OWASP likewise recommends *"user interaction based protection for highly sensitive operations"* ✅.

**Why it belongs in the stack but not at the top.** It is the only defence that addresses *intent* directly rather than inferring it from provenance — which makes it genuinely complementary, not merely layered. But it is not a CSRF defence in the sense the others are: it does not stop the forged request, it stops the *effect* by requiring a step the attacker cannot automate. Its practical weaknesses are equally clear: the usability cost limits it to a small number of actions; and it is vulnerable to the fact that users habituate and approve reflexively, which is an attack in its own right and one that lives outside this guide's scope. CWE-352's own note is the honest one: this mitigation *"can be bypassed using XSS"* ✅.

**Where it earns its place:** high-value irreversible actions (a payment to a new payee, a change of the registered mobile number, a password or 2FA reset, a change of the account's beneficiary details, a large transfer). These are exactly the actions where the *combination* of a CSRF flaw and a fast, automated attack is worst, and where an extra prompt is worth the friction. §11.3 places it by endpoint class on precisely this basis.

### 3.8 The Defence-in-Depth Argument, Stated Honestly

"Defence in depth" is the most abused phrase in security writing, so state the argument for this vulnerability class as a falsifiable claim rather than a slogan. The claim is:

> **For CSRF specifically, defence in depth is not the practice of deploying several provenance checks. It is the practice of deploying at least one browser-enforced control and at least one application-enforced control, because they fail for different reasons.**

The justification is that the six defences in this section partition into two groups with *disjoint* failure modes:

- **Browser-enforced** (Fetch Metadata, `SameSite`, the preflight consequence): they hold without any application logic being correct, and they cannot be manipulated by the attacker's page. They fail when the browser does not implement them, when the request is one the browser legitimately classifies as same-origin, when metadata is stripped in transit, or when a cookie attribute is misconfigured.
- **Application-enforced** (tokens, Origin/Referer checks, step-up): they work on any client, including the 2% with no Fetch Metadata and the browsers that do not default to `Lax`. They fail when the application logic is wrong, when the token is leaked or not bound, when an exemption list quietly widens, or when a single endpoint is omitted.

Neither group dominates the other, and the failure of one does not imply the failure of the other. That is what makes the layering real rather than decorative — and it is why §3.1's ordering is explicitly *not* a reason to deploy only the top entry. Two groups, both needed, is the whole argument. What does **not** qualify as defence in depth: a deny-list Fetch Metadata check plus an Origin check that accepts `null` plus a permissive CORS policy, all of which fail together for an attacker in a sandboxed context.

One boundary condition on the argument, stated plainly because it is where layering genuinely stops. **Against a compromised origin, the stack collapses.** XSS (§7.2) defeats every application-enforced control and can read the token, the session, and anything else the page can reach; an attacker who can execute script in your origin does not need CSRF at all, having something strictly better. Defence in depth protects against *implementations being wrong*; it does not protect against *the origin being lost*. That is why fixing XSS is not an alternative to CSRF defence, and CSRF defence is not a substitute for fixing XSS — the sections that follow have to say both.

### 3.9 The Combination a Modern Application Should Actually Deploy

Not a recommendation of one control, and not a list of all six. The following is what a web application with state-changing endpoints, a cookie-based session, and a requirement to work in ordinary browsers should actually have in production — stated as an obligation per surface, because the answer genuinely differs by surface.

| Surface | Must have | Should have | The reason it differs |
|---|---|---|---|
| **Form-driven HTML endpoints** (server-rendered pages, `<form method="POST">`) | Synchronizer token (framework-provided) on every unsafe method; session cookie `SameSite=Lax`+`Secure`+`HttpOnly`+`__Host-` | Fetch Metadata default-deny; Origin verification as fallback for absent metadata | A form cannot be required to carry a custom header, so §3.6 is unavailable and the token carries the load |
| **JSON / API endpoints called by your own SPA** | Custom-header requirement **and** strict CORS allowlist with `Access-Control-Allow-Credentials`; a token in that header if cookies are used for auth | Fetch Metadata default-deny; reject non-JSON content types outright | The preflight is the structural control; the token covers clients where the preflight alone was misconfigured |
| **Endpoints that must accept cross-site calls by design** (webhooks, third-party integrations, OAuth callbacks) | Explicit per-endpoint exemption in a documented list; a signature or an out-of-band secret if the caller is a server; `state`/nonce validation if it is a redirect leg | Logging on the exemption boundary; rate limits; independent authentication of the caller | These sit *outside* the global deny policy; that exemption is the risk, so it must be enumerated and owned |
| **High-value, irreversible actions** (payment to a new payee, credential or contact-detail change) | Everything above for its surface, **plus** step-up re-authentication | Independent transaction authorisation; fraud-rule evaluation; strong notification | The cost of a false positive is low and the cost of a false negative is not — §10.2 |
| **Pre-authentication endpoints** (login, password reset request, registration) | CSRF protection on the login POST (login CSRF, §2.6); `SameSite` on any pre-auth cookie | Origin verification; rate limits | The victim has no session yet, so session-bound tokens need care — this is the surface most often left unprotected |

**And the two universal rules that outrank all six defences**, because they are what make the defences testable and durable:

1. **No state change on a safe method.** Every state-changing endpoint is `POST`/`PUT`/`PATCH`/`DELETE`, without exception, and this is asserted by an automated test rather than by convention (§9.5). This single rule is what allows the browser's `SameSite` default to do real work.
2. **No endpoint is exempt by accident.** Every CSRF exemption in the codebase is an explicit, named, reviewed entry with an owner — because, as §12's list shows, the most common way a mature application becomes vulnerable again is not a missing control but an exemption that was added to fix a bug and never removed.

---

## 4. The Browser-Level Change and SameSite in Depth

### 4.1 What Changed, Precisely — Spec Value, First Browser, Version, Year

This is the pivotal fact of the topic and the one most often stated loosely, so it is worth separating into its four components and giving each a source.

**Which specification value it changed FROM and TO.** Two layers, and conflating them is the common error.

- *The original specification had no such attribute.* RFC 6265 (April 2011) — the base Cookie specification — **does not define `SameSite` at all.** Under RFC 6265, a cookie was attached to any request whose host, path and scheme matched its scope, with no notion of the initiating site. The honest way to state the "from" is therefore: **from no `SameSite` attribute and no cross-site restriction whatsoever** (effectively every request), not "from `SameSite=None`". `None` is a value defined by the *later* specification.
- *The current specification defines a `Default` state and a default enforcement mode.* In draft-ietf-httpbis-rfc6265bis, when a cookie arrives with no `SameSite` attribute — or with a value that is not one of `Strict`, `Lax`, `None` — the cookie's `same-site-flag` is set to **`Default`** ✅ (RFC 6265bis §5.6.7 step 1 sets `enforcement` to `"Default"`; §5.7 step 17 sets the cookie's `same-site-flag` to `"Default"` when no valid value is present). What `Default` *means* is then defined by the default enforcement mode: *"If the 'SameSite' attribute's value is something other than these three known keywords, the attribute's value will be subject to a **default enforcement mode that is equivalent to 'Lax'**. If a user agent uses 'Lax-allowing-unsafe' enforcement… then this default enforcement mode will instead be equivalent to 'Lax-allowing-unsafe'."* ✅ (RFC 6265bis §4.1.2.7).

So, stated exactly: **the change was FROM `same-site-flag = "Default"` being treated as unrestricted (send on same-site and cross-site requests alike — the RFC 6265 behaviour) TO `"Default"` being subject to an enforcement mode equivalent to `Lax`** (or to `Lax-allowing-unsafe`, where the user agent applies that transitional mode — §4.3). Note what this means for the second layer: the specification does **not** mandate `Lax`, and it does not mandate a *value*; it defines a *default enforcement mode*, and permits a less restrictive one. That distinction is precisely why the browser-by-browser reality in §4.6 diverges from the single sentence everyone quotes.

**Which browser shipped it FIRST, in WHICH VERSION, in WHICH YEAR.** **Google Chrome, in Chrome 80, in 2020.** ✅ Chrome 80 Stable was released **4 February 2020**; the enforcement was *not* in that initial stable build, and began rolling out to a limited stable population in the **week of 17 February 2020** ✅ (Chromium, "SameSite Updates" launch timeline). Chrome's own pre-announcement of the rule is earlier and is worth quoting for its completeness, because it is the sentence the rollout then softened: *"Starting in Chrome 80, cookies that do not specify a SameSite attribute will be treated as if they were SameSite=Lax with the additional behavior that they will still be included in POST requests to ease the transition for existing sites."* ✅ (Chromium, 26 September 2019 update.) That italicised proviso is the Lax+POST mitigation — §4.3.

**The companion half, shipped separately.** Chrome also shipped *`SameSite=None` requires `Secure`* as a distinct feature, not part of the same change. It was *"available as of Chrome 76 by enabling the `cookies-without-same-site-must-be-secure` flag"*, and rolled out to stable *"gradually… starting July 14, 2020"* ✅ (Chrome Platform Status, feature 5633521622188032, "Reject insecure SameSite=None cookies"). The practical consequence: a cookie that explicitly requests `SameSite=None` **without** `Secure` is rejected outright — and the specification encodes the same rule, aborting the storage algorithm *"unless the cookie's secure-only-flag is true"* when `same-site-flag` is `None` ✅ (RFC 6265bis §5.7 step 19).

### 4.2 How the Rollout Actually Proceeded

The rollout is a rare and instructive case of a security default being *shipped, rolled back, and resumed*, and the shape of it matters for anyone arguing "the browser handles this now". The chronology, from the Chromium project's own tracking page ✅ (chromium.org/updates/same-site, page last updated 18 March 2021):

| Date | Event |
|---|---|
| Early October 2019 | Experimental SameSite-by-default and None-requires-Secure behaviour to 50% of Chrome Canary/Dev users (78+); domain-joined and enterprise-registered devices **excluded** |
| 31 October 2019 | Chrome 79 Beta; experiment extended to 50% of Beta users, now **including** domain-joined/enterprise devices; enterprise policies made available |
| 10 December 2019 | Chrome 79 Stable released — **does not** receive the new behaviour |
| 19 December 2019 | Chrome 80 Beta; experimental behaviour still 50% |
| **4 February 2020** | **Chrome 80 Stable released — enforcement NOT included in this initial stable rollout** |
| Week of 17 February 2020 | Enforcement begins rolling out to Chrome 80 Stable for an **initial limited population** (excluding the US Presidents' Day holiday); monitoring and gradual increase planned |
| 2 March 2020 | Rollout population increased beyond the initial group; still a limited global fraction |
| 9 March 2020 | Population increased again; still a fraction of the stable population |
| **3 April 2020** | **Temporarily rolled back on Chrome 80 Stable**, explicitly *"In light of the extraordinary global circumstances due to COVID-19"*; non-stable channels continue at 50% |
| 28 May 2020 | Plan announced to resume coinciding with Chrome 84's stable release, with enforcement enabled for Chrome 80+ |
| **14 July 2020** | **Enforcement resumes**, gradual, for stable users on Chrome 80, 81, 83 and the newly released 84 |
| 28 July 2020 | Rollout population increased to a fraction of the overall Chrome 80+ stable population |
| **11 August 2020** | Target population increased to **100% of users on Chrome Stable 80+**, ramping up gradually; users receive it on restart |
| 8 January 2021 | Android WebView: the modern SameSite behaviour (Lax by default, None requires Secure, schemeful same-site) enabled by default for apps **targeting Android 12 and newer**; existing apps unaffected until they retarget |
| 18 March 2021 | The `#same-site-by-default-cookies` and `#cookies-without-same-site-must-be-secure` flags removed from `chrome://flags` as of **Chrome 91**; the `--disable-features=…` command-line flag to be removed in **Chrome 94** |

Two facts from that table deserve extraction because they are routinely missing from summaries:

1. **The enterprise escape hatch existed and had a defined life.** The policies `LegacySameSiteCookieBehaviorEnabledForDomainList` and `LegacySameSiteCookieBehaviorEnabled` reverted affected Chrome/ChromeOS instances to the legacy behaviour; their stated lifetime was originally "at least 12 months after the release of Chrome 80 stable", edited (29 May 2020) to **"until at least July 14, 2021"** ✅. An estate that relied on them had a defined expiry.
2. **The change is not uniform even inside one vendor's product line.** Chrome on iOS is described as *not* affected by the SameSite behaviour change ✅, and Android WebView lagged the browser by more than a year, gated on apps *retargeting* to Android 12 ✅. So "Chrome defaults to Lax" is true of Chrome-the-browser from mid-2020 onward, and false of every WebView-embedded client whose app never retargeted.

### 4.3 The "Lax+POST" Two-Minute Mitigation, and Whether It Still Exists

Chrome deliberately carved an exception out of its own new default to avoid breaking single-sign-on and other POST-based redirect flows, and the exception is subtle enough that it is worth stating in full.

**What the mitigation does.** A cookie that arrived with **no explicit `SameSite` attribute** is treated as `Lax` — *except* that for a limited window after the cookie is created, it is still attached to **top-level cross-site POST** requests. Chrome's own debugging guidance describes the observable symptom precisely: *"For flows involving POST requests, if a short delay (< 2 minutes) works but a long delay (> 2 minutes) does not work, you will also need to add `SameSite=None` and `Secure`…"* ✅. So the window is two minutes, measured from the cookie's creation. Chromium was explicit that it was provisional: *"the 2-minute window for 'Lax+POST' **is a temporary intervention and will be removed at some point in the future** (some time after the Stable launch of Chrome 80), at which point cookies involved in these flows will require `SameSite=None` and `Secure` even if under 2 minutes."* ✅ A testing flag to disable it and see the end state (`--enable-features=SameSiteDefaultChecksMethodRigorously`) was made available from Canary **80.0.3975.0**, per the 21 November 2019 update ✅.

**Why it exists, in the spec's own words.** This is not merely a Chrome hack: it was taken into the specification as a named enforcement mode. RFC 6265bis §5.6.7.2 defines **"Lax-Allowing-Unsafe" enforcement**, which *"allows cookies to be sent with a cross-site HTTP request if and only if it is a top-level request, **regardless of request method**"* — i.e. it *"waives the requirement for the HTTP request's method to be 'safe'"* ✅. The specification constrains it tightly and honestly: it is *not* a SameSite value; a user agent **MAY** apply it *only* to cookies that did not explicitly specify a `SameSite` attribute; and a user agent that applies it **SHOULD** restrict it to recently created cookies, with the rationale given from practice — *"Deployment experience has shown a cookie age of 2 minutes or less to be a reasonable limit."* ✅ The spec's own security consideration for the mode is the cleanest possible summary of its risk: *"As a more permissive variant of 'Lax' mode, 'Lax-allowing-unsafe' mode necessarily provides fewer protections against CSRF. Ultimately, the provision of such an enforcement mode should be seen as a **temporary, transitional measure** to ease adoption of 'Lax' enforcement by default."* ✅ (§8.8.6.) The motivating case the spec gives is a login flow whose final leg is a cross-site top-level POST carrying freshly set transactional state, where `Lax` would break the flow unrecoverably ✅.

**Does it still exist?** Two honest answers, and the guide will not overstate either.

- **In the specification:** yes — as of revision 22 (December 2025, the revision read for this guide) §5.6.7.2 still defines the mode and §4.1.2.7 still permits a user agent to use it as the default enforcement mode ✅. It has not been removed from the spec text.
- **In Chrome, in 2026:** ⚠ **not verified this pass.** Chromium's published intent was removal "at some point in the future" and the page's last update on the point was 18 March 2021. This guide found no primary source confirming that the two-minute carve-out has been retired in current Chrome builds, and none confirming that it is still active. Treat it as *possibly present* and design as though it is — see §14.

**Why it matters for CSRF even if you never rely on it.** The mitigation creates a narrow window in which the browser's structural defence is *off* for unlabelled cookies. An attacker who can cause the target origin to set a fresh unlabelled cookie and then immediately fire a cross-site top-level POST — a login CSRF chain is the obvious route — is operating inside that window. The defensive conclusion is not to attack the mitigation but to make it irrelevant: **always label your cookies explicitly.** A cookie that carries `SameSite=Lax` explicitly — rather than relying on the default — is not eligible for `Lax-allowing-unsafe` enforcement, because the spec permits the mode *only* for cookies whose `same-site-flag` was `"Default"` ✅. Explicit labelling is the single cheapest hardening step in this entire guide.

### 4.4 What `SameSite=Lax` Actually Permits

The crucial nuance, stated as an exhaustive list, because the common summary ("Lax blocks cross-site requests") is wrong in exactly the way that matters:

**Under `Lax`, the cookie IS attached to:**
- **Every same-site request**, whatever the method — including same-site `POST`, `PUT`, `DELETE`, and same-site subresource requests.
- **Cross-site top-level navigations that use a safe method** — a link click from another site, a `window.location` assignment, a form `GET`. The specification: `Lax` *"carves out an exception which sends same-site cookies along with cross-site requests if and only if they are top-level navigations which use a 'safe' (in the [HTTP] sense) HTTP method"* ✅ (§5.6.7.1). MDN states the same rule from the browser side: `Lax` permits cross-site requests that are *"a top-level navigation"* **and** *"uses a safe method: in particular, this excludes `POST`, `PUT`, and `DELETE`"* ✅.

**Under `Lax`, the cookie is NOT attached to:**
- Cross-site subresource requests — `fetch()`, `<img>`, `<script>`, `<iframe>` navigations ✅ (MDN: this *"would exclude, for example, requests made using the `fetch()` API, or requests for subresources from `<img>` or `<script>` elements, or navigations inside `<iframe>` elements"*).
- Cross-site top-level navigations using an **unsafe** method — i.e. a cross-site form `POST`, subject to the Lax+POST carve-out of §4.3.

**The two consequences that must be carried into design, because they are both routinely missed:**

1. **A state-changing `GET` remains exploitable under `Lax`.** It is a cross-site top-level navigation (`<img>` or link or `window.location`) using a safe method, which is precisely the permitted case. OWASP names this as *"the single most common way `SameSite`-based defenses fail in practice"* ✅. There is no browser default that can save an application from a mutating `GET`; only the application can.
2. **`Lax` is explicitly not a complete CSRF defence, by the specification's own admission.** The spec's §5.6.7.1 lists why, in two numbered reasons: *"Attackers can still pop up new windows or trigger top-level navigations in order to create a 'same-site' request… which is only a speedbump along the road to exploitation"*, and *"Features like `<link rel='prerender'>` can be exploited to create 'same-site' requests without the risk of user detection."* ✅ The specification then states the design intent directly: *"Lax enforcement provides reasonable defense in depth against CSRF attacks that rely on unsafe HTTP methods (like POST), but does not offer a robust defense against CSRF as a general category of attack."* ✅

A third property is easy to miss because it is about cookie *creation* rather than delivery: the attribute governs both. *"Cookies which assert 'SameSite=Lax' or 'SameSite=Strict' cannot be set in responses to cross-site subresource requests, or cross-site nested navigations. They can be set along with any top-level navigation, cross-site or otherwise."* ✅ (§4.1.2.7.) The storage-side rule sits at §5.7 step 18 and aborts storage of a non-`None` cookie received from a cross-site non-top-level request. The security reading: **`SameSite` also closes a cookie-*setting* attack surface** — a cross-site subresource cannot plant a `Lax`/`Strict` cookie on your origin — which is one of the mechanisms limiting the cookie-injection attack on the double-submit pattern (§5.4).

### 4.5 The Three Values in Full, and the `None`+`Secure` Requirement

| Value | Attached to | Practical effect | Cost |
|---|---|---|---|
| **`Strict`** | Same-site requests only ✅ | Strongest. A cross-site top-level navigation arrives *without* the cookie, so an inbound link lands logged-out | Inbound links from email, chat, search results and partner sites all land logged-out. Usable for sensitive sub-applications and for cookie prefixes on critical cookies, rarely for a consumer session |
| **`Lax`** | Same-site requests, **plus** cross-site top-level navigations using a safe method ✅ | Blocks the classic cross-site form POST and subresource vectors while keeping ordinary inbound links working | The state-changing `GET` survives; subdomain/site scoping applies; spec explicitly calls it defence in depth, not robust defence (§4.4) |
| **`None`** | Same-site **and** cross-site requests ✅ — **and requires `Secure`**, otherwise the cookie is rejected outright ✅ | The explicit opt-out of cross-site restriction. Needed for genuinely third-party cookies (embeds, cross-site SSO) | Removes the browser-level CSRF defence for that cookie entirely. Must be a deliberate, justified, narrowly scoped decision |
| **Absent / invalid value** | Determined by the user agent's **default enforcement mode** — `Lax`, or `Lax-allowing-unsafe` where that mode is used ✅ | Browser-dependent, and *not* the same in every browser (§4.6). If the enforcement mode is `Lax-allowing-unsafe`, unlabelled cookies are also sent on cross-site top-level **POST** within the recent-creation window (§4.3) | You do not know what your own cookie means. This is the argument for explicit labelling, above all others |

**Schemeful same-site.** The "site" used for the same-site test includes the **scheme** — MDN's definition of the cross-site comparison is *"requests originating from a different site, **including the scheme**, from the site that set the cookie"* ✅. So `http://app.example` and `https://app.example` are **not** the same site. Two consequences: (i) an HTTP→HTTPS upgrade mid-flow can strip cookies in ways that look like a CSRF failure but are a scheme mismatch, which is a common cause of "our `SameSite` change broke login"; (ii) Chromium grouped schemeful same-site with the SameSite-by-default behaviour under the heading "modern SameSite behaviour" and gated both behind the same Android-WebView retargeting condition ✅ — so an estate that audited only the Lax-by-default change may have missed that scheme-sensitivity arrived with it.

**Prefixes, because they interact with `SameSite`'s scoping weakness.** `__Secure-` requires the cookie to be set with `Secure`; `__Host-` requires `Secure`, `Path=/`, **and no `Domain` attribute** ✅ (MDN, and RFC 6265bis §5.7 steps 20–21 enforce both). The `__Host-` constraint is the specific answer to the subdomain problem: a cookie with no `Domain` attribute is host-only, so a sibling subdomain cannot set or overwrite it. And the interaction is security-relevant at cookie *creation*: RFC 6265bis notes the path comparison is *"not symmetric, ensuring only that a newly-created, non-secure cookie does not overlay an existing secure cookie, providing some mitigation against cookie-fixing attacks"* ✅ — the mechanism that makes `__Host-` a defence rather than just a naming convention.

### 4.6 What Firefox and Safari Actually Do — the Correction

This subsection exists because a claim repeated in the OWASP cheat sheet, in most vendor blogs, and in nearly every secondary guide is **out of date or wrong**, and a guide that repeats it gives its readers a false sense of coverage. The claim is: *"Chrome implemented `SameSite=Lax` as the default behavior in 2020, and Firefox and Edge have followed suit."* ✅ — that is OWASP's sentence, as published. It is accurate about Chrome, accurate about Edge (Chromium-based Edge inherits Chromium's behaviour), and **not accurate about Firefox today.**

**The Firefox record, from Mozilla's own tracker.** Firefox attempted the change, shipped it, and then **backed it out**:

- **The attempt began in 2020.** Bug 1622091 is titled *"[Experiment] Staged Rollout: Beta rollout of SameSite lax change Fx 79.0 to 81.0 Beta"* ✅ — a Beta experiment across Firefox 79–81.
- **It was shipped in Firefox 96.** Firefox 96.0 was released **11 January 2022** ✅, and Mozilla's own Firefox for Enterprise 96 release notes state: *"Firefox will now enforce the Cookie Policy: Same-Site=lax by default. There are two new policies that modify this behavior: `LegacySameSiteCookieBehaviorEnabled` and `LegacySameSiteCookieBehaviorEnabledForDomainList`."* ✅ The tracking flag on the meta-bug is `firefox96: verified` ✅. So as of Firefox 96 the feature was intended to be live.
- **It was then disabled.** Bug 1812297, *"Disable sameSite=lax by default on Nightly"*, is **RESOLVED FIXED** with milestone **"111 Branch"** ✅, and its description is unambiguous about the reason: *"There are currently too many issues with lax-by-default to consider shipping the feature, so we might as well stop breaking sites for our Nightly population. We are also going to remove the experimental flag."* ✅ In the same bug, the assignee states: *"this pref is set to false by default even in Nightly and **we have no plans for enabling the pref**."* ✅
- **The meta-bug is closed as WONTFIX, and Mozilla's current position is stated in it.** Bug 1617609, *"[meta] Enable sameSite=lax by default"*, is **RESOLVED WONTFIX** (with `firefox124: wontfix`) ✅. A Mozilla engineer's comment in it reads: *"Firefox does not implement Lax by default. A few years ago an attempt was made, however there was too much web breakage to proceed at the time. As a result the current behaviour is when no SameSite attribute is set we use **None** by default. We currently have no plans to attempt again in 2025, and it's not totally clear what our long term plans are."* ✅

**The conclusion, stated plainly: as of that record, Firefox treats a cookie with no `SameSite` attribute as effectively unrestricted (`None`), not as `Lax`.** An estate that sized its residual risk on "all modern browsers default to Lax" has overstated its coverage for the entire Firefox population. One ⚠ caveat belongs alongside this: the sequence (shipped in 96, disabled on the 111 branch) leaves the state of *releases between roughly 96 and 110* not fully pinned down by the sources read for this guide, and the caniuse feature table for "Set-Cookie: SameSite defaults to Lax" lists Firefox as *"disabled by default"* across the version ranges it covers ✅ — a data point consistent with the WONTFIX outcome and inconsistent with a live enforcement. The safe engineering reading is the conservative one: **do not assume Firefox withholds the cookie.** §14 records the limitation.

**The WebKit/Safari record.** WebKit *did* adopt Lax-by-default, later than the others and with a notable rough edge:

- WebKit bug 278353 is titled *"Update remaining tests to conform to **new SameSite=Lax by default cookie behavior**"* ✅, establishing that the behaviour change landed in WebKit.
- WebKit bug 279153 (*"iOS 18 does not allow a client application to specify SameSite=None"*, reported **4 September 2024**) is the clearest contemporaneous description of the change and its effect: the reporter's organisation had *"been relying on the default behavior, when SameSite is omitted for WebKit to consider it to be `SameSite=None`"*, and their analysis identifies *"the new WebKit change that breaks this assumption to now use `SameSite=Lax` as the default."* ✅ The framing in that bug — *"What is the purpose of introducing the backwards incompatible change for stricter cookie behavior (default SameSite=Lax) in WebKit?"* ✅ — dates WebKit's change to the **iOS 18 / Safari 18 (2024)** window, and demonstrates that it broke client-set cross-site cookies in hybrid apps.
- The follow-up: a WebKit engineer states *"SameSite=None cookies by default have been restored in iOS 18.0.1"* ✅ (3 October 2024), resolving the client-set-`None` regression. Read carefully, that is a **fix for client-specified `None` cookies in hybrid apps**, not a repeal of Lax-by-default for ordinary server-set cookies — but the distinction is subtle enough that this guide does not claim the final Safari behaviour for every cookie class. ⚠ Recorded in §14.

**What to do with this correction, operationally.** Three things follow, and none of them is "give up on `SameSite`":

1. **Never rely on the default.** Set the attribute explicitly on every cookie. Explicit labelling removes your exposure to the `Lax-allowing-unsafe` carve-out (§4.3), removes your dependence on which browser is in front of you, and makes the intent reviewable in a diff.
2. **Treat "the browser protects us" as a browser-and-version-dependent claim, and measure it.** §9.5's regression test is the mechanism: assert the *observed* `Set-Cookie` attribute set on the wire, and assert the *observed* cross-site behaviour in at least the browsers your population actually uses.
3. **Carry the correction into §3's ranking.** `SameSite` sits at position 2 in the defence stack *when it is explicitly configured by you*; as a *browser default alone* it is a partial, non-uniform control and must be paired with a token or Fetch Metadata policy for any estate with a meaningful Firefox, older-browser or embedded-client population.

### 4.7 Practical Misconfiguration Patterns

| Symptom | Cause | Guardrail |
|---|---|---|
| "Our SSO / payment / embedded flow broke after the SameSite change" | A genuinely cross-site cookie was unlabelled and is now withheld, or `SameSite=None` was set without `Secure` and the cookie was **rejected outright** ✅ | Label deliberately: `None; Secure` for genuinely cross-site cookies; confirm HTTPS everywhere first |
| "It works immediately after login then fails two minutes later" | The Lax+POST carve-out window (§4.3): fresh unlabelled cookie rides a cross-site top-level POST for ≤2 minutes, then stops | Set `SameSite` explicitly. This symptom is a *diagnostic* for the carve-out, not a bug in the browser |
| "Logout works cross-site but the action doesn't" | Logout is a `GET`; the action is a `POST`. The `GET` is a permitted cross-site top-level safe navigation under `Lax` | Fix the `GET` side-effect (§2.4); do not reach for `Strict` to paper over it |
| "It worked on our subdomain, broke on the sibling" | Site-scope confusion: the "site" is the registrable domain, not the origin | Never share a registrable domain with hosts you do not fully control; prefer `__Host-` (no `Domain`) on critical cookies ✅ |
| "Only Firefox users are affected" | Not necessarily a Firefox bug — see §4.6: the browsers do not agree on what an unlabelled cookie means | Explicit labelling removes the variable |
| "Cookie is set but never sent" | Most often `SameSite=None` without `Secure`, or `Secure` on an HTTP-dev environment, or a scheme mismatch (http vs https is cross-site under schemeful same-site) | Check the browser's cookie-storage panel for the rejection reason before changing application code |
| "We set `SameSite=None` everywhere to make an integration work" | Anti-pattern #2 (§12) — a global opt-out is indistinguishable from having no browser-level defence | Scope `None` to the specific cookies that need it, with a written justification |

---

## 5. The Token Patterns in Depth

### 5.1 Why a Token Works at All

The token mechanism rests on exactly one asymmetry, and it is worth naming before any of the variations, because every variation is a way of managing or hardening that one asymmetry: **the attacker can cause a request, but cannot read a response or the victim's page.** The attacker's page cannot read the value from a cross-origin response (SOP), and cannot read it out of the victim's document (absent XSS, §7.2). So the server can require a value back that the attacker cannot know.

The precise wording matters, because it shows where the defence's trust is rooted. OWASP: CSRF tokens *"prevent CSRF because without a CSRF token, an attacker cannot create valid requests to the backend server"* ✅ — and the requirements they must satisfy are structural:

- **Unique per user session** ✅
- **Secret** ✅
- **Unpredictable** — *"large random value generated by a secure method"*, i.e. a CSPRNG ✅

And the transmission rule, which is where implementations most often go wrong: a *synchronizer* token **must not be transmitted in a cookie** — *"A CSRF token should not be transmitted in a cookie for synchronized patterns"* ✅ — must not leak into logs or the URL, and should be carried in a form field or, preferably, a custom header, because *"Since requests with custom headers are automatically subject to the same-origin policy, it is more secure to insert the CSRF token in a custom HTTP request header via JavaScript than adding a CSRF token in the hidden field form parameter."* ✅ The reason a token must not ride a cookie in the synchronizer pattern is the whole point of the pattern: if the token travels as a cookie, the browser attaches it automatically on the forged request too, and the server would be comparing two attacker-supplied values.

### 5.2 The Synchronizer Token, End to End

Django's implementation is the clearest documented end-to-end synchronizer in mainstream use, and reading it as a design is more instructive than reading the pattern abstractly. Per Django's own documentation ✅ (Django 6.1 `ref/csrf`):

1. **Issuance.** A CSRF cookie holds *"a random secret value, which other sites will not have access to"*, sent whenever the token is requested by a view. **The secret changes each time a user logs in** ✅ — an important and often-omitted detail, because it bounds the value's lifetime to a session and defeats pre-login token fixation.
2. **Delivery into the page.** A hidden form field named `csrfmiddlewaretoken`, present in all outgoing `POST` forms. Critically, the field value is **not** the secret: *"It is scrambled differently with each response using a mask. The mask is generated randomly on every call to `get_token()`, so the form field value is different each time"* ✅ — explicitly to protect against **BREACH**, a compression side-channel, not against CSRF.
3. **Comparison.** For every request not using `GET`, `HEAD`, `OPTIONS` or `TRACE`, the cookie must be present and the field must be present and correct, else `403`. And the comparison is on the secret only: *"When validating the 'csrfmiddlewaretoken' field value, only the secret, not the full token, is compared with the secret in the cookie value. This allows the use of ever-changing tokens."* ✅ This is the mechanism that lets the page emit a different token per response while the server stores one value.
4. **Augmented by provenance checks.** Django verifies the `Origin` header, *"if provided by the browser, against the current host and the `CSRF_TRUSTED_ORIGINS` setting. This provides protection against cross-subdomain attacks"* ✅ — and for HTTPS requests with no `Origin`, it *"performs strict referer checking"*, so that *"even if a subdomain can set or modify cookies on your domain, it can't force a user to post to your application since that request won't come from your own exact domain."* ✅
5. **Method scope.** It *"deliberately ignores GET requests (and other requests that are defined as 'safe' by RFC 9110 Section 9.2.1)"* ✅ — and Django is candid that this is a bet on the application keeping its side: *"These requests ought never to have any potentially dangerous side effects, and so a CSRF attack with a GET request ought to be harmless."* ✅ The bet is exactly the §2.4 hazard, and Django names it as the app's responsibility.

The transferable design lessons, independent of framework: **(a)** a per-login secret rotation is cheap and closes a whole sub-class; **(b)** mask the token per response if you care about compression side-channels, and understand that this is a *different* threat from CSRF; **(c)** the synchronizer pattern is meaningfully stronger when it is *combined* with Origin/Referer verification, which is what Django does; **(d)** the framework will not stop you from breaking the invariant in item 5.

### 5.3 Per-Session Versus Per-Request — the Real Trade

OWASP frames the trade: tokens *"should be generated only once per user session or each request. Because the time range for an attacker to exploit the stolen tokens is minimal for per-request tokens, they are more secure than per-session tokens. However, using per-request tokens may result in usability concerns."* ✅ The usability failure mode is named precisely — the browser **Back** button *"can be hindered by a per-request token as the previous page may contain a token that is no longer valid"*, producing *"a CSRF false positive security event on the server-side"* ✅.

The honest engineering assessment, which the guide will state rather than hedge:

| | Per-session token | Per-request token |
|---|---|---|
| Replay window if *leaked* | The whole session | One request |
| Practical exploitability of the window | Low in the classical model, because the attacker cannot read the token to begin with — leakage requires a *different* bug (XSS, log exposure, referrer leakage) | Lower still, in principle |
| Real cost | Nothing | Back-button false positives, multi-tab races, retry/idempotency complexity, harder caching of rendered pages |
| Verdict | Correct default for most applications, especially when combined with login-time rotation | Justified where the token can plausibly leak (rendered pages cached in proxies, tokens in URLs, weak log hygiene) — but fix *those* first |

**The important qualifier:** OWASP is equally clear that tighter rotation is not a free win and that the perceived benefit is often illusory — updating the token *"too frequently, such as for each request, is a misconception that assumes it adds substantial security while actually harming the user experience"* ✅. The reason is structural: **if the attacker can read your token, they can read the *next* one too.** Per-request rotation does not close an XSS or leakage hole; it only shortens a window that was already opened by a different defect. The genuinely valuable rotation is the one nobody has to administer: regenerate on **authentication** (login, privilege change, session re-establishment), which Django does ✅ and which costs nothing.

### 5.4 The Double-Submit Cookie and Its Documented Exposure

**The naive pattern.** A cryptographically strong random value is set as a cookie *and* required back in a request parameter or header; the server accepts if the two match. No server-side state. The classic argument for its safety is the same-origin policy: *"Because of the same-origin policy, an attacker cannot read or modify the value stored in the cookie"* ✅ (CWE-352's own description of the technique, attributed to Felten and Zeller). OWASP adds the requirement that makes it work at all: the value must come back *"as a custom request header or form parameter ONLY. Cookie validation is INSECURE"* — because *"Browsers auto-send cookies on cross-site requests. Attackers can trigger this automatically. Security requires explicit client submission (header/param) proving user intent."* ✅

**Its documented exposure, stated concretely.** The pattern's premise is that the attacker cannot *write* a cookie on the target domain. That premise fails in several real situations, and OWASP now marks the naive pattern **DISCOURAGED** with an explicit warning: *"The Naive Double-Submit Cookie pattern is bypassable by an attacker who can write cookies on the target domain (e.g., via a vulnerable sibling subdomain, DNS takeover, or plaintext-HTTP cookie injection on a non-`__Host-` cookie). For new code, use the Signed Double-Submit Cookie pattern above. The naive pattern is documented for reference only."* ✅

The mechanism of the bypass: if the attacker can set a cookie on the target's registrable domain, they set *both* values — the cookie and the matching request parameter — to a value of their choosing, and the server's "do these two client-supplied strings match?" test passes. The attacker has not forged a secret; they have *supplied* the pair, and the naive pattern cannot distinguish a pair the server issued from a pair the client manufactured. Sources beyond OWASP document the same class: a first-party confirmation comes from **Django's own documentation**, which states the limit in its *Limitations* section — *"Subdomains within a site will be able to set cookies on the client for the whole domain. By setting the cookie and using a corresponding token, subdomains will be able to circumvent the CSRF protection. The only way to avoid this is to ensure that subdomains are controlled by trusted users (or, are at least unable to set cookies)."* ✅ An independent conference treatment of the same weakness — David Johansson's *"Bypassing CSRF Protections: A Double Defeat of the Double-Submit Cookie"* (OWASP London), cited by OWASP's cheat sheet — details cookie fixation and subdomain/MITM variants ✅.

**How the exposures should be ranked, honestly.** (i) A **vulnerable or attacker-controlled sibling subdomain** is the most practically reachable route, and it scales with estate sprawl — which is why this is a *greater* concern for a large enterprise than for a single-domain startup. (ii) **DNS takeover of a dangling subdomain** turns the same condition from "someone made a mistake on a subdomain" into "anyone can claim it". (iii) **Plaintext-HTTP cookie injection** on a non-`__Host-` cookie requires a network position; HTTPS-everywhere plus `__Host-` closes it. (iv) **XSS** defeats the pattern outright, as it defeats everything in this section.

The mitigations are therefore structural rather than cryptographic: use `__Host-` prefixed cookies for the token so a subdomain cannot set or overwrite it (§4.5) ✅; never share a registrable domain with hosts you do not control ✅; enforce HTTPS so plaintext injection is unavailable; and — the real fix — sign the token (§5.5). Note which of these is *already provided for free* by the platform: the `SameSite` attribute's creation-side rule (§4.4) prevents a cross-site subresource from setting a `Lax`/`Strict` cookie on your origin, so an attacker's cross-site page cannot plant the token cookie. The remaining exposure is the *sibling subdomain and network* cases, which `SameSite` cannot address because those requests are same-site by definition.

### 5.5 The Signed / HMAC-Bound Variant — What It Actually Buys

**The fix in one sentence:** instead of asking "do these two client-supplied values match?", ask "did *we* issue this value, and for *this* session?" A signature answers the first; a session binding answers the second; you need both.

OWASP's construction: an HMAC over a message that binds a session-dependent value (the server-side session ID) with a fresh random value, keyed by a server-side secret; the returned token is the HMAC plus the random value, with the session ID deliberately *not* carried in plaintext in the token ✅. Verification recomputes the HMAC from the request's random value and the *current* session and compares with a **constant-time** comparison — *"The `constantTimeEquals` function should be used to compare the HMACs to prevent timing attacks"* ✅. The token may be delivered in a cookie *without* `HttpOnly` so the client can read it and echo it back in a header — which is what makes this the natural stateless/SPA variant.

**What it buys, precisely.** Two things, and it is worth being exact because they are separate:

1. **It defeats the cookie-injection bypass.** A sibling subdomain or a network attacker can *write* a cookie, but cannot produce a valid HMAC over a message they control without the server's secret key. The forgery moves from "supply two matching strings" to "break HMAC-SHA-256", which is not a real option.
2. **It binds the token to the session**, so a token obtained for one session cannot be replayed into another. OWASP is emphatic that this is not optional: the signed pattern *"explicitly ties tokens to the user's authenticated session (e.g., session ID)"* and *"Simply signing tokens without session binding provides minimal protection and remains vulnerable to cookie injection attacks. Always bind the CSRF token explicitly to session-specific data."* ✅

**What it does not buy.** It does not survive XSS — an attacker with script in your origin reads the token cookie and echoes it correctly, HMAC and all. It does not survive a compromised origin. And it introduces a real operational dependency: **the HMAC secret becomes availability-critical.** A key rotation that invalidates in-flight tokens logs users out mid-flow; a multi-service deployment where each service holds a different key produces intermittent 403s that look like CSRF attacks and are not. This is the correct moment to note the honest scaling limit of stateless CSRF defence: it converts a storage problem into a key-distribution problem, and key distribution across regions and services is not free.

**The framework reality, so the reader knows what they are choosing between.** Spring Security's default for a servlet application is stateful — the `CsrfFilter` persists tokens via a `CsrfTokenRepository`, defaulting to `HttpSessionCsrfTokenRepository`, with `CookieCsrfTokenRepository` available for cookie storage and a custom repository supported ✅. Spring Security 6 changed two defaults relevant here: token loading is *"deferred by default to improve performance by no longer requiring the session to be loaded on every request"*, and the token *"now includes randomness on every request by default to protect the CSRF token from a BREACH attack"* ✅ — and Spring warns that *"The changes in Spring Security 6 require additional configuration for single-page applications"* ✅. Django, similarly, is hybrid: a stateful secret in a cookie, paired with per-response masking and a stateless-ish comparison against the cookie-held secret ✅. ASP.NET Core's antiforgery system is token-and-session based and can be extended to round-trip additional data per token via `IAntiforgeryAdditionalDataProvider`, with the client's username already embedded in the generated tokens ✅. **None of these defaults is a signed double-submit**, and that is a legitimate choice — they are all "synchronizer or stateful-cookie" designs whose security rests on server-side state rather than on a signature. The engineering question is not "which is better" but "which failure mode can I operate": stateful designs need session affinity and store; signed designs need key distribution. §5.7 is the reconciliation.

### 5.6 Where the Token Lives — Session, Cookie, Meta Tag, Form

| Storage / channel | Used by | Strength | Constraint |
|---|---|---|---|
| **Server-side session state** (token compared to stored value) | Django (secret in cookie, compared server-side), Spring `HttpSessionCsrfTokenRepository` | Strongest: the comparison value is not client-controllable at all | Requires server-side state; session affinity or a shared store in a horizontally scaled deployment |
| **Form hidden field** | Django (`csrfmiddlewaretoken`), Rails, most server-rendered frameworks | Works for HTML forms, which cannot carry custom headers; simplest to reason about | Requires every form to be rendered by a template that injects it — a hand-written form is a silent hole |
| **Custom request header** (`X-CSRF-Token`, `X-XSRF-Token`, `CSRF-Token`, `X-CSRFToken`, `X-CSRF-Token`) | SPAs and AJAX; the conventions differ per framework ✅ | Best for scripted calls: subject to SOP, and non-simple so a preflight is implied (§3.6) | Unavailable to a plain `<form>`; requires the client to be scripted |
| **`meta` tag read by script** | SPAs (Django's `csrf_token` in a template, then read by JS) | Clean separation: token embedded in the page once, used by all requests | Requires the page to be served by the application; fails for static-hosted SPAs calling a different origin |
| **Readable (non-`HttpOnly`) cookie, mirrored into a header** | The double-submit family; Angular's `XSRF-TOKEN`/`X-XSRF-TOKEN` convention | Stateless, and works across an SPA/API split | Deliberately JS-readable, so XSS-exposed; **must not** be the *only* comparison value (naive pattern, §5.4) — it must be signed/bound (§5.5) |
| **URL parameter** | Legacy | **Avoid.** Leaks via history, logs, proxies and `Referer` ✅ | OWASP: *"A CSRF token must not be leaked in the server logs or in the URL"* ✅ |

Two rules that cut across all six. **First, the synchronizer token must not travel as a cookie** ✅ (§5.1) — the cookie is the ambient channel, and putting the anti-ambient secret into it defeats the pattern. Second, **the token's transport determines which defence it composes with**: header-borne tokens get the preflight benefit and are the right choice for a scripted client; form-borne tokens do not, and must be paired with `SameSite` and Fetch Metadata instead. A resource reachable through *both* a form and an API is therefore the highest-risk handler in most applications, because it needs both sets of controls and usually gets neither.

### 5.7 Stateless Architectures and the Honest Limits

**The stateless problem.** A synchronizer token needs somewhere to store the reference value. In a stateless service — a horizontally scaled API with no session store, or a service that must not hold per-user state — the options are: (i) put the reference value in the token itself and sign it (a signed/HMAC double-submit, §5.5) — stateless, but now a signature key is a dependency; (ii) keep server-side state and accept the store and its affinity requirements; (iii) drop cookies for authentication entirely and use a bearer token (§6.1), which sidesteps the problem by removing the ambient credential. Option (iii) is the honest explanation for why so many modern architectures "don't have CSRF issues" — and §6.5 draws the conclusion carefully.

**The limits, stated without hedging.** Every claim in this list is supported by a source read for this guide, and every one of them is a reason not to treat a token as a cure:

1. **XSS defeats all of it.** The specification of the mitigation and the framework documentation agree, and it is not a caveat but a boundary: OWASP's cheat sheet opens with **"Remember that Cross-Site Scripting (XSS) can defeat all CSRF mitigation techniques!"** ✅; CWE-352's mitigations for the nonce pattern, the confirmation-request pattern and the double-submit pattern *each* carry the note *"Note that this can be bypassed using XSS (CWE-79)"* ✅; Django states that its protection *"assumes… that there aren't any cross-site scripting vulnerabilities on your site (because XSS vulnerabilities already let an attacker do anything a CSRF vulnerability allows and much worse)"* ✅. The Samy worm is the historical illustration of the composition 🚩 (widely documented; the OWASP cheat sheet cites it directly: *"XSS can be used to bypass both referrer and token based checks simultaneously. For instance, the Samy worm used an `XMLHttpRequest` to obtain the CSRF token to forge requests."* ✅).
2. **No token survives a compromised origin.** A token is a server-issued secret *for the client to hold*. If the origin is lost, the secret is lost with it. This is why §3.8's defence-in-depth argument stops at the origin boundary: token patterns address implementations being wrong, not the origin being taken.
3. **A token does not protect a safe method.** Requiring a token on a `GET` is possible but awkward, and frameworks deliberately exempt safe methods by default (Django explicitly ✅; Spring protects *"unsafe HTTP methods"* by default ✅). A state-changing `GET` is therefore outside the token defence's reach unless the team does something unusual — which is a fourth reason §2.4's rule is load-bearing rather than cosmetic.
4. **A token does not protect an exemption.** Every framework offers a way to switch the protection off — Django's `csrf_exempt` ✅, Spring's documented "disabling CSRF protection" path ✅, a per-route attribute in ASP.NET Core. Each exemption is a hole whose size nobody estimated. Anti-pattern #4 (§12).
5. **A token does not protect client-side CSRF.** If your own JavaScript constructs the request from attacker-controlled input, the token will be attached correctly by your own code ✅ (OWASP §"Client-Side CSRF Mitigation Techniques", §2.6).
6. **A token does not protect the login endpoint unless you put it there.** Django's documentation notes that login CSRF *"is also covered"* by its middleware ✅ — which is stated as a feature precisely because it is not automatic in every stack, and because the pre-authentication surface has no session to bind the secret to in the simplest designs.

**The reconciliation the reader should leave this section with.** The token is the **only** defence in the stack that protects an endpoint regardless of how the browser classifies the request — which is what makes it indispensable for the form-driven surface and for anything reached through a redirect chain or a legacy client. That strength comes from it being an *application* control, and so does its weakness: it is correct only if the application is correct. Pair it, always, with a browser-enforced control whose correctness does not depend on your code (Fetch Metadata policy or explicit `SameSite`), and the two failure modes stay disjoint — which is §3.8's argument, arrived at now from the token side.

---

## 6. The API and Single-Page-Application Era

### 6.1 Why a Bearer Token in a Header Is Structurally Different

The single most consequential change in CSRF exposure over the last fifteen years was not a defence. It was a credential change. Understanding why requires only one observation, and it is the observation §1.4 set up: **a bearer token in an `Authorization` header is not ambient.**

Compare the two credential models along the axis that matters:

| | Session cookie | Bearer token in `Authorization` header |
|---|---|---|
| Who attaches it to the request | **The browser**, automatically, according to the cookie's own rules | **The application's code**, explicitly, per request |
| Can a cross-site page cause it to be attached? | **Yes** — that is precisely the CSRF problem | **No** — the attacker's page cannot read the token, so it cannot set the header, and the browser will not add it for them |
| Where it must be stored for a browser client | The cookie jar (browser-managed) | Local storage / memory (script-managed), which makes it XSS-readable |
| What CSRF needs from it | Nothing — the browser volunteers it | Everything — and it cannot be had |

Microsoft's ASP.NET Core documentation states the conclusion directly: *"Placing a token in the browser local storage and retrieving it and using it as a bearer token provides protection against CSRF attacks. However, should the app be vulnerable to script injection via XSS or a compromised external JavaScript file, a cyberattacker could retrieve any value from local storage and send it to themselves."* ✅ And the guidance is explicit about where the concern lies: *"Don't be concerned about CSRF vulnerability if the token is stored in the browser's local storage. CSRF is a concern when the token is stored in a cookie."* ✅

Note precisely what that is and is not a claim. It is a claim that **the CSRF attack model does not apply**, because the attacker cannot make the browser attach a credential it does not control. It is *not* a claim that the architecture is more secure overall — it is a **trade of one vulnerability class for another**: CSRF risk falls to near-zero, XSS impact rises from "read the DOM" to "exfiltrate a credential". Whether that is a good trade depends entirely on which bug class you are more likely to have, and it is worth saying plainly that most teams are worse at preventing XSS than at not-writing-DOM-sinks. The token-in-local-storage pattern is also the one *the OAuth 2.0 security best-practice guidance discourages* for browser clients precisely on those grounds — a point the [FAPI guide](../banking/fapi_financial_grade_api_guide.md) covers in its flow-protection analysis; this guide does not re-derive it.

### 6.2 The CORS Interaction — a Read Restriction, Not a Send Restriction

CORS is the most conflated topic with CSRF in the whole field, so state the definition from the vendor-neutral source and then draw the line. MDN: CORS *"is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit **loading resources**."* ✅ — *loading resources*, i.e. **reading responses**. And MDN's own explanation of *why* simple requests are not preflighted is the cleanest possible statement of the boundary, because it is framed in terms of CSRF explicitly:

> *"The motivation is that the `<form>` element from HTML 4.0 (which predates cross-site `fetch()` and `XMLHttpRequest`) can submit simple requests to any origin, so anyone writing a server must already be protecting against cross-site request forgery (CSRF). Under this assumption, the server doesn't have to opt-in (by responding to a preflight request) to receive any request that looks like a form submission, since the threat of CSRF is no worse than that of form submission. However, the server still must opt-in using `Access-Control-Allow-Origin` to **share the response with the script**."* ✅

Read that last clause twice. **The request is received regardless; CORS governs whether the response may be shared with the script.** The design assumption stated in MDN is that the server must *already* be defending against CSRF — CORS is explicitly not that defence, and was never intended to be.

So, the distinctions that matter:

- **CORS is a read restriction.** It prevents an attacker's page from *reading* a cross-origin response. It does not prevent the *sending* of any request the browser can already make — form posts, image loads, simple `fetch`es. A permissive CORS policy therefore does **not** "cause" CSRF; it is a different (and serious) misconfiguration class.
- **But permissive CORS can reopen exposure** in the specific case where the application relies on a *non-simple* requirement, or on a custom header, as its CSRF defence. If the server answers `Access-Control-Allow-Origin` for the attacker's origin — or worse, reflects an arbitrary origin while also allowing credentials — then the preflight passes, and a request that *should* have been un-sendable becomes sendable. This is exactly the failure OWASP warns about with regex-matched subdomain allowlists: *"If an attacker is able to take over a subdomain (not uncommon with cloud services) your CORS configuration would allow them to bypass the same origin policy and forge a request with your custom header."* ✅
- **The credential rules are strict and are a design constraint, not a caveat.** Cookies are not sent on cross-origin requests by default; enabling them requires `Access-Control-Allow-Credentials: true`, and the browser *"will reject any response that includes `Access-Control-Allow-Origin=*` if credentials are allowed"* ✅ (OWASP). So the dangerous configuration is not `*` with credentials — the browser refuses that combination — but a **reflected or over-broad origin allowlist with credentials enabled**. That is the configuration that turns a read restriction into a send permission.
- **A bearer token is not affected by any of this**, which is the point of §6.1. CORS questions only arise for the client that must *read* a response — not for the server-side attacker who merely needs a request to land.

The practical rule for a design review: **ask what your CORS policy permits *reading*, and separately ask what your CSRF defence is.** Confusing the answers is the single most common architectural error in this area, and it produces the specific misstatement catalogued in §7.5.

### 6.3 Same-Site Versus Cross-Site API Calls in an SPA

An SPA introduces a distinction that server-rendered applications rarely have to think about, and it determines which defences are even available.

**Case A — the SPA and its API share an origin (or at least a site).** The SPA is served from `app.example` and calls `app.example/api/…`. Every request is same-origin or at least same-site, so `SameSite=Lax` (or `Strict`) is fully effective, and Fetch Metadata classifies requests as `same-origin`. The defences available are the full stack, and the architecture is the easy one. Note that this case is where a same-origin Fetch Metadata policy is *at its strongest* and simultaneously where a naive policy is at its most dangerous: a deny-list written as "reject if `cross-site`" is exactly the check that a same-origin XSS-forged request sails straight through.

**Case B — the SPA is served from one origin and calls an API on a different site.** Now every call is genuinely cross-site. `SameSite=Lax` will withhold the session cookie on all of them unless the cookie is labelled `None` — which removes the browser-level defence for that cookie entirely — and Fetch Metadata will report `cross-site` for *legitimate* traffic, which means a default-deny policy has to carve out the API surface. The defences that remain workable:

1. **Change the credential model.** Use a bearer token in an `Authorization` header (§6.1) and make the calls non-simple. This is the clean answer and the reason it is so common.
2. **Keep the cookie but require a custom header plus a strict CORS allowlist** (§3.6). The preflight becomes the structural control, and the token in that header covers the case where the allowlist is later widened by mistake. This is the pattern Spring Security flags as needing *"additional configuration for single-page applications"* in its version 6 changes ✅.
3. **Keep the cookie, label it `None; Secure`, and carry the weight on a signed, session-bound token** (§5.5) — the most common enterprise compromise, and the one with the most residual exposure, because it silently discards the browser-enforced layer.

**The uncomfortable third case, which is where real estates live: the split deployment.** Some endpoints are consumed by the SPA, some by server-rendered pages, and some by both — often across a migration that will not finish. The handler reachable through both a form and the API (§5.6's "highest-risk handler") lives here, and so does the endpoint that was exempted from the CSRF filter to make the SPA's first integration work and was never re-protected. §11's worked example is built on exactly this shape, because it is the shape most regulated enterprises actually have.

### 6.4 The Mobile and Hybrid Client Case

Three distinct client types, three different exposure profiles — and the distinction is routinely flattened into "mobile is safe":

**Native mobile applications that make their own HTTP requests.** A native app constructs and sends its own requests. There is no third-party page in a position to cause the app's HTTP client to attach a credential, so the classical CSRF model does not apply to the app-to-API leg. This is a real reduction in exposure, and it is structural rather than a defence — the same reason §6.1 gives. Two caveats: the app may store the credential insecurely (an ordinary client-side secret-storage problem, owned by the [distributed_auth_guide.md](distributed_auth_guide.md)), and the endpoints the app calls are the *same endpoints* a browser can call, so server-side CSRF defences must be scoped by surface rather than switched off globally.

**Hybrid applications embedding a webview.** The webview is a browser, so the browser's rules apply inside it — including HTML forms, redirects, and any web content the webview can be induced to load. The webview is *not* automatically protected by the same browser-level controls, because the modern `SameSite` behaviour in Android WebView only applies to apps **targeting Android 12 and newer**, and existing apps are unaffected until they retarget ✅ (Chromium, 8 January 2021).

**Client-set cookies in webviews, which is where the sharpest recent lesson sits.** WebKit bug 279153 documents exactly this failure: an enterprise iOS app had been relying on *"the default behavior, when SameSite is omitted for WebKit to consider it to be `SameSite=None`"*, and WebKit's Lax-by-default change broke its IdP cookie flows (reported 4 September 2024) ✅. The remediation required the app's cookie-setting code to specify a `SameSite` policy explicitly, and the regression was resolved for client-specified `None` cookies in *"iOS 18.0.1"* ✅ (3 October 2024). Two transferable lessons: **(a)** if your app injects cookies into a webview's cookie store rather than receiving them from a server, you are relying on platform defaults that are not stable, and you should set the policy explicitly; **(b)** a change in a browser default can break a *client* that has no way to defend itself, which is why §9's testing plan includes the client platforms, not just the desktop browser.

### 6.5 The Honest Claim — Exposure Reduced by Changing the Credential Model

This is the subsection that most CSRF writing gets wrong in the optimistic direction, so state it with precision.

**What is true.** A large fraction of modern HTTP APIs are structurally immune to classical CSRF, because they authenticate with a credential the client must explicitly attach — typically a bearer token in an `Authorization` header — and the browser will not volunteer it. That is a genuine elimination of the attack class for those endpoints, not a mitigation of it. Combined with two other trends — the removal of the state-changing `GET` from modern REST designs, and the maturing of built-in framework defence (the reason OWASP Top 10 removed CSRF, discussed in §8) — the honest explanation for CSRF's reduced *frequency* in modern application portfolios is exactly this **credential-model shift**, plus framework defaults, plus the browser change from §4. It is not evidence that the vulnerability was solved.

**What is not true, and must be said plainly.** Three residual populations keep the vulnerability live, and all three are common rather than exotic:

1. **Wherever the cookie model remains, the vulnerability remains.** Retail online banking is the paradigm case (§10). A banking session is a browser-managed cookie, on purpose, because the alternative — a bearer token in script-reachable storage — is worse for a regulated institution's threat model. Any application that keeps a cookie-authenticated browser session for its primary traffic has CSRF exposure, and its defence is the §3 stack, not the credential model.
2. **The mixed estate is the common estate.** An organisation rarely converts every endpoint at once. The endpoints that still accept cookies — and especially the ones that accept *both* a cookie and a bearer token — are the exposed surface, and they are usually the oldest and most business-critical. §5.6 and §6.3 both point at the same handler type.
3. **The "we use JWTs so we are immune" architecture that still carries a session cookie.** This is the most dangerous of the three, because immunity has been *asserted* in a design document and the cookie is still there, doing the authenticating. §12 lists it as an anti-pattern. The diagnostic is one question: *what credential does the server actually validate on this endpoint?* If the answer is a cookie, immunity is a story, not a property.

**The claim this guide will defend, stated in one sentence:** modern architectures reduced CSRF exposure mainly by changing the credential model rather than by adding a CSRF defence, so CSRF's decline is a *portfolio composition* effect — and it reverses the moment an organisation reintroduces a cookie-authenticated browser flow, which regulated financial services do on purpose, and which SPA migrations do by accident.

---

## 7. The Adjacent Bugs, Separated Exactly

### 7.1 The Shared Substrate — the Same-Origin Policy

All four bugs in this section are consequences, in different directions, of the same platform rule. Before separating them, anchor the substrate precisely, because every separation below is really a statement about *which direction* the SOP constrains:

- **The SOP constrains reads.** Script from origin A cannot read responses from origin B without CORS authorisation (§2.2, §6.2).
- **The SOP does not constrain sends.** The browser will transmit a wide range of cross-site requests without any permission from the target (§2.2).
- **The SOP does not constrain framing.** Origin B's page can be *loaded inside a frame* on origin A's page, and the rendered pixels are then composable — which is what makes clickjacking possible (§7.4).
- **The SOP does not constrain cookie attachment.** Cookie attachment follows the cookie's own scope and `SameSite` rules, not the SOP (§2.3). ASP.NET Core's documentation notes the asymmetry in exactly these terms: *"the same-origin policies that govern AJAX requests don't necessarily apply to HTTP cookies"* ✅.

Hold that list. CSRF exploits the second and fourth bullets; CORS is the mechanism governing the first; clickjacking exploits the third; XSS is the failure of the origin boundary itself, which invalidates all four.

### 7.2 CSRF vs XSS — the Asymmetry in Both Directions

The confusion here is not about definitions — everyone can define XSS and CSRF — it is about **relative severity and which one to fix first**, and the answer is not symmetric. Get both directions right.

**Direction 1: XSS defeats every CSRF defence.** This is the direction that is usually stated, and it is unconditionally true. An attacker with script execution in your origin is *inside* your origin, so:

- The synchronizer token is readable from the page (or from the response the script can make) and can be attached to a scripted request.
- The double-submit cookie, signed or not, is readable if it is JS-readable by design, and echoable either way.
- The Fetch Metadata headers will classify the request as `same-origin`, because it *is* same-origin — the strongest possible provenance signal, granted to an attacker.
- The `Origin` check will pass for the same reason.
- `SameSite` is irrelevant: the request is same-site.
- The step-up prompt can be triggered and, in the extreme, socially engineered alongside the attack.

The three sources read for this guide converge on this without hedging: OWASP's cheat sheet opens with the warning *"Remember that Cross-Site Scripting (XSS) can defeat all CSRF mitigation techniques!"* ✅; CWE-352's potential mitigations repeatedly annotate themselves with *"Note that this can be bypassed using XSS (CWE-79)"* ✅; Django's documentation states its protection assumes *"there aren't any cross-site scripting vulnerabilities on your site (because XSS vulnerabilities already let an attacker do anything a CSRF vulnerability allows and much worse)"* ✅. And the practical illustration is the Samy worm, which OWASP cites precisely as the composition: *"XSS can be used to bypass both referrer and token based checks simultaneously. For instance, the Samy worm used an `XMLHttpRequest` to obtain the CSRF token to forge requests."* ✅

**Direction 2: CSRF defence does nothing against XSS.** The reverse does **not** hold, and this is where design reviews go wrong. A perfect CSRF defence — token, SameSite, Fetch Metadata, step-up, all of it — leaves XSS completely untouched, because XSS does not need the browser to attach anything on the attacker's behalf. The attacker's script runs as your origin; there is no forgery to detect, no provenance to check, no intent to verify. **A team that has deployed excellent CSRF protection has not reduced its XSS risk by one unit.** Conversely, fixing XSS *does* reduce CSRF risk, because removing the origin compromise removes the one actor who can defeat every CSRF defence at once.

**The practical ordering that follows.** XSS is the higher-severity bug class in a cookie-authenticated application, not because it is more prevalent but because it **strictly dominates**: an XSS attacker can do everything a CSRF attacker can do, plus read data, plus persist, plus escalate. So the correct engineering order is: fix XSS first (it is the more powerful primitive), then deploy CSRF defences (which handle the attacker who *cannot* get script in) — and understand that the second step is not made redundant by the first, because most CSRF attackers are not in a position to obtain XSS and the two attack models have different reachability. The framing to avoid is the one that appears in real reviews: *"we have CSRF tokens, so we're covered"* — the token covers the attacker who cannot run script, which is most attackers; it covers nothing about the one who can.

**One more distinction that matters for remediation.** CSRF defences are *not* a partial XSS control, and treating them as one produces the wrong fix. If a review finds an XSS hole and a reviewer argues "the CSRF token limits the impact", that is incorrect on both counts: the token does not limit XSS impact at all, and an XSS attacker defeats the token. Fix the XSS with XSS controls (output encoding, a Content Security Policy, framework auto-escaping, avoiding `innerHTML`-class sinks) — see [security_by_design_guide.md](security_by_design_guide.md) for the coding-standard side and [cybersecurity_guide.md](cybersecurity_guide.md) for the discipline.

### 7.3 CSRF vs CORS

Covered in mechanism at §6.2; the separation stated compactly here so §7 stands alone.

| Question | CSRF | CORS |
|---|---|---|
| What does it describe? | An *attack*: unauthorised state change via a victim's ambient credential | A *permission mechanism*: which origins a browser will let a script read responses from |
| Is it a browser mechanism or an attack class? | Attack class (the mechanism it exploits is the credential-attachment rule) | Browser mechanism (an HTTP-header-based opt-in) |
| Does the browser prevent the request being sent? | **No** — this is the point | **Only** for non-simple requests, and only by failing the preflight |
| Can a permissive policy *cause* CSRF? | — | No. But a permissive policy can **remove a defence you were relying on** (the preflight veto) |
| What is the fix? | The §3 stack | A strict, explicit origin allowlist; credentials handled deliberately |

**The single most common misstatement** is some variant of *"we have CORS configured, so CSRF is handled"* or *"CORS is the browser's CSRF protection"*. Both are wrong for the same reason MDN's own explanation of simple requests makes unavoidable: **the request arrives whether or not CORS permits the response.** A server that receives, processes and commits a state change, and then declines to share the response with the attacker's script, has been fully exploited — the attacker never needed the response. This is also why the OWASP classical-CSRF walkthrough's careful note that a cross-origin `fetch` "will **not** be executed by modern web browsers thanks to same-origin policy restrictions" ✅ is so often over-read: it is true of that specific request shape, and it is not a general property.

**The related misstatement** is *"CSRF is a CORS problem"*. It is not. Clearing it up is usually a five-minute conversation with the difference between "may I read your response?" and "will you do what I asked?" — but it is a conversation that recurs, so this guide repeats it.

### 7.4 CSRF vs Clickjacking

The two are conflated because both involve tricking a user's browser into doing something on an authenticated site, and because both are often fixed in the same pull request. They are different bugs with different mechanisms and different fixes.

**Clickjacking** — OWASP's definition, "UI redress attack": *"when an attacker uses multiple transparent or opaque layers to trick a user into clicking on a button or link on another page when they were intending to click on the top level page. Thus, the attacker is 'hijacking' clicks meant for their page and routing them to another page."* ✅ The key structural fact: clickjacking relies on **framing** — the target page is loaded in an iframe, positioned and styled by the attacker, and the user's click lands on it. The user *is* interacting with the real application, through a real rendered UI, and — this is the crucial difference from CSRF — **the interaction is genuine from the server's point of view, because the user really did click the real button.** There is no forged request; there is a hijacked gesture.

| | CSRF | Clickjacking |
|---|---|---|
| Requires the target to be framed? | **No** | **Yes** — framing is the mechanism |
| Does the user interact with the real UI? | **No** — the request is generated without any user action on the target | **Yes** — the user's click lands on the genuine (invisible or disguised) target element |
| Does a CSRF token help? | **Yes** — it is the defence | **No** — the token is in the page the user is clicking, so it is present and valid |
| Does `SameSite` help? | **Yes**, for the cross-site cases it covers | **Partially** — OWASP lists setting auth cookies `SameSite=Strict` (or `Lax`) as one of three defences ✅, because it limits the contexts in which the framed session is authenticated |
| What is the primary fix? | The §3 stack | `Content-Security-Policy: frame-ancestors` (with `X-Frame-Options` for older-browser compatibility), plus the cookie setting and, for legacy clients, frame-busting code ✅ |

**The mutual blind spot, stated in both directions.** A CSRF token does nothing against clickjacking, because the token is legitimately present — the attacker never needed to forge anything. A `frame-ancestors` policy does nothing against CSRF, because a CSRF attack does not frame your page: an `<img>` tag, an auto-submitted form, and a top-level navigation all operate without any frame. Teams that add one and believe they have the other are the reason this section exists. Note also which control legitimately does both, partially: `SameSite` appears in both defence lists ✅ — it narrows clickjacking's reach (the framed page may not be authenticated) and it blocks a subset of CSRF. It is the only overlap, and it is a partial one in both cases, which is precisely why neither bug can be considered closed by setting it.

### 7.5 The Comparison Table, and the Misstatements It Corrects

| Bug | The shared substrate exploited | What the attacker supplies | What the attacker gets | The defence that works | The defence that does **not** |
|---|---|---|---|---|---|
| **CSRF** (CWE-352) | SOP does not constrain sends; cookies are ambient | Nothing — causes the victim's browser to send a request | A state change with the victim's authority | Provenance check (Fetch Metadata / Origin) **plus** a token **plus** explicit `SameSite` | CORS policy; `HttpOnly`; a secret cookie; HTTPS alone; "we only accept POST" |
| **XSS** (CWE-79) | The origin boundary itself has been lost | Hostile script that runs in your origin | Everything the page can do — reads, requests, persistence | Output encoding, CSP, framework escaping, safe sinks | Any CSRF defence — XSS defeats them all |
| **Clickjacking** | SOP does not constrain framing | A framed, disguised rendering of your real UI | A genuine user gesture aimed at the wrong element | `frame-ancestors` / `X-Frame-Options`; `SameSite` on auth cookies; frame-busting for legacy | CSRF tokens — the token is present and valid |
| **CORS misconfiguration** | SOP's read restriction, relaxed too far | A cross-origin page the server trusts too broadly | Reads it should not have | An explicit, strict origin allowlist; deliberate credentials handling | Being confused with a CSRF control; `Access-Control-Allow-Origin: *` with credentials is browser-refused, but a *reflected* origin with credentials is not |

**The misstatements this table exists to correct**, collected because each one appears in real design documents:

1. *"CORS protects against CSRF."* No — CORS restricts reads; CSRF needs a send. (§6.2, §7.3)
2. *"`HttpOnly` protects against CSRF."* No — `HttpOnly` stops script *reading* the cookie; the cookie is still *sent* on scripted and cross-site requests ✅. It is an XSS-impact control.
3. *"We use HTTPS, so CSRF isn't an issue."* No — ASP.NET Core's own documentation: *"Using HTTPS doesn't prevent a CSRF attack. The malicious site can send `https://…` a request just as easily as it can send an insecure request."* ✅ OWASP agrees: *"HTTPS by itself does nothing to defend against CSRF"* ✅ — it is a prerequisite for the defences being trustworthy, not a defence.
4. *"We only accept POST, so an attacker can't make a link."* No — a form and an image both work; OWASP addresses this misconception explicitly ✅ (§2.5).
5. *"We use a secret cookie, so it can't be forged."* No — *"all cookies, even the secret ones, will be submitted with every request"* ✅ (OWASP).
6. *"Multi-step flows are safe."* No — *"As long as an attacker can predict or deduce each step of the completed transaction, then CSRF is possible"* ✅ (§2.7).
7. *"Multi-factor authentication solves it."* No — as the same route through §2.4 and §4.6: MFA is verified at login, not per request, so a live session is a live session. This is the same reason step-up (§3.7) exists as a *separate* control from login MFA.
8. *"URL rewriting avoids it."* No — OWASP declines it: *"the user's session ID is exposed in the URL. We don't recommend fixing one security flaw by introducing another."* ✅
9. *"Referer checking alone is fine."* Partially, and OWASP is sceptical in isolation — *"This doesn't work in practice because the referrer header can be easily spoofed by an attacker. Additionally, some users or browsers might not send the referrer header due to privacy settings or policies, leading to false positives."* ✅ Read precisely: the *spoofing* claim applies to an attacker who already has a foothold (XSS or a client-side tool); the *suppression* claim is the practical one and is why `Origin` is preferred and why absence must be handled explicitly (§3.4). The correct reading is not "Referer checking is useless" but "Referer *alone* is a weak, high-false-positive control".
10. *"All modern browsers default to `SameSite=Lax`."* Not for Firefox, per Mozilla's own tracker ✅ (§4.6).

---

## 8. The History, Condensed and Verified

This section is deliberately short and deliberately thin in places, because a great deal of CSRF history is repeated across the web without a primary source. Where the record is genuinely thin, this section says so rather than reconstructing a narrative.

### 8.1 The Named Origin — 2001 to 2008

**The term's origin.** CSRF is widely credited to **Peter Watkins**, who described the attack pattern in a post to the **Bugtraq mailing list** in **June 2001** 🚩. The identifier and date are repeated consistently across the literature (a survey treatment states it directly: *"The Cross-Site Request Forgery (CSRF) attack was first introduced by Peter Watkins in June 2001 posting to the Bugtraq mailing list"* ⚠, citing the archive). **This guide flags it rather than asserts it**, because the Bugtraq archive itself was not read this pass and the claim's primary artefact is a 25-year-old mailing-list post. The naming itself is not in doubt — the alternate terms are recorded in the standards and weakness databases: Microsoft's threat-modelling tradition calls the class a **one-click attack** ✅ (ASP.NET Core documentation, and CWE-352's OWASP write-up), and MITRE records **"Session Riding"**, **"Cross Site Reference Forgery"** and **"XSRF"** as alternate terms ✅ (CWE-352, Alternate Terms).

**The academic formalisation.** The load-bearing historical document for modern defences is **Barth, Jackson & Mitchell, *"Robust Defenses for Cross-Site Request Forgery"*, ACM CCS '08 (15th ACM Conference on Computer and Communications Security), pages 75–88, October 2008**, DOI 10.1145/1455770.1455782, ISBN 978-1-59593-810-7 ✅. This citation is verified from the specification's own bibliography, where it appears as reference **[CSRF]** in draft-ietf-httpbis-rfc6265bis ✅ — a strong provenance chain, because it means the mechanism's current specification cites that paper as its conceptual foundation. That paper is the origin of the two defence ideas this guide treats as application-enforced: **origin-header verification** and **the double-submit cookie** (the latter attributed in CWE-352 itself to *"the 'double-submitted cookie' method as described by Felten and Zeller"* ✅).

**The weakness-database record.** CWE-352 is not a recent addition: the CWE change history shows the entry being updated from **CWE 1.0 (September 2008)** onward, with revisions through 2009 and later ✅. Its structure is worth recording because §1.3 uses it: CWE-352 is a **Composite** weakness whose required components are **CWE-346 (Origin Validation Error)**, **CWE-441 (Unintended Proxy or Intermediary / 'Confused Deputy')** and **CWE-613 (Insufficient Session Expiration)** ✅. Two design consequences follow directly from that composition and are easy to miss: (i) the confused-deputy component is not decoration — it is the formal statement of the mechanism; (ii) **CWE-613 is a component, so session lifetime is part of the CSRF surface** — a session that never expires is a CSRF primitive with no time bound, which is a genuine and often-overlooked reason to bound session lifetime — a topic the [distributed_auth_guide.md](distributed_auth_guide.md) owns from the session side.

### 8.2 The Defensive Practices That Emerged

The practices that became standard did so in a rough order that is worth recording, because each one addressed the previous one's failure:

| Era | Practice | Why it emerged / what limited it |
|---|---|---|
| Mid-2000s | **Synchronizer / form-key tokens** — a nonce per form or per session, verified server-side | The original defence. Requires server-side state; depends on every form being rendered through a token-injecting template |
| Late 2000s | **Origin / Referer checking** | Cheap and stateless, but the `null` and absent cases make it high-false-positive; the 2008 paper formalised origin verification as the more reliable half |
| Late 2000s | **Double-submit cookie** (Felten & Zeller; analysed in the 2008 paper ✅) | Stateless and scalable; carries the cookie-injection exposure that OWASP now warns about ✅ (§5.4) |
| 2010s | **Framework defaults** — CSRF middleware shipped on by default | The decisive practical shift, and the one OWASP itself cites when removing CSRF from the Top 10 (§8.5). Shifted the problem from "each developer must remember" to "the framework must be turned off" |
| 2010s | **Custom-header requirements for AJAX/APIs** ✅ | Elegant and stateless; unavailable to form-based endpoints |
| 2019–2020 | **`SameSite` by default in browsers** | Structural, requires no application change; incomplete and non-uniform (§4.6) |
| 2019–2023 | **Fetch Metadata headers** ✅ | The first defence designed for the request-metadata problem rather than for cookies; adopted across engines over a multi-year window (§3.2, §8.4) |

The pattern in that table is the same one exhibited by nearly every mature vulnerability class: **the defence migrates from the application developer to the platform.** CSRF's defences moved from "developers must add a hidden field" to "the framework does it" to "the browser does it". That migration is *why* the class became less frequent — and, as §6.5 argues, it is also why the class did not disappear.

### 8.3 The Standards Work Behind the Cookie Attribute

The `SameSite` attribute's standards history has a shape that matters for reading any modern claim about it:

- **RFC 6265** (HTTP State Management Mechanism, **April 2011**) defines the `Cookie` and `Set-Cookie` mechanics but **contains no `SameSite` attribute** ✅ — the base specification has no notion of cross-site restrictions on cookie attachment.
- `SameSite` arrives in the **`draft-ietf-httpbis-rfc6265bis`** revision series (the "bis" successor to RFC 6265). That draft is where `Strict`, `Lax` and `None` are defined, where the `Default` state and its enforcement are specified, and where the **"Lax-Allowing-Unsafe"** transitional mode is codified ✅ (§4.1, §4.3).
- **Status, dated:** the revision read for this guide is **`draft-ietf-httpbis-rfc6265bis-22`, dated December 2025** (expiry 4 June 2026), published by the IETF HTTP Working Group, editors Bingler et al. ✅ — that is, at the time of writing it remains an **Internet-Draft, not an RFC**. The practical consequence is worth stating: the mechanism everyone relies on is defined in a document that has not yet reached RFC status, which is one reason secondary sources about "the `SameSite` RFC" are frequently imprecise about which value is normative and which is a permissive option the specification leaves to the user agent.

### 8.4 The Standards Work Behind Fetch Metadata

Fetch Metadata is the newest defence in the stack and the only one conceived as a *request-classification* mechanism rather than a cookie rule, which is why it generalises beyond CSRF to the cross-site-leak class its abstract names ✅. Its documented timeline:

- **The specification** is a **W3C Working Draft**, published by the Web Application Security Working Group and edited by Mike West (Google); the revision read for this guide is **Working Draft 1 April 2025** ✅, on the Recommendation track, with the previous published version dated **31 October 2023** ✅ (Fetch Metadata Request Headers, W3C).
- **The four headers** — `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User` — with the enumerable values and semantics set out in §3.2 ✅.
- **Implementation arrived across engines over several years**, per the browser-support data reproduced in the specification's own references: `Sec-Fetch-Mode`, `Sec-Fetch-Site` and `Sec-Fetch-User` from **Chrome 76 / Edge 79 / Firefox 90 / Safari 16.4**, and `Sec-Fetch-Dest` from **Chrome 80 / Firefox 90 / Safari 16.4** ✅. Note the shape: a 2019 Chrome start, a 2021 Firefox, a **2023 Safari** — which is why OWASP can reasonably summarise support as *"all major browsers since March 2023"* ✅ and why any policy written before that had a much larger fallback population.
- **Adoption guidance followed from the standardisation**, not the other way around: OWASP's cheat sheet now recommends Fetch Metadata *"together with the fallback options"* described in it for software targeting only modern browsers ✅, and Microsoft's ASP.NET Core documentation documents the headers as a first-class input to its built-in protection ✅ — evidence that the mechanism has moved from specification into framework defaults, which is the same migration §8.2 describes.

### 8.5 How the Browser Default Change Moved the Baseline

The change's *baseline* effect — as distinct from its implementation history (§4.2) — is captured most objectively by what the security community did with its vulnerability rankings:

- **OWASP Top 10 — 2013:** CSRF was a named entry, **A8-Cross-Site Request Forgery (CSRF)** 🚩 (widely documented; the 2017 release notes reference the 2013 placement by name ✅).
- **OWASP Top 10 — 2017:** CSRF was **removed**, and OWASP's published reason is exactly the framework-and-browser effect: *"A8-Cross-Site Request Forgery (CSRF), as many frameworks include CSRF defenses, it was found in only 5% of applications."* ✅ Note the causal claim OWASP makes — **frameworks**, not browsers. That is the honest attribution for 2017, and it predates the browser default change's completion.
- **The browser change is therefore best understood as reinforcing an already-falling baseline, not as its cause.** It removed the defence-in-depth dependency on application code for the unsafe-method cross-site case, in the browsers that shipped it; it did not remove the vulnerability (§4.6, §6.5).

The accurate summary of the baseline shift, with dates: *between 2007 and 2017 CSRF went from a Top 10 entry to a removal-for-infrequency in OWASP's own estimation ✅; between 2019 and 2023 the browser default and then Fetch Metadata added platform-level controls ✅; and across the same period the credential model of new APIs changed in a way that removed the exposure entirely for those endpoints ✅ — three separate causes with three separate dates, routinely collapsed into the single sentence "browsers fixed CSRF."*

### 8.6 Where the Early Record Is Genuinely Thin

Stated plainly, because the alternative is to fill gaps with plausible narrative:

1. **The 2001 origin.** The attribution to a Bugtraq post of June 2001 remains a secondary-source claim here 🚩 (§8.1). The name and the concept are not in doubt; the specific artefact is unread.
2. **Early incident history is poorly documented — and this guide will not construct it.** The web's early CSRF incidents were mostly reported in mailing lists, conference talks and news posts rather than in structured advisories, and the pre-CVE-numbering era has no reliable incident register to search. The one early incident this guide will cite is the one OWASP cites, with its date and its source: a **2008 uTorrent exploit using a `GET`-based CSRF**, which OWASP describes as *"used on a mass scale to download malware"* ✅, sourced to a January 2008 technology-news write-up 🚩 — i.e. a *named source with a date*, but a news source rather than an advisory. That is the standard this guide applies: **a named source and a date, or no example at all.**
3. **No bank incident is asserted anywhere in this guide.** Deliberately. Claims that "bank X was hit by CSRF in year Y" circulate widely and are almost never traceable to an advisory, and a guide that manufactures one — even a plausible one — corrupts the record it exists to clarify. §10 addresses the banking angle structurally, from the shape of the risk, and the worked example in §11 is explicitly fictional.
4. **The Firefox middle years.** The state of Firefox releases between shipping Lax-by-default in 96 and disabling it on the 111 branch is not fully pinned down by the sources read (§4.6) ⚠.
5. **The current status of the Lax+POST carve-out in shipping browsers.** The specification still defines it; no primary source read this pass establishes whether current Chrome builds still apply it (§4.3) ⚠.
6. **The `SameSite` draft's transformation into an RFC.** The draft read is revision 22 (December 2025) and remained an Internet-Draft ✅; whether and when it publishes as an RFC is outside what this pass verified ⚠.

---

## 9. Testing and Verification

### 9.1 What a CSRF Test Is Actually Trying to Prove

A CSRF test is not trying to prove that a token field exists in the HTML. It is trying to prove **that a request originating from a different site, carrying the victim's live credentials, cannot cause a state change.** That is a behavioural property of a running system under a specific browser condition, and it is the reason scanner output cannot settle the question (§9.3). The test obligation decomposes into four claims, each of which needs its own evidence:

1. Every state-changing operation is reachable only by an unsafe method (§9.2, check 1) — asserted mechanically.
2. The request carries a provenance proof the attacker cannot produce — token, metadata classification, `Origin`, or a combination (§3.9).
3. The proof is *bound* (to the session or to a signature, §5.5) and not merely present.
4. Under a real browser, from a cross-site origin, with a real session, the operation **does not occur** (§9.4).

The repo's existing testing harnesses should be reused rather than rebuilt: [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) for the execution standard and reporting format, [test_orchestration_guide.md](test_orchestration_guide.md) for wiring the checks into a pipeline, and [chaos_engineering_guide.md](chaos_engineering_guide.md) for the failure-injection mindset that makes a "the defence silently stopped working" scenario a rehearsed one rather than a surprise.

### 9.2 The Manual Checks That Matter

| # | Check | How to observe it | Pass condition |
|---|---|---|---|
| 1 | **Method discipline** | Enumerate every route handler that writes state; list its permitted methods | No state-changing handler accepts `GET` or `HEAD` (§2.4). This is the highest-yield single check |
| 2 | **Token presence and binding** | Submit a state-changing request with (a) no token, (b) a syntactically valid but foreign token, (c) a token from a *different* session | All three rejected. (b) catches an unbound or unsigned token; (c) catches a shared/global token |
| 3 | **Token rotation** | Log in, note the token, log out and back in | The token differs (§5.2, Django's per-login rotation) |
| 4 | **Wire-level cookie attributes** | Read the `Set-Cookie` header on the actual response — not the source code | Every session cookie carries an explicit `SameSite`, plus `Secure`, `HttpOnly`, and preferably `__Host-` (§4.5). "Explicit" is the point: an absent attribute is browser-dependent (§4.6) |
| 5 | **`Origin` behaviour** | Send state-changing requests with (a) no `Origin`, (b) `Origin: null`, (c) a foreign origin, (d) your own origin | (a) handled by an explicit documented branch, (b) and (c) rejected, (d) allowed (§3.4). If (b) is allowed, anti-pattern #6 |
| 6 | **Fetch Metadata handling** | Send the same requests with the `Sec-Fetch-*` headers absent, `same-origin`, `same-site`, `cross-site`, and an invalid token value | The absent case must follow a *stated* policy rather than falling through (§3.2); invalid values ignored per spec; `cross-site` denied for unsafe methods |
| 7 | **Exemption inventory** | `grep` the codebase for every exemption — `csrf_exempt`, filter bypasses, `ignored` route configs, custom allowlists | Every hit has a named owner and a written justification (§3.9 rule 2) |
| 8 | **The both-routes handler** | For each endpoint, ask which client surfaces reach it: form, `fetch`, native app, third party | Any handler reachable via a simple request *and* an API path is loaded with both control sets (§5.6) |
| 9 | **Redirect-chain behaviour** | Exercise the SSO and payment-return legs end-to-end | Legitimate cross-site POST returns still work; nothing was "fixed" by exempting a whole controller |
| 10 | **Pre-auth surface** | Submit the login form and the password-reset request cross-site | Rejected (or otherwise protected) — login CSRF is real (§2.6) |

### 9.3 Automated Scanning and its False Positives

DAST scanners find a meaningful subset of CSRF problems and produce a specific, predictable pattern of error, so their output needs interpretation rather than acceptance.

**What scanners are good at:** detecting the *absence* of a token on a form or request, flagging a token that is identical across sessions (unbound), flagging cookies lacking `SameSite`/`Secure`/`HttpOnly` on the wire, and spotting a state-changing `GET` by crawling links. These are all real findings, and the wire-level cookie check in particular is one that a source review will miss.

**Where they go wrong, in both directions:**

- **False positives** dominate the count: a scanner reports "no anti-CSRF token" for endpoints that are legitimately protected by a header the scanner did not send, by Fetch Metadata, or by a bearer token it never acquired. A login form protected by a `SameSite=Strict` cookie and an `Origin` check is reported as vulnerable by scanners that only look for a hidden field. Triaging these consumes the time the real findings needed.
- **False negatives are the dangerous direction.** A scanner will not replicate the browser's credential-attachment behaviour (§9.4), so it cannot distinguish "protected" from "the scanner's request happened to lack the credential the browser would have supplied". It also will not generate a cross-site `<img>` request, will not exercise the Lax+POST window, will not test the `null`-origin case, and will not test the state-changing `GET` unless it happened to crawl the link.
- **The structural limitation:** a scanner is not a browser. It sends requests with whatever credential the tooling holds. The entire CSRF question is *whether the browser supplies a credential the attacker's page cannot*, which is a property of a browser with a live session — not of an HTTP client.

Treat scanner output as a *triage input*, never as evidence of coverage. The repo's [security_by_design_guide.md](security_by_design_guide.md) covers where DAST sits in the tooling pyramid, and [api_governance_guide.md](api_governance_guide.md) covers the API-surface inventory a CSRF scanner needs to be scoped at all.

### 9.4 The Test That Actually Matters — a Real Browser With a Real Session

This is the only test that answers the actual question, because it is the only one in which the browser attaches the credential. The shape is fixed and cheap to automate:

1. **Establish a real session** in a real browser (headless browser automation is fine — the requirement is a real browser engine with a real cookie jar, not a scripted HTTP client).
2. **Serve an attacker page from a genuinely different site** — a different registrable domain, not a different port or a `localhost` alias, because same-site versus cross-site is evaluated on registrable domain plus scheme (§4.5).
3. **From that page, attempt the state change by every vector the attacker actually has:** an auto-submitted `<form method="POST">`, a zero-size `<img src>` against a `GET` endpoint, a `fetch()` with a simple content type, a top-level navigation, and an `iframe` navigation.
4. **Assert the server state is unchanged** — the authoritative assertion, not the HTTP status. A 403 renders the attack ineffective; a 200 that also changed nothing is equally a pass; a 200 that *did* change state is a failure regardless of the response body.
5. **Repeat for each browser engine in your actual customer population.** The point of §4.6 is that engines differ; a single-engine test is a partial answer, and a Firefox result is not predictive of a Chromium result.
6. **Repeat with the cookie's attributes varied** — an unlabelled cookie, then `Lax`, then `Strict`, then `None; Secure` — because the test's *diagnostic* value comes from showing which attribute produces which outcome on the wire. This is also how a team reproduces the §4.7 symptom table deliberately instead of discovering it in production.
7. **Include the client platforms.** For an estate with a mobile channel, run the equivalent against the webview-embedded flow, since the Android-WebView retargeting gate and the iOS client-set-cookie issue (§6.4) mean the mobile result can differ from the desktop result on the same backend.

Where this belongs operationally: as a pipeline gate for the state-changing endpoints that matter, and as a periodic probe for the rest. Given the layered defence model of §3.8, the natural failure-injection test is a *degraded-metadata* test — strip `Sec-Fetch-*` and `Origin` from the request and confirm the *other* layer still holds — which is the CSRF-specific instance of the [chaos_engineering_guide.md](chaos_engineering_guide.md) approach: verify the fallback works by removing the primary.

### 9.5 The Regression Test — Encoding the Defence So It Cannot Silently Rot

The realistic failure mode for a mature application is not "we never added CSRF defence". It is "we added it, and then a change removed it without anyone noticing". The following assertions are cheap, belong in CI, and each one corresponds to a specific anti-pattern in §12:

- **Route assertion:** every handler in the state-changing route table declares only unsafe methods; the test fails on any `GET` mapping to a mutating handler. (Anti-patterns 1 and 7.)
- **Exemption assertion:** the set of CSRF exemptions equals a checked-in allowlist file, one entry per line with an owner. Any new exemption breaks the build until it is added deliberately. (Anti-pattern 4.)
- **Cookie assertion:** parse the actual `Set-Cookie` headers in an integration test and assert `SameSite`, `Secure`, `HttpOnly` and prefix presence explicitly. (Anti-patterns 2 and 5 — this catches the global `SameSite=None` regression that a code review of a single change will miss.)
- **CORS assertion:** assert the `Access-Control-Allow-Origin` behaviour for the origins you own and for a canary foreign origin, and assert that credentials are never combined with a reflected origin. (Anti-pattern 5.)
- **Header-policy assertion:** send a state-changing request with `Sec-Fetch-Site: cross-site` and with the header absent, and assert both outcomes match the documented policy — including that the absent case *denies* rather than falls through. (Anti-pattern 5's sibling.)
- **Browser-behaviour smoke test:** the §9.4 flow, reduced to one vector (the auto-POST) against the two or three highest-value endpoints.

---

## 10. The Regulated-Enterprise and Banking Angle

### 10.1 Why This Class Matters More Than Its Modern Frequency Suggests

CSRF's measured frequency has fallen (§8.5) and its severity in a financial institution has not. Five structural reasons, none of which is about frequency:

1. **The browser-authenticated cookie session is the core of retail banking, deliberately.** A bearer token in script-reachable storage is a worse trade for a regulated institution's threat model, because it converts an XSS bug from "read the DOM" into "exfiltrate the session" (§6.1). So the industry keeps the cookie, on purpose, and with it the exposure.
2. **The actions are high-consequence and largely irreversible.** Adding a payee, changing a registered contact number, resetting a credential, moving money. CSRF's impact ceiling is the victim's own authority (§1.3), and in retail banking that authority is the account.
3. **The attack is fire-and-forget and often silent** (§2.1, step 7). The attacker needs no response, so detection cannot rely on exfiltration traffic; the victim may notice only when a statement arrives.
4. **A CSRF flaw composes with the fraud economy.** An attacker who can move money needs a destination; the payee-management flow exists to create one. A CSRF flaw in payee management plus a CSRF flaw in payments is a complete chain, and §2.7 walked exactly that shape.
5. **The defence is testable and auditable, which means it is a controllable** — unlike many residual risks, this one can be demonstrated as present or absent, which is why §10.5 matters more than the raw risk number.

### 10.2 High-Value State-Changing Actions and the Step-Up Question

The design question for a bank is not "do we deploy CSRF tokens" — that is table stakes and usually framework-defaulted. It is **which actions justify a human-presence requirement in addition** (§3.7). A workable classification, by consequence rather than by technical surface:

| Action class | Examples (generic, fictional) | Defence expectation |
|---|---|---|
| **Credential and contact-detail changes** | password change, 2FA reset, registered mobile number change, email change | CSRF controls **plus** step-up. These are the actions an attacker must take to *retain* control, so the cost of a false negative is the whole account |
| **New payment destination creation** | add payee, add beneficiary, register a device for payments | CSRF controls **plus** step-up, and a cooling-off or out-of-band confirmation for the first payment to a new destination |
| **Value transfer** | domestic transfer, FX, scheduled payment setup | CSRF controls, plus transaction-level authorisation and fraud evaluation (§10.4) for new or unusual destinations |
| **Profile and preference changes** | address, statements, marketing consent | CSRF controls; step-up only where the field feeds authorisation or authentication |
| **Session and device actions** | logout, log out all devices, revoke a device | CSRF controls. Logout CSRF (§2.6) is low-impact alone and high-value to an attacker who wants to force a re-authentication into a flow they control |
| **Pre-authentication** | login submission, password-reset request | CSRF controls — login CSRF (§2.6) is the case most often left open, and the one with no session to protect |

The honest caveat about step-up: it is a *human-presence* control, and human-presence controls degrade under repetition. Placing it on too many actions trains the customer to approve without reading, which converts a security control into a compliance artefact. Place it on the small set where the cost of a false negative is catastrophic, and measure the approval-fatigue rate as a security metric rather than a UX complaint.

### 10.3 Mobile-Channel Differences

The mobile channel materially changes the CSRF picture, and the difference is mostly favourable — with two traps:

**Favourable:** a native app constructs its own requests, so the browser's credential-attachment rule does not apply to the app-to-API leg (§6.4). The classical attack has no foothold there, which means the mobile channel is often the *safer* channel for the same business functions, and the risk concentration sits on the web channel. Two operational consequences follow: the web channel is where CSRF testing should concentrate, and **the endpoints the app calls are still the endpoints a browser can call** — so server-side protection must be scoped by surface, never switched off globally because "the app doesn't need it".

**Trap 1 — the webview.** Any part of the mobile journey rendered in a webview is a browser context again, subject to the browser's rules and to the platform's `SameSite` gating (§6.4, Android WebView retargeting). A hybrid journey that is CSRF-safe in its native legs and CSRF-exposed in its webview legs is common, and the boundary usually falls at exactly the third-party or redirect hand-offs where the risk is highest.

**Trap 2 — the app-set cookie.** An app that injects cookies into a webview's cookie store is relying on platform defaults for the `SameSite` policy, which is the failure WebKit bug 279153 documents (§6.4). Set the policy explicitly from the app, and retest on every OS major version.

**Trap 3 — device-level trust.** Where the mobile channel uses hardware-backed device binding or app attestation, it is tempting to treat the mobile channel as inherently intent-verified. It is not: device binding proves *which device*, not *which human caused this request*, and a compromised or malicious app running on a bound device is a different problem from CSRF but not a smaller one. The [distributed_auth_guide.md](distributed_auth_guide.md) owns the device-binding mechanics.

### 10.4 Fraud Monitoring as a Complement, Not a Substitution

Fraud monitoring is often proposed as the answer to CSRF in a banking context, on the reasoning that a bank detects and blocks anomalous payments anyway. It is a genuine and valuable complement, and it is **not** a CSRF control. The distinction is structural, not a matter of maturity:

- **CSRF defence prevents the forged request from being authorised; fraud monitoring evaluates a request that has already been authorised** and decides whether to allow its effect. The forged payment is, from the server's view, an ordinary authenticated customer request — which is precisely the property that makes CSRF hard, and precisely the property that makes it *look normal* to most fraud rules.
- **The two have different failure modes and therefore genuinely compose.** A CSRF defence can fail open on a legacy client; fraud rules can fail to fire on a genuinely in-pattern transfer. Neither substitutes for the other.
- **Where fraud monitoring genuinely extends the CSRF defence**, and this is worth designing deliberately: behavioural signals that a CSRF attack perturbs — a new payee added and paid within an unusually short interval, a session that performs a payee-creation flow without the usual intervening page views, a first payment to a destination whose beneficiary details were changed recently, an action sequence that skips the confirmation step. These are *CSRF-shaped* fraud features, and they are cheap to add to rules that probably already exist. The pattern in §2.7 — a multi-step flow whose steps are individually forgeable — is exactly the flow where these signals carry the most information, which is the honest version of the claim that multi-step flows help: **they help by giving monitoring more observation points, not by preventing the forgery.**
- **What it must not become:** the argument that "we have fraud monitoring, so CSRF is covered". That argument fails the diagnostic in §6.5 — ask which control stops the forged request from being authorised. If the answer is "none", monitoring is compensating for a control gap, and compensating controls drift.

### 10.5 The Evidence and Audit Angle — Tested, Not Asserted

In a regulated estate the CSRF control has to be *demonstrable*, and the distinction that matters is between asserting a control and evidencing it. Three levels, in ascending order of audit value:

1. **Asserted** — "our framework enables CSRF protection by default." True, and worth nothing on its own, because the exemption list is where the holes live and it is not in this statement.
2. **Configured** — the wire-level evidence: the actual `Set-Cookie` headers, the actual route method table, the actual exemption list with owners, the actual CORS origin response. This is the level at which most internal reviews stop, and it is where §9.5's CI assertions pay for themselves, because they turn "configured" into something continuously verifiable rather than point-in-time claimed.
3. **Tested** — the §9.4 result: evidence, recorded with a date and a browser version, that a cross-site request against a live session did not produce a state change on the state-changing endpoint set. This is the level a penetration test or an audit can rely on, and the standard the repo's [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) already sets for execution evidence.

Two audit-shaped consequences worth stating explicitly. **First, the exemption list is the artefact auditors should ask for**, not the framework configuration — it is the direct analogue of the firewall rule review, and an unowned exemption is the finding. **Second, "the browser handles it" is not an acceptable control description** in a regulated context, for the reason §4.6 establishes: it is version-dependent, browser-dependent and regression-prone, and a control that is asserted to work in browsers that do not implement it is not a control. Describe the control as what *you* do — the explicit attribute you set, the policy you enforce, the test you run — and cite the browser behaviour as a supporting layer.

### 10.6 The Third-Party and Agency Question

Financial applications are assembled, not built — vendor products, agency-delivered front ends, white-labelled journeys, third-party widgets, and hosted components on a shared registrable domain. Four CSRF-shaped questions follow, and they belong in the vendor assessment, not in a post-delivery review:

1. **Whose cookie is it?** If a component sets a cookie on your registrable domain, it participates in your CSRF surface and in your sibling-subdomain trust boundary, whatever its own security posture is. A component hosted on a *subdomain* of your banking domain inherits site-scope trust (§3.3, §4.6) — which is a structural reason to host third parties on a separate registrable domain, and a question worth asking before the contract rather than after the integration.
2. **Does the component require `SameSite=None`?** Many embedded integrations do, and the request to widen the cookie's scope is the point at which a project adds a global opt-out (anti-pattern #2). The correct response is a scoped `None; Secure` on the cookie that genuinely needs it, plus a compensating token — never a global change, and never a change without the compensating control.
3. **Who tests the integrated flow?** Each party will have tested its own component inside its own perimeter. The CSRF question is about the *composition* — the redirect hand-off, the embedded frame, the form posted between domains — and it is precisely the seam nobody owns. Make the seam an explicit test case with a named owner (§9.4, step 7).
4. **Can you demonstrate the control over a component you do not operate?** §10.5's tested-level evidence has to cover the assembled journey, which means either a contractual testing right or a compensating test you run yourself against the live composition. Where the answer is "the vendor says so", the evidence level is "asserted".

---

## 11. The Cymbal Bank Worked Example

**Everything in this section is explicitly ILLUSTRATIVE and FICTIONAL.** Cymbal Bank is a fictional persona used across this repository for worked examples. No real institution is described, no real incident is referenced, and every figure, endpoint, timeline and cost in this section is invented to demonstrate a method. The numbers are not benchmarks and must not be reused as such.

### 11.1 The Scenario

A fictional retail online-banking platform at Cymbal Bank runs two front ends against one backend:

- **`cymbal-retail-web`** — the established server-rendered application. Session cookie authentication, HTML forms, a decade of accumulated controllers.
- **`cymbal-spa`** — a newer single-page application for the same customer base, calling the same backend APIs.
- **A partner integration** — a fictional wealth-management widget hosted on a subdomain of the retail banking domain, which requires cross-site cookies to function.
- **A mobile app** — native, calling the same APIs, with two webview-rendered journey legs (an onboarding hand-off and a payment-return leg).

A security review has been triggered by the SPA programme's completion, and the question put to the team is: **decide the defence stack.** The review is deliberately structured as the sequence a real review takes — inventory, selection, configuration, policy, sequencing, test, cost, decision — because the value of the worked example is the sequence, not the fictional answers.

### 11.2 Step 1 — the Endpoint Inventory

The first artefact is not a control. It is a table of state-changing endpoints with their methods and their consumers, because every later decision is scoped to it. The fictional subset that drove the decisions:

| Endpoint (fictional) | Method | Consumers | Reversible? | Notes |
|---|---|---|---|---|
| `/transfer` | `POST` | web, SPA, mobile | No | Core payment |
| `/payees` (create) | `POST` | web, SPA, mobile | Yes (delete) | Destination creation — the enabler |
| `/payees/confirm` | `POST` | web, SPA | Yes | Step 2 of the §2.7 shape |
| `/payees/{id}/preview` | **`GET`** | web, SPA | — | **Found to write a "previewed" state and to advance the flow.** The critical finding |
| `/profile/contact` | `POST` | web, SPA | Yes | Registered contact number — a retention primitive |
| `/password` | `POST` | web | No | Step-up already required |
| `/logout` | **`GET`** | web, SPA, mobile | Yes | Classic logout CSRF; low impact, useful as a probe |
| `/session/revoke-all` | `POST` | web, SPA | No | Reverse of the retention primitive |
| `/api/v1/…` (SPA surface) | `POST`/`PUT`/`DELETE` ×N | SPA, mobile | Mixed | `Content-Type: application/json` documented; **not enforced** at the edge |
| `/partner/…` | `POST` | third party | Mixed | Requires cross-site cookies; the integration that motivated the audit |

Three findings came out of the inventory alone, before any control was discussed, and they are the reason the inventory is step one:

1. **`/payees/{id}/preview` is a state-changing `GET`** — the anti-pattern §2.4 warns about, sitting inside the highest-value flow in the application. Under any browser default that permits cross-site top-level safe-method navigations (§4.4), this is exploitable.
2. **`/logout` is a `GET`** — separately exploitable, and the cheapest available probe for whether *any* CSRF defence is effective on this estate.
3. **The API surface documents a JSON content type but does not enforce it at the edge** — so the "non-simple request" property that the SPA's security argument depends on (§6.2) was a convention, not a constraint.

### 11.3 Step 2 — Defence Selection per Endpoint Class

The selection was made per class, not globally, because the surfaces genuinely differ (§3.9):

| Endpoint class | Token | `SameSite` | Fetch Metadata | Origin check | Custom header | Step-up |
|---|---|---|---|---|---|---|
| `cymbal-retail-web` forms | **Synchronizer token** (framework-provided) — mandatory | `Lax` + `Secure` + `HttpOnly` + `__Host-` | Default-deny, log-only first | Yes, as the absent-metadata fallback | n/a | On the high-value set |
| `cymbal-spa` → API | **Token in a custom header**, signed and session-bound | `Lax` where same-site; `None; Secure` only where it must be | Default-deny with a documented API carve-out | Yes | **Required** — plus a strict CORS allowlist | On the high-value set |
| Third-party `/partner/…` | Token where the partner can carry one; otherwise an out-of-band signature | Scoped `None; Secure` on the specific cookie only | **Explicitly exempt, by name** | Yes | Where the partner can | n/a |
| Pre-auth (`/login`) | **Added** — it was missing | — | Default-deny | Yes | n/a | n/a |
| Mobile-native legs | n/a on the native leg; browser rules apply in webviews | `Lax` `Secure` on server-set cookies; app-set cookies labelled explicitly from the app | n/a | Yes | n/a | On the high-value set |

The high-value set that receives step-up: `/password`, `/payees` (create), a first payment to a newly created payee, and `/profile/contact`. That is the §10.2 classification applied — the four actions an attacker needs in order to *retain* control or to *create a destination*.

### 11.4 Step 3 — the SameSite and Token Configuration

The concrete configuration decisions the review recorded, in the form they would go into an ADR:

- **Explicit labelling everywhere.** No cookie on the estate relies on the browser default, for the reasons in §4.3 and §4.6. Every `Set-Cookie` carries an explicit `SameSite`.
- **Session cookies:** `SameSite=Lax; Secure; HttpOnly`, `__Host-` prefix, host-only (no `Domain`). `Lax` rather than `Strict` because inbound links from email and search must land authenticated — a product constraint, recorded as such.
- **Token cookie (SPA surface):** `SameSite=Lax; Secure`, **no** `HttpOnly` (the client must read it), `__Host-` prefix, **signed and bound to the server-side session ID** — the §5.5 requirement that the signed variant is worthwhile only with session binding. The naive double-submit was rejected on the strength of §5.4: the estate cannot assume every subdomain is trusted, partly because of the partner integration.
- **Partner cookie only:** `SameSite=None; Secure`, scoped to that one cookie, with the token as the compensating control and the exemption recorded in the reviewed allowlist.
- **Token delivery per surface:** hidden field for forms; `meta` tag plus custom header for the SPA; per-login rotation on both (cheap, and it closes token fixation — §5.2).
- **Statelessness rejected for the API.** The team considered a signed-cookie-only design to avoid a session store, and rejected it because the estate already has a session store and had no appetite for adding key distribution as a new availability dependency (§5.5, §5.7).

### 11.5 Step 4 — the Fetch Metadata Policy

The policy was written as default-deny with explicit allow rules, and deployed log-only first, in the three modes OWASP recommends:

1. **Log-only for a full release cycle.** Every request that would have been denied is logged with its `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, method, path and user-agent, and analysed for false positives. The purpose of this phase is to *measure* the absent-header population rather than assume it (§3.2).
2. **Enforce with a documented absent-header branch.** The branch was configured as **deny** for the state-changing endpoint set and **fall back to the Origin/token checks** for the rest — the fail-safe option for sensitive endpoints, the fail-open-with-fallback option elsewhere. The requirement recorded was that the branch be explicit and covered by a test, because a deny-list implementation would have silently disabled the whole policy (§9.5).
3. **`same-site` handled conservatively**, not trusted by default, for the reason §3.3 gives: the estate shares a registrable domain with the partner subdomain. `same-site` requests to state-changing endpoints therefore still require the token. This was the single most consequential policy decision in the review, and it exists purely because the estate's subdomain map is not fully trusted.

The redirect-chain hazard (§3.2) was handled by allowlisting the two named return legs rather than by relaxing the policy globally — the difference between a two-line exemption and a whole-application carve-out.

### 11.6 Step 5 — Migration Sequencing and Compatibility Risk

Sequencing was chosen to *fail loudly in non-production* rather than subtly in production, on the principle that the compatibility risk is concentrated in clients the estate cannot observe directly:

| Phase | Change | Compatibility risk | Mitigation |
|---|---|---|---|
| 1 | Fix the state-changing `GET`s (`/payees/{id}/preview` → `POST`; `/logout` → `POST` with a form) | Email templates and SPA deep links pointing at the old `GET`s | Dual-support window with a redirect-and-prompt, then removal; both methods logged during the window |
| 2 | Explicitly label every cookie (no behaviour change intended) | A cookie that was silently relying on a permissive default may now be restricted | Deploy with the attributes that preserve *current observed* behaviour first, then tighten deliberately in phase 4 |
| 3 | Add the missing token to the pre-auth surface; enforce the JSON content type at the edge | Non-browser API clients sending the wrong content type | Inventory of API clients first; enforce in log-only, then by client cohort |
| 4 | Enforce Fetch Metadata (after the log-only cycle); tighten `SameSite` where the measurement supports it | The absent-header and redirect-chain populations — concentrated in older browsers, embedded clients, and the webview legs | Named exemptions per leg; feature-flag the policy; measure the failure rate by client class |
| 5 | Enforce the custom header on the SPA surface; narrow the CORS allowlist to the actual origins | The SPA must be redeployed in step; the partner integration must be updated | Coordinated release; canary; the token as the compensating control during the transition |

The two compatibility risks the team recorded as *accepted and watched* rather than eliminated: **the absent-header population** (older and embedded clients, whose exact size the log-only phase measured but could not reduce), and **the webview legs** (§6.4), where the platform's `SameSite` behaviour depends on the app's target SDK and on OS version.

### 11.7 Step 6 — the Testing Plan

The plan was the §9 structure, scoped to this estate:

- **CI assertions** (§9.5) for method discipline, the exemption allowlist, wire-level cookie attributes, and the CORS origin behaviour — on every merge.
- **The real-browser cross-site suite** (§9.4) against the state-changing endpoint set, on Chromium, Firefox and WebKit, with the three high-value endpoints in the smoke set — run nightly.
- **The degraded-metadata test** — strip `Sec-Fetch-*` and `Origin` and confirm the token layer still denies (§9.4, step 7's chaos framing).
- **A recurring penetration test** of the assembled journey — the seam between `cymbal-retail-web`, `cymbal-spa` and the partner component, which is the part no single team owns (§10.6) — using the repo's execution standard.
- **The wire-level cookie audit** as a weekly automated check, because a single unrelated change adding a global `SameSite=None` is the most plausible way this estate silently loses a defence layer (§12, anti-pattern 2).

### 11.8 Step 7 — the Options Comparison (ILLUSTRATIVE figures)

**Every figure below is fictional and illustrative.** They are shaped to demonstrate that the comparison is a *cost-and-residual* trade rather than a technical ranking, not to be reused or benchmarked. Figures are in a fictional currency and represent engineering effort, not money in any real currency.

| Option | Effort (ILLUSTRATIVE) | Residual exposure (ILLUSTRATIVE) | Main risk of this option |
|---|---|---|---|
| **A. Do nothing but rely on framework defaults** | 0 | Highest | The state-changing `GET`s remain exploitable regardless of framework defaults; the SPA surface has no defence at all; the partner cookie is global. Not viable — listed for completeness |
| **B. Tokens only (all surfaces), no metadata policy** | Low | Moderate | The state-changing `GET`s remain exploitable unless fixed separately; the token is defeated by XSS (§5.4, §7.2); no browser-enforced layer at all (§3.8) |
| **C. Metadata policy + tokens + explicit `SameSite`, sequenced (chosen)** | Moderate–high | Lowest of the options | Delivery risk concentrated in the compatibility window (§11.6); requires the `GET` remediation and the partner integration change |
| **D. Re-architect the SPA surface to bearer tokens** | High | Near-zero for the API surface; unchanged for the server-rendered web surface | Large blast radius; moves the XSS exposure to credential exfiltration (§6.1); does not help the partner integration at all |
| **E. `SameSite=None` everywhere plus a token, shortest path** | Low | Moderate–high | Silently discards the browser-enforced layer (§3.9); the token is the only control and is defeated by XSS; the subdomain trust problem is untouched |

Option D is the interesting rejection, and worth recording because it looks like the most modern answer. The team rejected it **not** on CSRF grounds — it would genuinely eliminate CSRF on that surface — but because it does not address the server-rendered surface where most of the business lives, and because it trades a CSRF exposure for a credential-exfiltration exposure that the institution weights more heavily (§10.1, reason 1). That is a defensible trade recorded explicitly, not a default.

### 11.9 The Recommendation, the Re-architecture, and the Accepted Residual

**The recommendation:** Option C — the layered stack of §3.9 applied per surface, sequenced as §11.6, with the `GET` remediation as a prerequisite rather than a parallel workstream, and the step-up set as §10.2 defined.

**What the team decided to RE-ARCHITECT rather than defend.** Two things, and both are deliberate choices to remove the exposure rather than mitigate it:

1. **The payee-creation and confirmation flow was re-architected**, not merely protected. The state-changing `GET` was removed, the steps were collapsed into a single idempotent `POST` with an explicit idempotency key, and the "preview" state was eliminated. The reasoning: the flow's multi-step structure was providing *no* security benefit (§2.7) while multiplying the surface and complicating the fraud signals (§10.4); collapsing it reduced both the CSRF surface and the number of places a forged request could land. **This is the guide's clearest example of re-architecture over defence** — the team removed the opportunity rather than protecting it.
2. **The partner integration was re-hosted onto a separate registrable domain.** This eliminated the need for a `SameSite=None` cookie and removed the partner subdomain from the estate's site-scope trust boundary entirely (§3.3, §10.6). It was the larger piece of work of the two and the one that removed the most residual risk, because it closed both the subdomain-trust problem and the global-cookie-scope temptation at once.

**The residual exposure the team accepted, and the reason.** Three items, each with a written justification — which is what "accepted risk" is supposed to look like, as opposed to what it usually is:

1. **The absent-`Sec-Fetch-*` population.** Accepted for the non-critical endpoint set, mitigated by the Origin/token fallback for that set and by the fail-safe deny on the state-changing set. **Reason:** the population is concentrated in clients the bank cannot upgrade and cannot refuse, and the fallback controls demonstrably hold on that path (verified by the degraded-metadata test). Accepted *with* the measurement, not instead of it.
2. **The webview legs.** Accepted pending the mobile roadmap's target-SDK upgrade, because the `SameSite` behaviour in Android WebView is gated on the app's target version (§6.4) and the app cannot be forced to retarget on the bank's schedule. Mitigated by explicit `SameSite` labelling on every server-set cookie, which removes the dependence on the platform default for the cookies the bank controls. **Reason:** the exposure is bounded, the mitigation is real, and the change is already on a funded roadmap.
3. **XSS as the boundary condition.** Accepted as *out of scope for this review* and recorded as an explicit dependency: no CSRF control in the deployed stack survives a compromised origin (§3.8, §7.2), so the CSRF review's residual is conditional on the XSS review's outcome. **Reason:** stating the dependency is the point. A CSRF sign-off that does not name XSS as its boundary condition is signing off on a control whose defeat condition it has not identified.

---

## 12. Anti-Patterns and Misconfigurations

Symptom, cause, guardrail. Each of these is a real pattern; the guardrail is the mechanism that prevents recurrence rather than the advice to be careful.

| # | Symptom | Cause | Guardrail |
|---|---|---|---|
| 1 | A state-changing operation that "works as a link" — usually in an email, a newsletter, or a confirmation flow | A `GET` handler that mutates state. The designer wanted a link rather than a form | CI assertion that no mutating handler accepts `GET`/`HEAD` (§9.5); email templates reviewed as code; a redirect-and-prompt for the legacy links (§11.6) |
| 2 | A single unrelated change breaks every login, and the fix is to set `SameSite=None` globally | An integration needed a cross-site cookie; the shortest fix was a global config change. Now the browser-level defence is off estate-wide | Scope `None; Secure` to the named cookie; require a compensating token and a review signature for any global cookie-attribute change; the wire-level cookie assertion in CI catches the regression (§9.5) |
| 3 | CSRF protection passes in testing and fails in production after a load-balancer change | The token is not bound to the session — it is a per-node or per-deployment secret, so a request reaching another node fails validation, or worse, a global token validates anywhere | Bind the token to the session (§5.5); the "foreign token from a different session must be rejected" test (§9.2, check 2b/c) |
| 4 | One endpoint has no protection; a review of the framework configuration shows protection "enabled" | An exemption added to make an integration work, never revisited. Django's `csrf_exempt`, a filter bypass, a route attribute | Exemption allowlist checked into the repo, one entry per line, with an owner, enforced by CI (§9.5, §10.5) |
| 5 | "CSRF is handled by our CORS policy" | The §7.3 conflation — CORS treated as a send restriction | The distinction documented in the ADR; the canary-origin CORS assertion in CI; the §7.5 misstatement list circulated in review |
| 6 | The Origin check accepts `null` | A pragmatic accommodation for a small percentage of legitimate traffic, then permanent | Reject `null` on state-changing endpoints and log the rate; scope any `null` acceptance to genuinely side-effect-free `GET`s only (§3.4) |
| 7 | `POST` endpoints are protected, but `PUT` and `DELETE` are not | Protection implemented per-endpoint or per-method rather than by an unsafe-method default. Often the `PUT`/`DELETE` handlers arrived later | Protect by method class, not per handler; assert the coverage of the full unsafe-method set in CI (§9.5) |
| 8 | "We use JWTs, so we're immune to CSRF" — and the endpoint still authenticates via a session cookie | A design assertion that was never verified against what the server actually validates (§6.5) | Ask the diagnostic question per endpoint: *which credential does the server validate here?* If it is a cookie, the endpoint is in scope. Record the answer in the endpoint inventory (§11.2) |
| 9 | The token is present, correct, and identical for every user | The token was treated as a global constant or a deployment secret rather than a per-session value | Token must be unique per session, secret, unpredictable (§5.1); the cross-session token test (§9.2, check 2c) |
| 10 | The defence works in the desktop browser and not in the mobile app's webview | The platform's `SameSite` behaviour is gated on the app's target SDK or OS version (§6.4) | Label cookies explicitly from both server and app; include the webview legs in the browser test matrix (§9.4, step 7) |
| 11 | The Fetch Metadata policy "is deployed" but blocks nothing | Written as a deny-list (`if site === 'cross-site' reject`) rather than default-deny, so the absent-header case and the invalid-value case both fall through | Default-deny with an explicit, tested absent-header branch (§3.2, §9.5) |
| 12 | A security review passes, and six months later the same finding returns | The control was configured but not asserted; no regression test encoded the invariant | §9.5's CI assertions, and the periodic cross-site probe as the standing evidence (§10.5, level 3) |

**The pattern underneath the list**, worth naming because it is the reason these anti-patterns recur rather than being one-off mistakes: **eleven of the twelve are not missing controls but degraded ones.** The estate had a defence; a change — usually motivated by a genuine compatibility problem — weakened it in a way that no test was watching. That is the argument for §9.5 being the most operationally important section of this guide, and it is the same argument the [security_by_design_guide.md](security_by_design_guide.md) makes about secure defaults needing enforcement rather than documentation.

---

## 13. The Claims Audit

The highest-risk claim classes in this topic are, in order: **the browser default change** (its value, its first browser, its version, its year, its rollout); **every version number and date**; **the CWE identifier and its composition**; **specification details** (which value is normative, which is a permissive user-agent option); and **every browser-support claim**. Each is treated accordingly below. Verification date for the ✅ rows: **September 2026**. Legend: ✅ verified against the named primary source this pass; ⚠ partially verified or unattributable; ❌ rejected as stated elsewhere; 🚩 widely repeated but not pinned to a primary artefact read this pass.

| # | Claim | Status | Source (and date) | Quality note |
|---|---|---|---|---|
| 1 | CSRF is CWE-352; MITRE's description turns on whether a request was *"intentionally provided by the user"* | ✅ | MITRE CWE-352, page last updated 30 April 2026 | Highest-risk class (weakness identifier). Direct quotation |
| 2 | CWE-352 is a **Composite** requiring CWE-346, CWE-441 and CWE-613 | ✅ | MITRE CWE-352, Composite Components table | Highest-risk class. Governs §1.3 and §8.1 |
| 3 | CWE-352 exists at least since CWE 1.0 (September 2008) | ✅ | MITRE CWE-352 change history (entry dated 2008-09-08, CWE 1.0) | Version-and-date class |
| 4 | CWE-352's alternate terms include Session Riding, XSRF, Cross Site Reference Forgery | ✅ | MITRE CWE-352, Alternate Terms | Terminology class |
| 5 | **Chrome was the first browser to enforce a Lax-equivalent default; version Chrome 80; year 2020** | ✅ | Chromium "SameSite Updates" launch timeline (page last updated 18 March 2021) | **The pivotal fact.** Chrome 80 Stable released 4 Feb 2020; enforcement began rolling out from the week of 17 Feb 2020 |
| 6 | Chrome 80 Stable's **initial** release did **not** include the enforcement | ✅ | Chromium "SameSite Updates", 4 Feb 2020 entry | Corrects the common "Chrome 80 shipped it" simplification |
| 7 | Enforcement was **rolled back on 3 April 2020** (COVID-19) and resumed **14 July 2020** | ✅ | Chromium "SameSite Updates", 3 April / 28 May / 14 July 2020 entries | Rollout-shape class |
| 8 | Rollout reached a target of **100% of Chrome Stable 80+** on **11 August 2020** | ✅ | Chromium "SameSite Updates", 11 Aug 2020 entry | Version-and-date class |
| 9 | Chrome's own pre-announcement: cookies with no `SameSite` treated as `Lax` **but still included in POST requests** | ✅ | Chromium "SameSite Updates", 26 Sept 2019 entry | Quoted verbatim in §4.1 |
| 10 | The Lax+POST carve-out window is **2 minutes** from cookie creation | ✅ | Chromium "SameSite Updates" test/debug guidance; **and** RFC 6265bis §5.6.7.2 | Source-confirmed twice, independently |
| 11 | Chromium described Lax+POST as *"a temporary intervention"* to be removed "some time after" Chrome 80 | ✅ | Chromium "SameSite Updates" FAQ text | Quoted. **Its current Chrome status is ⚠ — see §14** |
| 12 | The carve-out was **codified in the specification** as "Lax-Allowing-Unsafe" enforcement, applying only to cookies that did **not** explicitly specify `SameSite` | ✅ | RFC 6265bis §5.6.7.2 (rev. 22, December 2025) | **Specification-detail class.** Underpins the guide's "label explicitly" recommendation |
| 13 | The spec's own words: Lax-allowing-unsafe *"should be seen as a temporary, transitional measure"* | ✅ | RFC 6265bis §8.8.6 | Direct quotation |
| 14 | **The change is FROM `same-site-flag = "Default"` treated as unrestricted TO `"Default"` subject to an enforcement mode equivalent to `Lax`** | ✅ | RFC 6265bis §4.1.2.7 and §5.7 step 17 | **The most commonly mis-stated claim in the topic.** §4.1 resolves it |
| 15 | **RFC 6265 (April 2011) defines no `SameSite` attribute at all** | ✅ | RFC 6265 itself (base specification) | Corrects the widespread "changed from `SameSite=None`" framing |
| 16 | The spec defines a **default enforcement mode**, and permits `Lax-allowing-unsafe` — it does **not** mandate `Lax` | ✅ | RFC 6265bis §4.1.2.7 | Specification-detail class. Explains the browser divergence |
| 17 | `Strict` = same-site only; `Lax` = same-site + cross-site top-level **safe-method** navigations; `None` = both | ✅ | RFC 6265bis §4.1.2.7; MDN `Set-Cookie` | Specification-detail class |
| 18 | **`SameSite=Lax` still permits top-level `GET` navigation — so a state-changing `GET` remains exploitable** | ✅ | RFC 6265bis §5.6.7.1; MDN `Set-Cookie`; OWASP cheat sheet | **The crucial nuance.** §4.4 |
| 19 | The spec itself says Lax is *"reasonable defense in depth"* but *"does not offer a robust defense against CSRF as a general category of attack"* | ✅ | RFC 6265bis §5.6.7.1 | Direct quotation; corrects "Lax solves CSRF" |
| 20 | `SameSite=None` requires `Secure`, else the cookie is rejected outright | ✅ | RFC 6265bis §5.7 step 19; Chrome Platform Status feature 5633521622188032 | Source-confirmed twice |
| 21 | The None-requires-Secure feature was **available behind a flag as of Chrome 76** and rolled out to Stable from **14 July 2020** | ✅ | Chrome Platform Status, feature 5633521622188032 | Version-and-date class |
| 22 | "Schemeful same-site" — the scheme is part of the site comparison | ✅ | MDN `Set-Cookie` ("including the scheme"); Chromium groups it with the modern behaviour | Browser-behaviour class |
| 23 | `__Host-` requires `Secure`, `Path=/` and **no `Domain`**; `__Secure-` requires `Secure` | ✅ | MDN `Set-Cookie`; RFC 6265bis §5.7 steps 20–21 | Specification-detail class |
| 24 | **Firefox does NOT currently implement Lax-by-default; Mozilla states unlabelled cookies default to `None`** | ✅ | Mozilla Bugzilla 1617609 — RESOLVED WONTFIX (`firefox124: wontfix`), engineer comment: *"the current behaviour is when no SameSite attribute is set we use None by default"* | **The most valuable correction in this guide.** §4.6 |
| 25 | Mozilla **disabled** the pref: bug 1812297 RESOLVED FIXED, milestone "111 Branch" | ✅ | Mozilla Bugzilla 1812297 (description: *"too many issues with lax-by-default to consider shipping"*) | Version-and-date class |
| 26 | Firefox 96 (released **11 January 2022**) was documented as enforcing Lax-by-default | ✅ | Firefox 96.0 release notes (11 Jan 2022); Firefox for Enterprise 96 release notes; Bugzilla 1617609 tracking `firefox96: verified` | Version-and-date class. **The 96→111 transition is only partly explained — ⚠ §14** |
| 27 | Firefox's earlier attempt was a **Beta staged rollout across Fx 79–81 (2020)** | ✅ | Mozilla Bugzilla 1622091, as cited in bug 1617609's dependency list | Version-and-date class |
| 28 | **WebKit adopted Lax-by-default in the iOS 18 / Safari 18 (2024) window** | ✅ (window) / ⚠ (exact version) | WebKit Bugzilla 278353 (*"new SameSite=Lax by default cookie behavior"*); 279153 (reported 4 Sept 2024) | Browser-behaviour class. **Exact first version not pinned — §14** |
| 29 | WebKit's change broke client-set `SameSite=None` cookies in hybrid apps; *"SameSite=None cookies by default have been restored in iOS 18.0.1"* | ✅ | WebKit Bugzilla 279153, comment 17 (3 October 2024) | Direct quotation with a date |
| 30 | **OWASP's claim that "Firefox and Edge have followed suit" on the Lax default is inaccurate as regards Firefox** | ❌ | OWASP CSRF Prevention Cheat Sheet (as published) vs Mozilla bug 1617609 | **A rejected claim from a reputable source.** Stated in §4.6 with the correction |
| 31 | Chrome on iOS was *not* affected by the SameSite behaviour change | ✅ | Chromium "SameSite Updates" | Browser-support class |
| 32 | Android WebView's modern SameSite behaviour applies by default only to apps **targeting Android 12+** | ✅ | Chromium "SameSite Updates", 8 January 2021 entry | Browser-support class |
| 33 | Chrome's enterprise policies reverted legacy behaviour, stated lifetime edited to *"until at least July 14, 2021"* | ✅ | Chromium "SameSite Updates", 10 Feb 2020 entry (edited 29 May 2020) | Version-and-date class |
| 34 | SameSite flags removed from `chrome://flags` as of **Chrome 91**; CLI flag removed in **Chrome 94** | ✅ | Chromium "SameSite Updates", 18 March 2021 entry | Version class |
| 35 | **The Fetch Metadata headers are exactly `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User`** | ✅ | Fetch Metadata Request Headers, W3C WD 1 April 2025, §2 | Specification-detail class |
| 36 | `Sec-Fetch-Site` enumerable values: `cross-site`, `same-origin`, `same-site`, `none`; `Sec-Fetch-Mode`: `cors`, `navigate`, `no-cors`, `same-origin`, `websocket` | ✅ | Fetch Metadata spec §2.2–2.3 | Specification-detail class |
| 37 | `Sec-Fetch-Site: none` means a navigation *"explicitly caused by a user's interaction with the user agent (by typing an address… or by clicking a bookmark)"* | ✅ | Fetch Metadata spec §2.3, step 4 | Direct quotation |
| 38 | `Sec-Fetch-User` is *"delivered only for navigation requests, and only when its value is `true`"* | ✅ | Fetch Metadata spec §2.4 | Direct quotation |
| 39 | Servers **SHOULD ignore** a Fetch Metadata header containing an invalid value (forward-compatibility) | ✅ | Fetch Metadata spec §2.1–2.4 | Specification-detail class |
| 40 | In a redirect chain, `Sec-Fetch-Site` degrades across the whole URL list and does **not** recover — a chain returning to the original origin still reports `cross-site` | ✅ | Fetch Metadata spec §4.1, with worked example | Specification-detail class. Under- appreciated compatibility hazard |
| 41 | Headers are sent only to **potentially trustworthy URLs** (HTTPS, `wss`, `file`, localhost) | ✅ | Fetch Metadata spec §3; OWASP cheat sheet "Requirements" | Specification-detail class |
| 42 | Fetch Metadata support: `Sec-Fetch-Site`/`Mode`/`User` from Chrome 76 / Edge 79 / Firefox 90 / Safari 16.4; `Sec-Fetch-Dest` from Chrome 80 / Firefox 90 / Safari 16.4 | ✅ | Browser-support data in the Fetch Metadata spec's MDN references | **Browser-support class — treated as highest risk.** Note `Sec-Fetch-Dest` starts at Chrome 80, not 76 |
| 43 | OWASP summarises Fetch Metadata support as *"all major browsers since March 2023"* with *"over 98% global coverage"* | ✅ (attributed) | OWASP CSRF Prevention Cheat Sheet, citing caniuse | **Attributed to OWASP; not independently re-measured by this guide.** Stated as such in §3.2 |
| 44 | Fetch Metadata must be paired with a fallback: *"a fallback to standard origin verification headers **is a mandatory requirement**"* | ✅ | OWASP cheat sheet, §"Fetch Metadata headers" | Direct quotation |
| 45 | The recommended absent-header handling is **fail-safe (block) for sensitive endpoints**, or fail-open with a fallback — explicitly, not by default | ✅ | OWASP cheat sheet, §"How to treat Fetch Metadata headers on the server-side", steps 2.1–2.2 | Direct quotation. §3.2's absent-header argument |
| 46 | `Vary: Sec-Fetch-Site, Origin` is recommended for caching but *"does not impact CSRF defenses in any way"* | ✅ | OWASP cheat sheet, §"Rollout & testing recommendations" | Direct quotation; avoids over-claiming |
| 47 | **XSS defeats all CSRF mitigation techniques** | ✅ | OWASP cheat sheet (opening warning); CWE-352 mitigations (*"can be bypassed using XSS (CWE-79)"*); Django `ref/csrf` | Source-confirmed three times. §7.2 |
| 48 | The **synchronizer token** must be unique per session, secret and unpredictable; and must **not** be transmitted in a cookie | ✅ | OWASP cheat sheet, §"Synchronizer Token Pattern" and §"Transmitting CSRF Tokens" | Direct quotation |
| 49 | Per-request tokens are stronger in principle but cause Back-button false positives; over-rotation is *"a misconception that assumes it adds substantial security while actually harming the user experience"* | ✅ | OWASP cheat sheet, §"Synchronizer Token Pattern" | Direct quotation. §5.3 |
| 50 | The **naive double-submit is bypassable** by cookie injection — vulnerable sibling subdomain, DNS takeover, or plaintext-HTTP injection on a non-`__Host-` cookie; OWASP marks it **DISCOURAGED** and "for reference only" | ✅ | OWASP cheat sheet, §"Naive Double-Submit Cookie Pattern (DISCOURAGED)" warning | **Direct quotation of the warning.** §5.4 |
| 51 | Django's own documentation: subdomains can set cookies for the whole domain and *"subdomains will be able to circumvent the CSRF protection"* | ✅ | Django `ref/csrf`, §"Limitations" | **First-party confirmation of the injection exposure.** Independent of OWASP |
| 52 | The signed double-submit must be **bound to the session**; signing without binding provides *"minimal protection"* | ✅ | OWASP cheat sheet, §"Signed Double-Submit Cookie (RECOMMENDED)" | Direct quotation |
| 53 | HMAC comparison should be **constant-time** to prevent timing attacks | ✅ | OWASP cheat sheet, §"Pseudo-Code For Implementing HMAC CSRF Tokens" | Direct quotation |
| 54 | Django's synchronizer: secret in a cookie, masked per response against BREACH, **secret changes each time a user logs in**, comparison on the secret only | ✅ | Django `ref/csrf`, §"How it works", items 1–3 | **Framework-documentation class, quoted.** §5.2 |
| 55 | Django verifies `Origin` where provided (against the host and `CSRF_TRUSTED_ORIGINS`) and performs **strict referer checking** for HTTPS requests where `Origin` is absent | ✅ | Django `ref/csrf`, §"How it works", items 4–5 | Direct quotation |
| 56 | Spring Security protects unsafe methods **by default**; default repository is `HttpSessionCsrfTokenRepository`; Spring Security 6 defers token loading and adds per-request randomness for BREACH | ✅ | Spring Security reference, §"Cross Site Request Forgery (CSRF)" | Framework-documentation class |
| 57 | ASP.NET Core: a bearer token in local storage *"provides protection against CSRF attacks"*; CSRF *"is a concern when the token is stored in a cookie"*; `Sec-Fetch-Site` and `Origin` are forbidden request headers the page cannot forge | ✅ | Microsoft Learn, ASP.NET Core anti-request-forgery (page last updated 14 Sept 2026) | Framework-documentation class, quoted. §6.1 |
| 58 | The `Origin` and `Referer` headers are on the browser's **forbidden header** list, so only the browser can set them | ✅ | OWASP cheat sheet, §"Using Standard Headers to Verify Origin" | Specification-adjacent class |
| 59 | Recommended Origin-check strength: compare against the **entire origin**, so `example.org.attacker.com` does not pass for `example.org` | ✅ | OWASP cheat sheet, same section | Direct quotation. §3.4 |
| 60 | If neither `Origin` nor `Referer` is present, OWASP *"recommend[s] blocking"* | ✅ | OWASP cheat sheet, same section | Direct quotation |
| 61 | Accepting a **`null`** origin is exploitable, and OWASP notes the practice while acknowledging the risk | ✅ | OWASP cheat sheet, same section (*"Please note that, attackers can exploit this"*) | This guide's position (reject `null` for state-changing endpoints) is stricter than OWASP's accommodation, and §3.4 says so |
| 62 | **CORS is a read restriction, not a send restriction** | ✅ | MDN CORS guide (*"permit loading resources"*; server *"must opt-in using `Access-Control-Allow-Origin` to share the response with the script"*); OWASP cheat sheet | **The §7.3 conflation, resolved from two primary docs** |
| 63 | Cookies are not sent on cross-origin requests by default; credentials require `Access-Control-Allow-Credentials`; `Allow-Origin: *` is rejected by the browser when credentials are allowed | ✅ | OWASP cheat sheet, §"Custom Headers and CORS" | Direct quotation. §6.2 |
| 64 | A CORS regex allowing all subdomains is dangerous given subdomain takeover | ✅ | OWASP cheat sheet, same section | Direct quotation |
| 65 | **Clickjacking requires framing**; defences are CSP `frame-ancestors` (with `X-Frame-Options` for older browsers), `SameSite` on auth cookies, and frame-busting code | ✅ | OWASP Clickjacking page | Direct quotation. §7.4 |
| 66 | HTTPS *"by itself does nothing to defend against CSRF"* but is a prerequisite for the defences | ✅ | OWASP cheat sheet, §"Prevention measures that do NOT work"; ASP.NET Core docs (*"Using HTTPS doesn't prevent a CSRF attack"*) | Source-confirmed twice |
| 67 | *"Only accepting POST requests"* is **not** a defence | ✅ | OWASP `www-community/attacks/csrf` | Direct quotation. §2.5 |
| 68 | *"Multi-Step transactions are not an adequate prevention of CSRF"* | ✅ | OWASP `www-community/attacks/csrf` | Direct quotation. §2.7 |
| 69 | A secret cookie is not a defence — *"all cookies, even the secret ones, will be submitted with every request"* | ✅ | OWASP `www-community/attacks/csrf` | Direct quotation |
| 70 | **Login CSRF** is real and is a *data-exfiltration* route (the attacker logs the victim into the attacker's account) | ✅ | OWASP `www-community/attacks/csrf`; Django `ref/csrf` (login CSRF *"is also covered"*) | Source-confirmed twice |
| 71 | **Client-side CSRF** bypasses token and SameSite defences; mitigations are input isolation, validation, and predefined request data | ✅ | OWASP cheat sheet, §"Dealing with Client-Side CSRF Attacks"; Khodayari & Pellegrino, USENIX Security 2021 (cited by OWASP) | Academic + vendor-confirmed. §2.6 |
| 72 | `HttpOnly` does **not** stop the cookie being sent on scripted requests | ✅ | MDN `Set-Cookie` (*"will still be sent with JavaScript-initiated requests"*) | Direct quotation. Corrects misstatement #2 in §7.5 |
| 73 | **OWASP Top 10 removed CSRF in 2017** because *"many frameworks include CSRF defenses, it was found in only 5% of applications"*; it was **A8 in 2013** | ✅ | OWASP Top Ten 2017 Release Notes | **Version-and-date class.** §8.5 |
| 74 | The academic foundation: Barth, Jackson & Mitchell, *Robust Defenses for Cross-Site Request Forgery*, ACM CCS '08, pp. 75–88, October 2008 (DOI 10.1145/1455770.1455782) | ✅ | Cited as reference **[CSRF]** in RFC 6265bis's bibliography | Citation verified from the specification's own reference list |
| 75 | The double-submit method is attributed to **Felten and Zeller** | ✅ (as attributed) | CWE-352, Potential Mitigations (*"as described by Felten and Zeller"*) | Attributed; the primary Felten–Zeller artefact was not read |
| 76 | The **uTorrent 2008 `GET`-based CSRF** exploit *"used on a mass scale to download malware"* | ✅ (as attributed) | OWASP `www-community/attacks/csrf`, linking a January 2008 tech-news write-up | **The only historical incident cited in this guide.** Named source and date; a news source rather than an advisory — labelled as such in §8.6 |
| 77 | The Samy worm used `XMLHttpRequest` to obtain the CSRF token and forge requests — an XSS/CSRF composition | ✅ (as attributed) | OWASP `www-community/attacks/csrf` | Attributed to OWASP; the worm's own primary record was not read |
| 78 | CSRF was named by **Peter Watkins, Bugtraq, June 2001** | 🚩 | Repeated in the literature; a survey treatment states it directly and cites the archive | **Flagged, not asserted.** §8.1, §8.6 |
| 79 | Modern bearer-token APIs are **structurally immune to classical CSRF** (the credential is not ambient) | ✅ | Microsoft Learn (ASP.NET Core anti-request-forgery) | Framework-documentation class. §6.1 — with the explicit counterweight in §6.5 that this is a credential-model change, not a defence |

**No CVE identifier is cited anywhere in this guide.** That is deliberate: no CVE was verified against a primary advisory this pass, and inventing or half-remembering one would corrupt the record this section exists to protect. Where the guide needed an incident example, it used the single OWASP-attributed, dated one (row 76) and labelled its source quality.

---

## 14. What Could Not Be Verified

Each item below is stated as a limitation of *this pass*, not as a claim about the world. Where a search returned nothing, that is recorded as a tool limitation and never as evidence of absence.

1. **Whether Chrome still applies the Lax+POST two-minute carve-out.** ⚠ Chromium's published intent was removal "some time after" Chrome 80 stable ✅, and the tracking page's last entry on the point is dated 18 March 2021. No primary source read this pass confirms removal or continued application in current Chrome builds. **The specification still defines the mode** (RFC 6265bis §5.6.7.2, rev. 22, December 2025) ✅. Consequence: §4.3 treats it as possibly present and recommends explicit labelling regardless.
2. **The exact FIRST Safari/WebKit version to default to `Lax`.** ⚠ WebKit's adoption is established (bugs 278353, 279153 ✅) and dated to the iOS 18 / Safari 18 window by the bug's own framing (reported 4 September 2024) ✅, but the guide does not assert a precise first version, because no WebKit release note naming the version was read.
3. **Whether iOS 18.0.1's restoration of "SameSite=None cookies by default" affects server-set cookies, or only client-set ones.** ⚠ Comment 17 of WebKit bug 279153 states the restoration; the reproduction and the fix are about cookies set by the *client application*, and the guide declines to generalise it to server-set cookies.
4. **Firefox releases between 96 and 110.** ⚠ Firefox 96 (11 January 2022) is documented as enforcing Lax-by-default ✅ and the pref was disabled on the 111 branch ✅, but the state of the releases *between* those points is not established by the sources read. §4.6 recommends the conservative reading (do not assume Firefox withholds the cookie). Aggregate support data lists Firefox as "disabled by default" across its tracked ranges ✅, consistent with the WONTFIX outcome.
5. **A primary-artefact confirmation of the June 2001 Bugtraq attribution.** 🚩 The identifier and date are consistent across secondary sources; the mailing-list post itself was not retrieved.
6. **Independent measurement of the Fetch Metadata coverage figure.** ⚠ The "over 98% global coverage" figure is OWASP's, citing caniuse; this guide reports it as attributed and does not re-measure it.
7. **Whether the `SameSite` draft has since published as an RFC.** ⚠ The revision read (22, December 2025) remained an Internet-Draft with an expiry of 4 June 2026 ✅; no later status change was verified.
8. **`web_search` returned an empty result set for one Safari-targeted query** during this pass (a search for a Safari/iOS 18 `SameSite` change announcement returned no results), and two `caniuse` raw-data files requested by direct URL returned HTTP 404. Both are recorded as **tool limitations, not evidence of absence**; the Safari position in §4.6 rests on the WebKit bug records, which were retrieved directly.
9. **No incident, no institution, and no CVE.** The guide asserts no named institution's CSRF incident and cites no CVE number, for the reasons in §8.6 and §13. This is a deliberate scope limitation of the research, not a finding that such incidents do not exist.

---

## 15. The Glossary and Cross-References

### 15.1 Glossary

| Term | Definition | Where it is load-bearing |
|---|---|---|
| **Ambient authority** | Authority attached by the platform based on *who the caller is*, without the caller choosing to present it. A session cookie is the canonical web case | §1.1, §1.4, §6.1 — the root cause |
| **Bearer token** | A credential the client must attach explicitly (typically an `Authorization` header); not ambient, therefore outside the classical CSRF model | §6.1, §6.5 |
| **BREACH** | A compression side-channel attack on HTTP responses, relevant because a static CSRF token could be extracted from a compressed page — hence per-response masking | §5.2 (Django's mask), §5.5 (Spring's per-request randomness) |
| **Client-side CSRF** | An attacker manipulates the application's *own* JavaScript into issuing a state-changing request by controlling an input (URL fragment, `window.name`, `postMessage`). Bypasses token and `SameSite` defences | §2.6, §5.7 |
| **Confused deputy** | CWE-441. A component that wields its authority on behalf of a principal without verifying that the principal wanted the action. CSRF's formal mechanism | §1.3 |
| **Credential-attachment rule** | The browser's rule for when cookies are attached to a request: from the destination and the cookie's attributes, never from who initiated it | §2.3 — the single fact that makes CSRF possible |
| **Cross-site / same-site** | A relationship between an initiating context and a target, computed on **registrable domain plus scheme** — not on origin, and not on port. `http://app.example` and `https://app.example` are cross-site | §4.5, §9.4 |
| **CSRF token** | A per-session (or per-request) server-generated secret required back on state-changing requests; must be unique per session, secret and unpredictable | §5.1–§5.3 |
| **Double-submit cookie** | A secret in a cookie mirrored in a request parameter/header, accepted if the two match. The **naive** form is bypassable by cookie injection on the target domain | §5.4 |
| **Fetch Metadata headers** | `Sec-Fetch-Site`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest`, `Sec-Fetch-User` — browser-set request metadata describing *how* the request was made | §3.2, §8.4 |
| **`frame-ancestors`** | The CSP directive restricting which origins may embed the page in a frame; the primary clickjacking defence, with `X-Frame-Options` for legacy clients | §7.4 |
| **Lax-allowing-unsafe** | A specification-defined transitional enforcement mode that waives the "safe method" requirement for recently created cookies that did not explicitly specify `SameSite`; the spec's codification of Chrome's 2-minute carve-out | §4.3 |
| **Login CSRF** | Forcing a victim to log in *as the attacker*; a data-exfiltration route rather than a state-change route. The pre-authentication surface | §2.6, §10.2 |
| **Logout CSRF** | Forcing a logout; low impact alone, useful as a probe and as a step in a fixation or phishing chain | §2.6 |
| **`Origin`** | A browser-set request header giving the initiating origin; on the forbidden-header list, so a page cannot forge it. Absent in some legitimate cases and `null` in others | §3.4 |
| **Preflight** | The automatic `OPTIONS` request the browser sends before a non-simple cross-origin request, giving the server a veto before the real request is sent | §2.2, §3.6, §6.2 |
| **`Referer`** | A browser-set header giving the initiating URL (often reduced to the origin under modern referrer policy); suppressible, therefore a weaker provenance signal than `Origin` | §3.4 |
| **Registrable domain / site** | The public-suffix-determined domain used for the same-site comparison. Cookies are scoped to *site*, not *origin* — the root of the sibling-subdomain trust problem | §3.3, §4.6, §10.6 |
| **Safe / unsafe method** | Safe: `GET`, `HEAD`, `OPTIONS`, `TRACE` — retrieval-only by definition. Unsafe: `POST`, `PUT`, `PATCH`, `DELETE`. `SameSite=Lax`'s exemption is keyed to method safety | §2.4, §4.4 |
| **`SameSite`** | The cookie attribute limiting cross-site attachment; values `Strict`, `Lax`, `None`, plus a spec-defined `Default` state subject to a default enforcement mode | §4 |
| **Schemeful same-site** | The rule that scheme is part of the site comparison, so an HTTP↔HTTPS transition is cross-site | §4.5 |
| **Synchronizer token** | The classic token pattern: server-side reference value compared against a value returned by the client; must not travel as a cookie | §5.1–§5.3 |
| **Step-up authentication** | Requiring the human to prove presence again at the moment of a high-value action. A human-presence control, complementary to provenance controls rather than part of them | §3.7, §10.2 |
| **Top-level navigation** | A request that changes the address bar (link click, form submission, `window.location`), as distinct from a subresource fetch. The scope of `Lax`'s cross-site allowance | §4.4, §2.2 |
| **`__Host-` / `__Secure-`** | Cookie name prefixes enforcing attribute requirements: `__Host-` needs `Secure`, `Path=/` and no `Domain`, making the cookie host-only and unsettable from a subdomain | §4.5, §5.4 |

### 15.2 Where Each Sibling Guide Takes Over

| Topic | Guide |
|---|---|
| Session establishment, cookie mechanics, fixation, hijacking, rotation, expiry (including CWE-613's role) | [distributed_auth_guide.md](distributed_auth_guide.md) |
| The design-time discipline, the weakness rankings, the secure-coding standards, where DAST sits in the tooling pyramid | [security_by_design_guide.md](security_by_design_guide.md) |
| The threat-modelling method (STRIDE's *Spoofing*/*Repudiation* framing of this threat, attack trees, the threat register) | [threat_modeling_guide.md](threat_modeling_guide.md) |
| OAuth/OIDC flow protections, where the `state` parameter is the CSRF defence for the redirect leg, and why a browser client should not keep tokens in script-reachable storage | [../banking/fapi_financial_grade_api_guide.md](../banking/fapi_financial_grade_api_guide.md) |
| The general security discipline, posture and frameworks | [cybersecurity_guide.md](cybersecurity_guide.md), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md), [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) |
| API security, gateways, schema governance, API-surface inventory | [api_governance_guide.md](api_governance_guide.md) |
| Test execution standard and reporting format; pipeline orchestration; failure injection | [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md), [test_orchestration_guide.md](test_orchestration_guide.md), [chaos_engineering_guide.md](chaos_engineering_guide.md) |
| Regulatory reporting and compliance machinery behind §10.5's evidence discussion; operational resilience taxonomy | [../banking/regtech_guide.md](../banking/regtech_guide.md), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) |

### 15.3 Primary Sources Used

- **Fetch Metadata Request Headers**, W3C Web Application Security Working Group, Working Draft **1 April 2025** — https://w3c.github.io/webappsec-fetch-metadata/
- **Cookies: HTTP State Management Mechanism**, `draft-ietf-httpbis-rfc6265bis-22`, **December 2025** — https://www.ietf.org/archive/id/draft-ietf-httpbis-rfc6265bis-22.txt
- **RFC 6265** (HTTP State Management Mechanism, April 2011) — the base specification, for the absence of `SameSite`
- **MDN**: the `Set-Cookie` reference; the CORS guide; the CSRF glossary entry — https://developer.mozilla.org/
- **The Chromium Projects**, "SameSite Updates" (page last updated 18 March 2021) — https://www.chromium.org/updates/same-site/ ; **Chromium Blog** SameSite announcements (February and April 2020)
- **Mozilla Bugzilla** 1617609 (*[meta] Enable sameSite=lax by default*, RESOLVED WONTFIX) and 1812297 (*Disable sameSite=lax by default on Nightly*, RESOLVED FIXED, "111 Branch"); **Firefox 96.0 release notes** (11 January 2022) and **Firefox for Enterprise 96 release notes**
- **WebKit Bugzilla** 278353 (*Update remaining tests to conform to new SameSite=Lax by default cookie behavior*) and 279153 (*iOS 18 does not allow a client application to specify SameSite=None*, reported 4 September 2024)
- **OWASP**: the CSRF Prevention Cheat Sheet; the `www-community` CSRF attack page; the Clickjacking page
- **MITRE CWE-352** (*Cross-Site Request Forgery (CSRF)*) — page last updated 30 April 2026
- **Framework documentation**: Django `ref/csrf` (6.1); Spring Security, "Cross Site Request Forgery (CSRF)"; Microsoft Learn, "Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks in ASP.NET Core" (page last updated 14 September 2026)
- **Academic**: Barth, Jackson & Mitchell, *Robust Defenses for Cross-Site Request Forgery*, ACM CCS '08, pp. 75–88, October 2008; Khodayari & Pellegrino, USENIX Security 2021 (client-side CSRF, cited via OWASP)

---

## 16. The Closing Summary

Six ideas carry the whole topic. If a reader takes nothing else from this guide, these are the ones to keep:

1. **CSRF is a missing-intent bug, not an injection bug.** The request is well-formed, authenticated and authorised. What is absent is the user's intent, and it lives outside the request body — in the browser's decision about whether to attach a credential. Every useful defence is therefore a provenance check, and none of them inspect the payload (§1.2).
2. **The same-origin policy does not stop the request; it restricts the reading of the response.** This is the crux. An attacker who never sees a byte of the response can still move the money (§2.2).
3. **The defence stack has two halves with disjoint failure modes, and you need both.** Browser-enforced controls — Fetch Metadata, explicit `SameSite`, the preflight consequence of a custom header — hold without your code being correct, and they fail on browsers, redirect chains and stripped metadata. Application-enforced controls — tokens, `Origin` checks, step-up — work on every client, and they fail when your code is wrong. Defence in depth here means one of each, not three of the same kind (§3.8).
4. **Do not present CSRF as solved.** The browser default is real but non-uniform — Chrome shipped it first in Chrome 80 (2020) after a rollout that was announced, started, *rolled back in April 2020 and resumed in July*, and **Firefox does not currently implement it at all**, by Mozilla's own record (§4.1, §4.6). `SameSite=Lax` still permits the state-changing `GET`. Tokens are defeated by XSS. And wherever the cookie model remains, the vulnerability remains (§6.5).
5. **Fix the state-changing `GET`, label every cookie explicitly, and keep an owned inventory of exemptions.** These three mechanical habits prevent the majority of the failure modes in §12 — and eleven of those twelve anti-patterns are not missing controls but *degraded* ones, which is why the regression assertions in §9.5 matter more than any single control choice.
6. **Separate the adjacent bugs exactly.** XSS defeats every CSRF defence and no CSRF defence touches XSS; CORS governs reads and never governed writes; clickjacking needs a frame and a real click, so a token is no defence against it. Getting these distinctions right is most of what "understanding CSRF" actually means (§7).

For a regulated institution, the conclusion is narrower and more practical than the general one: the cookie-authenticated browser session is a deliberate architectural choice, it will be retained, and therefore this class stays live in retail banking regardless of its portfolio frequency elsewhere. The work is not to hope the browser handles it. The work is to configure the attributes explicitly, enforce a default-deny provenance policy with a tested absent-header branch, bind a token to the session, require human presence for the handful of actions that enable everything else, and be able to demonstrate — with a date, a browser version and a recorded result — that a request from another site, carrying a live session, changed nothing.

That is the difference this guide has been circling from the first section: a defence that establishes intent, and a server that can tell a real click from the request the user never made.

