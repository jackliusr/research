# Exchange, Clearing and Settlement Technology - The Operator's View

*The venue, the clearing house and the depository are not places where trades happen; they are operators of promises. A matching engine is a rulebook with a clock attached; a central counterparty is a legal device that converts many bilateral obligations into two claims on one balance sheet; a depository is an event-processing engine driven by legal dates. This guide describes those three institutions from the inside - their functions, their risk mechanics, the shape their technology takes because of those mechanics, and the obligations regulation places on them. The argument it tests throughout is that almost nothing of substance in this subject is about trading: novation, netting, initial and variation margin, the default waterfall, the delivery-versus-payment link and the settlement cycle are all loss-allocation mechanisms, and the technology exists to make those allocations deterministic, timely and provable. Where the evidence complicates that claim, the text says so rather than smoothing it over.*

Jack Liu Shurui, Solution Architect

### How this guide relates to the series

Three sibling guides split this territory, and this one deliberately does not re-tread any of them. The boundary is stated by name because the overlap is where a reader would otherwise be sold the same material twice.

| View | What it owns | Where it lives |
|---|---|---|
| **The bank view** | Front, middle and back office inside a bank; market data consumption; the bank's own view of infrastructure (its section 6), protocols and regulation; how the bank's post-trade operations work | [Capital Markets Architecture Guide](capital_markets_architecture_guide.md) |
| **The participant view** | The order lifecycle and the FIX tags that carry it; OMS and EMS; pre-trade risk and throttles; the market-data path; execution algorithms and smart order routing; venue connectivity and colocation from the client side; lit versus dark; the microsecond stack | [Trading System Software Architecture Guide](../technology/trading_system_software_architecture_guide.md) |
| **The operator view (this guide)** | The venue as an institution and its matching engine; the CCP as a risk mutualiser; the CSD as a depository and event processor; the rulebook, the waterfall, the settlement cycle and the regulatory obligations that bind all three | This file |

A fourth sibling covers a narrower subject and is referenced only where the two touch: [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md) handles the plumbing peculiarities of derivatives order flow (implied orders, symbol mapping, sequence continuity, price precision).

The test applied when deciding whether a passage belongs here: if the sentence describes something a bank or a broker does, it is cut or turned into a cross-reference. If it describes something a venue, a clearing house or a depository does, or something that is done to them by a rulebook or an authority, it belongs here.

**Verification convention.** ✅ = verified in this research pass against a primary or stable public source, with the source and date named in the text or in section 14. ⚠ = flagged: asserted from professional knowledge, or single-sourced, or not re-verified this pass. ❌ = checked and found wrong, or checked and found unverifiable. The full audit is section 14; the honest list of what could not be reached is section 15.

### Table of Contents

1. [The Overview and the Decoder Table](#1-the-overview-and-the-decoder-table)
2. [The Chain: Obligation, Novation, Net Position, Margin, DVP](#2-the-chain-obligation-novation-net-position-margin-dvp)
3. [The Venue: Order Book, Auctions and Matching Determinism](#3-the-venue-order-book-auctions-and-matching-determinism)
4. [The CCP: Novation, Netting, Margin and the Default Waterfall](#4-the-ccp-novation-netting-margin-and-the-default-waterfall)
5. [The CSD: Book Entry, DVP and the Corporate-Action Clock](#5-the-csd-book-entry-dvp-and-the-corporate-action-clock)
6. [The Settlement Cycle and T+1](#6-the-settlement-cycle-and-t1)
7. [Venue Technology and Latency: The Operator's Problem](#7-venue-technology-and-latency-the-operators-problem)
8. [The Regulatory Frame](#8-the-regulatory-frame)
9. [The Systemic-Risk Lens](#9-the-systemic-risk-lens)
10. [Failure Modes](#10-failure-modes)
11. [Worked Example: Cymbal Bank as Clearing Member](#11-worked-example-cymbal-bank-as-clearing-member)
12. [Anti-Patterns in Market Infrastructure](#12-anti-patterns-in-market-infrastructure)
13. [State of Practice](#13-state-of-practice)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, Glossary and Cross-References](#15-what-could-not-be-verified-glossary-and-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview and the Decoder Table

### 1.1 The claim this guide tests

A market is not a place where trades happen. It is a set of promises about what happens when one side fails. Everything durable in exchange, clearing and settlement technology is an implementation of one of those promises: novation says the clearing house will stand in for the defaulter; netting says many obligations collapse into one; margin says the loss will be pre-funded rather than discovered; the default waterfall says who pays in what order once the pre-funded money runs out; the delivery-versus-payment link says the seller will not part with the security without simultaneously receiving the cash. The trading is the visible part and the least interesting part. The promises are the product.

That framing is not a rhetorical flourish. It has consequences an architect can act on. It means the system of record for a CCP is not its trade database but its collateral and loss-allocation engines. It means the correctness requirement for a matching engine is not throughput but reproducibility. It means the most important clock in a depository is not the timestamp on a message but the legal date on which an entitlement is determined.

Where the framing strains, this guide says so. Section 3 concedes that a venue is also a commercial business selling access, and that some of its technology exists for competitive rather than prudential reasons. Section 9 concedes that the loss-allocation account of a CCP is incomplete: a CCP that is too interconnected to fail is ultimately backed by the state, and that backstop is not printed in the waterfall. Section 13 sets out which parts of this subject are genuinely settled and which are contested.

### 1.2 The decoder table

The vocabulary of this subject is used loosely in the industry, and the looseness causes real design errors - most commonly, treating "clearing" as a synonym for "settlement", which leads teams to build one system where two legal mechanisms are needed.

| Term | What it actually is | Who operates it | What it reallocates |
|---|---|---|---|
| **Venue** | A place, physical or electronic, where orders from participants are brought together under a published rulebook | An exchange or an alternative trading venue | Price risk, from the participant to the participant |
| **Exchange** | A venue that is also a listing authority and a rule-making body, and usually a for-profit company | The exchange operator | Price risk, plus the regulatory obligations of listing and surveillance |
| **Matching engine** | The deterministic program that applies the priority rules to an ordered stream of order commands and emits executions | The venue | Nothing by itself; it is the mechanism that makes the venue's rulebook executable |
| **CCP** (central counterparty) | A legal entity that interposes itself between the two sides of every trade it clears | The clearing house, authorised under EMIR in the EU or its local equivalent | Counterparty credit risk, from each participant to the CCP and, through the waterfall, to the mutualised member pool |
| **Novation** | The legal act by which the CCP becomes buyer to every seller and seller to every buyer | Performed under the CCP's rulebook | The counterparty link: one bilateral contract becomes two claims on the CCP |
| **Clearing** | The set of processes between execution and settlement: trade capture, novation, netting, margining, and the management of the resulting exposures | The CCP, or a clearing member acting for its client | Credit and liquidity risk, and the timing of when they are crystallised |
| **Settlement** | The final transfer of securities against final transfer of funds, discharging the obligation | The CSD and its settlement banks | Principal risk, if and only if the delivery and payment are truly simultaneous |
| **Custody** | Holding assets for another party and administering the associated entitlements and records | A custodian bank, or the CSD for its participants | Operational and legal risk of holding, not price or credit risk |
| **CSD** (central securities depository) | The institution that holds the definitive record of who owns which securities, and operates the settlement system for them | The depository | The risk that the record of ownership is wrong or the transfer is not final |

The distinction that matters most in practice is the one between clearing and settlement. Clearing produces a net obligation and a set of margined exposures against a CCP; settlement discharges the obligation by moving securities and cash. They are operated by different institutions, under different rulebooks, on different clocks, and they fail in different ways. A system that conflates them will get the failure handling wrong in both.

### 1.3 What this guide does not cover

Stated plainly, because the sibling guides own these and duplication is worse than a gap:

- Order-entry semantics, FIX sessions and the tag-level order lifecycle: [Trading System Software Architecture Guide](../technology/trading_system_software_architecture_guide.md) sections 2 and 7.
- OMS and EMS behaviour, pre-trade risk checks, throttles and kill switches from the participant's side: the same guide, sections 3 and 4.
- Execution algorithms, smart order routing and the participant's view of lit versus dark: the same guide, sections 6 and 8.
- Market-data normalisation and feed handling inside a bank: the same guide section 5, and the bank view's market-data section.
- How a bank organises its own post-trade operations, collateral desk and regulatory reporting: [Capital Markets Architecture Guide](capital_markets_architecture_guide.md), sections 3, 4 and 6.
- Implied-order generation, symbol mapping and sequence continuity in derivatives order plumbing: [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md).

What remains, and what the rest of this guide is about, is the operator's side of the hand-off: what the venue guarantees, what the CCP guarantees, what the CSD guarantees, and what happens to those guarantees when something breaks.

### 1.4 Reading path

An architect new to the subject should read section 2 first, because it is the map: it shows what each step in the chain transfers, what it guarantees, who carries risk in the interval between steps, and how the step fails. Sections 3 to 6 then walk the institutions in chain order. Sections 7 and 8 cover the technology shape and the legal frame. Sections 9 to 12 are the analytical layer - systemic risk, failure modes, a worked example and anti-patterns. Sections 13 to 16 are the evidence layer and the summary.

---

## 2. The Chain: Obligation, Novation, Net Position, Margin, DVP

### 2.1 Why a chain and not a pipeline

A pipeline implies that value flows and each stage adds something. A chain is more accurate: each link is a legal obligation, and each link is only as strong as its weakest neighbour. The obligation does not become "more settled" as it moves; it becomes a different obligation, owed by a different party, with a different failure mode. This is the single most useful mental correction for anyone arriving from a payments or order-routing background, where the mental model really is a pipeline.

The chain has six links and each one has a legal moment - a point in time at which the obligation changes character. Those moments, not the message flows, are what the systems must be built around.

### 2.2 The chain table

| Step | What is transferred | What it guarantees | Who bears risk in the interval | What happens if the step fails |
|---|---|---|---|---|
| **1. Execution** | A bilateral obligation: A owes cash, B owes securities, on a stated date | Nothing beyond the contract between A and B. The venue guarantees only that the match followed its rules | Both parties bear full counterparty risk on each other, unsecured | The trade is cancelled or busted under the venue's error-trade rules; the venue's rulebook decides who is bound |
| **2. Trade capture and affirmation** | Agreement on the economic terms and on the settlement details (account, place, standing settlement instructions) | That both sides hold the same economics before anything is netted | Both parties; a mis-affirmed trade is a live dispute with a settlement date attached | The trade fails to be affirmed, misses the clearing window, and becomes a fail before it was ever a settlement obligation |
| **3. Novation** | The bilateral obligation is replaced by two obligations, each owed by the CCP | That the counterparty is now the CCP, with its rulebook, its margin and its waterfall behind it | The CCP, from the moment of novation, for the whole exposure | If novation does not happen - rejection, ineligibility, a failed give-up - the parties are back to bilateral exposure, usually without having priced it |
| **4. Multilateral netting** | Many gross obligations become one net position per instrument, per account, per settlement date | That the participant's obligation is the net, not the gross | The CCP; the netting set is the CCP's exposure measure and the participant's funding requirement | A netting failure is a legal failure: if the netting set is not enforceable in insolvency, the CCP's exposure is gross and its margin was sized on a fiction |
| **5. Margining** | Cash or collateral posted against the net position, plus a daily settlement of the mark-to-market move | That the loss is pre-funded up to a modelled horizon, and that the day's price move is discharged rather than accumulated | The clearing member, whose liquidity is consumed; the CCP, for anything beyond the margin | A margin shortfall is a liquidity event first and a credit event second; the CCP calls, the member funds, or the member is in default |
| **6. Settlement (DVP)** | Final transfer of securities against final transfer of funds at the CSD | That ownership and cash change hands simultaneously, or that neither does | Both parties, but only for the duration of the settlement processing - if the DVP link is real, principal risk is eliminated, not merely reduced | A fail: the seller keeps the securities, the buyer keeps the cash, and the consequence is a buy-in, a cash penalty, or a close-out |
| **7. Custody and record** | The ongoing record of ownership, plus the entitlements that flow from it | That the holder's position is legally recognised and administratively serviced | The account holder, against the operational and legal risk of the intermediary chain | A record error is not self-correcting; it becomes a claim, and sometimes litigation, years later |

### 2.3 The four transitions that carry the argument

Read down the "what it guarantees" column and the thesis becomes visible as a sequence of substitutions rather than a flow of value:

- **Obligation to novation.** Before novation, the promise is the counterparty's. After it, the promise is the CCP's. The economic exposure has not changed size; it has changed debtor. Everything the CCP does afterwards - netting, margining, the waterfall - exists to make that new debtor credible.
- **Novation to net position.** Netting is only possible because every obligation now points at the same legal entity. This is the clearest illustration of the thesis: multilateral netting is not an arithmetic convenience, it is a legal consequence of novation. Where netting is not enforceable, the arithmetic is worthless.
- **Net position to margin.** The net position is the exposure; margin is the price of carrying it. Initial margin prices the possibility that the position cannot be closed at today's price; variation margin prices today's price. The two are different instruments with different purposes, and section 4.4 treats the distinction as the core of the subject.
- **Margin to DVP.** Margin covers the CCP's exposure to its member. DVP covers the member's exposure to its settlement counterparty. They are not substitutes, and a firm that has excellent margin discipline and a weak DVP link is unhedged against the risk it thinks it has removed.

### 2.4 The clocks

Each link has its own clock, and the clocks do not align by default. Execution is timestamped to the microsecond by the venue's sequencer. Novation is timestamped by the CCP's end-of-day or intraday clearing cycle. Netting is timestamped to a settlement date, which is a calendar date in a specific market's business-day convention. Settlement finality is timestamped by the CSD's processing cycle and by the moment the transfer becomes irrevocable under the applicable settlement finality law. Custody entitlements are timestamped to a record date, which is a date, not a time.

Most operational failures in this subject are clock failures: a trade that executed before the cut-off but affirmed after it; a corporate action whose record date fell in the gap between the trade date and the settlement date; a funding leg that arrives after the CSD's currency cut-off but before the CCP's margin call, or the reverse. The rest of this guide is, in a sense, an account of those clocks and of the mechanisms built to survive their misalignment.

### 2.5 A trace through the chain

One trade, walked through the links, to show where the state and the legal moment sit at each one. The trace is deliberately ordinary - an institutional equity purchase - because the ordinary case is where the design has to work.

1. **Execution.** The order is sequenced, matched, and an execution report is emitted with the venue's sequence number. Legal effect: a contract between buyer and seller on the venue's terms, with a settlement date implied by the market's cycle. The record that must exist: the execution, immutable, with the venue's sequence number and both parties' identities.
2. **Capture and affirmation.** The buying member allocates the fill to underlying accounts, enriches it with standing settlement instructions, and the institutional client affirms. Legal effect: agreement on the settlement details, not on the economics, which were fixed at execution. The record that must exist: the affirmed instruction, with the affirmation timestamp, because the timestamp determines whether the trade is in the cycle at all.
3. **Submission to clearing.** The trade is presented to the CCP for the relevant clearing cycle. Legal effect: novation, at the moment the rulebook specifies - which may be acceptance into the cycle, or the end-of-cycle cut-off, and the difference matters if a member defaults in between. The record that must exist: the CCP's acknowledgement, and the identity of the clearing member and the account (house or client) that now faces the CCP.
4. **Netting.** The gross trade joins the member's other obligations for the same instrument, account and settlement date, and collapses into a net position. Legal effect: the member's obligation is now the net, enforceable as such. The record that must exist: the netting set definition, because it determines what the CCP can net in the member's insolvency.
5. **Margin.** Initial margin is computed on the net position and the portfolio it belongs to; variation margin is computed from the change in mark since the previous call. Legal effect: a collateral obligation and a payment obligation, each with a deadline and a remedy for failure. The record that must exist: the valuation, the model version, the collateral schedule and the call notice - the evidence that the requirement was computed according to the published methodology.
6. **Settlement.** The member instructs the CSD before the cut-off; the instruction is matched against the counterparty's; the DVP transaction is processed in a cycle. Legal effect: final transfer of securities against final transfer of funds, at the moment the settlement finality regime defines. The record that must exist: the matched instruction, the cycle in which it settled, and the finality timestamp.
7. **Custody.** The book entry is updated at the participant, the entitlement schedule is refreshed for the next record date, and the position becomes available as collateral or for onward delivery. Legal effect: a change in the beneficial owner's entitlement, mediated by the nominee chain. The record that must exist: the position statement, and the reconciliation between the participant's record and the CSD's.

The point of the trace is that steps 3, 4 and 5 happen at the CCP, step 6 at the CSD, and steps 1, 2 and 7 at the members and their custodians. Three operators, three rulebooks, three clocks, and one economic event. Nobody owns the whole trace, which is why the integration contracts - the messages, the identifiers and the timestamps - are the load-bearing part of the architecture, and why an identifier that does not survive from step 1 to step 6 destroys the ability to reconcile at all.

---

## 3. The Venue: Order Book, Auctions and Matching Determinism

### 3.1 What a venue actually sells

A venue sells three things: a rulebook that participants can rely on, a matching service that applies the rulebook without discretion, and access to the other participants. The rulebook is the product. The matching engine is the machine that makes the rulebook executable at speed, and the market data feed is the evidence trail that lets everyone verify that it was applied consistently.

This is why venue technology arguments that focus only on speed miss the point. Speed matters commercially - a slower venue attracts less flow - but the property participants actually depend on is that the rule is applied the same way to everyone, every time, and that the application can be proved afterwards.

### 3.2 Priority rules: price-time versus pro-rata

| Rule | Mechanism | Where it is used | What it rewards | What it distorts |
|---|---|---|---|---|
| **Price-time priority** | Best price first; at the same price, the earliest order is filled first | Most equity order books and many derivatives books | Being early, and being fast enough to be early | Incentivises speed investment, and produces a queue-position economy in the feed and in colocation |
| **Pro-rata** | At the same price, incoming quantity is allocated across resting orders in proportion to their size | Common in futures and options markets, particularly for larger clip sizes | Being large | Rewards size over immediacy; a small resting order can be starved at a busy price level |
| **Hybrid / tiered** | Priority by category (e.g. designated market makers first), then price-time or pro-rata within category | Venues with liquidity-provider programmes | Quoting obligations | Creates a two-class book, and makes the fairness argument depend on the obligations being met |

The architecturally important point is that the priority rule is the fairness property, not the queue position. Queue position is an output of the rule applied to a sequence of events. Two participants who send identical orders at "the same time" get different queue positions, and that is not unfairness - it is the sequencer resolving an ambiguity that the network created. Section 7 returns to this, because it is where venue technology and market-integrity arguments collide.

### 3.3 The opening and closing auctions

An auction is a separate matching algorithm run at a fixed time. Instead of continuous matching, orders accumulate, the venue publishes an indicative clearing price and often an indicative imbalance, and at the uncrossing moment all crossing orders execute at a single price chosen to maximise executed volume (subject to the venue's tie-breaking rules).

The closing auction matters far more than its share of the trading day suggests, and the reason is not convenience. The closing price is a reference price for a large set of downstream obligations:

- **Fund valuation.** Collective investment schemes strike a net asset value from closing prices; a bad close misprices every fund that holds the security.
- **Collateral valuation.** CCPs and CSDs value collateral at close; the closing price directly sets the size of the next morning's margin requirement.
- **Index events.** Index providers use closing prices for rebalancing and for corporate-action adjustments, which turns the close into an execution benchmark for every index-tracking portfolio.
- **Contract settlement.** Derivatives that settle against an official closing price, and exchange-for-physical and basis transactions that reference the close, settle on that number.
- **Benchmarks.** Volume-weighted and closing-price benchmarks used to measure execution quality are computed from the auction.

So the auction is a risk-management device: it concentrates liquidity at a known moment so that a single, reproducible, defensible price exists. That is also why auction failures are so damaging. A closing auction that does not uncross, or that uncrosses at a price later shown to be wrong, does not merely inconvenience traders - it revalues collateral, mis-strikes funds, and triggers index and contract settlements across the market. Venue operators therefore treat the auction as a controlled process with a published schedule, a defined uncrossing algorithm, price collars, and a documented fallback (typically a delayed or extended auction, never a cancellation without a rule).

### 3.4 Circuit breakers and price bands

Two distinct mechanisms are often conflated:

- **Circuit breakers** halt or pause trading, either market-wide (an index-level move triggers a halt) or at the instrument level (a rapid price move triggers a short pause followed by an auction). The purpose is to interrupt an order-flow dynamic that is feeding on itself, and to give participants time to reassess.
- **Price bands / limit-up and limit-down** constrain the prices at which orders may rest or trade, either statically (relative to a reference price) or dynamically (relative to the last trade). Orders outside the band are rejected or held in a limit state; the instrument may enter a brief halt and reopen by auction.

The mechanism detail that matters for a system designer is the state machine. A limit state is a distinct instrument state with its own rules about which order types are accepted, whether the band moves, and how the instrument leaves the state. Venues publish these as separate state diagrams per instrument class. Getting the state machine wrong - for example, allowing an aggressive order to trade through a band during a reopening auction - is a market-integrity failure, not a bug. ⚠ The specific band and halt parameters are venue-specific and are not asserted here.

### 3.5 The operational properties a venue must have

Four properties, in order of importance to the operator:

**Determinism.** Given the same ordered input sequence and the same starting state, the matching engine must produce exactly the same output sequence. This is not a performance property; it is the property that makes the venue auditable, arbitrable and recoverable. It implies a set of engineering prohibitions: no wall-clock time inside matching logic (time comes from the sequenced message), no iteration over unordered collections where order affects allocation, no floating-point arithmetic in price and quantity calculations, no dependence on external services mid-match, and no non-idempotent retry paths.

**Fairness.** Every participant's order is subject to the same rule, applied in the order the sequencer established. Fairness is a construction, not a policy: it comes from a single ordered input stream, symmetric network access, published priority rules, and no privileged path into the book.

**Recoverability.** The engine must be able to restart into a state that all participants agree is correct. In practice this means the engine is a replicated state machine: an input log, periodic snapshots, and a replay path whose result is bit-identical to the original run. The recovered state must match what participants were told, which is a stronger requirement than "the database came back".

**Transparency of state.** Participants must be able to reconstruct the book and their own orders from the data the venue publishes. That is what the market data feed is for: it is not a convenience product, it is the venue's public evidence that its matching was correct.

### 3.6 The matching engine as a replicated state machine

Stated precisely, because the phrase is often used loosely: the engine's state is a function of the ordered command log. `state(t) = apply(state(0), command[1..n])`. Everything else - the gateway, the risk checks, the publisher, the drop copy - is either an input to that log or an output derived from it.

This has three consequences that shape the whole of section 7:

1. **The sequence number is the only authoritative timestamp.** Gateway receive times, NIC hardware timestamps and exchange trade times all exist, but the order that matters is the sequencer's. Any system that reconstructs trading from wall-clock timestamps will eventually disagree with the venue about what happened first.
2. **Recovery is replay.** A standby that has consumed the same log is, by construction, in the same state. A standby that has consumed a different stream - for example, one that dropped heartbeats or reordered snapshots - is not a standby, it is a second opinion.
3. **Determinism is a testable property.** You can run the log twice and diff the output. A venue that cannot do this does not know whether its recovery is correct; it only knows that it has not yet been caught.

---

## 4. The CCP: Novation, Netting, Margin and the Default Waterfall

### 4.1 Novation: what it actually means

Novation is a legal act, not a data transformation. In its classic form, the original contract between A and B is extinguished at the moment of clearing and replaced by two new contracts: A to the CCP, and the CCP to B. The CCP is now seller to every buyer and buyer to every seller. A's exposure to B has not been reduced, resized or repriced; it has been transferred to a different debtor, whose creditworthiness is defined by a rulebook, a margin methodology and a default fund rather than by B's balance sheet.

The legal technique varies by jurisdiction and by CCP. Some CCPs use classic novation; others use an open-offer or registered-contract construction in which the rulebook constitutes a standing offer that each participant accepts when its trade is matched, with the CCP becoming counterparty at that moment. ⚠ The technique is jurisdiction-specific and was not verified against a rulebook this pass. The economic and operational effect is identical, and the effect is what matters:

- **The counterparty link is severed.** After novation, A has no claim on B for that trade, and B has no claim on A. This is the point of the mechanism - it is what stops one member's failure from propagating bilaterally through the membership.
- **Two claims replace one.** A's claim is on the CCP, and the CCP's claim is on B. The CCP now carries the risk that either side fails, and the whole risk apparatus exists to make carrying it credible.
- **The CCP's promise is only as good as its resources plus its rules.** Novation does not create credit out of nothing; it concentrates credit risk onto an entity with a prescribed capital, margin and mutualised-loss structure. Whether that structure is adequate is the entire prudential debate.

### 4.2 Multilateral netting

Once every obligation points at the CCP, the CCP can net across the whole membership. Netting happens on at least three axes, and the axes matter because each one is a legal question before it is an arithmetic one:

- **By instrument and settlement date.** All of a member's buys and sells of the same instrument settling on the same date collapse into one net position.
- **By account.** House and client positions are netted separately, and within client business the account structure (omnibus versus individually segregated) determines what can be netted against what. ⚠ Segregation models are rulebook and jurisdiction-specific.
- **By product and clearing service.** Different clearing services have separate default funds and separate waterfalls, so netting does not cross service boundaries even at the same CCP.

Netting reduces the number and size of obligations that must be funded and settled, which is why it reduces liquidity needs. It also concentrates the residual obligation at a single point and a single moment, which is why it increases the consequences of a failure at that point. Both halves of that sentence are true simultaneously, and a design that only accounts for the first is the design that fails in a crisis.

### 4.3 What netting does not do

It does not reduce market risk. A member's net position carries the same price exposure as the gross position it replaced (assuming the trades offset in the same instrument). It does not reduce the CCP's exposure to a member whose positions are directional - netting only helps where the member's own book is two-sided. And it does not survive an unenforceable netting set: if the netting arrangement cannot be relied on in the member's insolvency, the CCP's true exposure is gross, and its margin was sized against a number that does not exist. This is why the legal opinion on netting enforceability is a piece of infrastructure in its own right.

### 4.4 Margin: initial and variation, distinguished by purpose

This is the part of the subject most often got wrong by people who understand the arithmetic but not the mechanism.

**Initial margin (IM)** is a forward-looking, model-based, posted-upfront estimate of the cost of closing out the position if the member defaults. Its defining properties:

- It estimates a **future** loss, not a current one. It covers potential future exposure over a liquidation horizon - the time to declare a default, auction or hedge the portfolio, and settle. That horizon is a modelling choice, not a market fact. ⚠ The specific horizon is set per CCP and per product class and is not asserted here.
- It is **model-based**: a scenario or historical-simulation construct calibrated to a confidence level, applied to the portfolio, with add-ons for concentration, wrong-way risk, liquidity and basis. Named methodologies exist as products - SPAN is the long-standing scenario-based example for listed derivatives - but the mechanism, not the label, is what an architect needs: a stressed loss distribution over a horizon, with a confidence target.
- It is **posted upfront and held**. It is not consumed by the day's price move; it sits there as the buffer against the move that has not happened yet.
- It is **not a forecast of the member's loss**. It is an estimate of the CCP's close-out cost, which is a different quantity, and one that depends on the CCP's ability to liquidate.

**Variation margin (VM)** is a settled-to-market payment that discharges the day's price move. Its defining properties:

- It is **backward-looking**: it is computed from the change in the mark since the last margin call, on the actual position.
- It is a **payment, not a deposit**. It moves value between the parties so that the exposure does not accumulate. Whether it is legally a settlement of the obligation or collateral against it matters for insolvency and accounting, and the distinction (settled-to-market versus collateralised-to-market) is a real one. ⚠ Presented from professional knowledge; not verified against a rulebook this pass.
- It is **the mechanism that turns a price move into a liquidity demand**. A volatility spike raises VM for every member with a position on the wrong side, at the same time, on the same day. That is a funding event, not a solvency event, and it is the channel through which a market shock becomes a liquidity shock.

The distinction between IM and VM is **purpose, not size**. A common error is to treat IM as "the big number" and VM as "the small daily one". In a violent move VM can dwarf IM for a directional member, and in a quiet market IM is the larger standing requirement. The correct statement is: IM pre-funds a hypothetical close-out; VM settles an actual price change. PFMI's own framing supports this: a CCP "should cover its current and potential future exposures to each participant fully with a high degree of confidence using margin and other prefunded financial resources" (Principle 4, Credit risk, ✅ CPMI-IOSCO, 16 April 2012) - "current" is the VM side of that sentence and "potential future" is the IM side.

**Intraday calls.** Margin is not only a daily event. PFMI states that a CCP "should have the authority and operational capacity to make intraday margin calls, both scheduled and unscheduled, from participants" (section 3.4, text preceding paragraph 3.4.17, ✅ CPMI-IOSCO, 16 April 2012). The architectural consequence is significant: the CCP's risk engine, its collateral valuation, its call notification and its payment path must all operate inside the trading day, not merely at end of day. Section 11 builds a worked example on exactly this.

### 4.5 The default waterfall

When a member defaults and its positions are closed out, the loss is allocated in a prescribed order. PFMI describes the typical shape (✅ paragraph 3.4.17, CPMI-IOSCO, 16 April 2012):

> "A CCP typically uses a sequence of prefunded financial resources, often referred to as a 'waterfall,' to manage its losses caused by participant defaults. The waterfall may include a defaulter's initial margin, the defaulter's contribution to a prefunded default arrangement, a specified portion of the CCP's own funds, and other participants' contributions to a prefunded default arrangement."

Note what that sentence does and does not say. It says "may include". It describes components, not a binding order. The binding order comes from the CCP's own rulebook - and PFMI requires that too (✅ paragraph 3.4.24): "The rules of an FMI should expressly set out the waterfall, including the circumstances in which specific resources of the FMI can be used in a participant default." The same paragraph adds a constraint that is easy to miss and operationally important: an FMI "should not include as 'available' to cover credit losses from participant defaults those resources that are needed to cover current operating expenses, potential general business losses, or other losses."

So the honest position is this: PFMI requires that a waterfall exist, be express in the rules, and be disclosed; it does not itself legislate one universal ordering. Anyone who tells you "PFMI prescribes the waterfall order" has not read 3.4.17 closely. The order is a rulebook artefact, and in the EU it is also a legal requirement under EMIR (⚠ described functionally here; the legal text could not be retrieved this pass - see section 15).

### 4.6 A published waterfall, layer by layer

The clearest way to see the mechanism is a real CCP's published order. Eurex Clearing publishes its waterfall and the sizes of its own layers. The order below is the CCP's own, and the euro figures are its own published figures as fetched on 26 October 2026 - they are quoted as an illustration of structure, not as an industry aggregate, and they will change.

| Layer | Resource | Whose money | Purpose |
|---|---|---|---|
| 1 | The defaulted member's margin collateral and its Default Fund contribution | The defaulter's | Internalise the loss in the party that caused it, before anyone else is touched |
| 2 | The CCP's own "Dedicated Amount" / Skin in the Game, EUR 143 million | The CCP's shareholders | Align the operator's incentive: the CCP loses its own money before it spends yours, which is the only reason members tolerate mutualisation below it |
| 3 | Non-defaulted members' Default Fund contributions, plus the "Second Skin in the Game", EUR 57 million | The surviving members, plus the CCP | Spread a tail loss across the surviving membership - the mutualised layer |
| 4 | Assessments (recovery cash calls) from members, taken simultaneously with a "Further Dedicated Amount" from the CCP on a pro-rata basis, maximum further amount EUR 300 million | Surviving members and the CCP together | The recursive layer: losses beyond the pre-funded pool are called, not pre-paid, and the CCP co-funds each call so the mutualisation is never one-sided |
| 5 | The CCP's remaining equity capital, EUR 600 million | The CCP's shareholders | The last private layer: the operating business is wiped out before any public money is contemplated |
| 6 | A letter of comfort from the parent exchange group, maximum EUR 600 million, with no third-party rights under it | The group | A parental backstop that is legally a comfort, not a guarantee - which is precisely why the "no third party has any rights" caveat matters |

Two structural observations that generalise beyond this CCP:

- **The layers alternate between "the defaulter pays" and "the collective pays".** Layer 1 is individual responsibility. Layers 2 and 3 interpose the operator before the collective. Layer 4 makes the collective pay again but forces the operator to co-pay. The alternation is deliberate: each mutualised layer is preceded by a layer that costs the operator money, so the operator has an incentive not to design a waterfall that relies on mutualisation too early.
- **The last layer is never really the last.** Beyond the letter of comfort sits the resolution authority and, in the limit, the state. Section 9 argues that this is the unprinted layer, and that its existence changes the incentive structure of every layer above it.

⚠ One further note: the same CCP applies a different waterfall to a different clearing model (an indemnified direct model, where an indemnifying clearing agent's indemnity covers losses after the defaulter's margin and Default Fund contribution). Waterfalls are per clearing service, not per institution. An architect integrating with "a CCP" must ask which service, because the loss allocation differs.

### 4.7 March 2020: the mechanism, and no magnitude

The March 2020 episode is the standard test case for margin mechanics, and it is usually told with numbers that this guide will not print. The FSB's own review states the finding with a notable hedge (✅ FSB, "Holistic Review of the March Market Turmoil", 17 November 2020):

> "Central counterparties remained resilient despite market turbulence, though margin calls may have been larger than expected in some cases."

That sentence is worth reading as a lesson in evidence discipline. "Remained resilient" is the conclusion. "May have been larger than expected in some cases" is the concession, and it is deliberately not quantified. The same review records that "the shock caused a fundamental repricing of risk and a heightened demand for safe assets", that dealers "may have been unwilling or unable to deploy substantial balance sheet capacity in an uncertain and volatile environment", and that "absent central bank intervention, it is highly likely that the stress in the financial system would have worsened significantly."

The mechanism, which is what an architect needs, does not require a magnitude:

1. A price shock raises variation margin for every member holding the losing side, on the same day, computed from the same closing prices. VM is a payment, so the demand is for cash, in the currency of the contract, by a deadline.
2. The same shock raises initial margin, because a model calibrated on historical volatility and correlations re-estimates a wider distribution. IM is a deposit, so the demand is for eligible collateral, and the eligibility rules are the CCP's.
3. Both demands land while the member's own clients are also being called, while the member's securities lending desks are seeing recalls, and while the funding markets that would normally intermediate are themselves stressed - which is the "unwilling or unable to deploy balance sheet" finding.
4. The result is a liquidity demand that is correlated across the membership by construction, because it is computed from the same prices. That correlation is the systemic feature: a CCP's margin system is a device that converts a market-wide price move into a market-wide simultaneous cash call.

No aggregate margin-call figure is printed in this guide, because no such figure was verified at a primary source in this pass. Where a mechanism is understood, a magnitude adds colour; where a magnitude is unverified, printing it adds only the appearance of evidence.

### 4.8 Where the thesis strains

Two honest complications, stated rather than buried:

- **Margin is not purely a loss-allocation device; it is also a behaviour-shaping device.** A CCP that raises IM in a stress event is not only pre-funding a loss, it is discouraging position-taking and forcing deleveraging. That is a policy effect, and it can amplify the move it is responding to. The procyclicality debate (section 9, section 13) is about exactly this: whether a mechanism that is individually prudent is collectively destabilising.
- **The waterfall's existence does not prove its adequacy.** A published order of layers is a statement about seniority, not about size. The sizes are the CCP's own estimates of what a stress might cost, and section 9's critique - that "skin in the game" may be calibrated to optics rather than to incentive - is a critique of exactly this gap.

---
## 5. The CSD: Book Entry, DVP and the Corporate-Action Clock

### 5.1 Two ways of not moving paper

A depository exists because physical certificates cannot be delivered at the rate the market trades. The historical answer was **immobilisation**: the certificates are held in a vault, usually in the name of a nominee or of the depository itself, and transfers between participants are recorded by book entry against the immobilised pool. The instrument still exists; only its movement has been suppressed. The later answer was **dematerialisation**: the certificate is abolished, and the authoritative record of the security is an electronic register operated by the issuer's agent or by the depository. There is nothing to immobilise because there is nothing physical.

Both solve the same problem - the delivery leg must complete inside the settlement window, and a certificate cannot be couriered inside a settlement window. Both create the same new problem, which is the real subject of a depository: when the record replaces the object, **the accuracy and finality of the record become the entire system of ownership**. A mistake in a register is not a lost document that can be reissued; it is a wrong statement of who owns what, propagating into entitlements, collateral values and votes.

The distinction matters operationally. In an immobilised structure there is still a physical instrument and an issuer agent whose register ultimately governs, so the depository's record is an intermediary layer that must reconcile upwards. In a dematerialised structure the depository's register (or the register it operates for the issuer) *is* the legal record, and there is no layer above to correct it. ⚠ Market-specific arrangements vary; the general distinction is standard but the details are jurisdictional.

### 5.2 Book-entry ownership and the nominee gap

The chain of records in a typical intermediated market runs: issuer, then the depository, then a participant (a custodian or broker), then the underlying client. Each link is a contractual and regulatory relationship, and each holds a record of the level below it.

The consequence is the **nominee gap**: the name on the register is usually the depository's nominee, not the investor. The investor holds a beneficial entitlement, enforceable through the intermediary chain, rather than legal title. This is not a technicality to be waved away:

- **Entitlements flow through the chain.** Dividends, coupons and voting instructions are paid and passed down, not sent directly. Every link is a place where an entitlement can be lost, delayed or misapplied.
- **Insolvency of an intermediary is the stress case.** Segregation rules, client-asset rules and the depository's participant-level structures exist to make the client's entitlement survive the intermediary's failure. Whether they work is a legal question about the specific structure, not a property of the technology.
- **The depository sees its participant, not the participant's client.** Its records are participant-level. That is why client-asset protection and position reporting are built as separate layers, and why "the CSD has your shares" is a statement about a chain, not about a vault.

### 5.3 Delivery versus payment: the three models

The definitive treatment remains the BIS report of September 1992, and its three models are still the vocabulary the industry uses (✅ BIS, Committee on Payment and Settlement Systems, "Delivery versus payment in securities settlement systems", Basle, September 1992).

**Model 1** - "systems that settle transfer instructions for both securities and funds on a trade-by-trade (gross) basis, with final (unconditional) transfer of securities from the seller to the buyer (delivery) occurring at the same time as final transfer of funds from the buyer to the seller (payment)." The report's conclusion on this model is unambiguous: it "can eliminate principal risk".

**Model 2** - "systems that settle securities transfer instructions on a gross basis with final transfer of securities from the seller to the buyer (delivery) occurring throughout the processing cycle, but that settle funds transfer instructions on a net basis, with final transfer of funds from the buyer to the seller (payment) occurring at the end of the processing cycle." The report notes that this design "has the potential to expose sellers of securities to substantial principal risk".

The mechanism of that exposure is worth stating precisely, because it is the most commonly misunderstood risk in settlement. In model 2, the seller delivers the securities early in the cycle and is paid at the end of it. Between delivery and payment, the seller has given up the asset and holds nothing but an unsecured claim on the buyer's settlement bank (or on the system's net funds position). If the buyer fails intraday, the seller is an unsecured creditor for the full value delivered. The system still calls itself DVP - delivery happens, payment happens, both happen "on the same day" - but the simultaneity that gives DVP its risk-eliminating property is absent. **Simultaneity is the property, not the label.**

**Model 3** - "systems that settle transfer instructions for both securities and funds on a net basis, with final transfer of funds from buyer to seller occurring at the end of the processing cycle." Netting both legs reduces the funding requirement and the number of transfers, at the cost of concentrating them at a single moment; some model 3 systems, the report notes, do not guarantee settlement at all.

The report's other durable contribution is its risk taxonomy, which an architect should keep separate: **principal risk** (the full value of the thing given away), **replacement-cost risk** (the cost of replacing the trade at current prices), and **liquidity risk** (the cash or securities needed at the settlement moment and not available). Netting reduces liquidity needs while concentrating them at one instant. Model 2 eliminates neither principal nor replacement-cost risk; it merely moves when they bite.

### 5.4 Corporate actions: an event-processing engine with a legal clock

The depository function that is least discussed and most operationally demanding is corporate actions. A depository is, in substance, an event-processing engine driven by a legal calendar: for each announced event, it must determine who is entitled, how much, by when, and what to do when the answer is a fraction, a choice, or a foreign tax question.

| Element | Mechanism | Where it goes wrong |
|---|---|---|
| **Entitlement determination** | Positions are snapshotted at a defined point; the snapshot, not the current position, drives the entitlement | The snapshot is taken against an unsettled trade population: trades executed but not yet settled are in the books, and the treatment of cum/ex positions is a market convention |
| **Record date versus ex date** | The record date fixes the entitlement; the ex date is the date from which a buyer no longer acquires the entitlement, and the two are separated by the settlement cycle in most markets | The gap between ex date and record date is normally sized to the settlement cycle. Shortening the cycle does not automatically move the convention, and a mismatch creates trades that are cum by the calendar and ex by the record - the classic corporate-action dispute |
| **Elections** | Events with options (cash or stock, currency choice, consent) require an instruction by a deadline, with a default election if none is received | The deadline is usually earlier than the record date and is set by the depository or the issuer's agent; a missed deadline silently produces the default, which is often the less valuable option |
| **Fractions and rounding** | Ratios that do not divide evenly produce fractional entitlements, which are rounded or cash-settled | Rounding direction and the cash price used for fractional settlement are economic terms, not implementation details; at scale they are a real P&L item |
| **Tax withholding** | Withholding depends on the investor's residence and on the documentation held in the chain; relief may be at source or by reclaim | The chain of intermediaries is where documentation is lost, so the rate applied depends on data quality several links away from the depository |
| **Proxy and voting** | Instructions flow down the chain and votes are aggregated per the issuer's rules, with share blocking or record-date locking in some markets | Aggregation, splitting and partial instructions are the hard cases; the deadline is often well before the meeting |

The architectural conclusion: a depository's corporate-action subsystem is a scheduling and state machine problem with legal deadlines, not a data-distribution problem. Its correctness criterion is that every entitlement is computed from the right snapshot, with the right ratio, applied at the right time, to the right account - and its failure mode is silent, because a misapplied entitlement looks exactly like a correct one until someone reconciles.

### 5.5 Why the depository's clock is not the venue's clock

The venue's clock is a sequence number. The depository's clock is a calendar with cut-offs. They interact in exactly one place, and it is where most settlement problems are born: the trade date to settlement date interval, which determines which corporate actions a buyer captures, which funding day the cash lands on, and whether a fail will cure before a record date. Section 6 is about shortening that interval, and section 5.4's record-date row is the reason shortening it is not a pure win.

---

## 6. The Settlement Cycle and T+1

### 6.1 What a cycle is, and why it exists

The settlement cycle is the interval between the trade date and the intended settlement date, expressed in business days: T+2 in most of Europe and Asia today, T+1 in the United States since 2024, with the EU and UK legislating for T+1 in 2027.

The interval exists for two reasons, and they pull in opposite directions:

- **It is a risk window.** Between trade date and settlement, each party is exposed to the other. Shortening the window reduces counterparty exposure, reduces the collateral and capital consumed by unsettled trades, and reduces the number of things that can go wrong between agreement and discharge.
- **It is a processing buffer.** Inside the window a chain of human and system steps must complete: allocation of an institutional fill to underlying accounts, confirmation of the economics, affirmation by the client, enrichment with standing settlement instructions, netting, funding, securities location and borrowing, and finally the settlement instruction at the CSD. Shortening the window compresses the buffer, and the buffer is where exceptions are handled.

Every T+1 programme is therefore a compression of exception handling, not a compression of happy-path processing. The happy path was already fast; the buffer existed for the trades that are not happy.

### 6.2 The United States: 28 May 2024

The US moved the standard cycle for most broker-dealer transactions in US securities from T+2 to T+1 on 28 May 2024, by amendments to Rule 15c6-1 under the Securities Exchange Act of 1934 (✅ SEC Division of Examinations, Risk Alert "Shortening the Securities Transaction Settlement Cycle", 27 March 2024, describing the move as made "pursuant to the adoption by the Securities and Exchange Commission of amendments to Rule 15c6-1"). The adopting release is Exchange Act Release No. 96930 of 15 February 2023, published at 88 FR 13872 on 6 March 2023 (✅ same source).

The same date carried a second load: 28 May 2024 was also the compliance date for new rules on the processing of institutional trades by broker-dealers and certain clearing agencies (including central matching service providers) and for recordkeeping amendments applying to registered investment advisers (✅ same source).

The SEC staff's own list of what the change compresses is the best available operational checklist, because it is written by the supervisor that examines against it (✅ all items from the same Risk Alert):

- Timeframes to close out most fail-to-deliver positions under Rule 204 of Regulation SHO are shortened.
- The window to send confirmations under Exchange Act Rule 10b-10 is shortened.
- The number of days available to obtain possession of customer securities before close-out under Rule 15c3-3(m) is reduced.
- Examination focus areas named: allocations, confirmations and affirmations (ACA); counterparties in other time zones; end-of-day trades; stock lending and recalls of loaned securities; and funding and liquidity for transactions such as international securities that still settle T+2 and ETF creations and redemptions whose underlying securities do not settle T+1.

### 6.3 The European Union: 11 October 2027

The EU will switch to T+1 on 11 October 2027 (✅ ESMA press release, 16 October 2026, "One year to go for T+1 settlement: the regulatory framework is complete, now let's make it happen", reference ESMA71-545613100-2889). ESMA states that with publication of the regulatory technical standards and the revised guidelines on standardised procedures and messaging protocols, "the regulatory framework is now ready".

The date itself was recommended earlier: ESMA's final report of 18 November 2024 (ref ESMA74-2119945925-1969) recommended a Q4 2027 migration, simultaneous across all relevant instruments, and recommended 11 October 2027 as the optimal date, while noting that amendment of CSDR and of the settlement discipline framework was needed for legal certainty (✅ as cited in the evidence base for this guide).

Critically for anyone planning the work, the first binding deadline is **not** the go-live. From 7 December 2026, investment firms and their clients must comply with new requirements on allocations and confirmations, including the mandatory use of electronic, standardised communication channels and international messaging standards, and machine-readable communication methods except in cases of temporary technical disruptions (✅ ESMA press release, 16 October 2026). ESMA had flagged 2026 as the critical preparation year in its statement of 20 July 2026 (ref ESMA74-2119945926-3773). The practical reading: the messaging and affirmation plumbing must be rebuilt more than a year before the cycle changes, because the cycle change is only survivable if the plumbing already works.

### 6.4 The United Kingdom: 11 October 2027, mandated through the CSD rulebook

The UK government "committed to legislate to mandate T+1 as the standard settlement period in the UK from 11 October 2027" (✅ HM Treasury policy paper, "Accelerated Settlement (T+1)", first published 6 February 2025, last updated 20 November 2025). The same note records that since 2014 the UK, with the EU, has required most securities trades to settle at T+2, and notes that a further shortening reduces counterparty risk and encourages automation.

The mechanism is the interesting part for an architect. The draft statutory instrument published on 20 November 2025 - "The Central Securities Depositories (Amendment) (Intended Settlement Date) Regulations 2026" - works by amending the CSD regulation to change the intended settlement date (✅ same source). In other words, the mandate is expressed as a change to the depository's rulebook obligation rather than as a conduct rule on investment firms. That choice tells you where the enforcement point sits: the depository's rulebook becomes the instrument that defines the default settlement date, and participants comply by complying with the CSD.

### 6.5 What actually changes, operationally

The cycle change is a change to the shape of the day, not to the length of the month. Six consequences, in rough order of how often they are underestimated:

**The ACA window.** Allocation, confirmation and affirmation must complete on trade date. In the US institutional model, the affirmation is the gate: an unaffirmed trade is out of the T+1 process and on the path to a fail. Compressing this window changes the operating model from "overnight batch with a next-morning clean-up" to "intraday, with an exception queue that must be empty before the market closes". This is why the EU made the messaging standards a hard requirement with a date of its own: a T+1 cycle running on email and spreadsheets is arithmetically impossible at volume.

**Securities lending and recalls.** A recall of loaned securities previously had a full day of slack between the recall and the delivery obligation. In a T+1 world, a recall that arrives after the lender's cut-off cannot be satisfied in time, so the borrower must either source the security elsewhere intraday or fail. The consequence is a structural change in lending terms - recall cut-offs, recall indemnities, and the price of same-day availability - not merely a faster process.

**FX funding.** The funding leg and the securities leg must now line up on the same day. For a cross-currency trade, the currency has to be in place before the securities settlement cut-off, which means the FX settlement must complete earlier than the securities leg - and FX settlement runs on its own infrastructure with its own cut-offs and its own settlement dates. Where the FX leg settles on a cycle longer than T+1, the participant must prefund or use an intraday facility. This is the constraint that catches firms which modelled T+1 as a securities-only project.

**ETF creation and redemption.** An ETF share settling T+1 whose underlying basket settles T+2 creates a structural mismatch: the authorised participant delivers the basket after receiving the shares, or must bridge the gap. The SEC staff named this explicitly as an examination focus (✅ Risk Alert, 27 March 2024).

**International securities with a T+2 local cycle.** A trade in a T+1 market settled through a T+2 market's infrastructure inherits the mismatch. The workaround is prefunding, a bridging facility, or a different settlement route - and each of those is a balance-sheet decision, not a technology one.

**The fail.** A fail is a failure to deliver securities on the intended settlement date. Its consequences are a buy-in (the buyer or the CSD arranges a replacement purchase at the failing party's expense), a cash penalty accruing for each business day of delay, and close-out of the position (⚠ the CSDR settlement discipline regime works this way: cash penalties and buy-in, with the buy-in regime's application suspended for a period by delegated regulation - described here functionally because the legal text could not be retrieved this pass). In a T+1 world there is less time to cure before the consequence bites, which is precisely why the SEC staff listed shortened Rule 204 close-out timeframes as a consequence of the change (✅ Risk Alert, 27 March 2024).

### 6.6 The thesis test

T+1 does not remove settlement risk. It shortens the window in which the existing mechanisms operate, and in doing so it relocates risk from the settlement date to the funding and affirmation window. The promises are unchanged - the seller still must be protected against the buyer's failure, the buyer against the seller's - but the time available to make good on them is halved, and the cost of a broken promise arrives sooner. A firm that reads T+1 as "the same process, faster" has understood the arithmetic and missed the mechanism. A firm that reads it as "the intraday funding and affirmation window is now the binding constraint on my business" has understood both.

---

## 7. Venue Technology and Latency: The Operator's Problem

### 7.1 The four stages

Almost every electronic venue, from an equity exchange to a derivatives market, is built as four stages in series:

1. **Gateway.** Terminates participant sessions, authenticates, applies admission and (where the venue offers it) pre-trade risk checks, enforces throttles and message limits, and translates the participant's protocol into the venue's internal command format. Its job is to be a controlled boundary: nothing enters the book that has not passed the same checks as everything else.
2. **Sequencer.** Assigns a single total order to all commands from all gateways, and publishes that order. This is the point at which "simultaneous" messages stop being simultaneous.
3. **Matching engine.** A single-threaded, deterministic consumer of the sequenced command stream. It owns the book state and emits executions, acknowledgements and rejections.
4. **Market data publisher.** Derives the public feed from the engine's output and fans it out to all subscribers, typically with its own sequence space and its own conflation and snapshot mechanisms.

Around these sit the operational systems - reference data, session management, surveillance, drop copies, clearing interfaces - but the four stages are the trading path, and their properties determine what the venue can honestly promise.

### 7.2 The sequencer is the fairness device

Participants experience latency as a continuum: their order leaves, time passes, something comes back. The venue does not have a continuum; it has an ordered log. Two orders that arrive within the same nanosecond are not simultaneous to the venue - the sequencer puts one before the other, and that ordering is the fact of the matter.

This is why the sequencer, not the matching algorithm, is where fairness is built:

- **One order, published, auditable.** The sequence number is the authoritative record of what happened first. Disputes are resolved by reference to it, not to gateway timestamps.
- **No privileged path.** If every gateway feeds the same sequencer and no participant has a route into the engine that bypasses it, then no participant can be filled ahead of an order that was sequenced earlier. Colocation changes *when* a participant's message reaches the sequencer; it does not change the rule the sequencer applies.
- **Back-pressure is a fairness decision.** When the inbound rate exceeds capacity, the venue must choose: drop, reject, queue, or throttle per session. Each choice advantages some participants over others, which is why the policy is published and why "we just slow down under load" is a market-integrity statement, not an engineering one.

### 7.3 Determinism, replay and the recovery requirement

The engine's output must be a pure function of its input sequence (section 3.6). In practice, the hazards that break this are unglamorous and recurrent:

- **Time inside the logic.** Any decision that reads a wall clock inside matching makes the output non-reproducible. Event time must be carried on the message, assigned by the sequencer, not sampled by the engine.
- **Order of iteration.** Allocating a fill across a price level by iterating a hash map produces a different allocation on a replay with a different hash seed or insertion order. The data structure must have a defined order, and that order must be the priority rule.
- **Arithmetic.** Prices and quantities must be handled in exact decimal or scaled integer form; binary floating point produces rounding differences that accumulate into position breaks, and a replay that produces a different fill quantity is not a replay.
- **External dependencies.** A matching decision that calls a reference-data service, a risk service or a clock service is non-deterministic by construction, because it depends on state outside the log.
- **Retries and idempotency.** A participant retry that the gateway forwards twice creates two orders. The venue needs a client-assigned idempotency key, or the retry must be resolved at the session layer, before sequencing.

Recovery follows from determinism: snapshot the state periodically, keep the complete command log, and rebuild by replaying from the snapshot. A standby that has consumed the identical log is in the identical state, which is why "warm standby" on a venue means "a second engine fed the same sequenced stream", not "a replica of the database". The requirement is not zero downtime; it is that the state after recovery equals the state the participants were told about. Those are different requirements, and confusing them produces systems that fail over quickly into a state nobody can reconcile.

### 7.4 Colocation as equalisation, not privilege

Colocation is often described as an advantage sold to the fastest participants. From the operator's side, the design intent is nearly the opposite: colocation exists so that the *marginal* advantage of physical proximity can be made equal for everyone who pays for it. Equal-length fibre from each participant cage to the venue's switches, symmetric cross-connects, identical power and cooling, a published cable-latency budget per cage, and a rule that no participant gets a shorter path - these are engineering commitments to a fairness property.

The commercial reality is that only some participants buy it, so it functions as a tier. But the operator's obligation is that within the tier, no participant is advantaged by the venue's own construction. When that obligation is breached - a cage with a shorter fibre run, a cross-connect that bypasses a switch - the venue has created a market-integrity problem that no amount of disclosure fixes, because the advantage is in the physics.

### 7.5 Why the operator's latency problem is a different problem

The participant optimises its own path: one order in, one acknowledgement out, as fast as possible, and the relevant statistic is the distribution of that single path's latency.

The operator optimises something structurally different:

- **Fan-out.** One execution must be delivered to thousands of subscribers, each with its own session, its own conflation preference and its own slow-consumer behaviour. The publisher's problem is delivering the same ordered stream to all of them without letting the slowest consumer dictate the pace.
- **Tail, not median.** The operator's risk is the p99.9 and the maximum: a gateway session that stalls for a second delivers stale acknowledgements and can cause a participant to trade on a book that no longer exists. A publisher that backs up under load delivers an out-of-order or delayed view to everyone downstream, which is a market-data integrity failure rather than a slow experience.
- **Jitter as a correctness issue.** For the participant, jitter is a performance problem. For the operator, a variable delay between sequencer and publisher means different participants observe events in different orders relative to their own actions, which is exactly the ambiguity the sequencer exists to remove.
- **Determinism under load.** The operator must degrade in a defined way. A venue that sheds load by dropping market data for some sessions has broken its own evidentiary trail.

The participant-side treatment of this - the microsecond stack, kernel bypass, feed handlers, the OMS/EMS path - is in the [Trading System Software Architecture Guide](../technology/trading_system_software_architecture_guide.md), sections 5, 7 and 9. It is deliberately not repeated here. The point of this section is narrower: the venue's latency work is about *equalisation and determinism under fan-out*, and a venue that markets its speed without publishing its tail behaviour and its fairness construction has told you nothing about the property that matters.

---

## 8. The Regulatory Frame

### 8.1 PFMI: the standard underneath everything

The foundational document is "Principles for financial market infrastructures", issued by the Committee on Payments and Market Infrastructures (CPMI) and the International Organization of Securities Commissions (IOSCO), dated 16 April 2012 (✅ CPMI-IOSCO, BIS, 16 April 2012). It sets out principles for payment systems, central securities depositories, securities settlement systems, central counterparties and trade repositories, plus a set of responsibilities for the authorities that regulate them.

Three things about its status are commonly misunderstood, and each has a practical consequence:

- **It is a standard, not a law.** PFMI has no direct legal force. It is implemented through jurisdiction-specific rules (in the EU, principally EMIR for CCPs and CSDR for CSDs; elsewhere through national regimes that adopt or reference it) and, below those, through each infrastructure's own rulebook. The chain is: standard, then law, then rulebook, then operating procedure, then system control. An architect who reads PFMI as if it were a regulation will over-specify in some places and miss the binding requirement in others.
- **It requires disclosure, and the disclosure is public.** PFMI operates with a companion disclosure framework and assessment methodology (✅ both are named in the PFMI text itself, which refers to the "CPSS-IOSCO Disclosure framework for financial market infrastructures" and the "CPSS-IOSCO Assessment methodology for the principles for FMIs and the responsibilities of authorities"). Infrastructures publish responses to the disclosure framework, which means the answer to "how does this CCP meet Principle 4" is a public document, not a confidential one. This is the single most useful research shortcut in the subject: the disclosures describe the actual mechanisms, in the operator's own words.
- **It is principle-based and therefore arguable.** "Should cover ... fully with a high degree of confidence" (Principle 4) is a standard that leaves the calibration open. That is deliberate - it accommodates different markets - and it is also the space in which the debates of section 13 live.

Two PFMI requirements are load-bearing for this guide and are quoted where they are used: paragraph 3.4.17 on the waterfall's typical components and paragraph 3.4.24 on the requirement that the rules expressly set the waterfall out (both ✅, section 4.5 above). Principle 4's cover requirement - resources sufficient for the default of the two participants and their affiliates that would cause the largest aggregate exposure in extreme but plausible conditions, for CCPs that are systemically important in multiple jurisdictions or have a more complex risk profile, and cover-one for the others - is the sizing standard behind the default fund (✅ CPMI-IOSCO, 16 April 2012).

### 8.2 The instruments that give it legal force

Described functionally, without article numbers, because the legal texts could not be retrieved in this pass (see section 15). Everything in this subsection is ⚠ on that basis: the descriptions are of the instruments' function and are standard professional knowledge, but they are not quoted from the legislation.

**EMIR** (the European Market Infrastructure Regulation) is the EU's CCP and trade-repository regulation. Functionally it does five things that matter to an operator: it requires CCPs to be authorised and to meet capital, margin, default-fund and governance requirements; it imposes the clearing obligation on specified classes of derivatives; it requires risk mitigation for non-cleared derivatives (including margin exchange); it requires transactions to be reported to trade repositories; and it establishes a framework for third-country CCPs, including recognition and, for the most systemic, enhanced supervision. For an architect, EMIR is the reason a CCP's margin methodology, default fund sizing and waterfall are not merely commercial choices - they are supervised, documented and stress-tested requirements. ⚠ The 2024 revision often called "EMIR 3" adds active-account and operational-efficiency requirements; not verified this pass.

**MiFID II and MiFIR** are the EU's trading-venue and transparency regime. Functionally, MiFID II sets organisational and conduct requirements (including for investment firms and for the operation of trading venues), and MiFIR sets the transparency and market-structure rules, including the categories of execution venue and the pre- and post-trade transparency obligations that go with them. For a venue operator, this is the law that determines what kind of venue it is allowed to be, what it must publish, and what it must record. ⚠

**CSDR** (the Central Securities Depositories Regulation) is the EU's depository regulation. Functionally it covers authorisation and supervision of CSDs, organisational and prudential requirements, access and interoperability, settlement discipline (cash penalties for fails and a buy-in framework), and the rules on internalised settlement. It is also the instrument being amended to introduce T+1 in the EU, and the instrument through which the UK is introducing T+1 domestically (section 6.4). ⚠ Note the pattern: the settlement cycle is set in the depository regulation, not in conduct rules. That is a design choice about where the obligation bites.

**The Settlement Finality Directive** is the EU instrument that protects transfer orders and collateral security from the consequences of insolvency proceedings. Functionally it does two things an architect must design around: it determines the moment at which a transfer order enters a designated system and the moment at which it becomes irrevocable, and it protects those orders (and qualifying collateral) from being unwound by the insolvency of a participant. This is the legal foundation of "settlement finality" as an engineering concept - it is why the CSD's processing cycle has a defined, published cut-off, and why the system's definition of "final" is a legal statement rather than a database flag. ⚠

### 8.3 The rulebook as the requirements document

The most useful reframing in this section: for a venue, a CCP or a CSD, **the rulebook is the requirements specification**. The regulation says the waterfall must be express in the rules (PFMI 3.4.24) and the law says what the rules must cover (EMIR, CSDR); the rulebook then states the actual mechanism - the margin methodology, the eligible collateral, the call timetable, the default procedure, the auction process, the allocation of losses. The systems implement the rulebook, and a system that implements something else is non-compliant regardless of whether it works.

This is why operator-side architecture reviews start with the rulebook and the disclosure, not with the code. It is also why a change to the rulebook is a change request to the technology estate, with a legal effective date and a communication obligation to participants - a coupling that banks and brokers, who change their own systems on their own schedules, routinely underestimate.

### 8.4 The layering, stated once

| Layer | Instrument or artefact | Force | Changes how often |
|---|---|---|---|
| International standard | PFMI (CPMI-IOSCO, 16 April 2012) | None directly; implemented nationally | Rarely |
| Law and regulation | EMIR, MiFID II/MiFIR, CSDR, Settlement Finality Directive (EU); national equivalents elsewhere | Binding | Years, but steadily |
| Regulatory technical standards and guidelines | Level 2 and 3 measures, including the CSDR settlement discipline RTS and the messaging guidelines with the 7 December 2026 deadline | Binding or comply-explain | Months to years |
| Rulebook and published procedures | The infrastructure's own contract with its participants | Contractual, supervised | Quarterly to annually |
| Operating procedure and system control | What the operator actually does, and what the system enforces | Internal | Continuously |

An architect's mistake at any layer is to design to the layer above and assume the layer below will follow. The waterfall is the clearest example: the standard says it must exist and be express, the law says it must meet certain requirements, and the rulebook states the order and the amounts - and it is the rulebook, not the standard, that a system must implement.

### 8.5 Reading a disclosure as an architecture document

Because the disclosure framework makes the substantive material public, a disclosure statement is the fastest way to learn how an infrastructure actually works, and it is also the closest thing to a specification an integrator will get. Six things to look for, in the order that matters for an integration:

- **The waterfall, as an order and as sizes.** Not the diagram, but the rulebook clause and the published layer amounts, plus the circumstances in which each layer may be used. If the disclosure gives a percentage split between the CCP's own layer and the mutualised pool, that ratio is the single most informative number on the page about the operator's incentive structure.
- **The margin methodology, including the model's stated horizon and confidence, and the intraday call policy.** A methodology that states its liquidation horizon and its confidence level is one an integrator can test against; one that does not is a black box with a supervisory approval behind it.
- **The collateral schedule.** Which assets, at what haircuts, with what concentration limits, and what happens when a member breaches a limit. The collateral schedule, not the margin number, is what determines whether a member can actually fund a call in a stress event, because the assets it holds may not be the assets the CCP accepts.
- **The default procedures.** The steps, the decision authority, the notice periods, the auction design and the fallbacks if the auction fails. This is the section that is most often read by legal and least often read by technology, and it is the section that determines what the systems must be able to do on the worst day.
- **The settlement cycle timetable and cut-offs.** For a CSD, the cycle structure, the currency cut-offs, the finality point and the behaviour of the system in each state are the operational interface.
- **The participant and account structure.** Membership categories, segregation models, and what the infrastructure can and cannot see of a participant's clients. This determines the data model of the integration, and getting it wrong is the most expensive kind of rework.

The habit worth building: read the disclosure before the technical specification, and read the rulebook clause before the disclosure. The disclosure tells you what the operator says it does; the rulebook tells you what it is contractually obliged to do. Where the two differ, the rulebook wins, and the difference is worth a question to the operator.

---
## 9. The Systemic-Risk Lens

### 9.1 Why these three institutions are treated differently

A venue, a CCP and a CSD are not ordinary businesses that happen to be large. They are systemically important for reasons that are structural rather than a matter of size, and the standard's own framing is a useful starting point: concentration, substitutability and interconnectivity.

- **Concentration.** Clearing and settlement are natural monopolies in a way that trading is not. A CCP that clears a product class clears most of it, because netting only works if the population is large - a member wants to net against everyone, which drives everyone to the same CCP. A CSD is a monopoly by construction: there is one register per issue. Trading, by contrast, fragments happily across dozens of venues.
- **Substitutability.** If a venue fails, flow moves to another venue in days. If a CCP fails, the product class has no clearing, and therefore no trading, until the positions are transferred or the CCP is reconstituted. Substitutability is close to zero on any relevant timescale, which is precisely why the failure cannot be allowed to happen in an uncontrolled way.
- **Interconnectivity.** Each institution sits between its members, and its members are each other's counterparties elsewhere. A CCP's members are banks, brokers and their clients; a CSD's participants are the same population plus issuers and agents. The interdependency graph is dense by design, because that density is what netting exploits.

### 9.2 The honest asymmetry

The standard argument for central clearing is that it reduces bilateral contagion: novation severs the counterparty link (section 4.1), so one member's failure does not propagate through a web of bilateral exposures. That argument is correct, and it is also incomplete in a way that an architect should be able to state without flinching.

Central clearing **reduces** the number of transmission paths and **concentrates** the remaining risk at a single node. Before clearing, a member's failure transmits to its bilateral counterparties in proportion to their exposures. After clearing, the same failure transmits to the CCP, and from the CCP to the membership in a prescribed order defined by the waterfall. The total exposure may be smaller - netting and margining see to that - but it now has a single point of failure whose failure would be simultaneously a failure of every market it clears. The correct statement is not "central clearing removes systemic risk" but "central clearing trades a diffuse, hard-to-measure, hard-to-manage web of bilateral exposures for a concentrated, measurable, managed exposure to one institution, whose own failure becomes a systemic event".

That trade is worth making, which is why the regime exists. But it means the CCP's resilience is not one risk among many - it is the load-bearing assumption of the whole structure.

### 9.3 What "too big to fail" means for a CCP specifically

For a bank, "too big to fail" means the state may intervene to prevent disorderly failure. For a CCP, the logic is sharper and arrives earlier: **a CCP cannot be allowed to fail in the middle of a default auction**. The failure would occur exactly when the market needs the CCP to be performing - positions unhedged, collateral in limbo, members unable to know their exposures - and there is no mechanism for pausing the market while a CCP is restructured. Resolution of a CCP therefore has to preserve continuity of the critical service, which is a different objective from resolving a bank.

The consequence is a layer that no waterfall prints. Below the CCP's equity, below the parental letter of comfort, sits the implicit commitment of the state (and, for a systemic CCP, of more than one state) that the clearing service will continue. Nobody has signed it; its existence is inferred from the fact that the alternative is unacceptable. And that inference changes the incentives of every layer above it:

- If the last layer is a public backstop, the mutualised layers are not really the last resort, and the discipline that mutualisation is supposed to impose on members is weaker than the rulebook implies.
- If the CCP knows the backstop exists, its incentive to size the default fund generously is weaker than the "skin in the game" design assumes.
- If members know it exists, their incentive to monitor the CCP's risk management is weaker than a pure mutualisation model would produce.

This is not a criticism of any particular institution. It is a structural feature of the arrangement, and it is the strongest evidence for this guide's thesis: the promise that ultimately holds the system together is a political one, and the technology and the rulebook are the part that can be written down.

### 9.4 Recovery versus resolution

The distinction is precise and often blurred:

**Recovery** is what the CCP does to itself, using the tools in its own rulebook, to restore a matched book and a funded balance sheet after a loss that has exhausted part of the waterfall. PFMI requires the planning (✅ the standard states that an FMI "should identify scenarios that may potentially prevent it from being able to provide its critical operations and services as a going concern and assess the effectiveness of a full range of options for recovery or orderly wind-down", and should prepare plans on that basis, providing authorities with information needed for resolution planning - CPMI-IOSCO, 16 April 2012, Principle 3). The tools themselves are rulebook artefacts, and the ones usually cited are:

- **Assessments / cash calls** - additional contributions from surviving members, as in layer 4 of the waterfall in section 4.6.
- **Variation margin gains haircutting (VMGH)** - reducing the variation margin paid to members who are in the money, so that the CCP retains cash it would otherwise pay out. Economically it converts a payment owed to winners into a loss allocated to them. ⚠ Not verified against a specific rulebook this pass.
- **Partial tear-up** - terminating a defined portion of the matched book, typically selected so that the reduction in exposure is balanced, so that the CCP's residual exposure falls. ⚠ Same caveat.
- **Auction and porting** - transferring the defaulter's positions and client business to other members, which is the primary recovery action and the one that determines whether the other tools are needed at all.

**Resolution** is what an authority does to the CCP when recovery has failed or is not credible: intervention, a stay on termination rights, transfer of the critical functions to a bridge or a purchaser, and allocation of losses through statutory powers rather than the rulebook. Resolution regimes for CCPs are newer and less tested than bank resolution regimes, and the cross-border case - a CCP clearing in many jurisdictions, with a resolution authority in one of them and members in all of them - is the hardest unsolved problem in the subject. ⚠ Presented as an assessment of the state of the regime; not sourced to a specific instrument this pass.

### 9.5 The open questions

Four, stated as questions rather than answers, because they are genuinely contested (section 13 sets out what is settled and what is not):

1. **Does mutualisation create moral hazard at the member level?** A member that knows a tail loss will be spread across the default fund has weaker incentives to monitor its own risk than one that would bear it alone. The counter-argument is that default fund contributions are sized to the member's own risk, so the incentive is preserved through the contribution rather than through the loss.
2. **Is "skin in the game" calibrated to the incentive or to the optics?** A layer that is large enough to be credible but small relative to the default fund does not meaningfully change the operator's behaviour; a layer that is large enough to change behaviour may be larger than the operator's shareholders will accept. There is no published calibration standard, and the sizes are set by the CCP within supervisory constraints.
3. **Is the cross-border resolution gap closable?** A CCP's members are in many jurisdictions, its collateral is in many jurisdictions, and its resolution authority is in one. Mutual recognition of resolution actions, and the treatment of a member's default fund contribution in its home insolvency, are unresolved.
4. **Does procyclicality have a fix, or only a trade-off?** Margin that responds to volatility is prudent for the CCP and destabilising for the system. Anti-procyclical tools (margin floors, through-the-cycle calibration, smoothing, and the regulatory caps and floors on margin changes) exist but all of them reduce the CCP's protection in exactly the scenario where it is needed. ⚠ The specific tools are regulatory-practice knowledge, not verified here.

### 9.6 Where the thesis holds and where it strains

The thesis holds strongly here: the systemic-risk literature on CCPs is, almost entirely, a literature about loss allocation - who bears the tail, in what order, and with what incentive. The parts that are not about loss allocation (throughput, market data, connectivity) are the parts that do not generate systemic risk.

It strains in one place, and the strain is worth naming. A CCP's most important promise - that it will continue to clear - is not a promise the CCP can keep on its own resources. It is kept by the arrangement as a whole, including the state. The loss-allocation account describes the part of the promise that is written down; the part that is not written down is what makes the written part credible. An architect who designs only to the written part has designed to the rulebook, which is correct, and has missed the reason the rulebook is believed, which is where the judgement sits.

---

## 10. Failure Modes

Six failure modes, each stated as Symptom, Cause and Guardrail. The purpose is not a risk register but a diagnostic vocabulary: each of these has a specific mechanism, and each is preventable by a specific design decision that is cheap before the event and unavailable during it.

### 10.1 A matching engine that halts and cannot restart deterministically

**Symptom.** The venue halts. On restart, participants' order states do not match what they were told before the halt: orders that were acknowledged are missing, fills that were reported are absent, or the book is at a state that corresponds to a point before the last published sequence number. Reconciliation takes hours and the venue cannot say with confidence which of two states is correct.

**Cause.** The engine's state was never a pure function of its input. Typical culprits: a matching decision that read a wall clock; allocation across a price level that depended on hash iteration order; a risk or reference-data call inside the match path; a recovery path that restored from a database backup rather than replaying the sequenced log, so the restored state reflected committed transactions rather than published events. The halt is not the failure; the inability to prove the post-restart state is.

**Guardrail.** Design the engine as an explicitly replicated state machine with a published input log, deterministic replay, and a periodic snapshot whose sequence number is part of the public record. Test recovery by replaying a production log in a test environment and diffing the output against the original - not by failing over in production and hoping. Publish the recovery point in sequence-number terms, so participants can verify their own state against it.

### 10.2 A margin model that procyclically amplifies stress

**Symptom.** A volatility event triggers a margin increase that forces members to delever into a falling market, which raises volatility further, which triggers another increase. Members reduce positions not because their view changed but because their collateral did, and the CCP's own close-out, if a member fails, happens into a market it has just helped to sell.

**Cause.** Initial margin is calibrated on recent volatility and correlation, with a short lookback and a confidence target. That is responsive, which is prudent for the CCP in isolation, and it is a positive feedback loop when applied simultaneously across the membership, because every member's requirement rises at the same moment and the rise is computed from the same prices. The mechanism is not a modelling error; it is the intended behaviour of a model that measures current risk, applied to a system whose participants all measure it at once.

**Guardrail.** Anti-procyclical margin tools: a floor under the margin requirement, calibration through a cycle rather than to the last observed volatility, limits on the rate at which requirements may rise, and an explicit policy for the release of margin in a stress unwind. Every one of these trades CCP protection for system stability, which is why the choice is supervisory rather than technical - but the choice must be designed, documented and stress-tested, not discovered during the event. ⚠ Specific regulatory caps and floors are practice knowledge, not verified here.

### 10.3 A default waterfall nobody has tested

**Symptom.** A member defaults. The CCP's default management group convenes, and discovers that the auction process has never been run with real positions; that the assessment mechanism requires a legal notice with a notice period that exceeds the time available; that the hedge leg needs a trading permission the group does not hold; or that the default fund contribution of a member in another jurisdiction cannot be called within the resolution timeline.

**Cause.** The waterfall is documented as a legal instrument and tested as a financial calculation - the arithmetic of layer depletion is modelled - but not as an operational process. The gap is always in the non-arithmetic parts: who decides, on what authority, with what notice, in what sequence, using which systems, with what fallbacks when the auction fails.

**Guardrail.** Rehearse the process, not the arithmetic. Run periodic default-management exercises with real portfolios, real systems and real decision-makers, including the auction, the hedging, the client-porting steps and the assessment calls. Test the notice mechanics against the legal calendar. Test what happens when the auction produces no bids - because that is the scenario in which the waterfall is actually used. Section 12's anti-pattern on documented-but-unrehearsed waterfalls is the same point from the design side.

### 10.4 A settlement fail that cascades through the funding leg

**Symptom.** One delivery fails. The buyer, who expected the securities, has already committed the cash - or has already onward-delivered to its own buyer. The onward delivery fails too. The cash that was to fund a currency purchase is not there, so the FX leg fails, so the counterparty's funding for a different settlement fails. By end of day the original single fail has produced a cluster of fails across instruments and currencies.

**Cause.** The settlement chain is a chain of dependencies, and the DVP property protects only the two parties in the specific transaction. A participant that has pre-sold securities it has not yet received has taken on delivery risk that DVP does not cover; a participant that has funded a purchase with the proceeds of a sale it has not yet settled has taken on timing risk that DVP does not cover. Shortening the cycle removes the slack that used to absorb these dependencies.

**Guardrail.** Model the funding leg and the delivery leg as one dependency graph, not two processes. Track fails as a liquidity event with a projection of downstream obligations, not as an exception queue with an owner. Hold buffers at the point where the graph converges - the settlement bank account and the securities lending inventory - and size them against the joint peak rather than each leg's individual peak. This is the mechanism the worked example in section 11 turns on.

### 10.5 A corporate action misapplied at scale

**Symptom.** A ratio or a withholding rate is applied incorrectly to a large population. Nothing fails; every message is well formed, every entitlement is paid, and the error is discovered weeks later when a client reconciles its expected dividend against the cash received. By then the position has moved, the reclaim window may have closed, and the correction is a manual process across thousands of accounts.

**Cause.** Corporate actions are processed per market, per event type, with market-specific conventions encoded as rules. A single wrong parameter (a rounding direction, an ex-date convention, a tax rate for a residence class, a default election) propagates to every account in the affected population. The failure is silent because the output is plausible.

**Guardrail.** Treat the corporate-action engine as a legal-date scheduler with four-eyes control on parameters, a mandatory dry-run against a shadow position file before the event, and an exception report that compares computed entitlements against an independent source for a sample. The control that matters is not the calculation but the *independent check of the parameter* before it is applied to the population - and a rollback path, which means the entitlement must be a computed, re-runnable artefact rather than a posted journal entry.

### 10.6 A depository outage with no participant playbook

**Symptom.** The CSD is unavailable during the settlement window. Participants do not know which of their instructions were processed, whether the cut-off will be extended, whether the failure is a delay or a loss, and whether they should be funding or holding. Each participant improvises; some fund, some do not; the funding market seizes because nobody knows the net position.

**Cause.** The depository's own recovery is a solved problem in principle (deterministic replay, published state) and the participants' recovery is not, because the participant's playbook depends on information only the depository can supply. The gap is communication and state disclosure under uncertainty, not technology.

**Guardrail.** A published outage protocol with defined states (delay, extended window, suspended, resuming), a defined set of messages participants will receive, an explicit statement of what is guaranteed and what is not during each state (for example, whether instructions already accepted will be processed or must be resubmitted), and a rehearsal with participants. The protocol must specify how finality is treated during the outage - which is a legal question, answered by the settlement finality regime, and must be pre-agreed rather than argued during the event.

### 10.7 How the six fit together

The six are not independent. They cluster into three pairs, and the pairing is a useful diagnostic:

- **10.1 and 10.6 are state problems.** Both come down to the same question: after the disruption, can anyone say with confidence what state the system is in, and does that state match what the counterparties were told? A venue that cannot replay and a depository that cannot describe its state during an outage have the same defect at different layers.
- **10.2 and 10.4 are correlation problems.** Both are about demands that arrive together because they are computed from the same prices or the same obligations, and both defeat an analysis that sizes each demand separately. Margin procyclicality is a correlation across members; a cascading fail is a correlation across legs.
- **10.3 and 10.5 are rehearsal problems.** Both are processes that are documented, correct on paper, and untested where the failure actually occurs - in the auction, in the parameter, in the notice period. Both are cheap to test and impossible to fix during the event.

If only one of the six can be mitigated this year, the order of return is: test the waterfall (10.3), then make the engine replayable (10.1), then build the joint funding projection (10.4). The others are important, but those three are the ones whose absence converts a contained event into a systemic one.

---

## 11. Worked Example: Cymbal Bank as Clearing Member

### 11.1 The setup, and the disclaimer

Cymbal Bank is a fictional institution used here to make one design decision concrete. It is a clearing member of a CCP for listed derivatives and a settlement participant in a CSD for the underlying cash equities. Nothing in this section is a claim about any real institution's systems, and no other institution is named: where a counterparty or a provider is needed, it is called "the CCP", "the CSD", "the central bank" or "the parent".

The scenario is a bank that has just been through a T+1 readiness review. The review was run from the participant's side, using the checklist the supervisor published (section 6.2), and it concluded that the bank's settlement processing was ready: affirmations were electronic, the allocation engine was intraday, and the fail rate was low. The one open item was collateral and liquidity, and the participant-side analysis sized it as follows: take the largest observed end-of-day variation margin call, add a buffer, hold that much High Quality Liquid Assets, and treat the intraday requirement as a fraction of the end-of-day number. That analysis is coherent, defensible, and wrong - not because the arithmetic is wrong but because it models one institution's demand in isolation.

### 11.2 What the operator view added

Looking at the same day from the CCP's and the CSD's side changes three facts.

**Fact one: the CCP's call is not an end-of-day event.** A CCP has the authority and the operational capacity to make intraday margin calls, both scheduled and unscheduled (✅ PFMI, section 3.4, text preceding 3.4.17, CPMI-IOSCO, 16 April 2012). The scheduled call lands during the trading day, computed from prices at that moment; an unscheduled call can follow a price move with only the notice the rulebook allows. The bank's end-of-day number is therefore not the peak of the requirement - it is one point on a curve that the bank does not control and cannot see in advance.

**Fact two: the CSD's DVP leg needs the same cash, in the same window.** Settlement at the CSD is a sequence of cycles with currency cut-offs. To receive securities, the bank must have funds at the settlement bank before the relevant cycle. That funding obligation is not netted against the CCP's margin call, does not share a cut-off with it, and cannot be satisfied with the collateral the CCP accepts - the CSD needs cash (or a cash-equivalent the settlement bank accepts), while the CCP accepts a broader collateral pool against haircuts.

**Fact three: the two demands are correlated, and both are driven by the same prices.** A price move that raises the CCP's intraday requirement also changes the value of the positions the bank is settling, and often coincides with clients drawing on their own liquidity. The demands are not independent risks that can be sized separately and added; they peak together, which means the buffer must cover the joint peak, and the joint peak is larger than either individual peak but smaller than their sum. Sizing on the sum wastes capital; sizing on either one alone is a funding failure waiting for a volatile day.

### 11.3 The decision

The design decision that changes is not "hold more collateral". It is a change in the *shape* of the liquidity position, and it has four components:

- **A pre-positioned collateral buffer at the CCP, not at the bank.** Collateral that is eligible, unencumbered, and already deposited at the CCP can satisfy an intraday call without a payment moving. Collateral held at the bank, however liquid, must travel, and travel takes time the intraday window may not have. The buffer is sized to the modelled intraday peak, not the end-of-day requirement.
- **A committed intraday credit line for the settlement leg.** The CSD leg needs cash at a specific moment; an intraday facility sized to the largest single cycle's funding requirement covers it without holding a standing cash balance all day. The line must be contractually committed, tested, and drawn at least once outside an emergency so that the operational path is proven.
- **A joint projection, computed intraday, replacing two separate ones.** The bank needs a single forward view of the day: expected variation margin at each call, expected settlement funding at each CSD cycle, expected client flows, and the collateral inventory available to each destination with its eligibility mapped. That projection is the actual deliverable of the project - the buffers are its output, and they must be recomputed as the day moves rather than set annually.
- **A collateral transformation path.** Because the CCP's eligible collateral and the CSD's settlement cash are not the same asset, the bank needs a standing route to convert one into the other intraday - repo, a securities lending desk with same-day recall capability, or a central bank facility where one is available. The route is a contractual and operational arrangement, not a system feature, and its capacity is part of the buffer.

### 11.4 Why the participant view could not have found this

The participant-side review asked: how large is my margin call, and do I have the money? Both questions are about the bank's own position, and both are answerable inside the bank's own data. Neither question reveals that the CCP's call and the CSD's funding leg are two claims on one pool of cash arriving in the same window, because that fact lives in the *other* institutions' calendars and collateral rules - in the CCP's call schedule and eligibility schedule, and in the CSD's cycle timetable and cut-offs.

The participant view also could not see the correlation, because correlation between the two demands is a property of the market infrastructure's design (both are computed from the same prices, on the same day) rather than of the bank's portfolio. A model built from the bank's own history would show two requirements that vary independently, because in the bank's history they were satisfied on different days by different balances. The operator view is what makes them the same event.

### 11.5 The thesis, applied

The decision changed because the bank stopped asking "how much margin will I be called for" and started asking "which promise is being kept, by whom, at what time, and with whose cash". That is the operator's question. The CCP's intraday call is a promise that the exposure is pre-funded before the close-out; the CSD's DVP leg is a promise that the seller is not exposed to the buyer; and both promises are discharged from the same balance sheet, on the same clock, with collateral that is not fungible between them.

The bank did not need a bigger end-of-day balance. It needed to understand the failure promises it was buying, and to shape its liquidity to match their timing. That is the whole argument of this guide, in one decision.

---

## 12. Anti-Patterns in Market Infrastructure

Nine patterns, each of which looks reasonable from the inside and fails in a specific way. The "why it looks reasonable" column matters as much as the failure: anti-patterns in this domain are almost always locally rational.

| Anti-pattern | Why it looks reasonable | Failure it causes |
|---|---|---|
| **Treating the sequencer as an implementation detail** | It is a component in the middle of the stack; the matching algorithm is where the intellectual work is, so the ordering mechanism is left to the platform team | The venue cannot prove what happened first. Disputes are resolved on gateway timestamps, fairness claims become unanswerable, and the recovery path has no authoritative log to replay |
| **Modelling margin without the procyclicality question** | The model's job is to measure risk; measuring it accurately is the requirement, and the model does that | The requirement rises for everyone at once in a stress event, forcing deleveraging into the move and turning a risk measure into an amplifier |
| **A waterfall documented but never rehearsed** | The waterfall is a legal instrument; it is drafted by legal, approved by the regulator, and published. Testing it looks like testing a contract | When it is used, the operational steps fail - notices, permissions, auction access, cross-border calls - and the layers are consumed in a disorderly sequence |
| **DVP asserted without checking the legal moment of finality** | Delivery and payment are both instructed in the same system on the same day, so it is DVP | The system is model 2 in disguise: securities move gross during the cycle, funds net at the end, and the seller carries principal risk it does not know it has |
| **Queue position treated as the fairness property** | Participants complain about queue position, so queue position must be what fairness means | The venue optimises allocation order while leaving the actual rule - the priority rule and the sequencing - opaque, and the fairness argument is conducted on the wrong variable |
| **Corporate actions handled per market with no shared clock** | Each market has its own conventions, so each market gets its own process, which is respectful of local practice | The same security held across two markets gets two different entitlement dates; the bank's aggregate position is right in each market and wrong in total, and no single process owns the discrepancy |
| **"Zero downtime" claimed for a system whose requirement is deterministic recovery** | Downtime is the metric everyone understands, and a system that never goes down is obviously better | Failover is optimised over recoverability: the standby takes over fast, into a state that differs from what participants were told, and the reconciliation takes longer than the outage would have |
| **Latency marketed instead of tail latency measured** | Speed sells, and the median is the number that is easy to produce and easy to compare | The distribution's tail is where the failures are: a p99.9 stall delivers stale data and stale acknowledgements, and no participant has been told what the tail looks like |
| **Collateral eligibility assumed to be fungible across institutions** | Collateral is collateral; a liquid asset is a liquid asset | The CCP's haircut schedule and the CSD's settlement cash requirement are different, so the buffer that satisfies one does not satisfy the other, and the shortfall appears at the worst moment (section 11) |

The common thread: each anti-pattern substitutes a locally measurable quantity for the property that actually matters. Queue position for fairness, median latency for reliability, downtime for recoverability, the margin number for the risk it represents, the contract for the process. In each case the substitute is easier to measure, easier to report, and wrong.

### 12.2 The pattern behind the patterns

Three of the nine deserve a paragraph, because they are the ones that survive review boards.

**The sequencer one is a governance failure, not a technical one.** No platform team deliberately sets out to make a venue unauditable. What happens is that the sequencer is procured as part of a matching engine product, the ordering mechanism is treated as vendor-internal, and the venue's own specification stops at "the engine receives messages". By the time a dispute arises, the venue owns an obligation it cannot evidence. The corrective is contractual and architectural at once: the ordered input log, its retention period and its replayability belong in the venue's own requirements, and the vendor's design must be shown to satisfy them.

**The DVP-assertion one is the most consequential, because it is invisible.** Every participant in a model 2 system believes it is in a DVP system - the messages say delivery and payment, the reports show both legs, and the settlement date is the same. The defect is the time between the two legs, and it is only observable in the system's cycle design: when securities are released relative to when funds are released, and what claim a seller has in the interval. The question to ask is never "is this DVP?" but "at what moment does the seller stop being exposed, and to whom?"

**The zero-downtime one is a requirements error that looks like ambition.** A requirement of "no downtime" is unfalsifiable in design and unachievable in practice, so it degenerates into a best-effort objective and the achievable requirement - deterministic recovery to a state the participants recognise - is never written down. The tell is a design review that can discuss availability figures for an hour and cannot answer "what sequence number will participants see after failover, and how will they verify their positions against it?"

---
## 13. State of Practice

The purpose of this section is to separate three kinds of statement that are routinely mixed together in this subject: what is settled and can be designed against, what is genuinely contested and must be handled with judgement, and what is promotional and should be treated as marketing until a mechanism is shown.

### 13.1 What is settled

- **Novation and multilateral netting.** The legal mechanism, its effect on the counterparty link, and its dependence on netting enforceability are settled. No serious practitioner disputes how a CCP interposes itself or why netting requires it. The rulebook technique varies (classic novation versus open offer) but the effect does not.
- **The DVP models.** The three-model taxonomy from the 1992 BIS report is still the working vocabulary, and the principal-risk consequence of gross securities against net funds is settled: it is model 2, and the seller is exposed. Systems that claim to be DVP and behave like model 2 are a known and recurring category.
- **The existence, structure and disclosure of the default waterfall.** PFMI requires that the rules expressly set the waterfall out (paragraph 3.4.24) and that the infrastructure disclose its rules and procedures; CCPs publish their waterfall order and their layer sizes. What the layers are, and in what order they are consumed, is public information for any authorised CCP.
- **The US T+1 migration.** The date, the instrument, the adopting release and the operational consequences are all documented by the supervisor that examines against them. There is nothing speculative about the US position.
- **The mechanics of the settlement cycle as a risk window and a processing buffer.** That shortening it compresses exception handling rather than happy-path processing is not a contested claim; it is what the supervisors' own checklists describe.
- **That a matching engine must be deterministic to be recoverable.** This is settled engineering, even where it is not practised.

### 13.2 What is contested

- **The calibration of skin in the game.** There is no published standard for how large the CCP's own layer should be relative to the default fund, and the competing objectives - credible alignment of the operator's incentive, versus a layer the shareholders will actually fund - are not reconcilable by formula. Different CCPs publish very different ratios between their own contribution and the mutualised pool.
- **The procyclicality of margin models.** That margin responds to volatility is intended; whether the response should be damped, and by how much, is a live supervisory debate with no settled answer. Every damping tool reduces the CCP's protection in the scenario where it is needed most.
- **Whether T+1 reduces risk or relocates it.** The risk-window argument is sound: a shorter cycle means less time exposed. The relocation argument is also sound: the same obligations must be discharged with less slack, so the operational and funding risk per unit of exposure rises, and the population that can participate without intraday liquidity shrinks. The net effect is empirical and jurisdiction-specific, and the EU and UK decisions to adopt T+1 do not settle it.
- **The resolution regime for CCPs.** Resolution regimes for CCPs are newer and less tested than those for banks. The cross-border case - one CCP, many jurisdictions, many resolution authorities - has no settled answer, and the treatment of mutualised resources in a resolution is not harmonised.
- **Tokenised settlement.** Whether distributed-ledger settlement of cash and securities will replace, complement or remain marginal to existing CSD infrastructure is genuinely open. The technology can demonstrably move a token; whether it can deliver legal finality, netting enforceability and central-bank-money settlement at the same time is the open question, and it is a legal question more than a technical one. ⚠ Assessment, not a sourced finding.

### 13.3 What is vendor or promotional

Four categories of claim should be read with the mechanism test applied:

- **Latency claims.** A latency number without a stated distribution, a stated measurement point and a stated load is not a claim about anything. "Sub-microsecond" measured at the switch, on an empty book, with one session, tells you nothing about the tail under a full fan-out. The operator-side question is always: what is the p99.9, measured where, under what load, and what happens to it when the publisher backs up?
- **"Real-time" clearing.** Clearing is a set of legal and risk processes with cut-offs; making the messaging real-time does not make the novation, the netting or the margin call real-time. A claim of real-time clearing usually means real-time *notification*, which is a different and much easier thing.
- **DLT settlement claims.** A demonstration that a tokenised bond can be issued and transferred on a distributed ledger is not a demonstration that the transfer is final against an insolvent estate, that the cash leg is in central-bank money, or that the netting set is enforceable. Those are the properties that make settlement infrastructure, and they are not properties of the ledger.
- **"Zero downtime" and "active-active".** For a matching engine, the requirement is deterministic recovery, not the absence of downtime. A vendor that markets availability without a replay story is marketing the wrong property. Section 12's anti-pattern list is the test.

The general rule: a claim about a mechanism can be checked against a rulebook, a disclosure or a specification. A claim about a magnitude cannot be checked at all unless the measurement conditions are published. Prefer mechanism.

### 13.4 The effect on design

The practical consequence of sorting claims this way is that the settled material is where the architecture comes from, and the contested material is where the judgement sits. A design that is built only on settled mechanics will be correct and will still be wrong about, say, the collateral buffer, because the buffer's size depends on a contested question (how much procyclicality to assume). The right posture is: build to the settled mechanism, parameterise the contested judgement, and make the parameter visible to the people who own the risk - rather than embedding an assumption in a model and calling it a requirement.

### 13.5 What the operators themselves disagree about

A useful signal: the disagreements that infrastructure operators have with each other in public are almost never about the mechanisms. They are about calibration and about scope, and both are places where an integrator should not assume that one operator's answer transfers to another.

- **The size of the operator's own layer.** Different CCPs publish markedly different relationships between their own contribution and the mutualised pool, and each can defend its own. Since no standard fixes the ratio, the ratio is a governance choice dressed as a risk parameter, and it should be read as evidence about the operator, not about the risk.
- **Whether a CCP should clear more or fewer products.** Portfolio margining across correlated products reduces members' margin; it also increases the concentration of risk in one clearing service and makes the cross-default and cross-netting questions harder. Operators disagree because the trade-off is real.
- **The treatment of client positions in a default.** Porting client business to another member is the stated objective; how much of the defaulter's client margin can be moved with it, and how the residual is treated, differs by model and is the subject of continuing argument.
- **Whether the settlement cycle should shorten further.** T+1 is legislated in three major markets; T+0 is discussed. Operators are noticeably more cautious in public than participants, which is unsurprising: the operator carries the processing obligation and the participant carries the funding obligation, and the two have different tolerances.

---

## 14. The Claims Audit

Every substantive factual assertion in this guide, with its status and its source. ✅ = verified this pass against a primary or stable public source. ⚠ = flagged: professional knowledge, or single-sourced, or not re-verified. ❌ = checked and unavailable or wrong. Where a row says "read from cache this pass", the document was retrieved and read in this session, not recalled.

| # | Claim | Status | Source and date |
|---|---|---|---|
| 1 | PFMI describes a typical waterfall as including the defaulter's initial margin, the defaulter's default fund contribution, a specified portion of the CCP's own funds, and other participants' contributions | ✅ | CPMI-IOSCO, "Principles for financial market infrastructures", 16 April 2012, paragraph 3.4.17, quoted verbatim; full text read from cache this pass |
| 2 | PFMI requires that the rules of an FMI "expressly set out the waterfall", including when specific resources may be used, and excludes resources needed for operating expenses from being counted as available | ✅ | Same document, paragraph 3.4.24, quoted verbatim; read from cache this pass |
| 3 | PFMI requires a CCP to cover current and potential future exposures fully with a high degree of confidence using margin and other prefunded resources, and for systemically important or complex CCPs to cover the default of the two largest participants and their affiliates (cover-two), with cover-one for others | ✅ | Same document, Principle 4 (Credit risk), key considerations, quoted verbatim; read from cache this pass |
| 4 | PFMI states a CCP should have the authority and operational capacity to make intraday margin calls, scheduled and unscheduled | ✅ | Same document, section 3.4, text preceding paragraph 3.4.17; read from cache this pass |
| 5 | PFMI requires recovery and orderly wind-down planning, and provision of information for resolution planning | ✅ | Same document, Principle 3, key consideration 4, quoted verbatim; read from cache this pass |
| 6 | PFMI operates with a companion disclosure framework and assessment methodology | ✅ | Same document, paragraph 1.35, which names the "CPSS-IOSCO Disclosure framework for financial market infrastructures" and the "Assessment methodology"; read from cache this pass |
| 7 | Eurex Clearing's waterfall order is: defaulter's margin and Default Fund contribution; the CCP's own Skin in the Game; non-defaulted members' contributions plus Second Skin in the Game; assessments simultaneous with a Further Dedicated Amount on a pro-rata basis; remaining equity; a parental letter of comfort | ✅ | Eurex Clearing, "Default Waterfall" and "Contribution of Eurex Clearing" pages, fetched 26 October 2026 |
| 8 | Eurex Clearing's published layer sizes are EUR 143 million (SITG), EUR 57 million (SSITG), maximum EUR 300 million (Further Dedicated Amount), EUR 600 million (remaining equity), maximum EUR 600 million (letter of comfort), with no third-party rights under the letter of comfort | ✅ | Same pages, fetched 26 October 2026. These are the CCP's own published figures on that date, not an industry aggregate and not a computation of this guide |
| 9 | The standard waterfall applies to one clearing model and a different waterfall (indemnifying clearing agent) applies to another | ✅ | Same pages, fetched 26 October 2026 |
| 10 | The US standard settlement cycle moved to T+1 on 28 May 2024 under amendments to Rule 15c6-1 | ✅ | SEC Division of Examinations, Risk Alert "Shortening the Securities Transaction Settlement Cycle", 27 March 2024, fetched this session |
| 11 | The adopting release is Exchange Act Release No. 96930, 15 February 2023, 88 FR 13872, 6 March 2023 | ✅ | Same Risk Alert, citing the release; fetched this session |
| 12 | 28 May 2024 was also the compliance date for institutional-trade processing rules (broker-dealers, certain clearing agencies including central matching service providers) and adviser recordkeeping amendments | ✅ | Same Risk Alert; fetched this session |
| 13 | T+1 shortens Rule 204 close-out timeframes, the Rule 10b-10 confirmation window, and the Rule 15c3-3(m) possession period, and the staff's examination focus includes ACA, other time zones, end-of-day trades, stock lending and recalls, and funding for T+2 international and ETF transactions | ✅ | Same Risk Alert; fetched this session |
| 14 | The EU will switch to T+1 on 11 October 2027 | ✅ | ESMA press release "One year to go for T+1 settlement", 16 October 2026, ref ESMA71-545613100-2889, fetched this session |
| 15 | The first EU deadline is 7 December 2026, for allocations and confirmations, including mandatory electronic standardised channels, international messaging standards and machine-readable methods except during temporary technical disruptions | ✅ | Same ESMA press release, 16 October 2026, fetched this session |
| 16 | The EU regulatory technical standards were published as OJ L_202601536, and ESMA considers the framework complete | ✅ (citation only) | Same ESMA press release, which links the RTS at that OJ identifier; the RTS text itself could not be opened (see row 26) |
| 17 | ESMA recommended 11 October 2027, with migration simultaneous across instruments, in a final report dated 18 November 2024 (ref ESMA74-2119945925-1969), noting CSDR amendment was needed for legal certainty | ✅ | ESMA final report of 18 November 2024, as recorded in the verified evidence base for this guide; not re-fetched in this session |
| 18 | The UK will mandate T+1 from 11 October 2027, and has required T+2 since 2014 | ✅ | HM Treasury, "Accelerated Settlement (T+1)", policy note "Mandating T+1 settlement in the UK", first published 6 February 2025, last updated 20 November 2025, read from cache this pass |
| 19 | The UK mechanism amends the CSD regulation's intended settlement date; the draft instrument published 20 November 2025 is "The Central Securities Depositories (Amendment) (Intended Settlement Date) Regulations 2026" | ✅ | Same HMT policy note, read from cache this pass |
| 20 | The 1992 BIS report sets out three DVP models, with model 1 able to eliminate principal risk and model 2 having "the potential to expose sellers of securities to substantial principal risk" | ✅ | BIS, Committee on Payment and Settlement Systems, "Delivery versus payment in securities settlement systems", Basle, September 1992, pp. 17-21, quoted verbatim; read from cache this pass |
| 21 | The FSB's review of the March 2020 turmoil states that CCPs "remained resilient despite market turbulence, though margin calls may have been larger than expected in some cases" | ✅ | FSB, "Holistic Review of the March Market Turmoil", 17 November 2020, quoted verbatim; fetched this session |
| 22 | No aggregate margin-call figure, percentage or "billions" figure is printed in this guide | ✅ (deliberate omission) | No crisis magnitude was verified at a primary source in this pass; the FSB's own wording is unquantified, and the mechanism (VM as a simultaneous, volatility-scaled cash call) is the finding |
| 23 | Corporate-action mechanics: entitlement by position snapshot at record date; ex date separated from record date by the settlement cycle; elections with deadlines and defaults; fractional entitlements rounded or cash-settled; withholding depending on residence and documentation in the chain; proxy instructions flowing down the chain | ⚠ | Professional knowledge, not verified against a market guide or CSD rulebook this pass |
| 24 | CCP recovery tools include assessments, variation margin gains haircutting, partial tear-up and auction/porting | ⚠ | Professional knowledge; PFMI requires recovery planning (row 5) but the specific tools were not verified against a CCP rulebook this pass |
| 25 | The legal technique for interposition varies (classic novation versus open offer / registered contract) | ⚠ | Professional knowledge; rulebooks not read this pass |
| 26 | EMIR, MiFID II/MiFIR, CSDR and the Settlement Finality Directive are described functionally, with no article numbers | ⚠ | Descriptions from professional knowledge; eur-lex.europa.eu could not be reached this pass, so no legal text was read |
| 27 | CSDR settlement discipline consists of cash penalties for fails and a buy-in framework, with the buy-in regime's application suspended for a period by delegated regulation | ⚠ | Professional knowledge; no penalty figure is printed and no article is cited |
| 28 | Auction prices are used as reference prices for fund NAVs, collateral valuation, index events, contract settlement and execution benchmarks | ⚠ | Structural industry knowledge; no individual use case was sourced to a rulebook or provider methodology this pass |
| 29 | Price bands and circuit breakers are implemented as per-instrument state machines with venue-specific parameters | ⚠ | Professional knowledge; no venue's parameters are asserted |
| 30 | Anti-procyclical margin tools include margin floors, through-the-cycle calibration, limits on the rate of increase, and policies for margin release | ⚠ | Professional knowledge; no regulatory text read this pass |
| 31 | A CCP cannot be allowed to fail mid-auction, so a public backstop effectively sits behind the last private layer | ⚠ | Analytical judgement, not a sourced finding; presented as an argument in section 9.3, not as a fact about any institution |
| 32 | A matching engine must be a deterministic replicated state machine to be recoverable and auditable | ✅ (engineering) | Standard distributed-systems reasoning, applied; no external source required. Presented as an engineering claim, not a regulatory one |

---

## 15. What Could Not Be Verified, Glossary and Cross-References

### 15.1 What could not be verified

Recorded plainly, because a guide that hides its gaps is worse than one that has them.

- **The EU legal texts.** eur-lex.europa.eu could not be reached on any fetch path attempted in this pass - the HTML, the ELI route, and the OJ identifier cited by ESMA (L_202601536) all failed, the last with an internal server error from the retrieval tool. Consequence: EMIR, CSDR, MiFID II/MiFIR and the Settlement Finality Directive are described functionally in section 8.2, with no article numbers, and every claim about them is marked ⚠. What is lost is the precision an architect eventually needs - the exact obligation, the exact timing requirement, the exact scope - and it must be recovered from the legal text before any design decision is final.
- **The EU regulatory technical standards and the revised CSDR guidelines.** ESMA's press release of 16 October 2026 was readable and is quoted, and it links the RTS and the final guidelines report, but neither underlying document was retrieved. The 7 December 2026 deadline and its content are therefore verified at the level of the supervisor's announcement, not at the level of the instrument that creates the obligation.
- **sec.gov HTML pages and fca.org.uk.** sec.gov PDFs were retrievable (the Risk Alert used throughout was read in full), but sec.gov HTML pages and fca.org.uk returned access failures. Consequence: the SEC's adopting release was cited through the staff Risk Alert that quotes it rather than read directly, and no FCA material is used at all.
- **Search as a discovery tool.** `web_search` returned empty results for several quoted and phrase-based queries in this pass, and usable results for others. An empty result was treated as a tool limitation, not as evidence that no material exists. Where a fact mattered and search failed, the primary URL was fetched directly instead.
- **CCP rulebooks.** No CCP rulebook was read in this pass. The waterfall order and layer sizes come from a CCP's published disclosure pages, which is a reliable source for the order and the sizes, but the recovery tools (VMGH, partial tear-up), the notice mechanics and the segregation models are described from professional knowledge and are flagged accordingly.
- **CSD rulebooks and market practice guides.** The corporate-action mechanics in section 5.4 and the buy-in and penalty mechanics in section 6.5 are described generically. Market-specific conventions - record-date offsets, election deadlines, withholding rates, the buy-in suspension in force - were not verified and must be taken from the relevant CSD's own guide before being implemented.

### 15.2 Glossary

| Term | Definition |
|---|---|
| **Novation** | The legal replacement of a bilateral contract by two contracts, each with the CCP, so that the CCP becomes buyer to every seller and seller to every buyer |
| **Multilateral netting** | The offsetting of many obligations across a membership into one net position per instrument, account and settlement date; possible only because novation makes the CCP the common counterparty |
| **Initial margin** | A forward-looking, model-based, upfront deposit estimating the cost of closing out a position over a liquidation horizon, at a confidence level |
| **Variation margin** | A payment that settles the change in the mark-to-market value of a position since the last call, so that the exposure does not accumulate |
| **Default fund** | A prefunded pool of members' contributions, held by the CCP, used after the defaulter's own resources and the CCP's own layer |
| **Default waterfall** | The prescribed order in which resources are applied to losses from a participant default, set out in the CCP's rules |
| **Assessment** | A call on surviving members for additional default fund contributions beyond their prefunded amount, used after the prefunded layers are exhausted |
| **Skin in the game** | The CCP's own funds placed in the waterfall, ahead of mutualised member contributions, intended to align the operator's incentive with prudent risk management |
| **DVP (delivery versus payment)** | A settlement mechanism linking the transfer of securities to the transfer of funds so that delivery occurs if and only if payment occurs |
| **CSD (central securities depository)** | The institution holding the definitive record of ownership of securities and operating the settlement system for them |
| **CCP (central counterparty)** | The entity that interposes itself between the parties to a trade, becoming buyer to every seller and seller to every buyer |
| **PFMI** | "Principles for financial market infrastructures", the CPMI-IOSCO standard of 16 April 2012 for payment systems, CSDs, securities settlement systems, CCPs and trade repositories |
| **CSDR** | The EU regulation governing central securities depositories, including authorisation, settlement discipline and the settlement cycle |
| **T+1** | A settlement cycle in which the intended settlement date is one business day after the trade date |
| **Immobilisation** | Holding physical securities certificates in a depository so that transfers can be effected by book entry rather than by moving the certificates |
| **Dematerialisation** | The elimination of the physical certificate, so that an electronic register is the authoritative record of the security |
| **Settlement finality** | The point at which a transfer order becomes irrevocable and unconditional, and is protected from unwind on the insolvency of a participant |
| **Buy-in** | A process by which a failed delivery is closed out by arranging a replacement purchase at the failing party's expense |
| **Fail** | A failure to deliver securities, or to pay, on the intended settlement date |

### 15.3 Cross-references

- The bank's own view of market infrastructure, its post-trade chain and its back-office operations: [Capital Markets Architecture Guide](capital_markets_architecture_guide.md), sections 4 and 6.
- The participant's view of the order lifecycle, pre-trade risk, execution, venue connectivity, colocation from the client side and the low-latency stack: [Trading System Software Architecture Guide](../technology/trading_system_software_architecture_guide.md), sections 2, 4, 7 and 9.
- Derivatives order plumbing - implied orders, symbol mapping, sequence continuity, multi-price precision: [Financial Trading Order Infrastructure](financial_trading_order_infrastructure.md).
- This guide, for a reader arriving from either sibling: [Exchange, Clearing and Settlement Technology - The Operator's View](exchange_clearing_settlement_technology_guide.md).

---

## 16. The Closing Summary

Sixteen sections, one argument. It is worth restating in the order the argument actually runs, because the chain of substitutions is the content.

**The chain is legal, not physical.** Execution creates an obligation, novation replaces the debtor, netting collapses the obligations, margin prices the residual, DVP discharges it, and book entry records the result. Each step transfers something other than value: a debtor, an exposure, a timing, a title. Every step has a legal moment, and most operational failures are failures to align those moments (section 2).

**The venue's product is its rulebook, and its matching engine is a determinism problem.** Price-time versus pro-rata is a fairness choice; the auction is a risk-management device because the close is a reference price for funds, collateral and indices; and the engine must be a replicated state machine whose output can be reproduced from its input sequence, or the venue cannot prove that it applied its own rules (sections 3 and 7).

**The CCP is the analytical centre.** Novation severs the counterparty link and concentrates the exposure; netting is a legal consequence of that severance, not an arithmetic convenience; initial margin pre-funds a hypothetical close-out while variation margin settles an actual price move, and the distinction is purpose, not size; and the waterfall is the written order in which the loss is allocated - the defaulter first, the operator second, the survivors third, the operator again alongside them, and the equity last (section 4).

**PFMI requires the waterfall to be express and disclosed; it does not prescribe one universal order.** That nuance is the honest centre of the regulatory account, and it is why the rulebook, not the standard, is the requirements document a system must implement (sections 4.5 and 8.3).

**The CSD is an event processor driven by legal dates.** Immobilisation and dematerialisation exist because certificates cannot be delivered at the rate the market trades; the nominee gap means entitlement flows through a chain rather than sitting with a holder; DVP is a property of simultaneity, and model 2 is the counter-example that shows how a system can be called DVP and still expose the seller to principal risk (section 5).

**T+1 compresses the exception-handling buffer, not the happy path.** The US moved on 28 May 2024 under Release 34-96930; the EU and the UK legislate for 11 October 2027, with the EU's messaging and affirmation requirements binding from 7 December 2026. The consequences are in the funding and affirmation windows - securities lending recalls, FX legs, ETF baskets, international securities that still settle T+2 - and the fail arrives sooner (section 6).

**The systemic question is an asymmetry, not a verdict.** Central clearing replaces a diffuse web of bilateral exposures with a concentrated exposure to one institution, whose failure would be systemic by construction. A CCP cannot be allowed to fail mid-auction, which means a public backstop sits behind the last private layer - unprinted, unquantified, and decisive for the incentives of every layer above it (section 9).

**The operator view changes decisions.** Cymbal Bank's collateral buffer was not too small because the bank had mis-measured its margin; it was mis-shaped because the CCP's intraday call and the CSD's funding leg are two claims on one pool of cash in one window, correlated by the same prices. The fix was pre-positioned eligible collateral, a committed intraday line, a joint projection and a transformation path - not a bigger end-of-day balance (section 11).

What the guide cannot do is close the gap it identifies. The mechanisms are documented, the waterfall is published, the dates are legislated, and the one thing that holds the structure together in the tail - the promise that the clearing service continues - is not in any of those documents. That is not a defect of the guide. It is the shape of the subject.

A market is not a place where trades happen; it is a set of promises about what happens when one side fails.
