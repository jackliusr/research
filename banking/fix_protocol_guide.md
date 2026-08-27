# FIX Protocol: The Language of Electronic Trading — A Comprehensive Guide

**The Trade Conversation — the FIX Protocol Definition and History (1992 Salomon Brothers × Fidelity to the FIX Trading Community), the Wire Format (tag=value, SOH, Header/Body/Trailer, the Checksum), the Protocol Layers (Session vs Application, Sequence Numbers, Gap-Fill Recovery), the Versions (FIX 4.0 → 5.0 SP2, FIXT 1.1, FAST, FIXatdl, FIXML, SBE, FIXP), the Application Messages (NewOrderSingle, ExecutionReport, Cancel/Replace, Quotes, Market Data), the Trading Ecosystem (OMS/EMS, Algos, SOR, Matching Engines, ECN/ATS), the Performance Variants (FAST, SBE, CME MDP 3.0, REST/JSON), the Head-to-Head Comparison, the Cymbal Bank Session-Lifecycle Worked Example, the Claims Audit, and the Glossary — from the 1992 Phone-Call Problem to the Binary Multicast Firehose**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Banking Domain / Capital Markets — the FIX (Financial Information eXchange) protocol: the de-facto electronic-trading messaging standard for pre-trade and trade workflows between institutional buy-side and sell-side firms, spanning equities, FX, rates and derivatives. The guide covers the definition and history (the 1992 Salomon Brothers × Fidelity Investments collaboration, FIX Protocol Ltd, the FIX Trading Community), the wire format (tag=value pairs delimited by SOH 0x01, the 8/9/35 header, the 10=Checksum trailer), the session vs application layer split (Logon, Heartbeat, TestRequest, ResendRequest, SequenceReset, Logout; MsgSeqNum; gap-fill recovery), the version family (FIX 4.0, 4.1, 4.2, 4.3, 4.4, 5.0, 5.0 SP1/SP2, FIXT 1.1, FIXP), the encodings (tag/value, FIXML, FAST, SBE, FIXatdl), the key application messages (NewOrderSingle 35=D, ExecutionReport 35=8, OrderCancelReplaceRequest 35=G, OrderCancelRequest 35=F, Quote 35=S, MarketDataRequest 35=V, MarketDataSnapshotFullRefresh 35=W, MarketDataIncrementalRefresh 35=X — with verified tag-level breakdowns), the ecosystem (OMS/EMS, algorithmic trading, smart order routing, matching engines, ECN/ATS, broker connectivity hubs), the performance/modern variants (FIX/FAST compression, SBE at CME, CME MDP 3.0 multicast, REST/JSON alternatives), the head-to-head comparison (FIX vs native venue APIs vs binary protocols vs ISO 20022 — cross-referencing the sibling ISO 20022 guide rather than re-deriving it), the banking context (a Cymbal Bank electronic-trading estate), the worked example (a full Cymbal Bank FIX session lifecycle: Logon, Heartbeats, NewOrderSingle → partial fill → fill, a sequence-number gap with ResendRequest + SequenceReset gap-fill recovery, and Logout), the claims audit, the "What Could Not Be Verified" section, and the glossary.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the FIX Trading Community (fixtrading.org — "The FIX Protocol Brief History", "Who We Are", the FAST and SBE and FIXatdl standards pages), the OnixS FIX Dictionary (onixs.biz/fix-dictionary — FIX 4.4 and FIX 4.2 message and field pages: NewOrderSingle <D>, ExecutionReport <8>, Logon <A>, Resend Request <2>, Sequence Reset <4>, Order Cancel Request <F>, Order Cancel/Replace Request <G>, Market Data Request <V>, Snapshot/Full Refresh <W>, Incremental Refresh <X>, OrdStatus <39>, ExecType <150>, plus the OnixS protocol-history article), the OnixS FIX Protocol overview page, the CME Group "Develop to Globex" page (iLink, MDP 3.0), the Databento CME Globex MDP 3.0 dataset page, Wikipedia (Financial Information eXchange, FAST protocol, FIXatdl — secondary, used for history cross-check only), and MarketsWiki (FIX Trading Community — secondary). Everything verified against a primary/secondary source in this pass is marked ✅; anything that could not be verified is flagged ⚠ honestly. No dates, tag numbers or codes were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — the capital-markets cluster):** [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) (the payments/clearing message standard — cross-ref §9, do not re-derive) · [Payment Rails](payment_rails_guide.md) (the rails and message-standards map — cross-ref §9) · [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) (OMS/EMS, order-routing and exchange infrastructure — cross-ref §7) · [Capital Markets Architecture](capital_markets_architecture_guide.md) (the trading architecture and message standards — cross-ref §7) · [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) (the platform view) · [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) (the Cymbal Bank persona conventions) · [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) (the Cymbal Bank correspondent worked-example conventions)
> **Companion guides (technology/, prefix `../technology/`):** [Zero Downtime System Design](../technology/zero_downtime_system_design_guide.md) (the 24/7 session estates — cross-ref the session-recovery angle)

---

**How to use this guide:** Section 1 is the overview — the one-paragraph answer and the key-facts table. Section 2 is the wire format — tag=value pairs, SOH, the header/body/trailer anatomy and the checksum, with a worked message. Section 3 is the history — the 1992 origins, the 1995 public release, FIX Protocol Ltd, the FIX Trading Community, and the version timeline. Section 4 is the protocol layers — the session layer vs the application layer, the admin message set, sequence numbers and gap-fill recovery. Section 5 is the versions and encodings — FIX 4.x, FIX 5.0/5.0 SP2, FIXT 1.1, FIXP, FIXML, FAST, SBE, FIXatdl. Section 6 is the application messages — the order lifecycle (NewOrderSingle, ExecutionReport, Cancel/Replace, Cancel Reject), quotes, and market data, with verified tag-level breakdowns of NewOrderSingle and ExecutionReport, and the verified OrdStatus/ExecType value tables. Section 7 is the ecosystem — OMS/EMS, algorithmic trading, smart order routing, matching engines, ECN/ATS and connectivity hubs. Section 8 is performance and modern variants — FIX/FAST compression, SBE, CME MDP 3.0, iLink, and the REST/JSON alternative. Section 9 is the head-to-head comparison — FIX vs native venue APIs vs binary protocols vs ISO 20022 (cross-referencing the sibling guide). Section 10 is the Cymbal Bank worked example — a full session lifecycle trace with Logon, Heartbeats, NewOrderSingle → partial fill → fill, a sequence gap with ResendRequest and SequenceReset gap-fill recovery, and Logout, plus how a bank's electronic-trading estate integrates FIX. Section 11 is the claims audit (✅/⚠/❌). Section 12 is "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references. Cross-references follow the repository convention: sibling guides in `banking/` are plain filenames; guides in `technology/` are prefixed `../technology/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (URL given in §11); ⚠ = flagged/unverified or approximate; ⚠-knowledge = well-documented industry knowledge that could not be re-verified live in this pass and is flagged honestly. The final word of the guide is "the trade conversation."

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
2. [The Wire Format — Tag=Value, SOH, Header/Body/Trailer](#2-the-wire-format--tagvalue-soh-headerbodytrailer)
   - 2.1 [The Tag=Value Field](#21-the-tagvalue-field)
   - 2.2 [The Message Anatomy — Header, Body, Trailer](#22-the-message-anatomy--header-body-trailer)
   - 2.3 [The BodyLength and the Checksum](#23-the-bodylength-and-the-checksum)
   - 2.4 [A Worked Message — ExecutionReport in the Clear](#24-a-worked-message--executionreport-in-the-clear)
3. [The History — 1992 to the FIX Trading Community](#3-the-history--1992-to-the-fix-trading-community)
   - 3.1 [The 1992 Origins — Salomon Brothers × Fidelity Investments](#31-the-1992-origins--salomon-brothers--fidelity-investments)
   - 3.2 [The Public Release and the Industry Take-Up](#32-the-public-release-and-the-industry-take-up)
   - 3.3 [FIX Protocol Ltd and the FIX Trading Community](#33-fix-protocol-ltd-and-the-fix-trading-community)
   - 3.4 [The Version Timeline — Verified and Flagged](#34-the-version-timeline--verified-and-flagged)
4. [The Protocol Layers — Session vs Application](#4-the-protocol-layers--session-vs-application)
   - 4.1 [The Session Layer](#41-the-session-layer)
   - 4.2 [The Session (Admin) Messages](#42-the-session-admin-messages)
   - 4.3 [Sequence Numbers and the Session State](#43-sequence-numbers-and-the-session-state)
   - 4.4 [Gap Detection and Recovery — ResendRequest and SequenceReset](#44-gap-detection-and-recovery--resendrequest-and-sequencesreset)
   - 4.5 [Session-Level vs Application-Level Behaviour](#45-session-level-vs-application-level-behaviour)
5. [The Versions and Encodings](#5-the-versions-and-encodings)
   - 5.1 [The FIX 4.x Family](#51-the-fix-4x-family)
   - 5.2 [FIX 5.0, 5.0 SP1/SP2 and FIXT 1.1](#52-fix-50-50-sp1sp2-and-fixt-11)
   - 5.3 [FIXP — the FIX Performance Session Layer](#53-fixp--the-fix-performance-session-layer)
   - 5.4 [FIXML — the XML Encoding](#54-fixml--the-xml-encoding)
   - 5.5 [FAST — FIX Adapted for Streaming](#55-fast--fix-adapted-for-streaming)
   - 5.6 [SBE — Simple Binary Encoding](#56-sbe--simple-binary-encoding)
   - 5.7 [FIXatdl — the Algorithmic Trading Definition Language](#57-fixatdl--the-algorithmic-trading-definition-language)
6. [The Application Messages](#6-the-application-messages)
   - 6.1 [The Order Lifecycle in FIX](#61-the-order-lifecycle-in-fix)
   - 6.2 [NewOrderSingle (35=D) — Worked Tag Breakdown](#62-newordersingle-35d--worked-tag-breakdown)
   - 6.3 [ExecutionReport (35=8) — Worked Tag Breakdown](#63-executionreport-358--worked-tag-breakdown)
   - 6.4 [OrdStatus and ExecType — the Verified Value Tables](#64-ordstatus-and-exectype--the-verified-value-tables)
   - 6.5 [OrderCancelReplaceRequest (35=G) and OrderCancelRequest (35=F)](#65-ordercancelreplacerequest-35g-and-ordercancelrequest-35f)
   - 6.6 [Quotes — QuoteRequest (35=R) and Quote (35=S)](#66-quotes--quoterequest-35r-and-quote-35s)
   - 6.7 [Market Data — Request (35=V), Snapshot (35=W), Incremental (35=X)](#67-market-data--request-35v-snapshot-35w-incremental-35x)
7. [The Trading Ecosystem — Where FIX Lives](#7-the-trading-ecosystem--where-fix-lives)
   - 7.1 [OMS and EMS](#71-oms-and-ems)
   - 7.2 [Algorithmic Trading and Smart Order Routing](#72-algorithmic-trading-and-smart-order-routing)
   - 7.3 [Matching Engines, ECN/ATS and Execution Venues](#73-matching-engines-ecnats-and-execution-venues)
   - 7.4 [Broker/Dealer Connectivity Hubs](#74-brokerdealer-connectivity-hubs)
8. [Performance and the Modern Variants](#8-performance-and-the-modern-variants)
   - 8.1 [The Latency Problem](#81-the-latency-problem)
   - 8.2 [FIX/FAST Compression](#82-fixfast-compression)
   - 8.3 [Simple Binary Encoding in Production — CME](#83-simple-binary-encoding-in-production--cme)
   - 8.4 [CME MDP 3.0 — the Binary Multicast Firehose](#84-cme-mdp-30--the-binary-multicast-firehose)
   - 8.5 [REST/JSON — the Low-Friction Alternative](#85-restjson--the-low-friction-alternative)
9. [The Head-to-Head Comparison](#9-the-head-to-head-comparison)
   - 9.1 [FIX vs Native Venue APIs](#91-fix-vs-native-venue-apis)
   - 9.2 [FIX vs Binary Protocols (SBE, MDP 3.0)](#92-fix-vs-binary-protocols-sbe-mdp-30)
   - 9.3 [FIX vs ISO 20022 — Cross-Referenced](#93-fix-vs-iso-20022--cross-referenced)
   - 9.4 [The Comparison Table](#94-the-comparison-table)
10. [The Cymbal Bank Worked Example — A Full Session Lifecycle](#10-the-cymbal-bank-worked-example--a-full-session-lifecycle)
    - 10.1 [The Scenario and the Counterparties](#101-the-scenario-and-the-counterparties)
    - 10.2 [Logon (35=A) — Negotiating the Session](#102-logon-35a--negotiating-the-session)
    - 10.3 [Heartbeats and the TestRequest](#103-heartbeats-and-the-testrequest)
    - 10.4 [NewOrderSingle → ExecutionReport — New, Partial Fill, Fill](#104-newordersingle--executionreport--new-partial-fill-fill)
    - 10.5 [The Out-of-Sequence Scenario — Gap, ResendRequest, Gap-Fill](#105-the-out-of-sequence-scenario--gap-resendrequest-gap-fill)
    - 10.6 [Logout (35=5) — Closing the Session](#106-logout-355--closing-the-session)
    - 10.7 [The Bank's Electronic-Trading Estate — How FIX Integrates](#107-the-banks-electronic-trading-estate--how-fix-integrates)
11. [The Claims Audit — Verified, Flagged, Rejected](#11-the-claims-audit--verified-flagged-rejected)
    - 11.1 [The Verified Claims (✅)](#111-the-verified-claims-)
    - 11.2 [The Flagged Claims (⚠)](#112-the-flagged-claims-)
    - 11.3 [The Rejected Claims (❌)](#113-the-rejected-claims-)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)

---

## 1. The Overview

### 1.1 The Short Answer

The **Financial Information eXchange (FIX) protocol** is the electronic-trading messaging standard of the institutional markets: the de-facto language in which buy-side firms (asset managers, hedge funds, banks' own desks) and sell-side firms (broker-dealers, execution venues) exchange pre-trade and trade information — indications of interest, quotes, orders, execution reports, cancels and replaces, and market data. It emerged in 1992 as a bilateral initiative between **Salomon Brothers** and **Fidelity Investments** to automate US-equities IOI and execution-report traffic that had previously been conducted by telephone, was publicly released in 1995, and grew into the industry standard maintained by the **FIX Trading Community** (formerly **FIX Protocol Ltd**), a non-profit, industry-driven standards body. FIX now spans equities, foreign exchange, fixed income/rates and exchange-traded and OTC derivatives, and is used by an estimated 10,000+ institutions.

The wire format is deceptively simple: a message is a sequence of `tag=value` fields separated by the **SOH** control character (ASCII 0x01, shown as `|` in this guide), wrapped in a header (BeginString `8=`, BodyLength `9=`, MsgType `35=`), a body of application fields, and a trailer ending in the checksum `10=nnn`. A NewOrderSingle looks like `8=FIX.4.4|9=...|35=D|49=...|56=...|55=...|54=1|44=...|38=...|40=2|10=nnn|`. The genius of the format is that it is human-readable, self-describing and extensible — which is exactly why it became the lingua franca of trading while more rigid standards stayed confined to their niches.

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Full name | Financial Information eXchange (FIX) | ✅ |
| First developed | 1992, by Robert "Bob" Lamoureux and Chris Morstatt, for Fidelity Investments × Salomon Brothers US-equities IOI/execution traffic | ✅ |
| First public release | 1995, as FIX 2.7 | ✅ |
| Governing body | FIX Protocol Ltd (founded 1998 ⚠-knowledge), now operating as the FIX Trading Community (rebrand 2015 ⚠-knowledge) | ⚠ |
| Wire format | ASCII `tag=value` fields delimited by SOH (0x01); trailer ends with `10=Checksum` | ✅ |
| Session protocol | FIX 4.x native session; FIXT 1.1 from FIX 5.0; FIXP for high-performance sessions | ✅ |
| Application messages | e.g. NewOrderSingle `35=D`, ExecutionReport `35=8`, OrderCancelRequest `35=F`, OrderCancelReplaceRequest `35=G`, Quote `35=S`, MarketDataRequest `35=V`, Snapshot/FullRefresh `35=W`, IncrementalRefresh `35=X` | ✅ |
| Encodings | tag/value (classic), FIXML (XML), FAST (streaming compression), SBE (binary) | ✅ |
| Asset classes | Equities, FX, fixed income/rates, futures and derivatives | ✅ |
| Adoption | 10,000+ firms, billions of messages per day (FIX Trading Community) | ✅ |
| Typical transport | TCP (point-to-point sessions); UDP multicast for high-volume market data (FAST/SBE/MDP 3.0) | ✅ |

---

## 2. The Wire Format — Tag=Value, SOH, Header/Body/Trailer

### 2.1 The Tag=Value Field

Every FIX field is a **tag=value** pair: a numeric tag that identifies the field semantically, an equals sign, and a value whose meaning is defined by that tag. Fields are separated by the **Start of Heading** control character — **SOH**, ASCII `0x01`. In documentation the SOH is conventionally drawn as `|` (this guide) or `^` (the OnixS dictionary samples) so the message can be printed on one line. Because each field carries its own tag, messages are **self-describing**: a parser does not need a pre-agreed field order to make sense of a message, and unknown tags can be skipped or logged rather than breaking the stream.

Examples of the field vocabulary used throughout this guide (all verified against the OnixS FIX 4.4 dictionary):

| Tag | Field name | Meaning |
| --- | --- | --- |
| 8 | BeginString | FIX version of the session, e.g. `FIX.4.4` or `FIXT.1.1` |
| 9 | BodyLength | Character count of the body (from tag 35 through the last body field, including SOHs) |
| 35 | MsgType | Message type, e.g. `A` Logon, `D` New Order Single, `8` Execution Report |
| 49 | SenderCompID | Sender's CompID (who is sending) |
| 56 | TargetCompID | Target's CompID (who should receive) |
| 34 | MsgSeqNum | Message sequence number within the session |
| 52 | SendingTime | UTC timestamp of transmission |
| 43 | PossDupFlag | `Y` = possible duplicate (retransmission) |
| 122 | OrigSendingTime | Original sending time of a retransmitted message |
| 1128 | ApplVerID | Application version ID (FIXT 1.1 / FIX 5.0+ header) |
| 10 | Checksum | Three-digit checksum, always the last field of the message |

### 2.2 The Message Anatomy — Header, Body, Trailer

A FIX message is composed of three sections:

1. **The header** — up to FIX 4.4, three required fields: `8` (BeginString), `9` (BodyLength) and `35` (MsgType). From FIXT 1.1 / FIX 5.0 the header carries five or six fields: `8`, `9`, `35`, `49` (SenderCompID), `56` (TargetCompID) and the optional `1128` (ApplVerID). The header commonly also carries `34` (MsgSeqNum), `52` (SendingTime) and, on retransmissions, `43` (PossDupFlag) and `122` (OrigSendingTime).
2. **The body** — the application or session payload, defined by the MsgType. The body for a NewOrderSingle, for example, contains the instrument, side, quantity, price and order-type fields; a repeating group is preceded by its count field (e.g. `268` NoMDEntries before a market-data entries group).
3. **The trailer** — exactly one field: `10` (Checksum), always expressed as three digits (e.g. `10=128`).

### 2.3 The BodyLength and the Checksum

**BodyLength (9=)** is the character count of everything from tag `35` (inclusive) up to but excluding tag `10` (the checksum), *including* the trailing SOH of each field. The Wikipedia example (used here as the worked illustration, with SOH drawn as `|`) shows the arithmetic:

```
8=FIX.4.2|9=65|35=A|49=SERVER|56=CLIENT|34=177|52=20090107-18:15:16|98=0|108=30|10=062|
              ^  5 +    10   +    10   +   7  +         21         +  5 +    7  ^       = 65
```

**Checksum (10=)** is the sum of the ASCII values of every character in the message up to (but excluding) the checksum field itself — including each SOH character, whose ASCII value is 1 — taken modulo 256, formatted as three digits. In the example above the sum is 4158; `4158 mod 256 = 62`, so the trailer reads `10=062`. The checksum is the receiver's first-line integrity check: a corrupted field almost always breaks the arithmetic and the message is discarded or a resend is requested.

### 2.4 A Worked Message — ExecutionReport in the Clear

The Wikipedia FIX article carries a real FIX 4.2 ExecutionReport (SOH drawn as `|`) that shows the whole anatomy — header, body and trailer — in one line:

```
8=FIX.4.2|9=178|35=8|49=PHLX|56=PERS|52=20071123-05:30:00.000|11=ATOMNOCCC9990900|20=3|150=E|39=E|55=MSFT|167=CS|54=1|38=15|40=2|44=15|58=PHLX EQUITY TESTING|59=0|47=C|32=0|31=0|151=15|14=0|6=0|10=128|
```

Reading it field by field: version `FIX.4.2`, MsgType `8` (ExecutionReport), sender `PHLX`, target `PERS`, ClOrdID `11=ATOMNOCCC9990900`, ExecType `150=E` and OrdStatus `39=E` (Pending Replace in FIX 4.2), symbol `55=MSFT`, security type `167=CS` (common stock), side `54=1` (buy), order quantity `38=15`, order type `40=2` (limit), price `44=15`, time-in-force `59=0` (day), last quantity `32=0`, last price `31=0`, leaves quantity `151=15`, cumulative quantity `14=0`, average price `6=0`, checksum `10=128`. This single line demonstrates the whole point of the design: a trader who knows the tag vocabulary can read an order's entire state at a glance.

---

## 3. The History — 1992 to the FIX Trading Community

### 3.1 The 1992 Origins — Salomon Brothers × Fidelity Investments

FIX emerged in **1992** as a **bilateral initiative between Salomon Brothers and Fidelity Investments** to standardise the electronic exchange of **Indications of Interest (IOIs)** and **execution reports** for US-equities trading (fixtrading.org, "The FIX Protocol Brief History"). The problem was telephony: institutional order and fill information was communicated verbally, and Fidelity had found that broker-dealer calls could be routed to the wrong trader or simply lost when the parties hung up (Wikipedia, "Financial Information eXchange"). The protocol's specification was authored by **Robert "Bob" Lamoureux and Chris Morstatt** (Wikipedia; OnixS history article, "Development started in 1992 by Robert Lamoureux and Chris Morstatt"). The OnixS history article notes that it was originally known as the **'Salomon Brothers' exchange**, later renamed to the broader **Financial Information Exchange Protocol**.

What distinguished FIX from the start, per the FIX Trading Community's own history, was that it was **simple, open and vendor-neutral**: human-readable tag=value messages carried over standard internet protocols, easy to implement, debug and extend. It was also deliberately given away: rather than retaining private ownership, the original developers invited peer firms to participate in refining the protocol — competing firms working together on shared infrastructure, with enhancements driven by real trading use-cases and decisions by consensus.

### 3.2 The Public Release and the Industry Take-Up

- **1995 — the public release.** FIX was publicly released in 1995 as **FIX 2.7** (fixtrading.org brief history: "In 1995, FIX was publicly released"; OnixS history: "The first public version became available in 1995, known as FIX 2.7"), marking the shift from a bilateral solution to an open industry standard.
- **Network effects.** Early adopters became advocates, encouraging counterparties to implement FIX so that both sides could benefit. Because FIX was free, royalty-free and non-proprietary, and because adoption reduced integration cost and operational risk, it spread across asset classes and geographies through the mid- and late 1990s without any regulatory mandate.
- **From equities to everything.** FIX initially addressed broker-dealer ↔ institutional-client equity traffic (IOIs, orders, executions). It expanded to support straight-through processing (STP) in the post-trade space during the 1990s (OnixS history), then into foreign exchange, fixed income and derivatives.
- **Scale.** The FIX Trading Community today estimates **10,000+ financial institutions, exchanges and clearing houses** use FIX, with billions of messages exchanged per day (fixtrading.org "Who We Are": "used by well over 10,000 firms sending billions of messages a day, has truly become the way the world trades").

### 3.3 FIX Protocol Ltd and the FIX Trading Community

As adoption grew, the protocol was placed under the stewardship of a **non-profit** organisation to preserve neutrality and prevent commercial capture. The organisation was **founded as FIX Protocol Ltd (FPL) in 1998** — a date carried by MarketsWiki's FIX Trading Community article and corroborated by Markets Media's August 2013 rebrand story ("Since its launch over 15 years ago…"); the precise incorporation date could not be verified against a primary registry source in this pass, so 1998 is flagged ⚠-knowledge. FPL developed and maintained the FIX standards through committees and working groups — including the Global Technical Committee (GTC) that approves specifications such as FIXatdl — and the FAST, FIXML and SBE standards families.

On **19 August 2013** FIX Protocol Ltd announced the **rebrand and name-change to "FIX Trading Community"**, alongside a new website (Markets Media, "FIX Rebrands Itself 'FIX Trading Community'", 08.19.2013). The FIX Trading Community describes itself as "the non-profit, industry-driven standards body at the heart of global trading" and the "custodians of the FIX Protocol (FIX) messaging language", maintaining FIX's IP and facilitating the committee network (fixtrading.org "Who We Are"). OnixS's history article adds that FIX is managed under a **"Purpose Trust"**, conceived to promote protocol adoption without the risk of undue exposure or commercial capture.

### 3.4 The Version Timeline — Verified and Flagged

| Version | Year | Notes | Status |
| --- | --- | --- | --- |
| FIX 2.7 | 1995 | First public release | ✅ (OnixS history) |
| FIX 3.0 | mid-1990s | Interim release | ⚠-knowledge |
| FIX 4.0 | ~1996 | Early institutional adoption | ⚠-knowledge |
| FIX 4.1 | ~1997 | Expanded message set | ⚠-knowledge |
| FIX 4.2 | ~1998 | The long-lived workhorse of the 4.x era; monolithic spec (application semantics + encoding + session in one document) | ⚠-knowledge (monolithic-spec claim ✅ per Wikipedia) |
| FIX 4.3 | ~1999–2001 | ExecTransType (tag 20) replaced; OrdStatus/ExecType value meanings reworked (see §6.4) | ⚠-knowledge (value change ✅ per OnixS appendix note) |
| FIX 4.4 | ~2003 | Still the most widely deployed FIX version in production estates; full order-lifecycle and market-data depth | ⚠-knowledge |
| FIX 5.0 | Q4 2006 | Session layer split out; StrategyParametersGrp tags 957–960 introduced | ✅ (Wikipedia FIXatdl: "released with the release of FIX 5.0 in Q4 2006") |
| FIXT 1.1 | 2006 era | FIX Transport session protocol; used with FIX 5.0 SP2 and later | ✅ (OnixS FIX Protocol page) |
| FIX 5.0 SP1 / SP2 | 2007–2008 era | Service packs of the 5.0 application layer | ⚠-knowledge |
| FAST 1.0 / 1.1 / 1.2 | 2006 / 2007 / Feb 2009 | FIX Adapted for Streaming versions | ✅ (Wikipedia FAST timeline; fixtrading.org "released in February 2009") |
| FIXatdl 1.0 / 1.1 | 28 Mar 2008 / 2010 | Algorithmic-trading definition language | ✅ (Wikipedia FIXatdl; fixtrading.org) |
| FIXP | ~2018 era | FIX Performance Session Layer (HPWG) | ⚠-knowledge |

**Reading the timeline honestly:** the only release years this pass could pin to a citable source are 1995 (FIX 2.7), Q4 2006 (FIX 5.0), Feb 2009 (FAST 1.2), 28 Mar 2008 (FIXatdl 1.0), plus the 1998 FPL founding and 19 Aug 2013 rebrand. The intermediate 4.x years are flagged ⚠-knowledge — widely documented in industry literature but not re-verified live in this pass. No year was invented.

---

## 4. The Protocol Layers — Session vs Application

### 4.1 The Session Layer

FIX is two protocols in one. The **session layer** provides *reliable, ordered, recoverable communication between FIX counterparties* (OnixS FIX Protocol page): a point-to-point, bidirectional stream of messages between two parties within a continuous sequence-number series, over TCP (or TLS-wrapped TCP in modern estates), with heartbeats and gap/replay recovery. The **application layer** specifies the fields and messages used at the application level — the orders, executions, quotes and market data that carry business meaning. Up to and including FIX 4.2, the specification was monolithic: application semantics, message encoding and the session protocol lived in one document. From **FIX 5.0 the session layer was separated** into its own specification, the **FIX Transport Session Protocol (FIXT)**, used in conjunction with FIX 5.0 SP2 and later application versions (Wikipedia; OnixS). The **FIX Performance Session Layer (FIXP)** is the newer high-performance, high-efficiency session protocol (see §5.3).

A FIX session has two roles: the **initiator** (the side that establishes the connection and sends the first Logon) and the **acceptor** (the side that listens and responds). Both sides maintain two independent sequence-number streams — one for each direction — and each side tracks both its own outgoing sequence number and the next expected incoming sequence number.

### 4.2 The Session (Admin) Messages

The session message set (all MsgType values verified against the OnixS FIX 4.4 catalogue and message pages):

| MsgType | Message | Key fields | Purpose |
| --- | --- | --- | --- |
| A | Logon | 98 EncryptMethod (Y), 108 HeartBtInt (Y), 141 ResetSeqNumFlag, 553 Username, 554 Password, 383 MaxMessageSize, 789 NextExpectedMsgSeqNum | Authenticates and establishes the session; must be the first message sent by the initiator; HeartBtInt (heartbeat interval, seconds) is agreed by both sides |
| 0 | Heartbeat | 112 TestReqID (required if in response to a Test Request) | Monitors link status; sent when nothing else has been transmitted for HeartBtInt seconds; must echo TestReqID when answering a Test Request |
| 1 | Test Request | 112 TestReqID (Y) | Forces the counterparty to respond with a Heartbeat carrying the same TestReqID |
| 2 | Resend Request | 7 BeginSeqNo (Y), 16 EndSeqNo (Y; 0 = infinity) | Requests retransmission of a single message, a range, or everything from a sequence number onward |
| 3 | Reject | 45 RefSeqNum, 371 RefTagID, 372 RefMsgType, 373 SessionRejectReason, 58 Text | Issued when a received message passes integrity checks but violates a session-level rule (e.g. 373=1 Required tag missing, 5 Value is incorrect, 11 Invalid MsgType — the full 0–17 + 99 reason table is in the dictionary) |
| 4 | Sequence Reset | 123 GapFillFlag (Y = Gap Fill mode; N/absent = Reset mode), 36 NewSeqNo (Y) | Gap Fill mode skips over messages in resend processing (heartbeats, aged orders); Reset mode jumps the sequence forward after an unrecoverable failure |
| 5 | Logout | 58 Text | Initiates or confirms termination of the session; the initiator waits for the confirming Logout (which gives the counterparty a chance to finish gap fills) before closing the connection |

### 4.3 Sequence Numbers and the Session State

Every FIX message carries **34=MsgSeqNum** in its header. Sequence numbers are strictly increasing per direction and form the backbone of FIX reliability: the receiver validates that each inbound message has the *next expected* sequence number. Consequences, all from the FIX 4.4 dictionary pages verified above:

- **In-order receipt** → process the message, advance the expected sequence number.
- **Gap detected** (a higher-than-expected sequence number arrives) → do **not** process the out-of-sequence message; buffer it and issue a **Resend Request (35=2)** asking for the missing range. The dictionary explicitly recommends requesting `7=first-missing, 16=0` (to infinity) because "0 represents infinity" and the broader request "allows for faster recovery in the presence of certain race conditions when both sides are simultaneously attempting to recover a gap".
- **Garbled message** (fails parsing/checksum) → disregard it; the next valid message will expose the gap and trigger the same recovery.
- **Retransmissions** are marked **43=PossDupFlag=Y** and carry the original **122=OrigSendingTime** in the header, so the receiver can deduplicate.
- **Sequence Reset-GapFill (35=4, 123=Y)** is the sender's tool for skipping messages it will not resend (administrative messages such as Heartbeats and Test Requests, or aged orders the sender no longer wishes to retransmit). Its header MsgSeqNum must be the beginning of the gap-filled range, and 36=NewSeqNo tells the receiver the next sequence number to expect *after* the skipped range.
- **Sequence Reset-Reset (35=4, 123=N or absent)** jumps the sequence number arbitrarily forward and is reserved for disaster recovery after an unrecoverable application failure — the dictionary warns it "may result in the possibility of lost messages" and must NOT be used as a normal response to a Resend Request.
- **Logon-time recovery:** on session establishment the counterparties may reset their sequences (141=ResetSeqNumFlag=Y) or negotiate the next expected number (789=NextExpectedMsgSeqNum), and the initiator may begin with a Resend Request to recover anything missed since the last session.

### 4.4 Gap Detection and Recovery — ResendRequest and SequenceReset

The canonical recovery dance, as specified in the FIX 4.4 Resend Request and Sequence Reset pages:

1. Receiver detects a gap (expected seq 5, receives seq 7; or receives a garbled message).
2. Receiver sends **Resend Request (35=2)** with `7=5|16=0` (from 5 to infinity).
3. Sender replays application messages 5 and 6 with `43=Y` and `122=OrigSendingTime` set, and skips administrative messages using **Sequence Reset-GapFill (35=4)** with `123=Y|36=<next expected>` — e.g. if message 5 was a Heartbeat and 6 a TestRequest, the sender emits one GapFill with `34=5|123=Y|36=7` covering both.
4. Receiver applies the replayed messages, advances its expected sequence number, and normal processing resumes.

The FIX 4.4 dictionary page for Sequence Reset gives a concrete composite example of a resend series where messages 8, 10 and 11 are application messages while 5–7 and 9 are administrative: the retransmission stream appears as GapFill(36=8), message 8, GapFill(36=10), message 10, message 11 — and receivers must be careful to discard duplicate GapFills that would lower the next-expected sequence number.

### 4.5 Session-Level vs Application-Level Behaviour

A useful discipline for architects: **session-level rules** govern the transport of bytes — BeginString/BodyLength/Checksum integrity, sequence-number ordering, CompID matching, timing (SendingTime accuracy) and session state transitions. Violations are answered with **Reject (35=3)** carrying a SessionRejectReason (373). **Application-level (business) rules** govern whether the *content* of a validly-delivered message makes sense — unknown symbol, invalid side for the venue, rejected order, unexecutable cancel. Those are answered with business-level rejects, most commonly an **ExecutionReport (35=8)** with ExecType=8 (Rejected) and OrdRejReason (103) for orders, an **Order Cancel Reject (35=9)** for failed cancels, or the generic **Business Message Reject (35=j)**. The dictionary's Reject page states the division explicitly: "messages should be forwarded to the trading application for business level rejections whenever possible" and "If an application-level message received fulfills session-level rules, it should then be processed at a business message-level."

---

## 5. The Versions and Encodings

### 5.1 The FIX 4.x Family

The FIX 4.x line (4.0, 4.1, 4.2, 4.3, 4.4) treated the protocol as a single monolithic specification, per Wikipedia: "Originally, the FIX standard was monolithic, including application layer semantics, message encoding, and session layer in a single technical specification. It remained monolithic through FIX version 4.2. Thereafter, message encodings and session layer specifications began to be split into separate documents." Two version boundaries matter for anyone reading old message logs:

- **FIX 4.2** is the version most legacy handbooks and the canonical Wikipedia worked example use. Its OrdStatus values include 5=Replaced and 8=Rejected (verified against the OnixS FIX 4.2 dictionary).
- **FIX 4.3** replaced the ExecTransType field (tag 20) and reworked the values of **OrdStatus (39)** and **ExecType (150)** — the OnixS 4.4 pages note under both fields: "SOME VALUES HAVE BEEN REPLACED — See Appendix 6-F: 1. Replaced Field: ExecTransType (tag 20) and values in ExecType and OrdStatus fields [replaced in FIX 4.3]". The practical consequence is documented in §6.4.
- **FIX 4.4** (the version this guide's tag breakdowns use) is the most widely deployed 4.x version in production estates and the last of the classic monolithic line. Its dictionary is the reference used throughout this guide.

### 5.2 FIX 5.0, 5.0 SP1/SP2 and FIXT 1.1

**FIX 5.0** (released Q4 2006, per the Wikipedia FIXatdl article) separated the session protocol from the application layer and introduced the **FIX Transport protocol, FIXT 1.1**. The defining innovation of FIXT 1.1 is that it **decouples the session version from the application version**: one session transport can carry application messages from different FIX application versions simultaneously. The mechanism is the **1128=ApplVerID** header field: with FIXT 1.1 the BeginString is `8=FIXT.1.1`, and each message's **35= MsgType** is interpreted under the application version named by ApplVerID (e.g. `1128=8` for FIX 5.0 SP2). OnixS states the pairing plainly: "The FIX Transport Session Protocol (FIXT) is the application version independent session layer that is used in conjunction with FIX 5.0SP2 and later versions of FIX."

The header change matters at the parser level: up to FIX 4.4 the header is `8 | 9 | 35` (plus standard fields); from FIXT 1.1 / FIX 5.0 the header is `8 | 9 | 35 | 49 | 56` with optional `1128` (verified via Wikipedia's tagvalue section). 5.0 SP1 and SP2 are service-pack releases of the 5.0 application layer; SP2 is the common production target.

### 5.3 FIXP — the FIX Performance Session Layer

For the ultra-low-latency tier, the FIX High Performance Working Group developed **FIXP**, a session protocol that supports **binary message encodings for both session and application messages** and gives peers explicit control over delivery guarantees (Wikipedia). On session establishment, point-to-point peers negotiate among:

- **Recoverable** — exactly-once delivery; gaps are recovered by retransmission.
- **Idempotent** — at-most-once delivery; gaps are reported to the application, which decides whether to recover.
- **Unsequenced** — no delivery guarantees, for use when recovery happens at the application layer or via another channel.

FIXP also covers **multicast** use cases (market data) with the same primitives, and its wire format is abstracted so peers may pick an encoding — early development used SBE. Delivery guarantees may be asymmetric: a trader may enter orders over an idempotent flow while executions return over a recoverable flow, avoiding retransmission delay on the latency-critical path.

### 5.4 FIXML — the XML Encoding

**FIXML** is the XML schema for FIX messages: semantically equivalent to tag=value, but taking advantage of XML parser technology. Wikipedia notes it "is commonly used for back-office and clearing applications rather than trading" — the tag=value form dominates the latency-sensitive front office, while FIXML's hierarchical readability suits post-trade systems and integration middleware.

### 5.5 FAST — FIX Adapted for Streaming

**FAST** ("FIX Adapted for STreaming") is the FIX Trading Community's compression-oriented encoding, developed by its **Market Data Optimization Working Group** to make efficient use of bandwidth in high-volume messaging "without incurring material processing overhead or latency" (fixtrading.org FAST page). Timeline (Wikipedia FAST protocol; fixtrading.org): the working group was formed in **2004** after Mike Cormack (then CEO of Archipelago) called for action on market-data volumes; a Proof of Concept ran in **2005**; **FAST 1.0 released 2006, 1.1 in 2007, 1.2 in February 2009**. The classic tag=value format had been judged too verbose with too high a processing overhead for the multicast market-data firehose.

Mechanically, FAST compresses by removing redundancy across a stream of messages: sender and receiver share **templates** that declare each field's presence, type and *operator* (e.g. constant, default, copy from previous, delta), and values are encoded with **stop-bit encoding** so small integers occupy few bytes. Adopters over the years include NYSE Arca, CME, ISE, Nasdaq OMX, Eurex, Xetra, BATS, OPRA, B3, the Moscow Exchange and the Shanghai Stock Exchange (Wikipedia FAST). CME's patented technology affected FAST — the fixtrading.org FAST page links the "Chicago Mercantile Exchange's agreement with FIX Trading Community which provides assurances to FIX Protocol users regarding the use of CME patented technology which affects the FAST Protocol."

### 5.6 SBE — Simple Binary Encoding

**Simple Binary Encoding (SBE)** is the FIX Trading Community's low-latency binary encoding standard, developed by the FIX High Performance Working Group because tag=value was "deemed no longer fit-for-purpose since it is character based rather than binary and its variable-length fields and messages result in non-deterministic performance" (Wikipedia). SBE messages use **primitive data types native to computing systems** (uint32, int64, floats, fixed strings), so decoding needs no character parsing; fields sit at **fixed positions** driven by **templates**, so routers and filters can reach key fields without cracking the whole message; and fixed-length elements make performance deterministic. Unlike tag=value and FIXML, an SBE message is **not self-describing**: the wire carries only data plus a minimal header identifying the template, and the template metadata (an XML schema published by the FIX Trading Community) is exchanged out-of-band between peers. OnixS summarises the adoption picture: SBE "has largely superseded FAST" as the modern binary encoding of choice, and in practice SBE is the encoding behind CME's iLink order entry and MDP 3.0 market data (see §8).

### 5.7 FIXatdl — the Algorithmic Trading Definition Language

**FIX Algorithmic Trading Definition Language (FIXatdl)** is the XML-based standard for describing algorithmic-trading strategies so that a buy-side OMS can render a broker's algorithm as a working order-entry screen without per-broker development (Wikipedia FIXatdl; fixtrading.org). Origin story, per Wikipedia: as sell-side firms began offering DMA and their proprietary algorithms to clients, every broker's strategy demanded its own parameter tags and its own UI layout. FPL's **Algorithmic Trading Working Group** (formed Q3 2004) first tried standardising the parameters as the **StrategyParametersGrp** repeating group — tags **957–960** — introduced with FIX 5.0 in Q4 2006; that alone was insufficiently adopted, so the group developed the XML approach proposed in 2005 by Daniel Clayden of J.P. Morgan. **FIXatdl 1.0 was approved by the FPL Global Technical Committee on 28 March 2008**; version **1.1** followed (the fixtrading.org FIXatdl page hosts the "FIXatdl 1.1 Overview", presented to the GTC on 5 Feb 2010). A FIXatdl document describes the strategy's parameter fields, their types, validations and dependencies, and the layout of the UI controls — so the same algorithm definition renders consistently across OMS/EMS vendors.

---

## 6. The Application Messages

### 6.1 The Order Lifecycle in FIX

The heart of FIX is the **order lifecycle**, driven by four request/response message pairs plus the market-data family. The canonical equity flow (all MsgTypes verified against the OnixS FIX 4.4 catalogue):

| Step | Direction | Message | MsgType |
| --- | --- | --- | --- |
| 1 | Buy-side → Sell-side | NewOrderSingle (submit an order) | D |
| 2 | Sell-side → Buy-side | ExecutionReport (acknowledge New, then report every state change and fill) | 8 |
| 3 | Buy-side → Sell-side | OrderCancelReplaceRequest (modify: price, quantity, instructions) | G |
| 4 | Buy-side → Sell-side | OrderCancelRequest (cancel the remaining quantity) | F |
| 5 | Sell-side → Buy-side | OrderCancelReject (cancel/replace refused) | 9 |
| 6 | Buy-side → Sell-side | OrderStatusRequest (ask "where is my order?") | H |
| 7 | Either | QuoteRequest (35=R) / Quote (35=S) for the RFQ model | R / S |
| 8 | Buy-side → Sell-side | MarketDataRequest / Snapshot / IncrementalRefresh for streaming market data | V / W / X |

The key idea to internalise: **every order event is reported with an ExecutionReport (35=8)**, and the report carries two state fields — **150=ExecType** (what *this message* is: New, Trade, Pending Cancel…) and **39=OrdStatus** (the *current* state of the order after this message). The two are usually identical on state-change reports; they differ on fill reports, where ExecType=F (Trade) is reported while OrdStatus may still be 1 (Partially Filled) or 2 (Filled).

### 6.2 NewOrderSingle (35=D) — Worked Tag Breakdown

The NewOrderSingle "is used by institutions wishing to electronically submit securities and forex orders to a broker for execution" (OnixS FIX 4.4 dictionary). Verified structure — the required fields are **11=ClOrdID, the Instrument component block (55=Symbol and friends), 54=Side, 60=TransactTime, the OrderQtyData block (38=OrderQty), and 40=OrdType**; 44=Price is required for limit order types; 59=TimeInForce absence means a Day order; 1=Account is optional but ubiquitous.

| Tag | Field | Req'd | Notes (verified) |
| --- | --- | --- | --- |
| 11 | ClOrdID | Y | Unique order identifier assigned by the institution (the buy-side's own key for the order) |
| 1 | Account | N | The account the order is booked to |
| 55 | Symbol | (Instrument block) Y | Ticker / instrument identifier, e.g. AAPL; FX uses "CCY1/CCY2" form (e.g. EUR/USD) |
| 54 | Side | Y | 1=Buy, 2=Sell, 5=Sell short… (full value list in §6.4 notes) |
| 60 | TransactTime | Y | Time the order request was initiated by the trader/system |
| 38 | OrderQty | (OrderQtyData) Y | Quantity of the order |
| 40 | OrdType | Y | 1=Market, 2=Limit, 3=Stop, 4=Stop limit… |
| 44 | Price | N | Required for limit orders; the limit price |
| 59 | TimeInForce | N | 0=Day, 1=GTC, 3=IOC, 4=FOK…; absence = Day |
| 21 | HandlInst | N | How the broker should handle the order on its floor |
| 18 | ExecInst | N | Execution instructions (may be space-delimited multiple values) |
| 100 | ExDestination | N | The execution venue/destination requested |
| 15 | Currency | N | Denomination currency of the quantity |
| 58 | Text | N | Free text |

**Worked NewOrderSingle** (tags verified; values illustrative; `|` = SOH; checksum computed by the engine — the `nnn` is a placeholder, not a real checksum):

```
8=FIX.4.4|9=115|35=D|49=CYMBAL|56=BROKER1|34=7|52=20260827-09:30:05.123|
11=ORD-20260827-000123|1=EQ-USD-001|55=AAPL|54=1|60=20260827-09:30:05.100|
38=1000|40=2|44=212.50|59=0|10=nnn|
```

Read it: session FIX.4.4, MsgType D (NewOrderSingle), sent by CYMBAL to BROKER1, sequence 7, sent at 09:30:05.123; client order ID ORD-20260827-000123, account EQ-USD-001, symbol AAPL, side 1 (Buy), transact time 09:30:05.100, quantity 1000, order type 2 (Limit), price 212.50, time-in-force 0 (Day).

### 6.3 ExecutionReport (35=8) — Worked Tag Breakdown

The ExecutionReport "is used to: 1. confirm the receipt of an order 2. confirm changes to an existing order (i.e. accept cancel and replace requests) 3. relay order status information 4. relay fill information on working orders 5. relay fill information on tradeable or restricted tradeable quotes 6. reject orders 7. report post-trade fees calculations" (OnixS FIX 4.4 dictionary). Verified structure:

| Tag | Field | Req'd | Notes (verified) |
| --- | --- | --- | --- |
| 11 | ClOrdID | Y | The client's order ID (echoed from the request) |
| 37 | OrderID | Y* | The sell-side's own order number; unlike ClOrdID it does not have to change across cancels/replaces (*required in practice; listed with the required block of the message) |
| 17 | ExecID | Y | Unique identifier of this execution message, assigned by the sell-side (0 for Order Status reports) |
| 150 | ExecType | Y | Purpose of this report: 0 New, F Trade, 4 Canceled, 8 Rejected, 6 Pending Cancel, E Pending Replace… |
| 39 | OrdStatus | Y | Current state of the order chain: 0 New, 1 Partially Filled, 2 Filled, 4 Canceled… |
| 55 | Symbol | (Instrument) Y | Instrument echoed |
| 54 | Side | Y | Side echoed |
| 38 | OrderQty | N | Order quantity (current version of the order) |
| 40 | OrdType / 44 Price / 59 TimeInForce | N | Order parameters echoed |
| 32 | LastQty | N | Quantity of this (last) fill — required when ExecType = Trade/Trade Correct |
| 31 | LastPx | N | Price of this (last) fill — required when ExecType = Trade/Trade Correct |
| 151 | LeavesQty | Y | Quantity open for further execution (0 once the order is no longer active) |
| 14 | CumQty | Y | Cumulative executed quantity for the chain of orders |
| 6 | AvgPx | Y | Average price of the executions |
| 1 | Account | N | Account echoed |
| 60 | TransactTime | N | Time the transaction represented by this report occurred |
| 103 | OrdRejReason | N | Order rejection reason (when ExecType = 8 Rejected) |

The dictionary states the master identity: **OrderQty = CumQty + LeavesQty** (with exceptions for terminal states such as Canceled/Expired/Rejected where LeavesQty may be 0). It also specifies that when an order is replaced, the quantities on the new order's reports are cumulative across the chain (order A's fills plus order B's fills).

**Worked ExecutionReport — a fill** (tags verified; values illustrative):

```
8=FIX.4.4|9=141|35=8|49=BROKER1|56=CYMBAL|34=41|52=20260827-09:30:06.002|
11=ORD-20260827-000123|37=BRK-ORD-889001|17=EXEC-889001-01|150=F|39=1|
55=AAPL|54=1|38=1000|40=2|44=212.50|32=400|31=212.48|151=600|14=400|6=212.48|
60=20260827-09:30:06.000|10=nnn|
```

Read it: ExecutionReport from BROKER1 to CYMBAL, sequence 41; ClOrdID ORD-20260827-000123, broker OrderID BRK-ORD-889001, ExecID EXEC-889001-01; ExecType F (Trade), OrdStatus 1 (Partially Filled); 400 shares filled at 212.48, 600 leaves, 400 cum, avg price 212.48. The order is not yet done — OrdStatus 1 tells the OMS to keep working it.

### 6.4 OrdStatus and ExecType — the Verified Value Tables

**The verification result up front:** the commonly quoted lifecycle `0=New, 1=Partially filled, 2=Filled, 4=Canceled, 5=Rejected, 6=Pending Cancel, A=Pending New, E=Pending Replace` is **correct except for "5=Rejected"** — in both FIX 4.2 and FIX 4.4 the value 5 is **Replaced** (removed from use in 4.4), and **Rejected is 8**. "5=Rejected" is a persistent myth from pre-4.3 documentation; the verified tables below are the ground truth.

**OrdStatus <39>, FIX 4.4** (verified against the OnixS FIX 4.4 dictionary — the task-prompt list verified item by item):

| Value | Meaning | Task-list claim | Verdict |
| --- | --- | --- | --- |
| 0 | New | 0=New | ✅ |
| 1 | Partially filled | 1=Partially filled | ✅ |
| 2 | Filled | 2=Filled | ✅ |
| 3 | Done for day | — | (omitted from task list) |
| 4 | Canceled | 4=Canceled | ✅ |
| 5 | Replaced (removed/replaced in 4.4) | 5=Rejected | ❌ — Replaced, not Rejected |
| 6 | Pending Cancel | 6=Pending Cancel | ✅ |
| 7 | Stopped | — | |
| 8 | Rejected | — | Rejected is **8**, not 5 |
| 9 | Suspended | — | |
| A | Pending New | A=Pending New | ✅ |
| B | Calculated | — | |
| C | Expired | — | |
| D | Accepted for bidding | — | |
| E | Pending Replace | E=Pending Replace | ✅ |

**OrdStatus <39>, FIX 4.2** (verified against the OnixS FIX 4.2 dictionary): 0=New, 1=Partially filled, 2=Filled, 3=Done for day, 4=Canceled, 5=Replaced, 6=Pending Cancel, 7=Stopped, 8=Rejected, 9=Suspended, A=Pending New, B=Calculated, C=Expired, D=Accepted for bidding, E=Pending Replace. The 4.2 table is the same as 4.4 except that 5=Replaced is still formally listed in 4.2 while 4.4 marks it removed, and 4.4 adds the replaced-ExecTransType values to the field's history. In both versions **8=Rejected**.

**ExecType <150>, FIX 4.4** (verified against the OnixS FIX 4.4 dictionary): 0=New, 1=Partial fill (replaced), 2=Fill (replaced), 3=Done for day, 4=Canceled, 5=Replaced, 6=Pending Cancel, 7=Stopped, 8=Rejected, 9=Suspended, A=Pending New, B=Calculated, C=Expired, D=Restated, E=Pending Replace, F=Trade, G=Trade Correct, H=Trade Cancel, I=Order Status. The values 1 and 2 (Partial fill/Fill) were replaced by F (Trade) in FIX 4.3 — modern fills are reported with `150=F` regardless of whether they complete the order.

**Side <54> values** (verified): 1=Buy, 2=Sell, 3=Buy minus, 4=Sell plus, 5=Sell short, 6=Sell short exempt, 7=Undisclosed (IOI/List only), 8=Cross, 9=Cross short, A=Cross short exempt, B=As Defined (multileg), C=Opposite (multileg), D=Subscribe (CIV), E=Redeem (CIV), F=Lend (financing), G=Borrow (financing).

**OrdType <40> values** (verified): 1=Market, 2=Limit, 3=Stop, 4=Stop limit, 6=With or without, 8=Limit with or without, 9=On basis, D=Previously quoted, E=Previously indicated, G=Forex-Swap, I=Funari, J=Market If Touched, K=Market with leftover as limit, L=Previous fund valuation point (CIV), M=Next fund valuation point (CIV), P=Pegged. (5, A, B, C, F, H removed — "no longer used" per the dictionary.)

**TimeInForce <59> values** (verified): 0=Day (or session), 1=Good Till Cancel (GTC), 2=At the Opening (OPG), 3=Immediate or Cancel (IOC), 4=Fill or Kill (FOK), 5=Good Till Crossing (GTX), 6=Good Till Date (GTD), 7=At the Close. Absence of the field = Day order.

### 6.5 OrderCancelReplaceRequest (35=G) and OrderCancelRequest (35=F)

**Order Cancel/Replace Request <G>** ("a.k.a. Order Modification Request") changes the parameters of an existing order — reduce/increase quantity, change limit price, change instructions — and must carry the full set of order fields with only the changed values altered (verified: "All of the application-level fields in the original order should be retransmitted with the original values… except the fields that are being changed"). Structure (verified): **41=OrigClOrdID (Y)** — the ClOrdID of the previous non-rejected order, **11=ClOrdID (Y)** — the *new* unique ID of the replacement, 37=OrderID (N), Instrument block (Y), Side 54 (Y), and the order parameters. The ClOrdID chains: each replace references the previous version via OrigClOrdID, and the sell-side may chain pessimistically (OrigClOrdID = last *accepted* ClOrdID) on its ExecutionReports. Note the guidance: use **F** to cancel all remaining quantity, use **G** to reduce — the F page is explicit that "the Order Cancel/Replace Request <G> should be used to partially cancel (reduce) an order".

**Order Cancel Request <F>** cancels *all* of the remaining quantity of an existing order (verified). Structure (verified): **41=OrigClOrdID (Y)**, **11=ClOrdID (Y)** — the cancel request's own unique ID, 37=OrderID (N), Instrument block (Y), Side 54 (Y), TransactTime 60 (Y), OrderQtyData (Y). The recommended response sequence: an ExecutionReport with ExecType=6 (Pending Cancel) unless the cancel can be immediately accepted (ExecType=4 Canceled) or rejected (Order Cancel Reject <9>).

### 6.6 Quotes — QuoteRequest (35=R) and Quote (35=S)

In the RFQ (request-for-quote) model used across fixed income, FX and listed options, FIX provides the **Quote Request <R>** → **Quote <S>** pair. The **Quote <S>** message (verified structure) is the market maker's response: **117=QuoteID (Y)**, **131=QuoteReqID (N — required when the quote answers a Quote Request)**, the Instrument block, **132=BidPx, 133=OfferPx, 134=BidSize, 135=OfferSize**, and 301=QuoteResponseLevel. The dictionary notes the quote is for a *single product*, that quotes can be sent unsolicited (identified by the absence of QuoteReqID), that an order can be generated against a quote using OrdType=D (Previously Quoted) with the QuoteID, and that a quote is cancelled either via **Quote Cancel <Z>** or by re-sending the Quote with bid/offer prices and sizes all zero. For streaming two-sided markets the dictionary steers implementers to the market-data messages instead: "The quote should not be used in tradeable and restricted tradeable quoting markets… to broadcast quotes to market participants."

### 6.7 Market Data — Request (35=V), Snapshot (35=W), Incremental (35=X)

The market-data family implements the subscription model for real-time quotes, trades, volume and open interest. **Market Data Request <V>** (verified structure): **262=MDReqID (Y)**, **263=SubscriptionRequestType (Y)** — 0=Snapshot, 1=Snapshot + Updates, 2=Disable previous Snapshot + Updates; **264=MarketDepth (Y)** — depth of the book requested; 265=MDUpdateType (required for Snapshot+Updates; 0=Full Refresh, 1=Incremental Refresh), 266=AggregatedBook, the **267=NoMDEntryTypes** repeating group of **269=MDEntryType** (0=Bid, 1=Offer, 2=Trade, 4=Opening, 5=Closing, 6=Settlement, B=Trade Volume, C=Open Interest, etc.), and the 146=NoRelatedSym group of Instruments requested.

**Market Data Snapshot/Full Refresh <W>** (verified structure): 262=MDReqID (conditionally required), Instrument block (Y), **268=NoMDEntries (Y)** — the count of entries — then per entry **269=MDEntryType (Y, first in the group)**, 270=MDEntryPx, 271=MDEntrySize, 272=MDEntryDate, 273=MDEntryTime, 274=TickDirection, 276=QuoteCondition, 277=TradeCondition, 290=MDEntryPositionNo, 346=NumberOfOrders. A Full Refresh contains the *complete* requested state of one instrument — the dictionary: "every update to a Market Data Entry results in a new Market Data message that contains the entirety of the data requested for that instrument".

**Market Data Incremental Refresh <X>** (verified structure): 262=MDReqID, **268=NoMDEntries**, and per entry **279=MDUpdateAction (Y)** — 0=New, 1=Change, 2=Delete; 278=MDEntryID (the entry's stable ID for change/delete references), 280=MDEntryRefID (the ID being changed when the ID itself changes), 269=MDEntryType, 270=MDEntryPx, 271=MDEntrySize, 55=Symbol. Bandwidth-conservation rules verified from the dictionary: an instrument need only be identified when its first entry is created; an entry may default to the previous entry's instrument; and a Change needs only the fields that changed plus MDUpdateAction.

---

## 7. The Trading Ecosystem — Where FIX Lives

FIX is the connective tissue of a five-layer trading ecosystem. (For the deeper architecture of each layer, cross-reference the sibling guides [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) and [Capital Markets Architecture](capital_markets_architecture_guide.md); this section is the FIX-shaped view.)

### 7.1 OMS and EMS

On the buy side, the **Order Management System (OMS)** is the system of record for orders: it captures the order from the portfolio manager, enforces compliance and pre-trade checks, tracks the order through its lifecycle via ExecutionReports, and feeds allocations downstream. The **Execution Management System (EMS)** is the trader's low-latency execution cockpit: real-time market data, order tickets, algos, and connectivity to brokers and venues. In the FIX world the OMS/EMS is typically the **FIX engine's application layer**: it generates NewOrderSingle/OrderCancelRequest/OrderCancelReplaceRequest messages, consumes ExecutionReports, and maintains the ClOrdID ↔ OrderID mapping. The FIXatdl story in §5.7 exists precisely because the OMS must render broker algorithms without bespoke screens per broker.

### 7.2 Algorithmic Trading and Smart Order Routing

**Algorithmic trading** strategies (VWAP, TWAP, implementation shortfall, participation, liquidity-seeking) live either at the sell-side broker (accessed by the buy-side as a black box via FIX, with parameters described by FIXatdl) or at the buy-side's own EMS. **Smart Order Routing (SOR)** is the buy-side/agency-side logic that decides which venue, which order type and what size to send where, based on real-time market data (FIX V/W/X feeds) and venue analytics; the resulting child orders go out as FIX NewOrderSingles to the chosen destinations. The FIX 4.4 dictionary even exposes SOR-era order attributes directly on the NewOrderSingle: 847=TargetStrategy, 848=TargetStrategyParameters, 849=ParticipationRate (mandatory for "Participate" strategies), and the peg/discretion instruction blocks.

### 7.3 Matching Engines, ECN/ATS and Execution Venues

The **matching engine** is the venue's core — the component that actually crosses orders. Exchanges, **ECNs** (electronic communication networks) and **ATSs** (alternative trading systems) expose FIX (or FIX-derived binary) order-entry gateways in front of their matching engines, and market-data feeds behind them. Venue FIX dialects are codified in **"FIX Rules of Engagement" (RoE)** documents — the OnixS FIX Protocol page notes that venues publish FIX RoE specifications defining "FIX Dictionary" dialect variants that extend or specialise the base FIX standard (venue-specific tags, modified validations, session parameters). CME's **iLink** is the canonical example: "Built on the industry-standard FIX protocol" (CME Group, "Develop to Globex"), iLink offers two gateway access models — the **Convenience Gateway (CGW)**, which routes all order-entry messages to all market segments transparently, and the **Market Segment Gateway (MSGW)**, a dedicated gateway per market segment; the modern incarnation is iLink binary order entry over SBE.

### 7.4 Broker/Dealer Connectivity Hubs

Sell-side brokers and connectivity vendors operate **FIX hubs** that concentrate hundreds of buy-side sessions into a few managed gateways: a single broker-side hub terminates each client's FIX session, performs drop-copy (sending each client a copy of its own execution traffic for risk/ops monitoring — the same concept as CME's Drop Copy service), applies throttling and risk limits per client, and fans messages out to trading desks, algos and DMA infrastructure. On the buy side, **FIX network providers** (the managed connectivity layer between firms) supply the TLS-secured FIX session connectivity, certification testing and sometimes hosted FIX engines, so a buy-side OMS can reach hundreds of brokers over one logical connection fabric. The session mechanics of §4 are exactly what makes this hub-and-spoke topology work: every leg is an independent, recoverable, sequenced session.

---

## 8. Performance and the Modern Variants

### 8.1 The Latency Problem

Classic tag=value FIX over TCP is verbose: every field repeats its tag and '=' on the wire, timestamps and prices are ASCII, and TCP's reliable ordered stream adds head-of-line behaviour under load. For order entry at human/algorithmic speeds this is fine; for the high-frequency market-data tier it is not. The FIX community's answer has been a ladder of encodings and transports — FAST compression, then SBE binary, then whole session protocols (FIXP) and venue-native binary platforms (CME MDP 3.0) that keep the FIX *semantics* while abandoning the ASCII wire format.

### 8.2 FIX/FAST Compression

**FAST** (§5.5) attacks bandwidth: templates plus stop-bit encoding plus cross-message operators (constant, default, copy, delta) strip away the redundant tag names and repeating values, typically compressing market-data streams by an order of magnitude. It is a *streaming* technology — "used mostly for sending multicast market data via UDP connections" (Wikipedia) — and its Session Control Protocol (SCP) manages the multicast recovery channels. The trade-off: the decoder must hold the same template and previous-value state as the encoder, so FAST feeds are stateful and error recovery is more involved than classic FIX's resend model.

### 8.3 Simple Binary Encoding in Production — CME

**SBE** (§5.6) is the modern binary encoding of choice and the one the FIX Trading Community's factsheet promotes as its low-latency encoding standard. Its production showcase is CME Group: the iLink order-entry interface has moved to **iLink Binary Order Entry** (SBE-encoded), and the market-data platform is built on it. Because SBE messages map directly onto native C/C++ structs — "given a pointer to a message buffer, accessing non-composite fields of the message amount to type-casting it to a pointer to structure and accessing structure members directly" (Wikipedia) — decode cost is effectively zero, which is what makes sub-microsecond market-data processing feasible.

### 8.4 CME MDP 3.0 — the Binary Multicast Firehose

**CME Market Data Platform 3.0 (MDP 3.0)** is CME Group's high-throughput market-data dissemination platform — the reference example of a *competing binary multicast protocol* in the FIX world (CME Group, "Develop to Globex"; Databento dataset documentation). Verified characteristics:

- **Positioning:** "Market Data Platform (MDP) 3.0 is a low latency and direct data access solution designed to improve customers' ability to process and act on real-time price information" (CME Group).
- **Encoding:** "MDP 3.0 includes the introduction of Simple Binary Encoding (SBE) and Event Driven Messaging to the CME Group Market Data Platform" (Databento). SBE "is based on simple primitive encoding, and is optimized for low bandwidth, low latency, and direct data access".
- **Model:** event-based dissemination of bid, ask, trade and statistical data, with **recovery and support services** for market data processing; delivered over **UDP multicast** (the classic high-volume market-data transport, per the FAST/SCP heritage), with the incremental feed carrying every book event and separate recovery channels for gap fill.
- **Granularity change:** "Since March 2017, MDP 3.0 has changed from providing aggregated depth at every price level (like CME's legacy FAST feed) to providing full granularity of every order event for every instrument's direct book" (Databento) — i.e. order-by-order (MBO) book reconstruction rather than aggregated price levels.
- **Coverage:** "MDP 3.0 is the sole data feed for all instruments traded on CME Globex, including futures, options, spreads and combinations" (Databento), covering the CME, CBOT, NYMEX and COMEX markets, and it is also used for EBS and BrokerTec market data (CME Group).

For the architect, MDP 3.0 is the honest demonstration that when throughput and determinism dominate, the industry abandons ASCII FIX on the market-data side while keeping the *semantics* (incremental updates, entries, recovery) that FIX invented.

### 8.5 REST/JSON — the Low-Friction Alternative

For integration tiers that do not need session-level state machines — retail brokerage APIs, mobile apps, internal dashboards, post-trade reporting — **REST/JSON APIs** have become the modern alternative to FIX. The FIX Trading Community itself has published standard mappings between FIX and JSON (and Google Protocol Buffers and ASN.1) as alternative encodings (Wikipedia), and venues increasingly offer both: a FIX session for professional DMA clients and REST/JSON endpoints for lower-touch integration. The comparison is not either/or — a typical bank estate runs FIX sessions to brokers and venues for the trading path, FIXML or JSON for the back office, and REST/JSON for onboarding, reference data and reporting.

---

## 9. The Head-to-Head Comparison

### 9.1 FIX vs Native Venue APIs

Every major venue exposes a **native API** — often FIX-derived (CME iLink, Eurex T7, Nasdaq FIX-based order entry) but sometimes proprietary binary (e.g. the various exchange-specific gateways). Native APIs win on venue-specific features (special order types, auction mechanics, position/tier-specific fields), latency (binary frames, minimal ceremony) and vendor support; they lose on portability — each venue is a new dialect, a new certification, a new operational burden. FIX wins on standardisation: one message vocabulary and session model across hundreds of venues and brokers, which is precisely why it became the *de-facto* standard rather than any single venue's API.

### 9.2 FIX vs Binary Protocols (SBE, MDP 3.0)

Classic FIX and the binary variants are not really rivals — **SBE/MDP 3.0 are FIX-family technologies** (SBE is a FIX Trading Community standard) that replace the ASCII encoding with binary templates while keeping FIX semantics. The genuine trade-offs: tag=value is self-describing and debuggable by eye (you can read a message in a log); SBE is faster and deterministic but requires out-of-band schema management and is illegible on the wire. FAST sits between: compressed and stream-oriented, but stateful and template-coupled. The rule of thumb used across the industry: **tag=value FIX for order entry and lower-frequency flows; FAST or SBE for market data; FIXP or venue binary for the extreme low-latency tier.**

### 9.3 FIX vs ISO 20022 — Cross-Referenced

FIX and ISO 20022 serve different ends of the trade lifecycle, and the sibling guide [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) is the authoritative reference for the ISO 20022 side (its message catalogue, business-process modelling and the payments/clearing migration); this guide only sketches the boundary. ISO 20022 is the rich, XML/JSON-based standard for *post-trade* financial messaging — payments, cash management, securities settlement instructions, trade confirmation and reporting — governed by ISO with SWIFT as the registration authority for much of it. FIX is the *pre-trade/trade* front-office standard: quotes, orders and executions, designed for low latency and high message rates, governed by the FIX Trading Community. The two meet in the middle: a trade executed over FIX produces settlement instructions that travel over ISO 20022 (or SWIFT MT/MX), and firms must map FIX trade data into ISO 20022 confirmation/settlement messages — one of the classic integration projects in the [Payment Rails](payment_rails_guide.md) and [Capital Markets Architecture](capital_markets_architecture_guide.md) clusters. Put simply: **FIX speaks the trade conversation; ISO 20022 speaks the money-and-settlement conversation.**

### 9.4 The Comparison Table

| Dimension | Classic FIX (tag=value) | Native venue API | Binary FIX-family (SBE/FAST) | CME MDP 3.0 | REST/JSON | ISO 20022 |
| --- | --- | --- | --- | --- | --- | --- |
| Scope | Pre-trade/trade, all asset classes | Venue-specific order entry/market data | Encoding of FIX semantics | Market data (futures/options, EBS, BrokerTec) | Low-friction integration, reporting | Post-trade: payments, settlement, confirmations |
| Wire format | ASCII tag=value, SOH-delimited | Proprietary/binary | Binary, template-driven | SBE binary over UDP multicast | JSON over HTTP(S) | XML/JSON (ISO 20022) |
| Self-describing | Yes | No | No (schema out-of-band) | No (Globex SBE schemas) | Yes (schema-optional) | Yes (XSD) |
| Session/reliability | Logon/heartbeat/seqnum/resend; TCP | Venue-specific | FIXP or venue session | Multicast + recovery channels | HTTP semantics | Store-and-forward, queues, SWIFT |
| Latency profile | Milliseconds (fine for order entry) | Low (native) | Sub-microsecond decode | Low-latency, high-throughput | Seconds — not for DMA | Not latency-sensitive |
| Governance | FIX Trading Community | Venue | FIX Trading Community | CME Group | Vendor/platform | ISO + SWIFT (see [ISO 20022 guide](iso_20022_core_processes_guide.md)) |
| Best for | Multi-broker order routing, standardisation | Maximum venue feature access | High-frequency market data | Full-order-book futures/options feeds | Onboarding, dashboards, mobile | Settlement, clearing, regulatory reporting |
| Reference | This guide §2–§6 | [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) | This guide §5.5–§5.6, §8 | This guide §8.4 | This guide §8.5 | [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) |

---

## 10. The Cymbal Bank Worked Example — A Full Session Lifecycle

### 10.1 The Scenario and the Counterparties

**Cymbal Bank** (the fictional Singapore-based bank persona used across this repository — see the conventions in [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) and [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md)) runs an institutional equities desk. Its OMS connects to **Meridian Securities** (a fictional sell-side broker) over a **FIX 4.4** session:

- Cymbal Bank is the **session initiator** (CompID `49=CYMBAL`), Meridian is the **acceptor** (`56=BROKER1`).
- Transport: TCP, TLS-wrapped; **HeartBtInt = 30** seconds (tag 108), agreed at logon.
- Encryption method `98=0` (none — confidentiality is provided by the TLS layer below FIX, per common practice).
- Sequence numbers reset each morning (`141=Y`) — a daily-session convention.
- In every message below, `|` represents SOH and `10=nnn` is a placeholder for the engine-computed checksum; all tags and values are from the verified FIX 4.4 vocabulary of §4–§6. The CompIDs, order IDs and timestamps are illustrative.

### 10.2 Logon (35=A) — Negotiating the Session

**Step 1 — Cymbal Bank → Meridian (initiator logon, seq 1):**

```
8=FIX.4.4|9=91|35=A|49=CYMBAL|56=BROKER1|34=1|52=20260827-01:00:00.000|
98=0|108=30|141=Y|553=cymbaltrader|554=********|10=nnn|
```

Field by field: version FIX.4.4; MsgType A (Logon); sender CYMBAL, target BROKER1; sequence 1; SendingTime 01:00:00.000 UTC (before the SGX open); EncryptMethod 0; **HeartBtInt 30** — the heartbeat interval Cymbal proposes; **ResetSeqNumFlag=Y** — both sides should reset their sequence numbers to 1 for this session; Username/Password for the application-level authentication (the dictionary notes "minimal security exists without transport-level encryption", hence TLS underneath).

**Step 2 — Meridian → Cymbal Bank (acceptor logon, seq 1):**

```
8=FIX.4.4|9=67|35=A|49=BROKER1|56=CYMBAL|34=1|52=20260827-01:00:00.120|
98=0|108=30|10=nnn|
```

Meridian echoes `108=30` — the dictionary requires that HeartBtInt "should be agreed upon by the two firms and specified by the Logon initiator and echoed back by the Logon acceptor". The session is now established: both sides expect the next inbound sequence number 2.

### 10.3 Heartbeats and the TestRequest

With HeartBtInt = 30, whichever side transmits nothing for 30 seconds must emit a **Heartbeat (35=0)** to keep the link alive. Suppose the desk is quiet between 01:00:30 and 01:01:00:

**Cymbal Bank → Meridian, seq 2 (Heartbeat):**

```
8=FIX.4.4|9=58|35=0|49=CYMBAL|56=BROKER1|34=2|52=20260827-01:00:30.000|10=nnn|
```

Now suppose Meridian's side goes silent beyond HeartBtInt + transmission time. Cymbal Bank's engine sends a **Test Request (35=1)** to force a response and verify the link:

**Cymbal Bank → Meridian, seq 3 (TestRequest):**

```
8=FIX.4.4|9=72|35=1|49=CYMBAL|56=BROKER1|34=3|52=20260827-01:02:00.000|112=PING-01|10=nnn|
```

**Meridian → Cymbal Bank, seq 2 (Heartbeat answering the TestRequest):**

```
8=FIX.4.4|9=67|35=0|49=BROKER1|56=CYMBAL|34=2|52=20260827-01:02:00.150|112=PING-01|10=nnn|
```

The dictionary requires the answering Heartbeat to echo `112=PING-01` so the requester knows it is a response to the Test Request and not a routine timeout heartbeat.

### 10.4 NewOrderSingle → ExecutionReport — New, Partial Fill, Fill

At 09:30:05 (market open) the Cymbal Bank equity desk buys 1,000 shares of AAPL, limit 212.50, day order. Cymbal Bank sends the order (this is the message broken down in §6.2):

**Cymbal Bank → Meridian, seq 4 (NewOrderSingle):**

```
8=FIX.4.4|9=115|35=D|49=CYMBAL|56=BROKER1|34=4|52=20260827-09:30:05.123|
11=ORD-20260827-000123|1=EQ-USD-001|55=AAPL|54=1|60=20260827-09:30:05.100|
38=1000|40=2|44=212.50|59=0|10=nnn|
```

**Meridian → Cymbal Bank, seq 3 (ExecutionReport — New):** Meridian acknowledges receipt. ExecType=0 (New) and OrdStatus=0 (New); no fill yet, so LastQty/LastPx are absent, CumQty=0, LeavesQty=1000.

```
8=FIX.4.4|9=128|35=8|49=BROKER1|56=CYMBAL|34=3|52=20260827-09:30:05.410|
11=ORD-20260827-000123|37=BRK-ORD-889001|17=EXEC-889001-00|150=0|39=0|
55=AAPL|54=1|38=1000|40=2|44=212.50|59=0|151=1000|14=0|6=0|
60=20260827-09:30:05.400|10=nnn|
```

**Meridian → Cymbal Bank, seq 4 (ExecutionReport — partial fill):** 400 shares fill at 212.48. ExecType=F (Trade), OrdStatus=1 (Partially Filled); LastQty=400, LastPx=212.48, CumQty=400, LeavesQty=600.

```
8=FIX.4.4|9=141|35=8|49=BROKER1|56=CYMBAL|34=4|52=20260827-09:30:06.002|
11=ORD-20260827-000123|37=BRK-ORD-889001|17=EXEC-889001-01|150=F|39=1|
55=AAPL|54=1|38=1000|40=2|44=212.50|32=400|31=212.48|151=600|14=400|6=212.48|
60=20260827-09:30:06.000|10=nnn|
```

**Meridian → Cymbal Bank, seq 5 (ExecutionReport — fill):** the remaining 600 shares fill at 212.51. ExecType=F, OrdStatus=2 (Filled); LastQty=600, LastPx=212.51, CumQty=1000, LeavesQty=0, AvgPx=212.498 — the identity OrderQty = CumQty + LeavesQty (1000 = 1000 + 0) holds.

```
8=FIX.4.4|9=141|35=8|49=BROKER1|56=CYMBAL|34=5|52=20260827-09:30:07.515|
11=ORD-20260827-000123|37=BRK-ORD-889001|17=EXEC-889001-02|150=F|39=2|
55=AAPL|54=1|38=1000|40=2|44=212.50|32=600|31=212.51|151=0|14=1000|6=212.498|
60=20260827-09:30:07.500|10=nnn|
```

Cymbal Bank's OMS reconciles: it matches ClOrdID ORD-20260827-000123 across all three reports, tracks OrderID BRK-ORD-889001, and books 1,000 shares at an average price of 212.498 against account EQ-USD-001.

### 10.5 The Out-of-Sequence Scenario — Gap, ResendRequest, Gap-Fill

Now the failure case the session layer exists for. Later that morning Cymbal Bank sends a second order (ORD-20260827-000124). Meridian sends back its ExecutionReport (seq 6) acknowledging it, and a few seconds later a report for a third order (seq 7). **Message 6 is lost in the network** (packet drop, TLS retransmit timeout, whatever the gremlin). Cymbal Bank's engine receives seq 7 while expecting seq 6:

**Cymbal Bank detects the gap.** Per the dictionary: do NOT process the out-of-sequence message; buffer it; send a Resend Request for the missing range — the recommended form is `7=<first missing>|16=0` (0 = to infinity):

**Cymbal Bank → Meridian, seq 5 (ResendRequest):**

```
8=FIX.4.4|9=67|35=2|49=CYMBAL|56=BROKER1|34=5|52=20260827-10:15:03.100|7=6|16=0|10=nnn|
```

**Meridian replays the gap.** In this scenario the missing message 6 turns out to have been an *administrative* Heartbeat, which the dictionary says the sender is not obliged to resend ("a number of administrative messages are skipped and not resent (such as Heart Beats, Test Requests)"). Meridian therefore sends a **SequenceReset-GapFill (35=4)** with its header sequence at the beginning of the skipped range, `123=Y`, `36=7` (the next sequence to expect), plus `43=Y` and `122=OrigSendingTime` because it is a retransmission:

**Meridian → Cymbal Bank, seq 6 (SequenceReset-GapFill):**

```
8=FIX.4.4|9=100|35=4|49=BROKER1|56=CYMBAL|34=6|43=Y|52=20260827-10:15:03.210|
122=20260827-10:14:59.998|123=Y|36=7|10=nnn|
```

Then Meridian re-sends the genuine application message 7 with PossDupFlag=Y and OrigSendingTime:

**Meridian → Cymbal Bank, seq 7 (retransmitted ExecutionReport):**

```
8=FIX.4.4|9=130|35=8|49=BROKER1|56=CYMBAL|34=7|43=Y|52=20260827-10:15:03.220|
122=20260827-10:15:00.500|11=ORD-20260827-000125|37=BRK-ORD-889003|17=EXEC-889003-00|
150=0|39=0|55=AAPL|54=1|38=500|40=2|44=213.00|59=0|151=500|14=0|6=0|
60=20260827-10:15:00.490|10=nnn|
```

Cymbal Bank's engine now has a contiguous stream: it advances its expected inbound sequence from 6 to 8, processes the replayed ExecutionReport (deduplicating on ClOrdID/ExecID if a copy had already arrived), and normal processing resumes. Note what did **not** happen: no order was double-executed (the GapFill skipped only the Heartbeat), and no report was silently lost (the ExecutionReport was replayed). That is the whole value proposition of the FIX session layer — recovery without ambiguity.

### 10.6 Logout (35=5) — Closing the Session

At end of day, Cymbal Bank initiates logout. The dictionary's rule: the logout initiator waits for the confirming Logout before closing the TCP connection, giving the counterparty a chance to finish any gap fills.

**Cymbal Bank → Meridian, seq 6 (Logout):**

```
8=FIX.4.4|9=76|35=5|49=CYMBAL|56=BROKER1|34=6|52=20260827-09:59:59.000|58=End of day|10=nnn|
```

**Meridian → Cymbal Bank, seq 8 (Logout acknowledgement):**

```
8=FIX.4.4|9=84|35=5|49=BROKER1|56=CYMBAL|34=8|52=20260827-09:59:59.300|58=Logout acknowledgement|10=nnn|
```

Both sides persist their outbound sequence numbers (Cymbal 7 next, Meridian 9 next) so that tomorrow's session can either reset (`141=Y`) or resume with a ResendRequest covering the gap since last session — the dictionary's recommended recovery on re-establishment.

### 10.7 The Bank's Electronic-Trading Estate — How FIX Integrates

A bank like Cymbal Bank does not run FIX in isolation; the protocol is the middle layer of an estate that the sibling guides describe in full ([Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md), [Capital Markets Architecture](capital_markets_architecture_guide.md)). The FIX-shaped integration points:

- **OMS → FIX engine:** the OMS owns the order lifecycle and ClOrdID allocation; the FIX engine owns the sessions. The seam between them is the application message set of §6 — the OMS never sees SOH bytes, the engine never sees portfolio books.
- **Pre-trade risk checks:** before a NewOrderSingle leaves the building, the OMS/risk layer validates limits (notional, quantity, concentration, instrument eligibility) against the bank's risk systems — the same obligations the MAS-oriented guides map for Cymbal Bank ([MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md)). FIX enters the picture as the enforcement point: a rejected pre-trade check means *no FIX message is sent at all*; a sell-side rejection arrives back as ExecType=8 with OrdRejReason.
- **Connectivity layer:** TLS-terminating gateways, FIX session monitoring (heartbeat statistics, resend counters, sequence-gap alarms — the ops team watches the §10.5 machinery), drop-copy streams into the risk and surveillance systems, and certified FIX engines from commercial vendors (the OnixS page notes "FIX standards are technical specifications rather than concrete implementations" — engines are the implementations).
- **Market data:** the bank's algos and SOR consume FIX market data (V/W/X) or venue-native feeds (e.g. MDP 3.0 for futures) and translate them into a normalized internal book.
- **Post-trade:** fills captured from ExecutionReports flow into allocation and settlement, crossing into ISO 20022 territory — the boundary described in §9.3 and the [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) guide.

---

## 11. The Claims Audit — Verified, Flagged, Rejected

### 11.1 The Verified Claims (✅)

| # | Claim | Source |
| --- | --- | --- |
| 1 | FIX emerged in 1992 as a bilateral Salomon Brothers × Fidelity Investments initiative to standardise IOIs and execution reports for US equities | fixtrading.org, "The FIX Protocol Brief History" (https://fixtrading.org/who-we-are/the-fix-protocol-brief-history/) |
| 2 | The specification was authored by Robert "Bob" Lamoureux and Chris Morstatt | Wikipedia, "Financial Information eXchange" (https://en.wikipedia.org/wiki/Financial_Information_eXchange); OnixS history article (https://www.onixs.biz/insights/exploring-the-history-behind-fix-protocol) |
| 3 | Originally known as the "Salomon Brothers' exchange"; first public version 1995 as FIX 2.7 | OnixS history article (same URL) |
| 4 | FIX was publicly released in 1995, deliberately open and vendor-neutral | fixtrading.org brief history |
| 5 | FIX Trading Community estimates 10,000+ institutions use FIX; billions of messages per day | fixtrading.org "Who We Are" (https://fixtrading.org/who-we-are/) |
| 6 | FIX Protocol Ltd rebranded to FIX Trading Community on 19 August 2013 | Markets Media, "FIX Rebrands Itself 'FIX Trading Community'" (https://www.marketsmedia.com/fix-renames-itself-fix-trading-community/) |
| 7 | FIX managed under a "Purpose Trust" governance | OnixS history article |
| 8 | Wire format: tag=value fields separated by SOH (ASCII 0x01); header 8/9/35 up to FIX 4.4; from FIXT 1.1/FIX 5.0 header is 8/9/35/49/56 plus optional 1128; trailer ends with 10=Checksum (3 digits, sum of ASCII values mod 256) | Wikipedia, "Financial Information eXchange" (tagvalue section, body length and checksum worked examples) |
| 9 | Message catalogue: 0=Heartbeat, 1=Test Request, 2=Resend Request, 3=Reject, 4=Sequence Reset, 5=Logout, 8=Execution Report, 9=Order Cancel Reject, A=Logon, D=New Order Single, F=Order Cancel Request, G=Order Cancel/Replace Request, R=Quote Request, S=Quote, V=Market Data Request, W=Market Data Snapshot/Full Refresh, X=Market Data Incremental Refresh | OnixS FIX 4.4 dictionary, Messages by MsgType (https://www.onixs.biz/fix-dictionary/4.4/msgs_by_msg_type.html) |
| 10 | Logon <A>: must be first message; 98=EncryptMethod (Y), 108=HeartBtInt (Y, echoed by acceptor), 141=ResetSeqNumFlag, 553/554 Username/Password | OnixS FIX 4.4, Logon <A> (https://www.onixs.biz/fix-dictionary/4.4/msgType_A_65.html) |
| 11 | Heartbeat <0> monitors the link (sent when nothing transmitted for HeartBtInt seconds); Test Request <1> forces a Heartbeat echoing 112=TestReqID; 112 required on the answering Heartbeat | OnixS FIX 4.4, Heartbeat <0> and Test Request <1> pages |
| 12 | Resend Request <2>: 7=BeginSeqNo (Y), 16=EndSeqNo (Y), EndSeqNo=0 means infinity; requesting 7..0 recommended for fastest recovery | OnixS FIX 4.4, Resend Request <2> |
| 13 | Sequence Reset <4>: 123=GapFillFlag (Y = Gap Fill mode, N/absent = Reset mode), 36=NewSeqNo (Y); GapFill header seq = beginning of skipped range; Reset mode only for disaster recovery and may lose messages | OnixS FIX 4.4, Sequence Reset <4> |
| 14 | Reject <3> for session-level violations; SessionRejectReason 373 values 0–17 and 99 (e.g. 1=Required tag missing, 5=Value is incorrect, 11=Invalid MsgType); business-level rejections go to the application (ExecutionReport/OrderCancelReject/Business Message Reject <j>) | OnixS FIX 4.4, Reject <3> |
| 15 | Logout <5> initiates/confirms termination; initiator waits for confirming Logout; 58=Text | OnixS FIX 4.4, Logout <5> |
| 16 | NewOrderSingle <D> required fields: 11=ClOrdID, Instrument block, 54=Side, 60=TransactTime, OrderQtyData (38), 40=OrdType; 44=Price required for limit; 59 absence = Day order | OnixS FIX 4.4, New Order Single <D> |
| 17 | ExecutionReport <8> structure: 11 ClOrdID, 37 OrderID (sell-side number), 17 ExecID (Y), 150 ExecType (Y), 39 OrdStatus (Y), 32 LastQty, 31 LastPx, 151 LeavesQty (Y), 14 CumQty (Y), 6 AvgPx (Y), 60 TransactTime; OrderQty = CumQty + LeavesQty | OnixS FIX 4.4, Execution Report <8> |
| 18 | OrdStatus <39> FIX 4.4: 0=New, 1=Partially filled, 2=Filled, 3=Done for day, 4=Canceled, 5=Replaced (removed), 6=Pending Cancel, 7=Stopped, 8=Rejected, 9=Suspended, A=Pending New, B=Calculated, C=Expired, D=Accepted for bidding, E=Pending Replace; FIX 4.2 identical except 5=Replaced still listed | OnixS FIX 4.4 and FIX 4.2, OrdStatus <39> pages |
| 19 | ExecType <150> FIX 4.4: 0=New, 1/2 replaced, 3=Done for day, 4=Canceled, 5=Replaced, 6=Pending Cancel, 7=Stopped, 8=Rejected, 9=Suspended, A=Pending New, B=Calculated, C=Expired, D=Restated, E=Pending Replace, F=Trade, G=Trade Correct, H=Trade Cancel, I=Order Status; values replaced in FIX 4.3 (ExecTransType tag 20 removed) | OnixS FIX 4.4, ExecType <150> |
| 20 | Side <54> values 1=Buy … G=Borrow; OrdType <40> values 1=Market, 2=Limit … P=Pegged; TimeInForce <59> values 0=Day … 7=At the Close | OnixS FIX 4.4 tag pages (54, 40, 59) |
| 21 | Order Cancel Request <F> cancels ALL remaining quantity (use G to reduce); F structure 41/11/37/Instrument/54/60/OrderQtyData | OnixS FIX 4.4, Order Cancel Request <F> |
| 22 | Order Cancel/Replace Request <G> changes parameters; must retransmit all order fields with original values except changed ones; 41=OrigClOrdID (Y), 11=ClOrdID (Y) | OnixS FIX 4.4, Order Cancel/Replace Request <G> |
| 23 | Quote <S>: 117=QuoteID (Y), 131=QuoteReqID (N), Instrument block, 132/133/134/135 Bid/Offer px/size; single product per quote | OnixS FIX 4.4, Quote <S> |
| 24 | Market Data Request <V>: 262=MDReqID (Y), 263=SubscriptionRequestType (Y), 264=MarketDepth (Y), 265=MDUpdateType, 267/269 entry-type group; Snapshot/Full Refresh <W>: 268=NoMDEntries (Y), 269=MDEntryType (Y first), 270/271 px/size; Incremental Refresh <X>: 279=MDUpdateAction, 278=MDEntryID, 280=MDEntryRefID | OnixS FIX 4.4, messages V/W/X |
| 25 | Session layer separated from application layer from FIX 5.0; FIXT 1.1 is the application-version-independent session layer used with FIX 5.0 SP2 and later | Wikipedia (FIXT section); OnixS FIX Protocol page (https://www.onixs.biz/fix-protocol.html) |
| 26 | FIX 5.0 released Q4 2006; StrategyParametersGrp tags 957–960; FIXatdl 1.0 approved by the FPL GTC on 28 March 2008 | Wikipedia, "FIXatdl" |
| 27 | FAST = FIX Adapted for Streaming; developed by the FIX Trading Community Market Data Optimization Working Group; FAST 1.0 (2006), 1.1 (2007), 1.2 (February 2009); CME patent-licence agreement with the FIX Trading Community | fixtrading.org FAST page; Wikipedia FAST protocol |
| 28 | SBE = Simple Binary Encoding, developed by the FIX High Performance Working Group; binary primitives, templates, fixed positions, not self-describing, schema out-of-band | Wikipedia, "Financial Information eXchange" (SBE section) |
| 29 | FIXP: FIX Performance Session Layer; delivery guarantees negotiated as Recoverable / Idempotent / Unsequenced; covers point-to-point and multicast | Wikipedia, "Financial Information eXchange" (FIXP section) |
| 30 | CME iLink is built on the industry-standard FIX protocol; CGW and MSGW access models; Drop Copy service | CME Group, "Develop to Globex" (https://www.cmegroup.com/solutions/market-access/globex/develop-to-globex.html) |
| 31 | MDP 3.0: low-latency direct market data for CME listed derivatives, EBS and BrokerTec; introduced SBE and event-driven messaging; since March 2017 full order-event granularity (vs legacy FAST aggregated depth); sole data feed for all CME Globex instruments | CME Group "Develop to Globex"; Databento GLBX.MDP3 dataset page (https://databento.com/datasets/GLBX.MDP3) |

### 11.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | FIX Protocol Ltd founded in 1998 | Secondary sources only (MarketsWiki "Founded as FIX Protocol Ltd in 1998"; Markets Media 2013 "Since its launch over 15 years ago"); no primary registry record verified in this pass |
| 2 | Release years of FIX 3.0, 4.0, 4.1, 4.2, 4.3, 4.4, 5.0 SP1/SP2, FIXP | Only 1995 (2.7), Q4 2006 (5.0), Feb 2009 (FAST 1.2) and Mar 2008 (FIXatdl 1.0) could be pinned to citable sources this pass; the others are ⚠-knowledge from industry literature |
| 3 | "10,000+ institutions" adoption figure | The FIX Trading Community's own estimate (fixtrading.org), not independently audited |
| 4 | MDP 3.0 delivered over UDP multicast with specific channel counts/recovery mechanics | Channel details live in CME's client wiki (atlassian) which resisted clean extraction; the multicast/recovery model is corroborated by the FAST/SCP heritage and Databento's recovery-services description but not fully re-verified here |
| 5 | Wikipedia FAST article's claim that "the approved standard is currently at version 5.0" | Conflicts with the FAST 1.x specification line (1.0/1.1/1.2 documented on fixtrading.org); treated as unreliable — flagged, not adopted |
| 6 | Reject <3> carries 45=RefSeqNum, 371=RefTagID, 372=RefMsgType | SessionRejectReason table verified; the remaining reject fields are from the standard's well-known structure and were not individually re-fetched this pass |
| 7 | FIX "Rules of Engagement" (RoE) as the universal venue-dialect mechanism | Stated by OnixS's FIX Protocol page; real RoE documents vary by venue — the generalisation is ⚠-knowledge |

### 11.3 The Rejected Claims (❌)

| # | Claim | Why rejected |
| --- | --- | --- |
| 1 | "OrdStatus 5 = Rejected" | **Wrong in both FIX 4.2 and FIX 4.4** — verified: 5=Replaced (removed in 4.4), Rejected=8. The myth survives from pre-4.3 documentation; §6.4 carries the verified tables |
| 2 | "FIX 4.4 header always includes 49 and 56" | Header 49/56 (+ optional 1128) applies from FIXT 1.1 / FIX 5.0; up to FIX 4.4 the header is 8/9/35 per the spec (verified) — though in practice most 4.4 sessions do carry 49/56, they are formally introduced with FIXT 1.1 |
| 3 | "MDP 3.0 is a FIX protocol" | It is a CME Group platform that uses FIX-family SBE encoding and FIX semantics; it is not "FIX" proper (CME Group describes it as its own Market Data Platform) |

---

## 12. What Could Not Be Verified

This section records, honestly, the facts this pass could not confirm — each is flagged ⚠ in the body of the guide where it appears:

1. **The exact incorporation date of FIX Protocol Ltd (1998).** The 1998 founding year is carried by MarketsWiki (secondary) and is consistent with Markets Media's August 2013 "over 15 years ago" remark, but no primary source (company registry, FPL/FIX Trading Community corporate history page) was retrievable in this pass. Treated as ⚠-knowledge, not fact.
2. **The release years of the intermediate FIX versions** — FIX 3.0, 4.0, 4.1, 4.2, 4.3, 4.4, 5.0 SP1, 5.0 SP2, and FIXP. Only the anchors 1995 (FIX 2.7), Q4 2006 (FIX 5.0) and 28 Mar 2008 (FIXatdl 1.0) were verified against citable pages. Two dedicated web searches for a version-year timeline returned empty results; the remaining years are marked ⚠-knowledge rather than invented.
3. **The detailed MDP 3.0 channel/recovery configuration** (channel counts, packet-recovery mechanics, session IDs). CME Group's client wiki (cmegroupclientsite.atlassian.net) is the canonical source but did not yield clean text this pass; the guide relies on CME Group's public "Develop to Globex" page and Databento's dataset documentation for the verified claims.
4. **The FAST "current version 5.0" statement on Wikipedia.** This conflicts with the documented FAST 1.x specification line on fixtrading.org and was therefore flagged rather than adopted.
5. **Reject <3> field-level details** beyond the verified SessionRejectReason table (45=RefSeqNum, 371=RefTagID, 372=RefMsgType) — well-known standard structure, but not individually re-fetched this pass.
6. **The precise share of production FIX estates running FIX 4.4 vs 5.0 SP2** — no citable industry statistic was retrieved; the guide says only that 4.4 "is the most widely deployed 4.x version", which is a qualitative industry-consensus statement, flagged ⚠-knowledge.
7. **The FIX Trading Community's "10,000+ firms" adoption figure** is the community's own estimate and could not be independently audited.
8. **The fixtrading.org "overview" page** returned only a cookie banner to the extractor, so the organisation's formal mission text quoted here comes from the "Who We Are" page instead.

---

## 13. Glossary

| Term | Meaning |
| --- | --- |
| SOH | Start of Heading — ASCII 0x01, the field delimiter of classic FIX tag=value messages |
| Tag | The numeric field identifier in a FIX message (e.g. 35=MsgType) |
| MsgType (35) | The message-type field; one char or string naming the message (A, D, 8, …) |
| CompID | Counterparty identifier (49=SenderCompID, 56=TargetCompID) |
| MsgSeqNum (34) | Per-direction, per-session monotonically increasing sequence number |
| PossDupFlag (43) | Y = this message is a retransmission (possible duplicate) |
| OrigSendingTime (122) | The original SendingTime of a retransmitted message |
| ApplVerID (1128) | Application version ID in FIXT 1.1 / FIX 5.0+ headers (decouples session from app version) |
| HeartBtInt (108) | Heartbeat interval in seconds, agreed at Logon |
| Initiate / Acceptor | The side that opens the session vs the side that accepts |
| ClOrdID (11) | The client's unique order identifier |
| OrderID (37) | The sell-side's order identifier |
| ExecID (17) | Unique identifier of an execution report |
| ExecType (150) | Purpose of an ExecutionReport (New, Trade, Canceled, Rejected, …) |
| OrdStatus (39) | Current state of the order (New, Partially Filled, Filled, Canceled, …) |
| CumQty (14) / LeavesQty (151) | Executed quantity to date / quantity still open (OrderQty = CumQty + LeavesQty) |
| IOI | Indication of Interest — the pre-trade message FIX was born to automate |
| OMS / EMS | Order Management System / Execution Management System |
| SOR | Smart Order Routing — venue-selection logic for child orders |
| ECN / ATS | Electronic Communication Network / Alternative Trading System |
| DMA | Direct Market Access — client orders reaching the market without broker intermediation |
| FIXT 1.1 | FIX Transport session protocol (FIX 5.0+) |
| FIXP | FIX Performance Session Layer (high-performance session protocol) |
| FIXML | The XML encoding of FIX messages |
| FAST | FIX Adapted for Streaming — compressed encoding for high-volume market data |
| SBE | Simple Binary Encoding — low-latency binary encoding (FIX Trading Community standard) |
| FIXatdl | FIX Algorithmic Trading Definition Language — XML for algorithm UI/parameters |
| MDP 3.0 | CME Market Data Platform 3.0 — SBE-based event-driven market data |
| iLink | CME Globex order-entry interface, built on FIX |
| Drop Copy | A copy of a firm's own execution traffic on a separate path, for risk/ops monitoring |
| RoE | Rules of Engagement — a venue's FIX dialect specification |
| Gap Fill | SequenceReset (35=4, 123=Y) covering skipped messages during resend recovery |

---

## 14. Cross-References and Further Reading

**Sibling guides in this repository (banking/):**

- [ISO 20022 Core Processes](iso_20022_core_processes_guide.md) — the post-trade message standard; FIX's counterpart across the settlement boundary (§9.3)
- [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) — OMS/EMS, order routing and exchange infrastructure (§7)
- [Capital Markets Architecture](capital_markets_architecture_guide.md) — the trading architecture and message standards (§7, §9)
- [Payment Rails](payment_rails_guide.md) — the rails and standards map (ISO 20022, ISO 8583, SWIFT) (§9.3)
- [Online Investment Trading Platforms](online_investment_trading_platforms_guide.md) — the retail/brokerage platform view
- [MAS Regulations, Guidelines and Industry Expectations](mas_regulations_guidelines_guide.md) — the Cymbal Bank persona and risk/conduct obligations (§10.7)
- [Resona Merchant Bank Asia](resona_merchant_bank_asia_guide.md) — the Cymbal Bank correspondent worked-example conventions

**Primary sources used in this pass:**

- FIX Trading Community — [The FIX Protocol Brief History](https://fixtrading.org/who-we-are/the-fix-protocol-brief-history/), [Who We Are](https://fixtrading.org/who-we-are/), [FAST](https://fixtrading.org/standards/fast/), [SBE](https://fixtrading.org/standards/sbe/), [FIXatdl](https://fixtrading.org/standards/fixatdl/)
- OnixS FIX Dictionary — [FIX 4.4](https://www.onixs.biz/fix-dictionary/4.4/index.html), [FIX 4.2](https://www.onixs.biz/fix-dictionary/4.2/index.html), [FIXT 1.1](https://www.onixs.biz/fix-dictionary/FIXT1.1/index.html), [FIX Protocol overview](https://www.onixs.biz/fix-protocol.html), [History of FIX](https://www.onixs.biz/insights/exploring-the-history-behind-fix-protocol)
- CME Group — [Develop to Globex](https://www.cmegroup.com/solutions/market-access/globex/develop-to-globex.html), [Market Data Platform](https://www.cmegroup.com/market-data/distributor/market-data-platform.html)
- Databento — [CME Globex MDP 3.0 dataset](https://databento.com/datasets/GLBX.MDP3)
- Markets Media — [FIX Rebrands Itself 'FIX Trading Community' (19 Aug 2013)](https://www.marketsmedia.com/fix-renames-itself-fix-trading-community/)
- Wikipedia — [Financial Information eXchange](https://en.wikipedia.org/wiki/Financial_Information_eXchange), [FAST protocol](https://en.wikipedia.org/wiki/FAST_protocol), [FIXatdl](https://en.wikipedia.org/wiki/FIXatdl) (secondary, history cross-check)
- MarketsWiki — [FIX Trading Community](https://www.seniorww.marketswiki.com/wiki/FIX_Trading_Community) (secondary, FPL 1998 founding)
- FIXimate — the FIX Trading Community's online FIX dictionary: [fiximate.fixtrading.org](http://fiximate.fixtrading.org/index.html)

**Closing note.** FIX succeeded because it was designed to be read by humans, debugged by engineers, extended by firms and governed by the industry — and because its session layer made electronic trading *recoverable*, not just fast. Thirty years on, the encodings have gone binary and the market-data firehoses have gone multicast, but the vocabulary is still the one Salomon Brothers and Fidelity agreed in 1992: every order, every fill, every cancel and every re-sent heartbeat is still, fundamentally, the trade conversation.
