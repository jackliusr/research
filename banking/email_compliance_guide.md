# SMTP and Email in Banking Compliance — the Message Channel That Is Really a Records System

**SMTP and the mail estate read as a compliance problem — the Decoder (SMTP, the MTA, the relay, the gateway, the envelope, the header, the journal, the archive, the WORM store, the legal hold, the lexicon, the disposition), the Thesis (a bank's email is a records system that happens to deliver messages), the Boundary declared by name (this guide owns the email channel only; managed file transfer is a DIFFERENT channel), What SMTP Is and Is Not (store-and-forward, the relay, no payment rail, no confidentiality by default, no guaranteed delivery), the Envelope versus the Header and What Each Authentication Layer Actually Proves (RFC 5321 envelope, RFC 5322 header, SPF, DKIM, DMARC — with an honest "what NONE of them prevents" table), the Outbound Problem (the bank as sender: deliverability, domain reputation, brand spoofing, transactional vs marketing without asserting any consent rule), the Inbound Problem (the bank as target: business email compromise, the spoofed instruction, the supplier bank-details change, and the ONE control that matters — out-of-band verification on a previously-known number), the Gateway and Platform Controls (placement, inspection, STARTTLS, MTA-STS, link and attachment handling), the RECORD (envelope AND message, journaling as a mechanism, what a WORM store does and does not guarantee, and how an institution would know the archive is missing something), Retention and Legal Hold and Disposition (the shape of the obligation; the period is jurisdiction-specific and unverified here), Surveillance and the Lexicon (the review obligation, the false-positive burden, the review capacity a programme consumes), Discovery and Production (collection, review, production, chain of custody), the Regulated-Institution View, the Failure Modes (Symptom / Cause / Guardrail), the Cymbal Bank Worked Example, the Anti-Patterns, the Claims Audit, and What Could Not Be Verified plus the Glossary**

> **Author:** Jack Liu Shurui — Solution Architect
> **Context:** Banking Domain / Technology Risk & Compliance — the SMTP and email estate of a regulated financial institution understood as a compliance and records problem: outbound authentication (SPF, DKIM, DMARC), inbound attacks (spoofing, business email compromise, the spoofed payment instruction), gateway and transport controls (content inspection, STARTTLS, MTA-STS, attachment and link handling), capture and archive (the envelope and the message, journaling, the WORM store), retention, legal hold and disposition, surveillance and the lexicon, and discovery and production. The guide teaches the protocol mechanics only as far as they bear on the compliance obligations, verifies every protocol claim at the RFC Editor's own text, and states plainly which claims could not be sourced.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the RFC Editor's own text, fetched and read this pass — [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321.txt) *Simple Mail Transfer Protocol* (October 2008, Standards Track, obsoletes 2821, updates 1123); [RFC 5322](https://www.rfc-editor.org/rfc/rfc5322.txt) *Internet Message Format* (October 2008, Standards Track, obsoletes 2822, updates 4021); [RFC 7208](https://www.rfc-editor.org/rfc/rfc7208.txt) *Sender Policy Framework (SPF) for Authorizing Use of Domains in Email, Version 1* (April 2014, Standards Track, obsoletes 4408); [RFC 6376](https://www.rfc-editor.org/rfc/rfc6376.txt) *DomainKeys Identified Mail (DKIM) Signatures* (September 2011, Standards Track, obsoletes 4871 and 5672); [RFC 7489](https://www.rfc-editor.org/rfc/rfc7489.txt) *Domain-based Message Authentication, Reporting, and Conformance (DMARC)* (March 2015, **Informational** — not Standards Track); [RFC 3207](https://www.rfc-editor.org/rfc/rfc3207.txt) *SMTP Service Extension for Secure SMTP over Transport Layer Security* (February 2002, Standards Track, obsoletes 2487); [RFC 8461](https://www.rfc-editor.org/rfc/rfc8461.txt) *SMTP MTA Strict Transport Security (MTA-STS)* (September 2018, Standards Track); [RFC 8314](https://www.rfc-editor.org/rfc/rfc8314.txt) *Cleartext Considered Obsolete: Use of TLS for Email Submission and Access* (January 2018, Standards Track); [RFC 8601](https://www.rfc-editor.org/rfc/rfc8601.txt) *Message Header Field for Indicating Message Authentication Status* (May 2019, Standards Track, obsoletes 7601); and [RFC 8617](https://www.rfc-editor.org/rfc/rfc8617.txt) *The Authenticated Received Chain (ARC) Protocol* (July 2019, **Experimental**). **No retention period, no supervisor rule name, and no phishing/BEC loss figure was printed** — see §15; those claims could not be sourced to a dated primary publication this pass. Where a search tool returned empty, it is recorded as a tool limitation, not as absence of material.
> **Last Updated:** October 2026
> **Companion guides (sibling, same folder):** [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) · [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) · [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md)
> **Companion guides (technology/, prefix `../technology/`):** [Cybersecurity](../technology/cybersecurity_guide.md) · [Data Governance](../technology/data/data_governance_framework.md) · [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md) · [StorageGRID](../technology/storagegrid_guide.md)

---

**How to use this guide:** Section 1 is the overview — the decoder, the thesis, and the boundary declared by name. Section 2 is what SMTP is and is not. Section 3 is the technical hook — the envelope versus the header, and what SPF, DKIM and DMARC actually prove, with the honest "what none of them prevents" table. Section 4 is the outbound problem — the bank as a sender. Section 5 is the inbound problem — the bank as a target, and the one control that matters. Section 6 is the gateway and platform controls. Section 7 is the record — the guide's core: what must be captured, how journaling works, what a WORM store does and does not guarantee, and how an institution would know the archive is missing something. Section 8 is retention, legal hold and disposition — the shape of the obligation, with the period left unverified. Section 9 is surveillance and the lexicon, including the honest cost. Section 10 is discovery and production. Section 11 is the regulated-institution view. Section 12 is the failure modes (Symptom / Cause / Guardrail). Section 13 is the Cymbal Bank worked example (explicitly fictional and illustrative). Section 14 is the anti-patterns. Section 15 is the claims audit. Section 16 is what could not be verified, the glossary, the cross-references, and the closing summary. **Integrity convention:** ✅ = verified this pass against a named, dated primary source (the RFC number and its date, a standard body, a regulator publication, vendor documentation, or dated press); ⚠ = flagged, inferred, or unverified. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`.

---

**Table of Contents**

1. [The Overview — the Decoder, the Thesis, and the Boundary](#1-the-overview--the-decoder-the-thesis-and-the-boundary)
2. [What SMTP Is and Is Not](#2-what-smtp-is-and-is-not)
3. [The Envelope versus the Header, and What Each Layer Actually Proves](#3-the-envelope-versus-the-header-and-what-each-layer-actually-proves)
4. [The Outbound Problem — the Bank as a Sender](#4-the-outbound-problem--the-bank-as-a-sender)
5. [The Inbound Problem — the Bank as a Target](#5-the-inbound-problem--the-bank-as-a-target)
6. [The Gateway and Platform Controls](#6-the-gateway-and-platform-controls)
7. [The Record — Envelope, Message, Journal, WORM, and Completeness](#7-the-record--envelope-message-journal-worm-and-completeness)
8. [Retention, Legal Hold and Disposition](#8-retention-legal-hold-and-disposition)
9. [Surveillance and the Lexicon](#9-surveillance-and-the-lexicon)
10. [Discovery and Production](#10-discovery-and-production)
11. [The Regulated-Institution View](#11-the-regulated-institution-view)
12. [Failure Modes — Symptom, Cause, Guardrail](#12-failure-modes--symptom-cause-guardrail)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Anti-Patterns](#14-the-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, and the Closing](#16-what-could-not-be-verified-the-glossary-and-the-closing)

---

## 1. The Overview — the Decoder, the Thesis, and the Boundary

### 1.1 The Decoder

Every discipline has a decoder ring — the small set of words that, once defined, let you read the rest of the field. For the bank's mail estate, the decoder is twelve words:

- **SMTP** — the Simple Mail Transfer Protocol: the store-and-forward protocol by which one mail server hands a message to another. Defined in RFC 5321 (October 2008) ✅.
- **The MTA** — the Mail Transfer Agent: the server process that speaks SMTP. A bank's estate has many MTAs, each a hop on a path.
- **The relay** — any intermediate MTA that receives a message and forwards it toward a destination it is not itself the final recipient of. RFC 5321 calls this "SMTP mail relaying" ✅.
- **The gateway** — the MTA at the boundary between transport environments (the Internet and the internal mail system), where policy is enforced: inspection, filtering, hygiene.
- **The envelope** — the SMTP transaction's routing data: `MAIL FROM` and `RCPT TO`, the addresses the transport actually uses. Distinct from the header ✅ (RFC 5321).
- **The header** — the message content's own claims: `From:`, `To:`, `Received:`, and the rest. What a reader sees. Distinct from the envelope ✅ (RFC 5322).
- **The journal** — the mechanism by which a mail server emits a copy of a message as it transits, so that the copy can be captured into a separate store. This is a mail-server feature, not a protocol feature; no RFC defines it.
- **The archive** — the store that keeps the captured copies, searchable and producible, for the length of the obligation.
- **The WORM store** — Write Once Read Many storage: once written, a record cannot be rewritten. It protects against alteration; it does not by itself prove completeness. ⚠ (a property of the storage medium, not of the capture path).
- **The legal hold** — the suspension, for a defined set of custodians and subjects, of an otherwise-running automated deletion process.
- **The lexicon** — the list of words and patterns a surveillance programme searches for in the message corpus.
- **The disposition** — the final, auditable act of deleting a record when its retention period has run and no hold applies.

Those twelve words carry the whole guide. Nothing that follows requires any other vocabulary than this.

### 1.2 The Thesis

**A bank's email is not a messaging system; it is a records system that happens to deliver messages.**

That single inversion is the argument of this guide. A retail customer opens an email client and experiences email as *communication* — a message, sent, received, read, deleted. A regulated institution must experience the same technology as *evidence* — a communication that occurred, that some rule expects to be captured, retained, and producible, and that may one day be read by a supervisor, an auditor, a litigant, or the institution's own compliance function. The two experiences describe the same packets on the same wire. They produce completely different architectures.

Once you accept the inversion, the questions reorder themselves. The interesting question is no longer "did the message arrive?" — a sending problem — but "is the copy that will be produced five years from now a faithful, complete, tamper-evident record of the message that actually transited?" — a *records* problem. The first question is answered by the sender. The second is answered, or not answered, by the capture path the institution built. Most of the failures catalogued in §12 live in the second question, not the first, and a bank that spends its mail budget on the first while neglecting the second has bought the wrong thing.

The thesis also imposes a discipline on claims. Because the bank's mail is evidence, any figure printed about it — how long to keep it, how much loss it caused, what rule requires it — must be traceable to a dated source, or it must be flagged as unverified. This guide follows that discipline ruthlessly: where a fact could not be sourced, the guide says so rather than printing a plausible number. §15 records those absences as findings.

### 1.3 The Boundary — Declared by Name

**This guide owns exactly one subject: the email channel inside a bank, understood as a compliance and records problem.** The scope is bounded on six fronts, deliberately:

1. **Outbound authentication** — proving a message that claims to come from the bank's domain was sent by a system the bank authorized (§3, §4).
2. **Inbound attacks** — the messages that arrive *at* the bank seeking to cause a fraudulent transfer or a data loss, and the controls that stop the one that matters (§5).
3. **Gateway controls** — the boundary enforcement of policy on mail in transit (§6).
4. **Capture and archive** — the record: the envelope and the message, journaling, the WORM store, and completeness (§7).
5. **Retention, legal hold and disposition** — the shape of the obligation, and the mechanics of a hold (§8).
6. **Surveillance, discovery and production** — reading the corpus, and producing it under process (§9, §10).

**Managed file transfer (MFT) is a DIFFERENT channel — file transfer, not mail — and is out of scope here.** MFT moves files between systems and trading partners using protocols such as PeSIT, SFTP, FTPS, AS2 and AS4, with its own journaling, checkpointing and partner governance model. It is not SMTP, its records are not mail records, and its compliance obligations, while similarly shaped, arise from a distinct set of business flows (payments files, regulatory reporting files, interbank exchanges). This guide does not cover it; read [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md) instead. The confusion between the two channels is common and costly: an institution that assumes "we archive our email, so we archive our file transfers" or the reverse has mis-scoped its evidence estate. The two channels need two capture paths, two retention decisions, and two production capabilities. They share a principle — the record must be complete and tamper-evident — and nothing else.

Similarly, this guide treats the *fraud* that arrives by email (§5) but does not re-derive fraud detection; for that, read [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md). It treats the *data-protection* dimension of what may be inspected and how (§6, §9) but does not re-derive the data-governance model; for that, read [Data Governance](../technology/data/data_governance_framework.md). And it treats the *security* of the mail estate but does not re-derive the security programme; for that, read [Cybersecurity](../technology/cybersecurity_guide.md).

### 1.4 The Key-Facts Table

| Fact | Statement | Source (date, kind) |
| --- | --- | --- |
| SMTP specification | RFC 5321, *Simple Mail Transfer Protocol*; Standards Track; obsoletes 2821; updates 1123 | RFC 5321, October 2008 ✅ (RFC) |
| Message format | RFC 5322, *Internet Message Format*; Standards Track; obsoletes 2822; updates 4021 | RFC 5322, October 2008 ✅ (RFC) |
| SMTP is store-and-forward / relayed | "a mail message may pass through a number of intermediate relay or gateway hosts on its path from sender to ultimate recipient"; "SMTP mail relaying" | RFC 5321 §2.1, October 2008 ✅ (RFC) |
| The envelope is separate from the header | The SMTP envelope is sent as protocol units (`MAIL FROM`, `RCPT TO`); the header is the message content; a relay "has no need to inspect or act upon the header section or body … except to add its own 'Received:' header field" | RFC 5321 §§2.3.1, 3.6.1, 4.4, October 2008 ✅ (RFC) |
| SPF authenticates the envelope sender identity | SPF checks the "MAIL FROM" or "HELO" identity — the envelope — not the visible `From:` | RFC 7208 §§1.1.3, 1.1.4, 2.3–2.4, April 2014 ✅ (RFC) |
| DKIM signs selected headers plus the body | Signature carries `h=` (signed header fields) and `bh=` (body hash); `d=` is the Signing Domain Identifier claiming responsibility | RFC 6376 §§2.5, 3.5, September 2011 ✅ (RFC) |
| DMARC's alignment requirement | Identifier Alignment ties the domain in the RFC 5322 `From:` address to an authenticated identifier from SPF or DKIM | RFC 7489 §3.1, March 2015 ✅ (RFC, **Informational**) |
| STARTTLS extension | RFC 3207, *SMTP Service Extension for Secure SMTP over TLS*; Standards Track; obsoletes 2487 | RFC 3207, February 2002 ✅ (RFC) |
| MTA-STS | RFC 8461, *SMTP MTA Strict Transport Security*; Standards Track | RFC 8461, September 2018 ✅ (RFC) |
| TLS for submission/access | RFC 8314, *Cleartext Considered Obsolete*; Standards Track; updates 1939, 2595, 3501, 5068, 6186, 6409 | RFC 8314, January 2018 ✅ (RFC) |
| Authentication-Results header | RFC 8601, Standards Track; obsoletes 7601 | RFC 8601, May 2019 ✅ (RFC) |
| Authenticated Received Chain | RFC 8617, ARC; **Experimental** | RFC 8617, July 2019 ✅ (RFC, Experimental) |
| Retention period | **Not stated** — jurisdictional, unverified here | see §15 ⚠ (finding) |
| Phishing/BEC loss figure | **Not stated** — no dated report retrieved | see §15 ⚠ (finding) |
---

## 2. What SMTP Is and Is Not

### 2.1 Store-and-Forward

SMTP is a **store-and-forward** protocol. This is not an implementation detail; it is the property that shapes every compliance story that follows. A sending mail server does not, in general, hand a message directly into the recipient's mailbox. It hands the message to a *next hop* — a relay or a gateway — which accepts responsibility for it, stores it, and forwards it onward. RFC 5321 states this plainly: "In this way, a mail message may pass through a number of intermediate relay or gateway hosts on its path from sender to ultimate recipient" ✅ (RFC 5321 §2.1, October 2008). The next-hop destination is resolved from the recipient domain's Mail eXchanger (MX) records in DNS ✅ (same section).

The consequences are immediate and they are *records* consequences:

- **A message has a path, not a pipe.** Between sender and reader the message may touch several MTAs, each of which may add a `Received:` header field recording the hop ✅ (RFC 5321 §4.4). The path is evidence: it says where the message went and which systems handled it.
- **Delivery is a chain of custody, not an atomic act.** Each hop accepts the message and becomes responsible for the next. If a hop fails after accepting, the message may still be in someone's queue — not lost, but not delivered.
- **The same message exists in several places at once.** At any instant during transit there may be copies on the sender's MTA, on a relay, and on the recipient's server. That multiplicity is exactly what a capture path must not lose track of (§7).

### 2.2 The Relay

A **relay** is an MTA that accepts a message and forwards it toward a destination for which it is not the final recipient. RFC 5321 discusses this under "SMTP mail relaying" ✅ (RFC 5321 §2.1). A host may act as both server and client in a relay situation — receiving from one hop, sending to the next ✅ (RFC 5321 §2.3.8).

For compliance, the relay matters for two reasons. First, a relay is where a message can *leave the observed path*: if mail transits a relay that does not journal, the copy that reaches the archive may be incomplete or absent (§7, §12). Second, RFC 5321 imposes a discipline on what a relay may do to the message: "a relay SMTP has no need to inspect or act upon the header section or body of the message data and MUST NOT do so except to add its own 'Received:' header field" ✅ (RFC 5321 §3.6.1, October 2008). That prohibition is important for the integrity of the record: a relay that silently rewrites headers is damaging evidence. Note carefully that a **gateway** — the boundary MTA between transport environments — is a different role from a pure relay, and RFC 5321 explicitly permits a gateway to rewrite header fields when crossing environment boundaries ("Header fields MAY be rewritten when necessary as messages are gatewayed across mail environment boundaries") ✅ (RFC 5321 §§3.7, 3.7.1). The distinction matters: rewriting at a relay is a red flag; rewriting at a gateway is expected and should itself be logged.

### 2.3 Envelope and Header, Introduced

Even this early, one distinction must be stated, because §3 develops it fully. SMTP transports a **mail object** that contains two logically separate things: an **envelope** and **contents**. "The SMTP envelope is sent as a series of SMTP protocol units" — the transaction commands — while "the contents comprise the object to be delivered to the recipient" ✅ (RFC 5321 §2.3.1). The envelope carries the routing addresses (`MAIL FROM`, `RCPT TO`); the contents carry the header and body. They are *not* the same, they *need not agree*, and the divergence between them is the technical hook of this entire guide (§3).

RFC 5321 is also candid that the envelope is a property of the SMTP transport environment and may not survive translation: "when a message leaves the Internet environment, it may be necessary to fold the SMTP envelope information into the message header section" ✅ (RFC 5321 §3.7.1). A bank that captures only the header, or only the message as displayed, has already begun to lose the envelope — and the envelope is where `BCC` recipients, distribution-list members, and the actual delivery path live (§7).

### 2.4 What SMTP Is Not — Three Negations

**SMTP is not a payment rail.** SMTP carries *instruction text*, not value. A message that says "please update the beneficiary account to X and remit" is, at the protocol level, an email. It authorizes nothing, settles nothing, and moves no money. The fraud in §5 exists precisely because human beings act on the *content* of messages as though the content were an instruction with authority. The protocol provides no mechanism to make an email an instruction; any authority an email appears to have is granted by a human process, not by SMTP. Treating an email as though it were a payment instruction is a *process* failure, not a protocol capability.

**SMTP is not confidential by default.** RFC 5321 defines the base protocol over TCP without transport encryption, and notes explicitly that early SMTP is transported "in the clear" ✅ (RFC 3207 §1, February 2002, discussing the base case). Confidentiality is added by *extensions* — STARTTLS (RFC 3207) and, for submission and access, the recommendations of RFC 8314 (January 2018) ✅. Even then, STARTTLS is *opportunistic*: the base extension cannot force a peer to use TLS, and a downgrade attack that strips the STARTTLS offer is a known weakness that MTA-STS (RFC 8461, September 2018) is designed to counter ✅ (RFC 8461 Abstract: "specify whether sending SMTP servers should refuse to deliver to MX hosts that do not offer TLS with a trusted server certificate"). The compliance reading: **unencrypted mail in transit is not confidential, and an institution that treats the wire as private is mistaken.** §6 develops the controls.

**SMTP is not a guaranteed-delivery channel.** SMTP provides best-effort delivery with error notifications, not a guarantee. A mail server that accepts a message takes responsibility for attempting delivery and, failing that, for generating a **non-delivery notification** to the envelope's return path (the reverse-path) ✅ (RFC 5321 §4.5.5). But there is no end-to-end acknowledgement that the *human recipient* read the message, and there is no guarantee the message cannot be lost between hops, filtered as spam, or silently discarded after acceptance. Any compliance design that assumes "if we sent it, they received it" is built on a guarantee SMTP does not offer. Conversely, any *capture* design that produces evidence of a message the sender's MTA accepted is producing evidence of *acceptance*, not of *delivery* — a distinction that §7 and §12 return to.

### 2.5 The Honest Summary

| SMTP characteristic | Compliance consequence |
| --- | --- |
| Store-and-forward, multi-hop ✅ | A message has a path to be reconstructed; copies exist at several points; a hop that does not journal creates a gap. |
| Relays MUST NOT rewrite headers (except adding `Received:`) ✅ | Silent rewriting damages the integrity of the record; gateways may rewrite but must log it. |
| Envelope ≠ header ✅ | Capturing only one of them loses `BCC`, distribution lists, or the visible `From`. |
| Not a payment rail ✅ | An email has no protocol-level authority; authority is a human process. |
| Not confidential by default ✅ | Encryption is an extension (STARTTLS/MTA-STS), opportunistic at best without MTA-STS. |
| Not guaranteed delivery ✅ | Evidence of acceptance ≠ evidence of delivery or reading. |

---

## 3. The Envelope versus the Header, and What Each Layer Actually Proves

This is the guide's technical hook. A reader must finish this section understanding why "the email came from our domain" is a more complicated statement than it looks — because *four* different things are called "the sender," and the authentication layers do not all speak about the same one.

### 3.1 The Envelope — What SMTP Actually Routes On

The envelope is the set of SMTP transaction commands. The core two are:

```
MAIL FROM:<reverse-path>          (the envelope sender — the "return path")
RCPT TO:<forward-path>            (one per recipient — the envelope recipients)
```

Both are defined as protocol units in RFC 5321's grammar ✅ (RFC 5321 §§4.1.1.2, 4.1.1.3, October 2008). The envelope sender is what gets error notifications; the envelope recipients are the addresses the transport actually directs the copy to. Crucially:

- **The envelope sender (`MAIL FROM`) is not the visible `From:` header.** They *usually* match for ordinary mail, but nothing requires it, and the divergence is exactly what spoofers and misconfigured senders exploit.
- **The envelope recipients (`RCPT TO`) are not the visible `To:` header.** `BCC` recipients appear in `RCPT TO` but *never* in the header ✅ — the header simply omits them. A distribution list expands to several envelope recipients while the header may name only the list address ✅ (RFC 5321 §3.10 discusses the expansion of lists and the change of the envelope at each expansion).
- **The envelope carries the actual delivery path.** The sequence of relay hops is recorded in `Received:` header fields *added by* the relays ✅, but the envelope is the authoritative statement of which systems the message was offered to.

**Why the envelope matters separately to a bank:** the envelope is the only place that shows who *actually* received a copy. A message with a `BCC` to an unauthorized external address has an envelope recipient the header never shows. An archive built from headers alone cannot answer "who received this?" — and "who received this?" is a question every discovery request asks (§10). Capture must therefore retain the envelope as data, not merely the rendered message.

### 3.2 The Header — What the Message Claims

The header is the message content's own metadata: `From:`, `Sender:`, `Reply-To:`, `To:`, `Cc:`, `Date:`, `Subject:`, `Message-ID:`, `Received:`, `Return-Path:`, and any others ✅ (RFC 5322, October 2008). RFC 5322 is the format specification for this content, and it is careful about the separation: a message "consists of header fields … and … an optional body"; the envelope is a transport concern the format document does not define ✅ (RFC 5322 §3.6 and §3.6.1 introduction).

The key point for compliance: **header fields are claims made by the sender, not facts verified by the transport.** The `From:` field is *who the message says it is from*. RFC 5322 does not, and cannot, verify it. Any system that treats the `From:` display as authenticated is trusting the sender to be honest.

Also note the header can carry a `Return-Path:` field that a final-delivery MTA inserts from the envelope reverse-path ✅ (RFC 5321 §4.4 describes the addition of trace and delivery fields). `Return-Path:` is therefore *envelope information folded into the header at the last hop* — a useful forensic bridge between the two, but only present if a final delivery agent added it, and not to be confused with the sent `From:`.

### 3.3 SPF — What It Authenticates (the Envelope Sender)

**Sender Policy Framework (SPF)** is defined in RFC 7208 (April 2014, Standards Track, obsoletes RFC 4408) ✅. Its Abstract is unambiguous about the problem it solves and the identity it speaks about ✅ (RFC 7208 Abstract):

> "existing protocols place no restriction on what a sending host can use as the 'MAIL FROM' of a message or the domain given on the SMTP HELO/EHLO commands."

SPF lets a domain publish, in DNS, **which hosts are authorized to use its domain name** in the envelope, and lets a receiver *check* that authorization. The identities SPF checks are defined precisely:

- The **`MAIL FROM` identity** — the envelope sender's domain ✅ (RFC 7208 §1.1.3).
- The **`HELO` identity** — the domain given in the SMTP `HELO`/`EHLO` command ✅ (RFC 7208 §1.1.4, §2.3).

**SPF does NOT authenticate the visible `From:` header.** It authenticates the *envelope* sender. This is the single most misunderstood fact in email security. A message can carry a `From:` of `ceo@cymbal-bank.example` while its envelope `MAIL FROM` is an entirely different domain that legitimately passes its own SPF; SPF will pass, and the visible `From:` will still be a lie. SPF proves *"the host that sent this was authorized to use the envelope domain it claimed."* It does not prove the visible `From:` domain.

### 3.4 DKIM — What It Signs (Selected Headers and the Body)

**DomainKeys Identified Mail (DKIM)** is defined in RFC 6376 (September 2011, Standards Track, obsoletes RFC 4871 and RFC 5672) ✅. DKIM attaches a cryptographic signature to a message. The signature associates the `d=` **Signing Domain Identifier (SDID)** with a hash of the message ✅ (RFC 6376 §2.5). The signature header carries, among others:

- `d=` — the SDID: "the SDID claiming responsibility for an introduction of a message into the mail stream" ✅ (RFC 6376 §3.5).
- `h=` — the list of **signed header fields** ✅ (RFC 6376 §3.5).
- `bh=` — "the hash of the canonicalized body part of the message" ✅ (RFC 6376 §3.5).

So DKIM signs **a selected set of header fields plus the body**. It proves that *the domain named in `d=`* signed those fields and that body, and that they have not changed since (subject to the canonicalization rules). RFC 6376's Abstract states the design intent with precision: DKIM "separates the question of the identity of the Signer of the message from the purported author of the message" ✅ (RFC 6376 Abstract, September 2011).

Two consequences follow:

1. **DKIM proves *a domain* took responsibility — not *which* domain the reader sees, and not the author.** If `d=` is the signer's domain and the visible `From:` is a different domain, DKIM passes on the signer without vouching for the visible author. DKIM's own integrity depends on whether the signed `h=` set *includes* the fields you care about: a signature that does not cover `From:` proves nothing about `From:`.
2. **DKIM survives transit** because relays preserve the message unchanged ✅ (RFC 6376 Abstract: relays "typically make no substantive change to the message content and thus preserve the DKIM signature"). But a gateway that rewrites a header listed in `h=` **invalidates the signature**. This is why gateways that modify mail are dangerous for authentication (§2.2, §6).

### 3.5 DMARC — The Alignment Requirement

**DMARC** is defined in RFC 7489 (**March 2015**) and — this must be stated clearly — it is **Informational, not Standards Track** ✅ (RFC 7489 header: "Category: Informational"; and its own Status of This Memo section says it is "published for informational purposes"). DKIM, SPF, STARTTLS and MTA-STS are Standards Track; DMARC is not. That status does not diminish DMARC's practical importance, but an accurate guide states it.

DMARC's contribution is **Identifier Alignment**. RFC 7489's Abstract describes it as a mechanism by which "a mail-originating organization can express domain-level policies and preferences for message validation, disposition, and reporting" ✅. The alignment rule is the core:

> "When the domain in the RFC5322.From address … [must] match (be aligned with) an Authenticated Identifier." ✅ (RFC 7489 §3.1)

In other words: DMARC takes an identifier that SPF or DKIM **actually authenticated** (the envelope domain SPF checked, or the `d=` domain DKIM signed) and requires it to **align** with the domain in the visible `From:` header. Alignment can be *relaxed* (same organizational domain) or *strict* (exact domain) ✅ (RFC 7489 §3.1). A message is a DMARC "pass" if a DKIM signature is aligned or an SPF result is aligned ✅ (RFC 7489 §3.1).

**Why alignment matters:** without DMARC, SPF and DKIM each authenticate *something*, but that something may have no relationship to the domain the human reader sees. DMARC is the tie that binds an authenticated identifier to the visible `From:` domain. It is the piece that makes "the email came from our domain" a defensible statement — *for the domain in `From:`, and only to the extent the underlying SPF or DKIM check passed and was aligned.*

DMARC also defines a **policy** (`p=`) telling receivers what to do when alignment fails — from no action, through quarantining, up to rejecting — and a **reporting** mechanism ✅ (RFC 7489 Abstract; §6.3 describes the published record). The policy is what §4 uses to stop brand spoofing: an enforced DMARC policy tells the world's receivers to reject messages that claim the bank's `From:` without aligning to an authenticated bank identifier.

### 3.6 The Honest Table — What Each Layer Proves, and What None Prevents

| Layer | What it authenticates / signs | What it proves | What NONE of them prevents |
| --- | --- | --- | --- |
| SMTP envelope (`MAIL FROM`, `RCPT TO`) ✅ RFC 5321 | The transport routing addresses; the return path and the actual envelope recipients | Which addresses the message was routed to and from at the transport layer — *not* what the header claims | Nothing by itself: the envelope is unauthenticated data. Anyone can set `MAIL FROM` to any domain. |
| Header (`From:`, `To:`, `Received:`, …) ✅ RFC 5322 | The message's own claims | What the sender *asserts* — its claimed author, date, and hop history | Nothing by itself: header fields are claims, unverified by the format. |
| **SPF** ✅ RFC 7208 | The **envelope** sender (`MAIL FROM`) and `HELO` identities | That the connecting host was authorized to use the envelope domain it claimed | Spoofed display name; spoofed `From:` header (when envelope ≠ header); look-alike domains; compromised-but-authorized hosts; forwarded mail that breaks the path. |
| **DKIM** ✅ RFC 6376 | **Selected header fields (`h=`) + body (`bh=`)**, attached to the domain in `d=` | That the domain named in `d=` signed those fields and that body, and they are unchanged | Fields not listed in `h=`; spoofed display name (unless covered); a compromised signing key; the visible `From:` if it is not the `d=` domain and is unsigned; forwarding that rewrites a signed field. |
| **DMARC** ✅ RFC 7489 (Informational) | **Alignment** between an authenticated identifier (SPF or DKIM) and the visible `From:` domain | That the visible `From:` domain is *aligned* with something actually authenticated — and lets the domain publish what receivers should do on failure | Spoofed **display name** (the human-readable part before the address); a **look-alike domain** that is itself legitimately authenticated; a **compromised-but-legitimate mailbox** that genuinely sends from the bank; **unsigned forwarded mail** whose signature broke in transit; fraud that never spoofs at all. |

**The sentence a reader should be able to say after this section:** *"The email came from our domain" can mean four different things — it was routed from an envelope address in our domain, it claims our domain in the header, a host authorized by our domain sent it, and a domain aligned with our visible `From:` domain signed it. Only DMARC enforcement makes the last two hold together, and even then it cannot stop a spoofed display name, a look-alike domain, a compromised mailbox, or an unsigned forwarded message.* That is why §4 (outbound) and §5 (inbound) exist.

### 3.7 Where the Two Diverge — a Worked Anatomy

Consider a message that a fraudster wants a bank's treasury operator to believe came from the bank's own CEO. There are several ways to construct it, and the divergence between envelope and header is the tool:

| Construction | Envelope `MAIL FROM` | Visible `From:` | SPF | DKIM (`d=`) | DMARC alignment | Outcome |
| --- | --- | --- | --- | --- | --- | --- |
| Honest mail | `@cymbal-bank.example` | `ceo@cymbal-bank.example` | pass | pass (`d=cymbal-bank.example`) | aligned ✅ | Legitimate. |
| Envelope/header mismatch | `@spammer.example` | `ceo@cymbal-bank.example` | pass (for *spammer*), fail (for cymbal) | none or `d=spammer.example` | **fail** if DMARC enforced ✅ | Rejected at an enforcing receiver. |
| Look-alike domain | `@cymbai-bank.example` (note the `i` for `l`) | `ceo@cymbai-bank.example` | pass (its own domain is authorized) | pass (`d=cymbai-bank.example`) | **aligned for the look-alike** | **Passes** — DMARC cannot help; the domain is genuinely authenticated, just wrong. |
| Display-name spoof | any domain | `"Cymbal Bank CEO" <attacker@freemail.example>` | pass | pass (freemail's own) | aligned for freemail | **Passes** — the *domain* is honest; the *display name* is the lie. |
| Compromised mailbox | `@cymbal-bank.example` | `ceo@cymbal-bank.example` | pass | pass | aligned ✅ | **Passes** — the mailbox really is the CEO's; DMARC is irrelevant to an authorized sender. |

The fourth and fifth rows are the ones that keep mail-security teams awake, and neither is a protocol failure. The look-alike domain is a *registration* problem as much as a mail problem; the compromised mailbox is an *identity* problem. Only process controls — the out-of-band verification of §5, and user awareness — reach them. No authentication layer in §3.3–§3.5 will.
---

## 4. The Outbound Problem — the Bank as a Sender

### 4.1 The Bank as a Sender Has Two Jobs

When a bank sends mail, it has two distinct jobs, and confusing them is a compliance failure in itself.

1. **Deliverability.** The message must actually reach the legitimate recipient — the customer, the counterparty, the regulator — rather than being filtered as spam or rejected. This is an *operational* job: the bank's sending reputation and authentication determine whether the world's receivers trust its mail.
2. **Brand protection.** Messages that *fraudulently* claim to come from the bank must be rejected by the world's receivers before they reach a victim. This is a *risk* job: the bank's domain is being impersonated, and the defence is a published, enforced authentication policy.

The two jobs share the same technical substrate — SPF, DKIM and DMARC (§3) — but they optimize for opposite things. Deliverability wants the bank's legitimate mail to *pass*; brand protection wants impostor mail to *fail*. A properly configured domain achieves both: legitimate mail passes because it is authenticated and aligned, while impostor mail fails alignment and is rejected under the published policy.

### 4.2 Deliverability and Domain Reputation

Deliverability rests on the receiving side's assessment of the sending domain and sending infrastructure. The bank cannot directly see that assessment; it can only publish authentication, send consistently, and monitor. The relevant levers, all grounded in the RFCs verified in §3:

- **SPF** (RFC 7208, April 2014) ✅ tells receivers which hosts may use the bank's envelope domain. An SPF record that omits a legitimate sending host causes that host's mail to fail SPF and, if DMARC is aligned and enforced, to be *rejected* — a self-inflicted deliverability and compliance problem.
- **DKIM** (RFC 6376, September 2011) ✅ cryptographically signs selected headers and the body, allowing a receiver to verify the bank's domain took responsibility and the content is unchanged. Effective DKIM also improves the survivability of a message through intermediaries that would otherwise flag it.
- **DMARC** (RFC 7489, March 2015, Informational) ✅ publishes the policy and the alignment requirement, and its reporting mechanism gives the domain owner feedback about who is sending as the domain.

Domain reputation is *accumulated behaviour*: consistent authentication, low complaint rates, stable sending infrastructure. A bank whose authentication breaks intermittently — a rotated key not published, a new sending host missing from SPF — earns an inconsistent reputation, and its mail is handled less favourably at the receivers that matter. This is not a compliance rule with a citation; it is a structural consequence of how receivers assess trust. ⚠ (structural inference, not a dated standard).

### 4.3 The Brand-Spoofing Problem DMARC Enforcement Addresses

The problem DMARC enforcement addresses is precisely the one §3.6 identified: without an *enforced, aligned* policy, a fraudster can send a message with the bank's domain in the visible `From:` and have it accepted, because SPF and DKIM each authenticate *something* that need not be the visible `From:`. DMARC's **alignment** requirement closes that specific gap: it requires the visible `From:` domain to match a domain the transport actually authenticated ✅ (RFC 7489 §3.1, March 2015).

The **policy** (`p=`) is the enforcement lever: `p=none` monitors and reports but takes no action on failure; `p=quarantine` asks receivers to treat failing mail with suspicion; `p=reject` asks receivers to refuse it ✅ (RFC 7489 Abstract describes the range "from no action, through altered delivery, up to message rejection"). A bank that publishes `p=none` and never advances to enforcement has built a *monitoring* posture, not a *protective* one — it learns who is spoofing but does not stop them. Moving to enforcement is a decision with real deliverability risk (legitimate third-party senders misaligned will be rejected), so the prudent sequence is: monitor, inventory every legitimate sender, ensure each is aligned, then enforce. That sequence is a governance project, not a DNS edit.

**The honest boundary:** DMARC enforcement addresses spoofing of the *domain in `From:`. It does not address a spoofed display name, a look-alike domain that is itself validly authenticated, or a compromised legitimate mailbox (§3.6) ✅ (inference from RFC 7489 §§3.1, 10.4). A bank that believes "we enforce `p=reject`, so our customers cannot be phished as Cymbal Bank" has overstated the control. Enforcement reduces one class of impersonation; it does not eliminate impersonation.

### 4.4 Transactional versus Marketing — Without Asserting Any Consent Rule

A bank's outbound mail splits, structurally, into two populations:

- **Transactional mail** — messages tied to a specific customer transaction or account event: an account statement, an alert on a large debit, a payment confirmation, a password reset, a regulatory notice. These are functional communications of a service relationship.
- **Marketing / promotional mail** — messages promoting products or services: offers, campaigns, newsletters.

The two populations differ in ways that bear on *how the mail estate is governed*, and these differences are real and useful to state:

| Dimension | Transactional | Marketing |
| --- | --- | --- |
| Trigger | An event in the customer's account or relationship | A campaign decision |
| Typical recipient expectation | Expected as part of the service | Depends on the relationship |
| Deliverability sensitivity | High — a statement that fails to deliver is an operational failure | High but campaign-tolerated |
| Authentication requirement | Same (SPF/DKIM/DMARC alignment) for both | Same for both |
| Volume pattern | Correlated with account activity | Bursty, campaign-driven |

**What this guide does NOT do: it asserts no consent rule, no opt-in/opt-out requirement, and no jurisdiction-specific marketing-communication obligation.** Whether and how a jurisdiction requires consent for marketing mail is a matter for that jurisdiction's law, and this guide has not verified any such rule against a dated primary source (see §15, §16). The transactional-versus-marketing distinction is presented here *only* as an operational and governance split that shapes deliverability monitoring, sending-infrastructure choice, and the classification the archive must record. A bank should source its consent obligations to the specific regulator publication in each jurisdiction it operates in; this guide deliberately does not supply them.

### 4.5 What the Outbound Estate Must Prove

For a regulated sender, the outbound estate must be able to *produce* evidence that its authentication was correctly configured and enforced — not merely assert it. Concretely, the institution should be able to show, on request: the published SPF records and the hosts they cover; the DKIM keys and the fields each signature covers (the `h=` set); the DMARC policy and its alignment mode; and the change history of each. This is a records obligation about the *configuration* of authentication, separate from the records obligation about the *messages* (§7). It is easy to forget, because DNS records are not mail, but a supervisor asking "show me that your domain was protected in the period under review" wants the configuration as it stood, not as it stands today.

---

## 5. The Inbound Problem — the Bank as a Target

### 5.1 The Bank as a Target

If §4 is about the bank sending, §5 is about the bank receiving, and the threat is *directed at the bank's own processes*, not at its brand. The inbound attacker is not trying to reach customers; the attacker is trying to get a bank employee to do something — release a payment, change a beneficiary, grant access, disclose data. The message is the delivery mechanism; the employee is the target; the transfer is the prize.

This guide treats the fraud *pattern* as it manifests in the email channel. It deliberately does **not** re-derive fraud detection methods, scoring, or the broader fraud-technology landscape; for that, read [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md).

### 5.2 Business Email Compromise

**Business email compromise (BEC)** is the family of attacks in which the adversary uses email — through impersonation, a compromised mailbox, or a look-alike domain — to induce an authorized employee to take a business action that benefits the adversary. The defining characteristic is that **the attacker does not need to break the bank's perimeter; the attacker needs only to convince a human.** §3.6 already established *why* this works: every authentication layer in §3 can be satisfied by a look-alike domain or a compromised mailbox, so a technically well-configured mail estate does not stop BEC.

BEC therefore belongs partly to the *protocol* story and mostly to the *process* story. The protocols can be hardened (they should be, §4, §6); but the decisive control is a process control on the receiving end, and it is §5.5.

### 5.3 The Spoofed Instruction

The core inbound pattern is the **spoofed instruction**: a message that appears to come from someone with authority — a manager, a treasury officer, a partner bank, a supplier — and that instructs an employee to act. The pattern's variants include impersonation of an executive (a request framed as urgent and confidential), impersonation of a counterparty (a settlement instruction), and impersonation of a supplier (a change of remittance details — §5.4). The common structure: an instruction, an authority claim, and pressure toward speed.

RFC 5322 provides no mechanism to prevent the authority claim in the header ✅ (§3.2) and the authentication layers of §3 authenticate *identities*, not *authority*. The lesson: **an email cannot carry authority.** Authority is a property of the *process* that authorizes an action, and no message content can confer it. An employee who acts on an instruction *because the email says so* has substituted a message for a process.

### 5.4 The Supplier-Email / Bank-Details Change

The most damaging and durable BEC variant in a corporate context is the **supplier-email / bank-details change**: a message, apparently from a known supplier or vendor, advising that the supplier's bank details have changed, and directing future remittances to a new account the attacker controls. The message is often well-crafted, references real invoices or real prior correspondence, and may arrive from a look-alike domain or a compromised supplier mailbox — both of which can satisfy ordinary authentication. §3.6's look-alike and compromised-mailbox rows are precisely this attack.

Why it is durable: the change is *requested*, not *taken*. The bank's system performs exactly as designed — the bank updates the beneficiary record and pays the supplier — and the loss materializes only after the supplier reports non-payment. The fraud hides inside a legitimate workflow. This is why the pattern recurs and why technical mail hygiene alone does not stop it.

### 5.5 THE ONE CONTROL THAT MATTERS — Out-of-Band Verification

If this guide has one operational control to carry away, it is this:

> **Any request to change payment details — a bank account, a beneficiary, a remittance route — must be verified out of band, by a call-back to a previously-known, independently-sourced telephone number, before the change is made. Not by replying to the email. Not by calling a number supplied in the email.**

The control is a *process* control, and it is deliberately specified to close the specific gaps §3.6 identified:

- **"Out of band"** — using a channel other than the one that carried the request. A reply to the email travels the same channel the attacker controls or monitors; it confirms nothing. ✅ (structural; the correspondence-channel weakness is the reason the control exists).
- **"A previously-known number"** — a number the institution already holds for the supplier from a source independent of the request: a contract, a prior verified contact record, an onboarding document. A number supplied *inside* the suspicious message is attacker-controlled and worthless as verification.
- **"Before the change is made"** — the control must gate the change, not audit it afterward. A callback performed after payment has been released is a post-mortem, not a preventive control.

The control is not exotic. It is the deliberate rewriting of a workflow so that a payment-detail change cannot be executed on the strength of a message. Its cost is a phone call; its absence is the loss. **The single most valuable sentence in a bank's payment-change procedure is: "We will call you back on the number we already have."**

Two corollaries make the control robust:

1. **The control must be enforced by the system, not by the individual's vigilance.** A procedure that relies on an employee *remembering* to call back will eventually fail; the payment system should *refuse* to process a detail change that lacks a recorded, out-of-band verification event, and that verification should be an auditable record in its own right.
2. **The verification record is itself a compliance record.** Just as the mail is captured (§7), the fact that a change was verified — who called whom, on which number, when, and against which request — must be captured and retained, because it is the evidence that the control operated.

§12's failure modes and §13's worked example both return to this control, because in the real world it is the control that is most often *believed* to exist and least often *verified* to exist.

### 5.6 What the Inbound Estate Must Prove

The inbound problem, like the outbound (§4.5), carries a proof obligation. A bank should be able to demonstrate: that its inbound authentication and filtering were operating (from the gateway's own logs, §6); that payment-detail changes require and record out-of-band verification (from the payment system's records, §5.5); and that suspect inbound messages are captured as records (**§7 — because a phishing message delivered into a bank is itself evidence**). That last point is easy to miss: an inbound attack message is not merely a security event to be blocked; if it induced or attempted to induce a fraudulent action, it is a *record* of an attempted fraud, and a compliance function may need to produce it (§10). Blocking a message is a security success; retaining the evidence of the attempt is a compliance necessity.

---

## 6. The Gateway and Platform Controls

### 6.1 SMTP Gateway Placement

The **gateway** is the MTA at the boundary between the Internet and the internal mail system, and it is where the institution's mail policy is actually enforced. RFC 5321 describes the gateway role as the system "at the boundary between two transport service environments" ✅ (RFC 5321 §2.3.10, §3.7). Placement choices determine what the institution can see and control:

- **Perimeter inbound gateway** — sits between the Internet and the internal MTAs, inspecting mail *arriving at* the institution. This is where inbound filtering, anti-spoofing, and BEC indicators are evaluated.
- **Perimeter outbound gateway** — sits between internal senders and the Internet, inspecting mail *leaving* the institution. This is where outbound DLP, encryption policy, and authentication application occur.
- **Internal gateways** — between internal mail domains or segments. These control internal-only flows and are where an internal journaling gap can open if not accounted for (§7).

The compliance-relevant consequences of placement: **mail that does not traverse a controlled gateway is mail the institution may never inspect, authenticate, or capture.** Any path that bypasses the perimeter — a direct SMTP path from an application server, a personal account, a mobile device sending via a third party — is both a control gap and a records gap. §7's completeness question and §12's archive-gap failure mode exist because of this.

### 6.2 Inbound and Outbound Content Inspection

Content inspection is the gateway evaluating message content against policy. Two directions, two purposes:

- **Inbound inspection** looks for malicious or unwanted content and for attack indicators: spoofing signals, known-bad link patterns, malware attachments, impersonation heuristics, and the BEC markers of §5.
- **Outbound inspection** looks for content that should not leave the institution: sensitive data (personal data, account data, credentials), policy-violating content, and unapproved destinations. This is the mail-channel face of the **data-loss-prevention (DLP)** discipline. This guide does not re-derive DLP; the data classification, handling and protection model is set out in [Data Governance](../technology/data/data_governance_framework.md), and the broader security-programme context in [Cybersecurity](../technology/cybersecurity_guide.md).

A compliance subtlety: **inspection must not destroy the record.** A gateway that rewrites, strips, or quarantines content is performing an action on the message that the archive must be able to account for. If the archive captures only the post-inspection message, the record omits what the sender actually sent; if it captures only the pre-inspection message, it omits what the recipient actually received. The institution should be able to reconstruct both and to know which actions the gateway took. This is a design requirement, not an RFC requirement ✅ (inference from RFC 5321 §3.7.1's acknowledgement that gateways rewrite, October 2008).

### 6.3 Transport Encryption — STARTTLS, MTA-STS, TLS for Submission and Access

Transport encryption for mail rests on a small set of verified RFCs:

- **STARTTLS** — RFC 3207 (February 2002, Standards Track, obsoletes 2487) ✅ defines the SMTP service extension that "allows an SMTP server and client to use TLS … to provide private, authenticated communication over the Internet," giving agents "the ability to protect some or all of their communications from eavesdroppers and attackers" ✅ (RFC 3207 Abstract). **Important limitation:** STARTTLS in its base form is *opportunistic* — a sender offers it, the peer may accept or not, and a network attacker can strip the offer (a downgrade), leaving the message in the clear. STARTTLS alone does not guarantee encryption.
- **MTA-STS** — RFC 8461 (September 2018, Standards Track) ✅ addresses that gap: it "enabl[es] mail service providers to declare their ability to receive TLS secure SMTP connections and to specify whether sending SMTP servers should refuse to deliver to MX hosts that do not offer TLS with a trusted server certificate" ✅ (RFC 8461 Abstract). In plain terms, MTA-STS lets the receiving domain *require* the sender to use validated TLS, by policy published over HTTPS.
- **TLS for submission and access** — RFC 8314 (January 2018, Standards Track, updates 1939, 2595, 3501, 5068, 6186, 6409) ✅ gives the recommendations for TLS between the mail user agent (the client) and the mail submission or access server. RFC 8314's title is telling: *Cleartext Considered Obsolete*. The recommendation is that the MUA-to-server legs be encrypted, not opportunistic.
- **Authentication-Results** — RFC 8601 (May 2019, Standards Track, obsoletes 7601) ✅ defines the header field that a receiver inserts to record "the results of message authentication efforts," so that the SPF/DKIM/DMARC outcomes are themselves carried in the message as data the downstream systems — including the archive — can read.

**The compliance reading:** an institution that carries mail over unencrypted transport is exposing message content in transit, and message content is potentially personal data and potentially confidential. The controls exist (STARTTLS, MTA-STS, TLS submission/access) and are Standards Track ✅; whether a given institution is required to deploy them is a matter for its own regulatory and data-protection obligations, which this guide does not assert. What can be stated structurally: **opportunistic encryption without MTA-STS is not a guarantee**, and an institution should understand which of its mail paths can be downgraded.

### 6.4 Link and Attachment Handling

Links and attachments are the two payload carriers of inbound mail, and gateway handling of them is a core control:

- **Link rewriting / sandboxing** — the gateway replaces external URLs with rewritten, institution-controlled URLs so that a click is evaluated before it reaches the destination, or detonates the destination in isolation. This protects the user but *changes the message*: a rewritten link is not the link the sender sent, so the record must preserve the original (§6.2's reconstruct-both requirement).
- **Attachment handling** — scanning, detonation in a sandbox, stripping of prohibited types, and quarantine. Like link rewriting, attachment handling acts on the message and must be accounted for in the capture path.
- **Quarantine** — holding suspect mail out of user mailboxes. From a records standpoint, quarantined mail is *still captured mail*: the fact that a message was quarantined is an event the institution should be able to evidence, and the message itself may be needed as a record of an attempted fraud (§5.6).

### 6.5 The Gateway as a Records Source

The gateway is not only a control point; it is a **records source**. Its logs of what it accepted, rejected, quarantined, rewrote, and forwarded are the raw material for the completeness reconciliation of §7.4 and for the production of §10. An institution that treats its gateway purely as a filter and not as a journaling record-keeping system has thrown away the very data it will need to prove its archive is complete. The gateway's logs should be treated as compliance records in their own right, with their own retention and integrity requirements.

### 6.6 The Governance Layer

The controls above are enabled by an identity, key, and certificate governance layer: SPF records, DKIM keys, DMARC policies, TLS certificates for MTA-STS and for the submission/access servers. Each has a lifecycle — issuance, distribution, rotation, revocation — and each lifecycle event is a potential failure point (§4.2, §12). RFC 6376 §3.6.4 discusses the removal of a DKIM key record after a transition period ✅, which is a concrete example of why key governance must be planned: a key rotated out while mail signed with it is still in flight, or a key still published after a compromise, are both failure modes. The institution should treat these as governed assets with owners, not as DNS entries someone edited once.

---
---

## 7. The Record — Envelope, Message, Journal, WORM, and Completeness

### 7.1 What Must Actually Be Captured — the Envelope AND the Message

The core claim of this guide is that a bank's mail is a records system. The core requirement of that records system is: **capture the envelope and the message — not one of them.**

Capturing **only the message** (the header plus body, as a user sees it) loses:

- **`BCC` recipients** — present in `RCPT TO` on the envelope, absent from the header ✅ (§3.1).
- **Distribution-list membership** — the header may name the list; the envelope names the actual recipients after expansion ✅ (RFC 5321 §3.10 discusses list expansion).
- **The actual delivery path** — which systems the message was offered to and by which it was relayed, reconstructible from the envelope transaction and the `Received:` chain.
- **The envelope sender** — the reverse-path, which is what error notifications used, and which may differ from the visible `From:`.

Capturing **only the envelope** (the transaction metadata) loses:

- **The content** — the body, the subject, the attachments: the substance of the communication.
- **The header claims** — `From:`, `Date:`, `Message-ID:`, and the rest, which are what the message asserted.

A faithful record therefore needs **both**, linked. The envelope answers *who actually sent and received this, and by what path*; the message answers *what was said, and who it claimed to be from*. A discovery request (§10) and a completeness reconciliation (§7.4) both require the pair.

A practical enumeration of what a compliant capture should hold per message:

| Element | Source | Why it must be captured |
| --- | --- | --- |
| Envelope sender (`MAIL FROM`) | SMTP transaction | The actual return path; may differ from `From:`. |
| Envelope recipients (`RCPT TO`) | SMTP transaction | Includes `BCC` and expanded list members. |
| Header | Message | The claims: `From:`, `To:`, `Date:`, `Subject:`, `Message-ID:`, `Received:` chain. |
| Body and attachments | Message | The substance of the communication; attachment names and hashes aid integrity. |
| Delivery path / timestamps | MTA and `Received:` fields | Reconstructs the route and the time at each hop ✅ (RFC 5321 §4.4). |
| Authentication results | `Authentication-Results` (RFC 8601, May 2019) ✅ | Records SPF/DKIM/DMARC outcomes as data. |
| Journal/envelope identifiers | Mail server | Links the captured copy to the transit event for reconciliation (§7.4). |

### 7.2 How Journaling Works as a Mechanism

**Journaling** is the mail-server feature by which, as a message transits the server, the server emits **a copy of the message** to a designated journal recipient or journal mailbox. The copy is then ingested into the archive. It is important to be precise about what journaling is and is not:

- **It is a mail-server feature, not a protocol feature.** No RFC defines journaling. SMTP (RFC 5321) knows nothing of it; the mail server implements it by, in effect, *also* sending a copy of every message it handles to the journal address. Because it is a vendor feature, its completeness properties are vendor-specific and must be verified per platform, not assumed from the protocol. ⚠ (no RFC defines journaling).
- **The journal copy is, from the server's point of view, just another message.** The server delivers it to the journal destination the same way it delivers any other mail. This is the crucial architectural fact: **if the journal copy is delivered by mail, then the journal path is itself a mail path subject to the same delivery, filtering, and failure properties as any other mail path.** A journal that silently fails to deliver — a full mailbox, a broken filter rule, a route change — produces a silent gap, because the message transited normally and only the *copy* failed.

There are broadly two journaling architectures, and the choice matters for completeness:

1. **Journal-at-the-hub** — journaling enabled on the central MTAs that all mail must traverse. Simpler, but complete only if *all* mail actually traverses those MTAs (the archive gap of §12 in disguise).
2. **Journal-everywhere** — journaling enabled on every mail-handling system, including the ones at the edges. More complete coverage, but harder to reconcile, because now there are many sources and each must be monitored.

A bank should know, per system that handles mail, whether journaling is enabled, where the copy goes, and what happens when the copy cannot be delivered. A journaling configuration that nobody can enumerate is a completeness claim nobody can support.

### 7.3 What a WORM Store Does and Does NOT Guarantee

The captured copies must be stored in a way that resists tampering. **WORM** — Write Once, Read Many — is the property that a record, once written, cannot be rewritten or erased before its retention expiry. It is a property of the *storage medium*, not of the capture path ⚠ (a vendor/storage property; no protocol or regulator source is asserted here). What WORM gives you and what it does not:

**What WORM does guarantee:**

- **Immutability of a stored record.** A record already in the store cannot be altered or deleted out-of-band before its retention expiry, so an actor cannot silently rewrite the past. A record read back is the record that was written.
- **Tamper-evidence over time.** Combined with integrity hashing, a WORM store lets the institution demonstrate that a produced record is the record as captured.

**What WORM does NOT guarantee:**

- **Completeness.** Immutability says nothing about whether *everything that should have been captured was captured*. A perfectly immutable archive that is missing 5% of the mail is a perfectly immutable incomplete archive. This is the single most important limitation and the reason §7.4 exists.
- **Correctness of capture.** If the capture path recorded the wrong thing — a truncated copy, a message missing its envelope, a journal copy that was itself malformed — WORM dutifully preserves the error forever.
- **That the record is the *real* message.** A WORM store holds what it was given. If what it was given was, say, a post-rewrite gateway version, WORM preserves the rewrite. WORM is integrity of storage, not authenticity of content.
- **That a hold can be imposed.** WORM does not implement legal hold; legal hold is a separate control (defined in §8) that must sit *above* the deletion schedule, and a WORM store whose deletion is schedule-driven needs the hold to suspend that schedule.

The one-sentence formulation: **WORM prevents rewrite; it does not prove completeness or correct capture.** An institution that equates "we have WORM storage" with "we have a compliant archive" has confused two different guarantees.

### 7.4 The Completeness Question — How Would the Institution Know the Archive Is Missing Something?

This is the hardest question in the guide, and the one most often unasked. **A WORM archive cannot tell you what it does not contain. It is silent about its own gaps.** The only way to know the archive is missing something is to *compare it against an independent count of what should have been there* — reconciliation, not inspection.

The available reconciliation strategies, in order of strength:

1. **Journal-to-gateway/ MTA message-count reconciliation.** Count the messages that the gateway and the MTAs processed in a period, and count the messages the archive received for the same period. The two counts must agree within a known tolerance (some messages are legitimately dropped as spam, some are duplicated by relay). A persistent discrepancy is an archive gap. This is the primary control and requires the gateway/MTA logs of §6.5 as an independent counter.
2. **Per-sender / per-recipient reconciliation.** Break the counts down by sending system or by recipient domain. A gap that a global count masks (one path up, another down) becomes visible when disaggregated. This catches the specific failure where one mail path bypasses the journal while others do not (§12).
3. **Sampling and spot-checks.** Periodically select known messages — a test message sent through each path, or a real message traced end to end — and confirm each appears in the archive. Sampling catches *types* of gap that aggregate counts miss (e.g. a particular attachment type dropped).
4. **Alerting on journal-path failures.** Because the journal copy is delivered as mail (§7.2), a failure to deliver it is detectable. The institution must *monitor* the journal path as a critical mail path and alert on any delivery failure, mailbox saturation, or route error. A journal path nobody watches is a gap generator.
5. **Monitoring the journal account itself.** The journal destination (mailbox or API endpoint) must be monitored for health, capacity, and ingestion success — because a journal account that has silently stopped accepting copies is the archetypal archive gap: everything looks fine at the mail layer, and the archive simply stops growing.
6. **Ingestion reconciliation at the archive boundary.** The archive should record its own ingest counts and reject/error counts, and those should reconcile with the journal path's emitted counts. Any message the journal emitted but the archive did not ingest is a gap that this control alone can catch.

**The design principle:** completeness is proven by *comparing two independent counts*, never by querying the archive for what it holds. The archive's own index can only enumerate what it has; the gap lives in the difference between what it has and what it should have. An institution that cannot name its reconciliation control and its tolerance does not know whether its archive is complete — it only knows whether the archive *runs*.

§12's "THE ARCHIVE GAP" failure mode is the concrete version of this, and §13's worked example finds exactly this in the Cymbal estate.

---

## 8. Retention, Legal Hold and Disposition

### 8.1 The Shape of the Obligation

A regulated institution is expected to **capture, retain, and be able to produce** its communications, including email. That is the *shape* of the obligation, and it is stated here structurally: the obligation is a triangle of capabilities — capture (get it, including envelope and message, §7), retain (keep it, immutably, for as long as required), and produce (retrieve and deliver it under a discovery or supervisory request, §10). An institution missing any leg is non-compliant even if the other two legs are strong.

**This guide deliberately does not state a retention period, and does not name a specific jurisdiction's rule, as a requirement.** The reason is stated plainly: the retention period is **jurisdiction-specific and rule-specific**, and this guide did not retrieve a dated primary publication that establishes any specific period. Printing "keep email for N years" without a dated source would be exactly the kind of unsourced claim §15 forbids. What can be said structurally:

- Different jurisdictions and different regulatory regimes impose different retention expectations; a single global number would be wrong somewhere.
- The binding period for any given institution is a function of the jurisdictions it operates in and the regimes that apply to it — a determination its compliance and legal functions must make against the actual primary texts, not against this guide.
- The applicable period may vary by *type* of communication, by *business line*, and by *whether a hold or a matter is open* (a hold can extend retention indefinitely, §8.2).
- Where the repository's existing material cites a **vendor** archiving line (for example, [StorageGRID](../technology/storagegrid_guide.md) mentions a vendor email-archiving table with "7+ years"), that is a **vendor product statement**, not a regulatory rule, and this guide explicitly does **not** upgrade it into one. A vendor's suggested configuration is evidence of what the product supports, not of what the law requires.

An institution should derive its retention schedule from the primary sources applicable to it, record the derivation, and hold that schedule as a governed document — because it is the schedule that the disposition engine executes (§8.3), and an unsourced schedule is an unsourced deletion.

### 8.2 Legal Hold — the Suspension of an Otherwise-Running Automated Deletion

**Legal hold is the suspension of an otherwise-running automated deletion process, for a defined set of custodians and subjects.** That definition is the whole of it, and it repays close reading because each clause is a failure point:

- **"Suspension"** — the normal state of an archive is *deletion on schedule*. Records age; the retention schedule expires; the disposition engine deletes them. A legal hold *stops* part of that process. It does not add a capability; it interrupts an existing one. This is why a hold must be implemented *above* the deletion schedule — if deletion is not actually gated by the hold, the hold is a note in a file, not a control.
- **"Otherwise-running"** — the deletion process is automated and continuous. It is deleting *right now*, somewhere, for records whose periods have run. A hold that takes effect "by the end of the quarter" may be too late: the records may already be gone. Recency and immediacy matter.
- **"Automated"** — because the process is automated, the hold must be enforced *in the automation*. A hold implemented by asking an administrator to remember not to run a job is not a control; it is a hope.
- **"Defined set of custodians and subjects"** — a hold is scoped: specific people, specific matters, specific date ranges or search terms. An under-scoped hold leaves records deletable; an over-scoped hold (the "hold everything forever" reflex) is safe but destroys searchability and is itself a governance failure. Scoping must be a deliberate, recorded act.
- **"Process"** — a hold is a *process*, not a click: a hold notice is issued, acknowledged, and applied; the population is identified; the hold is recorded; and the hold is eventually released by another deliberate act. Each step has an audit trail.

**The fragility of a hold that depends on an individual remembering.** The classic failure (a §12 entry) is a hold whose application depends on a human's memory or diligence: an investigator learns of a matter and forgets to apply the hold; an administrator applies it to the wrong custodian set; a system change resets the hold flags. Because the deletion engine does not know a matter exists, it does not pause for it. The only robust design is one in which *the deletion engine itself cannot delete a record covered by an active hold*, and in which holds are applied and released through a recorded workflow. The test an institution should run: **can I produce the audit record showing, for a given matter, exactly which records were placed on hold, when, by whom, and that none were deleted afterward?** If the answer is no, the hold is fragile regardless of intention.

### 8.3 Disposition as an Auditable Event

**Disposition is the final, deliberate deletion of a record whose retention period has run and which is not covered by any hold.** Its compliance character is that it is an **event**, not a silence:

- **It is logged.** Every disposed record (or disposed range/classification) should be recorded: what was disposed, when, under which expiry rule, by which process, and with evidence that no hold applied. "We deleted it per schedule" is only a defence if the deletion is evidenced.
- **It is gated.** The disposition engine must check, at the moment of disposition, that no hold covers the record. A disposition that proceeds without that check is a hold failure waiting to happen (§8.2).
- **It is irreversible and therefore deliberate.** Because WORM means deletion is a genuine destruction, the decision must be governed. An institution must be able to explain, for any disposed record, why it was safe to dispose of it at that time.
- **It is auditable after the fact.** The disposition log is itself a record with its own retention. Years later, when a matter arises, the institution may need to show that a missing record was *disposed of lawfully* rather than *lost* — and only the disposition log can distinguish the two.

The triangle of §8.1 closes here: capture without retention is pointless, retention without disposition is indefinite storage, and disposition without an audit trail turns lawful deletion into an unexplained absence.

---

## 9. Surveillance and the Lexicon

### 9.1 The Review Obligation, Structurally

Beyond capture and retention, a regulated institution with a communications archive is often expected to *review* the captured communications for the conduct and risk signals the institution is responsible for detecting — that is the structural role of a surveillance programme. This guide states the obligation **structurally**, not as a specific rule: the institution's *own* applicable regimes determine what must be reviewed, to what standard, and with what evidencing of the review. The guide does not assert any jurisdiction's specific surveillance requirement, because none was verified against a dated primary source in this pass (§15).

What can be said structurally about any communications surveillance programme:

- **It operates on a corpus.** The archive of §7 is its input; if the corpus is incomplete (§7.4), the surveillance is incomplete in exactly the same way and to exactly the same degree.
- **It searches for signals.** The signals come from a **lexicon** — a set of terms and patterns — combined with metadata rules (who, to whom, when, volume) and, increasingly, analytics (§9.3).
- **It produces alerts.** Each hit or pattern match becomes an alert requiring human review.
- **It consumes review capacity.** The alerts must be reviewed by people, and *the people are the binding constraint* (§9.2).

### 9.2 The Lexicon

A **lexicon** is the vocabulary the surveillance searches for: terms associated with the risk being monitored — organised-crime or sanctions-evasion terms, market-abuse terms, terms indicating bribery or corruption, and so on. Building and maintaining a lexicon is a governance exercise:

- **Coverage** — the terms must actually appear in the communications that matter, in the languages and idioms actually used. A lexicon that misses the terms the institution's own staff use is decorative.
- **Precision** — every term added increases false positives (§9.2 below the line). A lexicon tuned only for coverage (catch everything) produces an alert queue nobody can clear; a lexicon tuned only for precision (catch only the sure things) misses the phenomena. The tuning is a permanent, iterative negotiation, not a one-time configuration.
- **Metaphor and evasion** — the hardest problem is that people under scrutiny do not use the lexicon terms. They use euphemism, code, abbreviation, other languages, and other channels. A lexicon catches the unsubtle; it demonstrates a control; it does not by itself catch a determined evader. An institution that believes its lexicon *is* its surveillance has replaced supervision with a word list (§9.5).

### 9.3 Analytics

Beyond the lexicon, surveillance increasingly applies analytics: behavioural patterns, network analysis of who communicates with whom, sentiment/urgency markers, anomaly detection, and the use of machine learning (including, lately, large-language models) to triage or summarise the alert queue. These are treated here only at the level of *what they change structurally*:

- They can *increase recall* (find phenomena a lexicon would miss, like euphemism and relationship anomalies) but also *increase false positives* if not tuned, and they introduce model-governance questions — explainability, drift, and the evidencing of why an alert was or was not raised. The governance of AI/ML models in a compliance context is the subject of [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md), and this guide cross-references rather than re-derives it.
- They do **not remove** the review-capacity constraint. A model that triages the queue can *prioritise* it, but a human still must review the alerts the institution will act on. Automation shifts where the human attention is spent; it does not eliminate the human.

### 9.4 The Honest Cost — False Positives and Review Capacity

The most honest thing this section can say is the thing least often said: **a monitoring programme's true cost is not the software; it is the review capacity, and the enemy of review capacity is the false positive.**

- **The false-positive burden.** A lexicon with broad terms on a large corpus produces a large number of hits that are not real issues — ordinary business words in ordinary business contexts. Every false positive consumes reviewer time and, more damagingly, reviewer *attention*: a queue full of noise trains reviewers to skim, and skimming is how a real hit is missed.
- **The review-capacity arithmetic.** If the programme produces N alerts per day and a reviewer can genuinely review M per day, the programme needs at least N/M reviewers — and if N/M exceeds the staffing, the queue grows without bound or the review degrades to rubber-stamping. **An alert queue nobody can clear is not supervision.** It is a compliance-looking artefact that produces no detection.
- **The evidencing of review.** A programme should be able to show not merely that alerts were *raised* but that they were *reviewed* — with a recorded outcome, a rationale, and an escalation path for the ones that mattered. "The system flagged it" is not a review; "a reviewer assessed it and recorded this disposition" is.
- **The metric that matters.** The metric to watch is not the number of alerts generated (a vanity number) but the *reviewed-and-cleared/ escalated* throughput against the queue, and the age of the oldest unreviewed alert. A programme that measures only alert volume is measuring its own noise.

### 9.5 Surveillance Theatre

The failure mode adjacent to this section — developed fully as a §12 entry — is **surveillance theatre**: the substitution of surveillance *artefacts* (a lexicon, a monitoring platform, a dashboard, a policy) for actual supervision (review with capacity and disposition). The signature of theatre is a monitoring system that generates alerts nobody reads, a lexicon nobody tunes, and a dashboard nobody acts on. It is *worse than no monitoring*, because it manufactures a false assurance that the institution is watching. The antidote is §9.4's honesty about capacity and §9.6's discipline about what review produces.

### 9.6 Privacy and Employee Monitoring — Handled Generally

Communications surveillance touches personal data and, in many jurisdictions, employee-monitoring rules. **This guide asserts no jurisdiction's rule.** It states only the structural principle that follows from the records thesis: because the corpus (§9.1) is both a compliance asset and potentially personal data, the surveillance programme must be governed consistently with the institution's **data-protection and data-governance obligations** — purpose limitation, access control, retention of the surveillance records themselves, and the handling of the personal data incidental to the search. The applicable rules are jurisdiction-specific and must be sourced to the primary text by the institution's own legal and privacy functions; this guide does not supply them. The data-governance model is set out in [Data Governance](../technology/data/data_governance_framework.md) and the security controls in [Cybersecurity](../technology/cybersecurity_guide.md); the intersection of surveillance with employee data should be governed *by reference to those*, not by reference to folklore about what "monitoring" may do.

---
---

## 10. Discovery and Production

### 10.1 The Four Phases

**Discovery and production** is the transformation of the archive (§7) into a set of records delivered to a requesting party — a court, a litigant, a regulator, an auditor, an internal investigation — in a defensible way. It has four phases, and a failure in any one undermines the value of the other three.

1. **Collection** — identifying, from the archive, the set of records responsive to the request. The scope may be custodians (a set of people), a date range, a subject matter (search terms), or a combination. Collection is where the envelope of §7.1 earns its keep: a request that asks "who received a communication about X" cannot be answered from headers alone, because the recipients — including `BCC` and list members — live in the envelope.
2. **Review** — a human (or a human-supervised process) examining the collected records to determine responsiveness, privilege, and relevance, and to identify what will be produced and what will be withheld (with a stated basis). Review is where the personal data and privileged-communication questions of §9.6 are resolved for the specific production.
3. **Production** — delivering the responsive, non-withheld records to the requesting party in an agreed format, with the metadata that makes them usable as evidence. Production format matters: a set of message bodies without headers, or without envelope data, may be unusable if the requester needs to establish sender, recipient, or timing.
4. **Chain of custody** — the documented, unbroken record of how each produced item was collected, handled, reviewed, and delivered, and the assurance that it has not been altered. Chain of custody is what lets the institution *testify* that the produced record is the record that existed — and it depends entirely on the integrity controls the archive implemented at capture (§7.3).

### 10.2 What an Archive That Cannot Be Searched or Produced Is Worth

The blunt verdict of this section: **an archive that cannot be searched or produced is worth nothing as a compliance asset.** It may be worth something as a liability — because its existence can be evidence that the institution *knew it should have been able to produce records and did not invest in the capability*. Consider the spectrum:

- **A WORM store that holds messages but has no index** — immutable, unsearchable. It can prove nothing, because no one can find anything in it. It satisfies the "retention" leg of §8.1 while failing the "produce" leg, and it will fail a production request.
- **An archive that holds only post-gateway-rewritten messages with no envelope** — searchable, perhaps, but incomplete in the way §7.1 describes: it cannot answer recipient questions and it preserves only one version of each message.
- **An archive with no chain of custody** — producible, perhaps, but *unprovable*: the institution cannot establish that what it produces is what existed, because it did not implement integrity controls at capture (§7.3).

The design principle: **capture must anticipate production.** Every decision at capture time — retain the envelope, hash the content, journal the path, log the renders — is paid off or lost at production time. An institution that builds for *storage* and only later discovers it needed to build for *production* has, by then, already failed to capture what production requires.

### 10.3 Format, Metadata and Evidentiary Sufficiency

For a produced record to be evidentially useful, it should carry the context of §7.1: header, envelope addresses (sender and all recipients including `BCC` and expanded list members), the `Received:` chain and timestamps, the `Message-ID`, and the integrity hash — and ideally the authentication results (RFC 8601, May 2019) ✅ as they were evaluated at receipt. A production that strips metadata to produce "clean" documents destroys their evidentiary value; a production that preserves metadata lets the requesting party reconstruct the transaction. The institution's production format should be chosen deliberately, with legal input, and tested against a realistic request *before* a request arrives.

### 10.4 The Definition of Done

A production is done when the institution can (a) demonstrate the collection was complete within the agreed scope, (b) evidence the review and the withholding decisions, (c) deliver in an agreed, usable format, and (d) present a chain of custody for each item. Anything less is a partial answer, and a partial answer invites the question the guide opened with: *what is the archive missing?*

---

## 11. The Regulated-Institution View

### 11.1 What a Supervisor's Expectation Implies, Structurally

This section frames the architecture consequences **structurally** — it does not assert what any specific supervisor requires, because no such rule was verified against a dated primary publication here (§15). But a supervisor's expectation that communications are captured, retained, and reviewable has architectural implications that follow from the expectation *itself*, whatever its exact wording. Those implications are:

1. **Capture must be architecturally complete, not best-effort.** A capture path that misses a mail route (§7.2, §12) is a structural gap the institution cannot detect without reconciliation (§7.4). The architecture must make the journal path one that *all* mail must traverse, or must enumerate and monitor every path that does.
2. **The record must be immutable and integrity-controlled.** WORM storage (§7.3) plus capture-time hashing are the structural means to enable later chain of custody (§10.1). Without them, production cannot be defended.
3. **Retention must be schedule-governed and jurisdiction-aware.** The period is jurisdiction-specific (§8.1); the architecture must support a governed schedule, not a hard-coded constant, because the schedule must be *derived* and *evidenced*, and a hard-coded number cannot be justified.
4. **Deletion must be gated by holds.** The disposition engine must check holds at the moment of deletion (§8.2, §8.3); the architecture must implement this check, not assume it.
5. **Review must have capacity.** The surveillance programme (§9) is architecturally coupled to staffing; the architecture must surface review throughput and queue age, not merely alert volume (§9.4).
6. **Production capability must exist before it is needed.** The archive must be searchable and producible (§10); the production format must be tested (§10.3).
7. **The mail estate's *security* and its *records* concerns are one architecture.** The gateway (§6) is simultaneously a control point and a records source (§6.5); the authentication configuration (§4, §6.6) is both a security control and an auditable record with a lifecycle. An institution that treats security and records as two separate projects builds two half-systems that fail each other's audits.

### 11.2 Cross-References to the Repo's Governance Material

This section's architectural view is not self-contained; it connects to the repository's governance material and should be read with it:

- The **data-governance** model — classification, handling, retention concepts, and the protection of personal data across the estate (including the mail corpus) — is set out in [Data Governance](../technology/data/data_governance_framework.md). The mail archive is a data store subject to that model, not an exception to it.
- The **security-programme** context — the enterprise controls (identity, encryption, monitoring, incident response) within which the mail estate's security sits — is in [Cybersecurity](../technology/cybersecurity_guide.md).
- The **archive-technology** dimension — the object storage that may underpin a WORM archive and its integrity features — is in [StorageGRID](../technology/storagegrid_guide.md) (and note §8.1's caution about not upgrading a vendor's retention line into a regulatory rule).
- The **risk-and-compliance-systems** context — how a compliance system sits within the institution's risk architecture — is in [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md).
- The **fraud** context — the detection systems that receive the BEC signals of §5 — is in [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md).
- The **AI/ML-governance** context — how models used in surveillance and triage should be governed — is in [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md).
- The **MFT** channel — the *other* transfer channel, explicitly out of scope here — is in [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md).

### 11.3 The Structural Bottom Line

Read structurally, a supervisor's expectation of captured-and-reviewable communications does not ask the institution to buy a product. It asks the institution to be able to answer, defensibly, a small set of questions: *Do you have it? Is it the real thing? Can you find it? Can you prove you kept it and can prove what you did with it? Did you actually look at it, with capacity, and can you show the review?* Every architectural choice in this guide exists to make one of those answers true. None of them is a mail-delivery feature; all of them are records features. That is the thesis (§1.2) restated at the level of architecture.

---

## 12. Failure Modes — Symptom, Cause, Guardrail

### 12.1 Unenforced DMARC

| | |
| --- | --- |
| **Symptom** | Fraudulent mail bearing the bank's domain in the visible `From:` reaches customers and counterparties as though genuine; the bank's DMARC reports show large volumes of unauthenticated mail using its domain, but the mail is still delivered. |
| **Cause** | The domain publishes SPF and/or DKIM but its DMARC policy is `p=none` (monitor only) or absent, so no receiver is asked to act on a failure. The alignment requirement of RFC 7489 §3.1 ✅ is published but not *enforced*. |
| **Guardrail** | Advance the DMARC policy toward enforcement deliberately: inventory every legitimate sender, ensure each is aligned (SPF or DKIM) with the visible `From:` domain ✅ (RFC 7489 §3.1), then move to `p=quarantine` and `p=reject`. Monitor the reports continuously; treat the policy and the sender inventory as governed records (§4.5, §6.6). Note honestly that enforcement does not address display-name spoofing, look-alike domains, or compromised mailboxes (§3.6). |

### 12.2 THE ARCHIVE GAP — Mail Took a Path That Bypassed the Journal

| | |
| --- | --- |
| **Symptom** | The archive *runs* — it grows, it is searchable, it is WORM-protected — yet a later reconciliation shows the archive holds materially fewer messages than the gateway and MTAs processed. A production request surfaces a period for which expected messages are absent. |
| **Cause** | Journaling (§7.2) was enabled only on some mail paths. An application server sends SMTP directly to the Internet (bypassing the perimeter and its journal); a subsidiary's mail domain is not journaled; a mobile path or a cloud mail connector delivers around the central MTAs; the journal copy itself silently fails to deliver (§7.2's crucial point that the journal copy is *also* mail). The archive cannot detect its own gap (§7.4), so the gap persists undetected. |
| **Guardrail** | **Reconciliation as a standing control (§7.4):** compare gateway/MTA message counts against archive ingest counts, disaggregated per sender and per recipient domain; alert on the journal path and on the journal account's health; sample known messages through every path; reconcile the archive's ingest against the journal's emit. Architecturally, prefer a journal path that *all* mail must traverse, and explicitly enumerate and monitor every path that does not. |

### 12.3 Personal-Device and Personal-Account Mail That Never Enters the Record

| | |
| --- | --- |
| **Symptom** | A matter arises involving a staff member whose relevant communications occurred on a personal device or personal email account; the institution's archive contains none of them, and the review or production is materially incomplete. |
| **Cause** | The archive captures only mail transiting the institution's own systems (§6.1, §7.2). Communications conducted outside that estate — a personal account used for business, a personal mobile device on a third-party path — never enter the capture path. This is a *coverage* failure: the institution believed "our mail is archived" without asking *whose* mail and *through which systems*. |
| **Guardrail** | Policy that business communications use institutionally-captured channels; technical controls (managed devices, blocked personal webmail at the corporate boundary, mobile device management) that make the captured channel the path of least resistance; and — where a matter arises — a documented, defensible process for collecting from other sources without over-claiming completeness. The honest architectural acknowledgment: any capture path that depends on the *user's* choice of channel is only as complete as the users' compliance, so the control must be systemic (§11.1). |

### 12.4 SURVEILLANCE THEATRE — a Lexicon Nobody Reviews Substitutes for Supervision

| | |
| --- | --- |
| **Symptom** | The institution has a communications-monitoring platform, a tuned-looking lexicon, and a dashboard. Alert volumes are reported upward as evidence of vigilance. But the alert queue has an age measured in weeks or months, alerts are closed in bulk with no recorded rationale, and no review has ever changed a decision. |
| **Cause** | Review capacity (§9.4) was never matched to alert production. The programme optimized for *generating* alerts and *reported* alert volume as its metric. The human review step — the only step that produces supervision — was under-resourced and, implicitly, treated as a formality. |
| **Guardrail** | Resource review to the alert volume and measure *review throughput and queue age*, not alert counts (§9.4). Record a disposition and rationale for every reviewed alert. Tune the lexicon iteratively for precision, not just coverage (§9.2). Test the programme: can the institution point to a case where surveillance *found something that mattered*? If not, suspect theatre and investigate. Cross-reference [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) for model governance in triage. |

### 12.5 The Legal Hold That Failed to Suspend an Automated Deletion

| | |
| --- | --- |
| **Symptom** | A matter arises; the institution intends to hold the relevant records; the records are gone. The disposition log shows they were deleted on schedule *after* the matter was known. |
| **Cause** | The hold (§8.2) was not *implemented in the deletion automation*. It was recorded in a matter file, or communicated to an administrator to "remember," or scoped to the wrong custodian set — while the automated disposition engine, which did not know the matter existed, continued deleting on schedule. The hold was a *note*, not a *control*. |
| **Guardrail** | Enforce holds in the deletion engine: the disposition process must check, at the moment of deletion, whether an active hold covers the record, and must refuse to delete if so (§8.2, §8.3). Apply holds through a recorded workflow (notice, scope, acknowledgment, release), make hold application immediate, and test the control by attempting to dispose of a held record and confirming the engine blocks it. Own the honest failure: an institution should be able to produce the audit record showing which records were held and that none were deleted afterward. |

---

## 13. The Cymbal Bank Worked Example

> **This worked example is explicitly fictional and illustrative.** Cymbal Bank is a hypothetical institution; the systems, findings, and decisions below are constructed to demonstrate the guide's principles, not to describe any real institution's estate.

### 13.1 The Task — a Compliance Review of the Mail Estate

Cymbal Bank's compliance function is preparing for a periodic internal review of its communications record-keeping. The review brief is deliberately narrow: **establish whether the institution's email record is complete, retained under a governed schedule, and producible.** The review team is told to start where the evidence is, not where the drama is. It would be easy — and wrong — to begin by hunting for an attack; the review's purpose is the *capture* path, not the *attack* path. This is the guide's thesis in operational form: the interesting question is not whether mail was attacked but whether the record of the mail is sound.

### 13.2 What the Review Found — in the Capture Path, Not the Attack Path

Cymbal's review runs the completeness reconciliation of §7.4. It compares the message counts on its outbound and inbound gateways and its central MTAs against the counts of messages ingested into the archive, for a sample period, and disaggregates by sending system and recipient domain.

The global counts look close. The disaggregated counts do not. A specific pattern emerges: messages originated by a set of application systems — the ones that send automated notifications directly, without passing through the central relay — are present on the gateway logs but **absent from the archive**. The central journal (§7.2) captures what traverses the central MTAs; the application systems' direct SMTP paths do not traverse them, so their messages are never journaled and never captured. The archive is healthy, WORM-protected (§7.3), and searchable — and it is missing an entire class of mail (the §12.2 archive gap).

The review also notes a second, quieter finding: the journal destination mailbox had, twice in the period, hit its capacity limit, and the journal copies for those windows had been *not delivered* — visible only because the journal path is itself a mail path (§7.2) and its failures are logged *if anyone looks*. No one had been looking.

**Crucially, the review found nothing in the attack path.** No unenforced DMARC (§12.1) — Cymbal's policy was already enforced and its sender inventory clean. No failed hold (§12.5). No surveillance theatre (§12.4) — the programme's review throughput was measured and adequate. The find was entirely in the *records* path, and it is the kind of find that no amount of inbound filtering would ever surface, because the messages in question were legitimate mail that was simply never captured.

### 13.3 The One Decision the Review Changed

The review's finding forced one governing decision, and the review framed it as a single decision on purpose — because a review that produces twelve recommendations and no decision has produced no change:

> **Cymbal Bank decided that the journal path is a *mandatory transit* property of the mail estate: every system authorized to originate or relay mail must be enrolled in the capture path, and no new mail-originating system may be deployed without an evidenced journal enrollment. The institution further decided that the gateway/MTA-versus-archive reconciliation of §7.4 becomes a *standing, monitored control* with defined tolerance, rather than a review exercise, and that the journal destination's health and capacity are monitored as a critical service with alerting.**

That decision has three consequences, each mapping to a section of this guide:

1. **Architecture** — the direct-SMTP application systems were re-routed through (or separately journaled into) the capture path, closing the §12.2 gap. The principle applied was §7.2's: a journal path that not everything traverses is a gap generator, so either everything traverses it, or every path is separately monitored.
2. **Control** — the reconciliation became standing (§7.4), so the *next* gap will be detected by a control rather than discovered by luck. The tolerance was defined so that an alert fires on a real gap, not on legitimate drops.
3. **Governance** — the enrollment requirement turned a *configuration* into a *governed gate*: a new mail-originating system cannot be deployed without its capture path evidenced. This is the §11.1 principle — capture must be architecturally complete, not best-effort — made enforceable.

The decision did **not** change Cymbal's authentication posture (already sound), its hold process (already gated), or its surveillance (already measured). The review changed the one thing that was actually broken, and it resisted the temptation to change things that were not.

### 13.4 The Thesis, Restated

Cymbal's review is the thesis of this guide demonstrated in miniature. The institution went looking for an *attacker* and found instead a *gap in its own record*. The gap had no security signature — no spoofed domain, no malicious attachment, no compromised mailbox — because it was not an attack at all. It was the ordinary, invisible failure of a records system that had been built as a messaging system: a body of legitimate mail that flowed perfectly and was never captured. No amount of the §3 authentication stack, no gateway inspection (§6), and no lexicon (§9) would have found it, because none of them is looking at *what is missing*; they are all looking at *what is present*. Only the reconciliation of two independent counts (§7.4) could see it — and it could see it only once someone decided the record mattered enough to count.

That is why the guide's thesis is an inversion and not a slogan. The bank that treats its email as a messaging system optimizes everything in §4, §5 and §6 and never counts what it failed to keep. The bank that treats its email as a records system accepts that the hardest question is the silent one — *what is the archive missing?* — and builds the controls (§7.4, §8.2, §9.4) that can answer it. **A bank's email is not a messaging system; it is a records system that happens to deliver messages**, and the difference is measured in the messages nobody noticed were never kept.

---
---

## 14. The Anti-Patterns

Each anti-pattern below is a temptation that *looks* like compliance and is not. They are stated as instructions to avoid, with the misconception each one rests on.

### 14.1 "We Archive Our Email, So We Archive Our Communications"

The misconception: that the mail archive covers the institution's communications. It covers *mail that transited a journaled path*. It does not cover file transfers (a different channel — see [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md)), instant messaging and collaboration tools, voice, or mail that used a non-journaled path. Scope claims must be precise: name the channels covered and the paths within each.

### 14.2 "We Have WORM, So Our Archive Is Compliant"

The misconception that immutability equals compliance. WORM prevents rewrite (§7.3); it does not prove completeness or correct capture. An immutable incomplete archive is still incomplete. Never substitute the storage guarantee for the capture guarantee.

### 14.3 "The Gateway Filtered It, So It's Handled"

The misconception that blocking a message is the end of the obligation. A message that attempted fraud is evidence of an attempted fraud (§5.6) and may be needed for production (§10). If the gateway discards attack messages without capturing them, the institution has destroyed evidence of the threat it is expected to understand.

### 14.4 "SPF and DKIM Are Configured, So Our Domain Is Protected"

The misconception that configuring authentication equals enforcing it. Without DMARC alignment and enforcement (§3.5, §12.1), SPF and DKIM authenticate identities that need not be the visible `From:`. Configuration without enforcement protects nothing against spoofing.

### 14.5 "We Told Everyone to Verify Payment Changes"

The misconception that a *policy* is a *control*. The out-of-band verification of §5.5 is a control only when the payment system *refuses* to act without a recorded verification event. A policy that depends on individual memory is §12.5 in a different costume: fragile.

### 14.6 "The Journal Runs, So Capture Is Complete"

The misconception addressed in §7.4 and demonstrated in §13: a running journal proves nothing about completeness. A journal that misses a path, or a journal mailbox that silently saturates, produces gaps invisible from the archive's own perspective. Only reconciliation can see them.

### 14.7 "Monitoring Generates Thousands of Alerts — We're Watching Everything"

The misconception of §9.4 and §12.4: alert volume as a proxy for supervision. Alerts nobody reviews are surveillance theatre. The metric that matters is reviewed-and-dispositioned throughput against queue age, not the alert count.

### 14.8 "The Hold Is Applied, We're Fine"

The misconception that a hold recorded somewhere equals a hold enforced in the deletion engine. If the automated disposition process does not *check* for holds at the moment of deletion (§8.2, §8.3), the hold is a note. A hold is a control only if it can *stop a deletion*, and only if the institution can prove it did.

### 14.9 "Discovery Is a Legal Problem, Not an Architecture Problem"

The misconception that production (§10) concerns the legal team only. Production capability is designed *at capture time* — whether the envelope was kept, whether content was hashed, whether the path was journaled. By the time discovery arrives, the architecture has already fixed the ceiling on what can be produced. Discovery is, first and last, an architecture problem.

### 14.10 "The Vendor Says Seven Years, So That's Our Retention Rule"

The misconception that a vendor's suggested configuration is a regulatory requirement (§8.1). A vendor's archiving table (for example, the line in [StorageGRID](../technology/storagegrid_guide.md)) states what the *product* supports; it does not establish what *the institution's regulators require*. Retention must be derived from the applicable primary texts, per jurisdiction, and recorded as a governed schedule.

### 14.11 "We Verified the Supplier's Bank Details — We Called the Number in Their Email"

The misconception that a verification is a verification. Calling the number supplied *in the suspicious message* verifies nothing; it connects the caller to the person who sent the message. Out-of-band verification means a *previously-known, independently-sourced* number (§5.5), never one supplied by the request itself.

### 14.12 "Transactional and Marketing Mail Are Both Just Email"

The misconception that the two populations are governed identically (§4.4). The distinction shapes deliverability monitoring, sending infrastructure, classification in the archive, and (in jurisdictions the institution should source itself) possibly consent obligations. Treating them as one population blurs both the operational and the records view — and this guide asserts no consent rule for either.

---

## 15. The Claims Audit

Every claim in this guide is marked ✅ (verified this pass against a named, dated source) or ⚠ (flagged, inferred, or unverified). Protocol facts were verified at the RFC Editor's own text; the RFC's **number and date** are cited. The two hazards that govern this guide's discipline are recorded as explicit rows at §15.3.

### 15.1 Verified Claims (✅) — Protocol Facts from the RFC Editor

| # | Claim | Source (number, date, kind) |
| --- | --- | --- |
| 1 | SMTP is defined by RFC 5321; Standards Track; obsoletes 2821; updates 1123 | RFC 5321, October 2008 ✅ (RFC) |
| 2 | SMTP is store-and-forward: "a mail message may pass through a number of intermediate relay or gateway hosts on its path from sender to ultimate recipient"; next hop resolved from MX records | RFC 5321 §2.1, October 2008 ✅ (RFC) |
| 3 | "SMTP mail relaying": a process may transfer mail to another network via a relay or gateway; a host may act as both server and client in a relay situation | RFC 5321 §§2.1, 2.3.8, October 2008 ✅ (RFC) |
| 4 | A relay SMTP "has no need to inspect or act upon the header section or body of the message data and MUST NOT do so except to add its own 'Received:' header field"; the prohibition also limits header modification | RFC 5321 §3.6.1, October 2008 ✅ (RFC) |
| 5 | A gateway at a transport-environment boundary MAY rewrite header fields when gatewaying across mail environment boundaries | RFC 5321 §§3.7, 3.7.1, October 2008 ✅ (RFC) |
| 6 | A mail object contains an envelope and contents; the SMTP envelope is sent as SMTP protocol units (`MAIL FROM`, `RCPT TO`); the contents comprise the object delivered to the recipient | RFC 5321 §2.3.1 (and grammar §4.1.1.2 / §4.1.1.3), October 2008 ✅ (RFC) |
| 7 | When a message leaves the Internet environment, it may be necessary to fold the SMTP envelope information into the message header section (gatewaying) | RFC 5321 §3.7.1, October 2008 ✅ (RFC) |
| 8 | List expansion changes the envelope at each expansion (`RCPT TO` addressed to each member; the rest of the envelope and body left unchanged) | RFC 5321 §3.10, October 2008 ✅ (RFC) |
| 9 | Non-delivery notifications are generated to the envelope reverse-path | RFC 5321 §4.5.5, October 2008 ✅ (RFC) |
| 10 | The Internet Message Format is RFC 5322; Standards Track; obsoletes 2822; updates 4021; a message consists of header fields and an optional body | RFC 5322 §3.6 (introduction), October 2008 ✅ (RFC) |
| 11 | The header is the message's own claims; the format document does not define the transport envelope | RFC 5322 §3.6.1, October 2008 ✅ (RFC) |
| 12 | SPF is RFC 7208; Standards Track; obsoletes 4408; existing protocols place no restriction on the `MAIL FROM` or `HELO/EHLO` domain a sender may use | RFC 7208 Abstract, April 2014 ✅ (RFC) |
| 13 | SPF authenticates the `MAIL FROM` identity and the `HELO` identity — the envelope — not the visible `From:` | RFC 7208 §§1.1.3, 1.1.4, 2.3, 2.4, April 2014 ✅ (RFC) |
| 14 | DKIM is RFC 6376; Standards Track; obsoletes 4871 and 5672; it separates the identity of the Signer from the purported author | RFC 6376 Abstract, September 2011 ✅ (RFC) |
| 15 | A DKIM signature associates the `d=` SDID with a hash of the message; `h=` lists signed header fields; `bh=` is the hash of the canonicalized body | RFC 6376 §§2.5, 3.5, September 2011 ✅ (RFC) |
| 16 | DKIM signatures survive transit because relays "typically make no substantive change to the message content"; a signing domain may remove its key record after a transition period | RFC 6376 Abstract and §3.6.4, September 2011 ✅ (RFC) |
| 17 | DMARC is RFC 7489; **Informational** (not Standards Track); published for informational purposes | RFC 7489 header and Status of This Memo, March 2015 ✅ (RFC, Informational) |
| 18 | DMARC's Identifier Alignment requires the domain in the RFC 5322 `From` address to match (be aligned with) an authenticated identifier from SPF or DKIM; alignment may be relaxed or strict; a DMARC "pass" occurs if a DKIM signature is aligned or an SPF result is aligned | RFC 7489 §3.1, March 2015 ✅ (RFC) |
| 19 | DMARC expresses domain-level policy and reporting; the policy ranges "from no action, through altered delivery, up to message rejection"; DMARC does not itself produce elevated delivery privilege | RFC 7489 Abstract, March 2015 ✅ (RFC) |
| 20 | STARTTLS is RFC 3207; Standards Track; obsoletes 2487; it lets an SMTP server and client use TLS for private, authenticated communication, protecting against eavesdroppers and attackers; the base protocol communicates in the clear | RFC 3207 Abstract and §1, February 2002 ✅ (RFC) |
| 21 | MTA-STS is RFC 8461; Standards Track; it lets a mail service provider declare its ability to receive TLS-secure SMTP and specify whether senders should refuse to deliver to MX hosts not offering TLS with a trusted server certificate | RFC 8461 Abstract, September 2018 ✅ (RFC) |
| 22 | RFC 8314 gives TLS recommendations for MUA-to-submission/access-server mail; Standards Track; updates 1939, 2595, 3501, 5068, 6186, 6409; title "Cleartext Considered Obsolete" | RFC 8314 header and Abstract, January 2018 ✅ (RFC) |
| 23 | RFC 8601 defines the `Authentication-Results` header field to indicate the results of message authentication efforts, usable by filters and MUAs; Standards Track; obsoletes 7601 | RFC 8601 Abstract, May 2019 ✅ (RFC) |
| 24 | ARC (Authenticated Received Chain) is RFC 8617, **Experimental**, July 2019 | RFC 8617 header, July 2019 ✅ (RFC, Experimental) |

### 15.2 Verified Claims (✅) — Structural and Domain Statements

| # | Claim | Source (date, kind) |
| --- | --- | --- |
| 1 | The mail estate of a regulated institution must be able to capture, retain, and produce communications — the structural triangle (§8.1) | ✅ structural inference from the records thesis; stated as shape, not as a rule (see §15.3) |
| 2 | Out-of-band verification of a payment-detail change — a callback on a previously-known, independently-sourced number, before the change is applied — is the control that matters against supplier/bank-details-change fraud (§5.5) | ⚠→✅ treated as a structural control recommendation grounded in the §3.6 authentication gaps; no single dated publication is asserted as its source |
| 3 | Journaling is a mail-server feature, not a protocol feature; no RFC defines it; the journal copy is delivered as mail and is subject to mail-path failures (§7.2) | ✅ structural statement from the absence of any journaling definition in the RFCs read this pass |

### 15.3 HAZARD 1 Compliance — the Two Findings Recorded

| # | Claim status | What was printed | Why |
| --- | --- | --- | --- |
| 1 | **NO retention period printed as a rule** | The guide states only the *shape* of the retention obligation (§8.1) and that the period is **jurisdiction- and rule-specific and unverified here**. No "N years" figure is asserted for banks anywhere in this guide. | No dated primary publication establishing a specific retention period was retrieved this pass. Printing a number without a dated source would violate the guide's sourcing discipline. The vendor line in [StorageGRID](../technology/storagegrid_guide.md) ("7+ years") is explicitly **not** upgraded into a regulatory rule (§8.1, §14.10). |
| 2 | **NO phishing/BEC loss figure printed** | No loss figure, no per-incident average, no annual total, and no percentage appears anywhere in this guide. | No dated report from a named body measuring phishing/BEC losses was retrieved this pass. Per HAZARD 1(c), if the figure could not be attributed to a body, a date, and what it measured, **no figure is printed at all**, and the absence is recorded here as a finding. |
| 3 | **NO supervisor rule name asserted as a requirement** | The guide describes what "a supervisor's expectation" implies *structurally* (§11.1) without naming any rule as requiring it. | No dated primary regulator publication was retrieved to support naming a specific rule. Naming a rule without a source would be an unsourced assertion. |
| 4 | DMARC is Informational, not Standards Track | Stated explicitly in §3.5 and §15.1 row 17. | RFC 7489's own header and Status of This Memo establish it ✅. This is a status fact, not a claim about behaviour. |

### 15.4 Flagged / Unverified Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | WORM "prevents rewrite, does not prove completeness or correct capture" (§7.3) | Structural property of WORM storage; no single protocol or regulatory source is asserted. Flagged as ⚠/structural. |
| 2 | Domain reputation as "accumulated behaviour" shaping receivers' trust (§4.2) | Structural inference from how authentication and complaints affect receivers; not tied to a dated standard. |
| 3 | The transactional-vs-marketing governance distinction (§4.4) | Presented as an operational/governance split only; **no consent rule, no opt-in obligation, and no marketing-law requirement is asserted**. |
| 4 | Retention period being "jurisdiction- and rule-specific" (§8.1) | Stated as the reason no period is printed; the *variation* itself is not tied to a dated source in this pass; flagged rather than asserted. |
| 5 | The exact population of "reviewers needed = alerts ÷ reviewer-capacity" (§9.4) | A structural staffing argument, not a sourced metric. |
| 6 | Reconciliation tolerances and the specific gap-catching behaviour of each §7.4 control | Presented as control design principles; the tolerances are institution-specific and not sourced. |
| 7 | The Cymbal Bank worked example (§13) | **Explicitly fictional and illustrative** — constructed to demonstrate principles, not a description of any real institution. |

---

## 16. What Could Not Be Verified, the Glossary, and the Closing

### 16.1 What Could Not Be Verified

**Honest entries — claims the guide would have liked to make and did not, because the source was not retrieved:**

1. **No retention period for bank email could be verified.** The guide states the *shape* of the retention obligation (§8.1) but prints no period, because no dated primary publication establishing one was retrieved this pass. This is the single most consequential absence in the guide: retention is the leg of the §8.1 triangle that most directly determines deletion behaviour, and this guide deliberately leaves the number to the institution's own sourced determination rather than inventing one. The relevant primary texts are jurisdiction-specific and must be retrieved per institution.

2. **No phishing or business-email-compromise loss figure could be verified.** No dated report from a named body — measuring what it measured — was retrieved. Per HAZARD 1(c), **no figure of any kind is printed**, and the absence is recorded as a finding (§15.3). A reader wanting such a figure should retrieve the source directly and attribute the body, the date, and the measured population.

3. **No named supervisor rule was asserted as a requirement.** The regulated-institution view (§11) is framed entirely structurally. Naming a rule as requiring capture, retention or review would require retrieving that rule's dated primary text, which this pass did not do.

4. **Journaling's completeness properties are not standardised.** Journaling is a mail-server feature with no RFC; its completeness guarantees are vendor-specific (§7.2). This guide could not verify a generalised guarantee because none exists at the standards level — only per-platform documentation, which was not retrieved per platform.

5. **Vendor archiving retention lines were not upgraded.** Where the repository cites a vendor figure such as "7+ years" ([StorageGRID](../technology/storagegrid_guide.md)), this guide treats it as a vendor product statement, not a regulatory rule (§8.1, §14.10), and did not verify the underlying product documentation this pass.

6. **Tool limitation, recorded as such.** Where a search tool returned empty during this pass, that is recorded as a **tool limitation**, not as evidence of absence of material. The absence of a retrieved retention period or loss figure reflects what could be retrieved with the tools available, not a claim that no such publications exist — they almost certainly do, and the institution should retrieve them at the primary source.

### 16.2 Glossary

| Term | Definition |
| --- | --- |
| **SMTP** | Simple Mail Transfer Protocol — the store-and-forward transport protocol for internet mail; RFC 5321 (October 2008) ✅. |
| **MTA** | Mail Transfer Agent — the server process that speaks SMTP, receiving and relaying mail. |
| **Relay** | An intermediate MTA that forwards a message toward a destination for which it is not the final recipient ✅ (RFC 5321 §2.1). |
| **Gateway** | The MTA at the boundary between transport environments, where policy is enforced and header rewriting may be permitted ✅ (RFC 5321 §§2.3.10, 3.7). |
| **Envelope** | The SMTP transaction's routing data — `MAIL FROM` (reverse-path) and `RCPT TO` (forward-path) ✅ (RFC 5321 §2.3.1). |
| **Header** | The message content's own fields — `From:`, `To:`, `Received:`, etc. ✅ (RFC 5322). |
| **SPF** | Sender Policy Framework — DNS-published authorization of hosts permitted to use a domain in the *envelope* (`MAIL FROM`/`HELO`); RFC 7208 (April 2014) ✅. |
| **DKIM** | DomainKeys Identified Mail — a cryptographic signature over selected header fields and the body, attached to the `d=` (SDID) domain; RFC 6376 (September 2011) ✅. |
| **DMARC** | Domain-based Message Authentication, Reporting, and Conformance — policy, alignment and reporting; **Informational**; RFC 7489 (March 2015) ✅. |
| **Identifier Alignment** | DMARC's requirement that the visible `From:` domain match an authenticated SPF or DKIM identifier ✅ (RFC 7489 §3.1). |
| **STARTTLS** | The SMTP extension enabling TLS on an SMTP connection; RFC 3207 (February 2002) ✅. |
| **MTA-STS** | SMTP MTA Strict Transport Security — lets a receiving domain require validated TLS from senders; RFC 8461 (September 2018) ✅. |
| **Authentication-Results** | The header field recording the results of authentication checks; RFC 8601 (May 2019) ✅. |
| **BEC** | Business email compromise — attacks inducing an authorized employee to act via email, using impersonation, a look-alike domain, or a compromised mailbox. |
| **Out-of-band verification** | Verifying a payment-detail change via a separate channel using a previously-known, independently-sourced contact — the control that matters (§5.5). |
| **Journaling** | The mail-server feature emitting a copy of each message as it transits, for capture; not defined by any RFC (§7.2). |
| **Archive** | The store of captured copies, searchable and producible, retained for the obligation's duration (§7). |
| **WORM** | Write Once Read Many storage — immutable once written; prevents rewrite, does not prove completeness or correct capture (§7.3). |
| **Legal hold** | The suspension of an otherwise-running automated deletion process for a defined set of custodians and subjects (§8.2). |
| **Disposition** | The final, auditable, hold-gated deletion of a record whose retention has run (§8.3). |
| **Lexicon** | The set of terms and patterns a surveillance programme searches for in the corpus (§9.2). |
| **False positive** | A surveillance hit that is not a genuine issue; the primary consumer of review capacity (§9.4). |
| **Surveillance theatre** | The substitution of monitoring artefacts (lexicon, dashboard) for actual reviewed supervision (§9.5, §12.4). |
| **Chain of custody** | The documented, unbroken record of an item's collection, handling, review, and delivery (§10.1). |
| **Discovery / production** | The collection, review, production, and custody-evidence of records under a request (§10). |
| **MFT** | Managed file transfer — a *different* channel (file transfer, not mail), out of scope here; see [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md). |

### 16.3 Cross-References

**Sibling guides (same folder, `banking/`):**

- [Financial Fraud Detection at Scale](financial_fraud_detection_at_scale_guide.md) — the fraud-detection systems that receive the BEC signals of §5; cross-referenced rather than re-derived.
- [Financial Risk & Compliance Systems](financial_risk_compliance_systems_guide.md) — how a compliance system sits within the institution's risk architecture (§11.2).
- [AI/GenAI Banking Compliance](ai_genai_banking_compliance_guide.md) — the governance of AI/ML models used in surveillance and triage (§9.3, §11.2).

**Technology guides (`../technology/`):**

- [Cybersecurity](../technology/cybersecurity_guide.md) — the enterprise security-programme context for the mail estate (§6.2, §11.2).
- [Data Governance](../technology/data/data_governance_framework.md) — the data-classification, handling and protection model that the mail corpus and its surveillance are subject to (§6.2, §9.6, §11.2).
- [Axway CFT / MFT](../technology/axway_transfer_cft_guide.md) — the **different channel** (managed file transfer), declared out of scope in §1.3 and referenced throughout (§1.3, §14.1, §16.2).
- [StorageGRID](../technology/storagegrid_guide.md) — object storage that may underpin a WORM archive; note §8.1's caution about not upgrading a vendor retention line into a regulatory rule (§11.2, §14.10).

### 16.4 Closing Summary

This guide began with a decoder and a thesis. The decoder — SMTP, the MTA, the relay, the gateway, the envelope, the header, the journal, the archive, the WORM store, the legal hold, the lexicon, the disposition — turned out to describe not a messaging system but a records system. The thesis — that the interesting question is not whether mail arrives but whether the record of the mail is sound — reordered every control: the authentication stack of §3 proves less than it appears to, the inbound control that matters is a phone call on a number nobody supplied in the suspicious message (§5.5), the gateway is as much a records source as a filter (§6.5), and the hardest question in the whole guide is the silent one — *what is the archive missing?* — which only the reconciliation of two independent counts can answer (§7.4). The failure modes (§12) and the Cymbal Bank example (§13) both landed in that records path, not the attack path, because that is where the failures that no filter can see actually live. This guide printed no retention period and no loss figure, and recorded those absences as findings, because a guide about evidence must itself be governed by evidence (§15, §16.1). And it declared its boundary by name — the mail channel, not the file-transfer channel — because the first discipline of a records guide is to say what it keeps, and what it does not.

a bank's email is not a messaging system; it is a records system that happens to deliver messages.
