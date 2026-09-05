# Trading-System Software Architecture: From Order to Fill — A Comprehensive Guide

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui, Solution Architect
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Technology / Capital-Markets Systems (technology/ — with the banking/capital-markets cluster)
> **Audience:** Solution architects, trading-systems engineers, market-infrastructure engineers, risk and compliance technologists
> **Last Updated:** September 2026

*A systems-architecture deep-dive on how institutional trading systems are actually built: the order lifecycle and the FIX tags that carry it (condensed from the sibling FIX guide — never re-derived), the OMS-vs-EMS functional split and the vendor landscape, the market-data path from feed handler to normalized tape (cross-referenced), the execution stack — algorithmic engines, smart order routing, venue connectivity and the colocation edge — the low-latency engineering discipline (kernel bypass, ring buffers, cache/NUMA, FPGA), the regulatory order-path components (US SEC Rule 15c3-5 market access, EU MiFID II RTS 6 algorithmic-trading controls, condensed MAS), the market-structure context (lit vs dark, matching priority, Reg NMS/Reg ATS), surveillance and reconstructability, and a full Cymbal Bank worked example tracing one order through an execution desk — from the portfolio manager's algo launch to the exchange fill and the post-trade capture, ending at the trade's path.*

**Cross-references (repository convention — siblings in `banking/` are plain filenames, siblings in `technology/` are `../technology/` or local):** [FIX Protocol: The Trade Conversation](../banking/fix_protocol_guide.md) (the protocol itself — messages, tags, sessions, §6 tag tables — **cross-ref and condense, never re-derive**) · [Market Data Consumption](../banking/market_data_consumption_guide.md) (the market-data value chain — feeds, tapes, vendors — **cross-ref, do not re-derive**) · [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) (implied orders, symbol mapping, MDP 3.0 sequencing — **cross-ref**) · [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) (the front/middle/back umbrella — **cross-ref**) · [Market Making in Singapore](../banking/market_making_singapore_guide.md) (SGX Reach, colocation, venue connectivity — **cross-ref §7, condense**) · [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) (the Singapore regulatory overlay — **cross-ref, condense**) · [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) (kernel bypass, busy-polling, hot-path discipline — **cross-ref, do not re-derive**) · [Zero Downtime System Design](zero_downtime_system_design_guide.md) (session estates) · [DDS Guide](dds_guide.md) (the market-data bus) · [Enterprise Middleware and Integration](enterprise_middleware_integration_platform_guide.md) (the integration fabric) · plus the markets-cluster context guides ([TD Securities](../banking/td_securities_software_systems_guide.md), [Citadel](../banking/citadel_llc_guide.md), [Hedge Fund](../banking/hedge_fund_guide.md), [Fircosoft](../banking/fircosoft_guide.md), [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md), [Quant LLM Agents](ai_llm/llm_agents_quantitative_finance_guide.md)).

**How to read this guide.** §1 is the overview — the one-paragraph answer, the stack diagram and the key-facts table. §2 is the order lifecycle — the order-state model and the FIX tags that carry it, condensed from the sibling FIX guide (its §6 has the worked tag-level breakdowns). §3 is the OMS vs EMS functional split and the vendor landscape. §4 is the order path — pre-trade risk, throttles and the kill switch, tied to the regulatory mandates. §5 is the market-data path — architecture-level framing only, with the deep content cross-referenced. §6 is the execution architecture — algos, SOR and liquidity-seeking logic. §7 is venue connectivity — colocation, FIX vs native binary protocols, with a verified protocol table. §8 is market structure — lit vs dark, matching priority, Reg NMS/Reg ATS framing. §9 is low-latency engineering — the microsecond stack, kernel bypass, ring buffers, cache/NUMA, FPGA, with vendor claims flagged. §10 is surveillance and compliance — T+0 monitoring and reconstructability. §11 is the Cymbal Bank worked example — a trading-platform architecture review that traces one order through an execution desk. §12 is the claims audit (✅/⚠/❌). §13 is "What Could Not Be Verified". §14 is the glossary. §15 is cross-references and the closing summary. **Integrity convention:** ✅ = verified this pass against a primary or cited source (URL given in §12); ⚠ = flagged (vendor-internal, marketing, single-source or not re-verified this pass); ❌ = disputed claims found in the wild and rejected. No dates, tag numbers, protocol names or statistics were invented.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Order Lifecycle — Order States and the FIX Tags That Carry Them](#2-the-order-lifecycle--order-states-and-the-fix-tags-that-carry-them)
3. [The OMS and the EMS — the Functional Split](#3-the-oms-and-the-ems--the-functional-split)
4. [The Order-Path Components — Pre-Trade Risk, Throttles and the Kill Switch](#4-the-order-path-components--pre-trade-risk-throttles-and-the-kill-switch)
5. [The Market-Data Path — Feed Handlers to Normalized Tapes](#5-the-market-data-path--feed-handlers-to-normalized-tapes)
6. [The Execution Architecture — Algos, SOR and Liquidity Seeking](#6-the-execution-architecture--algos-sor-and-liquidity-seeking)
7. [Venue Connectivity — Colocation, FIX and Native Protocols](#7-venue-connectivity--colocation-fix-and-native-protocols)
8. [The Market-Structure Context — Lit, Dark and the Matching Rules](#8-the-market-structure-context--lit-dark-and-the-matching-rules)
9. [Low-Latency Engineering — the Microsecond Stack](#9-low-latency-engineering--the-microsecond-stack)
10. [Surveillance and Compliance — T+0 Monitoring and Reconstructability](#10-surveillance-and-compliance--t0-monitoring-and-reconstructability)
11. [The Cymbal Bank Worked Example — an Execution-Desk Architecture Review](#11-the-cymbal-bank-worked-example--an-execution-desk-architecture-review)
12. [The Claims Audit — Verified, Flagged, Rejected](#12-the-claims-audit--verified-flagged-rejected)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and the Closing Summary](#15-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

A trading system is a **state machine wrapped in a latency budget and boxed in by risk controls**. At its core sits the order — an instruction to buy or sell a quantity of an instrument — and the entire architecture exists to move that order from a decision-maker (a portfolio manager, a quant model, a market-maker's quoting logic) to a matching engine, and to move the truth about what happened to it back to everyone who needs to know: the trader, the risk desk, the compliance function, the position keeper and the client.

The canonical institutional stack, front to back, is:

```text
  Decision layer      Portfolio manager / quant model / algo strategy / client order
        │  parent order (list, benchmark, limit instructions)
        ▼
  OMS                 Order state, allocations, compliance-at-order-entry, audit
        │  child-order stream
        ▼
  EMS / algo engine   Slicing, scheduling; SOR decides venue per child
        │  routed child order (FIX or native protocol)
        ▼
  Risk gate           Pre-trade checks (limits, price bands, throttles) — REGULATED
        │
        ▼
  Fix gateway /       Session management, sequence numbers, drop copy
  connectivity hub ──► EXCHANGE / ATS matching engine
        │                │
        ▼                ▼  market data (bid/ask/depth/trades) returns via feeds
  Execution reports  Feed handlers → ticker plant → normalized tape → OMS/algos/risk
        │
        ▼
  Post-trade          Position, P&L, allocation, clearing/settlement, surveillance capture
```

Two properties distinguish trading systems from every other class of financial software. First, **the hot path is measured in microseconds** — the difference between an acceptable fill and a missed one is the time between a market event and the order that reacts to it, so the architecture's center of gravity sits in networking, memory layout and scheduling (§9). Second, **the state machine is shared with counterparties and regulators** — an order is not a private record but a conversation conducted in FIX messages with an exchange or broker, and that conversation is legally reconstructable (§2, §10). The [FIX Protocol guide](../banking/fix_protocol_guide.md) owns the protocol detail; this guide owns the architecture those messages flow through.

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Order state machine | New → PartiallyFilled → Filled; Canceled / Rejected / Expired / Pending* terminal and transitional states; carried in FIX OrdStatus (39) and ExecType (150) | ✅ §2 |
| Order messages | NewOrderSingle (35=D), ExecutionReport (35=8), Cancel (35=F), Cancel/Replace (35=G) | ✅ §2 (sibling FIX guide §6) |
| OMS vs EMS | OMS = order state, workflow and compliance; EMS = execution, venues, algos, smart routing | ✅/⚠ §3 (industry definitions; vendor internals ⚠) |
| Market-data path | Feed handlers → ticker-plant consolidation → normalized tape → consumers; multicast for the hot path | ✅/⚠ §5 (cross-ref market-data guide) |
| SOR | Venue selection, liquidity seeking, price/queue modeling, sweeps — logic published only in outline by most firms | ⚠ §6 |
| Venue protocols | NASDAQ OUCH/ITCH · NYSE Pillar FIX · Cboe FIX 4.2 + BOE · CME iLink 3 + MDP 3.0 · Eurex T7 (ETI/EOBI/EMDI) · LSE Millennium FIX gateway · SGX Reach | ✅/⚠ §7 (table) |
| US market-access rule | SEC Rule 15c3-5, adopted 3 Nov 2010 (Release 34-63241, 75 FR 69825, 15 Nov 2010): pre-trade credit/capital thresholds, erroneous-order price/size controls, direct-and-exclusive control, annual CEO certification | ✅ §4 |
| EU algo-trading regime | MiFID II Art. 17 + Commission Delegated Regulation (EU) 2017/589 (RTS 6), adopted 19 July 2016; Art. 12 kill functionality confirmed by ESMA Q&A | ✅ §4 |
| Low-latency toolbox | Kernel bypass (DPDK, Solarflare Onload/OpenOnload), busy-polling, ring buffers (LMAX Disruptor), cache-line/NUMA discipline, FPGA offload | ✅/⚠ §9 |
| The latency stakes | LMAX reports Disruptor 3-stage mean latency ~52 ns vs ~32.7 µs queue-based, 25M+ msg/s (vendor-measured) | ⚠ §9 (vendor claims) |

### 1.3 How This Guide Relates to the Series

The trading-stack deep content is deliberately *owned by its sibling guides*; this guide is the architecture view that assembles them. The division of labour:

| Angle | Where the deep content lives | This guide's role |
| --- | --- | --- |
| FIX protocol, tags, sessions, message tables | [FIX Protocol guide](../banking/fix_protocol_guide.md) §2–§6 | §2 condenses to the state machine + the tags every data model must persist |
| Market-data value chain, tapes, vendors, feeds | [Market Data Consumption guide](../banking/market_data_consumption_guide.md) | §5 frames the path and the latency tiers; cross-ref only |
| Symbology, MDP sequencing, implied orders | [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) | §5/§7/§8 point at the mechanics |
| SGX/venue connectivity, colocation, market-making | [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §7, §10 | §7 condenses the colo decision |
| Low-latency engineering craft | [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) | §9 summarizes for architects; flags vendor claims |
| MAS regulation and persona conventions | [MAS Regulations guide](../banking/mas_regulations_guidelines_guide.md) | §4.4 condenses; §11 follows the persona conventions |
| AML/filtering, ERM governance | [Fircosoft guide](../banking/fircosoft_guide.md), [Enterprise Risk Management guide](../banking/enterprise_risk_management_guide.md) | §10 cross-refs the compliance overlays |
| Front/middle/back umbrella | [Capital Markets Architecture guide](../banking/capital_markets_architecture_guide.md) | The umbrella this guide's stack sits under |

---

## 2. The Order Lifecycle — Order States and the FIX Tags That Carry Them

> **Division of labour with the sibling:** the [FIX Protocol guide](../banking/fix_protocol_guide.md) §6 carries the full worked tag-level breakdowns of NewOrderSingle and ExecutionReport, the OrdStatus/ExecType value tables and the session mechanics (sequence numbers, gap-fill). This section condenses that content to the architecture-relevant minimum: the state model and the tags every trading-system data model must persist. **Do not duplicate the sibling — cross-reference it.**

### 2.1 The State Machine Every Trading System Implements

However a firm models it internally, an order's public state is the union of what the *venues* and *brokers* report. The canonical states (FIX 4.4 OrdStatus values in parentheses):

| State | OrdStatus (39) | Meaning |
| --- | --- | --- |
| New | `0` | Accepted by the venue/broker, resting or working |
| Partially filled | `1` | Some quantity executed; the remainder still working |
| Filled | `2` | Fully executed |
| Done for day | `3` | No further execution will occur this trading day |
| Canceled | `4` | Remaining quantity cancelled by the firm or the venue |
| Rejected | `8` | Not accepted (risk rejection, invalid field, venue rule) |
| Suspended | `9` | Working but temporarily not executable |
| Pending Cancel | `6` | Cancel requested, acknowledgement pending |
| Pending New / Pending Replace | `A` / `E` | Transitional "request in flight" states |
| Expired | `C` | Time-in-force elapsed (e.g. a day order at the close) |
| Stopped | `7` | Stopped by a market-maker protocol (equities floor heritage) |

The one invariant every order database encodes: **OrderQty (38) = CumQty (14) + LeavesQty (151)** — the original quantity equals what has executed plus what remains. A fill is a state *transition* recorded by the venue and reported by an ExecutionReport whose `ExecType (150)` names the event (`0` New, `F` Trade, `4` Canceled, `8` Rejected, `E` Pending Replace…). `OrdStatus (39)` and `ExecType (150)` usually move together on a fill report but are different tags: status is *where the order is now*, exec-type is *why this report was sent* — that distinction matters when a single report carries a status change plus a trade.

### 2.2 The Tags That Carry the Order — Verified

The tag numbers below are verified against the FIX 4.4 standard as published by the FIX Trading Community (fixtrading.org — FIXimate, the FIX 4.4 specification with 18 June 2003 errata) and mirrored field-by-field in the OnixS FIX 4.4 dictionary; the sibling FIX guide §6.4 carries the same values, verified in its own pass:

| Tag | Name | Role in the lifecycle |
| --- | --- | --- |
| 35 | MsgType | Identifies the message (D, 8, F, G, 9, j…) |
| 11 | ClOrdID | Client's unique order identifier — the primary key of the order |
| 17 | ExecID | Unique identifier of *this execution report* |
| 37 | OrderID | Venue/broker-assigned order identifier |
| 41 | OrigClOrdID | Prior ClOrdID on cancel/replace requests |
| 54 | Side | 1 = Buy, 2 = Sell (plus short-sell variants) |
| 38 | OrderQty | Total order quantity |
| 40 | OrdType | 1 = Market, 2 = Limit, 3 = Stop, 4 = Stop-limit, P = Pegged |
| 44 | Price | Limit price (required for limit orders) |
| 59 | TimeInForce | 0 = Day, 1 = GTC, 3 = IOC, 4 = FOK, 6 = GTD |
| 39 | OrdStatus | Current state (table above) |
| 150 | ExecType | Why this ExecutionReport was sent |
| 151 | LeavesQty | Quantity still open |
| 14 | CumQty | Quantity executed to date |
| 6 | AvgPx | Average price of CumQty |
| 31 / 32 | LastPx / LastQty | Price and quantity of the *last* fill |
| 60 | TransactTime | Time of the order/execution event |

### 2.3 A NewOrderSingle and Its ExecutionReport — the Shape of the Conversation

A limit buy of 1,000 shares of a Singapore-listed equity, as a FIX 4.4 session would carry it (SOH shown as `|`, checksums elided — the sibling's §2 explains header/body/trailer anatomy):

```text
8=FIX.4.4|9=…|35=D|49=CYMBAL|56=SGX|34=7|52=20260821-09:30:00.125|11=CB-ORD-20260821-0001|
55=D05.SI|54=1|38=1000|40=2|44=1.2500|59=0|60=20260821-09:30:00.125|10=…|
```

The venue accepts, then reports a partial fill of 400 at 1.2500 and later a fill of the remaining 600 at 1.2505:

```text
8=FIX.4.4|9=…|35=8|49=SGX|56=CYMBAL|34=…|52=…|11=CB-ORD-20260821-0001|37=SGX-99123|
17=SGX-EXEC-001|150=F|39=1|55=D05.SI|54=1|38=1000|32=400|31=1.2500|151=600|14=400|
6=1.2500|60=…|10=…|

8=FIX.4.4|9=…|35=8|49=SGX|56=CYMBAL|34=…|52=…|11=CB-ORD-20260821-0001|37=SGX-99123|
17=SGX-EXEC-002|150=F|39=2|55=D05.SI|54=1|38=1000|32=600|31=1.2505|151=0|14=1000|
6=1.2502|60=…|10=…|
```

Architecturally, four facts about this exchange matter more than the syntax: (1) the **ClOrdID the client generates must be unique per session** — it is the idempotency key the venue uses to detect duplicate submissions; (2) the **ExecID is the idempotency key for fills** — downstream systems must deduplicate on (ExecID) and reconcile on (ClOrdID, CumQty, LastQty/LastPx); (3) **executions can arrive out of band from the order's other reports** (e.g. a fill after a Canceled status in some venue dialects — the state machine must tolerate it); (4) **state lives in at least three places** — the venue, the broker/OMS and the client — and they converge only through this message stream, which is why sequence-number recovery (sibling FIX guide §4) is a resilience feature, not a protocol curiosity. The market-data sibling's tape and the FIX guide's session layer carry those mechanics; the trading system's job is to persist every tag above in an **order-event store** that is the single reconstructable source of truth (§10).

### 2.4 Order Lifecycle Failure Modes

| Failure | Symptom | Architectural mitigation |
| --- | --- | --- |
| Duplicate fill applied twice | Position double-counted | Dedupe on ExecID; reconcile CumQty against OrderQty |
| Lost execution report | Order shows working, actually filled | Session gap-fill recovery; drop-copy reconciliation; venue order-status queries |
| ClOrdID collision | Venue rejects or confuses orders | Per-session unique ID generation; reject on collision |
| Stale order state after venue hiccup | Risk sees wrong exposure | Reconciliation sweeps against venue state; OMS "state re-sync" |
| Cancel racing a fill | Fill arrives after Canceled | Model PendingCancel; handle fill-after-cancel per venue rules |
---

## 3. The OMS and the EMS — the Functional Split

### 3.1 The Industry Definitions

The industry-standard functional split — used consistently across buy-side and sell-side vendor literature (FlexTrade's execution-management product pages, Charles River's OMS documentation, Bloomberg's EMSX materials) and the analyst glossaries — runs as follows:

- **OMS — Order Management System.** The system of record for orders: captures the order from the portfolio manager or client, manages its lifecycle state, enforces pre-trade compliance (mandates, restricted lists, limit structures), handles allocations and feeds the post-trade pipeline. The OMS answers "what are we trying to do, in what state is it, and are we allowed to do it?" ✅-industry-standard (definitions consistent across vendor literature; see §12 for the sources this pass actually fetched).
- **EMS — Execution Management System.** The trading desk's execution cockpit and routing engine: streaming market data, algos, smart order routing, venue/ broker connectivity, FIX session management, transaction-cost and execution-quality feedback. The EMS answers "how, where and at what price do we get this done?" ✅-industry-standard (same sources).

The boundary is a *continuum, not a wall* — most vendors sell OMS-EMS suites (or single platforms with both personalities), and the industry increasingly speaks of a merged "order and execution management" platform. Architecturally the distinction still matters because the two halves have opposite data gravity: the OMS is **state-heavy and transaction-light** (a database of orders with compliance rules, updated by events), while the EMS is **event-heavy and state-light** (a low-latency pipeline that should never block on a database write).

### 3.2 What the OMS Owns

| Responsibility | Notes |
| --- | --- |
| Order capture and state | The lifecycle of §2; parent-order records, child-order linkage |
| Pre-trade compliance | Mandate/restricted-list checks at order entry; the "compliance-at-the-speed-of-trading" layer |
| Allocations and indications | Block orders → allocation to funds/accounts (post-trade or pre-trade at the EMS) |
| Position and cash feeds | From the position-keeping/IBOR layer (see the [Investment Portfolio Operations guide](../banking/investment_portfolio_operations_guide.md)) |
| Audit and reconstructability | The order-event store of §2.3; surveillance and regulatory replay (§10) |
| Workflow | Approvals, desk routing, exception handling, integration to the [middleware fabric](enterprise_middleware_integration_platform_guide.md) |

### 3.3 What the EMS Owns

| Responsibility | Notes |
| --- | --- |
| Market-data presentation | Consolidated + direct feeds to the trader's blotter/charts (§5) |
| Algo selection and parameters | Launching broker algos (VWAP, TWAP, implementation shortfall) or internal engines (§6) |
| Smart order routing | Venue selection per child order (§6) |
| Venue/broker connectivity | FIX sessions and native protocols, order-entry gateways (§7) |
| Execution-quality feedback | Fill rates, slippage vs benchmark, venue statistics back to the trader |
| DMA/Sponsored access control | The client-facing edge that the risk gate (§4) must sit in front of |

### 3.4 The Vendor Landscape — Names, and an Honest Caveat

The named systems below are the ones that appear repeatedly in institutional-trading literature as the OMS/EMS incumbents. **Vendor-internal architecture (data models, latency figures, deployment footprints) is not public** — this guide can verify that these products exist and which category they occupy from vendor/analyst pages, and nothing more; every product-internal claim is flagged ⚠ in §12–§13:

| Vendor / product | Category in industry literature | Verification status |
| --- | --- | --- |
| FlexTrade — FlexTRADER EMS | Multi-asset EMS (flagship, vendor-described as "pioneering multi-asset EMS") | ✅ product/category; ⚠ internals |
| Fidessa (now part of ION Group) | Sell-side and buy-side OMS/EMS, order routing (equities heritage) | ⚠ name-level, industry knowledge |
| ION Group (broad suite incl. Fidessa, Wall Street Systems heritage) | Cross-asset trading, treasury and market infrastructure | ⚠ name-level |
| Charles River Development (State Street) | Buy-side OMS (institutional asset managers) | ⚠ name-level |
| Bloomberg EMSX / AIM | EMS and OMS on the Bloomberg network | ⚠ name-level |
| SS&C Advent | Buy-side OMS/portfolio management | ⚠ name-level |
| Ullink (acquired by ION) | Connectivity hub / NYFIX heritage | ⚠ name-level, M&A history |
| Portware (acquired by FactSet) | Multi-asset EMS/execution | ⚠ name-level, M&A history |
| TORA | Buy-side OMS/EMS (Asia-strong) | ⚠ name-level |
| Broadridge / Imagine (Imagine Software) | Front-to-back, portfolio/risk + execution for funds | ⚠ name-level |
| Macgregor XIP (FIS heritage) | EMS/XIP execution platform | ⚠ name-level, M&A history |
| FlexTrade, Linedata, Enfusion, SS&C Eze, etc. | The long tail of the buy-side OMS/EMS market | ⚠ name-level |

**Consolidation warning:** this landscape has been through two decades of M&A (Fidessa→ION, Ullink→ION, Portware→FactSet, Imagine→Broadridge, Charles River→State Street, Macgregor→FIS…). Any ownership statement ages fast, and the ones above not re-verified against primary registries this pass are marked ⚠ in §12. A 2026 architecture review should re-check ownership before writing a vendor into a strategy.

### 3.5 Where the Split Lands in the Code

A defensible target-state architecture keeps the OMS and EMS on **different deployment planes**: the OMS on the firm's standard transactional estate (database-backed, resilient, auditable — the [core-banking systems genre](../banking/posting_engine_core_banking_guide.md) patterns apply to its state handling), and the EMS on the **latency plane** (colocated, kernel-bypass-capable, single-writer event loops — §9). The two meet at a contract, not a shared database: the OMS emits parent-order intents and consumes execution summaries; the EMS consumes intents and emits fills. That contract is where the pre-trade risk gate (§4) is bolted in, and it is the seam the worked example in §11 walks through.

---

## 4. The Order-Path Components — Pre-Trade Risk, Throttles and the Kill Switch

Every order that leaves the firm passes a **risk gate** on the way to the venue. The gate is not optional engineering taste — in the US and the EU it is statute, and in Singapore it is supervisory expectation (condensed at §4.4).

### 4.1 The US Rule — SEC Rule 15c3-5 (Market Access)

Verified against the codified rule (17 CFR § 240.15c3-5, e-CFR) and the SEC's adopting release (Release No. 34-63241, File No. S7-03-10, adopted 3 November 2010; Federal Register 75 FR 69825, 15 November 2010 — compliance date roughly July 2011, six months after the effective date):

- **Scope.** Any broker-dealer with market access to an exchange or ATS — including customers using its Market Participant Identifier (sponsored/direct market access) — must "establish, document, and maintain a system of risk management controls and supervisory procedures reasonably designed to manage the financial, regulatory, and other risks of this business activity." ✅ §240.15c3-5(b)
- **Financial controls.** The system must "systematically limit the financial exposure" by being reasonably designed to (i) **prevent orders that exceed pre-set credit or capital thresholds** — per customer and the broker-dealer, and "where appropriate, more finely-tuned by sector, security, or otherwise" — and (ii) **prevent erroneous orders** "by rejecting orders that exceed appropriate price or size parameters, on an order-by-order basis or over a short period of time, or that indicate duplicative orders." ✅ §240.15c3-5(c)(1)
- **Regulatory controls.** Prevent orders before pre-order regulatory requirements are met; block restricted persons/securities; **restrict access to trading systems to pre-approved persons and accounts**; and assure "appropriate surveillance personnel receive immediate post-trade execution reports." ✅ §240.15c3-5(c)(2)
- **Direct and exclusive control.** The financial and regulatory controls must be "under the direct and exclusive control of the broker or dealer" — the customer (e.g. a hedge fund using sponsored access) cannot be the one running the broker's risk checks; limited contractual allocation of *regulatory* controls to another registered broker-dealer is permitted. ✅ §240.15c3-5(d)
- **Review and certification.** Annual effectiveness review of market-access business activity + annual **CEO certification** of compliance. ✅ §240.15c3-5(e)

**The "kill switch" caveat.** The industry shorthand for the required ability to immediately stop a customer's or the firm's access is "kill switch" or "kill functionality". The codified text fetched for this pass verifies the pre-trade rejection controls above and the exclusivity requirement; the adopting release's discussion of *immediate restriction of access* sits in release sections only partially fetched here — the *term* is therefore handled carefully in §12 (⚠-single-source for the exact release wording; the control itself is not in doubt).

### 4.2 The EU Regime — MiFID II Article 17 and RTS 6 (Regulation (EU) 2017/589)

Verified against EUR-Lex (Commission Delegated Regulation (EU) 2017/589 of 19 July 2016, OJ publication 31 March 2017 — "regulatory technical standards specifying the organisational requirements of investment firms engaged in algorithmic trading", i.e. RTS 6 under MiFID II Art. 17) and ESMA's interpretive material:

- **Article 17 MiFID II** requires a firm engaged in algorithmic trading to have "effective systems and risk controls" — resilient systems with sufficient capacity, "appropriate trading thresholds and limits", prevention of erroneous orders and systems that cannot be used for market abuse. ✅ (ESMA's interactive single-rulebook rendering of Art. 17)
- **RTS 6** operationalizes this: governance and testing of algorithms before deployment and after material changes, **pre-trade and post-trade controls**, and — per ESMA Q&A on MiFID II market structures — **Article 12 of RTS 6 requires kill functionality**: "the ability as an emergency measure to immediately pull any or all outstanding orders from any or all trading venues." ✅ (ESMA Q&A ESMA70-872942901-38, Q&A 3.16)
- The regime covers direct electronic access (DEA) in parallel — a firm providing DEA must apply equivalent controls to its clients' order flow (the EU structural sibling of the US 15c3-5 sponsored-access story).

### 4.3 The Components in the Gate

| Component | What it does | Typical implementation |
| --- | --- | --- |
| Credit/capital limits | Reject an order that would push a desk/account past its notional or loss threshold | In-memory limit counters updated by fills; checked per child order |
| Price bands / fat-finger | Reject orders outside a reference-price collar (e.g. ±X% of NBBO or last trade) or with absurd sizes | Reference price from the consolidated tape (§5); static + dynamic bands |
| Duplicate detection | Catch retransmitted or looping orders | ClOrdID/order-hash dedupe at the gateway |
| Rate/order throttles | Cap orders-per-second per strategy/account/venue | Token-bucket or fixed-window counters (see the [Distributed Rate Limiter guide](distributed_rate_limiter_guide.md)) |
| Symbol/restricted lists | Block banned securities, accounts, jurisdictions | Reference-data-driven allow/deny sets |
| Kill switch | Emergency stop: cancel all resting orders and block new ones — per account, per desk, or firm-wide | A privileged control path that bypasses the normal queue; venue-level kill where offered |
| Post-trade monitors | Immediate fill notification to surveillance personnel (15c3-5(c)(2)(iv)) | Drop-copy feeds to the surveillance store (§10) |

The gate's architecture rule: **risk checks run on the hot path, in the same process or machine as the gateway, in front of the venue session** — never in a downstream batch. Because 15c3-5 demands the broker-dealer's *direct and exclusive control* of the financial controls, a sponsored-access customer's orders pass through the broker's gate even when they bypass the broker's order-routing stack ("naked sponsored access" — where the customer's orders go straight to the venue with only the broker's MPID — is precisely what the rule was written to kill; the SEC release's background section discusses the sponsored-access practice at length ✅).

### 4.4 The MAS Overlay — Condensed

In Singapore the same architectural obligations arrive through the Securities and Futures Act 2001 and MAS's Technology Risk Management Guidelines rather than a single market-access rule. The repo's [MAS guide](../banking/mas_regulations_guidelines_guide.md) owns the detail (SFA 2001 at §2.3; TRMG 2013/2021 at §4.2; market-conduct overlays in the notices/guidelines chapters); condensed here: a CMS-licensed broker-dealer's electronic-trading systems must be resilient and recoverable (TRMG), the market-conduct regime reaches algorithmic behavior (the sibling's market-conduct chapter and the [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §8), and the practical architecture — pre-trade checks, throttles, kill capability, surveillance capture — is the same gate this section describes. MAS does not publish a single codified "market access rule" equivalent to 15c3-5 or RTS 6; firms licensed in multiple jurisdictions typically implement the union of the three regimes (⚠-knowledge — see §13).

---

## 5. The Market-Data Path — Feed Handlers to Normalized Tapes

> **Division of labour with the sibling:** the [Market Data Consumption guide](../banking/market_data_consumption_guide.md) is the repo's market-data deep-dive — the venue→feed→vendor→platform value chain, consolidated tapes (Reg NMS, CTA/CQ/UTP, the EU tape), feed types (Level 1/2, market-by-order vs market-by-price), vendor layer and consumption by trading/PMS/risk/analytics. The [DDS guide](dds_guide.md) §10 carries the internal bus design, [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) §2–§3 the symbology and MDP-3.0 sequencing mechanics, and [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) the feed-handler coding discipline. **This section is the architecture-level framing only — cross-reference, do not re-derive.**

### 5.1 The Path and Its Three Latency Tiers

```text
  VENUE matching engine
     │  direct feeds (proprietary, binary, multicast: ITCH, MDP 3.0, EOBI…)
     ▼
  Feed handler            Tier 1 — colocated, kernel bypass, parse-in-place   (~1 µs)
     │  normalized ticks (internal symbology, canonical price/size)
     ▼
  Ticker plant / bus      Tier 2 — consolidation, fan-out (multicast / shared memory)
     │
     ├──► EMS / algo / SOR hot path      (needs depth + trades, lowest latency)
     ├──► OMS / risk / surveillance      (needs the tape, tolerance in ms)
     └──► Data warehouse / tick DB       (needs everything, batch ok)  Tier 3
```

Three architectural facts frame the whole path: (1) **the venue's own feed is the only source of the venue's truth** — a consolidated or vendor tape is downstream of it, so latency-sensitive consumers subscribe to *direct* feeds and everything else consumes consolidated data; (2) **every hop normalizes** — exchange symbology to internal symbology (the [infrastructure sibling](../banking/financial_trading_order_infrastructure.md) §2 is the deep dive on symbol resolution), venue price/quantity conventions to canonical decimals, multiple encodings to one internal tick type; (3) **the tape must be reconstructable** — multicast is lossy by design, so each feed family ships a recovery path (retransmission requests, snapshot channels — see the same sibling §3 on CME MDP 3.0 sequencing) and the tick store keeps the record (§10).

### 5.2 Symbology and the Normalized Tape

The normalized tick is the market-data system's universal interface: one internal record — `{instrument_id, side, price, qty, type (bid/ask/trade/auction…), venue_ts, received_ts, source_feed}` — regardless of whether it arrived as a Nasdaq ITCH increment, a CME MDP 3.0 SBE packet or a FIX 35=X incremental refresh. Mapping tables (venue symbol → internal ID) are reference data, not code; a new venue onboards by adding a symbology map and a feed handler, not by changing consumers. The repo's market-data sibling (§5) and the infrastructure sibling (§2) carry the operational detail — including the futures-roll and corporate-action hazards that silently break naive mappings.

### 5.3 Delivery Mechanics — Multicast, TCP, UDP

| Transport | Where it dominates | Why |
| --- | --- | --- |
| UDP multicast | Venue direct feeds inside the colo; internal fan-out to many consumers | One packet to N subscribers; the latency floor; lossy → recovery design required |
| TCP (unicast) | FIX market-data sessions; recovery channels; cross-site distribution | Reliable, ordered; session semantics |
| UDP unicast / shared memory | Point-to-point low-latency lanes | Middle ground; shm for same-host fan-out |
| REST/WebSocket/streaming APIs | Vendor platforms, non-hot-path consumers, retail | Reach and simplicity, not latency |

The multicast question is where architecture meets procurement: **you only get the venue's multicast feed meaningfully inside (or very near) the venue's data centre**, which is why the colocation decision (§7.3) and the market-data decision are the same decision. Cross-site, the firm typically runs reliable point-to-point lanes (TCP or a message bus — the [DDS guide](dds_guide.md)) so the disaster-recovery site sees the same tape, delayed by the WAN, not a different tape.

### 5.4 The Market-Data Architecture Rules of Thumb

1. **One feed handler per feed family**, pinned to its own core, parsing in place without copying (see the low-latency sibling's feed-handler section).
2. **The hot-path consumer (SOR/algo) reads the ticker plant's output, not the vendor GUI's** — charting and execution must never contend for the same queue.
3. **Timestamp discipline**: record venue timestamp *and* local receipt timestamp at the handler; every downstream latency measurement depends on both.
4. **Entitlements and audit** are part of the path, not an afterthought — the market-data sibling's §9 covers the licensing/data-governance layer.
5. **The tick store is a first-class system** (columnar/time-series), fed by a side-channel copy of the tape — the quant research and surveillance consumers both depend on it (see [LLM Agents for Quantitative Finance](ai_llm/llm_agents_quantitative_finance_guide.md) for the research-consumer angle, condensed).
---

## 6. The Execution Architecture — Algos, SOR and Liquidity Seeking

### 6.1 The Parent/Child Model

Execution architecture is a **tree of orders**. A portfolio manager (or client) hands the desk a *parent order* — "buy 2,000,000 shares of D05.SI, limit 1.26, to be done with market impact under 15 bps of arrival price". The execution system slices that parent into *child orders* — timed, sized and routed so the aggregate behaves like one patient trader instead of one clumsy one. The taxonomy:

| Layer | Example | Owned by |
| --- | --- | --- |
| Parent (list/block) order | The PM's 2M-share instruction | OMS (portfolio/accounting context) |
| Strategy/algo order | "VWAP over the day" or "implementation shortfall, urgency 40%" | EMS/algo engine |
| Child orders | 40 × 50,000-share slices released as the day unfolds | Algo engine |
| Route-level orders | One child sent to SGX, one to a dark pool, one to a broker algo | SOR |
| Venue orders | The actual resting orders at each matching engine | Venues |

Each level has its own lifecycle (§2 states), its own risk checks (§4) and its own performance attribution — the fill quality of child orders against their benchmark aggregates into the parent's transaction-cost report. The [Quant LLM Agents guide](ai_llm/llm_agents_quantitative_finance_guide.md) carries the research/agent-execution angle of this pipeline (condensed — cross-ref, do not re-derive).

### 6.2 The Algorithm Taxonomy (Execution Side)

The classic institutional execution algos — present in every vendor EMS and broker algo suite, so their *behavior* is public even where their *implementation* is not:

| Algo family | Objective | Mechanism |
| --- | --- | --- |
| VWAP / TWAP | Track the day's volume-weighted average price (or a time schedule) | Schedule child orders proportional to forecast volume (or equal time slices); mostly passive |
| Implementation shortfall (IS) | Minimize slippage vs the decision price, balancing impact vs opportunity cost | Aggressive-in-the-money slicing; urgency parameter tunes aggressiveness |
| Participation / POV | Participate at a fixed % of observed volume | Child sizing from real-time volume; adapts to flow |
| Liquidity-seeking / dark | Find hidden liquidity first, minimize market impact | Sweep dark pools/ATS then lit; opportunistic |
| Closing / auction | Trade at/near the closing auction price | Build position into the close; auction participation |
| Passive / maker (post-only) | Earn the spread, avoid adverse selection | Resting limit orders with post-only instructions |

**Honesty note:** the *slicing mathematics and parameter tuning* of commercial algos are proprietary vendor/broker IP. Public material describes the families and their objectives; the internal state machines, alpha models and risk overlays are not published (⚠ — see §13).

### 6.3 The Smart Order Router — the Logic in Outline

The SOR answers one question per child order: **which venue(s), in what order, with what instructions?** Its logic has four publicly described components (visible across exchange "SOR 101" literature, vendor EMS pages and academic/industry execution-research surveys — the combination and the tuning are firm-specific ⚠):

1. **Venue selection and ranking.** A per-instrument, per-side preference list built from historical fill rates, displayed depth, fee tiers (maker/taker), and latency — refreshed continuously. Reg NMS-style markets add the regulatory twist that protected quotations must be handled per the trade-through rule (§8), so the router's venue list is partly *mandated*, not chosen.
2. **Price/queue modeling.** For each candidate venue, estimate the probability a child order at price *p* actually fills within the required horizon: depth at *p*, expected queue position given the venue's priority rule (§8.2), order-arrival intensity, and the odds the quote moves before the child reaches the front of the queue. This is the router's "queue position + adverse selection" trade-off — the core judgment call of smart routing.
3. **Liquidity seeking.** Decide how much to hunt *hidden* liquidity: which dark pools/ATSs to probe, in what sequence, with what order types (pegged, minimum-quantity), and when to give up and go lit. Dark liquidity is discovered only by sending orders — each probe leaks information and costs queue/time, so routers meter their dark flow.
4. **Sweep logic.** When speed matters more than queue position (e.g. a marketable order), the router *sweeps*: it fires simultaneous IOC child orders at every venue showing a price at or better than the limit — the "take the whole visible book" move — subject to price-collision rules and per-venue message budgets. The sweep is the hot-path moment where routing, risk and connectivity meet: hundreds of orders in milliseconds, each through the §4 gate, each on its own venue session (§7).

An honest architecture description of SOR therefore splits cleanly: **what a router does is public and standard; how any specific firm tunes it is its trading edge and is not published.** The distinction is respected throughout this guide (§12–§13).

### 6.4 The Execution-Stack Reference Layout

```text
        Parent order (from OMS)
               │
        ┌──────▼──────┐
        │  ALGO CORE  │  single-writer event loop; consumes ticks + fills; emits children
        └──────┬──────┘
               │  child intents
        ┌──────▼──────┐
        │  SOR        │  venue ranking, queue model, sweep planner (§6.3)
        └──────┬──────┘
        ┌──────▼──────┐        ┌──────────────┐
        │  RISK GATE  │───────►│  (pre-trade)  │  §4 — statutory
        └──────┬──────┘        └──────────────┘
               │  per-venue routed orders
        ┌──────▼──────┐
        │  FIX/order  │  one session per venue/broker; seq-nos, drop copy (§7)
        │  gateways   │
        └──────┬──────┘
               ▼  venue matching engines + execution reports back up the chain
```

The performance rule at every seam: **the child-order path is an in-memory, single-threaded pipeline** — no database on the hot path, no lock across the algo→SOR→gateway hop, no blocking network read (the [Low-Latency C/C++ guide](low_latency_cpp_development_guide.md) makes this its central architectural fact, and §9 summarizes the engineering).

---

## 7. Venue Connectivity — Colocation, FIX and Native Protocols

### 7.1 The Connectivity Stack

A firm's order does not "go to the exchange" — it travels a stack with distinct failure and latency characteristics at each rung: application (EMS/SOR) → session layer (FIX session, or a native session) → transport (TCP or multicast) → network (cross-connect or extranet) → the venue's gateway → the matching engine. Two facts dominate: **the venue gateway is the last software hop you share with everyone else** (queueing there is the great equalizer), and **physical distance is a hard floor** — light in fibre is ~5 ns/metre and each kilometre of distance adds nanoseconds that no software optimization removes. Hence colocation.

### 7.2 FIX vs Native Binary — the Protocol Table

The sibling [FIX guide](../banking/fix_protocol_guide.md) §9 owns the FIX-vs-native comparison. The architecture-level truth: FIX (tag=value over TCP, or its binary/compressed variants) is the *interoperability* layer — broker connectivity, cross-asset, OMS↔EMS — while the *native protocols* are the performance layer — binary encodings, multicast market data, and session models designed for the colo. Most venues offer both; the market makers and the algo desks use the native lane, the rest of the industry uses FIX. Verified protocol facts for the major venues (name-level verification this pass; see §12 for URLs):

| Venue / platform | Order entry | Market data | Verification |
| --- | --- | --- | --- |
| Nasdaq equities | **OUCH** (order entry/replace/cancel + executions; low-latency, order-centric); FIX also offered | **ITCH** (order-by-order book feed); TotalView depth | ✅ nasdaqtrader.com — OUCH/ITCH spec pages; OUCH 5.0 spec states Nasdaq book matching is price-time priority |
| NYSE Group | **Pillar FIX Gateway** — "a single protocol for firms to transact business with one or more of the NYSE Group markets", a component of the Pillar platform | NYSE feeds on Pillar-era architecture | ✅ NYSE Pillar FIX Gateway spec PDF (nyse.com) |
| Cboe equities (BZX/BYX/EDGA/EDGX) | **FIX** — "members use a subset of the FIX 4.2 protocol for order entry and drop copies"; **BOE** — Cboe U.S. Equities Binary Order Entry | Cboe depth/top-of-book feeds | ✅ Cboe Titanium U.S. Equities FIX Specification + BOE specification (cboe.com) |
| CME Globex | **iLink 3** — SBE-encoded, FIXP session layer; for futures, options, BrokerTec, EBS | **MDP 3.0** — SBE, event-driven, multicast; the sole market-data platform for Globex (since ~2017 full order-event granularity) | ✅ CME customer wiki (iLink Functional Specification; MDP 3.0 page) + cmegroup.com |
| Eurex (Deutsche Börse T7) | **FIX** and the native **ETI** entry interface on the T7 architecture | **EOBI/EMDI** broadcast interfaces on T7 | ✅ eurex.com T7 pages; OnixS T7 SDK page (interface-name level; expansions ⚠) |
| London Stock Exchange | **FIX Trading Gateway** (FIX 5.0) on **Millennium Exchange** | Millennium-era feeds | ✅ LSE technical library (MIT202 FIX Trading Gateway doc; MIT201 guide) |
| ICE Futures | FIX-based order entry; FIX/FAST and native feeds for market data | — | ⚠ name-level only this pass (see §13) |
| SGX | **Reach** trading engine; FIX connectivity | SGX market-data feeds | ✅ in-repo (Market Making in Singapore guide §7.2 "Reach and the SGX Data Centre") — condensed, cross-ref |

Additional verified/flagged notes: Nasdaq also runs FIX for order entry (its equities FIX spec), and OUCH/ITCH v5 are the modern versions; Cboe also offers an options FIX spec (not re-fetched this pass — ⚠). The sibling [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §10.7 and §7 carry the SGX connectivity worked context (condensed here by reference), and [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) §3.2 the MDP 3.0 sequencing mechanics.

### 7.3 Colocation — Where the Race Is Actually Run

The [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §7.3 documents the colocation angle for SGX (SGX's data centre, the "Reach" engine's low-latency positioning) and the [Singapore Data Centres guide](singapore_data_centres_guide.md) the facilities side — condensed here: colocation means putting the firm's servers **in the exchange's data centre** (or a metro-adjacent one for the matching engine's campus), taking the last mile of fibre out of the latency equation and enabling multicast feed access at its source. Architecture consequences: the colo estate is a *separate, hardened, low-human-touch environment*; the latency-plane systems (§9) run there; the firm's main data centre runs everything else; and the two are joined by high-bandwidth links that carry the *recovery* copy of everything the colo does (the colo can fail over to the main site at the cost of distance-latency, never the reverse for the hot path).

### 7.4 Connectivity Failure Modes

| Failure | Symptom | Mitigation |
| --- | --- | --- |
| Venue session drop | Orders in flight, state uncertain | Session recovery (seq numbers, gap-fill — FIX sibling §4); order-state reconciliation |
| Gateway queueing | Latency spikes at the venue | Message-budget throttling (client-side), session hygiene, pre-trade throttle (§4) |
| Cross-connect failure | Total loss of one venue lane | Dual venue paths / extranet backup; failover to broker routing |
| Multicast gap | Missing ticks | Recovery channels + tick-store gap detection (infrastructure sibling §3) |
| Colo outage | Hot path down | Main-site failover at distance-latency cost; venue DR facilities |

---

## 8. The Market-Structure Context — Lit, Dark and the Matching Rules

> **Division of labour:** Reg NMS and the consolidated-tape architecture are the [Market Data Consumption guide](../banking/market_data_consumption_guide.md)'s §2 territory (verified there in depth); the FIX guide §7.3 covers ECN/ATS and matching-engine context; [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) §1 covers exchange implied-order mechanics. This section is the condensed market-structure framing an architect needs to read venue protocol docs — cross-reference, do not re-derive.

### 8.1 Lit vs Dark

- **Lit venues** — exchanges and displayed ECNs/ATSs: public order books, visible quotes, trade-through protection under Reg NMS for US NMS stocks. This is where price discovery happens and where the visible tape comes from.
- **Dark venues** — dark pools and other non-displayed ATSs (and the hidden/non-displayed order types on lit venues): no pre-trade transparency, sizes often hidden, prices derived from the lit market (midpoint peg is the canonical form). They exist to let large orders trade without moving the visible market; their prices are *parasitic on lit price discovery* by design.

### 8.2 Matching Mechanics — the Priority Rules

The canonical lit-market rule is **price-time priority**: at any price level, the order that arrived first executes first. Verified at Nasdaq's own OUCH 5.0 specification ("Non-matching orders may be added to the Nasdaq Limit Order Book… where they wait to be matched in price-time priority") ✅. Consequences for the architecture: queue position is a *property of time of arrival* — which is why latency to the matching engine matters (a child order 100 µs late joins the queue 100 µs behind) and why **queue modeling** (§6.3) needs the venue's actual priority rule. Not every venue is pure price-time: derivatives markets mix FIFO with **pro-rata** allocation (fills shared proportionally across a price level), and auctions (open/close) use price-then-time or price-priority with random/time tie-breaks depending on venue (⚠ — per-venue detail varies; check the venue's matching spec — see §13).

### 8.3 The Regulatory Frame — Condensed

- **Reg NMS (US, adopted 2005)** — the framework that makes lit-market fragmentation safe: the Order Protection (trade-through) Rule 611 forces a router to respect protected quotations at other trading centers, the access rule, the sub-penny rule, and market-data rules. Architecture impact: the SOR's venue set and sweep logic are *regulatory-mandated shapes*, not free choices (cross-ref the market-data sibling §2 for the tape/plan structure this protects).
- **Reg ATS (US)** — the registration and disclosure regime for alternative trading systems; dark pools operate as ATSs under it, with Form ATS disclosure and (post-2018 amendments) fair-access and transparency requirements. Architecture impact: ATS connectivity is a regulated relationship with its own order-type and reporting obligations (⚠ — the 2018 Reg ATS amendments' specific provisions were not re-fetched this pass).
- **EU equivalent** — MiFID II's organized-trading-facility (OTF/MTF) taxonomy and its transparency regime play the same structural role in Europe (cross-ref the MAS/EU regulation coverage in the repo's regulatory guides; not re-derived here).

### 8.4 What the Architect Takes from Market Structure

1. **Fragmentation is the default** — a modern equities order must reach many books; connectivity breadth (§7) and routing logic (§6) exist because of it.
2. **Dark is a controlled tool, not a default** — dark orders carry information leakage and queue risk; the SOR meters them.
3. **Priority rules are per-venue configuration** — the queue model must read each venue's matching spec, and those specs change.
4. **Regulation shapes code** — trade-through logic, ATS disclosures and tape rules compile down to router configuration and reporting fields, so the architecture keeps a *regulatory-config layer* separate from the routing core.
---

## 9. Low-Latency Engineering — the Microsecond Stack

> **Division of labour:** the [Low-Latency C/C++ Development guide](low_latency_cpp_development_guide.md) is the repo's full engineering deep-dive — profiling, memory, the single-threaded hot path, kernel bypass in depth, NIC tuning, FPGA. This section is the architecture-level summary an *architect* needs to evaluate designs and vendor claims — the engineering practice itself is cross-referenced, not re-derived.

### 9.1 The Latency Anatomy

Every microsecond on the order path is owned by a layer. A representative tick→order budget (ranges are indicative engineering practice — the repo's low-latency sibling carries the measured treatment):

| Segment | What it contains | Order of magnitude |
| --- | --- | --- |
| Venue → firm | Multicast feed over the colo network | sub-µs to µs |
| Feed handler | Parse, normalize, publish | ~1 µs (kernel-bypass path) |
| Internal bus → algo | Ring buffer / shared memory hop | ~0.1–1 µs |
| Algo/SOR decision | State update, child generation, venue choice | µs |
| Risk gate | Limit checks in-line | sub-µs to µs |
| Gateway → venue | Order encoding, TCP/native send, venue gateway queue | µs (queue variance is the risk) |

The architect's rule: **measure each hop with hardware timestamps and publish the histogram** — latency *jitter* (the tail) matters more than the mean, because a queue-based competitor beats you on the mean but you beat them on the 99.9th percentile when it counts.

### 9.2 The Engineering Toolkit — Verified Practice Names

| Technique | What it is | Status |
| --- | --- | --- |
| Kernel bypass | Move packet I/O out of the kernel into userspace drivers | ✅ practice-level, in-repo verified |
| — DPDK | Data Plane Development Kit — the reference userspace poll-mode-driver framework (Intel-originated, now a broad open-source project); full NIC ownership, no kernel TCP | ✅ name + role, in-repo verified |
| — Solarflare Onload / OpenOnload | Kernel bypass that keeps the socket API (send/recv intercepted by a userspace library talking to the NIC); the historic choice for FIX-over-TCP order entry | ✅ name + role, in-repo verified |
| — AF_XDP / io_uring poll mode | Middle paths: kernel-controlled, zero-copy rings to userspace | ✅ name-level, in-repo |
| Busy-polling | Threads spin on the NIC/ring instead of sleeping on interrupts; `SO_BUSY_POLL`, disable interrupt coalescing | ✅ practice, in-repo |
| Ring buffers | Pre-allocated, lock-free, single-writer data exchange between stages — the **LMAX Disruptor** pattern | ✅ §9.3 (primary sources) |
| Cache/NUMA discipline | Cache-line padding (no false sharing), core pinning, NUMA-local memory, `isolcpus`/`nohz_full`/`SCHED_FIFO` | ✅ practice, in-repo |
| Hot-path single-writer model | One thread per core, each owning its writes; no locks on the hot path | ✅ practice, in-repo |
| FPGA offload | Hardware parsing/order entry on reconfigurable logic for the sub-µs tier | ✅ practice-level in-repo; ⚠ per-vendor specifics |
| Hardware timestamping | PTP/IEEE 1588 NIC timestamps for true path measurement | ✅ practice, in-repo |

### 9.3 The Disruptor Pattern — Verified at the Primary Source

The **LMAX Disruptor** is the best-documented public example of the ring-buffer/single-writer pattern in trading infrastructure, published by the LMAX exchange team (technical paper, "LMAX Disruptor: High performance alternative to bounded queues for exchanging data between concurrent threads", at lmax-exchange.github.io/disruptor; Martin Fowler's account, "The LMAX Architecture", 12 July 2011, at martinfowler.com):

- **The problem it names:** conventional queues conflate producer, consumer and storage concerns and introduce contention; the LMAX team measured queue-based latency "in the same order of magnitude as the cost of IO operations to disk" — "dramatically slow" for multi-stage pipelines ✅.
- **The pattern:** a pre-allocated ring buffer where a **single writer** publishes sequence-numbered entries and consumers read via sequence barriers — no locks on the path; cache-line padding (and later, variable-length padding) to stop false sharing between the sequences ✅.
- **The numbers (vendor-measured — treat as claims, not independent benchmarks):** the paper reports a three-stage pipeline with mean latency of **52 ns vs 32,757 ns** for an `ArrayBlockingQueue` equivalent (Table 4), and "over 25 million messages per second and latencies lower than 50 nanoseconds" on moderate clock rates ⚠-vendor-measured.
- **The LMAX architecture (Fowler):** a retail trading platform running its business logic **single-threaded** — "6 million orders per second on a single thread" of commodity hardware (measured on a 3 GHz dual-socket quad-core Nehalem with 32 GB RAM), entirely **in-memory with event sourcing** (state replayable from the journaled input stream), surrounded by input/output Disruptors for network I/O, with nightly snapshots and multiple replicas for microsecond failover ✅ (the architecture's creators: Martin Thompson, Michael Barker, Dave Farley — per Fowler's acknowledgments).
- **Why it matters to trading architects:** the Disruptor is the *proof-by-example* that a trading hot path can be single-writer, lock-free and deterministic — the same conclusions the repo's low-latency sibling reaches independently. Most production trading stacks use simpler bespoke rings than the full Disruptor library, but the *pattern* (single writer, sequence numbers, cache-line discipline, mechanical sympathy) is now standard vocabulary. ⚠-knowledge on breadth of production adoption — see §13.

### 9.4 FPGA and the Marketing Boundary

FPGA acceleration in market-data parsing and order entry is documented industry practice (exchange and vendor whitepapers; the repo's low-latency sibling §11 surveys it). The honest architectural position: **FPGA earns its cost only at the competitive edge** — sub-microsecond parsing where a CPU path cannot hold the tail, or where the firm *is* the venue/ecosystem. For a bank's execution desk, CPU + kernel bypass meets the SLO for the overwhelming majority of use cases (in-repo conclusion, cross-referenced). **Vendor claims of "nanosecond" end-to-end numbers, "deterministic" performance and "zero-copy everything" are marketing until measured in your rack** — treat every such number as ⚠ and re-measure with hardware timestamps (this is stated plainly in §13 as well).

---

## 10. Surveillance and Compliance — T+0 Monitoring and Reconstructability

> **Division of labour:** [Fircosoft](../banking/fircosoft_guide.md) (AML/filtering — the sanctions and transaction-monitoring angle) and [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) (firm-wide risk governance) are the repo's compliance deep-dives; the [MAS guide](../banking/mas_regulations_guidelines_guide.md) owns the Singapore market-conduct overlay and the [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §8 the conduct overlay for market makers. This section condenses the *trading-technology* slice: what the platforms must capture and replay.

### 10.1 The Two Sides of the Surveillance Coin

- **T+0 (same-day) monitoring** — detecting manipulation and conduct risk *as it happens*: spoofing/layering patterns (place-and-cancel cycling around the touch), wash trading, marking the close, momentum-ignition patterns, and desk/limit breaches. The detectors run on the firm's own order and market-data streams in near-real time (the same tapes §5 and §2 feed, replayed through pattern detectors), and on the *venue's* view of the firm's flow where venues publish it (drop copy — a copy of the firm's executions on a separate session, which the FIX guide's glossary defines).
- **Reconstructability** — the regulatory expectation that any order's life can be rebuilt later: *decision → order entry → every state change → every fill → allocation → settlement*, with timestamps, identities and parameters intact. Reconstructability is an *architecture property*: it requires the §2.3 order-event store (append-only, every report persisted with venue and local timestamps), the §5 tick store (the market context the order traded into), the drop-copy stream, and retention/disclosure tooling. If the order path is a pipeline, the audit path is a **journal of everything the pipeline ever did** — event sourcing in the LMAX sense (§9.3) is also the compliance answer.

### 10.2 The Verified Regulatory Anchors

- **US — SEC Rule 15c3-5(c)(2)(iv):** the risk-control system must "assure that appropriate surveillance personnel receive **immediate post-trade execution reports** that result from market access" ✅ (e-CFR text, fetched this pass) — the drop-copy-to-surveillance lane is statute for US market access.
- **US — market-access review:** 15c3-5(e) annual review + CEO certification (verified §4.1) — the surveillance/review loop is itself regulated.
- **EU — MiFID II / RTS 6:** algorithmic-trading firms must have systems that cannot be used for market abuse (Art. 17, verified §4.2); the kill functionality (RTS 6 Art. 12, ESMA-confirmed) is the emergency stop that surveillance operators pull when a detector fires.
- **Singapore — MAS:** market-conduct obligations under the SFA and MAS notices reach manipulative trading patterns in the same family as the EU/US regimes; the repo's [MAS guide](../banking/mas_regulations_guidelines_guide.md) carries the detail (cross-ref, condensed here by reference).

### 10.3 The Surveillance Data Architecture

```text
  Order-event store (every §2 tag, both timestamps) ──┐
  Tick store (the tape the orders traded into)  ──────┼──►  T+0 detectors (pattern rules,
  Drop copy (venue's copy of firm executions)  ───────┤      ML anomaly models)  ──► alerts
  Reference data (accounts, symbols, limits)   ───────┘          │
                                                                 ▼
                                        Investigation workbench: full replay of any
                                        order + the market around it (reconstructability)
```

Design rules: (1) the capture lanes are **side-channel copies** — surveillance must never risk the hot path, so it consumes a replicated tape, not the live queues; (2) every event carries **two clocks** (venue time + local receipt) because disputes are decided on evidence, and clock skew is the first thing challenged; (3) **retention is a requirement, not a cost line** — keep the order-event and tick stores for the regulated horizon and make replay a self-service tool; (4) the **detector rule set is config** — spoofing/layering signatures change as venues and enforcement evolve, so rules deploy without code changes (the [AI/GenAI compliance guide](../banking/ai_genai_banking_compliance_guide.md) and [Financial Fraud Detection guide](../banking/financial_fraud_detection_at_scale_guide.md) cover the model-driven angles; the [Fircosoft guide](../banking/fircosoft_guide.md) the AML/filtering overlay that runs alongside trade surveillance).
---

## 11. The Cymbal Bank Worked Example — an Execution-Desk Architecture Review

> **Persona convention:** across this repository, **Cymbal Bank** is the fictional bank used for worked examples (the persona conventions are documented in the [MAS guide](../banking/mas_regulations_guidelines_guide.md); the sibling [FIX guide](../banking/fix_protocol_guide.md) §10 runs a Cymbal Bank FIX-session lifecycle, and the [Market Making in Singapore guide](../banking/market_making_singapore_guide.md) §10 a Cymbal Bank prime-brokerage worked example). This example follows the same conventions: Cymbal Bank is a mid-sized bank headquartered in Singapore with a CMS licence, running an institutional execution desk for equities across SGX and the US markets.

### 11.1 The Scenario

An internal portfolio manager at Cymbal Bank's asset-management arm wants to buy **2,000,000 shares of D05.SI** (a Singapore-listed equity) with a limit of S$1.26 and an implementation-shortfall benchmark. The order is entered into the OMS at 09:12; the desk runs it through the day via the firm's execution platform. The architecture review traces **one child order** end-to-end and audits each component's responsibility against the earlier sections of this guide.

### 11.2 The Platform Under Review

```text
  [Asset Mgmt OMS]         Charles-River-class buy-side OMS (Cymbal's OMS vendor ⚠-internal)
        │  parent order CB-PAR-88231 (2,000,000 @ 1.26 IS)
        ▼
  [Execution Platform — Cymbal's own build, colocated at the exchange data centre]
     ┌──────────────┐   ┌──────────────────────────────┐
     │  Algo core   │   │  SOR (venue models per §6.3)  │
     │  (IS algo)   │   └──────────────┬───────────────┘
     └──────┬───────┘                  │
            ▼                          ▼
     [ Pre-trade risk gate ] ← limits, bands, throttles (§4)
            ▼
     [ FIX/native gateways: SGX FIX session + broker DMA sessions for US ]
            ▼
     SGX Reach matching engine  +  US venue via executing broker
            ▼  execution reports (35=8) back up the chain to the OMS
     [ Post-trade: positions, allocation, TCA, surveillance store ]
```

### 11.3 The Order Path — Step by Step

| # | Step | Component | Responsibility | Architecture notes |
| --- | --- | --- | --- | --- |
| 1 | **Algo launch** | EMS/algo core | PM's IS order sliced: urgency profile, participation cap, venue policy | The algo receives the parent from the OMS over the order contract (§3.5); it subscribes to the ticker plant's D05.SI depth (§5) |
| 2 | **OMS order record** | OMS | Parent order CB-PAR-88231 persisted: state New, limit 1.26, benchmark arrival price | The order-event store (§2.3) writes the intent with both timestamps; compliance-at-entry (mandate check) passes |
| 3 | **Child generation** | Algo core | First child CB-CHD-001: 50,000 shares, limit 1.2550 (in-the-money cap per IS logic), IOC-ready | Child inherits parent's risk context; each child gets its own ClOrdID namespace (§2.3) |
| 4 | **Pre-trade risk** | Risk gate | Check credit headroom (desk notional), price band (±1% vs arrival ✅ within), message throttle (desk at 62% of per-second budget), restricted list | 15c3-5-style checks under the broker-dealer's *direct and exclusive control* (§4.1); MAS TRMG-aligned system-resilience checks; if any check fails → child Rejected with reason tag in the report |
| 5 | **SOR decision** | SOR | D05.SI lit book at SGX shows 40,000 offered at 1.2545; dark pool A (SGX-supervised ATS) probed first with a pegged 20,000 min-qty order | Venue ranking from fill statistics; dark probe metered (§6.3); the queue model says the lit book's 40,000 at 1.2545 is capturable with a sweep |
| 6 | **Routing + gateway** | FIX gateway | Sweep: SGX FIX session gets the marketable child; 10,000 fills immediately against the 1.2545 offer | Session sequence numbers, Outbound rate ok; drop-copy stream carries the fill to surveillance (§10) |
| 7 | **Exchange ack/fill** | SGX Reach | ExecutionReport 35=8: ExecType F (Trade), OrdStatus 1 (Partially filled), LastQty 10,000, LastPx 1.2545, LeavesQty 40,000 | The §2.3 tags arrive in one message; the OMS updates CB-CHD-001; CumQty+LeavesQty = OrderQty invariant holds |
| 8 | **Fill aggregation** | Algo core | Remaining 40,000 re-priced into the book as a resting limit at 1.2550; subsequent fills stream in through the day | Each fill triggers: risk counter update (§4), TCA tick (slippage vs arrival), OMS position feed, surveillance copy |
| 9 | **Kill-switch drill** | Risk/surveillance | 11:47 — the T+0 detector flags a pattern of rapid place/cancel cycles from one sponsored-access client account; the desk operator pulls the per-account kill switch: all resting orders for that account cancelled, new orders blocked | RTS 6 Art. 12-style kill functionality (ESMA-confirmed); the privileged control path bypasses normal queues (§4.3); the event itself is journaled |
| 10 | **Parent completion** | OMS/EMS | 16:58 — parent 96% filled, 4% left to the close auction at the algo's discretion; final TCA: 9 bps slippage vs arrival, inside the 15 bps mandate | Post-trade: allocations to funds, settlement via the post-trade stack, order-event store sealed for the day |
| 11 | **Post-trade capture** | Back office + surveillance | Positions updated (IBOR), allocations sent, TCA report to the PM, surveillance store retains the full order + tick context | Cross-ref the repo's investops/trade-lifecycle and market-data siblings for the downstream detail |

### 11.4 Component Responsibilities — the Audit View

| Component | Must guarantee | Must never do | Verified against |
| --- | --- | --- | --- |
| OMS | State truth, compliance-at-entry, audit | Block on the execution path | §3.2; FIX sibling §7.1 |
| Algo core | Child discipline, benchmark adherence | Leak the parent's full size into one child | §6.1–6.2 |
| SOR | Venue choice per §6.3 logic, Reg NMS/venue-policy compliance | Chase liquidity without queue/impact modeling | §6.3, §8 |
| Risk gate | Pre-trade checks under direct-and-exclusive control | Be bypassable by any order path | §4.1–4.3 (15c3-5 verified) |
| Gateways | Session integrity, seq-number recovery, drop copy | Lose an execution report | §2.3, §7; FIX sibling §4 |
| Surveillance | T+0 detection + reconstructable record | Touch the hot path | §10; ERM/Fircosoft siblings |
| Post-trade | Position/P&L truth, settlement readiness | Re-derive state from memory | investops sibling (cross-ref) |

### 11.5 The Review's Findings

The architecture review of Cymbal's execution desk would land on a familiar set of findings, each mapping to a section of this guide:

1. **The OMS/EMS contract is clean** (§3.5) — parent intents in, fill summaries out, risk gate bolted at the seam; but the *child-order store* lives only in the EMS, so a rebuild of the parent's day requires the EMS journal — **finding:** replicate the child-event stream to the order-event store.
2. **Risk checks are in-line but throttles are process-local** — the desk's per-second budget is enforced per gateway process; a multi-gateway sweep (§6.3) can exceed the desk budget in aggregate — **finding:** centralize the throttle state (see the [Distributed Rate Limiter guide](distributed_rate_limiter_guide.md)).
3. **The SGX lane is colocated; the US lane rides a broker DMA pipe** — US child orders pay broker-gateway queueing; the SOR's venue stats must model it — **finding:** treat broker DMA as a venue with queue characteristics, not as an infinite pipe.
4. **Timestamp discipline is good at the gateways, weak at the OMS** — venue timestamps are preserved, but the OMS's own processing timestamps are application-level; reconstructability (§10) needs the two-clock rule everywhere — **finding:** add receipt timestamps at every boundary.
5. **Kill-switch coverage is per-account, not per-strategy** — RTS 6's "any or all outstanding orders" capability (ESMA Q&A) suggests a strategy-level kill as well — **finding:** extend the privileged control path to strategy and algo-instance scopes.

The desk's architecture is sound in its bones — single-writer hot path (§9), statutory risk gate (§4), reconstructable stores (§10) — and the review's findings are the *seams*, not the core: the seams are exactly where this guide's cross-referenced siblings carry the deep fixes.
---

## 12. The Claims Audit — Verified, Flagged, Rejected

**Convention:** ✅ = verified this pass against a primary/cited source (URL given); ⚠ = flagged (vendor-internal, marketing, single-source, or not re-verified this pass); ❌ = disputed/unverified claims found in the wild and rejected.

### 12.1 The Verified Claims (✅)

| # | Claim | Source |
| --- | --- | --- |
| 1 | FIX tags 35/11/54/38/40/44/59/60/39/150/151/14/6/17/31/32/37/41 carry the order lifecycle as described in §2 (MsgType, ClOrdID, Side, OrderQty, OrdType, Price, TimeInForce, TransactTime, OrdStatus, ExecType, LeavesQty, CumQty, AvgPx, ExecID, LastPx/LastQty, OrderID, OrigClOrdID) | FIX 4.4 specification (fixtrading.org — https://fixtrading.org/standards/fix-protocol/ and FIXimate https://fiximate.fixtrading.org/), mirrored field-by-field in the OnixS FIX 4.4 dictionary (https://www.onixs.biz/fix-dictionary/4.4/index.html); same values verified in the sibling FIX guide §6.4 |
| 2 | OrdStatus (39) FIX 4.4 values: 0 New, 1 Partially filled, 2 Filled, 3 Done for day, 4 Canceled, 6 Pending Cancel, 8 Rejected, 9 Suspended, A Pending New, C Expired, E Pending Replace (5=Replaced removed in 4.4; B/D carry legacy meanings) | OnixS FIX 4.4 OrdStatus page (via sibling fix_protocol_guide.md §6.4 — verified pass) |
| 3 | ExecType (150) includes 0 New, F Trade, 4 Canceled, 8 Rejected, E Pending Replace, plus correction/cancel/status values G/H/I | Same source as #2 |
| 4 | FIX 4.4 was published in 2003: the supported-4.4 specification package on fixtrading.org carries "18 June 2003" errata, and the repo umbrella guide's 2026 re-verification pass pinned 4.4 to 2003 via the FIXML 4.4 schema guide dated 9 Jan 2004 | https://fixtrading.org/standards/fix-protocol/ (FIX 4.4 Specification with 20030618 Errata); capital_markets_architecture_guide.md §13 |
| 5 | FIX 5.0 was released 30 December 2006 — "FIX 5.0 Specifications (30 December 2006)" section of the FIX Trading Community's legacy archive, which also documents the FIXT 1.1 session/application decoupling | https://fixtrading.org/legacy-fix-specifications/ |
| 6 | FIX Trading Community still supports FIX 4.2 and 4.4; other versions archived; development continues via FIX Latest extension packs | https://fixtrading.org/standards/fix-protocol/ |
| 7 | Nasdaq's OUCH is the order-entry protocol (enter/replace/cancel + executions) for Nasdaq equities; Nasdaq's limit order book matches in price-time priority ("Non-matching orders may be added to the Nasdaq Limit Order Book… where they wait to be matched in price-time priority") | https://nasdaqtrader.com/Trader.aspx?id=OUCH; OUCH 5.0 spec PDF https://www.nasdaqtrader.com/content/technicalsupport/specifications/TradingProducts/OUCH5.0.pdf |
| 8 | ITCH is the Nasdaq order-by-order market-data feed family; ITCH/OUCH 5.0 are the current-generation versions | nasdaqtrader.com TradingSpecs + OnixS Nasdaq page (name-level); b2bits/nasdaq docs corroborate |
| 9 | CME iLink is the Globex order-entry interface; iLink 3 is SBE-encoded with the FIXP session layer, serving futures, options, BrokerTec and EBS; MDP 3.0 is CME's SBE-based event-driven market-data platform | CME customer wiki — https://cmegroupclientsite.atlassian.net/wiki/spaces/EPICSANDBOX/pages/714539039/iLink+Functional+Specification and CME MDP 3.0 page; https://www.cmegroup.com/markets/ebs/ebs-connectivity.html |
| 10 | NYSE Pillar FIX Gateway: "a single protocol for firms to transact business with one or more of the NYSE Group markets", a component of the integrated Pillar platform | NYSE Pillar Gateway FIX Protocol Specification, https://www.nyse.com/publicdocs/nyse/NYSE_Pillar_Gateway_FIX_Protocol_Specification.pdf |
| 11 | Cboe U.S. equities members "use a subset of the FIX 4.2 protocol for order entry and drop copies" on BZX/BYX/EDGA/EDGX; Cboe also publishes the BOE (Binary Order Entry) specification | https://www.cboe.com/document/tech-spec/content/technical-specifications/cboe-titanium-u.s.-equities-fix-specification; https://cdn.cboe.com/resources/membership/US-Equities-BOE-Specification.pdf |
| 12 | Eurex T7 is Deutsche Börse Group's trading architecture ("ultra-low latency"); the T7 family exposes ETI order entry and EOBI/EMDI market-data interfaces (interface-name level) | https://www.eurex.com/ex-en/support/technology/t7; https://www.onixs.biz/t7-trading-platform.html |
| 13 | LSE runs Millennium Exchange; its FIX Trading Gateway documentation is a FIX 5.0 spec (MIT202), with the trading-system guide MIT201 | https://docs.londonstockexchange.com/sites/default/files/documents/MIT202%20-%20FIX%20Trading%20Gateway%20(FIX%205%200)%20-%20Issue%2013.1.pdf; MIT201 PDF (docs.londonstockexchange.com) |
| 14 | SGX's trading engine is Reach, run out of the SGX data centre (Singapore reference point for colocation) | In-repo verified — market_making_singapore_guide.md §7.1–§7.2; market_data_consumption_guide.md §5.3 |
| 15 | SEC Rule 15c3-5 (Exchange Act) requires brokers/dealers with market access to establish, document and maintain risk management controls and supervisory procedures managing financial and regulatory risks of market access | 17 CFR § 240.15c3-5 — https://www.law.cornell.edu/cfr/text/17/240.15c3-5; adopting release https://www.sec.gov/files/rules/final/2010/34-63241.pdf |
| 16 | 15c3-5 financial controls: prevent orders exceeding pre-set credit/capital thresholds (aggregate per customer and broker-dealer, finer by sector/security) and prevent erroneous orders via price/size parameters and duplicate detection | §240.15c3-5(c)(1), same URLs as #15 |
| 17 | 15c3-5 regulatory controls include restricting access to pre-approved persons/accounts and assuring surveillance personnel receive immediate post-trade execution reports | §240.15c3-5(c)(2), same URLs |
| 18 | 15c3-5 requires the financial and regulatory controls to be under the broker-dealer's direct and exclusive control (limited allocation of regulatory controls to another registered broker-dealer by written contract) | §240.15c3-5(d), same URLs |
| 19 | 15c3-5 requires annual effectiveness review and annual CEO certification; adopted 3 November 2010 (Release 34-63241; 75 FR 69825, 15 November 2010) | §240.15c3-5(e); release PDF dated "Date: November 3, 2010" (same URLs) |
| 20 | MiFID II Art. 17 requires firms engaged in algorithmic trading to have effective systems and risk controls — resilient systems with sufficient capacity, trading thresholds and limits, erroneous-order prevention | ESMA interactive single rulebook, Art. 17 — https://www.esma.europa.eu/publications-and-data/interactive-single-rulebook/mifid-ii/article-17-algorithmic-trading |
| 21 | Commission Delegated Regulation (EU) 2017/589 of 19 July 2016 = the RTS on organisational requirements for algorithmic trading (RTS 6), OJ 31 March 2017 | EUR-Lex — https://eur-lex.europa.eu/eli/reg_del/2017/589/oj/eng |
| 22 | RTS 6 Art. 12 (kill functionality): firms must be able "as an emergency measure to immediately pull any or all outstanding orders from any or all trading venues" | ESMA MiFID II Q&A (ESMA70-872942901-38, Q&A 3.16) — https://www.esma.europa.eu/publications-data/questions-answers/1612 |
| 23 | LMAX Disruptor technical paper: conventional queues add latency "in the same order of magnitude as the cost of IO operations to disk"; the Disruptor ring buffer uses a single writer, sequence barriers and cache-line padding | https://lmax-exchange.github.io/disruptor/disruptor.html |
| 24 | Disruptor measured claims (vendor-measured, flagged in body): three-stage pipeline mean latency 52 ns vs 32,757 ns for ArrayBlockingQueue (Table 4); "over 25 million messages per second and latencies lower than 50 nanoseconds" | Same paper as #23 |
| 25 | LMAX platform architecture (Fowler): single-threaded business-logic processor handling 6M orders/sec on commodity hardware (3 GHz dual-socket quad-core Nehalem, 32 GB), in-memory with event sourcing, input/output disruptors, nightly snapshots, three processors for failover; creators Thompson, Barker and Farley | https://martinfowler.com/articles/lmax.html (12 July 2011) |
| 26 | Kernel-bypass practice names and roles — DPDK (userspace poll-mode drivers), Solarflare Onload/OpenOnload (socket-API-preserving TCP kernel bypass), AF_XDP/io_uring poll mode, busy-polling (SO_BUSY_POLL), cache-line/NUMA discipline, PTP/IEEE 1588 hardware timestamping | In-repo verified — low_latency_cpp_development_guide.md §7 (its own verified pass); names cross-checked against vendor documentation |
| 27 | FlexTrade markets FlexTRADER as its "pioneering multi-asset Execution Management System" (EMS product/category — internals not public) | https://flextrade.com/products/flextrader-execution-management-system/ |
| 28 | OrderQty (38) = CumQty (14) + LeavesQty (151) invariant and the ClOrdID/ExecID idempotency roles | FIX 4.4 standard semantics — OnixS ExecutionReport page (via sibling fix guide §6.2, verified pass) |

### 12.2 The Flagged Claims (⚠)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | Any vendor-internal architecture, data-model or latency claim about Fidessa/ION, Charles River, Bloomberg EMSX/AIM, SS&C Advent, Ullink, Portware, TORA, Broadridge/Imagine, Macgregor XIP, FlexTrade internals, etc. | Not public; vendor pages are marketing; category placement verified, internals not (§3.4) |
| 2 | M&A/ownership statements in the §3.4 vendor table (Fidessa→ION, Ullink→ION, Portware→FactSet, Imagine→Broadridge, Charles River→State Street, Macgregor→FIS) | Industry knowledge not re-verified against registries this pass; ownership changes are frequent |
| 3 | The phrase "kill switch" as the exact 15c3-5 terminology | The codified rule text fetched this pass verifies the pre-trade rejection and exclusivity controls; the adopting release's discussion of immediate restriction of access sits in sections only partially fetched — the *capability* is not in doubt, the precise release wording is single-source in this pass (§4.1) |
| 4 | ICE Futures protocol specifics (order-entry dialect, market-data feed naming) | Could not be verified at a primary source this pass (repeated empty search results); name-level only (§7.2, §13) |
| 5 | "MAS has no single market-access rule equivalent to 15c3-5/RTS 6; firms implement the union of regimes" | Reasoned synthesis from the MAS guide's instrument map, not a MAS statement — ⚠-knowledge |
| 6 | Pro-rata allocation and auction tie-break details across derivatives venues | Per-venue matching specs vary; not re-fetched this pass (§8.2) |
| 7 | The 2018 Reg ATS amendments' specific provisions (fair access, Form ATS transparency) | Not re-fetched this pass (§8.3) |
| 8 | LMAX Disruptor performance figures and breadth of production adoption of the full library | Vendor-measured numbers; "most firms use simpler bespoke rings" is industry knowledge (§9.3) |
| 9 | FPGA "nanosecond"/"deterministic" end-to-end vendor claims; exchange FPGA-offer specifics | Marketing until measured in-rack; per-vendor specifics not fetched (§9.4) |
| 10 | Eurex interface acronym expansions (ETI = "Enhanced Trading Interface", EOBI = "Enhanced Order Book Interface", EMDI = "Enhanced Market Data Interface") | Interface names verified; the full expansions are standard industry usage but were not spelled out by the fetched sources (§7.2) |

### 12.3 The Rejected Claims (❌)

| # | Claim | Why rejected |
| --- | --- | --- |
| 1 | "OrdStatus 5 = Rejected" | Wrong — FIX 4.4: 5 was Removed/Replaced-history; Rejected = 8 (verified #2 above; same myth rejected in the sibling FIX guide §11.3) |
| 2 | "The exchange/venue is the source of the consolidated tape, so one feed suffices for everything" | The venue's direct feed and the consolidated tape are different products with different latency/coverage; the market-data sibling documents the tape structure — rejected as architecture advice |
| 3 | "The EMS can replace the OMS because fills update positions" | Positions, allocations, compliance-at-entry and settlement need the OMS's state/audit role; the functional split persists across vendor literature — rejected as a simplification |

---

## 13. What Could Not Be Verified

This section records, honestly, everything this pass could not confirm. Each item is flagged ⚠ in the body where it appears:

1. **Vendor-internal specifics across the OMS/EMS landscape** — internal data models, deployment footprints, latency figures and *proprietary algorithm/routing parameters* for Fidessa (ION), ION's broader suite, Charles River, Bloomberg EMSX/AIM, SS&C Advent, Ullink, Portware, TORA, Broadridge/Imagine, Macgregor XIP, FlexTrade and the long tail. Product existence and category placement are verifiable; internal claims are not public and vendor pages are marketing. ⚠
2. **Precise M&A ownership status of the vendor names in §3.4.** The consolidation narrative (Fidessa→ION, Ullink→ION, Portware→FactSet, Imagine→Broadridge, Charles River→State Street, Macgregor→FIS) is industry knowledge carried as ⚠, not re-verified against company registries or deal announcements this pass.
3. **The exact "kill switch" wording in SEC Release 34-63241's discussion sections.** The codified 15c3-5 text (pre-trade financial/regulatory controls, direct-and-exclusive control, annual CEO certification) was verified in full; the release's adopting-release *discussion* of immediate access restriction was only partially fetched (the release PDF is ~280K characters; head and tail windows were retrieved). ⚠ — the capability's existence is not in doubt, but this pass will not quote the release's exact kill-functionality sentence.
4. **ICE Futures protocol details** — order-entry FIX dialect version and market-data feed naming. Repeated web searches returned empty this pass and the ICE technical pages were not retrievable; the §7.2 table therefore marks ICE name-level only. ⚠
5. **SGX protocol specifics beyond the Reach engine and colocation facts** (session dialects, feed names). The sibling market-making guide §7 verified Reach and the data centre; SGX's protocol documentation was not re-fetched this pass — cross-referenced rather than re-derived. ⚠
6. **Per-venue matching-priority detail** beyond the verified Nasdaq price-time statement — pro-rata allocation rules, auction tie-breaks, hidden-order priority at CME, Eurex, ICE, SGX and the options venues. The repo's infrastructure sibling §1 covers CME/Eurex/ICE implied-order mechanics; full matching-spec verification per venue was out of scope. ⚠
7. **The 2018 Reg ATS amendments' specific provisions** (volume thresholds, fair-access and disclosure requirements). Reg ATS's existence and role are well-established (cross-ref market-data sibling §2); this pass did not re-fetch the 2018 amendment text. ⚠
8. **LMAX Disruptor performance figures as independent benchmarks.** The 52 ns mean, 25M+ msg/s and 6M orders/s figures are LMAX's own published measurements (paper/Fowler), not independent audits — reported as claims with the source named. ⚠
9. **The breadth of production adoption of the full Disruptor library vs bespoke single-writer rings.** "Most production stacks use simpler bespoke rings" is industry knowledge; no adoption survey exists to cite. ⚠
10. **MAS's position on market-access risk controls as a single codified rule.** The MAS guide verifies the instrument map (SFA 2001, TRMG, notices); the synthesis that Singapore regulates the same architectural controls through TRMG + conduct rules rather than a single 15c3-5 analogue is this guide's reading, ⚠-knowledge.

---

## 14. Glossary

| Term | Meaning |
| --- | --- |
| OMS | Order Management System — system of record for orders, state, compliance and allocations |
| EMS | Execution Management System — execution cockpit: algos, SOR, venue connectivity |
| Parent order | The full order instruction from PM/client; sliced into children |
| Child order | One slice of a parent, sent to one venue/broker |
| SOR | Smart Order Router — venue-selection and order-sweeping logic |
| Algo (execution) | Automated slicing strategy (VWAP, TWAP, IS, POV…) |
| ClOrdID (11) | Client-generated unique order ID; idempotency key for order entry |
| ExecID (17) | Unique execution-report ID; idempotency key for fills |
| OrdStatus (39) | Order's current state (New, Partially filled, Filled, Canceled, Rejected…) |
| ExecType (150) | Why an ExecutionReport was sent (New, Trade, Canceled, Rejected…) |
| CumQty (14) / LeavesQty (151) | Quantity executed / quantity still open (38 = 14 + 151) |
| Drop copy | Separate copy of the firm's executions, for risk/surveillance |
| FIX session | Sequence-numbered, recoverable FIX conversation (sibling FIX guide §4) |
| Direct feed | Venue's own proprietary market-data feed (ITCH, MDP 3.0, EOBI…) |
| Consolidated tape | Regulated merged quote/trade feed across venues (market-data sibling §2) |
| Feed handler | Software that decodes a venue feed into normalized ticks |
| Ticker plant | Consolidation/fan-out layer between feeds and consumers |
| Kernel bypass | Userspace network I/O without kernel involvement (DPDK, Onload/OpenOnload) |
| Busy-polling | Spinning on a NIC/ring instead of sleeping on interrupts |
| Ring buffer | Pre-allocated lock-free buffer for single-writer/concurrent-reader exchange |
| Disruptor | LMAX's ring-buffer concurrency pattern/paper (§9.3) |
| Cache-line / NUMA | Memory-hierarchy facts that dominate hot-path design (false sharing, local memory) |
| FPGA | Field-programmable gate array — hardware reconfigurable logic for sub-µs paths |
| Colocation | Running servers inside/near the exchange data centre |
| Pre-trade risk | Statutory checks before an order leaves the firm (15c3-5, RTS 6) |
| Kill switch | Emergency capability to cancel/block orders immediately (RTS 6 Art. 12) |
| 15c3-5 | SEC market-access rule: risk controls under direct-and-exclusive broker control |
| RTS 6 | EU delegated regulation (2017/589) on algorithmic-trading organisation |
| Sponsored/DMA | Client trading on the broker's MPID (sponsored) or via broker systems (DMA) |
| Price-time priority | Matching rule: best price first, earliest arrival first at that price |
| Pro-rata | Matching rule: fills shared proportionally across a price level |
| Dark pool / ATS | Non-displayed venue; alternative trading system under Reg ATS |
| Reg NMS | US market-structure regulation (trade-through Rule 611, tapes, access) |
| T+0 monitoring | Same-day detection of manipulative/conduct patterns |
| Reconstructability | Ability to replay any order's full life + market context from stored events |

---

## 15. Cross-References and the Closing Summary

**Sibling guides in this repository** (convention: `banking/` siblings are plain filenames; `technology/` siblings are local or `../technology/`):

- [FIX Protocol: The Trade Conversation](../banking/fix_protocol_guide.md) — the protocol: messages, tags, sessions, OrdStatus/ExecType value tables (§6), the FIX-vs-native comparison (§9) — cross-referenced throughout §2 and §7
- [Market Data Consumption](../banking/market_data_consumption_guide.md) — the market-data value chain, Reg NMS/tapes, feed types, consumption — cross-referenced in §5, §8
- [Financial Trading System Infrastructure](../banking/financial_trading_order_infrastructure.md) — implied orders, symbology mapping, MDP 3.0 sequencing, price alignment — §5, §7, §8
- [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) — the front/middle/back umbrella over this stack
- [Market Making in Singapore](../banking/market_making_singapore_guide.md) — SGX Reach, colocation (§7), trading connectivity (§10.7), conduct overlay (§8)
- [MAS Regulations, Guidelines and Industry Expectations](../banking/mas_regulations_guidelines_guide.md) — SFA/TRMG/notices, the Cymbal Bank persona conventions
- [Investment Portfolio Operations](../banking/investment_portfolio_operations_guide.md) — the post-trade lifecycle, IBOR/ABOR (§11 references)
- [Fircosoft](../banking/fircosoft_guide.md) and [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) — AML/filtering and firm-wide risk governance (§10)
- [Posting Engine / Core Banking](../banking/posting_engine_core_banking_guide.md) — transactional state-handling patterns the OMS's database side draws on (§3.5)
- Markets-cluster context: [TD Securities](../banking/td_securities_software_systems_guide.md), [Citadel](../banking/citadel_llc_guide.md), [Hedge Fund](../banking/hedge_fund_guide.md)
- [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) — the engineering deep-dive behind §9
- [Enterprise Middleware and Integration](enterprise_middleware_integration_platform_guide.md), [DDS Guide](dds_guide.md), [Zero Downtime System Design](zero_downtime_system_design_guide.md) — the fabric the trading estate runs on
- [Quant LLM Agents](ai_llm/llm_agents_quantitative_finance_guide.md), [Distributed Rate Limiter](distributed_rate_limiter_guide.md), [Singapore Data Centres](singapore_data_centres_guide.md) — adjacent deep-dives referenced in §6, §4, §7

**Primary sources used in this pass:** the FIX Trading Community ([FIX Protocol](https://fixtrading.org/standards/fix-protocol/), [Legacy FIX Specifications](https://fixtrading.org/legacy-fix-specifications/), [FIXimate](https://fiximate.fixtrading.org/)) · OnixS FIX 4.4 dictionary (https://www.onixs.biz/fix-dictionary/4.4/index.html) · Nasdaq (OUCH/ITCH pages and specs, nasdaqtrader.com) · CME Group customer wiki (iLink Functional Specification, MDP 3.0 pages) and cmegroup.com · NYSE Pillar FIX Gateway spec (nyse.com) · Cboe Titanium US Equities FIX + BOE specs (cboe.com) · Eurex T7 pages (eurex.com) and OnixS T7 page · LSE technical library (MIT201/MIT202, docs.londonstockexchange.com) · e-CFR §240.15c3-5 (law.cornell.edu) and SEC Release 34-63241 (sec.gov) · EUR-Lex 2017/589 and ESMA (Art. 17 rulebook page; Q&A ESMA70-872942901-38) · LMAX Disruptor paper (lmax-exchange.github.io/disruptor) and Martin Fowler's "The LMAX Architecture" (martinfowler.com) · FlexTrade EMS product page (flextrade.com). In-repo verified facts are credited to their sibling guides, which name their own primary sources.

**Closing note.** Every trading system, whatever its veneer of vendor dashboards and protocol dialects, is the same machine: a state machine for orders, a pipe for market data, a gate for risk, and a journal for the truth of what happened. The order lifecycle of §2 is the vocabulary; the OMS and EMS of §3 are the grammar; the market-data path, the routers, the gateways and the colocated racks of §5–§9 are the performance; the regulators of §4 and the surveillance of §10 are the guardrails; and the Cymbal worked example of §11 is the sentence they all compose. Architect it as those five parts, measure every hop, journal every event, and respect the venues' priority rules — and the system will do what the industry has spent forty years learning to do: take an intention, turn it into resting orders and fills across a fragmented, regulated, microsecond-competitive world, and bring the trade's path.

