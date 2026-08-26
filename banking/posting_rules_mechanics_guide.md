# The Posting Mechanics in Core Banking: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Banking / Banking Architecture — the posting-mechanics deep-dive: the calculation-layer companion to the posting-engine machinery guide — the debit/credit determination rules and the product × event × account matrix, the GL-string generation, the reversal and storno mechanics (the direct reversal vs the reversing-entry variants, the mid-period reversals, the regulatory aspects), the suspense and the exception layer (the suspense accounts, the repair queues, the unmatched items, the breaks), the posting precision (the rounding modes, the decimal-vs-float discipline, the balancing arithmetic), the multi-currency posting math (the FX conversions, the rate capture, the round-trip balancing), the idempotency and the replay safety (the idempotency keys, the duplicate detection, the replay), the reconciliation mechanics (the sub-ledger-vs-GL balancing, the control accounts, the breaks and the corrections), a fully worked postings library (13 journal entries, every amount re-computed with Python decimal), the golden-test discipline, the Cymbal Bank context, a full posting-mechanics design, and the summary ('the balanced books')
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview: The Posting-Mechanics View](#1-overview-the-posting-mechanics-view)
2. [The Posting Rules](#2-the-posting-rules)
3. [The Reversal Mechanics](#3-the-reversal-mechanics)
4. [The Suspense and the Exceptions](#4-the-suspense-and-the-exceptions)
5. [The Posting Precision](#5-the-posting-precision)
6. [The Multi-Currency Postings](#6-the-multi-currency-postings)
7. [The Idempotency](#7-the-idempotency)
8. [The Reconciliation](#8-the-reconciliation)
9. [The Worked Postings](#9-the-worked-postings)
10. [The Golden Tests](#10-the-golden-tests)
11. [The Banking Context: Cymbal Bank](#11-the-banking-context-cymbal-bank)
12. [The Worked Example: A Posting-Mechanics Design](#12-the-worked-example-a-posting-mechanics-design)
13. [The Summary: The Balanced Books](#13-the-summary-the-balanced-books)
14. [Glossary](#14-glossary)
15. [Verification Ledger](#15-verification-ledger)

---

### How to Read This Guide

This guide is the **posting-mechanics companion** to the repo's posting-engine overview. Where [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) covers the *machinery* — the accounting foundations, the posting lifecycle (§3), the engine architecture (§4), balance management (§5), GL integration (§6), the vendors (§7), the architect's view (§8), and one fully worked fund-transfer example (§9) — this guide goes one level deeper into the **mechanics themselves**: *which* account gets debited and *which* credited for every product × event combination, *how* the GL strings are assembled, *how* reversals are engineered so the books survive them, *how* suspense and exceptions are parked without polluting the P&L, *how* the arithmetic stays exact at scale, and *how* the same transaction can be replayed without double-posting. The two guides are designed to be read together: the sibling answers *how the engine is built and deployed*; this one answers *what the engine does with each transaction and why the books stay balanced*.

The pair deliberately mirrors the interest pair in the repo: [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) (the machinery) and [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) (the calculation layer). This guide is the **calculation-layer mirror for postings**: it assumes the sibling's machinery and deepens exactly what the sibling does not — the rules (§2), the reversals (§3), the suspense (§4), the precision (§5, the mirror of the interest guide's §6), the FX math (§6), the idempotency (§7), the reconciliation (§8), the worked-postings library (§9, the mirror of the interest guide's §10), and the golden-test discipline (§10, the mirror of the interest guide's §11).

- **Relationship map.** The umbrella guide is [core_banking_systems_guide.md](core_banking_systems_guide.md) (the core platform). The engine-family siblings are [posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) (the posting machinery), [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) (the interest machinery), [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) (the interest calculation layer), [banking_limits_domain_guide.md](banking_limits_domain_guide.md) (the limit engine), and this guide (the posting mechanics layer). The EOD context is in [core_banking_processes_guide.md](core_banking_processes_guide.md) §7; the payment flows that end in postings are in [payment_rails_guide.md](payment_rails_guide.md), [payments_hub_guide.md](payments_hub_guide.md), and [nets_software_systems_guide.md](nets_software_systems_guide.md).
- **Reading paths.** (1) *Architects:* §1, §2, §5, §6, §8, §10, §12. (2) *Product owners configuring rules:* §2, §3, §4, §9. (3) *Engineers implementing/testing:* §2, §3, §5, §6, §7, §9, §10. (4) *Finance/accounting:* §2, §3, §4, §8, §9, §13. (5) *In a hurry:* §1, §2.4, §9, §13.
- **A note on verification.** Researched August 2026. The accounting canon (Pacioli's 1494 *Summa*, the double-entry rule, the accounting equation, DEAD CLIC) was verified against the standard sources and the sibling's §1; the storno practice against Microsoft's Dynamics 365 localization documentation ("storno accounting uses negative debit or credit amounts to reverse original journal account entries" — the *red storno*); the suspense and control-account concepts against the accounting literature; the ISO 4217 minor-unit precision against the standard; the MAS notices against mas.gov.sg (Notice 610 is the *statistics-and-returns* notice — see the verification ledger for a correction to the sibling — and Notice 626 the AML/CFT notice with 5-year record retention); idempotency against the Stripe idempotency-request documentation; and **every amount in §9 was independently re-computed at 40-digit Python `decimal` precision during the writing of this guide** (the same discipline the interest guide applies to its §10). Claims that are engineering consensus rather than documented fact are marked **(consensus)**; anything that could not be verified is flagged in place and in the verification ledger (§15).

---

## 1. Overview: The Posting-Mechanics View

### 1.1 The Definition: What Posting Mechanics Are

The sibling guide defines **the posting engine** as the component that executes postings — balance updates, entry creation, GL posting — on every money movement from every product ([posting_engine_core_banking_guide.md](posting_engine_core_banking_guide.md) §4.1). **Posting mechanics** is the layer *inside* that engine that decides, for each transaction:

```text
which accounts  →  which sides (DR / CR)  →  which amounts  →  which GL strings
→  which value dates  →  which reversals undo it  →  which queue catches its failures
```

It is the difference between *the engine exists* and *the engine is correct*. A posting engine can have perfect atomicity, locking, and performance (the machinery) and still misstate the books if a rule debits the wrong account — the **wrong-but-balanced entry**, the hardest error class in banking software because the trial balance stays green while the P&L quietly lies (sibling §1.1, §4.3). Posting mechanics is the collection of *deterministic decisions* that make wrong-but-balanced entries impossible by construction: the rules are configured, reviewed, versioned, and golden-tested (§2, §10); the reversals are constrained to a small set of auditable patterns (§3); the exceptions have nowhere to hide (§4); the arithmetic is exact (§5); the currencies balance twice (§6); and the replay is provably safe (§7).

The discipline mirrors the interest calculation layer's: **calculation is deterministic pure math; booking is idempotent accounting** ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §1.2). For postings the equivalent rule is: **rule resolution is deterministic configuration; posting is atomic, balanced, and replayable** — a replayable rule history and entry stream let any historical day be recomputed and diffed against what was booked (§10.4).

### 1.2 The Overview Table

The table maps every mechanics topic of this guide to the sibling's machinery sections — where the machinery lives, and where this guide deepens:

| Aspect | This guide's treatment | Machinery home (sibling) |
|---|---|---|
| **Posting rules** | §2 — the product × event × account matrix, the account-determination precedence, the GL-string grammar | §4.3 (the posting rule engine), §4.4 (the account resolver) |
| **Reversal mechanics** | §3 — the storno variants, the reversing-entry patterns, the mid-period reversals, the regulatory constraints | §3.7 (reversals), §8.4 (the no-delete rule) |
| **Suspense and exceptions** | §4 — the suspense-account choreography, the repair queues, the unmatched items and breaks | §6.4 (suspense accounts), §3.8 (failed postings) |
| **Posting precision** | §5 — the rounding modes, the decimal-vs-float ban, the balancing arithmetic — the mirror of [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §6 | §8.2 (accuracy), §4.6 (the entry engine) |
| **Multi-currency postings** | §6 — the FX leg construction, the rate capture, the round-trip balancing | §4.10 (multi-currency posting), §6.5 (revaluation in EOD) |
| **Idempotency** | §7 — the idempotency keys, the duplicate detection, the replay safety | §8.2.2 (idempotency), §3.1 (key assignment at initiation) |
| **Reconciliation** | §8 — the sub-ledger-vs-GL balancing, the control accounts, the breaks and corrections | §1.9 (GL vs sub-ledger), §6.2 (the daily reconciliation) |
| **Worked postings** | §9 — 13 fully worked journal entries, all arithmetic re-computed | §9 (one worked fund-transfer example) |
| **Golden tests** | §10 — the verification discipline, the mirror of the interest guide's §11 | §8.9 (testing the posting engine) |
| **Banking context** | §11 — the Cymbal Bank posting flows | §4.1 (the engine's place in the core) |

### 1.3 The Mechanics Mindset: Four Properties

The same four properties the interest guide demands of the calculation layer ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §1.3), applied to postings:

- **Determinism.** The same (transaction type, product, event, accounts, amounts) resolves the same legs, everywhere, every time — no execution-order dependence, no "best guess" GL, no implicit defaults (§2.2).
- **Precision discipline.** Money arithmetic is decimal, never binary float; rounding happens at explicitly chosen points; methods are product parameters, not code accidents (§5).
- **Balancing as an invariant.** Σ DR = Σ CR is checked per transaction at build, per book per day at the trial balance, and per control account at reconciliation — three layers (§5.3, §8).
- **Auditability.** Every entry is immutable, attributable, and linked to its transaction and its reversal; the entry record is the audit atom (sibling §4.6, §8.4), and the reversal chain original → reversal → re-post must be reconstructible by a regulator (§3.4).

---

## 2. The Posting Rules

The posting rule is the engine's core configuration: *for transaction type T on product P for event E, debit account X and credit account Y for amount A* (sibling §4.3). This section mechanizes that sentence into the three artifacts the engine actually executes: the **debit/credit determination matrix** (which side, economically), the **account-determination rules** (which account, concretely), and the **GL-string generation** (which COA codes, in which order).

### 2.1 The Debit/Credit Determination Matrix

The first decision is *side*: for a given product and event, does the money move DR or CR on each leg? The side is never arbitrary — it follows from the account's place in the accounting equation and the DEAD CLIC rules (verified canon, sibling §1.3, §1.6): **Debit** increases Expenses, Assets, and Drawings; **Credit** increases Liabilities, Income, and Capital. The matrix below is the *banking translation* of DEAD CLIC: it maps every common product × event to the account that moves and the side it moves on. The rows are verified against the canonical treatments documented in the sibling (§1.6, §6.3, §9) and the open-source reference implementation of the corpus ([apache_fineract_guide.md](apache_fineract_guide.md) — the loan/deposit accounting entries; [fineract_database_models_guide.md](fineract_database_models_guide.md) — the `acc_gl_journal_entry` shape).

The three-column logic that generates every row:

```text
1. Identify the accounts the event touches: a customer account, a bank GL, or both (sibling §4.3 rule types).
2. Classify each account: customer DDA/CASA = liability (credit-normal); loan = asset (debit-normal);
   fee/interest income = revenue (credit-normal); interest expense = expense (debit-normal); suspense = control asset/liability.
3. Apply the economic direction: money into a customer account = CR (liability up); money out = DR (liability down);
   income earned = CR income GL; cost incurred = DR expense GL.
```

| Product | Event | Account that moves | Side | Why (the economic logic) |
|---|---|---|---|---|
| Savings / current (DDA) | Customer deposit (cash in) | Customer DDA | **CR** | The bank's liability to the customer increases |
| Savings / current (DDA) | Withdrawal / payment out | Customer DDA | **DR** | The bank's liability decreases — the customer's money leaves |
| Savings / current (DDA) | Transfer out | Payer DDA | **DR** | Money leaves the payer's account |
| Savings / current (DDA) | Transfer in | Payee DDA | **CR** | Money arrives at the payee's account |
| Savings / current (DDA) | Interest capitalization | Customer DDA | **CR** | The bank owes more (interest credited); the expense side is DR interest expense |
| Loan | Disbursement | Customer DDA | **CR** | Proceeds credited to the customer |
| Loan | Disbursement | Loan GL (asset) | **DR** | The bank's receivable increases |
| Loan | Repayment — principal | Loan GL (asset) | **CR** | The receivable decreases |
| Loan | Repayment — interest | Interest income GL | **CR** | Income earned (CLIC) |
| Loan | Repayment — total | Customer DDA | **DR** | The customer pays |
| Loan | Fee / charges | Fee income GL | **CR** | Income earned |
| Fee | Any fee levied | Customer DDA | **DR** | The customer pays; the fee is never a credit to the customer |
| Interest | Daily accrual (deposit) | Interest expense GL | **DR** | Cost incurred; offset CR accrued interest payable (liability up) |
| Interest | Daily accrual (loan) | Interest income GL | **CR** | Income earned; offset DR accrued interest receivable (asset up) |
| FX | Customer buys FX | Customer DDA (domestic) | **DR** | Domestic money leaves |
| FX | Customer buys FX | Customer FX account | **CR** | Foreign money arrives |
| FX | Bank's spread | FX income GL | **CR** | The spread is income (CLIC) |
| Suspense | Unmatched inward credit | Suspense GL | **CR** | The liability to an unknown beneficiary is parked |
| Suspense | Suspense resolution | Suspense GL | **DR** | The parked liability clears to its true account |
| Inter-branch | Cross-branch transfer | Inter-branch clearing GL | **DR / CR** | Per-branch books; the clearing nets to zero bank-wide |

**Verified, with one mirror-trap reminder** (sibling §1.6): every row above is the *bank's* bookkeeping view. "Debit the customer" and "the customer loses money" are the same sentence because customer DDA accounts are liabilities to the bank — the single most common source of mis-posted rules, which is why the matrix is product-configured and reviewed, never written ad hoc.

### 2.2 The Account-Determination Rules

The matrix decides *side*; the **account-determination** rules decide *which concrete account* each abstract leg lands on. The sibling documents the resolver's precedence chain (sibling §4.4: explicit account on the transaction → product rule → category/COA mapping → suspense). The mechanics layer adds the *determination grammar* — the ordered rule set the resolver applies per leg, each rule with its own precedence and its own failure mode:

```text
For each leg of the rule, resolve the account in this order:
1. Explicit account on the transaction     (payer/payee/loan account) → if invalid: fail validation (sibling §3.2)
2. Product-configured GL                   (this product's fee/interest GL) → if unconfigured: fail fast, never default
3. Category / COA mapping                  (Temenos category GL, sibling §1.7; FLEXCUBE STTM_TRN_CODE, sibling §7.2)
   → if unmapped: post to the designated suspense GL and alert
4. Branch / entity dimension               (per-branch GL books; the inter-branch clearing leg, §9 P9)
5. Currency dimension                      (currency-specific GL accounts; §6)
6. Suspense                               (the explicit, alerted fallback — never a silent default, sibling §6.4)
```

The mechanical rules that make this safe:

- **Fail-fast beats defaulting.** An unresolvable debit leg is a validation failure (the transaction rejects); an unresolvable *credit* leg in an inward flow may legitimately be a suspense case (the beneficiary is genuinely unknown — §4.1). The distinction is configurable per leg direction — the difference between a rejected transfer and a parked one.
- **One leg, one rule, no ambiguity.** Every leg resolves through exactly one path in the precedence chain; the resolver returns the *first matching* rule and records which rule it used on the entry (the audit atom carries the rule ID — sibling §4.6).
- **Determinism is a testable property.** The same transaction replayed against the same configuration resolves the same accounts — the golden-test assertion of §10.2, and the property that makes rule migration safe (sibling §8.9 parallel runs).
- **Customer legs resolve to sub-ledger accounts, GL legs to COA accounts.** A customer DDA leg updates the account's balance components *and* aggregates to its control GL (sibling §5.1, §1.8); a GL leg updates the GL account directly. The resolver must know which kind of account each leg produces — the seam the daily reconciliation checks (§8).

### 2.3 The GL-String Generation

The **GL string** is the full accounting identity of a posting leg, assembled by the engine and stored on the entry record (sibling §4.6 — the entry record fields). It is the mechanics-level answer to *"what does this leg say, completely?"* — the string a downstream GL, a reporting system, or an auditor can join on without re-deriving anything:

```text
GL string =  COA account  +  account type / normal side  +  currency
          +  branch / entity  +  product line  +  transaction code
          +  value date  +  booking date  +  amount + DR/CR sign
          +  reference (transaction ID)  +  reversal reference  +  rule ID
```

Two families of GL strings, per the sibling's GL architecture (§4.7, §6):

- **Per-currency GL strings** — FLEXCUBE-style explicit COA with per-branch, per-currency GL books: the string carries the currency as a first-class component (`1210.LOAN.SGD.BR01`), and each currency's books balance independently (§6.3).
- **Category-driven strings** — Temenos-style: the account's `CATEGORY` maps to the GL account, and the entry pair `STMT.ENTRY`/`CATEG.ENTRY` carries the accounting identity ([temenos_data_model_guide.md](temenos_data_model_guide.md) §5.3; sibling §7.1). The *string* is then the category path plus the entry pair, not a COA code.

The mechanics rules for GL-string generation:

1. **Every leg gets a complete string** — no leg may post with a null product line, currency, or transaction code; the entry log's queryability for statements, regulatory returns, and the reconciliation depends on it (sibling §6.7 — the data-lineage requirement).
2. **The transaction code classifies the leg** — FLEXCUBE's `STTM_TRN_CODE` drives GL mapping and reporting (sibling §7.2); the reversal leg carries a *reversal* transaction code so the string self-describes (§3).
3. **The string is assembled, not guessed** — generation is a pure function of (rule, resolved accounts, amounts, dates); the golden tests assert exact strings for exact inputs (§10.1).

### 2.4 The Rules Table

| Rule artifact | What it decides | Where it lives | Failure mode if wrong | Verification |
|---|---|---|---|---|
| **Debit/credit matrix** | Which side each leg moves on (§2.1) | Product configuration, per product × event | Wrong-but-balanced entries; P&L misstatement | Golden tests per product × event (§10) |
| **Account determination** | Which concrete account each leg hits (§2.2) | Resolver precedence chain + COA master | Money lands in the wrong account; suspense overflow | Replay determinism tests (§10.2) |
| **GL-string generation** | The complete accounting identity of each leg (§2.3) | Entry engine, per leg | Broken lineage; regulatory join failures | Exact-string golden tests (§10.1) |
| **Rule versioning** | Which rule version applied on which date | Rule history (append-only, effective-dated) | Retroactive misstatement; unreproducible history | Recompute/audit mode (§10.4) |
| **Rule governance** | Who changes rules, with what control | Maker-checker on rule changes, effective dates (sibling §3.3, §4.3) | Unreviewed rule change misposts at scale | Change audit trail; parallel runs (§10.3) |

---

## 3. The Reversal Mechanics

A reversal is the *undo of a posting* — a new posting, never a deletion (sibling §3.7; the no-delete rule of sibling §8.4.3; Pacioli's "do not erase" of sibling §1.2). The mechanics layer governs *how* the undo is engineered: which of the reversal variants applies, what the reversing entries look like, how mid-period reversals interact with the interest engine, and what the regulators require of the trail.

### 3.1 The Storno Variants: Direct Reversal vs Reversing Entry

The accounting literature and the vendor implementations distinguish two families of reversal mechanics — the **storno** (also *Stornierung*, the German-origin term for cancelling a booked entry by posting an equal and opposite reversal instead of deleting it) and the **reversing entry** (a scheduled automatic reversal, typically of an accrual or an adjustment, at the start of the next period). Verified: Microsoft's Dynamics 365 localization documentation defines storno accounting as using *negative debit or credit amounts* to reverse original journal account entries, noting that bookkeepers traditionally wrote storno entries in red ink (hence *red storno*); the practice keeps the original and the reversal both on record — the audit-trail property the sibling's §3.7 demands. The variants in a banking posting engine:

| Variant | Mechanics | Value dating | When used | Net effect |
|---|---|---|---|---|
| **Same-day storno (STORN)** | Mirror legs, opposite signs, same value date, linked to the original (sibling §3.7) | Same value date as the original | Error caught the same day (wrong amount, wrong account, duplicate) | The day closes as if the original never economically happened; both entries remain on record |
| **Direct reversal / negative posting** | A *new* entry with negative amounts against the same accounts, with its own reversal transaction code | Today's date (or a back value date) — the original day's books stand | The same-day window has passed; the original day's books must not be rewritten | The customer's balance is correct from the reversal's value date onward; both days' books stand |
| **Reversing entry (auto-reversal)** | A pre-scheduled opposite entry that posts automatically at a defined date — the classic accrual pattern | Next period's first day (or the defined reversal date) | Accruals, prepaids, and adjustments that must self-reverse in the next period | The temporary entry (accrual) nets to zero when the cash posting lands |
| **Correcting entry** | A *new* business entry that fixes the error (S$2 fee → S$5 fee posts a S$3 delta), with a correction narrative | Today | The original was wrong-but-processed and a delta is cleaner than a full undo | The books are right going forward; original + correction both stand |

The mechanical rule: **the reversal is the mirror image of the posting, leg for leg, and it must itself balance** — a reversal of a balanced transaction has Σ DR = Σ CR by construction (each original DR becomes a CR, each CR a DR, same amounts), and the golden tests assert this property (§10.2). The reversal references the original (reversal reference, reason, authorizer, timestamp — sibling §3.7) and the original is flagged as reversed, so the chain original → reversal → (re-post) is reconstructible in one query.

### 3.2 Reversal of the Attached Postings: Fees, Interest, and the Multi-Leg Chain

A transaction rarely posts alone: a transfer carries a fee (§9 P6 reverses both), a disbursement carries fees, a capture carries interchange. The mechanics rule: **reversing a transaction reverses its attached postings as a unit** — the reversal mirrors *every* leg of the original posting unit, including the fee legs (CR the customer, DR the fee income GL back) and any interest legs (the interest engine re-runs the value-dated arithmetic — §3.3). A partial reversal is a deliberate, configured business decision — never an engine default. The sibling's §9.6 shows the four-leg mirror; §9 P6 re-derives it.

### 3.3 Mid-Period Reversals: The Interest-Engine Interaction

The most delicate reversal class is the **mid-period reversal** — undoing a posting whose value date falls *inside an open accrual period*. The interest engine accrues on value-dated balances ([interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §2.3, §2.6 of the posting sibling): a back-dated reversal changes the value-dated balance history for days already accrued, so the interest engine must re-run the affected days. The mechanics:

1. **The posting engine reverses the money movement** — mirror legs on the reversal's value date (a same-day storno touches no accrual; a back-dated reversal does).
2. **The interest engine re-accrues the affected window** — the value-dated balance history is recomputed from the immutable entry stream ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §8.4 — the balance history is append-only; the recompute is a pure function), and the delta between what was accrued and what should have been accrued posts as an **accrual correction** (DR/CR the difference on the interest expense/income GL with the offset on the accrual balance).
3. **The accrual correction itself is a posting** — it goes through the same rules, balancing, and audit as any other; the correction references the original accrual postings it adjusts.

The discipline this enforces: **reversals are value-dated events, not booking-dated events.** The interest engine's own reversal pattern (reversing a mis-posted accrual — §9 P7 below) uses the same mirror mechanics, and the interest guide's precision policy — accrue unrounded, round at posting ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §6.4) — is what makes the correction exact rather than drift-ridden.

### 3.4 The Regulatory Aspects (flagged)

The reversal trail is a regulatory artifact, not an engineering convenience:

- **Record retention.** MAS Notice 626 (AML/CFT for banks) requires banks to keep transaction records — which includes the posting and reversal stream — for at least five years (verified: multiple MAS 626 summaries confirm the five-year retention; the notice itself is issued under the Banking Act and the Financial Services and Markets Act 2022). The general books-and-records obligation for Singapore banks sits in the Financial Services and Markets Act 2022 (the Banking Act's s.43 is the *inspection* power and cross-references FSMA 2022 s.169 for books and records — **flagged**: the FSMA provision text itself was not re-read this session; the existence of the cross-reference was verified on Singapore Statutes Online). The architectural consequence: the entry log and its reversal links must survive five-plus years, queryable and replayable (sibling §6.7, §8.5).
- **The no-delete rule.** "Delete" of an entry is not merely bad practice — it is the destruction of a record the bank is required to keep. The reversal chain is the only legal way to undo (sibling §8.4.3).
- **Maker-checker on reversals.** High-value and out-of-pattern reversals require dual control (sibling §3.3); the reversal record carries the checker's identity like any other entry (**flagged**: specific MAS text on reversal authorization was not isolated this session; the expectation is corpus-wide consensus).
- **Aged-reversal hygiene.** Regulators and auditors examine stale reversal queues the way they examine aged suspense (§4.3): a reversal raised but never authorized, or a reversal orphaned from its original, is a control finding waiting to happen.

### 3.5 The Reversal Table

| Reversal aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Storno definition | Equal-and-opposite cancellation posting; original + reversal both on record; "red storno" in the literature | ✅ Microsoft Learn (Dynamics 365 storno), accounting practice | §3.1; sibling §3.7 |
| Same-day storno | Mirror legs, same value date, net-zero day | ✅ Canonical double-entry; sibling §9.6 worked | §3.1; sibling §9.6 |
| Direct reversal / negative posting | New dated entry, negative amounts, reversal transaction code | ✅ Standard practice (consensus) | §3.1 |
| Reversing entry (auto-reversal) | Scheduled opposite entry, next-period accrual pattern | ✅ Standard accrual-accounting practice | §3.1 |
| Mid-period reversal | Value-dated re-run by the interest engine; accrual correction posting | ✅ Mechanics follow from the interest guides' value-dating discipline | §3.3; [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §8.4 |
| Reversal balances by construction | Mirror of a balanced transaction is balanced | ✅ Arithmetic property | §3.1; §10.2 |
| Five-year record retention | Entry + reversal stream retained and replayable | ✅ MAS Notice 626 (five-year retention, verified) | §3.4; §15 |
| FSMA 2022 books-and-records section | s.169, as cross-referenced by Banking Act s.43(3) | ⚠️ Cross-reference verified; provision text not re-read | §3.4; §15 |

---

## 4. The Suspense and the Exceptions

Suspense is where the engine parks what it cannot resolve — and the mechanics of *parking, aging, and clearing* determine whether the bank's books stay clean or quietly rot. The sibling defines the suspense GL and the clearing account (sibling §6.4) and the failure path (sibling §3.8); this section adds the choreography: what exactly lands in suspense, how the repair queues work, and how unmatched items and breaks are engineered to be visible, owned, and temporary.

### 4.1 The Suspense Accounts

**Verified** (accounting literature, e.g. the Wikipedia treatment): a **suspense account** is an account used temporarily to carry doubtful entries and discrepancies pending their analysis and permanent classification; it is used when the proper account cannot be determined at the time the transaction is recorded, and it should be cleared because it is for temporary use — suspense accounts are a control risk. In a core banking engine the suspense account is a *real GL account* (typically an asset or liability control account — sibling §6.4), and every suspense posting is itself a balanced double-entry posting:

- **Unmatched inward credit** — a FAST/SWIFT/MEPS+ credit arrives with a beneficiary that cannot be resolved (closed account, wrong number, missing reference): the engine credits the suspense GL rather than failing the inbound payment (the money has already arrived — it cannot be "rejected" back to the sender without a return message). §9 P8 works this end to end.
- **Unresolvable GL leg** — a rule whose category mapping is missing (sibling §4.4, §2.2 of this guide): the leg posts to the designated suspense GL *and raises an alert*; it is never silently defaulted.
- **Reconciliation difference** — the day-end sub-ledger-vs-GL check finds a residual difference (§8.3); the difference posts to suspense with an owner until the break is explained.
- **Breaks the engine cannot classify** — an entry whose counterparty leg failed to build (a system-level anomaly): the whole posting rolls back atomically (sibling §8.2.1), and the *event* lands in the repair queue (§4.2), not the balances.

The mechanics rules:

1. **Suspense is per-direction and per-currency.** A credit-suspense (we owe an unknown party) and a debit-suspense (we are owed by an unknown party) are different accounts in the COA; mixing them obscures the aging picture.
2. **Every suspense entry has an owner and a reason.** The entry record carries the suspense reason code, the source reference, and the owning operations team; a suspense posting with no owner is itself a control finding.
3. **Suspense ages and is reported.** The daily aging report (0–1 day, 1–7, 7–30, 30+) is a standing control; old suspense is a red flag that hides errors, fraud, and failed postings (sibling §6.4 — the MAS/auditor expectation). The mechanics target: **suspense balances are small, aged, and owned — and the clearing rate is a monitored metric.**

### 4.2 The Repair Queues

The **repair queue** (the exception queue of sibling §3.8, given its mechanics here) is where *failed or unpostable work* waits for human or automated repair. It is distinct from suspense: **suspense holds money that posted to a holding account; the repair queue holds work that did not post at all.** The queue items:

- **Rejected postings with retry metadata** — a batch posting whose GL was unavailable, a payment whose account was frozen mid-flight, a rule that failed to resolve (sibling §3.8). Each item carries the payload, the error, the idempotency key (so the retry cannot double-post — §7), and the retry policy.
- **The repair workflow** — operations inspects, then *re-drives* (safe because of the key), *corrects and re-posts* (a new transaction, a new key), or *cancels* (nothing posted — nothing to reverse). The queue has SLAs: MAS expects operational resilience around exactly these queues (sibling §3.8).
- **Queue hygiene mechanics** — items age to an aged-exceptions report; an item older than its SLA is an incident, not an inbox; every lifecycle step (created → assigned → repaired → closed) is audit-logged.

The mechanics rule that makes repair safe: **a repair is either a replay (same key, §7) or a new transaction (new key) — never a manual SQL poke at the balances.** Manual balance surgery is how posting engines become unreconcilable (§8.4).

### 4.3 The Unmatched Items and the Breaks (flagged)

- **Unmatched items** — inbound credits with no beneficiary, outbound debits with no confirmation, nostro movements with no trade. The matching engine (§8) pairs them against the transaction stream; an item that survives matching lands in suspense (§4.1) and its investigation is a work item, not a mystery.
- **The break** — the residual difference after reconciliation: the sub-ledger total minus the GL balance that is not explained by any known item (§8.3). A break is *always* investigated; the mechanics distinction is between an **explained break** (a rounding residual that belongs in the rounding reserve GL — §5.4; or a known timing difference that clears) and an **unexplained break** (a defect signal — a lost entry, a double post, a balance-update bug). The rules: breaks post to suspense with an owner (§4.1); unexplained breaks escalate; a *growing* break is an incident.
- **Flagged:** the precise MAS expectation on break thresholds (a percentage-of-balance tolerance or a hard dollar amount) is **not a published bright-line number this session could verify** — MAS expects reconciliation controls to be proportionate and documented (corpus-wide consensus via [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md) and the sibling's §6.2), but the specific tolerance is a bank-defined control parameter subject to examiner review. Treat "the regulator requires a documented tolerance" as verified and "the tolerance value" as bank-configured.

### 4.4 The Suspense Table

| Suspense aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Suspense definition | Temporary GL for doubtful entries pending analysis; cleared when classified; a control risk | [OK] Accounting literature | §4.1; sibling §6.4 |
| What posts to suspense | Unmatched credits, unresolvable GL legs, reconciliation residuals | [OK] Canonical practice | §4.1; §9 P8 |
| Suspense postings balance | Every suspense posting is a normal balanced DR/CR pair | [OK] Double-entry by construction | §4.1 |
| Repair queue | Failed/unpostable work, with retry metadata and idempotency keys | [OK] Consensus + sibling §3.8 | §4.2 |
| Aged-suspense hygiene | Aging reports, ownership, clearing-rate monitoring | [OK] MAS/audit expectation (consensus, sibling §6.4) | §4.1, §4.3 |
| Break tolerance values | Bank-defined, documented, examiner-reviewed | [FLAGGED] — no published bright line verified | §4.3, §15 |

---

## 5. The Posting Precision

The posting engine moves money, and money arithmetic is exact arithmetic. This section is the direct mirror of the interest guide's precision engineering ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §6): the same ban on binary floats, the same decimal discipline, the same rounding-method taxonomy — applied to the *balancing* problem, which is the posting engine's unique precision challenge.

### 5.1 The Decimal-vs-Float Ban

**Verified** (IEEE 754 arithmetic; the standard, well-documented property): binary floating point cannot represent 0.01 exactly — 0.01 is an infinite binary fraction (0.00000010100011110101110000…2), so every float arithmetic on money carries a representation error. In an interest engine the error accumulates into drift across millions of accounts × days ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §6.1); in a posting engine the error is worse — it threatens the *balancing invariant itself*: a float-computed Σ DR can differ from Σ CR by 1e-16, and the engine then has to decide whether to reject a transaction that is "mathematically balanced but float-unbalanced", or post an unbalanced one. Both are unacceptable. The rules:

- **Amounts are decimal or scaled integers, never floats.** Java `BigDecimal`, C# `decimal`, Python `decimal`, SQL `NUMERIC(38,x)` — or integer minor units (S$1.23 = 123 cents; JPY 123 = 123 yen — ISO 4217 minor units, §5.5).
- **Every division specifies scale and rounding mode.** A `BigDecimal.divide` without a scale has shipped real bugs; the engine's arithmetic helpers require (scale, mode) on every divide.
- **Balancing is an exact comparison.** Σ DR = Σ CR is compared on the *posted, rounded* amounts, not unrounded intermediates — the entry engine rounds each leg per its currency (§5.5), then checks the rounded sum; a transaction failing the rounded-sum check is rejected at build time (sibling §8.2.3).

### 5.2 The Rounding Modes

The same taxonomy the interest guide documents (§6.3), applied to postings. The method is a **product parameter** — configured, disclosed, and golden-tested, never an implementation accident:

| Mode | Tie rule | Posting-engine usage |
|---|---|---|
| **Round-half-up** | Ties round away from zero (2.5 → 3; 16.435 → 16.44) | The retail norm for customer-visible postings; simple to explain to customers and regulators |
| **Round-half-even (banker's)** | Ties round to the nearest even digit (2.5 → 2; 16.445 → 16.44) | Statistical/analytical contexts; default of IEEE 754; unbiased over uniform data |
| **Truncation / floor** | Always round down | Some fee calculations; never customer-visible deposits |
| **Ceiling** | Always round up | Rare; tax and some fee contexts |

Re-verified with Python decimal during the writing of this guide: 16.435 → 16.44 half-up; 16.445 → 16.45 half-up vs **16.44** half-even; 2.5 → 2 half-even. The mixing rule from the interest guide applies verbatim: a bank that rounds savings half-up and fees half-even is defensible; a bank that mixes methods *silently* is not.

### 5.3 The Balancing Arithmetic

The posting engine's balancing arithmetic has three layers, each independently enforced (sibling §8.2.3 names the same layers; the mechanics are here):

```text
Layer 1 — Per transaction:   Σ DR = Σ CR  on the posted, rounded legs (reject at build time)
Layer 2 — Per book per day:  trial balance = 0  (every GL account's debits and credits net to zero bank-wide)
Layer 3 — Per control account: Σ sub-ledger balances = GL balance  (the reconciliation, §8)
```

Two arithmetic facts the engine depends on:

1. **Rounding is additive only if the policy says so.** Round each of N legs, sum, and compare to rounding the sum once — the results can differ by a cent. The engine therefore has a defined **rounding order**: legs round individually at posting (the customer sees rounded legs), and the *balancing check* runs on the rounded values. Residual differences are not errors if they land in the rounding reserve (§5.4).
2. **The cent is not the only unit.** JPY posts in whole yen, BHD/KWD in three decimals, CLF in four (ISO 4217 minor units — verified against the standard, §5.5). "Round to 2 decimals" is a bug for JPY; the rounding scale comes from the leg's currency, never a global constant.

### 5.4 The Rounding Reserve

The **rounding reserve** (the interest guide's §6.5 concept, applied to postings) is the GL account that absorbs the difference between "Σ per-leg rounded postings" and "the unrounded economic amount". It is *expected and bounded*: the classic FX case — a conversion whose unrounded USD amount (739.644970…) rounds to 739.64, leaving a residual of 0.004970 USD ≈ S$0.0067 that must go *somewhere* balanced — §9 P12 posts it to the reserve. The rules: the reserve is a real, named GL; it reconciles monthly; a *growing* reserve is a rounding-policy defect, not a fact of life (interest guide §6.5 — the S$0.06-per-account-month example scales to S$0.72M/year at a million accounts; the same economics apply to posting rounding).

### 5.5 Per-Currency Precision

ISO 4217 defines the minor unit per currency (**verified** against the standard's minor-unit tables): SGD/USD/EUR 2 decimals, JPY 0, BHD/KWD 3, CLF 4. The posting engine's amount type is currency-aware: each leg rounds at its currency's scale (§5.1); FX-converted legs round at each currency boundary per that currency's rules (§6.3); balances may carry higher internal precision than display precision, as long as the display rounding is the *defined* one and internal precision never leaks into customer-visible postings (interest guide §6.6 — same discipline).

### 5.6 The Precision Table

| Precision aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Float ban | Money arithmetic in decimal/scaled integers; floats never touch balances | [OK] IEEE 754 property (0.01 inexact) | §5.1; [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §6.1 |
| Rounding modes | Half-up / half-even / floor / ceiling; method is a product parameter | [OK] Re-computed with Python decimal (16.435, 16.445, 2.5 ties) | §5.2 |
| Balancing layers | Per-transaction, per-day trial balance, per-control-account | [OK] Arithmetic invariant | §5.3; sibling §8.2.3 |
| Rounding order | Legs round at posting; balance check on rounded values | [OK] Consensus (defined-policy requirement) | §5.3 |
| Rounding reserve | GL absorbing Σ rounded vs unrounded differences; bounded and reconciled | [OK] Mirror of interest guide §6.5 | §5.4; §9 P12 |
| Per-currency scale | ISO 4217 minor units (SGD 2, JPY 0, BHD/KWD 3) | [OK] ISO 4217 | §5.5 |

---

## 6. The Multi-Currency Postings

The sibling's §4.10 establishes the multi-currency shape (conversion legs, the FX difference GL, EOD revaluation, round-off). This section is the mechanics: how the FX legs are *constructed* so both the transaction currency and the base currency balance, how the rate is captured so the entry is reproducible, and how the round trip proves the arithmetic.

### 6.1 The FX Conversions: Leg Construction

The canonical customer FX posting — a customer buys USD with SGD — produces legs in **two currencies plus a spread leg**, and the balancing discipline is *per currency* (verified by construction; re-computed in §9 P5):

```text
SGD book (transaction currency):  DR Customer SGD DDA   S$1,000.00
                                  CR FX clearing (SGD)  S$1,000.00        ← SGD book balances
USD book (foreign currency):      DR FX clearing (USD)  USD 739.64
                                  CR Customer USD DDA   USD 739.64        ← USD book balances
Conversion (base-currency view):  the USD clearing revalued at the day's mid 1.3500 → S$998.51
Spread (the bank's income):       S$1,000.00 − S$998.51 = S$1.49 → CR FX income GL
                                  (DR the SGD clearing by 1.49 so the two clearings net to zero)
```

The mechanics rules:

1. **Each currency's book balances independently.** The customer legs must balance in USD and in SGD separately — a conversion that balances only in the base currency is broken (it would leave an unmatched USD position). The FX clearing accounts are the vehicle: every cross-currency posting moves through them, and they net to zero at the day's rates.
2. **The customer receives the rounded amount.** USD 739.644970… pays out as **USD 739.64** (2 dp, half-up); the residual is not the customer's problem — it is the engine's rounding-reserve item (§5.4, §9 P12).
3. **The spread is income, booked to the FX income GL** — never embedded silently in the rate (the rate *displayed* to the customer is the sell rate 1.3520; the *accounting* rate is the mid 1.3500; the difference is the spread). The two rates must both be recorded on the entry — that is the rate capture (§6.2).
4. **Cross-branch and cross-currency together** compose the two mechanisms (inter-branch clearing legs of §9 P9 + the FX legs of §9 P5) — the engine builds them from the same rule grammar.

### 6.2 The Rate Capture (flagged)

**Rate capture** is the recording, on every FX entry, of the *exact rates and the rate source* used to convert — the FX mirror of the interest engine's "store the resolved rate, not the configuration" rule ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §7.2). The captured rate record carries: the mid rate, the customer buy/sell rate, the rate source (the rate service feed, the dealer's quote, the channel's displayed rate), the rate timestamp, and the rate-set version. Without it, an FX posting cannot be re-derived — the auditor's canonical question ("why 739.64 USD for S$1,000?") is unanswerable, and the EOD revaluation (§9 P11) cannot be reconciled.

**Flagged:** the *precise* rate-capture schema is engineering consensus rather than a documented standard — every core implements it (Temenos currency/rate tables, FLEXCUBE's rate master), but the exact field set is vendor- and bank-specific; the *requirement* (rate capture makes FX postings audit-replayable — sibling §4.10, §6.7) is not consensus but necessity. Also flagged: which rate (mid vs settlement) converts to base currency is an accounting-policy decision per product, recorded in the product's rule configuration.

### 6.3 The Round-Trip Balancing

The **round trip** is the self-check that proves a currency pair's arithmetic: convert an amount out and back at the same rates and the round trip returns to the starting balance *within the defined rounding policy* — the residual is exactly the double-spread and the double-rounding, and both must be accountable:

```text
S$1,000.00 → USD 739.64 (sell 1.3520) → back at buy rate 1.3460 → S$995.56
Round-trip residual: 1,000.00 − 995.56 = S$4.44 = the bank's two-sided spread + rounding
(verified by re-computation in §9 P14: 739.64 × 1.3460 = 995.55544… → 995.56 half-up;
 the two spreads — 1.49 outbound + 2.95 inbound — sum to exactly 4.44)
```

The round trip is not a business event — it is a **test invariant** (§10.2): the golden-test library converts out and back and asserts (a) both books balance at every step, (b) the residual equals the documented spread plus the rounding reserve delta, and (c) replaying the same rates produces the identical residual. A round trip whose residual is *not* the documented spread is a rate-capture or leg-construction defect.

### 6.4 The FX Table

| FX aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Per-currency balancing | Each currency's book balances via FX clearing accounts | [OK] Construction + §9 P5 re-computation | §6.1; §9 P5 |
| Customer rounding | Payable amount rounded at the foreign currency's scale | [OK] §9 P5 re-computation | §6.1 |
| Spread booking | Mid for accounting, sell/buy for customer; difference to FX income GL | [OK] Construction (consensus) | §6.1 |
| Rate capture | Mid, customer rate, source, timestamp, rate-set version on every FX entry | [FLAGGED] — requirement is consensus; exact schema is vendor-specific | §6.2, §15 |
| Round trip | Convert out and back; residual = double spread + rounding; a test invariant | [OK] Re-computed (§9 P14) | §6.3; §10.2 |
| EOD revaluation | Revalue open positions at the day's closing rates | [OK] §9 P11 re-computation; sibling §4.10, §6.5 | §9 P11 |

---

## 7. The Idempotency

Idempotency is the property that **the same transaction posts exactly once, even if the request arrives twice** — the sibling's §8.2.2 states it and names the failure ("a payments-hub retry double-debits the customer — the most expensive bug class in core banking"). This section is the mechanics: what the key is, how duplicate detection works, and why replay is safe.

### 7.1 The Idempotency Keys (flagged)

The **idempotency key** is the unique identifier the engine deduplicates on — assigned at initiation (sibling §3.1): the channel reference, the payment's reference/UETR, or an engine-generated ID. The mechanics:

- **Scope.** The key must be unique across the *posting unit* — one transaction, one key, all its legs. A key per leg would let a retry re-post half a transaction.
- **Composition.** A good key is *client-supplied, stable across retries, and unique across time*: the payments hub's transaction reference, the ISO 20022 message identifier ([iso_20022_core_processes_guide.md](iso_20022_core_processes_guide.md)), or a UUID minted at initiation and carried through the retry. **Flagged:** the exact key schema is implementation-defined — Stripe's documented pattern is an `Idempotency-Key` header the server keys its result cache on (verified: same key, same result, no double execution), and banking engines mirror it with transaction references; there is no banking-standard schema, so the composition rule (client reference + channel + date-scope) is engineering consensus.
- **Uniqueness enforcement.** The key is a unique constraint on the entry log — the database rejects a second insert with the same key, which is the atomic guarantee (the check and the insert are one transaction, so two racing retries cannot both pass the check — the same per-account serialization of sibling §4.5.1).

### 7.2 The Duplicate Detection

Two layers of duplicate detection, defense in depth:

1. **The idempotency key (exact match)** — the primary control: a retry with the same key finds the original posting and *returns it* (the channel gets the original confirmation, not a new posting). This is exactly-once semantics at the entry level.
2. **Heuristic duplicate checks (fuzzy match)** — the defensive layer for *different keys, same economic event*: Temenos's classic duplicate detection — the same debit account and amount within a window — flags a probable duplicate for review (sibling §7.1, verified against Temenos training material in the sibling). Card-scheme duplicate checks (same card, amount, merchant, timestamp) are the same pattern. Heuristic hits are *review items*, not automatic rejections — a legitimate same-account-same-amount transfer (a customer paying the same bill twice on purpose) must survive.

The mechanics rule: **the key layer is authoritative; the heuristic layer is advisory.** Only the key can *block*; heuristics only *flag*.

### 7.3 The Replay Safety

**Replay** is re-executing a posting — the repair-queue re-drive (§4.2), the batch restart, the EOD re-run, the audit recompute (§10.4). Replay is safe exactly when idempotency holds:

- **Replaying a succeeded posting returns the original result** — the key lookup finds the original entries; nothing re-posts (the batch-restart pattern of the interest run, [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §8.3, depends on this).
- **Replaying a failed posting re-executes cleanly** — the failed attempt left no entries (atomic rollback, sibling §8.2.1), so the retry is a first execution with the same key.
- **The ambiguous case is handled by the key, not by inspection** — "did it post or not?" after a timeout is answered by looking up the key; the operator never guesses, because a wrong guess either double-posts or loses the money.
- **Replay of a *reversal* is the same machinery** — the reversal carries its own key; a retried reversal instruction cannot reverse twice (§3.1's mirror property protects the second layer).

### 7.4 The Idempotency Table

| Idempotency aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Idempotency key | Unique per posting unit; unique constraint on the entry log | [OK] Concept verified (Stripe idempotency docs; sibling §8.2.2) | §7.1; sibling §8.2.2 |
| Key schema | Client reference + channel + date-scope composition | [FLAGGED] — no banking-standard schema; consensus pattern | §7.1, §15 |
| Duplicate detection | Key layer authoritative; heuristic layer advisory | [OK] Consensus; Temenos duplicate check in sibling §7.1 | §7.2 |
| Replay safety | Same key → same result; failed attempts left no entries | [OK] Follows from atomicity + keying | §7.3 |
| Exactly-once retries | Retried request returns original confirmation | [OK] Verified against Stripe pattern | §7.1 |

---

## 8. The Reconciliation

Reconciliation is the discipline that turns "the engine ran" into "the books are right" (sibling §6.2 — "the single most important control in core banking"). The mechanics: how the sub-ledger-vs-GL balance is computed, what the control accounts are, and how breaks are found, owned, and corrected.

### 8.1 The Sub-Ledger-vs-GL Balancing

The daily control (sibling §1.9): **Σ (sub-ledger balances for GL account X) = GL balance of account X**, for every control account, every branch, every currency. The mechanics of the run:

```text
1. Aggregate  — sum sub-ledger balances (or day deltas) per control GL, branch, currency
2. Compare    — against GL balances (real-time GL: continuous; batch GL: at the EOD GL close, sibling §4.7)
3. Explain    — each difference is a timing item (clearing/in-transit), a rounding residual (§5.4), or a break (§8.3)
4. Post       — explained residuals to their GLs (rounding reserve, clearing); unexplained to suspense, with an owner (§4.1)
5. Sign off   — the reconciled trial balance is the day's accounting proof (sibling §6.6)
```

The verified mechanics anchor: the **control account** is the GL account for which a subsidiary ledger exists — individual transactions are posted both to the control account and the subsidiary ledger, and the totals are compared when preparing a trial balance (verified: accounting literature; the canonical treatment). The sub-ledger is the *source* (sibling §1.8): the GL balance is a derived aggregate, and the reconciliation proves the derivation.

### 8.2 The Control Accounts

Every customer-facing GL with detail behind it is a control account: `2310 Savings Deposits` controls the savings sub-ledger, `1210 Loans and Advances` controls the loan accounts, the nostro GLs control the correspondent balances, the suspense GL controls the parked items (§4.1). The mechanics:

- **Every customer posting touches both sides of the control.** A debit to a customer DDA reduces both the account's balance and its control GL's balance — by the same amount, in the same transaction (sibling §4.6, §8.2.1 — the atomicity that makes the reconciliation *possible*).
- **Control accounts have owners and calendars.** Each control account reconciles on a schedule (daily for deposits and loans, daily-or-T+1 for nostro, weekly for low-volume GLs); an overdue reconciliation is a control failure.
- **The control-account list is configuration.** Adding a product means adding its control accounts to the reconciliation scope — a missed control account is a blind spot the size of the product.

### 8.3 The Breaks and the Corrections (flagged)

- **Finding breaks.** A break is the residual difference after matching and explanation (§4.3). The reconciliation mechanics *size* the break (the amount), *locate* it (which control account, branch, currency), and *date* it (first-seen date) — because the three dimensions are what make it investigable.
- **Correcting breaks.** The correction is a posting, never a balance edit: if a lost entry caused the break, the correction is the *re-posting* of that entry (replay, §7.3 — safe); if a wrong entry caused it, the correction is a reversal plus a re-post (§3); if the break is genuinely unlocatable, it posts to suspense and the investigation continues. The golden rule: **the correction must itself be a balanced, auditable posting** — a "fix" that edits a balance directly creates the next break.
- **Flagged:** the MAS expectation on reconciliation is documented as a *control expectation* (proportionate, documented, evidenced — sibling §6.2, [financial_risk_compliance_systems_guide.md](financial_risk_compliance_systems_guide.md)) rather than a published numeric tolerance; the specific break threshold is bank-defined (§4.3). The sibling's mislabel of MAS Notice 610 as "large exposures" is corrected in the verification ledger (§15) — Notice 610 is the statistics-and-returns notice (verified against mas.gov.sg).

### 8.4 The Reconciliation Table

| Reconciliation aspect | Mechanics | Verified? | Cross-ref |
|---|---|---|---|
| Sub-ledger-vs-GL balance | Σ sub-ledger = GL control balance, per branch/currency, daily | [OK] Canonical + sibling §1.9, §6.2 | §8.1 |
| Control account | GL account with a subsidiary ledger; both posted; totals compared at trial balance | [OK] Accounting literature | §8.2 |
| Control-account scope | Every customer-facing GL with detail is in the reconciliation scope | [OK] Canonical (consensus) | §8.2 |
| Break handling | Sized, located, dated; posted to suspense with an owner | [OK] Consensus + sibling §6.4 | §8.3 |
| Corrections are postings | Re-posting (replay), reversal + re-post, or suspense; never balance edits | [OK] Follows from no-delete + atomicity | §8.3 |
| Break tolerance | Bank-defined, documented, examiner-reviewed | [FLAGGED] — no published bright line | §8.3, §15 |

---

## 9. The Worked Postings

This section is the mirror of the interest guide's worked-examples library ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §10): fully worked journal entries, every amount shown, **every number independently re-computed at 40-digit Python `decimal` precision during the writing of this guide** (round-half-up at the posting currency's scale, ISO 4217 minor units — §5.5). The sibling's §9 works one example (the fund transfer with fee) to exhaustion; this library works the *posting-mechanics* surface: the interest postings, the loan lifecycle, the FX pair, the reversals, the suspense lifecycle, the clearing legs, and the rounding reserve. The discipline is the same as the interest guide's: each posting states the rule it resolves from (§2), the leg set, the balance check, and the balance deltas.

The fictional Cymbal Bank GL skeleton used throughout (COA codes follow the sibling's style, §1.7):

```text
1210  Loans and Advances (asset, debit-normal)          2310  Savings Deposits (liability, credit-normal) — control of the savings sub-ledger
2320  Current Accounts (liability, credit-normal)        2510  Accrued Interest Payable (liability)
2610  FX Clearing (SGD) / 2620 FX Clearing (USD)         2990  Suspense — Unmatched Credits (liability)
4100  Interest Income (revenue, credit-normal)           4120  Fee & Commission Income (revenue)
4200  FX Income (revenue)                                5110  Interest Expense (expense, debit-normal)
5301  Rounding Reserve (liability)                       5310  FX Revaluation Reserve (equity)
6210  Inter-branch Clearing (liability, nets to zero)    6510  Nostro — USD (asset) / 6520 Nostro — SGD (asset)
```

### P1 — Daily Interest Accrual (deposit), 2026-08-07

**Rule:** `INTEREST_ACCRUAL × SAVINGS → DR 5110 Interest Expense / CR 2510 Accrued Interest Payable` (the accrual-recognition pattern of the interest guides: [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §6.2; [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §5). Balance S$10,000.00, rate 2% p.a., ACT/365.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | 5110 Interest Expense | S$0.55 | — |
| 2 | 2510 Accrued Interest Payable | — | S$0.55 |

```text
Unrounded daily accrual: 10,000.00 × 0.02 / 365 = 0.547945205479…   (the accrual sub-ledger carries full precision)
Posted: S$0.55 (half-up) — the interest guide's 'accrue unrounded, round at posting' policy (§6.4)
Balance check: DR 0.55 = CR 0.55  ✓
```

### P2 — Interest Capitalization (31 days), 2026-08-31

**Rule:** `INTEREST_CAPITALIZE × SAVINGS → reverse the accrual, then credit the customer` — two postings on the capitalization event ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §5.4; the interest guide's §10 works the same arithmetic). Accrued for the month: 16.98630136… → **S$16.99**.

| # | Posting | Account | DR | CR |
|---|---|---|---|---|
| 2a | Reverse the month's accrual | 2510 Accrued Interest Payable | S$16.99 | — |
| 2a | | 5110 Interest Expense | — | S$16.99 |
| 2b | Capitalize to the customer | 5110 Interest Expense | S$16.99 | — |
| 2b | | Customer savings account S-1001 (sub-ledger of 2310) | — | S$16.99 |

```text
Unrounded: 10,000.00 × 0.02 × 31/365 = 16.98630136986… → posted S$16.99 (half-up)
Balance deltas: customer account 5,000.00 → 5,016.99; control GL 2310 rises S$16.99; 5110 nets to zero for the month
Balance check per posting: 2a DR 16.99 = CR 16.99 ✓ ; 2b DR 16.99 = CR 16.99 ✓
```

### P3 — Loan Disbursement, 2026-08-10

**Rule:** `LOAN_DISBURSE → DR the loan asset / CR the customer's DDA`. S$50,000.00 to customer L-5001, proceeds to current account C-2002.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | 1210 Loans and Advances (loan account L-5001) | S$50,000.00 | — |
| 2 | Customer current account C-2002 (sub-ledger of 2320) | — | S$50,000.00 |

```text
The loan is an asset to the bank: disbursing it INCREASES the receivable → DR (DEAD). The proceeds land in the
customer's current account → CR (the bank's liability to the customer rises). Balance check: 50,000 = 50,000 ✓
```

### P4 — Disbursement Fee, 2026-08-10

**Rule:** `LOAN_FEE × 0.5% capped S$150 → DR customer / CR fee income`. Unrounded fee 50,000 × 0.005 = S$250.00 → **capped at S$150.00** (the cap is a rule parameter, applied before posting — the mirror of the interest engine's rate/floor clamps).

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | Customer current account C-2002 | S$150.00 | — |
| 2 | 4120 Fee & Commission Income | — | S$150.00 |

```text
Balance check: 150 = 150 ✓. Note the mirror trap (sibling §1.6): the fee CREDITS the income account — income is CLIC.
```

### P5 — FX Conversion: Customer Buys USD, 2026-08-12

**Rule:** `FX_BUY × sell rate 1.3520, accounting mid 1.3500 → DR customer SGD / CR customer USD / CR FX income`, with per-currency balancing through the FX clearing accounts (§6.1). Customer pays S$1,000.00.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | Customer SGD DDA (sub-ledger of 2320) | S$1,000.00 | — |
| 2 | 2610 FX Clearing (SGD) | — | S$1,000.00 |
| 3 | 2620 FX Clearing (USD) | USD 739.64 | — |
| 4 | Customer USD DDA (sub-ledger of the USD GL) | — | USD 739.64 |
| 5 | 4200 FX Income (the spread; also nets the SGD clearing) | — | S$1.49 |

```text
USD payable: 1,000.00 / 1.3520 = 739.644970414… → USD 739.64 (2 dp, half-up — the customer gets the rounded amount)
Mid value of the rounded USD: 739.64 × 1.3500 = 998.514 → S$998.51
Spread: 1,000.00 − 998.51 = S$1.49 → CR 4200 (with DR 2610 by 1.49, leaving the SGD clearing at 998.51 to net
against the USD clearing's 998.51 SGD equivalent; the nostro funding leg settles the bank's USD position)
Per-currency balance: SGD book DR 1,000.00 = CR 1,000.00 ✓ ; USD book DR 739.64 = CR 739.64 ✓ ;
base-currency view 1,000.00 = 998.51 + 1.49 ✓
```

### P6 — Same-Day Storno of a Fund Transfer, 2026-08-07 (corrected 2026-08-07)

**Rule:** `REVERSAL_STORN × FUNDS_TRANSFER → mirror every leg of the original with opposite signs, same value date` (§3.1; the sibling's §9.6 reversal). The original: S$1,000.00 from A to B with a S$2.00 fee (sibling §9.2 — four legs). The storno:

| Leg | Account | DR | CR | References |
|---|---|---|---|---|
| 1R | Customer account A | — | S$1,000.00 | REV of TXN-1001 |
| 2R | Customer account B | S$1,000.00 | — | REV of TXN-1001 |
| 3R | Customer account A | — | S$2.00 | REV of fee leg |
| 4R | 4120 Fee & Commission Income | S$2.00 | — | REV of fee income leg |

```text
Balance check: DR 1,002.00 = CR 1,002.00 ✓ — the mirror of a balanced transaction is balanced by construction (§3.1).
Net effect on the day: A back to 5,000.00, B back to 2,000.00, 4120 back to its opening — while all eight entries
remain on record, linked original ↔ reversal (sibling §3.7 audit trail). Reversal carries its own idempotency key (§7.3).
```

### P7 — Mid-Period Accrual Reversal (rate correction), 2026-08-15

**Rule:** `ACCRUAL_REVERSAL → mirror the accrual postings, then re-accrue at the corrected rate` (§3.3 — the mid-period reversal pattern; the interest guide's recompute discipline, §11.4). The bank discovered it accrued July at 1.5% instead of 2.0% for account S-1001; July's accrual of S$12.74 (10,000 × 0.02 × 15/365 + 10,000 × 0.02 × 16/365… — the pro-rata split is the interest engine's job, [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §4.11) must be reversed and re-booked.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | 2510 Accrued Interest Payable | S$12.74 | — |
| 2 | 5110 Interest Expense | — | S$12.74 |

```text
The reversal is a mirror of the accrual posting (same mechanics as P6); the re-accrual at 2.0% then posts the
correct S$16.99 month (P2's arithmetic). The value-dated balance history and the accrual records are untouched —
the correction is a posting, exactly like any other (§8.3). Balance check: 12.74 = 12.74 ✓
```

### P8 — Suspense Lifecycle: Unmatched Inward Credit, 2026-08-18

**Rule:** `INWARD_CREDIT_UNMATCHED → DR the incoming clearing / CR suspense; on resolution, DR suspense / CR the beneficiary` (§4.1). A FAST credit of S$5,000.00 arrives for a closed account number.

| # | Posting | Account | DR | CR |
|---|---|---|---|---|
| 8a | Park the unmatched credit | 6510 Nostro — SGD (or incoming clearing) | S$5,000.00 | — |
| 8a | | 2990 Suspense — Unmatched Credits | — | S$5,000.00 |
| 8b | Resolve (beneficiary identified) | 2990 Suspense — Unmatched Credits | S$5,000.00 | — |
| 8b | | Customer DDA (the identified beneficiary) | — | S$5,000.00 |

```text
Every suspense posting is a normal balanced pair (8a: 5,000 = 5,000 ✓ ; 8b: 5,000 = 5,000 ✓). The suspense account
is a real GL liability control account; its balance aged 3 days in this case and cleared on resolution — the
target shape: small, aged, owned (§4.1). If the beneficiary never resolves, the item ages into the returns process.
```

### P9 — Cross-Branch Transfer, 2026-08-19

**Rule:** `FUNDS_TRANSFER × CROSS_BRANCH → customer legs plus inter-branch clearing legs, per-branch books` (sibling §4.3; §6.5's EOD). S$1,000.00 from Account A at Branch 01 to Account B at Branch 02.

| Leg | Account (book) | DR | CR |
|---|---|---|---|
| 1 | Customer A (Branch 01) | S$1,000.00 | — |
| 2 | 6210 Inter-branch Clearing (Branch 01 book) | — | S$1,000.00 |
| 3 | 6210 Inter-branch Clearing (Branch 02 book) | S$1,000.00 | — |
| 4 | Customer B (Branch 02) | — | S$1,000.00 |

```text
Branch 01 balances (1,000 = 1,000 ✓); Branch 02 balances (1,000 = 1,000 ✓); the inter-branch clearing account
nets to zero bank-wide (the branch 02 debit offsets the branch 01 credit) — the mechanism the day-end
branch-reconciliation proves (sibling §6.5). Add a fee and FX and the same grammar yields six or eight legs (§9.7 sibling).
```

### P10 — Loan Repayment: EMI Split, 2026-09-10

**Rule:** `LOAN_REPAY → DR the customer for the EMI / CR principal to the loan asset / CR interest to income` (§2.1; the reducing-balance arithmetic of [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §4.5). Loan S$50,000.00 at 5% p.a., 12 months, monthly EMI **S$4,280.37** (re-computed: EMI = P·r·(1+r)ⁿ/((1+r)ⁿ−1) with r = 0.05/12; total interest over the life S$1,364.44). Month 1: interest 208.33, principal 4,072.04.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | Customer current account C-2002 | S$4,280.37 | — |
| 2 | 1210 Loans and Advances (loan L-5001) | — | S$4,072.04 |
| 3 | 4100 Interest Income | — | S$208.33 |

```text
Month-1 interest: 50,000 × 0.05/12 = 208.3333… → S$208.33 ; principal 4,280.37 − 208.33 = S$4,072.04
Balance check: DR 4,280.37 = CR 4,072.04 + CR 208.33 = 4,280.37 ✓
Loan balance after month 1: 50,000 − 4,072.04 = S$45,927.96 (the reducing-balance base for month 2)
```

### P11 — EOD FX Revaluation, 2026-08-31

**Rule:** `REVALUATION × EOD → revalue open FX positions at the day's closing rate` (sibling §4.10, §6.5). USD nostro of USD 100,000.00 carried at 1.3500 (S$135,000.00); the closing rate is 1.3600.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | 6510 Nostro — USD (SGD equivalent uplift) | S$1,000.00 | — |
| 2 | 5310 FX Revaluation Reserve | — | S$1,000.00 |

```text
Revaluation: 100,000 × (1.3600 − 1.3500) = S$1,000.00 — verified: 100,000 × 1.3600 = 136,000.00 vs 135,000.00.
(The credit destination — reserve vs P&L — is an accounting-policy choice per the bank's hedge-accounting policy;
the posting mechanics are identical.) Balance check: 1,000 = 1,000 ✓
```

### P12 — The Rounding Reserve: Aggregate Round-Off, 2026-08-31

**Rule:** `ROUND_OFF × ACCUMULATOR → the aggregated sub-cent residuals post to the rounding reserve` (§5.4). Each FX conversion like P5 leaves an unrounded-vs-rounded residual: 0.0049704142… USD × 1.3500 = S$0.006710059… per transaction — absorbed inside the spread leg per-transaction, but *accumulated* and swept to the reserve. For a month of 1,000,000 such conversions:

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | 4200 FX Income (the rounding accumulator) | S$6,710.06 | — |
| 2 | 5301 Rounding Reserve | — | S$6,710.06 |

```text
Aggregate: 1,000,000 × 0.006710059171… = 6,710.059171… → S$6,710.06 (re-computed; half-up at 2 dp)
Balance check: 6,710.06 = 6,710.06 ✓. The reserve reconciles monthly; a growing balance is a rounding-policy
defect signal, not a fact of life (§5.4; the interest guide's §6.5 economics: at this scale, cents become millions).
```

### P13 — Card Authorization: Hold *** Capture, 2026-08-20

**Rule:** `CARD_AUTH → soft posting (hold on available, no GL) ; CARD_CAPTURE → hard posting + hold release` (sibling §8.1.3 — the two-phase reserve-and-settle pattern). A S$50.00 card purchase; account balance S$1,000.00.

| Stage | Mechanics | Ledger | Available |
|---|---|---|---|
| Before | — | S$1,000.00 | S$1,000.00 |
| Authorization | Hold S$50.00 — a soft posting: available −50, ledger unchanged, **no GL entry** (§2.3 sibling) | S$1,000.00 | S$950.00 |
| Capture | Hard posting: DR customer S$50.00 / CR merchant settlement GL S$50.00; hold released | S$950.00 | S$950.00 |
| (Auth expires, no capture) | Hold released — no money ever moved, nothing to reverse | S$1,000.00 | S$1,000.00 |

```text
Capture balance check: DR 50.00 = CR 50.00 ✓. The hold is a balance-component move (hold ↑, available ↓), not a
ledger move (sibling §5.1) — the memo/soft-posting discipline of sibling §2.2–2.3.
```

### P14 — The FX Round Trip: Selling Back, 2026-08-14

**Rule:** `FX_SELL × buy rate 1.3460, mid 1.3500 → DR customer USD / CR customer SGD / CR FX income` (§6.3). The customer sells back the USD 739.64 bought in P5.

| Leg | Account | DR | CR |
|---|---|---|---|
| 1 | Customer USD DDA | USD 739.64 | — |
| 2 | Customer SGD DDA | — | S$995.56 |
| 3 | 4200 FX Income (the buy-back spread) | — | S$2.95 |

```text
SGD credited: 739.64 × 1.3460 = 995.55544… → S$995.56 (half-up)
Mid value: 739.64 × 1.3500 = 998.514 → S$998.51 ; spread: 998.51 − 995.56 = S$2.95
Per-currency: USD book DR 739.64 = CR 739.64 ✓ ; SGD book (with the clearing/netting legs) DR 998.51 = CR 995.56 + 2.95 ✓
The round trip: the customer paid S$1,000.00 in P5 and receives S$995.56 back — a S$4.44 round-trip cost,
exactly the two spreads (1.49 + 2.95 = 4.44 ✓, re-computed) — the round-trip invariant of §6.3.
```

### The Postings Table

| # | Posting | Rule (§2) | DR total | CR total | Balanced | Verified |
|---|---|---|---|---|---|---|
| P1 | Daily interest accrual | INTEREST_ACCRUAL | 0.55 | 0.55 | ✓ | Re-computed (0.547945… → 0.55) |
| P2 | Interest capitalization (31 d) | INTEREST_CAPITALIZE | 16.99 | 16.99 | ✓ | Re-computed (16.98630136… → 16.99) |
| P3 | Loan disbursement | LOAN_DISBURSE | 50,000.00 | 50,000.00 | ✓ | Exact |
| P4 | Disbursement fee (capped) | LOAN_FEE | 150.00 | 150.00 | ✓ | Re-computed (250.00 → capped 150.00) |
| P5 | FX conversion (buy USD) | FX_BUY | 1,000.00 SGD / 739.64 USD | 1,000.00 SGD / 739.64 USD | ✓ (both books) | Re-computed (739.644970… → 739.64; spread 1.49) |
| P6 | Same-day storno | REVERSAL_STORN | 1,002.00 | 1,002.00 | ✓ | Mirror property |
| P7 | Mid-period accrual reversal | ACCRUAL_REVERSAL | 12.74 | 12.74 | ✓ | Mirror property |
| P8 | Suspense lifecycle | INWARD_CREDIT_UNMATCHED | 5,000.00 | 5,000.00 | ✓ | Exact |
| P9 | Cross-branch transfer | FUNDS_TRANSFER × CROSS_BRANCH | 1,000.00 per book | 1,000.00 per book | ✓ | Exact |
| P10 | Loan EMI repayment | LOAN_REPAY | 4,280.37 | 4,280.37 | ✓ | Re-computed (EMI, interest/principal split) |
| P11 | EOD FX revaluation | REVALUATION | 1,000.00 | 1,000.00 | ✓ | Re-computed (100k × 0.01) |
| P12 | Rounding-reserve sweep | ROUND_OFF | 6,710.06 | 6,710.06 | ✓ | Re-computed (1M × 0.006710059…) |
| P13 | Card hold and capture | CARD_AUTH / CARD_CAPTURE | 50.00 (capture) | 50.00 | ✓ | Exact; hold has no GL |
| P14 | FX round trip (sell back) | FX_SELL | 739.64 USD / 998.51 SGD | 739.64 USD / 998.51 SGD | ✓ (both books) | Re-computed (995.56; spread 2.95; round trip 4.44) |

---

## 10. The Golden Tests

This section mirrors the interest guide's verification-and-testing engineering ([interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §11) for the posting layer: a golden test is a hand-computed case — inputs plus expected entries, balances, and GL strings to the cent — executed against the engine after every change, with the golden values coming from *independent* computation (the Python-decimal re-derivations of §9, a spreadsheet, a second implementation), never from the engine itself.

### 10.1 The Golden-Test Library: Known-Value Cases

| Dimension | Cases the library must include (the §9 postings are the seed set) |
|---|---|
| **Rules** | Every product × event of the matrix (§2.1) with its exact DR/CR accounts, sides, and GL strings — P3, P4, P10 as seeds |
| **Balancing** | Multi-leg transactions (4-leg transfer + fee, 6-leg cross-branch, 5-leg FX); the per-currency balance proofs of P5/P14; rejection of a deliberately unbalanced instruction |
| **Reversals** | Same-day storno (P6), direct/back-dated reversal, mid-period accrual reversal (P7), reversal-of-a-reversal (returns to the original state), partial reversals (must be configured, never default) |
| **Suspense** | The full lifecycle of P8 — park, age, resolve; the repair-queue re-drive (§4.2) |
| **Precision** | The tie cases (16.435, 16.445, 2.5 — §5.2), the rounding-policy pairs (accrue-unrounded vs round-daily — the S$0.06 drift case), JPY/BHD minor-unit postings (§5.5) |
| **FX** | Conversion with spread (P5), sell-back round trip (P14 — residual must equal the documented double spread), EOD revaluation (P11), rate-capture fields on the entry |
| **Idempotency** | Replay of a succeeded posting (same key → same result, no new entries), replay of a failed posting (clean re-execution), racing duplicate keys, heuristic-duplicate flagging |
| **Dates** | Back-value and forward-value postings landing in the correct GL day; value-date changes re-running interest accrual (§3.3) |

The golden file's shape — (rule ID, product, event, accounts, amounts, dates, expected legs to the cent, expected GL strings, expected balance deltas) — is the contract between product owners and the engine, exactly as the interest guide's golden file is for accruals (§11.1).

### 10.2 Property-Based Testing

The invariants the posting layer must hold over *generated* cases:

- **Balance conservation.** Σ per-account postings = GL movement, for every run (the interest guide's §11.2 invariant, applied to postings).
- **DR = CR per transaction.** Every generated multi-leg instruction — including reversals and FX — produces balanced rounded legs (reject at build, sibling §8.2.3).
- **Reversal symmetry.** Reversing a posting and reversing the reversal returns the accounts to their original state, leg for leg.
- **Round-trip closure.** FX out-and-back returns to the starting balance within the documented spread + rounding-reserve delta (§6.3, P14).
- **Determinism.** Identical input replays produce byte-identical entries, balances, and GL strings (the §2.2 determinism property).
- **No-delete.** No operation can remove an entry; the only mutation is a linked reversal (§3).

### 10.3 Parallel Runs and the Vendor Checklist

Before cutover or after any rule change, the new configuration runs **in parallel** with the old over a full period and reconciles every entry and balance to the cent (sibling §8.9). For vendor cores, the acceptance checklist adds the posting-specific items: every rule of the matrix (§2.1) with exact legs; every reversal variant (§3.1); suspense and repair-queue behavior under failure injection; precision across all currencies; FX rate capture; idempotency under retry storms; and a full-period recompute (sibling §8.9, §7.7).

### 10.4 Recompute / Audit Mode

The posting engine's audit capability: re-derive any historical day from the immutable entry stream and rule history and diff against the booked balances and GL — the mechanism that answers *"show me this account's postings for June"* and proves the books to an auditor or regulator (§3.4, sibling §6.7). It requires the entry log to be append-only and the rule history to be effective-dated (§2.4); a bank that cannot recompute a historical period from recorded inputs does not know its own books.

### 10.5 The Tests Table

| Test | What it proves | Seed cases | Mirror of |
|---|---|---|---|
| Golden known-value cases | Exact legs, GL strings, balances for defined inputs | §9 P1–P14 | Interest guide §11.1 |
| Property tests | DR=CR, conservation, symmetry, round trip, determinism, no-delete | Generated | Interest guide §11.2 |
| Parallel runs | Rule/engine changes agree to the cent over a full period | Production feed | Interest guide §11.3; sibling §8.9 |
| Recompute/audit mode | Historical days re-derivable from recorded inputs | Entry stream replay | Interest guide §11.4 |
| Failure injection | Atomic rollback, idempotent retry, repair-queue behavior | Chaos tests | Sibling §8.9 |

---

## 11. The Banking Context: Cymbal Bank

Cymbal Bank is the Singapore retail and SME bank whose architecture the corpus documents (the persona's employer; see the repo's banking guides for the platform landscape). The posting mechanics of this guide are the layer *underneath* every Cymbal Bank flow the other guides describe:

- **The payments cluster** — a PayNow/FAST transfer ([payment_rails_guide.md](payment_rails_guide.md), [payments_hub_guide.md](payments_hub_guide.md), [nets_software_systems_guide.md](nets_software_systems_guide.md)) is, at the bottom, the four-leg posting of the sibling's §9 — P6's storno is what happens when that transfer is recalled same-day; the unmatched-inward-credit flow of P8 is the FAST credit that arrives with a closed beneficiary account; the cross-branch transfer of P9 is the two-branch FAST settlement through MEPS+.
- **The interest engine** — the daily accrual P1 and the capitalization P2 are the postings the interest run books for Cymbal Bank's savings book ([interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §2.4, §6.2; [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §5); the mid-period reversal P7 is the correction flow when a rate error is found mid-month; the loan EMI split P10 is what the loan module hands to the posting engine on repayment day.
- **The open-source reference** — the Fineract guides ([apache_fineract_guide.md](apache_fineract_guide.md), [fineract_database_models_guide.md](fineract_database_models_guide.md)) show the same mechanics in an open core: the `acc_gl_journal_entry` rows and the loan/deposit accounting rules are the P1–P14 patterns in open source.
- **The vendor cores** — the same postings run in Temenos (`STMT.ENTRY`/`CATEG.ENTRY`, sibling §7.1; [t24_programming_guide.md](t24_programming_guide.md)) and FLEXCUBE (`ACTB_DAILY_LOG`, sibling §7.2; [oracle_flexcube_data_model_guide.md](oracle_flexcube_data_model_guide.md) §5) — a migration or greenfield evaluation tests each vendor's mechanics against the §10 library, not marketing slides.
- **The limits engine** — every balance-moving posting checks limits first ([banking_limits_domain_guide.md](banking_limits_domain_guide.md)); P3/P10/P13's debit legs are the guarded legs.
- **The batch window** — the EOD interest run and GL close (P1/P2/P11 in sequence) run inside the window sized in [../technology/capacity_sizing_guide.md](../technology/capacity_sizing_guide.md), with the restart-safety of [../technology/zero_downtime_system_design_guide.md](../technology/zero_downtime_system_design_guide.md) — replay (§7.3) makes a midnight restart safe.
- **The event-driven future** — the outbox pattern ([../technology/kafka_guide.md](../technology/kafka_guide.md)) carries posting intents with exactly-once delivery; the idempotency keys of §7 are the contract that makes it safe.

### The Banking Table

| Cymbal Bank flow | Posting mechanics involved | This guide | Machinery home |
|---|---|---|---|
| PayNow/FAST transfer (internal) | 4-leg transfer + fee; same-day recall = storno | §9 P6; §2.1 matrix | Sibling §9 |
| PayNow/FAST inward, closed account | Suspense park → resolve → credit | §9 P8; §4.1 | Sibling §6.4 |
| Cross-branch settlement | Inter-branch clearing legs, per-branch books | §9 P9; §2.2 | Sibling §4.3 |
| Monthly interest capitalization | Accrual reversal + capitalization | §9 P1–P2; §3.3 | [interest_engines_core_banking_guide.md](interest_engines_core_banking_guide.md) §6.2 |
| Loan disbursement + fee | Asset DR / DDA CR; capped fee | §9 P3–P4 | Sibling §6.3 |
| Loan repayment | EMI split: principal CR / interest CR | §9 P10 | [interest_calculation_engine_guide.md](interest_calculation_engine_guide.md) §4.5 |
| FX conversion (app) | Per-currency balancing + spread | §9 P5; §6.1 | Sibling §4.10 |
| EOD FX revaluation | Revaluation reserve entry | §9 P11 | Sibling §6.5 |
| Card authorization → capture | Hold (soft) → capture (hard) | §9 P13 | Sibling §8.1.3 |
| Retried payment after timeout | Idempotency key returns the original | §7 | Sibling §8.2.2 |
| EOD GL close | Control-account reconciliation, breaks to suspense | §8 | Sibling §6.2, §6.6 |

---

## 12. The Worked Example: A Posting-Mechanics Design

The scenario — a familiar Cymbal Bank flow, designed to exercise the mechanics of this guide end to end: **Friday 2026-08-28, 23:55 — the last five minutes before the EOD cut-off.**

**The events.** (1) A PayNow transfer of S$480.00 from customer A to customer B (same branch) initiates at 23:57 — inside the cut-off, so it posts with a Saturday value date per the clearing calendar. (2) An inward FAST credit of S$2,000.00 arrives at 23:58 for an account closed that afternoon — the beneficiary cannot be resolved. (3) At 23:59 the payments hub retries a transfer it believes failed at 23:54 — the original posted; the retry carries the same idempotency key. (4) The EOD interest capitalization run is mid-flight when the GL module becomes briefly unavailable. (5) The day-end reconciliation finds a S$0.01 break on the savings-deposits control GL.

**The design, per the mechanics of this guide:**

1. **The rules.** The transfer resolves through the matrix (§2.1): DR payer / CR payee, same branch — two legs, no clearing legs needed (P3's cousin; the four-leg version when a fee applies, sibling §9.2). The value date is Saturday — the entry carries booking date 2026-08-28 and value date 2026-08-29; the balance engine applies the availability rule (the funds show as cleared per the FAST scheme's T+0 availability — the value date governs interest, not availability, sibling §2.6).
2. **The idempotency.** The retry at 23:59 carries key `PAY-20260828-88412`; the engine's key lookup finds the original entries and returns the original confirmation — no double post (§7.1). The heuristic duplicate layer would have *flagged* it for review even without the key; the key *blocked* it.
3. **The suspense.** The inward FAST credit parks per P8: DR Nostro — SGD S$2,000.00 / CR 2990 Suspense S$2,000.00, with the suspense reason code and an owner. Monday's operations matches it to the customer's new account and resolves it — DR Suspense / CR the new DDA (§4.1).
4. **The failure and the repair queue.** The interest capitalization run's GL write fails on the unavailable GL module; the posting unit rolls back atomically (no half-posted interest, sibling §8.2.1) and the item lands in the repair queue with its payload, the error, and its (account, value date, period) key (§4.2). The GL module recovers in twelve minutes; the re-drive replays the batch — the accrual keys make the restart safe (the interest guide's §8.3 checkpoint pattern; §7.3 of this guide). The day still closes on time: the EOD pipeline's control totals hold the GL close open for exactly this reconciliation, and the replayed run reconciles to the cent.
5. **The break.** The reconciliation finds S$0.01 on the savings-deposits control GL — the residual of the interest roundings across 1.4 million savings accounts (the §5.4 phenomenon; P12's mechanics). It is an *explained* break: it posts to the rounding reserve with its owner and clears the control-account check (§8.3). The control accounts — 2310 Savings Deposits, 2320 Current Accounts, the nostro GLs, the suspense GL — all reconcile, and the trial balance is signed off (sibling §6.6).

**The lessons:**

- **The mechanics are a system, not a feature list.** The same five minutes exercised the rules, the value dating, the idempotency, the suspense, the repair queue, the atomicity, and the reconciliation — and the design that survived was the one where each mechanism was *configured and tested in advance* (§10), not improvised at 23:59.
- **Every failure mode has a designed landing zone.** The retry landed on the idempotency key, the unknown beneficiary on suspense, the failed batch on the repair queue, the rounding residual on the reserve — no failure mode was left to become a break, and no break was left without an owner.
- **The value date is the spine.** Saturday's value date, Monday's booking of the suspense resolution, the midnight batch restart — every mechanism respected the three-date discipline (sibling §4.9) and the value-dated balance history the interest engine depends on (§3.3).
- **The books closed balanced.** All five events posted, parked, or replayed — and the trial balance proved it: *Σ DR = Σ CR*, the control accounts reconciled, and the bank's books were ready for Monday.

---

## 13. The Summary: The Balanced Books

The posting engine is Pacioli's 1494 rules compiled into software (sibling §1.2) — and the posting mechanics are the layer that makes those rules *true at scale*:

- **The rules** decide the sides and the accounts — the product × event × account matrix, the determination precedence, the GL strings — deterministically and by configuration, so a wrong-but-balanced entry is designed out, not hunted down (§2).
- **The reversals** undo without erasing — the storno variants, the mid-period corrections, the audit-linked chain regulators can reconstruct (§3).
- **The suspense and the exceptions** give every unresolved item a landing zone — parked, aged, owned, cleared; a repair queue whose replays are safe (§4).
- **The precision** keeps the arithmetic exact — decimal, never float; rounding at defined points; the balancing invariant checked three ways; the rounding reserve absorbing what rounding leaves behind (§5).
- **The multi-currency postings** balance twice — every currency's book, the spread booked, the rate captured, the round trip provable (§6).
- **The idempotency** makes retries free — one key, one posting, replay-safe restarts (§7).
- **The reconciliation** proves the books — sub-ledger to GL, every break owned and corrected by posting, never by edit (§8).
- **The worked postings** are the proof — fourteen entries, every cent re-computed (§9); **the golden tests** keep it true — known-value cases, invariants, parallel runs, recompute (§10).

**The final word — the balanced books.** Every mechanism in this guide exists for one outcome: at the end of every day, at every branch, in every currency, for every control account, the bank's books balance — *Σ debits equals Σ credits*, the sub-ledgers agree with the GL to the cent, every reversal links to its original, every suspense item has an owner, and every cent is re-derivable from the entries on record. That is the operational meaning of "the balanced books": not a slogan, but a daily, evidenced, auditable state — the state the posting engine exists to produce, and the mechanics in this guide exist to make certain.

---

## 14. Glossary

| Term | Definition |
|---|---|
| **Posting** | The accounting record a transaction produces: the set of DR/CR legs that move balances and GL accounts (sibling §2.1) |
| **Double entry** | The system in which every transaction affects at least two accounts; Σ debits = Σ credits always (sibling §1.1) |
| **Debit (DR)** | A left-side entry; increases assets/expenses, decreases liabilities/equity/revenue (DEAD) |
| **Credit (CR)** | A right-side entry; increases liabilities/equity/revenue, decreases assets/expenses (CLIC) |
| **Journal entry** | The record of a posting's legs — the unit the entry engine builds, balances, and commits atomically (sibling §4.6) |
| **GL string** | The complete accounting identity of a leg — COA account, type, currency, branch, product, transaction code, dates, references (§2.3) |
| **Sub-ledger** | The account-level detail behind a GL control account; the source of truth for customer balances (sibling §1.8) |
| **General ledger (GL)** | The bank's summarized book of accounts, one row per GL account per branch/currency (sibling §1.8) |
| **Control account** | A GL account with a subsidiary ledger behind it; both posted; totals compared at the trial balance (§8.2; verified concept) |
| **Storno** | The equal-and-opposite cancellation posting (German accounting practice; "red storno"); original and reversal both remain on record (§3.1) |
| **Reversal** | The counter-posting that undoes a posting — a new posting, never a deletion (sibling §3.7) |
| **Reversing entry** | A scheduled automatic opposite entry, typically reversing an accrual at the next period's start (§3.1) |
| **Suspense account** | A temporary GL holding doubtful entries pending analysis and classification; cleared when resolved; a control risk (§4.1) |
| **Repair queue** | The queue of failed/unpostable work with retry metadata, error, and idempotency key (§4.2) |
| **Break** | A residual difference after reconciliation — explained (rounding, timing) or unexplained (a defect signal) (§4.3, §8.3) |
| **Rounding** | The defined method and point at which amounts are truncated to the currency's scale; a product parameter (§5.2) |
| **Decimal** | The exact-arithmetic representation of money (BigDecimal / decimal / NUMERIC); floats are banned for money (§5.1) |
| **Float** | (a) The uncleared-funds balance component (sibling §5.1); (b) shorthand for binary floating point, banned for money (§5.1) |
| **FX** | Foreign exchange; the conversion of a posting across currencies with per-currency balancing (§6) |
| **Rate capture** | Recording the exact rates, source, timestamp, and rate-set version on every FX entry (§6.2) |
| **Round trip** | Converting an amount out and back; the residual must equal the documented double spread + rounding (§6.3, §9 P14) |