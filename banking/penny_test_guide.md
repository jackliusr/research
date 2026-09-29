# The Penny Test in Banking — The Cheapest Test Is Also the Cheapest Attack

**Three Practices Under One Name — Micro-Deposit Account Verification (Sense 1), Instrument Validation by Tiny Authorisation and Its Fraud Typology Card Testing (Sense 2), and the Live Nominal-Value Go-Live Test Payment (Sense 3) — the Decoder (Micro-Deposit, Name-Match, Prenote and the Zero-Value File, Nominal Value, Pre-Authorisation, Card-on-File Validation, Smoke Test and Certification Test, Suspense Entry, Cut-Off), the Unifying Mechanism (the Information a Payment Yields Does Not Scale with Its Value, While the Cost of the Test Does), the Dual-Use Problem in a Bank's Own Monitoring, the Test Protocol by Rail Class, the Certification-Versus-Production Question, the Accounting, Reconciliation and Audit Angle, the Customer-Consent and Data-Protection Angle for Sense 1, the Alternatives and Their Displacement, the Anti-Patterns, a Cymbal Bank Test-Protocol Worked Example, the Claims Audit, the Glossary, What Could Not Be Verified and the Closing Summary — from the Two-Small-Amounts ACH Micro-Deposit to the Real-Time Rail Test Payment**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Payments Infrastructure and Financial Crime — the *penny test*, the three distinct banking practices that share the name and the mechanism: **Sense 1**, account and beneficiary verification by micro-deposit (the two small random amounts, the confirmation that proves control, the onboarding and payee-setup use case); **Sense 2**, validation of a payment instrument by a tiny authorisation (card-on-file validation and the counter-flow, the fraud typology most commonly called card testing, carding, account testing, enumeration or card checking); **Sense 3**, the live nominal-value payment run that proves a payment channel or rail end-to-end before or after go-live (the smoke test, the go-live test payment, the certification-versus-production question). The decoder (micro-deposit, name-match, prenote and zero-value file, nominal value, pre-authorisation, card-on-file validation, smoke test, certification test, suspense entry, cut-off), the unifying mechanism (a tiny value is the right probe because the information a payment yields does not scale with its value while the cost of the test does), the dual-use problem (the same mechanism is the cheapest way for a customer to prove they control an account and the cheapest way for a fraudster to prove a stolen instrument is live), the test protocol by rail class, the accounting, reconciliation and audit discipline, the consent and data-protection angle, the alternatives and their displacement, the anti-patterns, a worked test-protocol design, the claims audit and the glossary.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the vendors that publicly document micro-deposit verification (**Dwolla** developer documentation — two deposits of less than $0.10, random amounts, verification attempt limits, sandbox behaviour; **Plaid** Auth documentation — the Auth method comparison and coverage, automated / same-day / instant micro-deposits over RTP or FedNow, database verification); the processors that publicly document card testing (**Stripe** documentation — "card testing", its synonyms and its consequences); the ACH operator (**Nacha** — the public ACH Network description, Same Day ACH processing and per-payment limits); the real-time rail operators (**The Clearing House** — the RTP network public material and the RTP Document Library, including the **RTP Network Compliance Bulletin 1-2024 on the "No Searching for Accounts" rule, dated 5 September 2024**, which is the single most directly relevant public scheme document for Sense 3; **Federal Reserve Financial Services** — FedNow Service public resources and Operating Circular 8); the payer-name-checking schemes (**Pay.UK** — Confirmation of Payee public material, launched 2020; the **European Payments Council** — the Verification of Payee (VOP) scheme public material and its published evolution calendar). Scheme rulebooks are version-specific and much of their substance sits behind membership — see §15 and §16 for what is verified, what is flagged and what could not be established at source. The repo's sibling guides are cross-referenced by name rather than re-derived. NOTE: in this pass **`web_search` returned empty result sets for every query** (a tool limitation, recorded in §16), so every source below was retrieved by direct extraction from primary URLs; fetch date **2026-09-29** unless otherwise stated.
> **Last Updated:** September 2026
> **Companion guides (sibling, same folder — the payments cluster):** [Payment Rails](payment_rails_guide.md) (the rails taxonomy, the real-time map, the message standards, the clearing and settlement, the rail-selection context — this guide owns the *testing practice* that runs on those rails, not the rails themselves), [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) (the fraud-detection discipline — this guide owns the card-testing *typology* and cross-references the detection machinery there, including its card-testing pattern at line 476), [Micropayment Options Research](micropayment_options_research.md) (micropayments as a *product* and an economics problem — the distinction is stated in §1.5), [IBPS Payment Connect](ibps_payment_connect_guide.md) (the Confirmation-of-Payee and name-record angle, and the rule-engine test matrix in its §8 at line 471), [Financial Infrastructure](financial_infrastructure_guide.md) (the repo's other Confirmation-of-Payee mention, line 413), [Supply Chain Finance](supply_chain_finance_guide.md) (the repo's name-match beneficiary-verification material), [SWIFT Alliance Access](swift_alliance_access_guide.md) and [SWIFTNet FileAct](swiftnet_fileact_guide.md) (the correspondent messaging and file-transfer path), [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) (the message definitions), [Singapore Fintech and Payments](singapore_fintech_payments_guide.md), [CNAPS](cnaps_guide.md), [NUCC NetsUnion](nucc_netsunion_guide.md), [Payments Hub](payments_hub_guide.md) (the rail adapters and the hub architecture), [Airwallex](airwallex_guide.md) and the platform guides (the sandbox and simulated-rails offer — the *simulation alternative* to a live test), [Mojaloop](mojaloop_guide.md) (the sandbox-demo angle), [FAPI Financial-Grade API](fapi_financial_grade_api_guide.md) (the open-banking API angle behind instant verification), [Posting Engine / Core Banking](posting_engine_core_banking_guide.md) (the entry, the suspense account and the reversal mechanics), [Operational Resilience Framework](operational_resilience_framework_guide.md) (the change-and-test governance around go-live)
> **Companion guides (technology/, prefix `../technology/`):** [Event Stream Processing](../technology/event_stream_processing_guide.md) (the card-testing CEP pattern at line 1047 — cross-referenced, not re-derived), [Distributed Rate Limiter](../technology/distributed_rate_limiter_guide.md) (rate limiting as a card-testing mitigation), [Cybersecurity](../technology/cybersecurity_guide.md) (the carding-rings actor category and the stolen-credential market), [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (the 24/7 rails and the cut-off calendar — why a live test on a 24/7 rail has no maintenance window to hide in)

---

**How to use this guide:** Read §1 first even if you only want one of the three senses — the three senses are different practices that share a name, and most of the confusion in this area comes from people using one sense while listening in another. Section 2 is the unifying mechanism, the guide's actual thesis. Sections 3, 4 and 6–7 develop the three senses. Section 5 is the dual-use problem, which is the operational reason a bank cannot treat Sense 1 and Sense 2 as separate subjects. Section 8 applies the practice rail class by class. Section 9 separates certification evidence from production readiness. Sections 10 and 11 are the accounting/audit and consent/data-protection disciplines. Section 12 is the alternatives, §13 the anti-patterns, §14 the worked example. Sections 15 and 16 close with the claims audit, the absences, the glossary and the cross-references. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; `technology/` guides are prefixed `../technology/`. **Integrity convention:** ✅ = verified in this pass against a primary source retrieved on 2026-09-29, or verified in a cross-referenced guide's own ledger; ⚠ = flagged, uncertain, version-specific, member-restricted or otherwise not established at source. Scheme rulebook substance that is not publicly available is flagged, never reconstructed from memory. **Every amount, limit, timing and message type in this guide is either sourced below or flagged. Nothing in this guide is a scheme rule reconstructed from memory.**

---

## Table of Contents

1. [Overview, the Decoder and the Boundary](#1-overview-the-decoder-and-the-boundary)
   - 1.1 [The Three Senses in One Paragraph](#11-the-three-senses-in-one-paragraph)
   - 1.2 [The Definitions — Senses 1, 2 and 3, and What a Practitioner Calls Them](#12-the-definitions--senses-1-2-and-3-and-what-a-practitioner-calls-them)
   - 1.3 [Why the Name Collides](#13-why-the-name-collides)
   - 1.4 [The Decoder — the Terms This Guide Uses](#14-the-decoder--the-terms-this-guide-uses)
   - 1.5 [The Thesis and the Boundary](#15-the-thesis-and-the-boundary)
   - 1.6 [The Scope Statement — All Three Senses Are in Scope](#16-the-scope-statement--all-three-senses-are-in-scope)
2. [The Unifying Mechanism — Why a Tiny Value Is the Right Probe](#2-the-unifying-mechanism--why-a-tiny-value-is-the-right-probe)
   - 2.1 [The Analysis (Labelled)](#21-the-analysis-labelled)
   - 2.2 [The Worked Side-by-Side — the Small Probe Versus the Large Payment](#22-the-worked-side-by-side--the-small-probe-versus-the-large-payment)
   - 2.3 [Where the Analysis Breaks Down](#23-where-the-analysis-breaks-down)
3. [Sense 1 — Micro-Deposit Verification](#3-sense-1--micro-deposit-verification)
   - 3.1 [The Mechanism and the Flow](#31-the-mechanism-and-the-flow)
   - 3.2 [Where It Is Used](#32-where-it-is-used)
   - 3.3 [The Security Property — the Amount Is the Secret](#33-the-security-property--the-amount-is-the-secret)
   - 3.4 [The Weaknesses, Developed Properly](#34-the-weaknesses-developed-properly)
   - 3.5 [The Failure Cases](#35-the-failure-cases)
   - 3.6 [Consent and Data Exposure — the Amount Goes on Someone's Statement](#36-consent-and-data-exposure--the-amount-goes-on-someones-statement)
   - 3.7 [The Displacement — Name-Match, Account Checking and Instant Confirmation](#37-the-displacement--name-match-account-checking-and-instant-confirmation)
4. [Sense 2 — Card Testing as a Fraud Typology](#4-sense-2--card-testing-as-a-fraud-typology)
   - 4.1 [The Mechanism](#41-the-mechanism)
   - 4.2 [Why the Attacker's Test Is Cheap and the Merchant's Cost Is Not](#42-why-the-attackers-test-is-cheap-and-the-merchants-cost-is-not)
   - 4.3 [The Observable Pattern at the Level Sources Support](#43-the-observable-pattern-at-the-level-sources-support)
   - 4.4 [What the Merchant and Acquirer Actually Suffer](#44-what-the-merchant-and-acquirer-actually-suffer)
   - 4.5 [The Cardholder-Facing Version](#45-the-cardholder-facing-version)
   - 4.6 [The Typology by Mechanism, and Where the Detection Machinery Lives](#46-the-typology-by-mechanism-and-where-the-detection-machinery-lives)
5. [The Dual-Use Problem in the Bank's Own Monitoring](#5-the-dual-use-problem-in-the-banks-own-monitoring)
   - 5.1 [The Same Mechanism, Opposite Parties](#51-the-same-mechanism-opposite-parties)
   - 5.2 [What Actually Distinguishes the Two — and What Does Not](#52-what-actually-distinguishes-the-two--and-what-does-not)
   - 5.3 [An Unresolved Ambiguity, Stated Plainly](#53-an-unresolved-ambiguity-stated-plainly)
   - 5.4 [The Collision with the Bank's Own Monitoring](#54-the-collision-with-the-banks-own-monitoring)
   - 5.5 [The Whitelist That Becomes a Hole — the Guardrails](#55-the-whitelist-that-becomes-a-hole--the-guardrails)
6. [Sense 3 — The Live Go-Live Test Payment](#6-sense-3--the-live-go-live-test-payment)
   - 6.1 [The Sandbox-Versus-Network Argument](#61-the-sandbox-versus-network-argument)
   - 6.2 [What Each Hop of the Chain Establishes](#62-what-each-hop-of-the-chain-establishes)
   - 6.3 [What a Nominal but Real Value Buys That a Zero-Value File Does Not](#63-what-a-nominal-but-real-value-buys-that-a-zero-value-file-does-not)
   - 6.4 [The Pre-Conditions for a Safe Live Test](#64-the-pre-conditions-for-a-safe-live-test)
7. [Sense 3 Continued — the Discipline a Live Test Demands](#7-sense-3-continued--the-discipline-a-live-test-demands)
   - 7.1 [The Accounting Consequence — a Real Entry in Two Institutions' Books](#71-the-accounting-consequence--a-real-entry-in-two-institutions-books)
   - 7.2 [The Reconciliation Visibility Rule](#72-the-reconciliation-visibility-rule)
   - 7.3 [The Irreversibility Problem on Instant Rails](#73-the-irreversibility-problem-on-instant-rails)
   - 7.4 [The Pre-Documented Expectation](#74-the-pre-documented-expectation)
   - 7.5 [Containment, Reversal and the Evidence Retained for Audit](#75-containment-reversal-and-the-evidence-retained-for-audit)
8. [The Test Protocol by Rail Class](#8-the-test-protocol-by-rail-class)
   - 8.1 [Batch and File-Based Credit Transfer](#81-batch-and-file-based-credit-transfer)
   - 8.2 [Instant Account-to-Account and Real-Time Schemes](#82-instant-account-to-account-and-real-time-schemes)
   - 8.3 [The Correspondent and Cross-Border Messaging Path](#83-the-correspondent-and-cross-border-messaging-path)
   - 8.4 [Domestic Low-Value and High-Value Clearing Systems](#84-domestic-low-value-and-high-value-clearing-systems)
   - 8.5 [The Cross-Rail Principle](#85-the-cross-rail-principle)
9. [The Certification-Versus-Production Question](#9-the-certification-versus-production-question)
   - 9.1 [Where a Scheme Requires Certification or a Sandbox Instead of a Live Probe](#91-where-a-scheme-requires-certification-or-a-sandbox-instead-of-a-live-probe)
   - 9.2 [What a Certification Test Actually Establishes](#92-what-a-certification-test-actually-establishes)
   - 9.3 [The Honest Distinction — Scheme Evidence Versus Production Readiness](#93-the-honest-distinction--scheme-evidence-versus-production-readiness)
10. [The Accounting, Reconciliation and Audit Angle](#10-the-accounting-reconciliation-and-audit-angle)
    - 10.1 [The Entry, the Suspense Account and the Reversal](#101-the-entry-the-suspense-account-and-the-reversal)
    - 10.2 [How the Test Appears to an Auditor](#102-how-the-test-appears-to-an-auditor)
    - 10.3 [An Unrecorded Test Payment Is a Control Failure](#103-an-unrecorded-test-payment-is-a-control-failure)
11. [Customer Consent and Data Protection for Sense 1](#11-customer-consent-and-data-protection-for-sense-1)
    - 11.1 [The Consent the Account Holder Gives](#111-the-consent-the-account-holder-gives)
    - 11.2 [What the Act of Verification Discloses](#112-what-the-act-of-verification-discloses)
    - 11.3 [The Retention Question](#113-the-retention-question)
    - 11.4 [Verifying an Account the Customer Owns Versus Profiling One They Do Not](#114-verifying-an-account-the-customer-owns-versus-profiling-one-they-do-not)
12. [The Alternatives and Their Displacement](#12-the-alternatives-and-their-displacement)
    - 12.1 [Name-Match and Account-Checking Services](#121-name-match-and-account-checking-services)
    - 12.2 [Open-Banking-Style Instant Confirmation](#122-open-banking-style-instant-confirmation)
    - 12.3 [The Scheme's Own Validating Mechanisms](#123-the-schemes-own-validating-mechanisms)
    - 12.4 [The Sandbox and the Simulated Rail](#124-the-sandbox-and-the-simulated-rail)
    - 12.5 [The Comparison Table](#125-the-comparison-table)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Cymbal Bank Worked Example — Designing a Test Protocol](#14-the-cymbal-bank-worked-example--designing-a-test-protocol)
    - 14.1 [The Situation](#141-the-situation)
    - 14.2 [Certification Versus Live — the Split](#142-certification-versus-live--the-split)
    - 14.3 [The Nominal-Value Probe and Where It Lands in the Books](#143-the-nominal-value-probe-and-where-it-lands-in-the-books)
    - 14.4 [Reconciliation and Suspense Treatment](#144-reconciliation-and-suspense-treatment)
    - 14.5 [The Test Cymbal Refuses to Run in Production](#145-the-test-cymbal-refuses-to-run-in-production)
    - 14.6 [The Monitoring Annotation, with Its Expiry](#146-the-monitoring-annotation-with-its-expiry)
    - 14.7 [The Evidence Pack, and the Thesis](#147-the-evidence-pack-and-the-thesis)
15. [The Claims Audit — Verified, Flagged, Rejected](#15-the-claims-audit--verified-flagged-rejected)
    - 15.1 [The Verified Claims (✅)](#151-the-verified-claims-)
    - 15.2 [The Flagged Claims (⚠)](#152-the-flagged-claims-)
    - 15.3 [The Rejected or Not-Found Claims and the Absences](#153-the-rejected-or-not-found-claims-and-the-absences)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified](#161-what-could-not-be-verified)
    - 16.2 [The Glossary](#162-the-glossary)
    - 16.3 [The Cross-References](#163-the-cross-references)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. Overview, the Decoder and the Boundary

### 1.1 The Three Senses in One Paragraph

In banking practice, **"penny test"** — and its cousins *penny drop*, *micro-deposit*, *carding* and *smoke test* — covers **three different practices that happen to share a name and a mechanism**. They are performed by different people, in different departments, for different reasons, against different counterparties, and each demands its own discipline. They are:

1. **Sense 1 — micro-deposit verification.** Sending one or more small, randomly determined amounts *into* an account the customer claims to own, and requiring the customer to report those amounts back, so that the reporting proves control of the account.
2. **Sense 2 — instrument validation by tiny authorisation.** Putting a very small charge *against* a payment instrument to establish whether the instrument is live; performed legitimately as card-on-file validation by merchants, and performed as the opening move of card fraud by attackers — the typology normally called **card testing**.
3. **Sense 3 — the live nominal-value go-live test payment.** Sending one real, nominal-value payment *through the real production path* — real counterparty, real scheme, real credentials, real cut-off — to establish that a payment channel actually works end to end.

The three share exactly one thing: **a tiny amount of value, deliberately chosen to be small, used as an instrument of verification rather than as a payment for anything.** That shared mechanism — not the shared name — is this guide's subject.

### 1.2 The Definitions — Senses 1, 2 and 3, and What a Practitioner Calls Them

| Sense | One-sentence definition | The phrase a practitioner actually uses | Who runs it | The party being verified |
| --- | --- | --- | --- | --- |
| **1. Micro-deposit verification** ✅ | Two (or more) small, unpredictable credits are sent to an account and the holder must report the amounts, proving they control that account. | *"micro-deposit verification"*, *"micro-deposits"*, *"two-small-deposits"*, *"ACV"* (account verification), and in India *"penny drop"* ⚠ | The onboarding/verification platform or the merchant's payment provider; sometimes the bank itself | The customer's own account (usually), or an account the customer claims to be able to receive into |
| **2. Instrument validation by tiny authorisation** ✅ | A nominal charge (or a zero/nominal authorisation or a card-setup validation) is placed against a payment instrument to establish that the instrument is live and usable. | *"card testing"*, *"carding"*, *"card validation"*, *"account testing"*, *"enumeration"*, *"card checking"* ✅ (all five synonyms are used by Stripe's own public documentation); *"zero-dollar auth"* / *"pre-authorisation"* for the legitimate merchant-side variant ⚠ | Two opposite parties: the legitimate merchant (card-on-file validation) and the fraudster (card testing) | A payment card, and behind it an issuer's authorisation decision |
| **3. Live nominal-value go-live test payment** ✅ | One real, small-value payment is sent through the real production path — production credentials, production routing, real counterparty, real cut-off — to prove the channel works end to end. | *"live smoke test"*, *"go-live test payment"*, *"penny test"*, *"test transaction"*, *"live proof"*, *"end-to-end live test"* | The bank's or platform's payment delivery team, with the counterparty's operations team | The payment channel itself: connectivity, credentials, routing, message construction, the counterparty's acceptance, and the cut-off |

The single most important consequence of that table: **the same €0.01 line can be all three things, and only context tells you which.** A €0.01 credit into a retail account is Sense 1 if the account holder is being asked to report it back, Sense 3 if it is the bank's own test against a scheme participant, and neither if it is an attacker probing whether an account exists. §5 develops what happens when a bank's monitoring has to tell them apart.

### 1.3 Why the Name Collides

The collision is not accidental, and it is worth naming the reason precisely, because it explains why the *discipline* differs so much between senses even though the *mechanism* is identical:

- **Sense 1 and Sense 2 are the same act performed by opposite parties.** A tiny credit into an account proves *the sender can reach the account*; a tiny debit against a card proves *the card can be charged*. In both cases the amount is small for the same reason: to make the probe cheap and unremarkable. The difference is not the mechanism but the **state of the relationship**: in Sense 1 the probe confirms a relationship the customer has declared, while in Sense 2 the probe *creates* a relationship the instrument holder never authorised.
- **Sense 1 and Sense 3 both use a real payment to prove a real thing, and both are legitimate.** Both send value, both create real ledger entries, both are done by a bank or a bank-adjacent party. The difference is **what is being tested**: Sense 1 tests *control of an account by a human*; Sense 3 tests *the plumbing between institutions*.
- **Sense 3 and Sense 2 both put an entry somewhere the counterparty can see, and both can be mistaken for noise.** A test payment that trips monitoring looks like a fraud probe; a fraud probe that lands under a monitoring threshold looks like a test payment. That symmetry is the whole of §5.

### 1.4 The Decoder — the Terms This Guide Uses

These ten terms recur throughout and are used throughout with exactly these meanings. Where a term is commonly used loosely, the loose usage is flagged.

| Term | What this guide means by it | Status |
| --- | --- | --- |
| **Micro-deposit** | A small credit (or small credit-and-debit pair) sent to a bank account for the purpose of verification, not payment. Dwolla's public documentation describes two deposits of less than $0.10 with random amounts, posting in 1–2 business days. | ✅ Dwolla developer documentation, retrieved 2026-09-29 |
| **Penny drop** | The Indian-market name for the same mechanism performed by a gateway rather than an ACH originator: a nominal IMPS/NEFT/UPI credit used to retrieve the account holder's registered name or to confirm the account is live. **The term itself is widely used in vendor and market material; this pass could not retrieve a primary vendor page for it** — treat any specific amount, timing or API behaviour attributed to "penny drop" as ⚠ unverified. | ⚠ flagged — no primary source retrieved this pass |
| **Name-match** | A service that asks the *receiving institution* whether the name the payer has supplied agrees with the name on the payee account, and returns a match / close match / no match / cannot-check outcome. It moves no value. | ✅ Pay.UK Confirmation of Payee (launched 2020) and EPC Verification of Payee public material, retrieved 2026-09-29 |
| **Prenote / zero-value file** | A file-based mechanism for giving the receiving institution advance notice of an entry *without moving value* — classically the zero-dollar prenotification entry in the ACH model. Its purpose is to let the receiving side reject bad account data *before* value moves. | ⚠ flagged — the concept is widely documented; the specific Nacha prenote rule text was not retrievable this pass (nacha.org returned a page-not-found for the rule page; see §16) |
| **Nominal value** | A deliberately trivial but **non-zero, real** amount — €0.01, $1.00, the smallest whole unit of the currency — used when the point of the payment is to establish that a real payment works, not to transfer wealth. The non-zero property is load-bearing: it forces a real settlement entry on both sides (§6.3). | ✅ — mechanism, not a scheme rule |
| **Pre-authorisation** | An authorisation held against a payment instrument ahead of a charge, of which the small-value form is used to validate the instrument. Used here in the merchant/card sense, and distinguished from the micro-deposit, which is a *credit* into a bank account. | ✅ Stripe public documentation on card setup and card testing, retrieved 2026-09-29 |
| **Card-on-file validation** | The legitimate merchant-side use of Sense 2: validating a stored instrument at setup so that future charges do not fail, and so the merchant does not hold dead credentials. Stripe's public documentation notes that *card setup* validation is preferred by fraudulent actors precisely because **card validation and authorisations during card setup don't typically show up on cardholder statements** — which is the cardholder-visibility difference between the legitimate and the abusive use of the same mechanism. | ✅ Stripe documentation, retrieved 2026-09-29 |
| **Smoke test / certification test** | A *smoke test* is the team's own quick "does it work at all" run; a *certification test* is the scheme's or operator's structured evidence-gathering exercise. They are not the same thing and do not prove the same thing — §9. | ✅ — distinction developed in §9 |
| **Suspense entry** | A ledger account that holds value whose ultimate allocation is not yet determined. In Sense 3 the suspense account is where the test payment is *seen* if it is booked at all — and the reason §7.2's rule exists is that a test payment that never touches suspense is invisible to the people whose job is to notice anomalies. | ✅ — the mechanism is defined in the repo's posting-engine material, cross-referenced in §10 |
| **Cut-off** | The time by which an instruction must be received for a given processing cycle. For Sense 3 the cut-off is one of the things the live test is *testing*, and it is the one that a sandbox can never test because a sandbox has no real cycle to miss. | ✅ — the concept is established in [payment_rails_guide.md](payment_rails_guide.md); specific cut-off times are scheme- and member-specific and are flagged where used |

### 1.5 The Thesis and the Boundary

**The thesis is stated in the title: the cheapest test is also the cheapest attack.** The reason is in §2, and it is a property of the payment, not of the tester: **the information a payment yields about the state of the world does not scale with its value, while the cost of making the payment does.** One cent and one hundred thousand euros carry the same structural information — the account exists, the route works, the counterparty accepts, the credentials are live — and the one-cent version costs a fraction of a basis point of the larger one, appears on no one's exception report by value, and in Sense 2 shows up on a cardholder statement as an amount no one bothers to query.

That is why the same amount, the same direction of value, and arguably the same message type can be: a customer proving ownership; a merchant checking a stored card; a bank proving its channel before go-live; or an attacker deciding which of eleven thousand stolen card numbers is worth cashing out. **The mechanism is cheap to the tester and nearly free to the attacker, and it is exactly as cheap to the attacker as it is to the legitimate user.** A control that makes the legitimate use convenient makes the illegitimate use convenient. That is the guide's unifying observation, and it is why the subject is worth a guide at all.

**The boundary, declared by name.** This guide does not re-derive what its siblings already own:

| Owned elsewhere | This guide's relationship to it |
| --- | --- |
| [payment_rails_guide.md](payment_rails_guide.md) — the rails: taxonomy (card, ACH, real-time, wire/RTGS, cross-border, CBDC), the real-time world map, message standards (ISO 20022, ISO 8583, SWIFT MT/MX), clearing and settlement, rail selection. Verified: that guide contains **zero** references to test payments, penny tests or smoke tests. | This guide owns the **testing practice that runs on those rails**. Where this guide names a rail mechanic, it is to say what a test on that rail can and cannot establish — not to describe the rail. |
| [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) — the fraud-detection discipline, the feature engineering, the CEP patterns, the monitoring architecture. It names card testing once, at line 476 ("Card testing: small amounts → large decline → large attempt"). | This guide owns the **typology and the mechanism of card testing**; the **detection machinery stays there** and is cross-referenced. §4.6 states this division explicitly and the guide does not restate detection design. |
| [micropayment_options_research.md](micropayment_options_research.md) — micropayments as a **product**: small-value payments as a business, an economics problem and a decision framework (see its §1 Overview & Problem Definition, §2 Traditional Payment Systems (Why They Fail), §10 Decision Framework). | **The distinction is definitional, and it is the cleanest one in this guide:** in a *micropayment*, the small value **is the point of the product** — the customer wants to pay a small amount and the business question is whether that can be done economically. In a *penny test*, the small value is an **instrument of verification** — nobody wants to pay anything; the payment exists to establish something else. A penny test is a payment that is not a payment. |
| [airwallex_guide.md](airwallex_guide.md) and the platform guides — the sandbox and simulated-rails offer, as those vendors describe it in their own documentation. | This guide treats the sandbox as the **simulation alternative** to a live test (§6.1, §12.4), and describes each vendor's sandbox only as that vendor's own description. |
| [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md), [financial_infrastructure_guide.md](financial_infrastructure_guide.md), [supply_chain_finance_guide.md](supply_chain_finance_guide.md) — the repo's Confirmation-of-Payee and name-match beneficiary-verification material. | This guide **cross-references and does not re-derive** it: the CoP mentions are at [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 461 and [financial_infrastructure_guide.md](financial_infrastructure_guide.md) line 413, and the name-match beneficiary-verification material is in [supply_chain_finance_guide.md](supply_chain_finance_guide.md) (line 491 and line 564: payout only to a name-matched verified account). §3.7 uses them without restating them. |
| The rails-mechanics guides ([swift_alliance_access_guide.md](swift_alliance_access_guide.md), [swiftnet_fileact_guide.md](swiftnet_fileact_guide.md), [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md), [singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md), [cnaps_guide.md](cnaps_guide.md), [nucc_netsunion_guide.md](nucc_netsunion_guide.md), [payments_hub_guide.md](payments_hub_guide.md)) and the repo's AML/KYC material. | This guide names them as the owners of their rails' mechanics and their onboarding regimes, and uses them only as the objects of a test protocol (§8). |

### 1.6 The Scope Statement — All Three Senses Are in Scope

**All three senses are in scope in this guide, and they are treated as three different practices that happen to share a name and a mechanism.** §1 must be explicit about this because the most common failure in this subject area is arguing across senses: the security property that makes Sense 1 work (the amount is a shared secret) has nothing to do with the control that limits Sense 2 (rate and velocity), and the audit discipline Sense 3 demands (a real entry in two institutions' books) is irrelevant to both. **Conflating them produces bad controls.** A team that reads "penny test" as one thing will, for example, try to stop card testing by asking the cardholder to confirm an amount, or will treat a live go-live test as a security event.

**What is in scope:** the mechanism and its economics (§2); each sense developed properly (§3, §4, §6, §7); the ambiguity the mechanism creates for a bank's own monitoring (§5); the practice per rail class (§8); certification versus production (§9); accounting, reconciliation and audit (§10); consent and data protection for Sense 1 (§11); the alternatives and why they are displacing Sense 1 in particular (§12); the anti-patterns (§13); and a worked test-protocol design (§14).

**What is out of scope, by name:** the rails themselves ([payment_rails_guide.md](payment_rails_guide.md)); the detection machinery ([financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md), and the CEP pattern at [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) line 1047); micropayments as a product ([micropayment_options_research.md](micropayment_options_research.md)); AML/KYC onboarding and sanctions screening (owned by the repo's financial-crime material); and any real institution's particular test protocol. **The only bank persona in this guide is Cymbal Bank, which is fictional, and no real bank is asserted anywhere in this guide to run any particular test.**

---

## 2. The Unifying Mechanism — Why a Tiny Value Is the Right Probe

### 2.1 The Analysis (Labelled)

> **LABELLED ANALYSIS.** The following is reasoning from the mechanism, not a claim about any scheme, vendor or institution's behaviour. It is labelled so that a reader can take the reasoning without taking it as sourced fact. Everything asserted about a specific scheme, amount, timing or message type elsewhere in this guide is sourced or flagged; this sub-section is where the guide reasons rather than reports.

Consider what a payment actually *establishes* — what you know after it completes that you did not know before. A credit transfer that reaches its destination successfully establishes, jointly and simultaneously, that:

1. the beneficiary account exists and is open to receive (or the credit would have been returned);
2. the routing information is correct at the level of the scheme's routing table;
3. the scheme message the sender constructed was accepted by the scheme;
4. the sending institution's credentials, certificates and connectivity are live;
5. the receiving institution accepts messages of that type and from that sender;
6. the counterparty's operational process actually processed the instruction;
7. the timing assumptions hold — the instruction met the cut-off, or the rail's 24/7 operation was real;
8. the funds reached a specific account, not merely a specific institution; and
9. in a name-match regime, that the name held against an account either agrees or does not.

**Now the key observation: none of items 1–9 is a function of the amount.** A credit of €100,000 and a credit of €0.01 exercise the same routing table, the same message schema, the same certificate chain, the same counterparty acceptance logic and the same cut-off. The *information* is, to a first approximation, invariant to value.

Meanwhile, the *cost* is not invariant at all. The cost of a payment has these components: a scheme or operator fee (often flat or floor-priced per item); internal processing cost; the funding and liquidity requirement; the capital and credit exposure (small if the payment is small and immediate, larger if it is not, and potentially the entire amount if the payment is irrevocable); the monitoring and review cost (a large payment attracts four-eyes and enhanced scrutiny, a small one does not); the accounting cost (a large entry needs reconciliation against a materiality threshold; a small one can be written off); and — the one people forget — **the cost of being unable to reverse it**.

Write it as a ratio. Let *I* be the information established by a completed payment and *C(v)* the cost of a payment of value *v*. Then:

> **I(v) ≈ I** for all *v* in the range where the payment exercises the same path, while **C(v)** is increasing in *v*, and includes a term that is unbounded in *v* when the rail is irrevocable.

Therefore the **information per unit of cost**, *I(v)/C(v)*, is maximised at the smallest value that still exercises the full path — and it is maximised *hardest* precisely where the cost has a large value-proportional and irrevocability term, which is to say **on the real-time and high-value rails**. This is why a penny test is not a cheap form of a real test: **at the margin, it is the strictly better test**, and a €100,000 test buys nothing a €0.01 test does not — except the ability to say you moved a large amount, which is not a property anyone needs. The same arithmetic falls out of Sense 2, where the attacker's cost per instrument tested is bounded by the smallest amount the merchant's own system will accept at all.

**And the same argument, read backwards, is the attack.** Any tester, legitimate or not, faces the same *I(v)/C(v)* ratio. Nothing in the mechanism distinguishes the tester's intent. That is the whole reason this guide exists: the property that makes the probe good makes it good for everybody.

### 2.2 The Worked Side-by-Side — the Small Probe Versus the Large Payment

The following is a **worked comparison of what each probe establishes and what it costs**. It is labelled illustrative: the *categories* of information and cost are the argument; the specific figures below are placeholders to show the shape of the trade and are **not** any institution's cost tariff. Where a real, sourced figure is available (for example, that the RTP network's published per-transaction limit is $10 million ✅), it is cited in §15 instead.

| Dimension | The €0.01 probe | The €100,000 payment | Which buys more information? |
| --- | --- | --- | --- |
| Account exists and is open | Yes (credit not returned) | Yes (credit not returned) | **Identical** |
| Routing correct to scheme level | Yes | Yes | **Identical** |
| Message accepted by the scheme | Yes | Yes, provided it is within value limits — and note the reverse: **if €100,000 exceeds a per-transaction limit or a member-restricted threshold, the large payment fails for a reason unrelated to the plumbing**, which is information *about the limit*, not about the channel ⚠ (limits are scheme- and member-specific; see §15) | The small probe is *cleaner*, because it cannot be entangled with a limit breach |
| Certificates / connectivity / credentials live | Yes | Yes | **Identical** |
| Counterparty accepts this message type from this sender | Yes | Yes | **Identical** |
| Cut-off and 24/7 operation | Yes, if sent at the boundary you actually care about | Yes | **Identical**, provided the small probe is sent at the boundary too — the *timing* of the probe, not its value, is what tests the cut-off |
| Funds reached a specific account | Yes, typically with confirmable identification and reconciliation of a trivial amount | Yes | **Identical in kind**; the small amount is *easier* to reconcile unambiguously |
| Requires four-eyes / manual release / threshold review | Usually not | Usually yes — which means the €100,000 test **does not exercise the same authorisation path as an automated payment**, adding a difference rather than a benefit | Small probe exercises the path more faithfully for low-value products |
| Liquidity / funding requirement | Negligible | Material, and on the sending side may require a prefunded position | Small probe is operationally cheaper |
| **Exposure if the rail is irrevocable** | €0.01 irrecoverable | €100,000 irrecoverable | Small probe is the *only* defensible choice (§7.3) |
| Cost of the disputes / claims process if something goes wrong | ~0 — lower than the paperwork cost | Material, and creates a customer-affecting liability, an incident, and possible compensation | Small probe is categorically safer |
| Cost to the counterparty's operations | nil-to-trivial; unlikely to be escalated | Material to them too; will be escalated; damages the relationship you are trying to establish | Small probe |
| Effect on booking, suspense and reconciliation | Still a real entry — **which is the point**: it tests the booking path (§7) | A real entry | Identical in kind |
| Effect on monitoring and AML rules | A small credit, may still trip velocity or structuring rules | A large payment will trip threshold rules and manual review | Different *rules*, same *problem*; see §5 |

**Read the column on the right as a whole.** On every dimension where the question is "does the channel work?", the €100,000 payment answers no more than the €0.01 payment. On every dimension where the question is "what does it cost to be wrong?", the €100,000 payment is worse by five to seven orders of magnitude, and on an irreversible rail it is worse by the entire amount. **The conclusion of the worked comparison is therefore not a slogan but an engineering result: pick the smallest value that still forces a real settlement entry on both sides — and on an irrevocable rail, pick nothing at all unless you are certain (§7.3).**

### 2.3 Where the Analysis Breaks Down

The analysis is a model, and it has edges. Three of them matter, and each produces a real failure mode:

- **The tokenisation limit.** The claim "the payment exercises the same path" is false if the path is *value-dependent*. Some paths are: value limits per transaction, per day, per scheme and per member; review thresholds that route large payments to a different queue; rate or value ceilings on a test-enablement profile configured by the counterparty; and products that only exist above a floor. **A €0.01 probe cannot prove a €1,000,000 payment works if the £1,000,000 case routes through a different rule path.** The honest formulation is: *a penny test proves the plumbing; it does not prove the limits.* Testing limits is a separate exercise at the boundary values — and the repo already contains the discipline for that, in [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md), whose rule-engine test matrix (line 471) already lists *"each limit boundary (daily and annual, at limit and one cent over)"*. **That is the same idea as this guide's "one cent" and it is the correct way to test a limit: at the value, and one unit over it.** The penny test is the *connectivity* probe; the boundary test is the *limit* probe; they are complementary and both are needed.
- **The minimum-value floor.** Some merchants, schemes and products have a minimum transaction value, and some authorisation paths do not exist at the smallest denomination. Where a floor exists, the practical probe is *the floor*, not "one cent" — the discipline is *as small as the path permits*, not a religious one-cent rule.
- **The non-zero requirement.** The analysis assumes the probe moves value. On some paths it does not need to: a prenote or a zero-value file establishes item 1 (account exists and is well-formed) **without moving value at all**, and where a scheme supports it, it is strictly cheaper than a credit. But it establishes *only* item 1 and a subset of item 2 — it does not establish items 3–8 for a *credit* product, because it is not a credit. §6.3 develops this; the short version is that a zero-value file and a penny test are not substitutes.

---

## 3. Sense 1 — Micro-Deposit Verification

### 3.1 The Mechanism and the Flow

The mechanism in its canonical ACH form, as the vendors who operate it describe it publicly:

1. The customer adds a bank account (routing/transit number and account number) to a service — a broker, a wallet, a lender, a payroll platform, a payables system.
2. The service initiates one or more **micro-deposits** to that account. Dwolla's public documentation states the pattern as **two deposits of less than $0.10**, with **random amounts**, which post to the customer's account in **1–2 business days** ✅ (Dwolla developer documentation, retrieved 2026-09-29).
3. The amounts are not disclosed to the customer. They must be *observed* by the customer, in the account, on the statement or in the banking app.
4. The customer reports the amounts back in the originating application. **The order in which they are entered does not matter** ✅ (Dwolla).
5. Correct entry → the account is marked verified; the service now has an account number it believes the customer controls.
6. Wrong entry → the attempt fails. Dwolla's published behaviour is **three verification attempts**, after which a max-attempts event fires and the two posted amounts can no longer be used; the documented recovery path is to **remove the funding source, wait 48 hours, re-add it, then re-initiate micro-deposits** ✅ (Dwolla). The three-attempt limit and the 48-hour cooldown are the anti-brute-force control: without them, the proof of control would be weakenable by guessing.
7. In the **sandbox**, Dwolla's documentation notes that **any amount below $0.10 will allow immediate verification** ✅ — a useful, and dangerous, illustration for §5: the sandbox deliberately removes the security property, which is exactly why a sandbox pass cannot be evidence that the *production* flow is safe.

**Variant: the same mechanism with faster rails.** Plaid's public documentation describes the same two-small-amounts idea over different rails: **Automated Micro-deposits** (1–2 business days, US), **Same-Day Micro-deposits** over Same Day ACH (US, about one business day plus waiting for user interaction), and **Instant Micro-deposits**, which Plaid documents as making the micro-deposit **over RTP or FedNow** with **manual verification of the code by the user in as little as 5 seconds** ✅ (Plaid Auth documentation, retrieved 2026-09-29). The same source states that Instant Auth covers **approximately 95%** of users' eligible bank accounts, with the remaining **~5%** served by micro-deposit or database verification methods ✅. That is the single cleanest public statement of Sense 1's *addressable share*: micro-deposit verification is what you fall back on when credentials-based or matched verification is not available.

**Variant: the credit-and-debit pair.** Some implementations post two small *credits*; others post a small credit and a matching debit, or a debit and reversal. The mechanism is identical and the trade is the same: a debit proves control by taking money (harder to ignore, more alarming to the customer), a credit proves it by leaving money (easier to accept, easier for the customer to miss).

### 3.2 Where It Is Used

- **Account onboarding for money movement** — linking a bank account to a wallet, broker, lender, payroll or marketplace payout profile, so that funds can be pulled from it or pushed to it.
- **Payee and beneficiary setup** — establishing, before a first payment, that a nominated account can receive.
- **Payout account validation in financing and supply-chain programmes** — the repo's own material describes the equivalent control in name-match terms: payout only to a **verified account**, validated against the payer's records, to prevent diversion of funds ([supply_chain_finance_guide.md](supply_chain_finance_guide.md), lines 491 and 564) — a programme can perform the same control with a micro-deposit instead of a name-match, which is precisely the choice §12 analyses.
- **Payer-side validation of a recurring collection mandate**, where the merchant needs to know that the account it will debit is real and reachable before it starts debiting.
- **Cross-border and correspondent onboarding**, where the receiving institution wants proof that a nostro/vostro instruction reaches a named beneficiary account before the relationship carries volume.

### 3.3 The Security Property — the Amount Is the Secret

The security property is worth stating exactly, because every weakness in §3.4 is a consequence of it:

> **The proof of control does not depend on a secret the customer holds. It depends on information that is only visible to a party with access to the account — the two randomly determined amounts.**

Three things follow, and all three are load-bearing:

1. **The amounts must be unpredictable.** If the deposits were always $0.13 and $0.27, the "proof" would be a public fact and the verification would prove nothing. **The unpredictability of the amount is the entire security control.** Any implementation that uses fixed amounts — or a small, enumerable set of amounts — has silently reduced micro-deposit verification to a formality, because a party who has the routing and account number but no access to the account could pass it. This is why Dwolla's documentation says *random amounts* and why Dwolla's sandbox relaxation ("any amount below $0.10 will allow immediate verification") is a documentation-level demonstration that the control and the amount-unpredictability are the same thing ✅.
2. **The lookup API is not the verification.** A vendor can offer a service that *retrieves* the account holder's registered name from an account number (the "penny drop" pattern in some markets). That is a **name-match by another route**, and it establishes a *different* property: it establishes that the account is live and what name is registered against it, **not** that the requester controls it. Confusing the two is a recurring error, and it is the reason §1.4 flags "penny drop" rather than folding it into "micro-deposit" ⚠.
3. **The attempt limit is part of the control.** With few, small, random amounts there is a tiny but non-zero guessing space; the control requires a bounded number of attempts and a cooling-off period, as Dwolla's published three-attempt limit and 48-hour recovery path illustrate ✅. Removing the attempt limit would convert an information-theoretic proof of control into a rate-limited guessing game.

### 3.4 The Weaknesses, Developed Properly

Micro-deposit verification is not a weak control; it is a **strong control with a high latency cost and several ugly edge cases**, and it is being displaced for reasons of speed, experience and — importantly — because a better question is available (§3.7). The weaknesses, in the order they actually bite:

**1. Latency.** The ACH form takes **1–2 business days** for the deposits to post ✅ (Dwolla's published timing), plus however long the customer takes to look. That means an onboarding funnel with a multi-day dead zone in the middle, during which the customer is *not yet* able to transact. Plaid's Same-Day Micro-deposits reduce this to about a business day; Instant Micro-deposits reduce it to *five seconds* by using RTP or FedNow ✅ — but only where the receiving institution is reachable on those rails, which is the constraint that makes the fast variants a partial solution rather than a replacement. **The latency is a property of the rail used for the probe, not of the mechanism**: this is the clearest case in the guide where the choice of rail is the design decision.

**2. User experience and support burden.** The customer must: notice two small credits, distinguish them from other activity, remember the amounts, return to the originating application, enter two amounts in cents, and do so within three attempts. Every one of those steps is a drop-off point. The failure mode is not "the customer is untrustworthy" but "the customer could not see the credits" or "the customer transposed two numbers", and the cost lands on the support queue. This is why onboarding conversion, not security, is the usual reason programmes abandon micro-deposits.

**3. The amount-unpredictability dependency (§3.3).** The control is only as strong as the unpredictability of the amounts and the boundedness of the attempts. Both are implementation properties that can be lost in a migration, an integration change or a sandbox-to-production configuration drift.

**4. The reconciliation-visibility weakness.** The micro-deposit is a real credit into a real account. If the receiving side (the customer's own bank) does not recognise it, it becomes — from the *receiving* institution's point of view — an unidentified small credit, which is exactly the class of item that lands in suspense (§10). At scale, a programme that generates micro-deposits produces a stream of small unexplained credits across the banking system. This is one of the real, rarely-discussed externalities of Sense 1, and it is the *same* externality as Sense 3: **a test payment is a real payment, and real payments that nobody expects become exceptions for the people who did not know a test was running.**

**5. The "who is actually being verified" weakness.** The mechanism verifies that *somebody with visibility into the account* can read two amounts. In the vast majority of cases that is the account holder. It does not verify beneficial ownership, it does not verify that the person entering the amounts is the account holder rather than someone with access to a statement, and it does not verify that the account is the *right* account for the payment's purpose.

### 3.5 The Failure Cases

These are the cases that produce support tickets and, occasionally, false verification:

- **Joint accounts.** Where two holders share an account, either can perform the verification, and the verification is silent about which one did. For a control that is supposed to establish *control of the account*, joint-account verification establishes "at least one of the holders, or someone who could see the statement".
- **Statements the holder cannot see.** Where the holder cannot access the statement, or where the credits are presented in a way the holder does not recognise — under a descriptor that names the originating platform, or aggregated under a service name — the legitimate holder fails a test they are entitled to pass. This is a **usability failure that masquerades as a security success**: the control fired, the customer was honest, and the customer was rejected.
- **Accounts where the credit is aggregated or netted.** If the receiving institution aggregates small credits under a service-fee line, or if the customer's view is of a balance rather than a transaction list, the two amounts may be individually invisible. The control then depends on an information display the originator does not control.
- **Posting delay and cycle interaction.** Where the credit lands *after* the customer has already looked, the customer concludes nothing arrived and abandons the flow — a timing failure on the originator's side that presents as customer error.
- **Product mismatch.** Where the account is a product that does not accept third-party credits of the type sent (a collection account, a loan account, an account closed to credits), the micro-deposit returns — which is *valid information* (item 1 of §2.1 is answered negatively), as long as the return is recognised as an answer rather than as a system error.
- **The sandbox/production gap.** In a sandbox the amounts are usually relaxed or fixed ✅ (Dwolla documents any amount below $0.10 as sufficient in sandbox), which means the sandbox validates the *integration* but cannot validate the *control*. Any programme that treats a sandbox micro-deposit pass as evidence that production verification is sound has tested the plumbing and believed it tested the lock. This is the Sense 1 instance of §6.1's sandbox-versus-network argument, and §9 generalises it.

### 3.6 Consent and Data Exposure — the Amount Goes on Someone's Statement

The mechanism has an unusual property that deserves its own paragraph, because it is not a bug and cannot be designed away: **to prove that someone controls an account, the originator writes information onto that account's record — a line item that the account holder must then read back.** Consider what that means:

- **The verification *creates data* on the receiving side.** The account holder has not asked for a payment, but receives one, with a description written by the originator. The account now contains a small unexplained credit, and the description is whatever the originator's payment reference system produced. Where the reference is a meaningful string, the account holder's statement may disclose the *identity of the requesting service* to anyone who later sees the statement — a spouse, an accountant, an employer, a landlord, in a litigation disclosure, or a bank clerk reviewing a transaction.
- **The proof is the *content*, not the *fact*.** This is why §3.3 insists the amounts are the secret: if the fact of the credit were the secret, then any statement viewer would have it. The design correctly makes the *amounts* private, but the *existence and description of the credit* is, by necessity, public to the account and to anyone entitled to look at it.
- **There is no equivalent of a "quiet" micro-deposit.** Unlike a name-match — which is a query-and-answer that leaves no transaction — a micro-deposit leaves a mark. The mark is small, but it is a mark, and its presence is why the consent question is not rhetorical.

**The consent the customer gives** is the customer's authorisation for the service to initiate a credit into a nominated account. §11 develops the substantive questions (what exactly is being consented to; what is disclosed; what is retained; and the difference between verifying an account the customer owns and probing one they do not), and the guide's position is stated there plainly rather than here, because it deserves the space.

### 3.7 The Displacement — Name-Match, Account Checking and Instant Confirmation

Micro-deposit verification is being displaced, and the displacement is worth understanding precisely because it is *not* that name-match is a newer version of the same thing. **A name-match asks a different and better question.**

**What a name-match asks.** Confirmation of Payee, as Pay.UK publicly describes it, is an **account name-checking service** that checks an account name (including a personal/business indicator) **before** initiating or collecting a payment, to reduce misdirected payments and provide assurance that a payment is going to the intended account holder ✅ (Pay.UK, retrieved 2026-09-29). Pay.UK's public material states it **launched in 2020**, that **over 300 organisations** have implemented it, that **more than 2 million checks** are completed every day, that it is an **API-based peer-to-peer service with no central infrastructure** but with a directory of participating organisations, and that the **Payment Systems Regulator mandated its expansion in 2024** ✅. It is also documented as available through a **direct** model (build in-house or via a third-party service provider, with the PSP as a Direct Participant) or an **aggregator** model (onboarding through a CoP Aggregator, with the PSP as an Indirect Participant) ✅. Pay.UK separately documents **Payer Name Verification** for checking names before setting up or amending Bacs Direct Debit payments ✅.

**What the European equivalent asks.** The EPC's **Verification of Payee** scheme, as the EPC publicly describes it, provides **inter-PSP rules, practices and standards** for SEPA, lets the payer's PSP ask the payee's PSP to verify the IBAN and name (and optionally an unambiguous identification code such as a VAT number, LEI or social-security code), and returns a response such as **match, no match, close match with the name, or match/verification check not possible** ✅ (EPC, retrieved 2026-09-29). The EPC is explicit about the boundary, and the sentence is worth quoting in substance: it is a **messaging functionality**, it is **not a payment means or payment instrument**, and **it cannot be relied upon to identify a private or legal person** ✅. The EPC's published evolution calendar lists **publication of VOP rulebook version 1.1 on 16 March 2026**, **an effective date of 20 September 2026**, and **v2.0 publication expected at the end of November 2026** ✅ — a concrete reminder that this area is version-specific and moving.

**Why the name-match wins, on the merits.**

1. **Speed.** A name-match is a query with an immediate answer; a micro-deposit is a payment with a cycle time. The difference is days versus seconds on the slow path, and seconds versus single-digit seconds on the fast path — but on the fast path the micro-deposit has spent a *real payment* to get there, whereas the name-match spent a *message*.
2. **No value moves.** No ledger entry, no suspense item, no unidentified credit in someone's statement, no funding, no liquidity, no reversal problem. The externality of §3.4(4) disappears.
3. **It asks the *right* question.** Micro-deposits prove **control**: "can you see this account?" Name-match proves **correspondence**: "is this the account of the person you said you are paying?" For the dominant use case — I am about to pay a supplier, a landlord, a new payee — correspondence is the question that matters, and control is a proxy that can be satisfied while the correspondence is wrong. A customer can verify control of an account that is not the intended beneficiary's; a name-match is the check that catches exactly that. **This is the observation the task asks for and it is the honest reason for the displacement: the name-match asks a better question than the deposit does.**
4. **It scales without a support queue.** No "I can't see the amounts" tickets.

**What the name-match cannot do, and why micro-deposits have not vanished.** A name-match requires a *directory and a responding institution*. It answers only where both sides participate: Pay.UK's CoP is a UK domestic overlay service with an onboarding regime ✅, and the EPC's VOP is a SEPA scheme with a published effective date ✅; outside those footprints, or where the receiving institution is not a participant, there is no one to answer. A name-match also proves nothing about *control*: it says the name on the account matches, not that the person asking is entitled to transact on it. And it is a data disclosure of its own (§11). Hence the current settlement: **name-match where the scheme and the directory exist; open-banking instant confirmation where the API exists; database or matched verification where the network exists** — Plaid's own public documentation describes exactly this ladder, with Instant Auth at ~95% coverage, Instant Match at ~800 additional US institutions, database verification, and micro-deposits as the remaining path ✅ — **and micro-deposit verification where none of the above is available, or where the control question genuinely is the question.**

**The repo's material is not re-derived here.** The repo's Confirmation-of-Payee mentions are at [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 461 and [financial_infrastructure_guide.md](financial_infrastructure_guide.md) line 413, and its name-match beneficiary-verification material — payout only to a name-matched, verified account — is in [supply_chain_finance_guide.md](supply_chain_finance_guide.md) at lines 491 and 564. Read those for the mechanism; §12 below compares them as *alternatives to a penny test* and stops there.

---

## 4. Sense 2 — Card Testing as a Fraud Typology

### 4.1 The Mechanism

Sense 2 is Sense 1's mechanism with the value moving the other way and the relationship inverted: instead of a small credit *into* an account the customer claims to own, it is a small **authorisation against a payment instrument**, and its purpose is to establish whether that instrument will accept a charge. When the instrument is stolen, this is **card testing**, and the attacker is establishing one bit of information per attempt: *live* or *dead*.

The publicly documented description is unambiguous, and it comes from a processor that sees the traffic. Stripe's own documentation defines card testing as fraudulent activity "where someone tries to determine whether stolen card information is valid so that they can use it to make purchases", and states the other common terms for it as **"carding", "account testing", "enumeration" and "card checking"** ✅ (Stripe documentation, "Protect yourself from card testing", retrieved 2026-09-29). The same source describes the two mechanisms the attacker uses:

1. **Card setup.** Stripe's documentation states this is *preferred by fraudulent actors* because **card validation and authorisations during card setup don't typically show up on cardholder statements**, which reduces the likelihood of the cardholder noticing and reporting ✅.
2. **Payments.** Stripe's documentation states that card testers **create small amount payments, which cardholders are less likely to notice and report as fraudulent** ✅.

And the industrial shape of the attack: testers use **scripts to test a large amount of card information at once**, collecting **3D Secure authentication or issuer responses** to determine which card data is valid, after which valid cards are **cashed with merchants or resold on the dark web** ✅ (Stripe). **Note what the attacker is buying with the small charge: not goods, not a relationship — an issuer's answer.** The charge is a question, and the answer is the product.

### 4.2 Why the Attacker's Test Is Cheap and the Merchant's Cost Is Not

The asymmetry that makes Sense 2 a fraud typology rather than an annoyance is that **the attacker pays per *attempt* and the merchant pays per *successful-looking event***. Concretely, and staying within what sources establish:

- **The attacker's unit cost is the smallest amount the target's own system will accept**, plus whatever the stolen-card supply costs. Stripe's documentation says the small-amount route is chosen because it is **less likely to be noticed and reported** ✅ — the attacker is not minimising the *fee*, they are minimising the *probability of being noticed*, and both happen to be minimised by the same choice of a small amount. §2's ratio argument is therefore sharper in Sense 2 than anywhere else in this guide: the attacker is optimising *information per unit of attention*, and a penny is the optimal probe for exactly that objective.
- **The merchant's cost is not per attempt.** Stripe's documented consequences are the merchant-side ledger, and they are worth listing because each one is a cost the merchant bears while the attacker bears almost nothing: **disputes** (successful test payments get noticed by customers and surface as **Early Fraud Warnings** or fraudulent disputes) ✅; **higher decline rates** — card testing associates a large number of declines with the merchant, which "might damage the reputation of your business with card issuers and card networks, which makes all of your transactions appear riskier", so the merchant's decline rate for *legitimate* payments can rise **even after the card testing ceases** ✅; **additional fees**, including authorisation fees under custom pricing and dispute fees ✅; **infrastructure strain**, because card testing "usually results in numerous network requests and operations" that can "overburden your infrastructure and disrupt legitimate activity" ✅; and **damage to data quality**, because "revenue from card testing might look like good, new customers in your data", making it hard to see real business growth ✅.
- **The merchant also risks programme-level consequences.** Stripe's documentation states that a large amount of card testing resulting in Early Fraud Warnings or Disputes "might, for example, enlist you into **Card Monitoring Programmes**" ✅. Whatever the commercial detail of any particular programme, the shape is the point: **the merchant's penalty is not the test charge, it is the aggregate behaviour pattern attached to the merchant's identity.**

**This is the cleanest instance of the guide's thesis in a fraud setting.** The attacker runs a probe that costs a few cents and answers a question worth a great deal; the merchant absorbs the consequences of *being the venue* in which that question was asked. Neither party paid for a purchase. Gold under §2: the merchant's cost is not proportional to the attacker's value, and cannot be bounded by capping the amount.

### 4.3 The Observable Pattern at the Level Sources Support

This sub-section deliberately lists only what is sourced, and it is short on purpose — **the detection discipline, the feature engineering and the monitoring architecture belong to [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md), and are not restated here.** What sources establish about the *pattern*:

| Observable | What the source says | Status |
| --- | --- | --- |
| **Small amounts** | Stripe: card testers "create small amount payments, which cardholders are less likely to notice and report as fraudulent". Stripe's identification guidance lists "a spike in suspicious payments **with low transaction amounts**, often with nonsensical customer names and email addresses" as a symptom | ✅ Stripe documentation, retrieved 2026-09-29 |
| **Volume in failed authorisations rather than in successes** | Stripe: "You can identify most card testing activity by a **significant increase in failed authorisation and payments**"; symptoms include a **spike in failed or blocked payments**, a **spike in requests with 402 errors**, and in particular an outcome of **`generic_decline`** | ✅ Stripe documentation, retrieved 2026-09-29 |
| **Scripted, high-volume, programmatic** | Stripe: "fraudulent actors use **scripts to test a large amount of card information at once**", and card testers "can use your publishable key and use it to retry a large number of payments on your website". Stripe also states plainly that "simple firewall rules or filters based on a single heuristic such as IP addresses are usually not sufficient" | ✅ Stripe documentation, retrieved 2026-09-29 |
| **Dispersion across merchants and geographies** | Stripe frames the consequence as **ecosystem-wide** ("both Stripe and our financial partners want to help you stop it", and testing "has negative impacts on the financial system as a whole") which is consistent with dispersion, but does not itself quantify it. The repo's own stream-processing material describes the *pattern primitive* directly: **"Rapid successive transactions: 3+ in 60 seconds at different merchants"** and **"Geographic hopping: geographically impossible sequence in short time"** | ✅ Stripe (ecosystem framing) + ✅ the repo's pattern list at [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md); ⚠ no source in this pass quantified dispersion |
| **The decline-then-large-attempt signature** | The repo already names it, twice and consistently: **"Card testing: small amounts → large decline → large attempt"** at [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) line 476, and the same pattern in the CEP list at [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) line 1047 | ✅ repo-internal, cross-referenced not re-derived |
| **The setup-channel variant that leaves no statement trace** | Stripe: card setup validations "don't typically show up on cardholder statements", which is why fraudulent actors prefer it — **the observable consequence is that a cardholder-visible alert will not fire for this variant at all**, and detection has to sit at the merchant/acquirer layer | ✅ Stripe documentation, retrieved 2026-09-29 |

**Note the discipline in that table.** No rate, no count, no percentage and no loss figure appears, because **none was available at source in this pass** — Stripe's public documentation describes the pattern qualitatively and publishes no card-testing statistics; the absence is recorded in §16 rather than filled with a plausible number.

### 4.4 What the Merchant and Acquirer Actually Suffer

At the level publicly established, and in the order that they typically become *someone's job*:

1. **Disputes and early warnings.** Stripe names Early Fraud Warnings and fraudulent disputes as a direct outcome, and recommends **refunding fraudulent payments to avoid disputes** ✅. Each of these is an operational workflow with a clock on it.
2. **Fee exposure.** Authorisation fees (custom pricing plans) and dispute fees are named by Stripe as additional-fee outcomes ✅.
3. **Decline-rate reputation, which outlives the attack.** Stripe's documentation is explicit that a high decline rate "might damage the reputation of your business with card issuers and card networks, which makes all of your transactions appear riskier", and that this "can result in an increased decline rate for legitimate payments, **even after card testing ceases**" ✅. **The legacy effect is the part that makes this a governance issue rather than a support ticket**: the attack ends, the penalty does not.
4. **Monitoring-programme and acquirer attention.** Stripe names Card Monitoring Programmes as a possible escalation ✅. The shape any practitioner recognises: the acquirer's risk function becomes involved, and the remediation is a control programme with evidence obligations.
5. **Infrastructure and engineering load.** "Numerous network requests and operations" that can overburden infrastructure and disrupt legitimate activity ✅ — a denial-of-service by product, arriving through the payment endpoint rather than the network edge.
6. **Analytics corruption.** Test revenue resembling new-customer revenue ✅ — which is a *decision* risk: pricing, growth and cohort decisions get made on contaminated data.
7. **For the acquirer, and for the issuing bank:** Stripe's public position that "merchants, card networks, and Stripe share responsibility to prevent it" ✅ names the shared-responsibility structure without quantifying any party's burden. In Sense 2, the issuer whose card is being enumerated is the party with the earliest, cheapest detection opportunity — it sees many attempts on its own BIN range across merchants — and that is precisely the bank-side monitoring question §5 takes up.

### 4.5 The Cardholder-Facing Version

There is a legitimate-instrument, cardholder-facing half of Sense 2 that is usually discussed separately and should not be. **A nominal charge on a statement is, for the cardholder, an early warning** — and, for the attacker, the thing the "card setup" route is designed to avoid. Stripe's documentation states the mechanism directly: the setup-validation route is preferred by fraudulent actors *because those validations typically do not appear on cardholder statements*, thereby reducing the chance the cardholder notices ✅. Two consequences follow, and both are actionable:

- **The absence of a visible nominal charge is itself a signal.** Where an issuer or a merchant relies on "the cardholder will spot a penny charge", that control is defeated by exactly the variant that avoids the statement. Stripe also notes the small-amount payments route exists precisely because such amounts are **less likely to be noticed and reported** ✅ — so even the variant that *is* visible is engineered to be missed.
- **On the legitimate side, a small charge is a normal, useful and expected artefact.** Card-on-file validation exists so that future charges do not fail; the merchant has a genuine interest in not storing dead credentials, and the cardholder has an interest in not being interrupted later. **The mechanism is not the problem — the absence of the cardholder's authorisation for it is.** §11's distinction between verifying what the customer owns and probing what they do not applies here in card form, and §5 is where the two senses collide.

### 4.6 The Typology by Mechanism, and Where the Detection Machinery Lives

**Stated plainly and by mechanism, the typology is this:**

> **Sense 2 is the use of a deliberately minimal authorisation to extract, at the lowest possible cost and the lowest possible visibility, a binary answer from an issuer: whether a payment instrument will accept a charge. It is used legitimately to validate an instrument the holder has authorised, and abusively to validate instruments the holder has not. The mechanism is identical in both cases; only the authorisation differs.**

**Where the detection machinery lives.** This guide does *not* cover how to detect, score, feature-engineer, alert on, throttle or investigate card testing, and deliberately so: the detection discipline is owned by [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md), with the stream-processing pattern primitives in [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) (line 1047) and the rate-limiting mitigation angle in [../technology/distributed_rate_limiter_guide.md](../technology/distributed_rate_limiter_guide.md). What this guide does own is the **typology and the dual-use consequence**, which is §5 — because a bank that has never thought about Sense 1 and Sense 2 as one mechanism, in one alerting estate, will discover the connection the hard way.

---

## 5. The Dual-Use Problem in the Bank's Own Monitoring

### 5.1 The Same Mechanism, Opposite Parties

Put the two senses side by side and the problem states itself:

| | **Sense 1 — the legitimate probe** | **Sense 2 — the abusive probe** |
| --- | --- | --- |
| What is being established | That an account exists and is controlled by the customer | That a stolen instrument is live |
| Who initiates | The service, at the customer's request, against the customer's own account | The fraudster, without authorisation, against someone else's instrument |
| Value direction | A small **credit** into a bank account | A small **debit/authorisation** against a card |
| Amount chosen small because… | the test must be cheap and the amount must be unremarkable (and in Sense 1, *unpredictable*) | the test must be cheap and the amount must be unlikely to be noticed and reported ✅ |
| Counterparty's experience | An unexplained small credit appears on their statement; they are asked to report it | An unexplained small charge appears (or, in the setup variant, nothing appears at all) ✅ |
| What the bank sees on its own leg alone | A small credit to a personal account | A small authorisation on a card |

**The two rows in bold are the operational problem.** A bank's payment-monitoring estate sees *value, counterparty, instrument, channel, cadence and customer* — not intent. On the dimensions it can see, a micro-deposit into a customer's account and a card-testing authorisation against a customer's card can look like two instances of the same benign category: *small amount, low risk, no action*. **The bank's own verification traffic is indistinguishable in amount from the attacker's validation traffic, because the whole point of both is that the amount carries no signal.**

### 5.2 What Actually Distinguishes the Two — and What Does Not

**What does NOT distinguish them:** the amount. The instrument type is *mostly* but not entirely distinguishing (Sense 1 runs on account-to-account rails, Sense 2 predominantly on card rails — but a small debit against a bank account is a perfectly good validation probe too, and Sense 1's variants now run over instant rails ✅, which removes the cleanest historical separator). The direction of value is not distinguishing either: Sense 1's credit-and-debit variants and Sense 2's refund-then-charge sequences both invert it. **Amount, direction and channel are all features of the mechanism, and the mechanism is deliberately featureless.**

**What does distinguish them, at the level sources support:**

| Discriminator | Why it works | What it is not |
| --- | --- | --- |
| **The relationship and its expectation** | The RTP network's public rule for test payments effectively turns on relationship: the rule permits a payment message to determine whether an account number is valid "when an account number has been given to the Sender by an intended Receiver **who is expecting to receive one or more Payments**, ACH credit entries, or other credit payments from the Sender", and RTP's own Compliance Guide example is a broker sending **a small test RTP Payment** to confirm the routing and account information a consumer gave it, **where the consumer is expecting to receive an RTP Payment** from that broker ✅ (RTP Network Compliance Bulletin 1-2024, dated 5 September 2024, retrieved 2026-09-29). The rule's test is **expectation**, i.e. a pre-existing declared relationship — not the amount | Not a bank-side discriminator a monitoring system can compute from a transaction alone; it is a *policy* discriminator that has to be engineered into the process |
| **Counterparty pairing and history** | Sense 1 is a probe between a customer and a service the customer has an active onboarding relationship with; Sense 2 is a probe by a counterparty the cardholder has no relationship with | ⚠ Not verified in this pass as a *control* — it is the standard operational discriminator and is stated here as reasoning, not as a sourced scheme rule |
| **Cadence and dispersion** | A legitimate verification is a single probe (or a small number) against one account for one customer; the abusive pattern is *volume*, scripted, and dispersed — "scripts to test a large amount of card information at once" ✅ and, in the repo's pattern language, "3+ in 60 seconds at different merchants" and geographic hopping ✅ | Dispersion is a *pattern* signal, which is exactly why the detection machinery belongs to [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) and not here |
| **Originating identity and credential** | A verification probe from an established, credentialled originator against its own customer base is a different event from a probe from an anonymous or newly-seen originator ✅ (Stripe names the misuse of a *publishable key* and leaked *secret keys* as the enablers of card testing, and states that "when your credentials are leaked or stolen, card testers can create payments and set up cards using your secret key") | The distinction is only available if the bank can see the originating credential's reputation — which is a capability, not an inference |
| **The downstream confirmation step** | Sense 1 has a **built-in, human-gated completion**: the amounts must be reported back, and the proof is the reporting (§3.3). Sense 2 has no such step and needs none — one bit is the whole answer | The presence of a confirmation step is visible to the *originator*, not to the receiving bank or issuer |

**Read the "not" column as the honest limit of each row.** Every discriminator that works is either a *policy* artefact (relationship, mandate) or a *pattern* artefact (volume, dispersion, credential reputation) — and neither is visible at the level of the single small transaction. **The amount is the one feature that is always visible and always useless.**

### 5.3 An Unresolved Ambiguity, Stated Plainly

**This is a genuine unresolved operational ambiguity, not a solved problem, and the guide will not pretend otherwise.**

Say it in the form an operator would say it: *a small credit arrives at a retail account. Is it the customer's own account-verification flow, a third party verifying that this customer exists and will accept credits, or a fraudster mapping accounts? From the receiving bank's leg alone — value, counterparty, channel, reference — there is no reliable way to tell, and the reference text is the only clue, which is why the effort to put a meaningful string there is so tempting. That temptation is precisely what the RTP rules forbid (below).*

The consequences are concrete rather than philosophical:

- **A bank cannot use amount to triage.** A "small-amount, no action" rule — the efficient default — is exactly the rule that files both the legitimate and the abusive probe in the same silent bucket.
- **A bank cannot demand the answer from the originator either.** The originator can *state* that a probe is its own verification flow; that statement is an assertion, and the receiving bank has no independent way to verify it. This is where weak whitelisting is born (§5.5).
- **Regulation and rules have moved on the *permission* question without removing the ambiguity.** The RTP rule cited above settles *permissibility* ("You may not do this") but not *detectability* ("Here is how the receiver tells"). The prohibition is real and enforceable — the RTP Compliance Bulletin cites "the potential for penalties for non-compliance under the rules enforcement provisions of section X of the RTP Operating Rules" ✅ — but a rule cannot give a receiving bank a signal it does not have. Nacha has similarly hardened the monitoring obligations on the ACH side over 2026: the RISK MANAGEMENT TOPICS **Fraud Monitoring Phase 1** rules took effect **20 March 2026** and **Fraud Monitoring Phase 2** on **19 June 2026** (with a practical effective date of **22 June 2026**, as **19 June is a federal holiday**), alongside **Company Entry Descriptions** rules effective **20 March 2026**, all described as part of "a larger Risk Management package intended to reduce the incidence of successful fraud attempts and improve the recovery of funds after frauds have occurred" ✅ (nacha.org rules index, retrieved 2026-09-29). **The direction of travel is towards more monitoring obligation, on the same featureless transactions.**
- **The ambiguity has a *cost*, and it is not symmetric.** False negative on a Sense 1 probe: a customer cannot complete onboarding. False negative on a Sense 2 probe: an instrument gets cashed out. Both cannot be optimised with one amount-based rule. So the honest operational answer is **not a better threshold but a set of relationship- and pattern-based controls outside the payment amount**, which is a design statement this guide can make while leaving the construction to the detection discipline.

### 5.4 The Collision with the Bank's Own Monitoring

Now the collision, which is where this stops being a fraud-systems conversation and becomes a delivery-programme one. **A bank's own go-live test payment is designed to be indistinguishable from an ordinary payment — that is the point (§6.3) — and a bank's own fraud and AML monitoring is designed to alert on payments that look odd.** Therefore:

1. **The test payment will trip the bank's own monitoring.** It is a first-time counterparty, possibly a new channel, possibly a new corridor, deliberately small, possibly at an unusual hour chosen to hit a cut-off boundary, possibly repeated. That is, feature for feature, a plausible alert. On a **24/7 rail with no maintenance window** — TCH's own public material for the RTP network claims availability "around the clock, including bank holidays, weekends and after hours" and "100% uptime, zero scheduled downtime" ✅ (The Clearing House, retrieved 2026-09-29; the vendor's own description) — there is no quiet hour in which to hide, so the collision is guaranteed rather than occasional.
2. **The test payment may also trip sanctions and AML screening**, because the screening overlay usually runs on the same leg: a first-time beneficiary and a new corridor with no transaction history is a legitimate screening workload, not a false positive to be silenced.
3. **The predictable remedy is to annotate or whitelist the test traffic** — put the test's counterparty, or its reference pattern, or its amount, on a list that suppresses alerting. **This is the moment the control is created, and it is also the moment the hole is created**, because a suppression rule is indistinguishable, to everyone downstream, from a control that has been switched off.

### 5.5 The Whitelist That Becomes a Hole — the Guardrails

The remedy is not to refuse to annotate. Refusing produces the opposite failure: an un-annotated test that generates alerts the team learns to dismiss, which is §13's "test payment invisible to reconciliation" in monitoring form. **The remedy is to annotate tightly enough that the annotation cannot outlive the test.** Stated as guardrails, each of which is a testable property:

| Guardrail | The property it must have | Why |
| --- | --- | --- |
| **Time-bounded** | The suppression **expires on a date**, automatically, and the expiry is visible in the control register | A whitelist with no expiry is a permanent control exemption created for a one-week test — §13's anti-pattern |
| **Mandate-bound** | The suppression references **a specific test mandate** — a named change, a named change record, a named counterparty, a named test window, a named test owner | So that when the mandate closes, the control closure is a step in the closure checklist rather than a memory test |
| **Counterparty-and-purpose-scoped, not amount-scoped** | Suppress on **(counterparty, channel, product, direction, test reference)** — **never on amount** | An amount-scoped suppression is the worst possible design: it silences every real payment that happens to be small, i.e. it blinds the monitoring exactly where §5.1 says the mechanism is featureless |
| **Non-silent** | The suppressed events are still **recorded and reviewable**, with a marker, rather than dropped | The purpose is to stop the *investigation workload*, not to stop the *visibility*. Suppression that loses the record makes the test unauditable (§7.5) |
| **Reviewed at closure** | Evidence that the suppression was **removed/expired** is part of the go-live evidence pack | The evidence pack is what converts "we did the test" into "we can prove the test and its containment" |
| **Not written into the payment** | The annotation lives **in the bank's own monitoring and records**, not in the payment's own data fields | This is a scheme-rule point, not merely good practice: RTP's public Compliance Bulletin states that the **"Debtor name" field in the pacs.008 message (index 2.855) should be used to identify the Sender** and **not for other purposes, such as to include a verification code**, noting that a verification code in that field "may make it difficult for the Receiver to recognize the RTP Payment as being related to an expected payment from the Sender" ✅ (RTP Compliance Bulletin 1-2024, 5 September 2024). **The scheme has already forbidden the most natural annotation method.** Any annotation must therefore be out-of-band |

**The formulation to keep:** *the annotation is a control, and a control without an expiry and a mandate is not a control — it is a hole with documentation.*

---

## 6. Sense 3 — The Live Go-Live Test Payment

### 6.1 The Sandbox-Versus-Network Argument

**A sandbox proves your code. It does not prove the network.** This is not a slogan about testing culture; it is a precise statement about what a sandbox physically is: a simulated counterparty, under the sandbox provider's control, with sandbox credentials, answering on sandbox time. The vendors describe their sandboxes exactly that way — Dwolla's documentation calls the sandbox "a free, full-featured environment that simulates real API interactions, allowing you to build, experiment, and validate your application before going live" ✅, and, tellingly, its micro-deposit sandbox **relaxes the verification control entirely**, accepting **any amount below $0.10** for immediate verification ✅ (Dwolla, retrieved 2026-09-29). That relaxation is the whole argument in one line: **the sandbox deliberately removed the security property of the production flow (§3.3), which is a legitimate convenience for testing an integration and a proof that a sandbox pass is not evidence about production behaviour.**

Four things a sandbox structurally cannot establish, each of which is the actual value of a live test:

1. **That the counterparty accepts your file or message.** In a sandbox, the "counterparty" is the sandbox's own stub, built to accept what the sandbox documentation says. The real counterparty's receiving application, its validation rules, its mapping of your fields into its own processing, its tolerance for your narrative and reference content, and its *operational* willingness to act on what you send — none of that is in the sandbox. The real one accepts or rejects **you**, not your schema.
2. **That the scheme accepts your routing.** A sandbox does not route. Scheme routing is a live directory question: is the sending participant enabled, is the beneficiary institution reachable for this message type, is the routing/transit number valid **and enabled**, is the corridor open. TCH's public material makes the reachability point explicitly in a related context — the Compliance Bulletin notes that "the fact that an Account is valid and active for receiving RTP Payment Messages does not ensure that the account is also reachable through other payment networks" ✅ (RTP Compliance Bulletin 1-2024) — i.e. **reachability is per-network, and it is a live property.** RTP also publishes an enabled routing/transit-number list ✅, which is the artefact a live test is really validating against.
3. **That certificates and connectivity are live.** In a sandbox, the credentials are sandbox credentials, and by construction they work. In production the questions are: is the certificate chain valid, is it the right certificate for the environment, has it been exchanged with the counterparty, is the IP allow-list correct, is the channel is up *now*, and does the counterparty's operator accept your sender identity. **All of these are facts about the world, not about your code.**
4. **That the cut-off is what you think it is.** A cut-off is an operational property of a live cycle: the time by which an instruction must be received for a given processing cycle, on a given day, in a given time zone, under that day's calendar. A sandbox has no cycle to miss. Closely related is the 24/7 question on instant rails — the vendors' own claim is instructive precisely because it is the kind of thing that must be *tested*, not *read*: TCH's public material asserts "100% uptime, zero scheduled downtime" and continuous availability ✅ (The Clearing House's own description), and if a bank's operating model depends on that, the dependency is established by running a payment at an awkward hour, not by reading the claim.

**The asymmetry to state in a steering committee:** a sandbox pass tells you that the *code* is not obviously broken. It tells you nothing about whether the *money* will move. Those are different risks with different consequences, and only one of them is cheap to discover in production.

### 6.2 What Each Hop of the Chain Establishes

A live test payment is a chain, and **each hop must be separately established, because the failure modes are different and the owner of each hop is different.** This is the table to walk through in the go/no-go.

| Hop | What the hop is | What the live test establishes at this hop | Failure looks like |
| --- | --- | --- | --- |
| 1. Origination | The instruction is created in the bank's own channel/product | That the product constructs a valid instruction from real user input, including the naming and reference fields that the immediate counterparty will read | Rejected at construction; invalid character set; missing mandatory field |
| 2. Internal routing | The instruction is routed to the payment platform, passed limits, screening, monitoring and authorisation | That the *whole internal* path works — including, critically, that the test does not legitimately trip a control it should not (§5.4) | Held in a control, rejected by screening, stuck in an unmonitored queue |
| 3. Message construction | The instruction is rendered as the scheme's message (ISO 20022 or the applicable standard) | That the constructed message is schema-valid, that mandatory fields are populated the way the counterparty expects, and that optional content that matters in practice is present | Schema validation failure; field content that the receiver cannot map |
| 4. Connectivity and credentials | Certificates, keys, HSM, channel, IP allow-list, sender identity | That the bank's production credentials are live, accepted and correctly provisioned for **this** counterparty and environment | Channel down, certificate rejected, sender not recognised |
| 5. Scheme/operator acceptance | The scheme or operator receives the message | That the sending participant is enabled for the message type and the counterparty is reachable, and that the scheme's own validations pass | Message rejected at scheme; beneficiary institution not reachable; message type not permitted |
| 6. Counterparty receipt and application | The receiving institution receives and processes the message | That **they** accept what you send, apply it, and are operationally aware of it — the hop with no sandbox equivalent | Received but unprocessable, unmatched, or quietly parked |
| 7. Credit to the beneficiary | Value lands in the specific beneficiary account | That the value path completes to the account, not merely to the institution | Returned, or credited to the wrong place |
| 8. Accounting and booking at both ends | Entries in the sender's and receiver's books | That the entry is *booked the way your design says it is booked* (§7.1) — the hop that most teams discover has been designed only on paper | Entry in an unexpected account, in suspense, or missing |
| 9. Reconciliation and statements | The payment is visible to reconciliation and on statements | That the payment **appears where reconciliation looks** (§7.2) | Reconciled as an unexplained break; or worse, invisibly absorbed |
| 10. Cut-off and cycle behaviour | Timing against the real cycle | That the timing assumptions hold — the actual cut-off, the actual value date, the actual cycle on a real business day | Missed cycle; value date different from expectation |
| 11. Reversal / unwind path | Whatever the agreed recovery route is | That the recovery route **works** — including that the counterparty has the ability and the mandate to respond | No agreed route; a request that nobody actions |

**The two hops that a sandbox can genuinely cover are 1 and 3, and partially 2. Hops 4–11 all require the production path.** That is the arithmetic of the sandbox-versus-network argument, expressed as a table a project can act on.

### 6.3 What a Nominal but Real Value Buys That a Zero-Value File Does Not

The mechanism this guide calls a *prenote* or a *zero-value file* — advance notice of an entry without value — is genuinely useful, and where a scheme supports it, it is **strictly cheaper than a credit**. But it is not the same test, and the difference is worth stating precisely, because the two are frequently conflated in project plans.

| Property | **Zero-value file / prenote-style notice** | **Nominal but real value (the penny test)** |
| --- | --- | --- |
| Account/element exists and is well-formed | ✅ establishes it | ✅ establishes it |
| Receiver's structural validation of your data | ✅ establishes it (this is precisely the mechanism's purpose: let the receiving side reject bad data before value moves) | ✅ establishes it |
| The **value** path: funding, settlement, booking | ❌ does not exercise it — there is no value | ✅ establishes it — and this is the only mechanism that does |
| A **real entry in two institutions' books** | ❌ no entry | ✅ a real entry (§7.1) |
| Statement line and reconciliation visibility | ❌ nothing to reconcile | ✅ a line item that must be matched, and a test of whether reconciliation notices |
| The counterparty's operational response to an actual credit | ❌ absent | ✅ present |
| Reversal/unwind behaviour | ❌ nothing to unwind | ✅ exercises the recovery route — including, on an irreversible rail, the *absence* of one (§7.3) |
| Cost | lower | higher (a real payment, real fees, real liquidity) |
| Available everywhere | ⚠ scheme- and product-specific; the zero-value/micro-entry concept is documented in ACH practice but the specific rule text was not retrievable for this guide this pass — see §16 | ✅ wherever a payment can be sent |

**The rule that follows is short: a zero-value notice tests the *paperwork*; a nominal-value payment tests the *payment*.** If the question the programme needs answered is "will the receiving institution accept our file format", a prenote answers it cheaply and should be used. If the question is "will money move, book, appear on a statement and, if necessary, come back", only a real payment answers it — which is why Sense 3 is a *payment* and not a *file*. And the corollary is the discipline of §7: once you accept that the probe must carry value, you accept that it carries value **into someone else's books**, and you accept the responsibility that follows.

### 6.4 The Pre-Conditions for a Safe Live Test

A live test is safe when its blast radius is *bounded in advance*. The pre-conditions below are the checklist; each item exists because of a specific failure described elsewhere in this guide. A programme that cannot tick all of them has not earned a production probe — and the honest alternative is to test in a certification or simulated environment and accept the residual uncertainty (§9).

1. **The counterparty, engaged properly.** A named contact with an agreed window, so the counterparty knows the test is coming — which day, from which test account, with which reference — **and their own change/ops process engaged, not just the technical contact**, because the receiving institution has to book it and someone there has to recognise it. RTP's rule turns on expectation ✅ (§5.2); an unexpected credit is a payment-processing exception for them.
2. **The value, chosen deliberately.** The smallest value that still forces a real settlement entry — the floor of the path, not a round number (§2.3) — and never a value that requires an approval path the real product will not use.
3. **The unwind plan, agreed in writing before the test.** Who asks, how, in what form, by when, and what happens if the answer is no — plus the reconciliation and suspense wiring in place *before* the probe runs (§7.2), the monitoring annotation with its expiry and mandate (§5.5), and the pre-documented expectation (§7.4) covering the expected entries at both ends, timestamps, statement description and monitoring behaviour.
4. **The stop condition and the evidence capture, both defined in advance.** A criterion for halting the sequence (an unexpected rejection, an alert firing unexpectedly, the counterparty's operational process escalating) with an owner empowered to halt it; the evidence capture specified so the evidence exists rather than being reconstructed (§7.5); and **environment discipline** — test credentials and accounts where the product supports them, production credentials only where the credential path *is* the test, and the two never confused, because the most expensive live-test incidents are the ones where the operator believed they were in the sandbox.

---

## 7. Sense 3 Continued — the Discipline a Live Test Demands

> **This is the section a bank project team should be able to hand to an auditor.** It is written to be read as a control description: each sub-section states the discipline, the reason, and what the evidence looks like.

### 7.1 The Accounting Consequence — a Real Entry in Two Institutions' Books

**A live test payment is not a message; it is value, and value is booked.** This is the difference that separates Sense 3 from every other kind of testing a delivery team does, and it is the difference a delivery team is least equipped to handle, because a test that has "passed" feels finished while the accounting consequence remains.

The entry has two sides, and they are not symmetrical:

| Position | The sender (the bank running the test) | The receiving institution (the counterparty) |
| --- | --- | --- |
| What it is | A **deliberate, authorised** debit: the test account's balance drops by the nominal value plus any fee | An **unexpected** credit into a named account, or into a temporary/suspense position if it cannot be allocated |
| Who owns the decision | The bank's change programme, with a named owner, under the test mandate | Nobody — until someone notices. **This is the operational unfairness of Sense 3 and it must be planned for, not discovered** |
| How it is booked | To the account/product the design says, or to a test/suspense account if the design says so — **either way, booked deliberately and stated in advance** | Per the counterparty's own rules; the receiving institution must be able to identify it as expected (§7.4) |
| How it is reversed | Per the agreed plan: a return, a recall request, a compensating entry, or nothing (if the rail is irrevocable) — see §7.3 | Per the same plan, executed by the counterparty |
| Audit trail | Change record, test mandate, authorisation, payment reference, accounting entries, reconciliation evidence, closure evidence | Their own record; the bank's evidence pack should include the counterparty's confirmation |

**The three questions a project must be able to answer in advance, in writing:** **(a) *Who owns the entry?*** — which legal entity, which account, which cost centre, and whose balance sheet carries the exposure until it is unwound. ***(b) How is it reconciled?*** — which reconciliation, at which frequency, with which matching key, and what the exception path is. ***(c) How is it reversed, and if it cannot be, who approved that?*** — because §7.3's answer is sometimes "it cannot", and an unapproved irrecoverable test payment is a governance failure regardless of how small it is.

**On the "it's only a cent, book it to expense" position:** the argument that the amount is immaterial is true and irrelevant. The entry's purpose is not to be material — it is to prove that the **booking path works**, which is one of the eleven hops in §6.2. A test payment written off to an expense account before reconciliation has *deliberately skipped* the hop it was supposed to test. **The amount being trivial is what makes the test cheap; it is not a reason to take the test less seriously, and the discipline is the same in kind as for a €100,000 probe.**

### 7.2 The Reconciliation Visibility Rule

> **The rule: a test payment that is invisible to the reconciliation and suspense processes is worse than no test at all, because it trains the team to ignore an exception.**

The reasoning is not a preference for tidiness. It is about what a control learns. A reconciliation function exists to notice that something is in the books that should not be, or is not in the books when it should be. **Every unrecognised item that passes through it without being raised is a training example that the function is not to be believed.** A test payment that (a) moves real value, (b) is not expected by reconciliation, and (c) is silently matched, absorbed, written off or simply never noticed delivers this outcome:

- the anomaly was present, and the control did not fire;
- if the control *did* fire, someone suppressed it without a record;
- the next time a genuinely unexplained small credit appears — a customer's own Sense 1 probe, a mistake, or something worse — the prior experience is that small unexplained items are noise.

**Three consequences, and each is a requirement:**

1. **The test payment must be *expected* by the reconciliation process** — wired in as a known item with a matching key and an owner, so that reconciliation can *find* it. Discovering that reconciliation finds it is part of what the test establishes (§6.2, hop 9).
2. **The item must reach suspense where suspense is the correct destination.** Where the receiving side cannot allocate the credit, the correct behaviour is a suspense entry — a visible, owned, ageing exception — and not a write-off. **A suspense item is the system working; a silent absorption is the system failing.**
3. **The exception must be closed explicitly, with evidence, as part of the test's closure.** An exception closed by timeout, by threshold, or by an operator's judgement that "it's the test payment" is not a closure — it is a suppression, and §5.5's reasoning applies identically.

**And the same rule applies to monitoring** (§5.4–5.5): a test payment invisible to monitoring, or a suppression without expiry, is worse than no suppression, for exactly the same reason.

### 7.3 The Irreversibility Problem on Instant Rails

This is the hardest constraint in the guide, and it changes what can safely be used as a probe.

**On a batch or file-based rail, the probe is reversible in the ordinary way.** Entries can be returned, recalls can be honoured, and the unwind is an operational routine with a known clock. The probe's maximum loss is bounded by the recovery discipline, and a wrongly-booked test payment is an accounting problem, not a capital problem.

**On an instant rail, the credit is final, and "final" is the operators' own word for it.** The Clearing House's public description of the RTP network lists settlement as **"Instant Settlement: Final, anytime, every day"** ✅ (The Clearing House, retrieved 2026-09-29 — the operator's own description). That is the property that makes instant payments instant, and it is also the property that makes them unsuitable for a test whose recovery is assumed. The recovery route that *does* exist is **request-shaped, not reversal-shaped**: TCH publishes **"RTP® Network Request for Return of Funds Guidelines and Suggested Practices"** ✅, and its rule-interpretation set includes **"Returning Funds After Providing an Accept Response to a Credit Transfer"** (effective 25 July 2024) ✅ — both retrieved from the RTP Document Library on 2026-09-29. **A request is a request: it requires the other side to act, and it can be declined.** On the FedNow side, the public resources place transfers under **Operating Circular 8** and **subpart C of Regulation J** ✅ (Federal Reserve Financial Services, retrieved 2026-09-29), i.e. under a legal regime rather than under a scheme return convention — again, finality with a defined framework rather than a reversal button.

**What follows is a design rule, not a caution:**

| Rail class | Probe choice |
| --- | --- |
| Batch/file-based credit transfer (returnable, recalls customary) | A real nominal-value payment is the default and correct probe |
| Instant account-to-account (final settlement, recovery by request) | **Choose the value as if you will not get it back**, and prefer the *smallest legitimate value* or, where the scheme supports it, a certification-environment test for the unwind path. **Do not run a probe whose recovery is a prerequisite for the test being acceptable** |
| High-value RTGS-class | A nominal-value real probe where a test/round-trip facility exists; otherwise certification, because the credit's finality and the amounts involved mean irrecoverability is not a rounding error |
| Correspondent/cross-border messaging | Message-level tests are the *default*, not the fallback; a value-bearing test is chosen deliberately and cleared with the correspondent relationship owner, because a stray credit in a correspondent flow creates a broken-value investigation on someone else's books |

**The cross-rail principle, stated once, to be reused in §8: a probe that cannot be reversed must be chosen differently from one that can.** Two consequences: first, on an irreversible rail the *minimum* is constrained by prudence rather than by the path's floor; second, some tests — specifically, tests of the *unwind* path itself — **must be run where the unwind can be observed safely**, which is a certification environment, and this is exactly the reasoning the worked example in §14 uses for the one test Cymbal declines to run in production.

### 7.4 The Pre-Documented Expectation

**The discipline: state, in advance and in writing, exactly what the test is expected to produce — so that a failure is *detectable* rather than *absorbed*.**

Why this is a control and not project-management theatre: a test with no written expectation cannot fail, because nothing is compared. In Sense 3 the comparison points are numerous and each one is a hop from §6.2. The pre-documented expectation should therefore specify:

- **The expected accounting entries at both ends** — accounts, amounts, value dates, currencies, and which entity's books (§7.1).
- **The expected timestamps** — send time, scheme acceptance, credit, and the actual cut-off for that cycle, so that "it arrived late because of the cut-off" is caught as a *result* rather than accepted as an *explanation*.
- **The expected statement/advice lines at both ends**, including the description text the counterparty will see — which is also the mechanism by which the counterparty's operations team recognises the payment (and which, on RTP, must **not** be smuggled into the Debtor name field ✅).
- **The expected reconciliation outcome** — which reconciliation will match it, on which key, in which cycle, and the expected suspense behaviour if it fails to match (§7.2).
- **The expected monitoring behaviour** — that it is suppressed by mandate *or* fires and is investigated, and which of the two is intended (§5.5).
- **The expected reversal/unwind behaviour and timing**, or the explicit statement that no unwind is attempted and why (§7.3).
- **The expected *negative* results too** — what should *not* happen: no unexpected fees, no unexpected interim status, no message the counterparty does not process.

**The test is then a diff between the written expectation and the observed reality, and every non-empty diff is a finding.** A test payment whose outcome is described afterwards, from what happened, has produced a narrative rather than evidence.

### 7.5 Containment, Reversal and the Evidence Retained for Audit

**Containment.** The containment plan is what makes the test *bounded*: test accounts and test references where the product supports them; a named owner; a stop condition; a defined counterparty window; and the reversal plan agreed in advance (§6.4). Where a test payment cannot be fully contained — any test on an irreversible rail — that fact should be **an explicit, approved exception in the mandate**, not an unexamined design feature. Containment also has an outward face: **the counterparty must be able to contain it on their side**, which is what their operational awareness buys.

**Reversal.** The reversal path must be *tested*, not merely drafted. It appears in §6.2's hop list for the reason that a reversal plan which has never been executed is an assurance someone wrote, not a control. Where a rail's reversal is a *request* (as the RTP material's "Request for Return of Funds" framing indicates ✅), the plan must name who makes the request, in what form, to whom, and what happens when the answer is no.

**The evidence pack.** Retained for audit, and by the guide's own standard enough to let a reviewer reconstruct the test without asking anyone:

| Evidence | What it establishes |
| --- | --- |
| The **test mandate** — the change/CR reference, the test's purpose, owner, window, scope of rails/counterparties, and the *approved* irrecoverability exceptions | That the test was authorised, and by whom; the anchor for §5.5's mandate-binding |
| The **pre-documented expectation** (§7.4), dated **before** the test ran | That the test could fail, and that the comparison was defined in advance |
| **Counterparty agreement** — their confirmation of the window and the expected credit, and their contact | That the counterparty was not surprised; the §5.2 relationship/expectation property, evidenced |
| **The payment record** — instruction, reference, message, and the scheme/counterparty acknowledgements | That value moved, when, from whom to whom |
| **Accounting evidence** — entries on both sides, the reconciliation match or the suspense item, and the exception closure | §7.1 and §7.2 compliance, evidenced rather than asserted |
| **Monitoring evidence** — the annotation/suppression record with its **expiry**, the events it suppressed (retained, not dropped), and the **proof of removal/expiry** | §5.5's guardrails, evidenced: the control existed, was bounded, and was switched off |
| **The reversal/unwind record**, or the approved exception | §7.3 compliance |
| **The closure record** — the diff between expectation and reality, every deviation raised as a finding, and the disposition of each finding | That the test produced a result rather than a narrative |

**The standard, in one sentence:** *if a reviewer cannot tell from the pack what was expected, what happened, what it cost the books at both ends, and that the monitoring suppression is gone, the pack is not evidence of a controlled test.*

---

## 8. The Test Protocol by Rail Class

The practice is the same everywhere — *put the smallest real value through the real path and compare the result with a written expectation* — but **what can safely be probed changes with the rail's reversibility, its message model and its cut-off structure.** This section applies §7's cross-rail principle rail class by rail class. The rails themselves are owned by [payment_rails_guide.md](payment_rails_guide.md); what follows is only the test protocol.

### 8.1 Batch and File-Based Credit Transfer

**The rail family:** deferred-net-settlement, file- or batch-oriented credit transfer — the ACH model, Bacs-style batch credit, and the file-transfer paths used to move instruction files between institutions (see [swiftnet_fileact_guide.md](swiftnet_fileact_guide.md) for the file-transfer mechanics).

**What the rail's own public documentation establishes about its structure** (Nacha, retrieved 2026-09-29): the ACH Network "is open for processing payments **23¼ hours every business day** and **settles payments four times a day**", with settlement occurring when the Federal Reserve's settlement service is open (closed on federal holidays and weekends, and on business days from **6:30 p.m. ET to 7:30 a.m. ET**) ✅. Same Day ACH went live in **2016**, and the network's publicly stated per-payment limit is **$1 million**, rising to **$10 million** under a rule amendment whose effective date is **17 September 2027** ✅ (nacha.org).

**The protocol.** This is the rail class where the full protocol is cheapest and most complete:

1. **A zero-value/notice-level step first**, where the scheme supports it — establishing that the receiving institution can accept and validate your data before value moves (§6.3). Where the scheme does not support a value-free notice, skip to step 2 and accept that the first probe carries value.
2. **A real nominal-value credit** to the designated test account, sent through the production path, at the *smallest legitimate value*.
3. **A return/reversal exercise** — because on this rail class the unwind is an ordinary operation with a defined window, and the only way to know it works is to run it. **Do not skip this because the forward path worked**; the return is the control that makes the next probe safe.
4. **Reconciliation and suspense verification** (§7.2) — the item must be found by reconciliation, and where it cannot be allocated, it must be *seen* in suspense.
5. **A cut-off-boundary probe** where the product's cut-off matters, run at the boundary the business actually depends on.

**The specific discipline for this rail class:** the batch rail's cycle structure makes *timing* the most common false conclusion. "The payment arrived the next day" is not a result until it is compared with the written expectation for that cycle (§7.4). And the 23¼-hour/settlement-window structure means a test run at the wrong hour tests the *calendar*, not the channel — which is fine, provided the expectation says so in advance.

### 8.2 Instant Account-to-Account and Real-Time Schemes

**The rail family:** the RTP network and FedNow in the US, SEPA Instant in the euro area, Faster Payments in the UK, FAST/PayNow in Singapore, UPI and the instant schemes in India, PIX in Brazil, NPP in Australia — the global instant class mapped in [payment_rails_guide.md](payment_rails_guide.md), with the Singapore-specific layer in [singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md).

**What is established at source for the US instant rails** (The Clearing House's own materials, retrieved 2026-09-29):

- **Finality.** Settlement is "**Instant Settlement: Final, anytime, every day**" ✅, and the network operates continuously — "available around the clock, including bank holidays, weekends and after hours" ✅.
- **Value model and limits.** RTP is described as high-limit — "send up to **$10 million per transaction**" ✅ — and settles on a **prefunded, real-time, gross** basis, with prefunding held in a special deposit account at the FRBNY jointly owned by Funding Participants and Funding Agents (the "Prefunded Balance Account"), each payment settled in real time by decreasing the Sending Participant's and increasing the Receiving Participant's Net Position ✅ (RTP Network Readiness Checklist, v1.0, September 2020).
- **Messaging.** TCH "has adopted the **ISO 20022 standard** for RTP network messaging", with foundational messages being the **Credit Transfer, Message Status Report (Confirmation), Request for Payment, Request for Information, Remittance/Invoice Detail, Request for Return of Funds**, plus system administrative messages ✅ (RTP Readiness Checklist; the same guide's message-specification list adds the published **RTP Message Specifications (Version 5.0)** ✅, retrieved from the RTP Document Library).
- **The receiving side's obligations, which are what a live test is really probing.** A Receiving Participant "must **immediately** respond to a Payment Message (with an '**Accept**,' '**Reject**' or '**Accept without Posting**' message)"; for all accepted payments the Receiving Participant "must provide **immediate funds availability**"; and participants "must immediately make available information regarding the status of an RTP Payment to their customers" ✅. **The "Accept without Posting" option deserves attention in a test design**: an accepted message does not automatically mean a posted customer credit, so the test's expectation must state which of the three responses is expected and what the customer's balance should show.
- **Two prohibitions that bear directly on this guide.** The readiness checklist's list of RTP Rule prohibitions includes "**no foreign payments, no searching for accounts**" ✅ — corroborating the Compliance Bulletin's "No Searching for Accounts" rule (§5.2, §5.5). **This is the scheme telling you, in a readiness document aimed at new participants, that using the rail as an account-discovery device is not permitted** — which is precisely the dual-use boundary that Sense 1 and Sense 3 sit either side of.
- **FedNow.** Public material describes it as an instant payment infrastructure allowing eligible depository institutions to "send and receive instant payments in real time, around the clock, every day of the year", with recipients having "full access to funds immediately" ✅, with **ISO 20022** as the messaging standard ✅ and transfers governed by **Operating Circular 8** and **subpart C of Regulation J** ✅ (Federal Reserve Financial Services, retrieved 2026-09-29).

**The protocol for this rail class**, and it is *not* the batch protocol:

1. **The value is chosen as if it will not come back** (§7.3). On this rail class the credit is final, and the recovery route is a **Request for Return of Funds** — a request message, not a reversal ✅ — whose outcome depends on the other participant acting. So: value = smallest legitimate, and **no probe whose acceptability depends on its recovery.**
2. **The receiver-response behaviour is part of the expectation.** Test that the counterparty returns the correct one of Accept / Reject / Accept without Posting, and that what they return matches the customer-facing effect you designed.
3. **Continuity is tested at an awkward hour on purpose** (§6.1) — because the *claim* of continuous operation ✅ is a claim, and the business case usually depends on it.
4. **The unwind path is exercised separately or explicitly waived.** Where the rail cannot be unwound, the honest options are (a) test the unwind in a certification environment, or (b) record an approved irrecoverability exception in the mandate (§7.5). **The one thing not to do is to plan a production probe and assume the return.**
5. **Reconciliation and suspense verification** in the same terms as §8.1, with the added note that on a final-settlement rail a misposted credit cannot be quietly repaired — which raises the importance of getting the booking design right *before* the probe.

### 8.3 The Correspondent and Cross-Border Messaging Path

**The rail family:** correspondent banking and cross-border messaging — the SWIFT message estate (MT/MX) and the file-transfer and alliance-access layers, whose mechanics are owned by [swift_alliance_access_guide.md](swift_alliance_access_guide.md), [swiftnet_fileact_guide.md](swiftnet_fileact_guide.md) and [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md).

**The protocol for this rail class is message-first, and deliberately so.** The reason is containment: a value-bearing test in a correspondent path does not just create an entry in two banks' books — it can create a broken-value investigation, a claim, and an enquiry across a correspondence relationship that exists to be reliable. **A stray credit in a correspondent flow is professionally expensive in a way that a stray credit between two retail accounts is not.** Therefore:

1. **Message-level tests are the default** — a message the counterparty has agreed to receive and acknowledge, exercising the same schema, the same credentials and the same connectivity, without moving value.
2. **A value-bearing test is a deliberate, separately approved step**, cleared with the correspondent relationship owner and the counterparty's operations function, at a value chosen so that irrecoverability is immaterial *and* pre-agreed as such.
3. **The correspondent's own conventions govern.** What is a test in your institution may be a live instruction in theirs; the agreement must be explicit about which.
4. **Free-format field content is the real risk.** The counterparty's processing depends on the fields you populate, and the practical failure mode of a live test is not rejection but *silent misapplication* — a payment applied to an account on a narrative-matching basis when it should have been applied by reference. **On the RTP rail the scheme solved this by forbidding verification codes in the Debtor name field ✅; the cross-border equivalent is a relationship-level agreement on what belongs in the free-text fields, and the test is the moment to exercise it.**

**Flagged, honestly:** this pass could not retrieve SWIFT's own public documentation — **swift.com failed to return content on every URL attempted** (see §16), and the ISO 20022 catalogue site likewise. Therefore **no specific SWIFT message type number, test-message convention, or CSP-attestation requirement is asserted in this guide.** The message types and conventions for this path should be taken from the counterparty's documentation and the applicable SWIFT materials, not from this guide. ⚠

### 8.4 Domestic Low-Value and High-Value Clearing Systems

**The rail family:** the domestic clearing landscape — for the markets this repository covers, that includes CNAPS ([cnaps_guide.md](cnaps_guide.md)), the NUCC/NetsUnion clearing layer ([nucc_netsunion_guide.md](nucc_netsunion_guide.md)), the Hong Kong and mainland cross-boundary corridor ([ibps_payment_connect_guide.md](ibps_payment_connect_guide.md)), Singapore's FAST/PayNow and the NETS/BCS local layer ([singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md), and [nets_singapore_guide.md](nets_singapore_guide.md) for the local scheme and clearing-house operator), and the hub architectures that sit above them all ([payments_hub_guide.md](payments_hub_guide.md)).

**The protocol difference between low-value and high-value is one of proportion, not kind:**

| | **Domestic low-value** | **Domestic/RTGS high-value** |
| --- | --- | --- |
| Probe viability | A real nominal-value payment is usually the right probe | A real probe may be inappropriate because of the amounts' finality and the value of the payment's *implied* message; **prefer a scheme-provided test/round-trip facility, or certification**, and treat any production probe as a separately approved exception |
| The dominant test target | The *product*: limits, cut-off, proxy resolution, beneficiary name handling, reconciliation, customer-facing status | The *plumbing between institutions*: message validity, credentials, routing, cut-off, and the counterparty's acceptance of the message |
| Limit testing | Must be tested at the boundary, not at the middle | Same, plus the value-date and liquidity implications |
| Reversal | Routine, within a return window | Not routine; defined by the scheme's procedures and, in practice, by the relationship |
| Reconciliation visibility | The test item must be found, and suspense behaviour verified | Identical, with higher materiality |

**A repo datapoint worth citing here, because it makes the boundary-testing rule concrete.** [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 471 already lists, inside its rule-engine test matrix, **"each limit boundary (daily and annual, at limit and one cent over)"** ✅. That is the same "one cent" instinct this guide's §2.3 describes — and it is the correct companion to the penny test. **The penny test proves the channel; the boundary test proves the limit; the boundary test must be run *at* the boundary and one unit over it, which no single small probe can do.** A test protocol that runs only a penny test has tested connectivity and left every limit assumption untested.

### 8.5 The Cross-Rail Principle

> **A probe that cannot be reversed must be chosen differently from one that can.**

Stated once, and applied above: **reversibility determines the value, the venue and the approval level of the probe.**

- **Reversible rail** (batch/file-based credit transfer with an ordinary return window): a real nominal-value payment is the default probe, and the unwind is part of the test.
- **Irreversible rail** (instant credit, final settlement, recovery only by request): the value is chosen as if it will not return, the unwind path is tested elsewhere or explicitly waived, and the probe carries an approved exception if it is irrecoverable.
- **Message-oriented rail** (correspondent/cross-border): the default probe is a message, and a value-bearing test is a separately approved act, because the counterparty's operational cost is part of your cost.
- **Certification-gated rail** (see §9): some environments provide the full message and connectivity path *without* final value, and where they exist they should be used **before** the production probe rather than instead of it — the certification environment is where you test the unwind; production is where you test that the money actually moves.

---

## 9. The Certification-Versus-Production Question

### 9.1 Where a Scheme Requires Certification or a Sandbox Instead of a Live Probe

Schemes do not all allow a live production probe, and some do not allow it as the *primary* evidence. What is established at source in this pass:

| Scheme/environment | What is publicly established about certification or gated access | Status |
| --- | --- | --- |
| **RTP (US)** | Participants select a **"Message Persona"** with mandatory and optional RTP messages, and "**your FI will determine which Persona it will implement and seek certification for** during your onboarding process" ✅. TCH also states that it "**performs technical testing with TPSPs** to confirm they are capable of processing messages in accordance with the RTP technical specifications" ✅ — i.e. testing is done against the access-provider layer as well as the participant. Separately, all RTP Participants must **audit their compliance at least once each calendar year** against the RTP Participation Rules and Operating Rules, **as required by the RTP Operating Rules section IX.A.2**, complete and retain the **RTP Self-Audit Form** as evidence, and report **material findings of non-compliance to the participant's audit committee or equivalent body** ✅ | ✅ The Clearing House, RTP Readiness Checklist (v1.0, September 2020) and RTP Participant Self-Audit page, retrieved 2026-09-29 |
| **FedNow (US)** | The technical and service information resource for participants — **FedNow DevRel** — is **credential-gated**: "available to FedNow Service participants and service providers to access service and technical information using their **FedLine Solutions credentials**", provisioned through the participant's own access administration, and "use of FedNow DevRel is for **credentialed-only access**" ✅. Transfers are governed by **Operating Circular 8** and **subpart C of Regulation J** ✅ | ✅ Federal Reserve Financial Services, retrieved 2026-09-29 |
| **UK CoP / SEPA VOP (name-checking overlay services)** | Both are **participation-based overlays**: Pay.UK describes CoP as an API-based peer-to-peer service with a **directory** of participating organisations and an **onboarding process** with direct and aggregator access models (including a published list of aggregators who have completed Pay.UK's onboarding requirements) ✅; the EPC describes VOP as **inter-PSP rules, practices and standards** with a rulebook, an API specification and an API security framework ✅. **Neither moves value, so neither has a "test payment" — their testing question is conformance to a scheme's own specification** | ✅ Pay.UK and EPC, retrieved 2026-09-29 |
| **Card schemes** | ⚠ **Not verified in this pass.** Public card-scheme documentation on certification, sandbox and test-card conventions was not retrieved; the only card-side material obtained is the processor-level card-testing documentation cited in §4. **No card-scheme certification requirement is asserted in this guide** | ⚠ flagged |
| **SWIFT** | ⚠ **Not verified in this pass** — swift.com did not return content on any URL attempted. No SWIFT certification, test-message or attestation requirement is asserted here | ⚠ flagged |
| **ISO 20022 catalogue** | ⚠ **Not verified in this pass** — iso20022.org did not return content. Message-definition claims in this guide are therefore limited to those published inside a scheme's own material (e.g. the **pacs.008 "Debtor name" field, index 2.855**, cited by RTP's Compliance Bulletin ✅) | ⚠ flagged |

**The pattern worth noting:** across the sources actually obtained, **the gates are real and they are of three kinds** — *technical certification* (RTP message personas; TCH's testing of TPSPs), *credential-gated documentation* (FedNow DevRel), and *participation/onboarding with a published directory* (CoP, VOP). Each kind produces a different answer to "can I run a live probe before go-live, and is that probe evidence?"

### 9.2 What a Certification Test Actually Establishes

A certification test establishes **conformance**: that what you send conforms to the scheme's specification for the message set it covers, and that the scheme's or operator's process can accept and route it. Read the RTP formulation closely — a participant "will determine which Persona it will implement and **seek certification for**" ✅ — and the scope is clear: **certification is scoped to a persona, i.e. to a message set.** It is evidence about *conformance to a standard*, for *a defined set of messages*.

What follows, and each point is a limit rather than a criticism:

1. **Certification is about the standard, not about your counterparty.** Conformance to a scheme's message specification says nothing about whether a *specific* receiving institution's application maps your fields the way you expect, or whether that institution's operations team will recognise your payment (§6.2, hop 6).
2. **Certification is about your message set, not about your product.** A persona covering credit transfers says nothing about your payout product's limit logic, your reconciliation design, your suspense handling, or your customer-facing status display.
3. **Certification is (usually) about a non-final or simulated path, so it is silent on value.** The certification exercise can validate the message and the routing logic; it cannot validate that *your* production funding, booking, settlement and statement path behaves as designed (§6.3).
4. **Certification is point-in-time, and the scheme itself is versioned.** RTP's Operating Rules and Participation Rules carry published effective dates and a Compendium of rule-change summaries ✅; Nacha's rule calendar is dated by effective date ✅; the EPC publishes VOP rulebook versions with effective dates ✅. A certification pass on one version is evidence about that version.
5. **Certification does not commit your counterparty to anything.** The RTP readiness checklist's own account of the live flow — the Receiving Participant confirms, accepts and makes funds available immediately, **or rejects for certain reasons specified in the RTP Rules** ✅ — is the reminder that at the moment of production, the answer is theirs to give.

### 9.3 The Honest Distinction — Scheme Evidence Versus Production Readiness

| Question | Who answers it | With what | What it costs to be wrong |
| --- | --- | --- | --- |
| Does our message conform to the scheme specification for the messages we need? | The scheme/operator (certification, message-persona testing) and the access provider (TCH's technical testing of TPSPs ✅) | Certification evidence | Rejection at the scheme; a step back to remediation |
| Does the counterparty accept and process what we send? | The counterparty, operationally, in production | A live probe, or the counterparty's own confirmation — **and only the live probe is evidence about the live path** | Received but misapplied or parked; a payment you believe happened and did not |
| Are our credentials, connectivity and sender identity live and accepted? | The world — the channel, the counterparty, the scheme | A live probe; no certification substitutes | A go-live that fails at the first real payment |
| Is the cut-off, the cycle and the 24/7 claim what we assumed? | The operator's operating calendar, tested in production time | A live probe at the boundary | Value-date surprises; missed cycles; a customer-facing promise broken on day one |
| Does value **book, reconcile and appear** as designed? | Our own accounting and reconciliation functions, exercising the real path | A live probe that is expected by reconciliation and suspense (§7.2) | An unbooked or invisibly absorbed item; an audit finding; a control that has learned to ignore exceptions |
| Are we compliant with the scheme's ongoing obligations? | Us, and our audit function | Our own annual self-audit and retained evidence (as the RTP rules require: audit at least annually, retain the Self-Audit Form, report material findings to the audit committee ✅) | Regulatory/scheme findings; escalating to a compliance matter |

**The distinction, in one line: certification is evidence that you built to the specification; a live test payment is evidence that the money moved.** A programme that presents certification as production readiness has answered a *conformance* question and reported it as an *operational* one — and it is exactly the same substitution error as treating a sandbox pass as a network proof (§6.1), one level up the governance chain.

---

## 10. The Accounting, Reconciliation and Audit Angle

### 10.1 The Entry, the Suspense Account and the Reversal

§7.1 established that a live test payment creates two real entries. This section states the accounting mechanics at the level a control description needs, and cross-references the repo's own posting material ([posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md)) rather than re-deriving it.

**The entry.** The sending side books a debit for the nominal value plus any fee, against whatever account the design specifies. **The design question — whether that is the normal customer account, an internal test account, or a designated suspense/test account — should be decided before the probe, not after.** Each choice has a consequence: a normal customer account exercises the customer-facing path (good) but creates a customer-visible artefact (may be undesirable); an internal test account is quieter but exercises a *different* booking path from the one production will use, which weakens the test (§2.3's "the path is value-independent" assumption).

**The suspense account.** On the receiving side — and on the sending side whenever the item cannot be allocated — the correct destination for value whose allocation is not yet determined is **suspense**: a visible, owned, ageing exception with a defined resolution route. The control-relevant properties are that the item is *visible*, that it is *owned*, that it *ages*, and that it is *closed with a reason*. This is the mechanical counterpart of §7.2's reconciliation visibility rule, and the one sentence to remember is: **a suspense item is the system working; a silent absorption is the system failing.**

**The reversal.** Three shapes exist, and the choice follows the rail (§8.5):

1. **Return** — the receiving side returns the entry through the scheme's own return mechanism. Cheap, routine, and available on the returnable rail classes.
2. **Recall or request** — the originator asks for the funds back. On the instant rails this is the documented shape: RTP publishes **Request for Return of Funds** guidelines ✅ and a rule interpretation on **returning funds after providing an Accept response** (effective 25 July 2024) ✅. **A request can be declined, which is why §7.3 insists the probe be affordable without it.**
3. **Compensating entry** — the originator books an offsetting entry of its own so that the *net* position and the accounting outcome are as intended, while the original entry remains in the record. This is the honest fallback where the rail cannot return, and it must be **an approved decision, recorded in the mandate**, not an operator's workaround.

**The audit consequence that follows from all three:** the reversal shape must be *known before the probe*, because shape 3 changes what the test costs and who has to approve it.

### 10.2 How the Test Appears to an Auditor

An auditor does not see the test. They see the **record the test left**, and they read it against the controls the institution claims to operate. The most useful way to write this is as the questions an auditor will ask, with the evidence that answers each:

| The auditor's question | The evidence that answers it | Where it is produced |
| --- | --- | --- |
| Was this payment authorised? | The test mandate / change record, the named owner, the approved scope and any irrecoverability exception | §7.5 evidence pack |
| Was it expected by the operational functions it touched? | The counterparty's confirmation; the reconciliation's pre-registration of the item; the monitoring annotation with its mandate and expiry | §6.4, §7.2, §5.5 |
| Was it booked the way the design says? | The accounting entries at both ends, matched to the pre-documented expectation | §7.1, §7.4 |
| Did the controls actually fire? | The reconciliation match or the suspense item and its closure; the monitoring events and their disposition | §7.2, §5.4 |
| Is the exception closed? | The closure record for every deviation between expectation and outcome, with a disposition per finding | §7.4, §7.5 |
| Is the institution's *ongoing* obligation met? | For an RTP participant, the annual self-audit: audit of compliance at least once each calendar year per Operating Rules section IX.A.2, the retained Self-Audit Form, and material findings reported to the audit committee or equivalent body ✅ | ✅ The Clearing House, RTP Participant Self-Audit page, retrieved 2026-09-29 |
| Could this control be abused? | The evidence that the monitoring suppression was scoped to counterparty/purpose (not amount), that it was time-bounded, and that it was **removed** | §5.5 |

**A note on the last row, because it is the one most often missed.** An auditor reviewing a live-test control will look for the *residual* risk the control creates — and the residual risk of "annotate the test traffic" is a standing exemption in the monitoring estate. **The evidence that the exemption is gone is part of the control's evidence, not an administrative footnote.**

### 10.3 An Unrecorded Test Payment Is a Control Failure

> **A test payment that is not recorded, not expected by reconciliation, and not evidenced is a CONTROL FAILURE even when the test itself succeeded.**

The clause "even when the test itself succeeded" is the whole point. The test's technical success is one outcome; the control outcome is whether the institution's *processes around* value movement behaved. Consider what a successful but unrecorded test payment demonstrates:

- **The message path works** (good).
- **The institution can move real value without its reconciliation function noticing** (a finding).
- **The institution can suppress a monitoring alert without a record or an expiry** (a finding).
- **The institution's evidence about the test is unreconstructable** (a finding, because it means the assurance is a narrative).
- **And the institution's staff have now experienced an unexplained small item being normal** — which is the §7.2 training effect, now applied to a real control.

**The asymmetry that makes this worth emphasising: the technical success is measurable, and the control failure is not.** A team will report the first and never see the second. The remedy is structural, not cultural: **the test cannot be closed until the accounting, reconciliation, monitoring and evidence items are closed** — i.e. the test's definition of done includes the §7.5 evidence pack, signed by the owner who holds the mandate, and the accounting items stay open until they are resolved rather than being written off to expense.

**And the same standard applies to the *scheme's* view.** Where a scheme requires an annual self-audit and the reporting of material findings to the audit committee ✅, a bank's internal answer to "what did our live tests touch, and how were they contained" is exactly the kind of question that self-audit exists to surface. **The cost of not having the answer is not the penny — it is the finding.**

---

## 11. Customer Consent and Data Protection for Sense 1

Sense 3's consent question is a change-management question, and Sense 2's consent question is the absence of consent. **Sense 1's consent question is the interesting one**, because a legitimate, customer-requested process nonetheless writes information onto an account and discloses it to a third party in order to prove something. This section states the questions plainly; the applicable legal regimes for a given institution are a matter for that institution's privacy and compliance functions, and the repo's AML/KYC material owns the onboarding angle.

### 11.1 The Consent the Account Holder Gives

Take the mechanism apart and the consent is doing more work than a single "yes" suggests. The customer is authorising, at least:

1. **A credit into an account they nominate** — value appearing in their account, with a description the originator writes. Where the variant is a credit-and-debit or debit-based probe, the authorisation is broader still: the originator is being permitted to *debit*, and in many implementations that permission persists as a payment mandate for later collections. **The verification is frequently the first exercise of a mandate that will be used for real money later, and the consent language usually covers both.**
2. **The use of the account data they supply** — routing/transit number and account number — for the stated purpose. This is data the customer contributes, which is the good case: the customer is the data subject *and* the consenter.
3. **The reporting of the amounts back** — the customer performs the proof, and the originator records the outcome. What is retained here is not merely "verified" but an attempt history: how many attempts, which amounts, when (§11.3).
4. **The disclosure of the account's existence and reachability to the originator.** This is the substantive thing the customer is giving away, and it deserves to be said in the consent rather than assumed: the originator learns that the account exists, accepts credits, and is reachable through the rail used. **That is a small but real fact about the customer's financial life, and it is durable** — a verified bank account on file at a platform is an asset the platform holds, and one it may keep long after the verification.

**The vendor documentation describes the flow, not the consent.** Dwolla's documentation states that bank account verification is required before a customer can *send* funds, and quotes the requirement in the ACH context as **"bank account verification prior to sending funds is required by the ACH network"** ✅ (Dwolla, retrieved 2026-09-29 — the vendor's own description of the requirement). Its public framing of the lower-risk use case is also instructive: adding a bank account via the API without verification leaves the funding source **unverified**, and an unverified funding source is **allowed to receive funds but not to send them** ✅. **That is the consent boundary expressed in product terms: verify when you need to take or send, and not merely because you can.**

### 11.2 What the Act of Verification Discloses

The bookkeeping of disclosure runs in both directions, and the second direction is the one that gets missed:

- **What the originator learns about the account:** that it exists, that it accepts credits of this type, that it is reachable through this rail, and — through the verification outcome — that the person completing the flow can see the account's transactions.
- **What the account's record discloses to anyone who can see it:** a small credit, with a description written by the originator, at a time fixed by the originator's schedule. §3.6 developed this; the point to carry here is that **this disclosure is unavoidable** — it is the mechanism. The only levers are the *amount* (kept secret and unpredictable), the *descriptor* (whose content is the originator's choice and should be chosen to disclose as little as it can while remaining recognisable — and note that on the RTP rail the scheme has forbidden the most informative variant, keeping verification codes out of the Debtor name field ✅), and the *number* of probes (one series, not a pattern).
- **What the absence of disclosure means.** Because the amounts are the secret (§3.3), a legitimate verification *cannot* be explained on the statement. **The stronger the control, the more opaque the disclosure** — an unavoidable trade in this mechanism, and one that sits badly with a data-minimisation instinct that would prefer the statement to say "account verification for <service>, amounts withheld on request".

### 11.3 The Retention Question

Three different things are retained, and they should have three different lives:

| What is retained | Why it exists | The retention principle |
| --- | --- | --- |
| **The two amounts** | To verify the customer's report. They are the secret, and they must be held in order to check the answer | **The shortest life of the three.** Once verified — or once the attempt limit is exhausted — the amounts have no function; retaining them extends only the window in which a disclosure matters. Dwolla's documented attempt limit and its documented 48-hour recovery path both show the shape: the amounts are valid for a bounded episode, then abandoned ✅ |
| **The verification outcome and its evidence** | To prove the account was verified, for dispute, audit and possibly regulatory purposes | **Retained per the institution's evidence policy** — the *result* is the durable fact, not the amounts |
| **The account data (routing/account number) and the mandate** | To move money later | **Governed by the payment relationship**, and here vendor practice is instructive: Plaid's documentation describes **consent/TAN expiration** for account-and-routing-number data obtained through its Auth flows, with defined remediation paths and the note that using the numbers without completing a refresh **may result in ACH returns** ✅ (Plaid Auth documentation, retrieved 2026-09-29) — i.e. the data's validity is itself time-bounded, and the vendor documents that fact. **That is the correct instinct for any stored verified account: the verification is a fact at a point in time, not a permanent statement about the account.** |

**The question to ask of any implementation:** *if the account were closed yesterday, how would we know, and who would tell us?* A verification performed eighteen months ago is evidence about eighteen months ago; an institution that treats it as current assurance has quietly converted a control into an assumption.

### 11.4 Verifying an Account the Customer Owns Versus Profiling One They Do Not

**This is the line that matters most in Sense 1, and the schemes have drawn it themselves.**

| | **Verifying an account the customer owns** | **Probing an account the customer does not own** |
| --- | --- | --- |
| Whose data is being established | The customer's own account, with the customer's participation | A third party's account, with no participation by that party |
| Who is the data subject | The customer, who is also the requester | A person who never spoke to the requester |
| What the third party experiences | They complete a flow they initiated | An unexplained credit appears on their account, and their statement discloses the requester's existence to whoever looks |
| The scheme's own view | **Permitted where the relationship exists** — the RTP rule permits the use of payment messages to determine whether an account number is associated with a valid, active account **when the account number was given to the Sender by an intended Receiver who is expecting to receive one or more payments from that Sender** ✅, and its own Compliance Guide example is the broker's **small test RTP payment** to a consumer who is expecting an RTP payment from that broker ✅ | **Prohibited.** "RTP Participants are **not permitted** to send RTP Payments to verify the account and routing information of accountholders **who do not expect to receive** one or more RTP Payments, ACH credits, or other credit payments from the Sender" ✅. The RTP readiness checklist states the same prohibition in its list of ongoing obligations as "**no searching for accounts**" ✅ |
| The mechanism | The mechanism is the same | The mechanism is the same |
| What distinguishes them | **Relationship and expectation — not the amount** | — |

**The RTP rule is the cleanest public statement of the principle available in this pass, and its usefulness is that it is stated as a rule, not a principle.** An institution wanting a test for its own conduct can adopt the rule's test almost verbatim: *is there an intended receiver who gave us the account number and who expects a payment from us? If not, we are not validating an account — we are searching for one.*

Two practical corollaries:

- **A name-lookup service that retrieves the holder's registered name from an account number is on the wrong side of this line unless the requester has a relationship.** It is the classic "penny drop" pattern, and the reason §1.4 flags it rather than folding it into Sense 1 is that it answers a different question and attracts a different permission test. ⚠
- **Scale is evidence.** A single probe is a verification; a pattern of probes against accounts the requester has no relationship with is what "searching for accounts" describes, and it is the same behavioural shape that Sense 2 produces against card BIN ranges (§5.1). **The mechanism at volume stops being a test and becomes a scan** — which is the second half of this guide's thesis.

---

## 12. The Alternatives and Their Displacement

For each alternative to a penny test: what it establishes that a penny test does not, and what it cannot.

### 12.1 Name-Match and Account-Checking Services

**What it is:** a query to the payee's institution that returns an outcome on the name supplied against the account, moving no value. Pay.UK's Confirmation of Payee (launched 2020; 300+ participating organisations; **more than 2 million checks a day**; API-based peer-to-peer with a directory; PSR-mandated expansion in 2024) ✅ and the EPC's Verification of Payee (inter-PSP rules and standards, ISO 20022-based API, responses of match / no match / close match / verification not possible, explicitly **not a payment instrument** and **not a means of identifying a person**) ✅.

**Establishes that a penny test does not:** *correspondence* — that the account belongs to the person or entity you named — which is the question that matters when you are paying an intended beneficiary. It also answers **in seconds, with no value moved, no ledger entry, no suspense item and no mark on the payee's statement.**

**Cannot:** prove control (it answers "is this the right account", not "is this yours"); work outside scheme/directory coverage; serve as evidence that money can flow (it is not a payment); and it discloses data of its own to the responding institution.

**The displacement, stated honestly.** This is the sense in which Sense 1 is *being displaced rather than improved*: the name-match asks a better question than the deposit does (§3.7), and where the scheme exists it is the correct default. Two repo sources hold the mechanism and should be read rather than re-derived here: [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 461 and [financial_infrastructure_guide.md](financial_infrastructure_guide.md) line 413 for the CoP angle, and [supply_chain_finance_guide.md](supply_chain_finance_guide.md) lines 491 and 564 for name-match beneficiary verification as an anti-diversion control.

### 12.2 Open-Banking-Style Instant Confirmation

**What it is:** the customer authorises an API connection to their own bank, through which the account and routing data are returned (and, in some products, the balance), with the account verified as part of the flow. The vendors document it as the *default-fast* path and the micro-deposit as the fallback: Plaid states **Instant Auth covers approximately 95% of users' eligible bank accounts**, with the remaining **~5%** served by micro-deposit or database verification ✅, and lists **Instant Match** for approximately **800 additional US institutions** and **Database Auth** (US and Canada) ✅. Dwolla's own comparison puts **Dwolla + Open Banking at ~85% U.S. bank coverage**, with "real-time account verification" and the bank account "automatically verified and ready to send and receive funds immediately after being added", against the API route at **100% coverage** but requiring verification ✅ (both retrieved 2026-09-29 — **each vendor's own description of its own coverage**; the percentages are the vendors' claims, not independently verified).

**Establishes that a penny test does not:** instantaneous verification with no value moved and no statement entry; and in some flows, additional facts (balance, account type, ownership data) that no micro-deposit can establish.

**Cannot:** cover the institutions outside the vendor's integration footprint — which is precisely why micro-deposits remain in every vendor's toolkit as the fallback ✅; and it is a data-sharing arrangement, with its own consent and expiry questions (Plaid documents consent/TAN expiration and the returns that follow from using stale numbers ✅).

**The practical settlement in the sources: it is a ladder, not a replacement.** Credentials/instant or matched verification first where available; database verification where the network exists; micro-deposits where none of those reach. **The penny test is the residual control, and residuals are honoured when a resilience plan is written and abandoned when a happy path is found.**

### 12.3 The Scheme's Own Validating Mechanisms

**What they are:** the mechanisms a scheme provides, as distinct from the ones an originator improvises — a permitted test/validation payment confined to a defined relationship and expectation ✅; a scheme message set that validates structure before value moves (the message-persona and technical-specification route ✅); a return-of-funds path with published guidelines ✅; a self-audit obligation with retained evidence ✅; and, on the ACH side, the industry's own investment in *account-validation infrastructure* rather than in probing — see Nacha's own news listing of a **multi-responder account validation network** integration, dated **20 April 2026** ✅ (nacha.org, retrieved 2026-09-29; title-level fact — the substance of the service was not retrieved and is not asserted here).

**Establishes that a penny test does not:** scheme-sanctioned permission boundaries (the "no searching for accounts" rule ✅), and a compliance framework that folds testing into ongoing audit rather than into an improvised probe ✅.

**Cannot:** substitute for the originator knowing what it actually needs to know — a scheme cannot tell you whether *your* counterparty's operations team will recognise *your* reference.

### 12.4 The Sandbox and the Simulated Rail

**What it is:** a simulated environment that mimics production. Vendors describe them as such ✅ (Dwolla: a free, full-featured environment simulating real API interactions, and the micro-deposit sandbox accepting **any amount below $0.10** ✅; Plaid documents sandbox testing for each Auth method ✅; the repo's [airwallex_guide.md](airwallex_guide.md), [adyen_guide.md](adyen_guide.md) and [mojaloop_guide.md](mojaloop_guide.md) cover the platform sandbox offer and sandbox demos).

**Establishes that a penny test does not:** near-instant, free, repeatable coverage of the integration's code paths, including error and edge cases that would be expensive to reproduce in production; and — as the relaxed-micro-deposit example shows ✅ — the ability to exercise a flow *without* its production security control.

**Cannot:** establish anything about the counterparty, the scheme's routing, production credentials or the cut-off (§6.1); and the sandbox's deliberate relaxation of the security property is the standing proof that a sandbox pass is evidence about your code and nothing else. **The sandbox is the cheapest test. It is also the least informative. That is why it is not the last test.**

### 12.5 The Comparison Table

| Alternative | Establishes that a penny test does not | Cannot establish | Cost of using it | Displaces which sense? |
| --- | --- | --- | --- | --- |
| **Name-match** (CoP ✅ / VOP ✅) | Correspondence between the name given and the account held; instant; no value moved; no statement entry | Control; availability outside scheme/directory coverage; that money can flow | A scheme/overlay participation and onboarding (with direct or aggregator access ✅) | **Sense 1** — the primary displacement, and a better question (§3.7) |
| **Open-banking instant confirmation** (vendor-described ✅) | Instant verification, often with balance/ownership data, at high coverage ✅ | The institutions outside the vendor's footprint — the residual that keeps micro-deposits alive ✅; also consent/expiry management is yours | Integration plus consent lifecycle management | **Sense 1**, partially — coverage-limited |
| **Scheme validating mechanisms** (permits, message specs, audits ✅) | Permission boundaries and a compliance frame; a message set that is validated before value moves | Your counterparty's operational reception of your specific payment; your own booking path | Governance and conformance work | **Sense 3**, partially — it is evidence *about conformance*, i.e. §9's distinction |
| **Sandbox / simulated rail** ✅ | Code correctness, edge cases, repeatability, at zero value | Anything about the network (§6.1) | Integration work only | None — it is the *complement* to Sense 3, not a substitute |
| **Zero-value file / prenote-style notice** ⚠ | Data validity and receipt without value (§6.3) | The value path, the booking, the statement, the reversal | Scheme-dependent availability | **Sense 3**, partially — the cheap first step where supported |
| **Not testing at all** | Nothing | Everything | The cost of discovering it in production | — |

---

## 13. The Anti-Patterns

Each with symptom, cause and guardrail.

| # | Anti-pattern | Symptom | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Testing on an irreversible rail with a value you cannot unwind** | A live probe is planned with a "we'll get it back" assumption, on a rail whose settlement is final ✅ | The batch-rail mental model carried onto an instant rail; the recovery path assumed rather than read | §7.3: value chosen as if it will not return; unwind tested in a certification environment or explicitly waived and approved in the mandate; **no probe whose acceptability depends on its recovery** |
| 2 | **A test payment invisible to reconciliation** | Reconciliation finds nothing; the item is absorbed, written off to expense, or never mentioned in the closure | The test was run by a delivery team that did not pre-register the item with the functions it would touch | §7.2: the item is expected by reconciliation, reaches suspense where suspense is correct, and the exception is closed with evidence **as part of the test's definition of done** |
| 3 | **A permanent monitoring whitelist created for a one-week test** | A suppression rule exists with no expiry and no mandate, months after the test closed; nobody can say who created it | Annotating test traffic is the obvious remedy and an unbounded annotation is the cheapest version of it | §5.5: time-bounded, mandate-bound, **counterparty/purpose-scoped and never amount-scoped**, non-silent (events retained), with proof of removal in the evidence pack |
| 4 | **Asserting a sandbox pass proves production routing** | A go/no-go pack cites sandbox test results as evidence of production readiness; the first live payment fails at the counterparty or the scheme | The substitution error: a test of *code* reported as a test of the *network* | §6.1 and §9.3: state explicitly which of §6.2's eleven hops the evidence covers; certification answers conformance, not operations; the live probe is the only evidence about the live path |
| 5 | **Treating a card-testing alert as noise** | A spike in declines/402s/low-value authorisations is dismissed as "wireless testing" or "just probes from our own acquisition flow" and not investigated | Small amounts are individually unremarkable — by design ✅ (§5.1) — and volume signals arrive in a queue that is measured on closure rate | §4.3 and the repo's own pattern list ([financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) line 476; [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) line 1047); the guardrail is the detection discipline owned there, plus the rule that **a low-value-alert type is not auto-closed** |
| 6 | **Verifying a beneficiary with a deposit where a name-match would answer better** | Beneficiary onboarding in a scheme-covered market uses micro-deposits, taking days and generating statement noise, when CoP/VOP-class checking is available ✅ | "We always do micro-deposits"; the deposit flow exists and the name-match onboarding does not | §3.7 and §12: ask which question is actually being asked — **control or correspondence** — and prefer the cheaper, faster, value-free instrument that answers it; keep micro-deposits for the residual where the overlay does not reach |
| 7 | **Letting the amount be predictable in a micro-deposit scheme** | Fixed or enumerable micro-deposit amounts; or a test/sandbox relaxation carried into production configuration | The unpredictability requirement (§3.3) is invisible in code review and easy to lose in a migration or a "make testing easier" change | §3.3: **the amount is the secret** — unpredictable amounts, a bounded attempt limit (Dwolla documents three attempts and a 48-hour recovery path ✅), and a configuration test that asserts randomness and attempt limits as *controls*, not as implementation details |
| 8 | **Annotating the test traffic inside the payment itself** | A verification code, "TEST" marker or scope note placed in a payment's party or reference fields | The natural place to tell the receiver "this is a test" is the payment data | §5.5: on RTP this is expressly prohibited for the Debtor name field ✅; the annotation belongs out-of-band, in your monitoring and your records, and the payment's fields are for the sender's identity |

---

## 14. The Cymbal Bank Worked Example — Designing a Test Protocol

> **Cymbal Bank is a fictional institution.** This worked example is explicitly illustrative: it shows one reasonable way to apply the disciplines above to a go-live, and it is the **only bank persona in this guide**. No real bank is asserted anywhere in this guide to run any particular test, and no detail below should be read as a description of any institution's actual practice.

### 14.1 The Situation

Cymbal Bank is going live on a **new payment channel**: a real-time payout product, sending instant credit transfers to beneficiaries at a partner set of receiving institutions through a new access path. Two features are new and both need proving: a **payee-verification feature** that checks the beneficiary name against the receiving institution before a first payment, and a **payout rail** whose settlement is final.

The programme has five candidate tests on the table, and the discipline of §8 and §9 says that not all five belong in production.

### 14.2 Certification Versus Live — the Split

| Test candidate | Where Cymbal runs it | Why |
| --- | --- | --- |
| Message conformance for the message set the product uses | **Certification** | Conformance is answered by the scheme's or operator's certification route (the RTP model of a message persona for which a participant "seeks certification" ✅ is the pattern), and a live payment proves nothing about conformance that certification does not prove better |
| Payee-verification feature behaviour — match, close match, no match, and the *cannot-check* outcome | **Certification/test environment for the schema and error paths; live for the counterparty's actual behaviour** | The response vocabulary is a scheme property ✅; what the *specific* receiving institution actually returns, in production, is not |
| Connectivity, credentials, sender identity | **Live** | Facts about the world (§6.1) |
| The value path: funding, booking, statement, reconciliation | **Live** | Only a real payment exercises it (§6.3) |
| The **unwind** on the final-settlement rail | **Certification environment** — see §14.5 | The production rail cannot be unwound on demand (§7.3) |

**What Cymbal's go/no-go pack therefore says, explicitly:** certification evidence is reported as *conformance evidence*; the live probe is reported as *operational evidence*; and the pack states which of §6.2's eleven hops each item covers. **The pack never says "certified, therefore ready."**

### 14.3 The Nominal-Value Probe and Where It Lands in the Books

Cymbal sends **one real credit of S$0.01** from a designated internal test account to a designated beneficiary account at the first receiving institution, on an agreed day inside an agreed window, with the counterparty's operations team expecting it.

Decisions taken in advance, in writing:

- **The value is S$0.01** — the smallest value that still forces a real settlement entry on both sides (§2.2). It is not a round number, because a round number is a value someone might later mistake for a real instruction.
- **The account it lands in:** the beneficiary's own nominated account at the partner institution, deliberately, because Cymbal wants the *customer-facing* path exercised — and Cymbal has agreed with the counterparty that the credit is expected and will be unwound per §14.4.
- **Where it lands in Cymbal's books:** a debit to the internal test account or cost centre named in the mandate, with the fee (if any) booked to the same cost centre, and **the entry left open until the accounting, reconciliation and closure items are resolved** (§10.3). It is not written off to expense on the day it is sent.
- **Who owns it:** the named owner in the test mandate, who is also the person who signs the closure record.

### 14.4 Reconciliation and Suspense Treatment

Before the probe runs, the reconciliation function is told the item's expected amount, value date, counterparty, reference and owner — so that it **can find it** (§7.2). Two outcomes are pre-defined and both are acceptable:

- **Matched** — Cymbal books it as designed and the reconciliation match is captured as evidence.
- **Unallocated at the counterparty** — the credit is visible as a suspense item, owned and ageing, and is then closed *by the agreed reverse route* with the closure reason recorded.

**The failure Cymbal is specifically watching for is neither of those: it is the item silently absorbed.** The programme's closure criteria say that an item which cannot be shown to have matched or been suspense-handled is an open finding regardless of the amount — and the finding is recorded against the *process*, not against the penny.

### 14.5 The Test Cymbal Refuses to Run in Production

**Cymbal refuses to test the unwind — the return/recall path — on the production rail.**

The rail settles finally ("Instant Settlement: **Final**, anytime, every day" ✅ is the operator's own description of the RTP class this example models), and the recovery route that exists on that class is **request-shaped**: a request for return of funds, which the other participant may or may not action ✅. Running the unwind test in production would mean Cymbal accepting that its recovery depended on a counterparty's discretion — and, worse, **planning a probe whose acceptability depended on its own recovery**, which §7.3 forbids.

So Cymbal runs the unwind exercise in the **certification environment**, accepts that the certification result is evidence about the *message flow* rather than about the production counterparty's behaviour, and records in the mandate an explicit, owner-approved statement that **the production probe is treated as irrecoverable** — S$0.01, approved as such, and not a surprise anyone discovers later. **That single decision is the clearest expression of §8.5's cross-rail principle in the whole example: the rail could not be unwound, so the probe was chosen differently.**

### 14.6 The Monitoring Annotation, with Its Expiry

Cymbal's new counterparty, new channel and unusual-hour test are a plausible alert (§5.4), so Cymbal annotates the test — under guardrails taken straight from §5.5:

- **Scope:** the suppression is keyed on **(counterparty, channel, product, direction, test reference)** — **not on amount**. Cymbal's fraud-monitoring owner explicitly rejects the amount-keyed version, because it would silence every genuine small payment on the new channel.
- **Mandate:** the suppression cites the change record, the test window, the counterparty and the named owner.
- **Expiry:** it expires **automatically at the end of the test window**, and the expiry date is recorded in the control register.
- **Non-silent:** the suppressed events are still captured and reviewable, so the test's monitoring behaviour can be evidenced rather than merely asserted.
- **Out-of-band:** nothing is written into the payment's own fields — a point Cymbal's team discovered from the scheme's own rule that the **Debtor name field of the pacs.008 message (index 2.855)** is for identifying the sender and **not for verification codes** ✅ (§5.5).
- **Closure evidence:** the pack includes proof that the suppression expired or was removed.

### 14.7 The Evidence Pack, and the Thesis

Cymbal's pack contains: the test mandate with its approved irrecoverability exception; the **pre-documented expectation** dated before the probe (expected entries at both ends, expected timestamps, expected statement description, expected reconciliation outcome, expected monitoring behaviour, expected non-events); the counterparty's confirmation; the payment and acknowledgement records; the accounting entries on both sides; the reconciliation match or the suspense item with its closure; the monitoring annotation record with its expiry and the retained suppressed events; the proof of removal; the certification-environment unwind result; and the closure record listing every deviation as a finding with a disposition.

**Then Cymbal states the test's result in one sentence:** *the channel moved value, booked it at both ends, was found by reconciliation, was contained by a bounded and now-expired annotation, and left evidence — at a total cost of one cent and one properly closed suspense item.*

**And that is the thesis, in the form a project can act on: the cheapest test is also the cheapest attack.**

---

## 15. The Claims Audit — Verified, Flagged, Rejected

### 15.1 The Verified Claims (✅)

Every claim below was retrieved from the named source on the date shown. **Vendor and operator statements are labelled as the vendor's or operator's own description** — they are accurate reports of what that party publishes, not independently corroborated facts.

| # | Claim | Source | Date | Quality |
| --- | --- | --- | --- | --- |
| 1 | Micro-deposit verification = two deposits of less than $0.10 with random amounts; post in 1–2 business days; the order amounts are entered does not matter; **three attempts** then the amounts are unusable; recovery = remove funding source, wait 48 hours, re-add, re-initiate | Dwolla developer documentation, "Verify Bank with Micro-deposits" | retrieved 2026-09-29 | Vendor's own documentation — high for its own product flow |
| 2 | In Dwolla's sandbox, **any amount below $0.10** allows immediate verification | Dwolla, same page | retrieved 2026-09-29 | Vendor's own documentation — high |
| 3 | Dwolla sandbox = "a free, full-featured environment that **simulates** real API interactions"; unverified funding source can **receive** but not **send**; verification required before sending funds, stated in ACH terms | Dwolla developer documentation, "Testing in the Sandbox" and "Bank Funding Source" | retrieved 2026-09-29 | Vendor's own documentation — high for its own product |
| 4 | Instant Auth covers **~95%** of users' eligible bank accounts; remaining **~5%** use micro-deposit/database verification; Instant Match for **~800** additional US institutions; **Instant Micro-deposits** make the deposit **over RTP or FedNow** with manual verification **in as little as 5 seconds**; **Same-Day Micro-deposits** over Same Day ACH in ~1 business day; **Database Auth** US/CA; micro-deposit-verified Items cannot be used with other Plaid products except Auth/Transfer, with a partial Identity Match exception (~30%) | Plaid Auth documentation ("Additional Auth flows", "Instant Auth, Instant Match, & Instant Micro-deposits") | retrieved 2026-09-29 | Vendor's own documentation — high for its own flows; **the coverage percentages are the vendor's claims** |
| 5 | Plaid documents **consent/TAN expiration** for Auth account-and-routing-number data, with remediation via update mode, and states that using the numbers without completing the flow **may result in ACH returns** | Plaid Auth documentation | retrieved 2026-09-29 | Vendor's own documentation — high |
| 6 | Card testing = determining whether stolen card information is valid; synonyms **carding, account testing, enumeration, card checking**; attackers use **card setup** (preferred, because such validations **don't typically show up on cardholder statements**) and **small amount payments** (less likely to be noticed and reported); **scripts** test large amounts of card data at once, collecting **3D Secure authentication or issuer responses**; valid cards are cashed or resold on the dark web | Stripe documentation, "Protect yourself from card testing" | retrieved 2026-09-29 | Vendor's own documentation — high as a description of the typology |
| 7 | Card-testing consequences per Stripe: disputes (Early Fraud Warnings / fraudulent disputes); **higher decline rates** that can persist for legitimate payments **even after card testing ceases**; additional authorisation and dispute fees; infrastructure strain; possible enrolment in **Card Monitoring Programmes**; corrupted business data | Stripe, same page | retrieved 2026-09-29 | Vendor's own documentation — high; **no statistics, rates or loss figures are published in it** |
| 8 | Card-testing identification symptoms per Stripe: significant increase in **failed** authorisations/payments; spikes in failed/blocked payments and in **402** errors; a `generic_decline` outcome; spikes in suspicious payments **with low transaction amounts** | Stripe, same page | retrieved 2026-09-29 | Vendor's own documentation — high |
| 9 | Card testing is not stopped by a single heuristic such as IP address; abuse of a **publishable key**, and leaked/stolen secret keys, are named enablers | Stripe, same page | retrieved 2026-09-29 | Vendor's own documentation — high |
| 10 | Confirmation of Payee = an **account name-checking** service for UK domestic payments; **launched 2020**; **over 300 organisations**; **more than 2 million checks per day**; **API-based peer-to-peer with no central infrastructure** but with a directory; **PSR mandated its expansion in 2024**; direct and aggregator access models; a published list of aggregators; **Payer Name Verification** for Bacs Direct Debit set-up/amendments | Pay.UK, "Confirmation of Payee" | retrieved 2026-09-29 | Operator's own published material — high |
| 11 | Verification of Payee = inter-PSP rules/practices/standards for SEPA, **API-based using ISO 20022 elements**; the payer's PSP asks the payee's PSP to verify IBAN and name (optionally an identification code such as VAT number, LEI or social-security code); responses include **match, no match, close match, verification not possible**; it is **not a payment means or payment instrument** and **cannot be relied upon to identify a natural or legal person**; published evolution calendar — **rulebook v1.1 published 16 March 2026**, **effective 20 September 2026**, consultation 1 April–30 June 2026, **v2.0 expected end November 2026** | European Payments Council, "Verification of Payee" | retrieved 2026-09-29 | Scheme owner's own published material — high |
| 12 | **RTP Operating Rule II.E.1** as quoted in the bulletin: payment messages may only be used to determine whether account numbers are associated with valid, active accounts **when the account number was given by an intended Receiver who is expecting one or more payments** from the Sender; **RTP Participants are not permitted** to send payments to verify account/routing information of accountholders **who do not expect to receive** payments from the Sender; the Compliance Guide's permissible example is a broker's **small test RTP Payment** to a consumer expecting an RTP payment from that broker; the **"Debtor name" field in pacs.008 (index 2.855)** should identify the Sender and **not** carry a verification code; non-compliance may attract penalties under section X of the Operating Rules | The Clearing House, **RTP Network Compliance Bulletin 1-2024, "No Searching for Accounts Rule"** | bulletin dated **5 September 2024**; retrieved 2026-09-29 | **Operator's own public compliance bulletin — the highest-quality source obtained in this pass** |
| 13 | RTP's own published claims: transactions **up to $10 million**; "**Instant Settlement: Final, anytime, every day**"; availability "around the clock, including bank holidays, weekends and after hours"; "**100% uptime, zero scheduled downtime**"; cumulative **1.7 billion transactions / over $3.4 trillion** since 2017; **142 million transactions / $576 billion** in Q2 2026; daily average value **$6.6 billion** as of August 2026; **over 1,357 participants** as of August 2026 | The Clearing House, RTP network page | retrieved 2026-09-29 | Operator's own marketing and statistics — high as *what the operator claims*; ⚠ the uptime claim is not independently verified here |
| 14 | RTP document set (public, RTP Document Library): Operating Rules **effective 06-01-2026** with upcoming versions **09-30-2026** and **10-04-2026**; Participation Rules likewise; **RTP Message Specifications v5.0**; Reports Specification v5.2; **Request for Return of Funds Guidelines and Suggested Practices**; rule interpretation "**Returning Funds After Providing an Accept Response to a Credit Transfer**" **effective 25 July 2024**; Compliance Bulletins 1-2026 (fraud reporting) and 1-2024 (no searching for accounts); **Risk Management and Fraud Control Requirements Schedule**; RTP Self-Audit workbook and form; enabled routing/transit-number list | The Clearing House, RTP Document Library and RTP Participant Self-Audit page | retrieved 2026-09-29 | Operator's published document set — high as existence/dating of documents |
| 15 | RTP Participants must **audit compliance at least once each calendar year** (RTP Operating Rules **section IX.A.2**), retain the **RTP Self-Audit Form** as evidence, and report **material findings of non-compliance to the audit committee** or equivalent body | The Clearing House, RTP Participant Self-Audit page | retrieved 2026-09-29 | Operator's own rule requirement — high |
| 16 | RTP participants select a **"Message Persona"** and "**seek certification**" for it during onboarding; TCH "**performs technical testing with TPSPs**" to confirm capability against the RTP technical specifications | The Clearing House, **RTP Network Readiness Checklist v1.0** | document dated **September 2020**; retrieved 2026-09-29 | Operator's own onboarding guidance — high for the onboarding process at that date; ⚠ dated |
| 17 | RTP flow and obligations per the readiness checklist: payments clear and settle individually in real time with **immediate finality**; **prefunded, real-time, gross settlement** with a Prefunded Balance Account at the FRBNY; foundational messages are **Credit Transfer, Message Status Report (Confirmation), Request for Payment, Request for Information, Remittance/Invoice Detail, Request for Return of Funds** plus system administrative messages; **ISO 20022** adopted; a Receiving Participant must **immediately** respond **Accept / Reject / Accept without Posting** and provide immediate funds availability for accepted payments; sending participants must use **multi-factor authentication** for customers initiating payments and must provide the Receiver's name for consumer-originated payments; a written **OFAC** compliance programme is required; principal may not be reduced to collect fees; prohibitions include "**no foreign payments, no searching for accounts**"; participants must act on fraud alerts and report fraud; national banks must notify the OCC **30 days** before go-live (**OCC Interpretive Letter 1157, December 2017**) | The Clearing House, RTP Network Readiness Checklist v1.0 | document dated **September 2020**; retrieved 2026-09-29 | Operator's own onboarding guidance — high; ⚠ the rule set has been amended since (see claim 14) — **rulebook versions govern** |
| 18 | ACH Network: open **23¼ hours** each business day; **settles four times a day**; Federal Reserve settlement closed on federal holidays and weekends and on business days **6:30 p.m.–7:30 a.m. ET**; **Same Day ACH went live 2016**; the network's stated per-payment limit is **$1 million**; a rule amendment will raise the Same Day ACH per-entry limit to **$10 million effective 17 September 2027** | Nacha, "The ABCs of ACH" and the Nacha rules index | retrieved 2026-09-29 | Operator's own published material — high |
| 19 | Nacha 2026 risk-management rule calendar: **Risk Management Topics — Fraud Monitoring Phase 1 effective 20 March 2026**; **Phase 2 effective 19 June 2026** (practical effective date **22 June 2026**, 19 June being a federal holiday); **Company Entry Descriptions** rules effective **20 March 2026**; a new sanctions-related return reason code (**R90**) effective **17 September 2027**; the package's stated purpose is to "reduce the incidence of successful fraud attempts and improve the recovery of funds after frauds have occurred" | Nacha, rules index | retrieved 2026-09-29 | Operator's own rule listing — high |
| 20 | Nacha's own news listing includes an announcement dated **20 April 2026** of a "multi-responder account validation network" integration | Nacha news listings | item dated **20 April 2026**; retrieved 2026-09-29 | Operator's own news listing — **title-level fact only**; the substance of the service was not retrieved and is not asserted |
| 21 | FedNow: enables eligible depository institutions to send and receive instant payments "in real time, around the clock, every day of the year", with recipients having full access to funds immediately; uses **ISO 20022**; transfers governed by **Operating Circular 8** and **subpart C of Regulation J**; the **FedNow DevRel** technical resource is **credential-gated** ("credentialed-only access") to participants and service providers | Federal Reserve Financial Services — FedNow pages and resources | retrieved 2026-09-29 | Operator's own published material — high |
| 22 | The RTP network's enabled routing/transit-number list and the participant list are published; the RTP network can be joined without ownership/membership of The Clearing House; participation is open to any federally insured depository institution and certain other FIs | The Clearing House, RTP pages | retrieved 2026-09-29 | Operator's own published material — high |
| 23 | The repo's internal datapoints relied on by name: card testing named at [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) line 476 and [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) line 1047; CoP mentions at [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 461 and [financial_infrastructure_guide.md](financial_infrastructure_guide.md) line 413; name-match beneficiary verification at [supply_chain_finance_guide.md](supply_chain_finance_guide.md) lines 491 and 564; the limit-boundary test listing "at limit and one cent over" at [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) line 471 | The repository's own guides | verified in this pass 2026-09-29 | Repo-internal — high |

### 15.2 The Flagged Claims (⚠)

Flagged claims are things this guide states but could **not** establish at source. They are flagged inline where used and collected here.

| # | Flagged item | Why it is flagged | How it is used in this guide |
| --- | --- | --- | --- |
| 1 | **"Penny drop"** as the Indian-market name for nominal-credit account verification, and any specific IMPS/NEFT/UPI amount, API behaviour or timing attached to it | No primary vendor or scheme page could be retrieved: razorpay.com returned empty content on every URL attempted, cashfree.com/docs returned 404s, and setu.co returned 404s. **The term is treated as market vocabulary, not as a sourced mechanism** | §1.4 (decoder), §11.4 — flagged, never given a mechanism |
| 2 | **Prenote / zero-value file** rule substance (amount, timing, the specific rule text, availability across rails) | nacha.org returned page-not-found for the rule and glossary URLs attempted; the concept appears in this guide as a **mechanism**, not as a quoted rule | §1.4, §6.3, §12.5 — the mechanism is described, the rule is not asserted |
| 3 | **Nacha "micro-entry" rule** and the account-validation rule specifics | Not retrieved at source in this pass | Referred to only generically in §6.3 and §12.3; **no rule text, precision-window or effective date is asserted** |
| 4 | **Card-scheme certification, sandbox and test-card conventions** (any scheme's) | Card-scheme public documentation was not retrieved | §9.1 flags the row explicitly; no card-scheme certification requirement is stated anywhere in this guide |
| 5 | **SWIFT message types, test-message conventions and CSP attestation requirements** | swift.com failed to return content on every URL attempted (a scrape/tool failure, not evidence of absence) | §8.3 and §9.1 flag it; the repo's SWIFT guides are cross-referenced for mechanics rather than re-derived |
| 6 | **ISO 20022 message definitions** beyond the one field cited inside the RTP bulletin | iso20022.org failed to return content on every URL attempted | The only ISO 20022 field claim in this guide — **pacs.008, index 2.855, "Debtor name"** ✅ — is sourced to the RTP bulletin, not to the ISO catalogue |
| 7 | **Card-testing statistics**: incidence, loss figures, decline rates, detection rates | **None published in the sources retrieved**; Stripe's documentation is qualitative | §4.3 and §4.4 state the pattern qualitatively and record the absence rather than supplying a number |
| 8 | **Operator claims about uptime, volume, value, participant counts and coverage percentages** (TCH's "100% uptime, zero scheduled downtime"; Plaid's ~95%/~800/~30%; Dwolla's ~85%) | Each is the party's own published claim and is not independently corroborated in this pass | Cited **as the party's own description**, never as an independently verified fact (§15.1 claims 4, 13) |
| 9 | **Specific cut-off times** for any rail | Not established at source; only the structure (e.g. ACH's settlement-window closure, and the absence of a cut-off on continuous rails) is sourced | §1.4 and §6.2 treat the cut-off as a **thing to test**, not as a stated time |
| 10 | **Amount limits** other than the public Nacha and RTP figures | Not established | §2.2 and §8.4 state that limits are scheme- and member-specific and must be tested at the boundary; no limit is asserted except the sourced ones |
| 11 | **RTP rule substance** beyond the Compliance Bulletin 1-2024 quotation and the readiness-checklist summary — i.e. the full text of rule II.E.1, the risk-management schedule, and the current rule versions | The Operating/Participation Rules PDFs are published, but only their titles, dates and the quoted passages were read in this pass; **the rulebook is version-specific and the versions change on published dates** (06-01-2026 current, with 09-30-2026 and 10-04-2026 upcoming, per claim 14) | Quoted passages are quoted exactly; everything else is cited as a document's existence and date |

### 15.3 The Rejected or Not-Found Claims and the Absences

| Item | Status | Note |
| --- | --- | --- |
| Any **loss, fraud-rate, decline-rate or detection-rate figure** for card testing, micro-deposit fraud or test payments | ❌ **Not found** | Deliberately absent from this guide; the sources retrieved publish no such figures |
| Any **scheme rule reconstructed from memory** (message type numbers, cut-offs, micro-entry amounts, certification steps) | ❌ **Rejected by method** | This guide contains no such reconstruction; where a rule's substance could not be retrieved it is flagged ⚠ (§15.2) |
| **`web_search` results** | ❌ **Tool failure, not an absence** | Every `web_search` query returned an **empty result set** in this pass. Recorded as a tool limitation (§16.1). All sourcing was done by direct extraction from primary URLs. This must **not** be read as "no material exists" on any subject searched |
| **razorpay.com, cashfree.com/docs, setu.co, developer.mastercard.com** | ❌ **Not retrieved** | Razorpay returned empty content; Cashfree and Setu returned 404s; Mastercard's developer site returned only a loading shell. The Indian-market "penny drop" mechanism and card-scheme account-status services are therefore **unverified** in this guide |
| **swift.com, iso20022.org** | ❌ **Not retrieved** (scrape failure) | Flagged in §8.3, §9.1 and §15.2 |
| A **single published source that names "the penny test"** as a term of art across all three senses | ❌ **Not found** | The unifying framing in §2 is this guide's own **labelled analysis**, not a sourced convention — stated as such in §2.1 |
| A **published account of a bank's go-live test-payment protocol** | ❌ **Not found** | No real institution's protocol is described in this guide; the only worked protocol is the fictional Cymbal Bank example (§14), which is labelled illustrative |

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified

Stated as absences and limitations, in the discipline the repository requires:

- **Tool limitation.** `web_search` returned an **empty result set for every query** in this pass. This is a tool failure, **not evidence of absence** — all sourcing was performed by direct extraction from primary URLs, fetched 2026-09-29.
- **Rule texts and named mechanisms not retrieved.** The Nacha prenote/zero-value-entry rule text and the Nacha micro-entry and account-validation rule specifics (nacha.org returned page-not-found on the URLs attempted); and the Indian-market "penny drop" mechanism (razorpay.com returned empty content; cashfree.com/docs and setu.co returned 404s; developer.mastercard.com returned only a loading shell). **No rule text, amount, precision window, timing or API behaviour is asserted for any of them** — they appear in this guide as mechanisms or as vocabulary, flagged ⚠.
- **Vendor and standards sites not retrieved.** swift.com and iso20022.org failed to return content, and no public card-scheme documentation on certification, sandbox or test-card conventions was obtained. Therefore **no SWIFT message type, test-message convention or CSP attestation requirement; no ISO 20022 message definition beyond the one field quoted inside a scheme bulletin; and no card-scheme certification requirement is asserted anywhere in this guide.**
- **No figures.** No card-testing statistic, loss figure, decline rate or detection rate was available at source and none is stated; **no specific cut-off time is asserted for any rail** (cut-offs are treated as things to test, not as published constants); and **no compensation, cost or market-size figure appears** except the operator-published statistics and limits attributed by name and date in §15.1, which are labelled as the operator's own claims.
- **Rulebook scope, and two absences of a different kind.** The full RTP rulebook text was not read — only the Compliance Bulletin 1-2024 quotations, the readiness-checklist summary (v1.0, September 2020) and the titles and effective dates of the rule documents (rule substance beyond the quoted passages is flagged ⚠, §15.2 item 11). Separately: **no published source retrieved in this pass names "the penny test" as a single term covering all three senses** — the unification in §2.1 is this guide's own labelled analysis — and **no bank's actual test protocol was found or is described**, the only worked protocol being the fictional, labelled example in §14.

### 16.2 The Glossary

| Term | Meaning as used in this guide |
| --- | --- |
| **Penny test** | This guide's shorthand for the shared mechanism: a nominal-value payment used as an instrument of verification rather than as payment for anything. Carries three distinct meanings in practice (§1) |
| **Sense 1 / Sense 2 / Sense 3** | Account/beneficiary verification by micro-deposit; instrument validation by tiny authorisation (legitimate card-on-file validation, or the fraud typology card testing); the live nominal-value go-live test payment |
| **Micro-deposit** | A small credit (typically two, of less than $0.10 in the documented ACH implementations) sent to an account to prove control ✅ |
| **Penny drop** | The Indian-market name for nominal-credit account verification by a gateway; treated in this guide as vocabulary and flagged ⚠ |
| **Micro-entry** | A small credit used for account validation; the rule specifics were not retrieved this pass ⚠ |
| **Name-match** | A value-free query to the payee's institution returning an outcome on the name supplied against the account (CoP, VOP) ✅ |
| **Confirmation of Payee (CoP)** | The UK account name-checking overlay service, launched 2020 ✅ |
| **Verification of Payee (VOP)** | The EPC's SEPA scheme for payer-PSP → payee-PSP payee verification; rules, practices and standards; not a payment instrument ✅ |
| **Prenote / zero-value file** | Advance notice of an entry without moving value, used to validate account data before value moves ⚠ (rule text not retrieved) |
| **Nominal value** | A deliberately trivial but real, non-zero amount; used because it forces a real settlement entry on both sides |
| **Pre-authorisation** | An authorisation held against an instrument ahead of a charge; the small-value form is the card-side validation probe ✅ |
| **Card-on-file validation** | The legitimate merchant-side Sense 2: validating a stored instrument at setup |
| **Card testing / carding / account testing / enumeration / card checking** | The abusive Sense 2: small authorisations (or card-setup validations) used to determine whether stolen card data is live ✅ |
| **Card setup** | The validation-at-setup variant that, per Stripe, typically does not appear on cardholder statements and is therefore preferred by fraudulent actors ✅ |
| **Smoke test / certification test** | A team's own quick "does it work" run, versus a scheme's or operator's structured conformance exercise — the latter is evidence about conformance, not about production readiness (§9) |
| **Message persona** | The RTP construct describing which message set a participant implements and seeks certification for ✅ |
| **Suspense entry** | A ledger position holding value whose allocation is undetermined; a visible, owned, ageing exception is the system working |
| **Cut-off** | The time by which an instruction must be received for a given processing cycle — a thing to test, not a published constant in this guide |
| **Finality / irreversibility** | The property of instant-rail settlement; the reason a probe must be chosen differently where value cannot be unwound ✅ |
| **Request for Return of Funds** | The RTP message by which funds are requested back — a request, not a reversal ✅ |
| **Test mandate** | The named change record authorising a test, its window, scope, owner and any irrecoverability exception |
| **Monitoring annotation / whitelist** | A suppression of alerting for test traffic; a control that must be time-bounded, mandate-bound and non-silent (§5.5) |
| **Reconciliation visibility rule / cross-rail principle** | A test payment invisible to reconciliation and suspense is worse than no test (§7.2); a probe that cannot be reversed must be chosen differently from one that can (§8.5) |

### 16.3 The Cross-References

**Owned by this guide:** the practice of establishing something with a nominal-value payment — Sense 1 (micro-deposit verification), Sense 2 (instrument validation and its fraud typology) and Sense 3 (the live go-live test payment) — and the discipline each sense demands.

**Cross-referenced and not re-derived:** [payment_rails_guide.md](payment_rails_guide.md) (the rails, the taxonomy, the real-time map, the message standards — and the owner of rail mechanics); [financial_fraud_detection_at_scale_guide.md](financial_fraud_detection_at_scale_guide.md) and [../technology/event_stream_processing_guide.md](../technology/event_stream_processing_guide.md) (the detection machinery and the pattern primitives); [micropayment_options_research.md](micropayment_options_research.md) (micropayments as a product — distinguished in §1.5); [ibps_payment_connect_guide.md](ibps_payment_connect_guide.md) and [financial_infrastructure_guide.md](financial_infrastructure_guide.md) (the repo's Confirmation-of-Payee material), [supply_chain_finance_guide.md](supply_chain_finance_guide.md) (name-match beneficiary verification); [swift_alliance_access_guide.md](swift_alliance_access_guide.md), [swiftnet_fileact_guide.md](swiftnet_fileact_guide.md), [iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md), [singapore_fintech_payments_guide.md](singapore_fintech_payments_guide.md), [cnaps_guide.md](cnaps_guide.md), [nucc_netsunion_guide.md](nucc_netsunion_guide.md), [payments_hub_guide.md](payments_hub_guide.md), [nets_singapore_guide.md](nets_singapore_guide.md) (their rails' mechanics); [airwallex_guide.md](airwallex_guide.md), [adyen_guide.md](adyen_guide.md), [mojaloop_guide.md](mojaloop_guide.md) (the sandbox and simulated-rail offer); [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) (entry, suspense, reversal mechanics); [operational_resilience_framework_guide.md](operational_resilience_framework_guide.md) (change-and-test governance); [fapi_financial_grade_api_guide.md](fapi_financial_grade_api_guide.md) (the open-banking API layer behind instant verification); and [../technology/distributed_rate_limiter_guide.md](../technology/distributed_rate_limiter_guide.md) and [../technology/cybersecurity_guide.md](../technology/cybersecurity_guide.md) (rate limiting as mitigation; the stolen-credential market behind Sense 2).

### 16.4 The Closing Summary

**Three practices, one name, one mechanism.** Micro-deposit verification proves that a customer controls an account. Instrument validation proves that a card will accept a charge — and in the wrong hands that is card testing, the cheapest opening move in payment-card fraud. The live nominal-value test payment proves that a channel, a counterparty, a scheme route, a set of credentials and a cut-off all work together in production, which no sandbox can prove.

**What unites them is not the name but the arithmetic.** The information a payment yields — account exists, route valid, credentials live, counterparty accepts, timing holds — does not scale with the value, while the cost of the test does, and on an irreversible rail the cost is the whole amount. So the smallest value that still forces a real settlement entry is the best probe that exists, for the legitimate user and for the attacker alike.

**That symmetry is the operational problem, not a curiosity.** A bank's monitoring sees value, counterparty, channel, cadence and customer — not intent — and the amount, the one feature always visible, is the one feature designed to carry no signal. What distinguishes a customer's verification from a fraudster's validation is relationship, expectation, cadence, dispersion and credential reputation, and the schemes themselves have drawn that line by rule rather than by threshold. The ambiguity is real and unresolved; the controls that manage it are outside the amount.

**And a live test is not free of consequence merely because it is small.** It is a real entry in two institutions' books, it must be seen by reconciliation and suspense — because a test payment the controls never notice is worse than no test — it cannot be unwound at will on a final-settlement rail, and its monitoring annotation is itself a control that becomes a hole the moment it loses its expiry and its mandate. Run it small, choose it for the rail's reversibility, expect it in advance, contain it, evidence it, close it.

The mechanism is cheap, which is exactly why it must be governed: the cheapest test is also the cheapest attack.
