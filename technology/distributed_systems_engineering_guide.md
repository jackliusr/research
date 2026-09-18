# Distributed Systems Engineering — the partial failure

**The Discipline Layer — the Failure Model and the Assumption Ledger, Time and Causality, the Impossibility Results and Their Precise Conditions, the Consistency Spectrum and Its Conflated Pairs, the Consensus Families and What Each Costs, Replication and Partitioning as Choices, Failure Detection, Leases and Fencing, Delivery Semantics and Exactly-Once Effects, Distributed Transactions and Their Alternatives, the Engineering Judgement, a Cymbal Bank Worked Example, the Anti-Patterns, the Claims Audit, and What Could Not Be Verified**

> **Author:** Jack Liu Shurui — Solution Architect
> **Byline:** Jack Liu Shurui, Solution Architect
> **Series:** Technology / Distributed Systems — the discipline of *deciding, explicitly, which hard questions your system answers and which it silently defers*.
> **Audience:** architects and senior engineers who are choosing consistency, replication and coordination strategies; reviewers who have to say whether a design is sound; and engineers who have read a product guide, believed the word "strong", and shipped.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Integrity convention:** ✅ = verified this pass against the cited primary source (publisher record, author page, or product documentation); ⚠ = flagged — asserted, plausible, or dependent on a non-primary source; ❌ = rejected (asserted somewhere, found false or unsupported). Used in every table that makes a factual claim.
> **Claim classes:** **PROVEN RESULT** (a theorem with a published proof), **ENGINEERING CONVENTION** (a widely used practice with no theorem behind it — true because everyone agreed), **VENDOR MARKETING** (a product's claim about itself; treat as a lead, not a fact).
> **Illustrative figures:** Every number attributed to **Cymbal Bank** (a **fictional** institution, and the only bank persona used in this repository) is **explicitly illustrative and fictional** — a worked shape for a decision, not a benchmark, not a survey, and not a claim about any real institution.
> **Verification method, stated up front:** This pass had **live web access through a self-hosted Firecrawl backend**. `web_search` returned **empty result sets for every query attempted** (three queries, all empty — recorded in §14 as empty searches, not as negative facts). Every verification below therefore comes from **`web_extract` against a primary URL**: Lamport's own publication record at `lamport.azurewebsites.net/pubs/pubs.html` (extracted 2026-09-18), the USENIX conference records for ATC 2014 and OSDI 1999, the paper PDFs themselves (FLP, Gilbert–Lynch, PACELC, HAT, Spanner, Kleppmann's locking post), and the ACM Digital Library — which **rejected scraping on every attempt** (five URLs, all HTTP 500 from the extractor). Where the ACM DL blocked, I fell back to the **author's own publication list or the paper's own PDF**, and I say which in §14. Where I could not confirm something, it is in §15.

**How this guide is organised.** §1 draws the boundary — what the discipline is, why a product-by-product education leaves gaps, and what this guide is *not* — and gives the decoder. §2 is the failure model and the assumption ledger; it should leave you distrusting your own architecture diagram. §3 is time, ordering and causality, the one area where this repository measured **zero** prior coverage. §4 is the impossibility results, stated at their precise conditions rather than at their folklore strength. §5 is the consistency spectrum and the pairs people confuse. §6 is consensus — the families and what each one costs. §7 is replication and partitioning as a decision framework. §8 is failure detection, membership, leases and fencing; it is the section most likely to catch a live bug in your system. §9 is delivery semantics, idempotency and exactly-once. §10 is distributed transactions, condensed and cross-referenced. §11 is the engineering judgement. §12 is the Cymbal Bank worked example. §13 is the anti-patterns. §14 is the claims audit. §15 is what could not be verified. §16 is the glossary, §17 the cross-references, §18 the closing.

## Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder) — 1.1 what the discipline is · 1.2 why product-by-product education leaves gaps · 1.3 what this guide is not · 1.4 the cross-reference boundary · 1.5 the decoder
2. [The Failure Model](#2-the-failure-model) — 2.1 the taxonomy · 2.2 why the model determines the algorithm · 2.3 partial failure as the defining condition · 2.4 ambiguous failure · 2.5 the assumption ledger
3. [Time, Ordering and Causality](#3-time-ordering-and-causality) — 3.1 why physical clocks cannot order events · 3.2 skew and drift · 3.3 monotonic vs wall · 3.4 happens-before · 3.5 logical and vector clocks · 3.6 causal delivery · 3.7 the worked indeterminacy
4. [The Impossibility Results and What They Actually Mean](#4-the-impossibility-results-and-what-they-actually-mean) — 4.1 FLP · 4.2 CAP and its misreading · 4.3 the Brewer self-correction · 4.4 PACELC · 4.5 the escape hatches and their prices
5. [The Consistency Spectrum](#5-the-consistency-spectrum) — 5.1 strongest to weakest · 5.2 the commonly conflated pairs · 5.3 the isolation-anomaly catalogue · 5.4 which level does the business process actually need
6. [Consensus](#6-consensus) — 6.1 the classical family · 6.2 Raft and why it displaced it · 6.3 the Byzantine family · 6.4 leader election and log replication · 6.5 membership change · 6.6 operational realities
7. [Replication and Partitioning](#7-replication-and-partitioning) — 7.1 the three topologies · 7.2 synchronous vs asynchronous · 7.3 partitioning strategies · 7.4 rebalancing and hot partitions · 7.5 secondary indexes · 7.6 the cross-partition operation that ruins the plan
8. [Failure Detection, Membership and Time](#8-failure-detection-membership-and-time) — 8.1 heartbeats and timeouts · 8.2 accrual failure detectors · 8.3 gossip · 8.4 leases · 8.5 fencing and two generations of leader
9. [Delivery Semantics, Idempotency and Exactly-Once](#9-delivery-semantics-idempotency-and-exactly-once) — 9.1 the three modes and what each promises · 9.2 why exactly-once delivery is not achievable end to end · 9.3 what vendors actually claim, quoted · 9.4 the mechanics of exactly-once effects · 9.5 the conclusion
10. [Distributed Transactions and Their Alternatives](#10-distributed-transactions-and-their-alternatives) — 10.1 atomic commit · 10.2 three-phase commit · 10.3 sagas and compensation · 10.4 CRDTs · 10.5 the decision rule
11. [The Engineering Judgement](#11-the-engineering-judgement) — 11.1 writing down invariants · 11.2 coordination required vs avoidable · 11.3 the cost model · 11.4 designing for the undetectable · 11.5 the honest admission
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example) — 12.1 the flow · 12.2 the invariants · 12.3 consistency per step · 12.4 replication, partitioning, fencing · 12.5 alternatives and costs · 12.6 the recommendation
13. [The Gotchas and Anti-Patterns](#13-the-gotchas-and-anti-patterns) — nine, each with symptom, cause and guardrail
14. [The Claims Audit](#14-the-claims-audit) — verified / flagged / rejected, with source quality and date
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary](#16-glossary)
17. [Cross-References and Further Reading](#17-cross-references-and-further-reading)
18. [Closing Summary](#18-closing-summary)

## 1. The Overview, the Boundary and the Decoder

### 1.1 What the Discipline Is

Distributed systems engineering is not a body of knowledge about products. It is a small set of **hard questions** that every distributed product must answer, plus the accumulated theory about which combinations of answers are *possible*. The theory is unusually well-settled: most of the load-bearing results were proved between 1978 and 2002 and have not been overturned. The practice is unusually badly settled: the same wrong answers are rediscovered by each generation of engineers, usually in production, usually at 03:00.

The thesis of this guide is that **every product in this repository's distributed cluster is a particular set of answers to the same small set of hard questions, and the job of the discipline is to make those questions visible**. When you can see the questions, a product's documentation stops being a feature list and becomes a set of declarations — some explicit, most silent — about the questions. And a silent answer is the dangerous kind, because nobody ever reviewed it.

The questions, which recur in every section:

1. **What am I assuming about failure?** (§2)
2. **How do I know the order in which things happened?** (§3)
3. **What is provably impossible, so that I stop trying?** (§4)
4. **Which consistency guarantee does this business process actually require?** (§5)
5. **Am I willing to pay coordination to get it?** (§6, §11)
6. **How is state placed, and what happens when the placement must change?** (§7)
7. **How do I know a node is dead, and what stops a zombie from acting?** (§8)
8. **What happens if this message is processed twice? Zero times?** (§9, §10)

### 1.2 Why a Product-by-Product Education Leaves Gaps

An engineer who learns distributed systems by reading product documentation learns a **set of answers without the questions**. The failure mode is specific and reproducible:

- They learn that CockroachDB is "Postgres-compatible and strongly consistent" and that Cassandra is "highly available". They then build a system in which one service writes to CockroachDB and consumes from Cassandra-mediated event streams, and never notice that **the two products answer the ordering question differently** — so the composition has no ordering guarantee at all, even though each component has one.
- They learn that Kafka with `acks=all` "does not lose data". They then build a consumer that reads, calls a payment API, and commits its offset — and lose the payment, or duplicate it, because **`acks=all` answers the broker-durability question, not the end-to-end delivery question** (§9).
- They learn that Redis "supports distributed locks" and that a lock has a TTL. They then use it to protect a resource that must be written at most once, and discover that **a lease without a fencing token is not a mutual-exclusion mechanism** (§8.5).

None of these are product bugs. Every one of them is a **question the team never knew it had answered**. The dedup evidence in this repository makes the gap concrete:

| Measurement (grep -rci over all `.md`, 2026-09-18) | Value | Reading |
| --- | --- | --- |
| Files whose top hit for `distributed system` is the DDIA companion | `technology/ddia_study_companion_guide.md` — 14 hits | The theory is discussed *inside a book companion*. |
| Then | `technology/distributed_auth_guide.md` 11, `technology/deterministic_engineering_guide.md` 6, `technology/dds_guide.md` 5 | Nothing above 14 hits in a 256-file `technology/` directory. |
| Files mentioning `raft` | 22 | Consensus is *named*, widely. |
| Files mentioning `paxos` | 10 | Named. |
| Files mentioning `quorum` | 36 | The most-mentioned term in the repo — and it appears mostly as *configuration advice*, not as arithmetic. |
| Files mentioning `vector clock` | **0** | **The measured zero.** Causality has no coverage anywhere in this repository. |
| Files mentioning `crdt` | 2 | Near-zero. |
| Files mentioning `cap theorem` | 3 | Three files, all passing. |
| Files mentioning `consistent hashing` | 7 | |
| Files mentioning `two-phase commit` | 11 | |
| Files matching `linearizab*` | 9 | The word appears; the definition rarely does. |
| Files mentioning `byzantine` | 2 | |

The five existing system-design meta-guides and their real sizes, all verified by reading the working tree:

| Guide | Lines | Frame it owns |
| --- | --- | --- |
| `technology/nalsd_system_design_guide.md` | 705 | The *NALSD interview method* |
| `technology/grokking_system_design_companion_guide.md` | 716 | Companion to the Grokking *course* |
| `technology/system_design_interview_insiders_guide.md` | 721 | Companion to a *book* about interviews |
| `technology/google_system_design_interview_guide.md` | 708 | The Google *rubric and levels* |
| `technology/ddia_study_companion_guide.md` | 709 | Companion to *DDIA* — the closest existing treatment of the theory |

Every one of those five is **interview-framed or book-framed**. Each treats distributed systems as subject matter to be *recalled* for an evaluation. None of them owns distributed systems as a subject to be **engineered** — as a set of choices with theorems constraining them and invoices attached. This guide is that missing layer: the failure models, the impossibility results, time and ordering, the consistency spectrum, the consensus families, replication and partitioning, failure detection, delivery semantics, and the judgement for choosing among them.

### 1.3 What This Guide Is Not

**This is neither a sixth interview guide nor a book companion.** Stating that plainly matters, because the repository already has five of the former and one of the latter, and adding a seventh interview-framed document would produce a longer shelf and no new capability. The differences are structural, not stylistic:

| | The five meta-guides | This guide |
| --- | --- | --- |
| **Frame** | An evaluation. The reader is being assessed, or is preparing to be. | An engineering review. The reader is about to commit a design. |
| **Unit of content** | A system (design Twitter, design a rate limiter). | A **question** (what am I assuming about failure?). |
| **Success criterion** | Can you produce a defensible answer in 45 minutes? | Can you name the invariant your system must preserve, and say what it costs? |
| **Treatment of theory** | Recall-oriented: name CAP, name the trade-off. | **Conditions-oriented**: CAP is about *one* consistency model under *partition*, and 'CA' is not an available option (§4.2). |
| **Treatment of products** | As design ingredients. | As *answer sets* to the questions in §1.1 — and the product guides own the products themselves (§1.4). |
| **What it optimises** | Coverage of likely questions. | Avoiding a specific class of production failure (§11.5). |

The distinction that matters most: an interview guide can afford to say "you'd need consensus here" and move on. This guide cannot, because "you'd need consensus here" is the beginning of the sentence — the rest of it is *which family, under which failure assumptions, at what latency cost, and what happens when the quorum is unreachable* (§6).

### 1.4 The Cross-Reference Boundary

This guide does not re-derive what its siblings own. The boundary is explicit, and it was verified by `ls` against the working tree before writing — **`technology/temporal_guide.md` does not exist**; the file that exists is `technology/temporal_workflow_guide.md`, which is cited under that name.

| Owner | What it owns | What this guide does |
| --- | --- | --- |
| `technology/cockroachdb_guide.md` | CockroachDB: its architecture, its SQL surface, its operations. | Cross-references it as one *answer set* — Raft per range, serializable by default, HLC-based timestamps. Does not re-derive. |
| `technology/kafka_virtualization_guide.md`, `technology/kafka_alternatives_guide.md`, `technology/event_stream_processing_guide.md` | Kafka's mechanism, ISR/`acks`, transactions and idempotent producer; the alternatives; stream processing. | §9 cross-references these for **mechanism** and confines itself to the **semantics** — what `acks=all` does and does not promise. |
| `technology/polardb_vs_oceanbase_guide.md` | PolarDB vs OceanBase: shared-storage vs shared-nothing, Paxos-based replication. | Cited in §7 as a concrete partitioning/replication contrast. |
| `technology/openbao_vs_vault_guide.md` | Secret management, including the HA/storage-backend choice. | Cited in §8 as an example of lease semantics in a product. |
| `technology/oracle_sharding_guide.md` | Sharding mechanics, shard directors, cross-shard queries. | Cited in §7.5–§7.6 as the worked treatment of partitioning's operational cost. |
| `technology/apache_seata_guide.md` | **The saga / TCC / AT pattern space in depth.** | §10.3 **defers to it by name** and does not re-derive the patterns. |
| `technology/temporal_workflow_guide.md` | Durable execution, workflow-as-code, retries and compensation framed as code. | Cited in §10 as the durable-execution framing of compensation. |
| `technology/dds_guide.md` | DDS: QoS, durability and reliability profiles, discovery. | Cited as a product whose QoS profiles *are* an answer set to §9's delivery question. |
| `technology/cephfs_alternatives_guide.md` | Distributed filesystem alternatives. | Cited in §7 for placement and rebalancing at the storage layer. |
| `technology/monolith_to_microservices_guide.md` | Service decomposition, boundaries, migration. | Cited in §12 — the *decomposition* is that guide's subject; the *coordination* it forces is this one's. |
| `technology/chaos_engineering_guide.md` | **The testing techniques** — fault injection, experiment design, blast radius. | §11 and §13 **cross-reference it rather than duplicating**; it owns *how you would find out*. |
| `technology/deterministic_engineering_guide.md` | **Determinism, replay, deterministic simulation testing.** | §9.4 and §11 cross-reference it for deterministic replay as a mechanism. |
| `technology/ddia_study_companion_guide.md` | The closest existing treatment of the theory, in book-companion form. | Cross-referenced as *closest* — this guide does **not** duplicate its chapter map; it cites the primary papers directly instead. |

### 1.5 The Decoder

Vendor documentation uses a shared vocabulary with **product-specific meanings**. Reading the same word across three products and assuming it means one thing is the single most common source of wrong designs. Decode first.

| Term | What it usually means | What it *actually* denotes | The trap |
| --- | --- | --- | --- |
| **node** | "A server" | A process that participates in the protocol. One machine may host many nodes; one node may be a process that is paused, not dead (§8.1). | Assuming node failure = machine failure. A **stopped process on a healthy machine** is the case that breaks lease-based designs (§8.5). |
| **replica** | "A copy of the data" | A participant holding a copy *and* (usually) a vote in a quorum. | "3 replicas" can mean 3 voters (consensus) or 3 copies with one voter (leader-based async) — completely different failure behaviour. |
| **leader / follower** | "Primary / secondary" | leader = the node whose log order is authoritative *for the current term*; follower = a node that accepts the leader's order. | There is no global "the leader"; there is a leader **per term** or **per partition/shard**. A five-shard system has five. |
| **quorum** | "Majority" | Usually **⌊n/2⌋+1** for consensus; in leaderless stores it is *configurable read and write sets* (R + W > N), which is a **different guarantee** and not a majority. | §13.5: quorum arithmetic that silently lost fault tolerance (e.g. a 4-node cluster tolerating one failure, not two). |
| **term / epoch** | "A counter" | A monotonically increasing integer identifying a **leadership incarnation**. Every message carries it; higher term wins. | The term is *not* a clock and cannot be compared across shards. |
| **log** | "The append-only file" | The **ordered sequence of state-machine commands**. Order in the log *is* the consistency model. | "We have a log" says nothing about whether all replicas agree on its order — that is what consensus buys (§6). |
| **state machine** | "A service" | A deterministic function of an ordered command sequence. Replication works by replicating the *sequence*, not the state. | If the function is non-deterministic (uses `now()`, a random value, or an iteration over a hash map), replicas diverge. This is the **deterministic state machine requirement** and it is an engineering obligation, not a product feature. |
| **shard / partition** | "A piece of the data" | The unit of **independent** consensus and placement. Cross-shard = cross-consensus-group = no shared order. | §7.6: the cross-partition operation that seemed fine in design and is a distributed transaction in production. |
| **split brain** | "Two leaders" | Two nodes that each believe they are the leader **for the same term/partition** and can both act. | Split brain is usually prevented for *safety* (the old leader cannot commit) but **not for side effects** (the old leader can still call the payment API). That is exactly what fencing exists for (§8.5). |
| **fence / token** | "A lock ID" | A **monotonically increasing number** attached to every write, which the *storage system* checks and rejects if stale. | A fence only works if **the resource validates it**. A client-side check is not a fence (Kleppmann, *How to do distributed locking*, 8 Feb 2016 ✅). |
| **membership** | "The cluster config" | The set of nodes that count toward quorums. Changing it is itself a consensus problem (§6.5). | Naive reconfiguration can create **two disjoint majorities** in adjacent configurations. |
| **witness** | "Tie-breaker" | A non-data-bearing voting member — it holds votes, not data. | A witness improves **availability under failure**, not durability: losing it does not lose data, but losing it can block writes. |

Two more patterns worth decoding once:
- **"strong consistency"** is a marketing phrase, not a definition. Ask *which* one (§5). The answer is almost always narrower than the reader assumed.
- **"exactly-once"** appears in three different scopes — producer-to-broker, broker-internal, and end-to-end effect — and only the third matters to a business process (§9.2).

## 2. The Failure Model

### 2.1 The Taxonomy

A **failure model** is a statement of what a faulty component is allowed to do. It is the most consequential single choice in a distributed design, because — as §2.2 shows — it silently determines which algorithms are even on the table. The classes, in increasing severity:

| Class | Definition | What a faulty node may do | Real-world plausibility |
| --- | --- | --- | --- |
| **Crash-stop** | A node halts and never returns. | Nothing, forever. | An idealisation. Real hardware is replaced and the node *does* come back. |
| **Crash-recovery** | A node halts, then restarts, possibly later, possibly with lost volatile state. | Resumes from durable state; may have forgotten everything in memory. | **The default model for real systems.** This is what a JVM restart after an OOM kill is. |
| **Omission** | A node keeps running but drops messages — send-omission or receive-omission. | Fails to send, or fails to receive, some messages; otherwise behaves. | Network congestion, full socket buffers, a GC pause that spans a socket timeout. |
| **Timing** | A node runs correctly but its *speed* is wrong — too slow, or its clock is wrong. | Correct computation at arbitrary speed; clock may drift or jump. | **The most under-modelled and most common real failure.** A 90-second packet delay has been observed in production (GitHub incident, documented in Bailis & Kingsbury, *The Network is Reliable*, ACM Queue 12(7), 2014 — cited as [7]/[8] in Kleppmann's post ✅). |
| **Byzantine** | A node may behave *arbitrarily* — send contradictory messages, lie, collude, or be controlled by an adversary. | Anything. | Protocol bugs, storage corruption, a compromised host. Rare in a controlled datacentre, decisive in an open one — and the reason the BFT family exists (§6.3). |

Nothing in this taxonomy is exotic. The **timing** class is the one that catches careful engineers, because a timing failure looks exactly like correct behaviour that happens to be slow.

### 2.2 Why the Model Determines the Algorithm

The failure model is not a philosophical preface — it is an input to the applicability of the algorithm. This is a **PROVEN RESULT**, not advice:

- **Fully asynchronous + at least one crash fault ⇒ no deterministic consensus protocol can guarantee termination.** Fischer, Lynch and Paterson, *Impossibility of Distributed Consensus with One Faulty Process*, JACM 32(2):374–382, April 1985 ✅. The paper is explicit about its assumptions, and they are worth quoting rather than paraphrasing, because the folklore version is looser than the theorem: "we show the surprising result that no completely asynchronous consensus protocol can tolerate even a single unannounced process death. We do not consider Byzantine failures, and we assume that the message system is reliable — it delivers all messages correctly and exactly once." ✅ And on what makes it bite: "Crucial to our proof is that processing is completely asynchronous; that is, we make no assumptions about the relative speeds of processes or about the delay time in delivering a message. We also assume that processes do not have access to synchronized clocks, so algorithms based on time-outs, for example, cannot be used… Finally, we do not postulate the ability to detect the death of a process, so it is impossible for one process to tell whether another has died (stopped entirely) or is just running very slowly." ✅
- **Add a synchrony assumption and consensus becomes solvable.** Dwork, Lynch and Stockmeyer, *Consensus in the Presence of Partial Synchrony*, JACM 35(2):288–323, April 1988 ✅ (conference version at PODC 1984, pp. 103–118, per the FLP reference list ✅). The synchrony does not have to be guaranteed from the start — it only has to hold *eventually*, for long enough.
- **Tolerate Byzantine faults ⇒ you need more replicas than crash tolerance.** The classical requirement for crash/omission agreement is 2f+1 processes for f faults; for Byzantine agreement the classical bound is 3f+1. Lamport, Shostak and Pease, *The Byzantine Generals Problem*, ACM TOPLAS 4(3):382–401, July 1982 ✅; the threshold is the *raison d'être* of the 3f+1 arithmetic in PBFT (§6.3).

So the model determines applicability, and the determination flows one way: **choosing a weaker failure model buys you a stronger algorithm at a lower cost, and that trade is only legitimate if you can defend the model.** Most teams cannot defend it, and assume it anyway. That assumption is what §2.5 exists to expose.

### 2.3 Partial Failure as the Defining Condition

The defining condition of distributed systems is **partial failure**: some components fail while others continue, and *the failure is not directly observable by the components that remain*. In a single process, a failed function call throws; the caller knows. In a distributed system, a failed remote call produces... nothing. No exception, no response, no error code — silence. The caller cannot distinguish:

- the request never arrived,
- the request arrived and the work completed but the response was lost,
- the request arrived and the work did not start,
- the request is still in flight,
- the request arrived and the work completed **twice** because of a retry.

This is the reason distributed systems are hard, and it is why the discipline is mostly about **deciding what to do with ambiguity** rather than about clever algorithms. An algorithm can decide consistently; only a design can decide *what ambiguity means*.

### 2.4 Ambiguous Failure — Slow Is Indistinguishable from Dead

The single most useful sentence in this guide: **without a synchrony assumption, a slow node and a dead node produce identical observations.** FLP names this: because there are no clocks and no failure detector, "it is impossible for one process to tell whether another has died (stopped entirely) or is just running very slowly" ✅. The consequence is a permanent two-sided error for any timeout-based detector:

| A timeout fires… | …and the node was dead | …and the node was merely slow |
| --- | --- | --- |
| **Correct action** | Failover happens; availability preserved. | The node is still working — you now have the **two-generations problem** (§8.5). |
| **Failure consequence** | Wasted time; a brief stall. | **Two nodes acting.** The old "slow" node continues its work, the new leader starts, and any lock, lease or assignment is now held by two parties. |

There is no timeout value that avoids both columns. **You do not choose the timeout; you choose which column you can survive.** This is why §8.5's fencing token is not an optimisation but a requirement: fencing is the mechanism that makes the right-hand column *safe* rather than merely rare.

### 2.5 The Assumption Ledger

Every design silently assumes things about the network, the clocks and the machines. Writing them down is the cheapest form of review available, and it is the practice I recommend most strongly in this guide. The ledger below is the one I bring to a design review; the entries are real designs I have reviewed, generalised.

| Common design | Silently assumes about **the network** | Silently assumes about **clocks** | Silently assumes about **machines** | What breaks when the assumption fails |
| --- | --- | --- | --- | --- |
| **Leader with a lease** (§8.4) | That the leader can be told it lost leadership *before* the new leader acts | That **clock rates are bounded** — the lease is only valid if the leader can measure its own expiry | That a paused process resumes promptly | A GC pause longer than the lease ⇒ the old leader acts after expiry ⇒ two actors. Fencing is the only fix (§8.5). |
| **3-node Raft / Paxos cluster** (§6) | That a majority can reach each other *in both directions* — an asymmetric partition is worse than a symmetric one | Not needed **for safety**; needed for election timeouts (**liveness only**) | That all three do not share a power feed or a rack | A symmetric partition stalls writes (correct!); an asymmetric one can stall liveness *and* mislead dashboards. |
| **2PC coordinator** (§10.1) | That the coordinator's decision eventually reaches every participant | — | That the coordinator's durable log survives | Coordinator dies between prepare and commit ⇒ participants **block, holding locks**, indefinitely. This is the classic blocking failure (§10.1). |
| **Kafka producer with `acks=all`** | That the ack reflects durable replication | — | That brokers' disks are honest about `fsync` | Losing the whole ISR at once still loses data; and `acks=all` says nothing about the consumer side (§9.2). |
| **Idempotent consumer with a dedup table** | That retries carry a **stable key** | — | That the dedup store is transactional with the effect | If the key is regenerated per retry, dedup silently never fires. If the dedup write and the effect are not atomic, you have moved the duplicate, not removed it (§9.4). |
| **Retry with exponential backoff** | That retries are few, and that the callee can absorb them | — | That jitter is present | Without jitter, retries synchronise into a **thundering herd** that arrives exactly when the callee is weakest (§13.4). |
| **TTL-based cache invalidation** | That the invalidation message is not lost | That **clocks are roughly synchronised across nodes** | — | Both assumptions are false at scale; the entry lives longer than intended, and *how much longer* is unbounded (§3.2). |
| **Last-write-wins with wall-clock timestamps** (§3.7) | — | That clocks are monotone and comparable | — | LWW **silently discards the losing write** and returns a coherent-looking, wrong value. No error, no alert, no anomaly — just a decision nobody made (§13.2). |
| **Snowflake-style ID generation** | — | That the clock **never goes backwards** | That node IDs are unique and never reused | A backward clock step (NTP) can mint duplicate IDs; a reused node ID on a replaced host can too. |
| **Snowflake / Kafka timestamps for business ordering** | — | That event time ≈ wall-clock time | — | Cross-node ordering by timestamp is **not sound** (§3.1) — and it is the most common ordering bug in event-driven estates. |
| **Consensus quorums that span two data centres** | That inter-DC latency is stable enough to meet the write SLO | — | That the link is not the single shared dependency | Every write pays one or two WAN round trips; a cross-region partition can cost availability for the minority side (§6.6, §7.2). |
| **"Strong consistency" as stated in a slide** | — | — | — | Untested under partition. Nine times in ten, nobody ever ran the partition (§13.9). |
| **Monitoring and dashboards** | That metrics arrive | That all clocks agree so that correlation works | — | During an incident, the metrics from two hosts are **misaligned in time**, and the causal story you read off the graph is not the one that happened. |

Two habits come out of the ledger:
1. **For every row you cannot defend, note the compensating mechanism.** A lease without a fence has no defence (§8.5). A retry without idempotency has no defence (§9.4).
2. **The ledger is a test plan.** Each "what breaks" cell is a chaos experiment awaiting a hypothesis — the techniques belong to `technology/chaos_engineering_guide.md`, and the deterministic-replay techniques to `technology/deterministic_engineering_guide.md`. This guide says *what to test*; those say *how*.

## 3. Time, Ordering and Causality

> **Why this section exists:** `vector clock` appears in **zero** files in this repository (measured 2026-09-18). `linearizab*` appears in nine files, almost never followed by a definition. Causality is the one part of this discipline with no prior coverage here — and it is the part that most often explains an otherwise inexplicable production bug.

### 3.1 Why Physical Clocks Cannot Order Distributed Events

Lamport's 1978 paper opens with the problem, not the solution, and the framing is still the best one available:

> "In a distributed system, it is sometimes impossible to say that one of two events occurred first. The relation 'happened before' is therefore only a partial ordering of the events in the system. We have found that problems often arise because people are not fully aware of this fact and its implications." — *Time, Clocks, and the Ordering of Events in a Distributed System*, **Communications of the ACM 21(7):558–565, July 1978** ✅

Two things in that paper matter more than their fame suggests. First, Lamport's motivating example is an **airline reservation system**: "we specify that a request for a reservation should be granted if it is made before the flight is filled" — and the whole point is that "this concept must be carefully reexamined". A business rule expressed as "before" is a business rule about *causal* order, and a timestamp is not a causal order. Second, he is explicit that the problem is not solved by having clocks: "if the specification is in terms of physical time, then the system must contain real clocks. Even if it does contain real clocks, there is still the problem that such clocks are not perfectly accurate and do not keep precise physical time." ✅

The formal definition is three conditions, and it is worth having exactly:

- (1) If `a` and `b` are events in the same process, and `a` comes before `b`, then `a → b`.
- (2) If `a` is the sending of a message and `b` is the receipt of that same message, then `a → b`.
- (3) If `a → b` and `b → c`, then `a → c`.
- Two distinct events are **concurrent** if neither `a → b` nor `b → a`. ✅

**Concurrency is not simultaneity.** Two events are concurrent when neither *could have causally affected* the other. That is a statement about information flow, not about nanoseconds — which is why it survives even with perfect clocks.

### 3.2 Skew and Drift — the Reality

A **clock** here means a machine's `CLOCK_REALTIME`. It is wrong in three ways simultaneously:

| Defect | Mechanism | Magnitude to expect in a well-run estate |
| --- | --- | --- |
| **Offset** | The clock reads a value that differs from true time now. | Tens of milliseconds under NTP; **hundreds of milliseconds to seconds** on a VM or container host whose clock is being disciplined aggressively. |
| **Skew** | Two clocks disagree *with each other* at a given instant. | The offset difference. Skew is what breaks LWW and cross-node ordering. |
| **Drift** | The clock's *rate* differs from true time, so offset grows between corrections. | 10–100 ppm for commodity quartz (roughly 1–9 seconds per day); corrected by NTP, but correction is not instantaneous. |

Three additional facts that break intuitive designs:

1. **NTP can step the clock backwards.** After a large correction (or a VM restore, or a leap-second handling choice), `CLOCK_REALTIME` can jump *backwards* — which is fatal to any design that assumes a monotonic wall clock (Snowflake IDs, "latest wins" comparisons, TTL freshness checks).
2. **Leap seconds** are a genuine hazard: smearing and stepping are both used, and both produce cross-node disagreement for a window.
3. **The only engineering solution to uncertainty is to expose it and wait it out.** This is what Spanner's TrueTime does, and the cost is instructive: TrueTime "keeps uncertainty small (**generally less than 10ms**) by using multiple modern clock references (GPS and atomic clocks)" — and having got uncertainty that small, Spanner still has to *wait*: "If the uncertainty is large, Spanner slows down to wait out that uncertainty." ✅ (Corbett et al., *Spanner: Google's Globally-Distributed Database*, OSDI 2012). The lesson is not "buy atomic clocks". The lesson is that **bounded uncertainty, purchased at enormous cost, is the only sound way to use physical time for ordering** — and if you are not paying that cost, you must not use physical time for ordering.

### 3.3 Monotonic vs Wall Clock

| | Wall clock (`CLOCK_REALTIME`, `System.currentTimeMillis()`) | Monotonic clock (`CLOCK_MONOTONIC`, `System.nanoTime()`) |
| --- | --- | --- |
| Advances | Roughly with true time | Faithfully at a steady rate |
| Comparable across hosts? | **Nominally yes, actually no** | **Never** — the epoch is arbitrary per host (often boot) |
| Can go backwards? | **Yes** (NTP step, VM migration, manual set) | **No** |
| Safe for | Displaying a human timestamp; coarse retention windows | **Measuring elapsed time — timeouts, leases, rate limiters, backoff** |

The rule is short: **use the monotonic clock for every duration, and the wall clock only for things a human will read.** Every lease in §8.4, every timeout in §8.1 and every retry budget in §13.4 must be measured on a monotonic clock. A lease computed with `System.currentTimeMillis()` is a lease that a backward NTP step can extend or a forward step can truncate — and neither will be logged.

### 3.4 Logical Clocks

Lamport's answer to §3.1 is to *stop using physical time for causality*. Give each process a counter `C`; increment on each event; on send, attach `C`; on receive, set `C = max(C, C_msg) + 1`. The correctness condition is stated as a requirement rather than derived:

> **Clock Condition.** For any events `a`, `b`: if `a → b` then `C(a) < C(b)`. ✅

The paper is careful about the converse, and the care is the point: "**we cannot expect the converse condition to hold as well**, since that would imply that any two concurrent events must occur at the same time." ✅ So a Lamport clock gives you a **total order that is consistent with causality** — `a → b` implies `L(a) < L(b)` — but `L(a) < L(b)` does **not** imply `a → b`. You cannot distinguish *causally ordered* from *arbitrary tie-break*.

That gap is exactly where real bugs live. If you use Lamport clocks to resolve a conflict, you are choosing a winner arbitrarily among concurrent events; the system is consistent but the choice carries no business meaning.

### 3.5 Vector Clocks and What They Actually Buy

A **vector clock** keeps one counter *per process*, and merging is component-wise max. Independently proposed by Fidge and Mattern in 1988–89 ⚠ (see §15 — the Mattern date is inconsistent across sources; the CRDT 2011 reference list dates it 1989 and gives the venue as *Int. W. on Parallel and Distributed Algorithms*, pp. 215–226 ✅, while the secondary literature commonly cites 1988). What a vector clock buys, precisely:

| Comparison of `V(a)` vs `V(b)` | Conclusion | Lamport clock can do this? |
| --- | --- | --- |
| `V(a) ≤ V(b)` component-wise, not equal | `a → b` | Yes |
| `V(b) ≤ V(a)` component-wise, not equal | `b → a` | Yes |
| Neither ≤ the other | **`a ∥ b` — genuinely concurrent** | **No** |
| Equal | Same event | N/A |

So a vector clock does one thing a Lamport clock cannot: **it detects concurrency.** That is its entire value, and it is a large value — because concurrency detection is the precondition for *any* correct conflict-handling decision (§3.7).

Its costs are real and are why it is not universal:
- **Size grows with the number of writers**, not with the number of events. A vector clock per key with 50 writers is 50 counters carried on every write.
- **It must be maintained per replicated object**, which means it must be stored durably with the object — it is part of the data, not part of the transport.
- **It must be truncated in practice.** Dynamo used vector clocks to detect concurrent versions, and reported that the clock size is bounded in practice precisely because unbounded growth is unworkable; truncation trades false concurrency (safe — you fall back to application merge) for a size bound ⚠ (DeCandia et al., *Dynamo: Amazon's Highly Available Key-value Store*, **SOSP 2007, pp. 205–220** ✅ is verified; the truncation-limit detail is from the paper's own description and I flag it as ⚠ because I did not re-read that passage this pass — see §15).

### 3.6 Causal Delivery

**Causal delivery** is the delivery guarantee that matches the order causality demands: a replica does not deliver a message until it has delivered every message that happened-before it. It is the natural companion to vector clocks (the vector is the mechanism; causal delivery is the policy), and it is what a business process usually *means* when it says "events must arrive in order".

The important limitation: **causal delivery is per-object and per-conversation, not global.** If a user's profile write and their payment write are not causally related by any message or shared state, causal delivery will happily reorder them — correctly, because their order is genuinely undefined. Many "we need global ordering" requirements dissolve once that is understood; the requirement was really "we need ordering *within this aggregate*" (§11.2).

### 3.7 The Worked Indeterminacy — Two Events Whose Order You Cannot Determine

This is the section's payoff. Assume the most common real-world setup: two replicas, last-write-wins conflict resolution on a wall-clock timestamp, and clocks disciplined by NTP but **not** bounded to TrueTime-standards.

**Clock configuration (realistic, not adversarial):**

- Node **A**: `CLOCK_REALTIME` is **600 ms slow**.
- Node **B**: `CLOCK_REALTIME` is **600 ms fast**.
- Skew between A and B: **1.2 s**. This is large but attainable — it is ordinary on a virtualised host under CPU contention before NTP re-disciplines, and it is the *steady-state* value for a container restored from a snapshot.

**Events:**

| Event | True time | Stamped by | Timestamp written |
| --- | --- | --- | --- |
| `e₁` — client writes `address = "1 Raffles Place"` on **B** | 10:00:00.000 | B (fast) | **10:00:00.600** |
| `e₂` — client writes `address = "8 Marina Boulevard"` on **A**, *after the client has seen* `e₁` and is deliberately correcting it | 10:00:00.500 | A (slow) | **10:00:00.500 − 0.600 = 09:59:59.900** |

**Two conclusions follow, and they are different in kind.**

**(a) The timestamps are not merely wrong — they are misleading in the direction that loses data.** The causal fact is `e₁ → e₂`: the second write happened after, and *because of*, the first. The correct final value is `"8 Marina Boulevard"`. The stored timestamps say `e₂` (09:59:59.900) precedes `e₁` (10:00:00.600). A last-write-wins register therefore keeps `e₁` and **discards the correction**. Nothing errors. Nothing alerts. The customer's new address is silently replaced by their old one, and the system reports success for both writes.

**(b) From the timestamps alone, the order is *undeterminable*, not just wrong.** Look at the two timestamps with no other information:
- They differ by 700 ms.
- The maximum plausible clock skew between two unrelated hosts is unbounded by anything the reader knows (NTP is *usually* good; it is not *guaranteed* to be).
- Therefore `e₁` could precede `e₂`, `e₂` could precede `e₁`, or the two could be genuinely concurrent — and **no function of `(10:00:00.600, 09:59:59.900)` alone can distinguish these cases.**

This is the indeterminacy, stated exactly: **a per-event timestamp is a point estimate with an unstated error bar, and comparing two point estimates is only sound if the error bars are disjoint.** Wall-clock timestamps never carry their error bars. TrueTime does, and that is the whole of its design: it returns an *interval* `[earliest, latest]` with the guarantee that true time lies inside it, so a comparison is only sound when the intervals are disjoint — and when they are not, Spanner waits ✅.

**What each fix actually costs:**

| Fix | What it buys | What it costs |
| --- | --- | --- |
| **Vector clocks** + application merge | Correct concurrency detection; `e₁ ∥ e₂` and `e₁ → e₂` become distinguishable | Clock size per object; **the application must now handle "concurrent"** — which is real engineering work, not a library flag |
| **TrueTime-style bounded uncertainty** | Sound comparison of physical timestamps | Atomic clocks and GPS per datacentre; **wait out the uncertainty** on every transaction; a hard dependency on an entire timing infrastructure ✅ (Spanner, OSDI 2012) |
| **Hybrid Logical Clocks** | Physical-time-shaped timestamps that respect causality and stay close to wall time; the timestamp is bounded by physical time and preserves the causal property ⚠ (Kulkarni et al., 2014 — see §15) | Implementation complexity; still no total order without coordination |
| **Single writer per key** | Removes the conflict entirely — the coordinator's order *is* the order | Availability of that writer; latency to reach it; this **is** coordination (§11.2) |
| **CRDTs** | Concurrent updates converge with no coordination, if the operation is genuinely commutative or a semi-lattice join ✅ (Shapiro et al., SSS 2011) | Only works when the *semantics* are conflict-free; an address is **not** a CRDT (§10.4) |

The last row is the honest one to end on: **there is no merge rule for "the customer's address" that is correct in general**, because the two writes are not commutative — one was a correction. The right answer for that key is a single writer, and the cost is coordination. Most systems avoid paying it and accept silent loss instead, usually without anyone having written down that decision. §13.2 is the anti-pattern version of this paragraph.

## 4. The Impossibility Results and What They Actually Mean

> **Why this section exists:** these are the most name-dropped and least accurately stated results in the field. Every one of them is narrower and more specific than its folklore form. Getting the conditions exact is not pedantry — it is the difference between "consensus is impossible, so we gave up" and "consensus requires a synchrony assumption, so here is ours".

### 4.1 Consensus with Faulty Processes — FLP, at Its Precise Conditions

**PROVEN RESULT.** Fischer, Lynch and Paterson, *Impossibility of Distributed Consensus with One Faulty Process*, **Journal of the ACM 32(2):374–382, April 1985** ✅ (originally presented at the 2nd ACM Symposium on Principles of Database Systems, March 1983, pp. 1–7, per the paper's own reference list ✅).

The folklore version is "consensus is impossible in an asynchronous system". The paper's version is more specific, and the specifics are what you can act on. Quoting the assumptions directly ✅:

| The model | As stated in the paper |
| --- | --- |
| **The result** | "every protocol for this problem has the possibility of nontermination, even with only one faulty process" |
| **Faults considered** | "We do **not** consider Byzantine failures" — crash/stopping only |
| **Message system** | "we assume that the message system is reliable — it delivers all messages correctly and exactly once" |
| **Timing** | "processing is completely asynchronous; that is, we make no assumptions about the relative speeds of processes or about the delay time in delivering a message" |
| **Clocks** | "processes do not have access to synchronized clocks, so algorithms based on time-outs, for example, cannot be used" |
| **Failure detection** | "we do not postulate the ability to detect the death of a process, so it is impossible for one process to tell whether another has died (stopped entirely) or is just running very slowly" |
| **Problem instance** | Binary consensus; every process starts with a value in `{0,1}`; only "some process eventually make a decision" is required for the proof |
| **Atomic broadcast assumed** | Yes — "an 'atomic broadcast' capability is assumed"; messages may still be delayed arbitrarily and delivered out of order |

**What is forbidden.** In a fully asynchronous system with at least one crash fault and no failure detector and no clocks, no deterministic protocol can guarantee both **agreement** and **termination**. Something must be weakened or assumed.

**What is *not* forbidden.** The theorem does not say consensus cannot be *solved* — it says it cannot be **guaranteed to terminate**. This distinction is the entire practical resolution. Real consensus protocols are *safe* in a fully asynchronous system (they never decide two different values) and *live* only when the timing assumptions hold long enough. That is why a Raft cluster whose majority is unreachable **stops accepting writes** rather than producing a wrong answer — the correctness property is preserved precisely by giving up the liveness property, which is exactly what the theorem says you must do.

The paper also notes the consequence in the language of its own era, and it is the sentence that should be taped to a design-review wall: "The asynchronous commit protocols in current use all seem to have a 'window of vulnerability' — an interval of time during the execution of the algorithm in which the delay or inaccessibility of a single process can cause the entire algorithm to wait indefinitely. It follows from our impossibility result that **every commit protocol has such a 'window'**". ✅

**The escape hatch, and its price.** Dwork, Lynch and Stockmeyer, *Consensus in the Presence of Partial Synchrony*, **JACM 35(2):288–323, April 1988** ✅ (conference version: PODC 1984, pp. 103–118 ✅). The escape is not to be fully synchronous; it is to assume the system is synchronous *eventually*, or synchronous *outside* unknown bounds. Both are sufficient to make consensus solvable. The price: **the assumption is not provable from inside the system**, and any algorithm's liveness now depends on an assumption the operator cannot verify — it can only be falsified by an outage.

### 4.2 CAP and the Common Misreading

**PROVEN RESULT.** Gilbert and Lynch, *Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services*, **ACM SIGACT News 33(2):51–59, June 2002** ✅.

The theorem is a formalisation of a conjecture made by Eric Brewer in an invited talk. The talk's own slides are the cleanest record of the claim, and I verified them directly ✅: **PODC Keynote, 19 July 2000, "Towards Robust Distributed Systems"**, Brewer (UC Berkeley / Inktomi), slide headed *The CAP Theorem*, showing Consistency / Availability / "Tolerance to network Partitions" with the caption "**Theorem: You can have at most two of these properties for any shared-data system**". ✅ That is the "pick 2 of 3" framing — and it is Brewer's own, which matters for §4.3.

Gilbert and Lynch's definitions are what the mathematics actually constrains, and they are **narrower than the words suggest**:

- **Consistency** means **atomic/linearizable consistency** for a single read/write object — "there must exist a total order on all operations such that each operation looks as if it were completed at a single instant." The note is explicit: "This is equivalent to requiring requests of the distributed shared memory to act as if they were executing on a single node." ✅ The paper goes further: atomic consistency "has a different meaning than the *Atomic* in ACID, as it subsumes the database notions of both *Atomic* and *Consistent*". ✅
- **Availability** means "every request received by a non-failing node in the system must result in a response" ✅ — with no bound on latency, so it is a *weak* requirement per-request and a *strong* one under partition.
- **Partition tolerance** means the network "will be allowed to lose arbitrarily many messages sent from one node to another" ✅.

And the theorem's own precision: "It is impossible in the asynchronous network model to implement a read/write data object that guarantees **Availability** and **Atomic consistency** in all fair executions (including those in which messages are lost)." ✅

**The two misreadings that cause damage.**

**(i) "CAP says pick 2 of 3, so we can be CA."** Under a partition — which is not something you schedule — **CA is not an available option.** The only way to have C and A is to be P-intolerant, i.e. to *not have a partition*, which is a property of your network rather than of your design. The paper's own Corollary makes the sharp version: even if you require atomic consistency only "in fair executions in which **no** messages are lost", availability-plus-atomicity is still impossible ✅. The reason is worth internalising: "in the asynchronous model an algorithm has no way of determining whether a message has been lost, or has been arbitrarily delayed in the transmission channel." ✅ So a system cannot confine its consistency guarantee to the good times, because it cannot tell that it is in the good times.

**(ii) "CAP says we must weaken consistency even when there is no partition."** This is the misreading Abadi names explicitly: "it is wrong to assume that DDBSs that reduce consistency in the absence of any partitions are doing so due to CAP-based decision-making. In fact, **CAP allows the system to make the complete set of ACID guarantees alongside high availability when there are no partitions.** Therefore, the theorem does not completely justify the default configuration of DDBSs that reduce consistency." ✅ (Abadi, IEEE Computer 45(2):37–42, February 2012.) The theorem constrains behaviour *during* a partition and says nothing about behaviour between partitions. Systems that are weakly consistent by default made a **different** choice, usually for the reason in §4.4.

### 4.3 The Self-Correction

Brewer himself revised the framing. *CAP Twelve Years Later: How the "Rules" Have Changed*, **IEEE Computer 45(2):23–29, February 2012** ⚠ — **flagged**: I could not retrieve the published article this pass (InfoQ's version of the article returned a bot challenge; the IEEE page was not extracted; see §15). I therefore state the correction at the level I can defend without quoting it:

- The **"2 of 3" formulation is acknowledged as misleading**, because partitions are rare and the trade-off only binds *while a partition is in progress*. Outside a partition, C and A are both available, and the interesting engineering question is *latency*.
- The operational consequence — that **partition handling should be designed explicitly**, including a strategy for what the system does at the start of a partition and how it reconciles on heal — is the part that changed practice.
- The related result that anticipates it, and which I also could not verify this pass ⚠, is Fox and Brewer, *Harvest, Yield, and Scalable Tolerant Systems*, **HotOS 1999**, which reframes availability as a continuum (`harvest` = fraction of data answered; `yield` = fraction of requests answered). Brewer's own 2000 keynote slides already carry the intuition in the form "All systems are probabilistic…" and "**Capacity * Completeness == Constant**" ✅, which is the harvest/yield trade stated in one line.

Because §14 is a claims audit, I will not launder an unverified quotation into a verified one. The *substance* of the Brewer correction is stated above as ⚠ and repeated in §15.

### 4.4 The Extension: PACELC

**ENGINEERING FORMULATION (with an argument, not a theorem).** Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design*, **IEEE Computer 45(2):37–42, February 2012** ✅ — note the same issue and month as the Brewer article.

PACELC extends CAP by asking a second question: *if there is a Partition, how do we trade Consistency against Availability; Else (in normal operation), how do we trade Consistency against Latency?* The paper's opening argument is that the second question is the one that actually shaped modern systems: the consistency-versus-latency trade-off "arguably has been more influential on DDBS design than the CAP tradeoffs" ✅.

The reason the `ELC` half is unavoidable is stated crisply: "a high availability requirement implies that the system must replicate data… the possibility of failure, **even in the absence of the failure itself**, implies that the availability requirement requires some degree of data replication during normal system operation." ✅ That is the elegant part — replication is needed *because failure is possible*, and replication is what creates the latency cost. The trade-off exists before anything fails.

Three clarifications the paper itself insists on, all of which correct common usage:

1. **CAP describes a scenario, not a permanent state**: "CAP only posits limitations in the face of certain types of failures, and does not constrain any system capabilities during normal operation." ✅
2. **The partition branch of CAP is not where most systems live**: "network partitions are somewhat rare, and are often less frequent than other serious types of failure events in DDBSs." ✅
3. **`PC` does not mean strongly consistent**: "PC does not indicate that the system is fully consistent; rather it indicates that the system does not reduce consistency **beyond the baseline consistency level** when a network partition occurs — instead, it reduces availability." ✅ This is the subtle one, and it is exactly the misreading §5 exists to prevent: a system can be `PC/EC` and still be weaker than serializable, because its *baseline* is weaker.

### 4.5 The Other Impossibilities, and Their Hatches

| Forbidden | Source | Escape hatch real systems use | What it costs |
| --- | --- | --- | --- |
| Guaranteed termination of consensus in a fully asynchronous system with ≥1 crash fault | FLP, JACM 32(2), 1985 ✅ | **Partial synchrony** (DLS, JACM 35(2), 1988 ✅) — timeouts and leaders; randomised protocols (a separate family, ⚠ not verified this pass) | Liveness depends on an unverifiable timing assumption; a partition stalls the minority |
| Atomic consistency **and** availability during a partition | Gilbert–Lynch, SIGACT News 33(2), 2002 ✅ | Choose: **CP** (reduce availability — minority refuses) or **AP** (reduce consistency — accept, reconcile later) | CP: the minority is unavailable. AP: conflicts must be reconciled, and reconciliation is application work (§3.7) |
| Serializability with high availability under partition | Bailis et al., *Highly Available Transactions: Virtues and Limitations*, **PVLDB 7(3), VLDB 2014** ✅ — "serializable transactions — the gold standard of traditional ACID databases — are not achievable with high availability in the presence of network partitions" ✅ | **Highly Available Transactions (HATs)**: weaker isolation levels that *are* achievable — Read Committed, causal consistency with sticky availability, monotonic reads, and others | "highly available systems are **fundamentally unable to prevent concurrent updates to shared data items and cannot provide recency guarantees for reads**" ✅ — i.e. no lost-update prevention, no read-your-writes |
| Snapshot Isolation / Repeatable Read with high availability | Bailis et al., VLDB 2014 ✅ — "besides serializability, Snapshot Isolation and Repeatable Read isolation are not HAT-compliant" | Weaker levels, or coordination for the specific conflicting operations | Must detect conflicts between concurrent updates, which "we show is unavailable" ✅ |
| Byzantine-fault-tolerant consensus with fewer than 3f+1 processes | Lamport/Shostak/Pease, TOPLAS 4(3), 1982 ✅ (threshold family); Malkhi & Reiter, *Byzantine Quorum Systems*, 1998 ⚠ (§15) | 3f+1 replication, or accept a weaker failure model | Four replicas to tolerate one fault; message complexity in the PBFT family (§6.4) |

Two facts anchor this table in operational reality. First, the availability of weaker guarantees is not a compromise nobody would choose — the HAT paper's own summary of the benefit is measured: "HATs offer a **one to three order of magnitude latency decrease** compared to traditional distributed serializability protocols" ✅. Second, the discipline's honest overall verdict comes from Gilbert–Lynch's closing line: "**most real-world systems today are forced to settle with returning 'most of the data, most of the time'**" ✅ — which is a design position, not a failure.

## 5. The Consistency Spectrum

> **Why this section exists:** `linearizab*` appears in nine files in this repository, and almost never with a definition. Product documentation uses "strong consistency" for at least four different guarantees. This section is the antidote.

### 5.1 Strongest to Weakest, Defined

The definitions below are the standard ones from the primary literature. The single most important structural fact: **consistency of single objects and isolation between transactions are different axes**, and most confusion comes from collapsing them.

| Guarantee | Precise definition | Axis | Primary source |
| --- | --- | --- | --- |
| **Linearizability** | There exists a total order on all operations such that each operation appears to take effect atomically at a single instant **between its invocation and its response**, and the order respects real time: if op1 completes before op2 begins, op1 precedes op2. | **Single-object, real-time** | Herlihy & Wing, *Linearizability: A Correctness Condition for Concurrent Objects*, **ACM TOPLAS 12(3):463–492, July 1990** ✅ |
| **Strict serializability** | Serializability **plus** the real-time constraint of linearizability — the serial order must respect the order of non-overlapping transactions. | Multi-object, real-time | The combination; Spanner's *external consistency* "is equivalent to linearizability" ✅ (OSDI 2012) |
| **Serializability** | The concurrent execution of transactions is equivalent to *some* serial execution of those transactions. **No real-time constraint** — a serializable history may order transactions arbitrarily as long as some serial order explains the result. | Multi-object, transactions | Papadimitriou, *The Serializability of Concurrent Database Updates*, 1979 ⚠ (§15) |
| **Snapshot Isolation (SI)** | Each transaction reads from a **consistent snapshot** taken at its start; a transaction commits only if no concurrent committed transaction wrote the same data it wrote (first-committer-wins). | Multi-object, transactions | Berenson et al., *A Critique of ANSI SQL Isolation Levels*, **Proc. ACM SIGMOD 1995, pp. 1–10, San Jose, June 1995** ✅ — SI is *defined in this paper*: "An important multiversion isolation type, Snapshot Isolation, is defined" ✅ |
| **Repeatable Read** | A transaction's re-reads of rows it has already read return the same values. | Multi-object | ANSI SQL-92, as critiqued in Berenson et al. ✅ |
| **Causal consistency** | Operations that are causally related are seen by every node in the same order; concurrent operations may be seen in different orders. | Cross-object, causal | Achievable as a HAT ✅ (Bailis et al., 2014) |
| **Session guarantees** (read-your-writes, monotonic reads, monotonic writes, writes-follow-reads) | Per-client-session guarantees about what that client will observe. | Per-session | Terry et al., *Session Guarantees for Weakly Consistent Replicated Data*, **PDIS 1994** ✅ |
| **Eventual consistency** | If updates stop, all replicas eventually converge. | Weakest | Formally: Eventual Consistency (EC) = eventual delivery + convergence + termination ✅ (Shapiro et al., SSS 2011) |
| **Strong Eventual Consistency (SEC)** | EC **plus** *strong convergence*: replicas that have delivered the same updates have **equivalent state immediately**, with no conflict-resolution step. | Weakest, but deterministic | Shapiro et al., **SSS 2011, pp. 386–400** ✅ — "A solution to the CAP problem, Strong Eventual Consistency" |

Two notes that correct common usage:

- **Snapshot Isolation is not serializable, and the paper that defined it says so.** Berenson et al.: SI "avoids the ANSI SQL phenomena, **but is not serializable**" ✅. It nonetheless eliminates dirty reads, non-repeatable reads and phantoms — which is why it *feels* strong and is so often described as such.
- **"Strong consistency" is not a term with a technical definition.** When a vendor writes it, the reader should substitute one of the rows above and then ask which. §5.4 gives the procedure.

### 5.2 The Commonly Conflated Pairs

This table is the most directly useful artifact in the section. Each row is a pair I have seen treated as synonymous in a design review or a vendor deck.

| Pair often conflated | Why they are different | The consequence of confusing them |
| --- | --- | --- |
| **Linearizability vs serializability** | Linearizability is about **single objects** and honors **real time**; serializability is about **multi-object transactions** and does **not**. | A system described as "serializable" may reorder non-overlapping transactions in ways a client's wall-clock intuition forbids. A system described as "linearizable" may have no multi-object atomicity at all. |
| **Serializability vs strict serializability** | Strict serializability adds real-time ordering to serializability. | A "serializable" system can produce a history that no external observer would find consistent with the order they experienced — and this is legal. |
| **Snapshot Isolation vs serializability** | SI permits **write skew**: two transactions read overlapping data, write disjoint data, and both commit ✅ (Berenson et al.) | Balances, limits and invariants enforced by a read-then-write check are **not protected** under SI. This is the classic "our invariant broke under SI" incident. |
| **CAP's C vs ACID's C** | CAP's C is **atomic/linearizable consistency for a single object**; ACID's C is *application-defined invariant preservation*. Gilbert–Lynch are explicit that atomic consistency "has a different meaning than the *Atomic* in ACID, as it subsumes the database notions of both *Atomic* and *Consistent*" ✅ | "We're CP, so we're ACID-consistent" is a category error. |
| **PACELC's `PC` vs strongly consistent** | Abadi: "PC does not indicate that the system is fully consistent; rather it indicates that the system does not reduce consistency beyond the **baseline** consistency level when a network partition occurs" ✅ | A `PC/EC` system with a weak baseline is still weak, and the label hides it. |
| **Causal consistency vs linearizability** | Causal preserves only the **causal** partial order; linearizability imposes a **total** order consistent with real time. | A causally consistent store can return a value that is stale with respect to real time but consistent with causality — and a dashboard built on it can show a "time-travelling" number without any bug. |
| **Read-your-writes vs linearizability** | Read-your-writes is a **per-session** guarantee, achievable without coordination (Bailis et al. classify it as a HAT ✅); linearizability is **global**. | A system with read-your-writes can show user A their own update while user B sees the old value for minutes. That is usually fine, and it is not "strong consistency". |
| **Eventual consistency vs "converges quickly"** | EC has **no bound** on the convergence time — it is a liveness property with no latency guarantee. | Treating EC as "converges in milliseconds" makes every downstream SLA assumption unfounded. |
| **Eventual Consistency vs Strong Eventual Consistency** | EC permits replicas to diverge until they run a **conflict-resolution** step; SEC requires equivalent state from the same updates, immediately ✅ | EC requires a merge function that must itself be correct and deterministic; SEC removes that requirement, but only for data types whose operations are genuinely commutative or form a semi-lattice. |
| **"At least once" vs "exactly once"** | Different scopes of the same words (§9). | §9 exists because of this row. |

### 5.3 The Isolation-Anomaly Catalogue

The ANSI SQL-92 standard defined isolation levels by **prohibiting phenomena**, and Berenson et al. showed that the prohibition is too weak — the ANSI definitions "fail to characterize several popular isolation levels, including the standard locking implementations of the levels" ✅. The anomalies worth knowing, because each one is a real business-process failure:

| Anomaly | Mechanism | Which level permits it | Business shape it takes |
| --- | --- | --- | --- |
| **Dirty read** (ANSI P1) | Reading data a concurrent transaction has written but not committed ✅ | Read Uncommitted | Acting on a value that is later rolled back |
| **Non-repeatable / fuzzy read** (ANSI P2) | Re-reading an item yields a different value because another transaction committed in between ✅ | Read Committed | A validation step and its use disagree |
| **Phantom** (ANSI P3) | A re-run of a predicate read returns a different **set** of rows ✅ | Repeatable Read (under some implementations) | A "no existing record" check passes twice and two records are created |
| **Lost update** | Two transactions read-modify-write the same item; one's write is overwritten | Read Committed and weaker | A decrement performed once, counted twice — or vice versa |
| **Write skew** | Two transactions read an overlapping set, write **disjoint** items, and both commit — violating a cross-item invariant | **Snapshot Isolation** ✅ (Berenson et al. introduced it to describe SI's gap) | Two withdrawals each check the balance, each sees enough funds, both succeed, the balance goes negative |
| **Read skew / inconsistent read** | Different items read at different logical times, showing a state that never existed | Read Committed | A transfer observed mid-flight: debit applied, credit not |
| **Causal violation** | A read observes an effect without its cause | Any level weaker than causal | A reply visible before the message it replies to |

The **operational** point: the anomaly catalogue is the justification for weak isolation, not an argument against it. Bailis et al. found that weak isolation is "overwhelmingly the default setting in these stores and is often the only option offered" ✅, and that most of the weaker models are achievable with full high availability — while "none of the achievable models prevents concurrent modifications" ✅. So the decision is: **if your business process requires preventing concurrent modification of shared data, that operation must be coordinated**, and no amount of product configuration removes the requirement.

### 5.4 Which Level Does the Business Process Actually Need?

The question is never "how strong can we afford?" It is **"which anomalies would be a defect in this process?"** The procedure:

1. **Write the invariant in one sentence.** Not "the balance must be correct" — "*sum of ledger entries = stated balance*", or "*a payment is authorised at most once*".
2. **Ask whether any permitted anomaly can violate it.** If yes, that operation needs the level that forbids the anomaly, or an explicit coordination step.
3. **Ask which operations touch shared mutable state.** Only those need the stronger level. In most systems this is a small minority of operations — which is why Bailis et al. can recommend "a combination of HAT and (ideally sparing use of) non-HAT isolation levels" ✅.
4. **Ask what the invariant is worth.** A trading position is not a page-view counter.
5. **Design the fallback for when the strong level is unavailable.** If the answer is "writes stop", say so and size the blast radius. This is the coordination-avoidance decision of §11.2.

Worked shape, which §12 instantiates:

| Operation | Invariant at risk | Type in §5.3 | Level genuinely needed |
| --- | --- | --- | --- |
| Append an immutable event | None (monotonic, no conflict) | — | Anything; idempotency matters more than isolation (§9) |
| Read a customer's own profile | Staleness only | — | Read-your-writes ✅ — a session guarantee, no coordination |
| Authorise a payment against a daily limit | No double-spend of the limit | Lost update / write skew | Linearizable or serializable on the limit counter — **coordination** |
| Update a display name | Concurrent writes; last wins is acceptable if declared | Lost update (accepted) | Any, **if** the team has written down that it accepts loss |
| Enforce a uniqueness constraint across partitions | No duplicate key | Phantom across shards | Coordination, or a deterministic key-derived placement (§7.6) |

Row 4 is the one that gets skipped. "Last write wins" is a legitimate answer — the illegitimate part is that nobody wrote it down, which is §13.2.

### 5.5 The Honest Statement

**Most systems that claim strong consistency claim something narrower than their readers assume.** Four specific ways this happens:

1. **The claim is scoped to a single object or a single shard.** A store with linearizable single-key operations and no multi-key transactions is not "strongly consistent" in the sense most readers mean — it is linearizable per key, and cross-key reads can observe inconsistent states (§5.2 row 1).
2. **The claim is about the default configuration, but a component was configured otherwise.** Read replicas with asynchronous lag are the most common instance: the primary is serializable, the replica the dashboard reads is not, and the dashboard is what someone acts on.
3. **The claim is `PC` in the PACELC sense** — consistency is not reduced *below its baseline* during a partition, which says nothing about the baseline ✅.
4. **The claim was never tested under partition.** This is the most common and the most dangerous, and it has its own anti-pattern (§13.9). A consistency claim that has not been exercised with a partition injected is a hypothesis, not a property — and the experiment that would test it is a chaos experiment, owned by `technology/chaos_engineering_guide.md`.

## 6. Consensus

> **Why this section exists:** `raft` appears in 22 files in this repository, `paxos` in 10, `quorum` in 36. Consensus is almost always *named* and rarely *costed*. The trade-off is the content of this section — there is no universally correct family.

### 6.1 The Problem, Stated Once

Consensus is: a set of processes each proposes a value; all non-faulty processes must decide on a **single** value, that value must be one that was actually proposed (non-triviality), and no two processes may decide differently (agreement). Its practical instance is the **replicated state machine**: order the commands, feed them to a deterministic state machine, and every replica computes the same state. Consensus's role is to agree on **log position `i`**. Everything else in this section is an implementation choice around that.

One consequence deserves its own sentence, because it is an engineering obligation rather than a product feature: **the state machine must be deterministic.** If replica A and replica B process command 42 with `now()`, a random draw, or iteration order over a hash map, they diverge — with perfect consensus and a correct log. This is the same determinism discipline that `technology/deterministic_engineering_guide.md` owns for testing, applied here as a *correctness* requirement.

### 6.2 The Classical Family

| Algorithm | Primary source | Contribution | What it costs |
| --- | --- | --- | --- |
| **Paxos (Synod)** | Lamport, *The Part-Time Parliament*, **ACM TOCS 16(2):133–169, May 1998** ✅ | The original: a set of acceptors agreeing on one value, tolerating a minority of failures, with safety independent of timing. Lamport notes the paper "was first submitted in 1990, setting a personal record for publication delay" ✅ | Notoriously hard to read; the failure mode of the *paper*, not the algorithm |
| **Paxos Made Simple** | Lamport, **ACM SIGACT News 32(4), Whole Number 121, December 2001, pp. 51–58** ✅ | The same algorithm in plain prose; Lamport wrote it because "at the PODC 2001 conference, I got tired of everyone saying how difficult it was to understand the Paxos algorithm" ✅ | Does not cover multi-instance (log) Paxos, which is what production systems need |
| **Multi-Paxos** | The *Part-Time Parliament* / *Paxos Made Simple* extension: a long-lived leader driving instances in sequence | A long-lived leader removes a phase per decision — the practical reason Paxos is usable at all | Leader election is outside the algorithm; the "made simple" paper defers it |
| **Fast Paxos** | Lamport, **Distributed Computing 19(2), October 2006, pp. 79–103** ✅ | Learning in **two** message delays instead of three, at the cost of more processes. Motivated by a lower bound: "an algorithm that can make progress despite f faults and can achieve consensus in two message delays despite e faults requires more than 2e+f processes" ✅ | More acceptors; a **conflict** costs an extra round — "Fast Paxos can take 3 message delays in the event of conflict, when two values are proposed concurrently" ✅ |
| **Cheap Paxos** | Lamport & Massa, **DSN 2004, Florence, June–July 2004** ✅ | "tolerates up to f failures with **f+1 main processors and f auxiliary ones**" ✅ — the auxiliaries need not participate in every decision | Requires a **configuration master** to summon the auxiliaries; the fault-tolerance guarantee is weaker in the window before they are used |
| **Vertical Paxos** | Lamport, Malkhi & Zhou, *Vertical Paxos and Primary-Backup Replication*, **PODC 2009** ✅ (author PDF dated 9 February 2009, corrected 26 August 2009 and 10 March 2019) | Makes **reconfiguration part of the algorithm**, with read/write quorums and an auxiliary configuration master. Explicitly bridges the gap between consensus algorithms and real replication protocols: "The gap is not accidental; the abstract models for defining the classic consensus algorithms do not fully capture the requirements from those distributed systems" ✅ | An external configuration master is itself a replicated state machine — a second system to build and operate |
| **Viewstamped Replication** | Oki & Liskov, **PODC 1988** ⚠ (§15) | Independently arrived at the same result as Paxos, framed as primary-backup with view numbers | Not verified this pass; treated as historically parallel rather than cited for a specific claim |
| **Paxos Commit** | Gray & Lamport, *Consensus on Transaction Commit*, **ACM TODS 31(1):133–160, 2006** ✅ | Reframes transaction commit as a set of consensus instances, making it non-blocking. Lamport's own account: "we figured out that **Two-Phase Commit is the trivial version of Paxos Commit that tolerates zero faults**" ✅ | More message rounds and more state than 2PC in the failure-free case; the payoff is that the coordinator's failure no longer blocks (§10.1) |

**Quorum arithmetic** is the part that gets mis-set. For `n` acceptors tolerating `f` crashes, the classical requirement is `n ≥ 2f+1`: a majority of 3 tolerates 1; a majority of 5 tolerates 2. Two adjacent configurations must intersect — which is why Vertical Paxos insists on overlapping read/write quorums ✅ and why Raft's membership change uses a joint consensus: the USENIX abstract states Raft "includes a new mechanism for changing the cluster membership, which uses **overlapping majorities** to guarantee safety" ✅.

### 6.3 Raft and Why It Displaced the Classical Family in Practice

**Ongaro & Ousterhout, *In Search of an Understandable Consensus Algorithm*, USENIX ATC 2014, Philadelphia, pp. 305–319, June 2014** ✅ — awarded **Best Paper** ✅ (verified from the USENIX proceedings record).

The authors' own framing is precise about what changed and what did not: "Raft is a consensus algorithm for managing a replicated log. It produces a result **equivalent to (multi-)Paxos**, and it is **as efficient as Paxos**, but its structure is different from Paxos; this makes Raft more understandable than Paxos and also provides a better foundation for building practical systems." ✅

So Raft is **not** a stronger algorithm. It is a **reorganisation** of the same problem with three deliberate design moves, each addressing an engineering rather than a theoretical difficulty:

1. **Decomposition.** "Raft separates the key elements of consensus, such as leader election, log replication, and safety" ✅ — which makes each independently specifiable and testable.
2. **Stronger coherence, fewer states.** It "enforces a stronger degree of coherency to reduce the number of states that must be considered" ✅. This is a real trade: Raft constrains *how* leadership can move (a new leader must hold all committed entries) to shrink the space of behaviours an implementer must reason about.
3. **A specified membership-change mechanism.** "Raft also includes a new mechanism for changing the cluster membership, which uses overlapping majorities to guarantee safety." ✅

The paper's evidence for understandability was **empirical**: "Results from a user study demonstrate that Raft is easier for students to learn than Paxos." ✅ That is a legitimacy claim of a specific kind, and it is worth being precise about: **the reason Raft displaced Paxos in industrial use is not that it is more correct — it is that more engineers implement it correctly.** Implementability is a first-class system property, and the primary source says so in its own abstract.

### 6.4 The Byzantine Family

**Castro & Liskov, *Practical Byzantine Fault Tolerance*, 3rd USENIX Symposium on Operating Systems Design and Implementation (OSDI 99), New Orleans, February 1999** ✅ (extended version: *ACM Transactions on Computer Systems*, 2002 ⚠ — the OSDI 1999 record is verified; the TOCS extension's volume and pages are from recollection and are listed in §15).

PBFT solves a **different problem**, and this is the most important sentence in the subsection: crash-fault consensus defends against processes that **stop**; Byzantine consensus defends against processes that **continue while behaving arbitrarily**. The paper's own motivation is precise about why anyone would care: "We believe that Byzantine-fault-tolerant algorithms will be increasingly important in the future because malicious attacks and software errors are increasingly common and can cause faulty nodes to exhibit arbitrary behavior." ✅ Note that it names *software errors* alongside attacks — the failure class is wider than "adversary".

What PBFT contributed was not the Byzantine bound but the **practicality**:

- The bound itself is classical: agreement withstanding `f` Byzantine processes requires `n ≥ 3f+1` (the Lamport/Shostak/Pease threshold family, TOPLAS 1982 ✅).
- What was missing was viability: "**Whereas previous algorithms assumed a synchronous system or were too slow to be used in practice**, the algorithm described in this paper is practical: it works in asynchronous environments like the Internet and incorporates several important optimizations that improve the response time of previous algorithms by **more than an order of magnitude**." ✅
- And the measured claim, from the abstract: "We implemented a Byzantine-fault-tolerant NFS service using our algorithm and measured its performance. The results show that our service is only **3% slower than a standard unreplicated NFS**." ✅

**The price, stated plainly:** three times the replication for the same fault tolerance (`3f+1` versus `2f+1`) — four replicas to survive one Byzantine fault — and message complexity that is quadratic in the number of replicas in the classic formulation. **The judgement:** for a closed system inside one administrative domain with controlled hardware, the crash-fault model is usually the right one, and paying `3f+1` for a threat you have not modelled is waste. Byzantine tolerance earns its cost when the trust boundary is real: an open, permissionless network; a system spanning mutually distrusting administrative domains; or a threat model that explicitly includes insider processes. **Choose the failure model first, then the family — not the reverse.**

### 6.5 Leader Election, Log Replication and Membership Change — Conceptually

- **Leader election.** Consensus algorithms need a distinguished proposer for liveness, not for safety. The mechanism is a term/epoch number plus a quorum of votes for that term: a candidate increments the term, requests votes, and wins on a majority. A **higher term always wins**. The safety property is *at most one leader per term*, achieved purely by quorum intersection — which is why the quorum arithmetic of §6.2 is not a configuration detail but the load-bearing structure.
- **Log replication.** The leader appends to its log and replicates; a follower accepts an entry once the leader's term is known. The critical rule in Raft's formulation is that **an entry is committed only when it is stored on a majority**, and a new leader must hold every committed entry. The consequence for readers: **a replied-to write and a durable write are different events unless you are told otherwise.** This is why a write acknowledgement from a consensus system means "durable on a quorum", and why `acks=all` in Kafka means something narrower still (§9).
- **Membership change.** Removing or adding a voter changes what a majority *is*, and a naive switch can create **two disjoint majorities** in adjacent configurations — each of which can elect a leader. The two established mechanisms are **joint consensus** (the intermediate configuration requires majorities of both old and new sets — Raft's "overlapping majorities" ✅) and Vertical Paxos's **external configuration master** with intersecting read/write quorums ✅. The operational rule: **never change cluster membership by editing configuration files and restarting nodes.** That is a partition you created on purpose.

### 6.6 Operational Realities — Quorum Arithmetic, Latency, and When Not to Use Consensus

**Quorum arithmetic that silently loses fault tolerance.** The formula is not "more than half of *what I expected*", it is "more than half of *what is currently in the voter list*":

| Cluster size | Majority required | Failures tolerated | The mistake that happens |
| --- | --- | --- | --- |
| 3 | 2 | 1 | Deploying across 3 zones in one region and calling it geo-redundant |
| 4 | 3 | **1** | **The common one:** believing a 4-node cluster tolerates 2 failures. It tolerates 1, and tolerates it *less* gracefully than 3 nodes because there is no spare majority. |
| 5 | 3 | 2 | Placing 3 of 5 voters in one datacentre — that datacentre's failure is now a majority loss |
| 5 with 1 witness | 3 | 2 (for votes) | A witness carries votes but not data: losing it can block writes (§1.5) |
| 6 | 4 | **2** | Same class of error: 6 tolerates 2, not 3 |

Three rules follow. **(i) Use odd numbers** — even sizes buy nothing in fault tolerance and add a node to every quorum. **(ii) Place voters by failure domain, not by convenience** — a quorum that can be lost with one datacentre is a quorum you have not really replicated. **(iii) Re-count after every membership change**, because the denominator moved.

**The latency cost.** Every consensus decision costs at least one round trip to a majority — and if the majority spans data centres, that is a WAN round trip on **every write**. This is the `ELC` term of PACELC made concrete ✅. The practical consequences:

- Co-locate a majority in one region and pay a WAN trip only for cross-region durability, or accept WAN latency on every write. There is no third option.
- **Batch and pipeline.** Consensus cost is per *decision*, not per byte — which is why production systems pipeline and batch log entries. Lamport's own note on Spanner: "Our implementation of Paxos is **pipelined**, so as to improve Spanner's throughput in the presence of WAN latencies" ✅ (OSDI 2012).
- **Reads need not always pay it.** Spanner's read-only transactions are lock-free and read at a timestamp without a consensus round: "**reads access state directly from the underlying tablet at any replica that is sufficiently up-to-date**" ✅.

**When not to use consensus.** Consensus is the correct answer to "we must agree on one order". It is the wrong answer to most other questions:

| Situation | Consensus? | Better |
| --- | --- | --- |
| Ordering immutable events per entity | Often no | A single writer per entity, or a log partition keyed by entity — the ordering is already local (§11.2) |
| Commutative operations (counters, sets with clean semantics) | No | A CRDT: "Replicas of any CRDT are guaranteed to converge in a self-stabilising manner, despite any number of failures" ✅ (Shapiro et al., SSS 2011) |
| Idempotent retries of a request | No | An idempotency key (§9.4) — the duplicate is harmless, so no agreement is needed |
| Deciding a value that a single owner already decides | No | A single writer. Consensus is how you *elect* that writer, once |
| Two writes to a human-visible field where either outcome is acceptable | No | Last-write-wins — **provided you write it down** (§13.2) |
| Preventing a double-spend or enforcing a uniqueness invariant | **Yes** | Consensus, or a design that removes the need (deterministic placement, §7.6) |

The last row is what consensus is actually for, and it is a small fraction of most systems.

## 7. Replication and Partitioning

### 7.1 The Three Topologies, as Choices

| | **Single-leader** | **Multi-leader** | **Leaderless** |
| --- | --- | --- | --- |
| **Writes go to** | One node per partition | Any of several designated leaders | Any replica the client chooses |
| **Ordering** | Total, per partition, by construction | Per-leader only; **cross-leader order undefined** | None globally; per-key order via version/quorum |
| **Conflict on concurrent write** | Impossible (one writer) | **All writes accepted; detected on replication** — needs resolution (LWW, vector clocks + app merge, CRDT) | Same as multi-leader, detected at read by comparing versions |
| **Failure behaviour** | Leader loss = unavailability until election | Local writes continue during partition; **reconciliation after heal is on you** | Reads/writes continue if quorum reachable; minority side loses availability |
| **Write latency** | Synchronous: one RTT to a follower. Asynchronous: local | Local — the reason to choose it | Depends on R and W (§7.2) |
| **Consistency achievable** | Serializable / linearizable, with consensus | Causal at best without coordination; convergent with CRDTs | Tunable, but not linearizable without quorum reads+writes *and* a serialiser |
| **Its characteristic failure** | Election storms; the two-generations problem (§8.5) | **Silent conflict, resolved arbitrarily** | Read repair and anti-entropy cost; quorum mis-set (§13.5) |
| **What it is for** | Any invariant that must not be violated | Multi-region write locality where conflicts are genuinely mergeable or genuinely rare | Availability-first key-value workloads |

The decision is not "which is best". It is: **does my write path have an invariant that concurrent writes could violate?** If yes, single-leader (or consensus). If no, the other two buy you latency — and you owe the team a written statement of how conflicts resolve.

This is the same three-way structure Abadi reduces to in the PACELC paper: "there are only three alternatives for implementing data replication: the system sends data updates to all replicas at the same time, to an agreed-upon master node first, or to a single (arbitrary) node first" ✅ — and each implies a different consistency/latency position.

### 7.2 Synchronous vs Asynchronous — the Durability/Latency Trade

| Mode | The write is acknowledged when | Durability on ack | Latency | Failure consequence |
| --- | --- | --- | --- | --- |
| **Asynchronous** | The leader has written locally (and maybe to a log) | **None on other replicas** | Local write only | Leader loss can lose acknowledged writes — an RPO > 0 that must be stated |
| **Synchronous (quorum)** | A quorum has stored it | Survives minority loss | One RTT to a majority — WAN if the quorum spans regions | A quorum partition blocks writes (correctly) |
| **Semi-synchronous** | A *subset* has stored it (often one synchronous follower) | Survives that specific failure | One RTT to one follower | Loses writes if both the leader and that follower fail together |

Two things that are commonly mis-set:

- **"Quorum" in a leaderless store is not a majority.** The rule is `R + W > N` for read/write intersection, and it is a *weaker* condition than majority-based agreement: it prevents stale reads for a single key, but it does **not** give linearizability, does not order operations, and does not survive the failure patterns a majority does. Treating `R+W>N` as synonymous with `2f+1` is a category error with its own anti-pattern (§13.5).
- **`min.insync.replicas` and `acks=all` are two settings that only work together.** `acks=all` with a `min.insync.replicas` of 1 is *not* durable replication. The mechanism belongs to `technology/kafka_virtualization_guide.md`; the semantic point is that the acknowledgement describes the *broker's* durability and not the *consumer's* effect (§9.2).

### 7.3 Partitioning Strategies

| Strategy | Mechanism | Strength | Failure |
| --- | --- | --- | --- |
| **Range** | Partition by key ranges | Range scans and ordered reads are cheap; a natural fit for time-series and ledger data | **Hot partitions**: all writes for "today" land on one partition |
| **Hash** | Partition by `hash(key) mod N` | Even spread; removes locality-based hotspots | **Resharding is catastrophic**: changing `N` moves almost every key |
| **Consistent hashing** | Place nodes and keys on a ring; a key belongs to the next node clockwise | A node joining or leaving moves only its neighbour's keys — the reason it exists | Uneven distribution with few nodes; needs rebalancing logic |
| **Consistent hashing with virtual nodes** | Each physical node appears as many points on the ring | Smooths distribution; a departing node's load spreads across many | More metadata; still leaves the hot-**key** case unsolved |

The distinction that matters operationally: **virtual nodes fix hot *nodes*, not hot *keys*.** If a single key receives a large share of traffic, no partitioning scheme helps, because the key is indivisible. The fixes for a hot key are different in kind: give it a dedicated partition; split the *value* into sub-keys with an aggregation step; or add a write-local buffer and accept a delay. (Placement and rebalancing at the storage layer are `technology/cephfs_alternatives_guide.md`'s territory; shard-key selection and balancer mechanics are adjacent to `technology/oracle_sharding_guide.md`.)

### 7.4 Rebalancing and Hot Partitions

**Rebalancing** is moving partition ownership between nodes, and it is a *distributed coordination problem in its own right*. Three rules:

1. **Movement must be a consensus operation, not a configuration edit.** The ownership map must not be editable from two places at once (§6.5).
2. **Transfer in a way the reader can explain.** A hand-off that requires a quiescent partition converts a rebalance into an outage for the keys on it.
3. **Rate-limit it.** Rebalancing competes with serving traffic for the same network and disk; an unbounded rebalance during peak is an outage you scheduled.

**Hot partitions** come from four distinct causes, and the fix depends on which: **key skew** (a celebrity key — no partition scheme fixes it), **range locality** (time-ordered keys — salt the key or hash the prefix), **request-pattern skew** (one client hammering one key — cache or apply back-pressure), and **rebalancing-induced skew** (an uneven ring — virtual nodes). Misdiagnosing the cause is common: teams add nodes for a hot key and nothing improves, because the key is atomic.

### 7.5 Secondary Indexes Across Partitions

A secondary index answers "find all rows where `x = v`", and if the data is partitioned by `id`, the index the query needs is not the index the storage is organised by. Two strategies, with their costs:

| Strategy | How | Cost |
| --- | --- | --- |
| **Local (document-partitioned) index** | Each partition indexes only its own data | A query for `x = v` must **scatter to every partition** and merge — latency proportional to partition count, and availability equal to the *least* available partition |
| **Global (term-partitioned) index** | The index itself is partitioned by the indexed term | A **write** must now update two differently-partitioned structures — i.e. a distributed write. If it must be atomic, that is 2PC or a saga (§10) |

This is the point at which an innocent "we'll just add a secondary index" becomes a design decision with a distributed-systems cost. The global index makes reads cheap and writes expensive-and-distributed; the local index makes writes cheap and reads expensive-and-fragile. **Neither is free, and the choice should be recorded.**

### 7.6 The Cross-Partition Operation That Ruins the Plan

Every partition plan has an operation that does not fit, and it is usually discovered in production. The anatomy is always the same:

1. The system is partitioned by a key chosen for even distribution — say `customer_id`.
2. A business requirement arrives that spans two customers: *"transfer between accounts"*, *"no two accounts may share a tax ID"*, *"total exposure across the group must not exceed X"*.
3. That requirement is an **invariant over two partitions**. There is no order shared between them, so no local check can enforce it.
4. The result is a distributed transaction **smuggled in later** — usually as a sequence of local transactions with a compensating step written by hand, i.e. an ad-hoc saga with no coordinator, no compensation log and no failure testing.

The two honest outcomes: **make the operation local** (choose a partition key that keeps the invariant inside one partition — often possible, e.g. partition by `account_group` rather than `customer_id`, accepting a different skew), or **make the operation explicitly distributed** (consensus on a shared counter, or 2PC/saga with the machinery of §10). What is not available is a third option where the invariant holds and neither cost is paid.

The good news is that step 2 is often avoidable by choosing the partition key against the *invariants* rather than against the *data volume*. That is a design-time decision worth an hour of argument, and it is one of the highest-value uses of the invariant list in §11.1.

## 8. Failure Detection, Membership and Time

> **Why this section exists:** it is the one most likely to catch a live bug. The two-generations-of-leader failure (§8.5) is present in a large fraction of systems that use leases, and it is usually not a bug anyone has looked for.

### 8.1 Heartbeats and Timeouts — Always Wrong in One Direction

A heartbeat detector says "dead" when no heartbeat arrives within a timeout. It therefore has exactly two error modes, and §2.4 established that **both are unavoidable**:

| Setting | Consequence | Who notices |
| --- | --- | --- |
| **Timeout too short** | False positives: healthy-but-slow nodes are declared dead. Failover fires; two nodes may act. | Users, as flapping and duplicated work |
| **Timeout too long** | False negatives: a dead node is not detected, and the system stalls waiting for it. | Users, as an outage with a clean dashboard |

There is no value that is right for both, because the decision it needs (alive versus dead) requires information the detector does not have. The engineering response is **not** to tune the timeout (though you must still choose one) but to ensure that **a wrong answer is safe**. Three properties make it so:

1. **The detector's output gates liveness, not safety.** A false "dead" must not permit two writers to commit. In a quorum system this holds by construction: the old leader may *believe* it is the leader, but it cannot commit without a majority, and the new leader's election proves the old one lacks one.
2. **Side effects must be fenced** (§8.5). Quorum safety protects the *log*; it does nothing for an external API call the old leader makes. This is the gap.
3. **Backoff and jitter must be present on any action triggered by the detector.** A timeout that triggers a retry storm finds the weakest node with the most load (§13.4).

Two practical notes. **Measure the timeout against a real distribution, not a mean** — the relevant statistic is a high percentile of observed round-trip times under load. And **a health check is not a timeout**: an endpoint that returns 200 because the process is alive is not evidence that the process is making progress. A node in a long stop-the-world GC pause answers nothing; one in a deadlock may answer everything.

### 8.2 Accrual Failure Detectors — the Tunable Trade-off

Chandra and Toueg's contribution was to make the detector's two error modes a **specifiable, tunable property** rather than an artefact of a magic number: *Unreliable Failure Detectors for Reliable Distributed Systems*, **Journal of the ACM 43(2):225–267, March 1996** ✅ (corroborated in the reference lists of both Kleppmann's 2016 post ✅ and SWIM ✅).

The core idea: a **failure detector** is an abstraction that may be wrong, and the useful way to characterise it is by its guarantees, not its implementation. The families are:

| Class | Guarantee | Reading |
| --- | --- | --- |
| **Perfect (P)** | No false positives, and every crash eventually suspected | Needs synchrony |
| **Eventually perfect (◇P)** | After some unknown time, behaves perfectly | The realistic target: correct *eventually*, unreliable in the meantime |
| **Strong (S)** | No false positives; some crashed process is eventually suspected *by every* correct process | |
| **Eventually strong (◇S)** | After some time, strong | **This is what real consensus protocols require** — it is why FLP is not fatal (§4.1) |

The **accrual** refinement changes the output from a **boolean to a suspicion level**: instead of "dead", the detector reports a continuous value derived from the observed distribution of inter-arrival times, and the application chooses the threshold. The tunable trade-off becomes explicit: **raise the threshold → fewer false positives, slower detection; lower it → faster detection, more false positives.** Moving the decision from the detector's configuration to the application's policy is the advance, because the application is the only component that knows which error it can survive. (The specific 1997 Aguilera–Toueg heartbeat paper is ⚠ unverified this pass; see §15.)

### 8.3 Gossip for Membership and Dissemination

The problem gossip solves is scale: all-to-all heartbeating imposes a "message load on the network and group that grows **quadratically** with the group size" ✅ (SWIM, verified). Two papers define the space:

- **Epidemic algorithms.** Demers, Greene, Hauser, Irish and Larson, *Epidemic Algorithms for Replicated Database Maintenance*, **Proc. 6th Annual ACM Symposium on Principles of Distributed Computing (PODC), 1987, pp. 1–12** ✅ (also published in *Operating Systems Review* 22(1):8–32, 1988 ✅ — both venues are real; cite the PODC one, note the journal version). This is the origin of the anti-entropy and rumour-mongering terminology, and the source of the property that makes gossip attractive: probabilistic, robust dissemination with no coordination.
- **SWIM.** Das, Gupta and Motivala (Cornell University), *SWIM: Scalable Weakly-consistent Infection-style Process Group Membership Protocol*, **DSN 2002** ⚠ (the paper PDF was extracted ✅; the DSN 2002 venue attribution is flagged in §15). SWIM's insights, all quoted or paraphrased from the paper itself ✅:
  - **Separate the two functions.** "SWIM separates the failure detection and membership update dissemination functionalities of the membership protocol." ✅ The reason: heartbeating conflates them, and the conflation is the source of the quadratic cost.
  - **Randomised probing instead of heartbeating.** "Processes are monitored through an efficient peer-to-peer periodic randomized probing protocol." ✅
  - **Constant per-member cost.** "Both the expected time to first detection of each process failure, and the expected message load per member, **do not vary with group size**." ✅ This is the scaling property.
  - **Suspicion to cut false positives.** "The rate of false failure detections in the SWIM system is reduced by modifying the protocol to allow group members to **suspect** a process before **declaring** it as failed — this allows the system to discover and rectify false failure detections." ✅ The three-state model (alive / suspect / dead) with a suspicion timeout is directly reusable in application-level designs.
  - **Piggybacked dissemination.** "Information about membership changes… is propagated via piggybacking on ping messages and acknowledgments." ✅ Dissemination costs nothing extra because it rides on traffic that is happening anyway.

The one caveat SWIM states about itself is the honest engineering note: it provides **weakly consistent** membership — "membership lists at different members need not be consistent across the group at the same (causal) point in time" ✅. So gossip membership is a **liveness and scale** mechanism, not a **safety** mechanism. For safety you still need quorum intersection, and the membership view gossip produces must not be the thing that decides who may commit.

### 8.4 Leases

**Gray and Cheriton, *Leases: An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency*, SOSP 1989** ✅ (verified via Kleppmann's citation of the paper for exactly the lease concept).

A **lease** is a lock with an expiry, granted for a bounded interval. Its appeal is that it converts an unbounded failure (a client that never releases) into a bounded one (the lease expires). Its cost is the assumption ledger's lease row (§2.5): **the mechanism only works if the holder can measure its own expiry, and the holder's clock and the holder's own execution are the two things a distributed system cannot guarantee.**

The mechanism starts on a monotonic clock (§3.3) and is still not sufficient. The failure mode is not clock drift. It is **the holder not running**:

> "**You cannot fix this problem by inserting a check on the lock expiry just before writing back to storage. Remember that GC can pause a running thread at any point, including the point that is maximally inconvenient for you (between the last check and the write operation).**" — Kleppmann, *How to do distributed locking*, 8 February 2016 ✅

Kleppmann's enumeration of the pauses is the practically useful part: a stop-the-world GC pause (observed lasting "**several minutes**" in HBase's case, which he documents ✅); a page fault; a synchronous read from a network-backed block device; CPU contention; and `SIGSTOP`. He also notes that **network delay is not bounded either**: in a documented GitHub incident "packets were delayed in the network for approximately **90 seconds**" ✅, so "an application process may send a write request, and it may reach the storage server a minute later when the lease has already expired."

The consequence, stated exactly: **a lease gives you mutual exclusion in the happy path and a two-writer window otherwise.** The lease defines who *should* hold the resource. It does not prevent anyone from *acting*.

### 8.5 Fencing — the Two-Generations-of-Leader Problem, Walked Through

This is the failure that fencing exists to solve, and it is worth walking through concretely because the sequence is counter-intuitive: **every step is correct, and the outcome is still corruption.**

**The setup.** A single-leader replication group. The leader holds a **lease** for 10 seconds, renewed on a timer. Writes to the shared resource (a file, a row, an external counter) are performed by the leader. The lease is intended to guarantee at most one leader acts. All components are correct as written.

**The sequence.**

| # | Time | What happens | Why it looks fine |
| --- | --- | --- | --- |
| 1 | T+0.0 s | **Leader L1** acquires the lease. Term becomes 11. | Correct election |
| 2 | T+2.0 s | L1 begins a read-modify-write on the shared resource: read the value, compute, write. | Normal operation |
| 3 | T+2.1 s | A **stop-the-world GC pause** starts in L1's process. L1 stops executing — but is not dead, and its process is not gone. | Nothing is logged; no error is raised |
| 4 | T+3.0 s | Followers stop receiving heartbeats. Their timeout fires. They conclude L1 is dead (§8.1 — it is indistinguishable from slow). | **A correct, unavoidable inference** |
| 5 | T+3.1 s | **Leader L2** is elected. Term becomes **12**. L2 acquires the lease and begins its own work on the same resource. | Correct: the lease had expired by the followers' clocks, and the new term is higher |
| 6 | T+3.2 s | L2 performs its own read-modify-write on the resource and writes. Term 12's write lands. | Correct |
| 7 | T+4.5 s | **The GC pause ends.** L1 resumes — at the instruction after the read, before the write. Its code has not re-checked anything, and even if it did, a check cannot close the window between the check and the write ✅. | **This is the instant the system breaks** |
| 8 | T+4.6 s | L1 completes its write. Its lease has expired by wall-clock time, but L1 has no reliable way to know it and proceeds. | **There are now two generations of leader, and both have written.** L2's committed write is silently overwritten by a stale one. |

The durable damage: L2's write is lost, and worse, the resource now reflects a computation performed by a leader that had already been deposed. If the resource is a ledger, the entries are inconsistent. If it is an assignment of work, the work was done twice. **No component behaved incorrectly.** The failure was that **the lease was treated as a guarantee of exclusion, when it only defines a permission that the holder cannot reliably evaluate.**

**The fix — fencing tokens.** Kleppmann's solution, and it is the correct one:

> "The fix for this problem is actually pretty simple: you need to include a **fencing token** with every write request to the storage service. In this context, a fencing token is simply a number that increases (e.g. incremented by the lock service) every time a client acquires the lock." ✅

Applied to the sequence above: L1's write carries token **11**; L2's write carries token **12**. The resource — not the client — rejects any write whose token is lower than the highest it has seen. At step 8, L1's write arrives with token 11, the resource has already seen 12, and the write is **rejected**. Corruption avoided.

**Three properties that make it work, and the ways it is implemented wrongly:**

1. **The token must be monotonically increasing across acquisitions** — which means the *lock service* issues it, and it must survive the lock service's own failover. In the sequence above the term number is already exactly such a token; systems that have a term/epoch have the token for free and often do not use it.
2. **The resource must validate it.** This is the property that is missed. "Fence" is often implemented as a client-side check: the client reads the current token, compares, and proceeds. **That is not a fence** — the client is the component whose execution cannot be trusted to reach the write (§8.4). **If the storage system cannot be made to check the token, there is no fence.**
3. **The check and the write must be atomic at the resource.** A resource that checks the token and then writes in a separate step has reintroduced the same window one layer down — but now it is a *local* window on a single system, which is a much smaller problem and usually closable.

**Where this appears in real systems.** Spanner is the instructive example, because it solves the lease-expiry problem *properly* and shows what it costs. Its Paxos implementation "supports long-lived leaders with **time-based leader leases**, whose length defaults to **10 seconds**" ✅, and it uses TrueTime to make lease intervals provably disjoint — a proof that relies on each new leader's quorum overlapping the previous leader's quorum, and on TrueTime's *bounded* uncertainty. The published argument is a chain of inequalities over `TT.now()`, `TT.after()`, `TT.before()` and a **single-vote rule**: a replica will not grant another lease vote until `TT.after(t_end)` is true, and the lease is deemed expired when `TT.before(min_r(v_leader) + 10)` is false ✅. The `min_r(v_leader_i,r) + 10` construction is the fence, expressed as a time interval rather than a counter, and its soundness depends entirely on TrueTime's bound ✅. **The general lesson: you can replace a fencing token with tightly bounded time *only if you have bought the clock infrastructure that makes the bound real.** Otherwise you have the sequence above.

**The check for your own system.** Three questions, and a "no" to the second is a bug:

1. Can any component hold a resource based on a lease or a timeout?
2. Does every write to that resource carry a **monotonically increasing token**, and does **the resource** reject stale ones — as opposed to the client checking?
3. If the answer to (2) is "we use a lease and the client checks it": **what happens if that client's process is paused for longer than the lease** — and can you say, from your logs, whether it ever has?

## 9. Delivery Semantics, Idempotency and Exactly-Once

> **Why this section exists:** this is the highest-risk class of claim in the field. It is where vendor marketing diverges most sharply from what is achievable, and where a false belief costs money the same day. The treatment below is deliberately blunt.

### 9.1 The Three Modes, and What Each Actually Promises

Three phrases are used across every messaging product. Kafka's documentation states them in one line each, and it is worth quoting verbatim because it is the cleanest statement in the industry **and it is the vendor's own**:

> - *At most once* — "Messages may be lost but are never redelivered."
> - *At least once* — "Messages are never lost but may be redelivered."
> - *Exactly once* — "Each message is processed once and only once." ✅

Read the third line again and notice the shift in noun. The first two are statements about **messages**; the third is a statement about **processing**. That shift is the entire problem, and most "exactly-once" disagreements in a design review are one person reading the message-scope and the other reading the effect-scope.

| Mode | The literal promise | The failure it permits | What it costs | Who is responsible for the gap |
| --- | --- | --- | --- | --- |
| **At most once** | Delivered 0 or 1 times | **Silent loss** | Nothing — it is the cheapest | Nobody notices until reconciliation |
| **At least once** | Delivered 1 or more times | **Duplicates** | Retries, plus an idempotent consumer | The **application** |
| **Exactly once** | Processed once | *Nothing, within the stated boundary* — and the boundary is the whole question | Transactions, ordering, latency | Whoever drew the boundary |

The asymmetry that matters: **at-least-once and at-most-once are both achievable end to end by a simple protocol. "Exactly-once delivery" is not.** The reason is not an implementation gap; it is a result.

### 9.2 Why End-to-End Exactly-Once *Delivery* Is Not Achievable

The argument is three lines and it is decisive.

1. A sender that wants to be sure its message arrived needs the receiver to **acknowledge**.
2. The acknowledgement is itself a message over the same unreliable channel, so *it* needs an acknowledgement.
3. You either terminate the regress at some finite number of messages — in which case the last sender is permanently uncertain whether its final message arrived — or you never terminate.

Therefore: **any protocol that uses finitely many messages over a channel that may lose them leaves the sender unable to distinguish "the message was lost" from "the acknowledgement was lost."** The sender has exactly two available policies — resend (duplicates are possible) or do not resend (loss is possible) — and no third policy exists. This is the classic *coordinated attack* / *two-generals* problem, and it is the reason "exactly-once delivery" is a category error rather than a hard engineering target.

**Citation caution, stated because this guide audits its own claims.** The two-generals result is folklore-canonical and its attribution is routinely sloppy. The formal analysis is usually credited to Akkoyunlu, Ekanadham and Huber, *Some Constraints and Tradeoffs in the Design of Network Communications*, **SOSP 1975** ⚠ — **I did not verify this record this pass**, and I am therefore not citing it as a proven source; it is listed in §15. Jim Gray's *Notes on Data Base Operating Systems* (1978) ⚠ is the other common attribution, also unverified here. The **argument** above stands on its own and needs no citation; only the **attribution** is flagged.

**What *is* achievable is exactly-once *effects*** — and it is a different property, achieved by a different mechanism. The rest of this section is about that property.

### 9.3 What Vendors Actually Claim, Quoted and Bounded

Every exactly-once claim in the industry is **true within a stated boundary**, and the boundary is where the engineering lives. Three quotes, each from the vendor's own documentation, each dated to this pass:

**Kafka, on the industry's claims generally** — and note that this is the vendor warning you about other vendors, which makes it unusually credible:

> "Many systems claim to provide 'exactly-once' delivery semantics, but it is important to read the fine print, because sometimes these claims are misleading (i.e. they don't translate to the case where consumers or producers can fail, cases where there are multiple consumer processes, or cases where data written to disk can be lost)." ✅ — Apache Kafka documentation, *Message Delivery Semantics*, extracted 2026-09-18

**Kafka, on what its own exactly-once covers** — the scope statement is explicit and it ends at the system boundary:

> "Kafka supports exactly-once delivery in Kafka Streams, and the transactional producer and the consumer using read-committed isolation level can be used generally to provide exactly-once delivery when reading, processing and writing data on Kafka topics. **Exactly-once delivery for other destination systems generally requires cooperation with such systems**, but Kafka provides the primitives which makes implementing this feasible." ✅ — same source

**Confluent, making the boundary a sentence of its own:**

> "Also, note that many systems claim to provide exactly once delivery semantics, but this might not always what you think it is. For example, these system may not account for cases where a **consumer or producer outside of the system** has failed." ✅ — Confluent documentation, *Message Delivery Guarantees for Apache Kafka*, extracted 2026-09-18

That is the pattern every reader should internalise, and it generalises past Kafka:

| The claim | What it actually covers | Where the boundary is | Who it does **not** protect |
| --- | --- | --- | --- |
| "Exactly-once within Kafka" | Produce-to-topic, consume-from-topic, offset commit — all inside the broker | The Kafka cluster's own topics | The payment API the consumer calls between read and commit |
| "Exactly-once in Flink" | Operator state + input-stream position, restored from a checkpoint | The dataflow's stateful operators, given a **rewindable source** | A sink that is not transactional or idempotent |
| "Idempotent producer" | Duplicate *entries in the log* are prevented | The producer→broker leg only | Duplicate *effects* downstream of the topic |
| "No message loss" (`acks=all`, ISR) | The message is committed once all in-sync replicas have applied it, and is not lost while one replicating broker is alive | **Broker durability** | Anything the consumer does after reading it |

The last row is the one that costs the most money, and §1.2 already names it: **`acks=all` answers the broker-durability question and tells you nothing about the end-to-end delivery question.**

### 9.4 The Mechanics That Make Exactly-Once *Effects* Achievable

Five mechanisms; each is a different way of making a duplicate harmless, and each has a cost and a failure mode.

| Mechanism | How it works | Cost | Its own failure mode |
| --- | --- | --- | --- |
| **Idempotency key** | The client generates a unique key per logical operation; the server records key → result and returns the stored result on any repeat | A durable key store on the effect side; a uniqueness constraint | **The retention window.** Stripe's documentation is explicit: keys "can be removed... after they're at least 24 hours old. We generate a new request if a key is reused after the original is pruned." ✅ A retry after the window is a **new operation** |
| **Deduplication window / seen-set** | The consumer keeps the set of processed message IDs and skips known ones | State that grows; you must bound it | "Bounded" means a retry older than the window is processed again. **The window length is a correctness parameter, not a tuning knob** |
| **Transactional coupling of read and process** | The consumer's position and its output are committed in the **same** transaction | Requires a transactional broker *and* a transactional sink | Only works when both sides are in the same transactional system |
| **Store the offset with the output** | Put the consumer position in the same datastore as the produced result, so "data and offsets are updated, or neither is" ✅ (Kafka's own framing) | The output store now has a schema the framework controls | Only possible if you own the output store |
| **Deterministic replay from a checkpoint** | Snapshot state + stream position; on failure, restore and replay | Checkpoint overhead and recovery time | Requires a **rewindable source** — Flink states the precondition plainly: "For this mechanism to realize its full guarantees, the data stream source (such as message queue or broker) needs to be able to rewind the stream to a defined recent point" ✅ |

Two verified details worth keeping, because they are where the guarantees are thinner than the headline:

- **Flink's exactly-once is checkpoint-and-replay, not transaction.** Its documentation states the mechanism and its limit together: "A streaming dataflow can be resumed from a checkpoint while maintaining consistency *(exactly-once processing semantics)* by restoring the state of the operators and replaying the records from the point of the checkpoint" ✅ — and where checkpoint **alignment is skipped** for latency, "On a restore, these records will occur as duplicates" ✅. The snapshot mechanism derives from Chandy–Lamport ✅ (Flink's docs say so directly).
- **Kafka's transactional boundary is the producer, not the consumer.** Kafka's own words: "In Kafka, the consumer and producer are separate, and it is only the producer which is transactional. It is however able to make transactional updates to the consumer's position (confusingly called the 'committed offset'), and it is this which gives the overall exactly-once behavior." ✅ The trick is that the offset write rides in the producer's transaction; a side effect outside Kafka does not.

The **transactional outbox** pattern is the generalisation of row 4 for systems that are not Kafka: write the state change and the outgoing event in one local ACID transaction, then a relay publishes. Its thorough treatment belongs to `technology/monolith_to_microservices_guide.md` and to the event-streaming guides; the semantic point here is only that it converts a distributed write into a local one plus an at-least-once relay — which is exactly the composition §9.5 recommends.

### 9.5 The Conclusion

**Effectively-once effects come from idempotency and transaction design, not from the transport.** Stated as a design rule:

1. **Assume the transport gives you at least once.** It is the only mode that does not silently lose data, and the only one you can rely on end to end.
2. **Make the effect idempotent, and put the idempotency where the effect happens** — a uniqueness constraint on the effect side, not a check on the caller's side (§8.5's fence rule, applied to duplicate suppression rather than to stale writes).
3. **Write down the deduplication window** and treat it as a correctness parameter: state the maximum retry age your system actually has (retry budget × backoff, plus any operator replay window, plus any queue redelivery delay), and confirm the window exceeds it.
4. **Do not ask a vendor for exactly-once delivery.** Ask which boundary their exactly-once covers, and then check whether your effect sits inside it. Usually it does not.
5. **Cross-reference rather than re-derive the mechanism:** `technology/kafka_virtualization_guide.md` and `technology/event_stream_processing_guide.md` own Kafka's ISR/`acks`/transaction machinery, `technology/kafka_alternatives_guide.md` the alternatives, and `technology/deterministic_engineering_guide.md` owns deterministic replay as a mechanism. This section owns only the semantics.

## 10. Distributed Transactions and Their Alternatives

> **This section is deliberately condensed and cross-referenced.** `technology/apache_seata_guide.md` owns the saga / TCC / AT pattern space **in depth** — including isolation countermeasures and recovery — and this section does not re-derive it. What follows is the *decision structure*: what each mechanism can and cannot promise, and how to choose.

### 10.1 Atomic Commit and Its Blocking Failure Mode

Two-phase commit is a **prepare** phase (each participant votes, and by voting yes it surrenders the right to decide unilaterally) followed by a **commit/abort** phase (the coordinator broadcasts the decision). The correctness of the protocol comes from that surrender: a participant cannot unilaterally abort after voting yes, because others may have committed.

**The blocking failure mode, stated as the price of that correctness.** The moment a participant has voted yes, it holds locks and waits for a decision. If the coordinator fails *after* some participants have voted yes but *before* any has learned the outcome, the participants are in doubt and **must wait** — they cannot abort (someone may have committed) and cannot commit (someone may have aborted). That waiting period is unbounded, and locks are held throughout it.

Helland's position paper states the operational verdict directly: 2PC protocols "can easily block when nodes are unavailable", and he likens platforms that promise global serializability to the **Maginot Line** — bypassed in 1940, and in the distributed case bypassed by developers, because "the projects founder because the performance costs and fragility make them impractical" ✅ (*Life beyond Distributed Transactions: an Apostate's Opinion*, CIDR 2007, pp. 132–141, verified from the paper PDF; note that this is an explicitly labelled **position paper** — "the positions expressed in this paper are personal opinions", in the author's own words ✅ — so it is a practitioner's argument, not a theorem).

**The non-blocking direction.** §6.2 already records the resolution, and it is the reason this guide puts consensus before transactions: Gray and Lamport showed that commit *is* a consensus problem, and Lamport's own summary is the sharpest available — 2PC is "the trivial version of Paxos Commit that tolerates zero faults" ✅ (*Consensus on Transaction Commit*, ACM TODS 31(1):133–160, 2006). You can make commit non-blocking by running a consensus instance per participant decision; the price is more rounds and more state in the failure-free path, paid to remove an unbounded stall in the failure path.

### 10.2 Three-Phase Commit — and Why It Is Not the Answer

Three-phase commit adds an intermediate **pre-commit** phase so that, under a crash fault with bounded delay, a surviving participant can complete the protocol without the original coordinator. Two things must be said precisely, because 3PC is routinely over-credited:

- It removes blocking **for crash faults under a synchrony assumption**. To distinguish a crashed coordinator from a slow one, participants need a bound on message delay — the same assumption §4.1's escape hatch requires.
- It does **not** solve the partition case. Under a partition, a group that cannot reach the coordinator cannot know whether it will return, and an incorrect completion can produce divergence; the protocol's guarantees are bound to the synchrony assumption, exactly as FLP implies they must be.

**The judgement:** an extra round trip on *every* commit, in exchange for a narrower blocking window that still depends on an unverifiable assumption, is a poor trade against consensus-based commit, which pays its cost only in the failure path. 3PC's primary source is Skeen's 1981 work ⚠ — **not verified this pass**; listed in §15.

### 10.3 Sagas and Compensation — and Their Semantic Limits

The saga was defined in Garcia-Molina and Salem, *Sagas*, **Proc. ACM SIGMOD 1987, pp. 249–259** ✅ (verified from the paper PDF; ACM ISBN record `0-89791-236-5/87/0005/0249` visible in the paper's own notice). A saga is a long-lived transaction broken into sub-transactions T₁…Tₙ, each with a compensating transaction Cᵢ, and the guarantee is:

> "The database management system guarantees that either all the transactions in a saga are successfully completed or compensating transactions are run to amend a partial execution." ✅

Three limits, all of them from the paper itself, and the third is the one that decides financial designs:

1. **Compensation is semantic, not physical.** "The compensating transaction undoes, from a *semantic point of view*, any of the actions performed by Tᵢ, but does not necessarily return the database to the state that existed when the execution of Tᵢ began." ✅
2. **It is not nested atomicity.** A saga "is like a nested transaction, except that (a) a saga only permits two levels of nesting… and (b) at the outer level **full atomicity is not provided**". ✅
3. **Sagas provide no isolation, and the paper is explicit:** "Note that other transactions might see the effects of a partial saga execution. When a compensating transaction Cᵢ is run, **no effort is made to notify or abort transactions that might have seen the results of Tᵢ before they were compensated for by Cᵢ**." ✅

Limit 3 is the one that breaks payment-adjacent designs, and it has a name in practice: **intermediate states are visible.** A saga does not make a transfer atomic; it makes it *eventually repaired*. Between Tᵢ and Cᵢ a downstream reader can observe a state that the invariant was supposed to forbid — and that reader may have already acted (sent a notification, released goods, filed a report). This is why §5.3's "causal violation" row and §12's worked example both come back to the same question: **which intermediate states are observable, and who is allowed to act on them?**

**What cannot be compensated.** The honest list, and it should be written into the design document:

- **An effect at another institution.** A payment settled in a scheme's books is compensated by a *new, opposite payment*, not by erasure. The compensator needs its own funding, its own authorisation, and it can fail.
- **A fee, a spread, or a rate.** Compensation cannot restore an FX rate, an interest accrual, or a fee already charged by a third party.
- **A physical or legal effect.** Goods shipped, a contract formed, a message a human has read.
- **A notification.** You can send a correction, not an un-send.
- **Anything a downstream party has already committed to** on the strength of the intermediate state — which is limit 3 restated as a business risk.

**The pattern ownership is not this guide's.** `technology/apache_seata_guide.md` owns saga / TCC / AT in depth, including the isolation-anomaly countermeasures that address limit 3 within a transaction framework, and it was revised in a prior window against the SIGMOD 1987 source. `technology/temporal_workflow_guide.md` owns the durable-execution framing — compensation expressed as code with durable state and deterministic replay. Cite those by name; do not re-derive them.

### 10.4 CRDTs — for Genuinely Commutative Conflicts

Where §10.1–§10.3 all pay to *order* operations, conflict-free replicated data types remove the need to order: replicas converge because the merge is a join on a semi-lattice, so any delivery order produces the same state. §5.1's row for Strong Eventual Consistency is this property — "replicas that have delivered the same updates have **equivalent state immediately**, with no conflict-resolution step" ✅ (Shapiro, Preguiça, Baquero and Zawirski, *Conflict-free Replicated Data Types*, **SSS 2011, pp. 386–400**).

The precondition is **semantic, not technical**, and this is where CRDTs get over-applied. You must be able to state, in business terms, that *both orders produce an acceptable state*. Counters that must not go negative, balances, and unique allocations usually fail that test. Sets of seen documents, presence flags, append-only event collections, and monotonically increasing aggregates usually pass it. The test is the same invariant test as §5.4 — and if a CRDT's merge produces a state your invariant forbids, the CRDT was the wrong tool, not a bug in the CRDT.

### 10.5 The Decision Rule

| Situation | Mechanism | What it costs | What you must write down |
| --- | --- | --- | --- |
| All-or-nothing across two partitions, and every step is semantically reversible | **Saga with compensation** | No isolation; visible intermediate states; compensations can fail | Which intermediate states are observable, and the compensator's failure policy |
| All-or-nothing, participants support prepare, blocking is acceptable | **2PC** | Unbounded stall and held locks when the coordinator fails in the decision window | The in-doubt policy and the lock-hold alarm |
| All-or-nothing, blocking is not acceptable | **Consensus-based commit** | An extra consensus group to run and upgrade | The quorum's failure domains |
| Concurrent operations genuinely commute | **CRDT / semi-lattice** | State growth; a merge that must be correct and deterministic | The statement that both orders are business-acceptable |
| A single logical effect that may be retried | **Idempotency key** (§9.4) | A durable key store; a retention window | The window, and the retry budget it must exceed |
| The invariant could be made local by choosing a different partition key | **Nothing — change the key** | A different skew profile | That you considered it before choosing a mechanism |

The rule in one line: **prefer removing the need for a distributed transaction over choosing a mechanism for one.** §7.6 is the highest-leverage version of that preference — a partition key chosen against the *invariants* rather than against the data volume eliminates the cross-partition operation entirely.

## 11. The Engineering Judgement

> **Why this section exists:** §§1–10 are the discipline's theory. This is the practice — and the practice is mostly about making decisions explicit rather than about choosing better algorithms.

### 11.1 Writing Down the Invariants

Every section so far has needed the same artefact: a list of the invariants the system must preserve. It is the input to the partition-key choice (§7.6), the consistency-level choice (§5.4), the coordination decision (§11.2) and the anti-pattern review (§13). A usable invariant has five fields, and the fifth is the one that gets skipped:

| Field | What it holds | Bad version | Good version |
| --- | --- | --- | --- |
| **Statement** | One sentence, checkable | "The balance must be correct" | "`Σ ledger entries = stated balance`" |
| **Scope** | Which data, which partitions | "The account" | "All entries for one `account_id` — one partition" |
| **Anomaly it forbids** | From the §5.3 catalogue | "Inconsistency" | "Lost update on the daily-limit counter" |
| **Mechanism** | What enforces it | "The database" | "Linearizable compare-and-set on the limit row" |
| **Behaviour when the mechanism is unavailable** | This one | *(absent)* | "Writes are rejected; the API returns 503; the limit is not exceeded" |

The fifth field is what makes the list operational: it turns "we rely on the database" into "we stop taking payments when the quorum is unreachable, and here is the alarm." A design review that produces no fifth column has not reviewed the failure behaviour — it has reviewed the happy path.

### 11.2 Coordination Required vs Avoidable

The central practical question is not *which* consistency level, but **whether this operation needs coordination at all**. The framing comes from the highly-available-transactions line of work already cited in §4.5: some guarantees are achievable without coordination, and the achievable set is larger than intuition suggests. The 2015 follow-up work on *coordination avoidance* (Bailis et al., PVLDB 2015 ⚠ — **not verified this pass**, listed in §15) formalises the test as *invariant confluence*: an invariant is coordination-free for a set of transactions if divergent, unreconciled states can always be merged without violating the invariant.

The practical version of that test, which needs no paper:

> **Can two replicas independently apply different operations and then merge, without any query to the other, and still satisfy the invariant?**

- **Yes** → coordination-free. Use a CRDT, a session guarantee, or simply separate writers per key. This is where the availability is.
- **No, but the invariant only binds a subset of operations** → coordinate only those operations. This is §5.4 step 3, and in most systems it is a small minority of the traffic — which is why the HAT paper can recommend "a combination of HAT and (ideally sparing use of) non-HAT isolation levels" ✅.
- **No, and the invariant binds every path** → the operation is inherently coordinated. Say so in the design, size the blast radius, and stop looking for a way around it. There isn't one; §4.5 is the proof.

The failure mode this prevents: teams that have heard "avoid coordination" and therefore *describe* a coordinated operation as coordination-free — usually by omitting the cross-partition check from the design and adding it, unannounced, in a spring release (§13.7).

### 11.3 The Cost Model of Coordination

Coordination is not free and it is not binary. Three costs, all real, all differently visible:

| Cost | Shape | Where it shows up | Illustrative magnitude (fictional, for shape only) |
| --- | --- | --- | --- |
| **Latency** | One round trip to a quorum per coordinated decision; a WAN trip if the quorum spans regions | Tail latency on the coordinated path; a p99 that is a function of geography | ~0.5 ms same-DC; ~30–80 ms cross-region, per decision — and **per decision**, so a 5-decision flow pays it 5× |
| **Availability** | The operation is unavailable whenever a quorum is unreachable | Incidents that present as "the database is down" when it is the network | A 3-node quorum in 3 failure domains tolerates 1 failure; a 2-domain split stops writes entirely |
| **Operational complexity** | A consensus group to deploy, monitor, upgrade, back up, and reason about during an incident | Runbooks, on-call load, upgrade risk, and the cost of being wrong in a reconfiguration (§6.5) | One more stateful system with its own failure modes and its own 03:00 page |

The asymmetry that makes the decision tractable: **uncoordinated designs are cheap and unavailable-to-nobody; coordinated designs are expensive and correct.** So the optimisation is not "reduce the cost of coordination" — it is **"reduce the count of coordinated operations"** (§11.2), which is a design-time activity, not a runtime one.

### 11.4 Designing for the Failure You Cannot Detect

§2.4 and §8.1 established the uncomfortable fact: the failure you cannot detect is the one that matters, because a process that is paused rather than dead is indistinguishable from a slow one, and the detector's answer is necessarily wrong in one direction. You cannot fix the detector. You can change what the detector's answer is *allowed to cause*. Four design rules, each traceable to a section:

1. **Make effects idempotent** so that a duplicate caused by a false "dead" is harmless (§9.4). This is the single highest-value rule, because it converts an unavoidable uncertainty into a no-op.
2. **Fence every external side effect** so that a stale writer's write is rejected *by the resource* (§8.5). Quorum safety protects the log; the fence protects the world.
3. **Require a quorum for any decision, and let the minority stall** (§6.6). The stalled minority is the *correct* behaviour under an undetectable failure — it is FLP's liveness concession being made deliberately rather than accidentally.
4. **Never require certainty about another node's state to make progress.** If completing a step needs to know whether a peer is alive, that step is fragile by construction. Restructure so that the answer only affects *when* work happens, not *whether* it is correct.

### 11.5 The Honest Admission

**Most distributed-systems failures in production are design decisions that were never made explicitly, rather than algorithms implemented wrongly.** This is not a rhetorical flourish; it is what the four sections above compose to. Concretely, the four shapes it takes:

- **A decision made by not deciding.** Last-write-wins was chosen because nobody specified conflict resolution (§13.2). The code is correct; the policy does not exist.
- **A decision deferred until production.** The cross-partition invariant was not in the design, so no partition key was chosen to contain it (§7.6, §13.7).
- **An assumption that was never stated.** The lease was treated as mutual exclusion (§8.4–§8.5) without asserting that the holder's execution is trustworthy — which no distributed system can assert.
- **A claim that was never tested.** "We have strong consistency" that has never been exercised under a partition (§5.5, §13.9). Untested, it is a hypothesis.

All four are review failures rather than engineering failures, which is why the fix is procedural: **make the decision list a deliverable** (§11.1), and **test the claims** rather than the code. The techniques for the testing half — fault injection, experiment design, blast radius, deterministic simulation — belong to `technology/chaos_engineering_guide.md` and `technology/deterministic_engineering_guide.md` and are not duplicated here; what this guide contributes to that work is the *list of things worth injecting*: a partition across the quorum, a pause longer than a lease, a retry that crosses the deduplication window, a duplicate delivered inside the transactional boundary, and a clock that steps backwards.

## 12. The Cymbal Bank Worked Example

> **Cymbal Bank is fictional.** It is the only bank persona used anywhere in this repository. Every figure below is **explicitly illustrative and fictional** — selected to show the *shape* of a cost comparison, not to represent any real institution, benchmark or survey. The design, the mistakes and the reasoning are the content; the numbers are scaffolding.

### 12.1 The Flow and the Decomposition

Cymbal Bank is decomposing **outbound instant payment initiation** — a corporate customer instructs a payment from one of their accounts to a beneficiary at another institution, subject to a contractual daily outbound limit. The decomposed design has five services, which is the shape that created the problem:

| Service | Responsibility | State |
| --- | --- | --- |
| **Payment API** | Accepts the instruction, authenticates, applies the idempotency key | None (stateless) |
| **Limits service** | Tracks the customer's daily outbound total against the contractual cap | A per-customer counter |
| **Ledger service** | Posts the debit against the account; maintains the balance from entries | Per-account entry log |
| **Scheme adapter** | Submits the credit transfer to the payment scheme; receives the settlement outcome | Position/status per instruction |
| **Notify + Report** | Tells the customer; feeds the regulatory reporting pipeline | Out of the money path |

The team's first instinct — standard for a microservices decomposition, and the reason `technology/monolith_to_microservices_guide.md` exists — was that each service owns its own datastore and they coordinate through events. That instinct is what exposed the problem: **a single payment must decrement the limit and debit the ledger, and the two now live in different consensus groups.**

### 12.2 The Invariants, Written Down First

Following §11.1, before choosing anything:

| # | Invariant | Scope | Anomaly that would break it | Mechanism | If the mechanism is unavailable |
| --- | --- | --- | --- | --- | --- |
| **I1** | `Σ ledger entries = stated balance`, per account | One account | Torn read across entries | Snapshot read within one partition | Read fails; no partial balance is ever served |
| **I2** | Daily outbound total ≤ contractual cap, per customer | One customer | **Lost update / write skew** on the counter | Serialized compare-and-set on the counter | Payments for that customer are **declined**, not queued |
| **I3** | An instruction produces **at most one** scheme submission | One instruction ID | Duplicate on retry | Idempotency key on `instruction_id`, enforced by the scheme adapter | Retries resolve to the stored result |
| **I4** | Every authorised payment eventually reaches the reporting feed | Reporting pipeline | Loss | Transactional outbox + at-least-once relay | Relay lags; the feed is late, and the lag is alarmed |
| **I5** | The submitting customer sees their own payment immediately | That customer's session | Stale read | Read-your-writes routing to the primary for that account | Falls back to the primary; never to a lagging replica |

I2 and I3 are the load-bearing pair. I2 is the one that attracts coordination; I3 is the one that lets you avoid ordering everything else.

### 12.3 The Failure Model, Chosen Explicitly

Per §2.1–§2.2, the model is chosen before the algorithms:

- **Crash-recovery**, not crash-stop. Every service persists state and restarts; nodes come back with their state and then rejoin. This is the honest model for a system with a database behind it, and it is stronger than crash-stop only in that recovery is possible — it does not make detection easier.
- **Not Byzantine.** The system is closed, inside one administrative domain, on controlled hardware, with no mutually distrusting participants. Paying `3f+1` (§6.4) for a threat that is not in the model would be waste — but note the model *is* written down, which is what makes that a decision rather than an omission.
- **Timing: partial synchrony** (§4.1's escape hatch). The system is synchronous enough, often enough, for timeouts to work most of the time. The consequence is accepted explicitly: **liveness depends on an assumption that cannot be verified from inside**, so an outage is the only disproof.
- **Ambiguous failure is the design driver** (§2.4). A paused scheme adapter cannot be distinguished from a dead one, which is why I3 is an at-most-once invariant enforced by an idempotency key rather than by a hopeful belief about the adapter's liveness.

### 12.4 The Ordering and Idempotency Requirements

- **Ordering needed:** the ledger's entries for one account must be totally ordered — the balance is a fold over them. Nothing outside an account needs a shared order.
- **Ordering *not* needed:** the limit decrement relative to the ledger post for a *different* customer; the notification relative to the scheme submission; the reporting feed relative to anything (it needs completeness, not order).
- **Idempotency required at four points**, and the shape matters: the API accepts an `instruction_id` and the client is expected to reuse it on retry; the limit counter is decremented by a *delta keyed to the instruction*, not by a "current value" write; the ledger post is keyed by `instruction_id` so a replay is a no-op; the scheme adapter stores the outcome keyed by `instruction_id` and returns the stored outcome on a repeat. **None of these depend on the transport's delivery mode** — which is §9.5's conclusion applied.

### 12.5 The Consistency Level Each Step Actually Needs

This table is the direct application of §5.4, and it is where the design got smaller:

| Step | Invariant at risk | Level genuinely needed | Coordination? |
| --- | --- | --- | --- |
| Accept the instruction | none | none | No |
| Check and decrement the daily limit | I2 — lost update / write skew | **Serializable / linearizable on the counter** | **Yes** |
| Post the ledger debit | I1 — torn read | Serializable **within the account partition** | Only intra-partition |
| Submit to the scheme | I3 — duplicate | none (idempotency key instead) | No |
| Acknowledge to the customer | I5 — stale read | Read-your-writes | No (primary read) |
| Notify | none (duplicate is acceptable) | eventual | No |
| Feed reporting | I4 — loss | eventual + at-least-once | No |

**One step needs global coordination.** The team had assumed five did, because five involved state. This is the single most valuable output of the exercise: the coordination count fell from an assumed five to one, and the reason is that four of the five requirements are *idempotency* requirements (solvable locally) rather than *ordering* requirements (which are not).

### 12.6 Replication, Partitioning, and the Key That Changed the Design

The limit and the ledger must both be updated for an invariant-bearing payment. Their natural keys differ: the limit is per **customer**, the ledger is per **account**. A customer can hold several accounts.

**The rejected plan.** Partition the ledger by `account_id` and the limits service by `customer_id`, then coordinate between them. This is a cross-partition operation (§7.6) on **every payment**, and it is exactly the shape that gets "smuggled in later" (§13.7).

**The plan chosen.** Partition **both** by `customer_id`, so that a customer's accounts and their limit counter are placed in the **same partition**, and a payment's limit decrement plus ledger post become **one local ACID transaction inside one consensus group**. The invariant I2 is no longer cross-partition at all — it is enforced by an ordinary transaction in an ordinary database, with Raft providing the partition's own durability and leader election.

**What that choice cost, stated honestly (this is the part that is usually omitted):**

| Cost | Consequence |
| --- | --- |
| **Skew** | A corporate group with thousands of accounts in one partition becomes a hot partition. Mitigation: a synthetic bucket inside the customer's partition, with the limit counter sharded within the customer's own consensus group |
| **Blast radius** | One Raft group now hosts both services' data for that customer; losing quorum for it stops both functions for that customer |
| **Migration** | Moving a customer to a larger shard later is a data movement with a freeze, not a routing change — an operational cost accepted up front |
| **Cross-customer operations** | A "group total exposure" constraint would still be cross-partition. The team checked, and the contract does not have one — so this cost was *avoided by knowing the invariants*, not by luck |

Secondary indexes across partitions (§7.5) follow the same reasoning: the beneficiary-name search used by operations is a **local** index on each partition with a scatter-gather, accepted because it is an operational query, not a money-path query — an explicit choice of read cost over a distributed write.

### 12.7 Failure Detection and the Fence

The one place Cymbal Bank writes outside its own transactional boundary is the **settlement file** handed to the scheme: committed ledger entries are exported by a single writer, and the scheme will reject a file containing a stale generation of entries.

- The writer holds a **lease** on the export responsibility, renewed on a timer.
- **The lease alone is insufficient**, per §8.5: a stop-the-world pause in the writer can outlive the lease, and the writer can resume and export. The team walked the eight-step sequence in §8.5 against their own design and found it applicable — because "every step is correct, and the outcome is still corruption."
- **The fence:** every export carries a **monotonically increasing generation number** issued by the consensus group, and **the scheme ingestion process rejects any file whose generation is lower than the highest already accepted**. The token comes from the Raft term/epoch the group already maintains (§8.5 point 1), which is why this needed no new mechanism — only the decision to use it.
- **The property that makes it a fence rather than a gesture:** the *receiver* validates. The writer never checks its own token, because the writer is the component whose execution cannot be trusted to reach the write (§8.5 point 2).

### 12.8 Alternatives Considered, with Their Trade-offs

| Alternative | Why it was considered | Why it was rejected (or kept) |
| --- | --- | --- |
| **2PC across the limits and ledger services** | It gives exactly the guarantee I2 needs, atomically | **Rejected.** Blocking failure mode with held locks (§10.1), an unbounded in-doubt window on the money path, and a coordinator to operate — for an invariant that the partition-key choice makes local for free |
| **Saga with compensation** (`limit decrement` and `ledger post` as two local transactions with a compensator) | No long-held locks; no coordinator in the happy path | **Rejected for the money path.** Compensation cannot un-submit a scheme instruction, cannot restore a fee, and — decisively — saga intermediate states are visible (§10.3, limit 3), which means a downstream reader could act on a payment that is about to be compensated |
| **A consensus-based global counter per payment** | Removes the partition-key constraint | **Rejected.** A consensus decision per payment adds a quorum round trip to every payment (§11.3) and adds a second stateful system, to buy back a constraint that co-location had already removed |
| **CRDT for the limit** | Coordination-free by construction (§10.4) | **Rejected on the invariant.** A limit that must never be exceeded is not mergeable: two independently decremented copies can each be within the cap while the merge is over it. The invariant is not I-confluent (§11.2) |
| **Idempotency key on the scheme submission** | Makes duplicate submission harmless | **Kept** — this is what allows the whole scheme leg to run without ordering or coordination |
| **Co-locate limit and ledger per customer partition** | Makes I2 a local transaction | **Chosen.** The cost is skew and blast radius (§12.6) — accepted, written down, and sized |

### 12.9 The Recommendation, and What Was Deliberately Left Distributed

**Recommended design.** Services are decomposed as in §12.1 but **share a consensus group per customer**, partitioned by `customer_id`. A payment whose limit and ledger effects fall in one partition commits as **one local transaction** inside that partition's Raft group. The scheme leg is idempotent by `instruction_id` and carries a fenced generation number. Notification and reporting are at-least-once, idempotent consumers fed from a transactional outbox.

**What the team chose NOT to make distributed — and why this is the most important line in the section:**

- **The limit and the ledger** were kept in one transactional domain, despite the decomposition instinct. They are one invariant's two halves; splitting them created a distributed transaction where none was required.
- **Notification** was deliberately left at-least-once and idempotent. Ordering notifications relative to ledger state would require coordination for a requirement nobody stated.
- **Regulatory reporting** was left eventually consistent with an alarmed lag, because I4 is a completeness invariant, not a consistency invariant. (Where a *reporting deadline* is a legal obligation, that obligation is a different invariant with its own mechanism — and it is a business-process design question, not a distributed-systems one.)
- **The customer status dashboard** reads from a lagging replica and shows a "as of" timestamp, because I5 is per-session and the dashboard is not the submitting session.
- **Analytics** was left entirely outside the transactional boundary — the one place where eventual consistency costs nothing at all.

**Cost comparison, illustrative and fictional (shape only):**

| Design option | Added latency on the money path | Availability exposure | New operational surface | Illustrative 5-year build+run |
| --- | --- | --- | --- | --- |
| 2PC across two services | +1 coord. RTT + lock hold | Payments block on the in-doubt window | A coordinator + in-doubt runbook | ~1.4 units |
| Saga across two services | no extra RTT | higher (no locks) | compensator + reconciliation + visible-intermediate-state analysis | ~1.3 units |
| Global consensus counter per payment | +1 quorum RTT per payment | payments stop without quorum | a second consensus group | ~1.5 units |
| **Co-located, one partition** | **none added** | **one group's quorum per customer** | **none new — one system fewer** | **~1.0 unit (baseline)** |

The baseline is the *uninteresting* option, which is the finding: after writing the invariants down, the cheapest design was also the most correct one. The three alternatives each add cost to remove a constraint that a partition key had already removed.

**And the honest caveat.** The co-located design is only available because of a *fact about the business*: Cymbal Bank's contract has a per-customer cap and no cross-customer group constraint. Had a group-exposure invariant existed, I2 would genuinely be cross-partition, and the team would have had to choose between the coordination options above — paying real cost for a real requirement. **Whether an invariant is coordination-free is a business fact, discovered by asking, not an architecture preference.**

## 13. The Gotchas and Anti-Patterns

> Each row is a failure the discipline predicts. The **symptom** is what you observe; the **cause** is which section explains it; the **guardrail** is what prevents it. The last two rows are the ones that survive code review, because nothing in the code is wrong.

### 13.1 The Unstated Invariant

- **Symptom.** An incident review cannot agree on whether the outcome was a bug. Two parties describe the same behaviour as "correct by design" and "a data-integrity violation".
- **Cause.** The invariant was never written down (§11.1), so the system's behaviour is judged against an unstated, and therefore differing, expectation.
- **Guardrail.** The five-field invariant table of §11.1, produced *before* the mechanism is chosen. If the review cannot point at the row, the row does not exist.

### 13.2 The Last-Write-Wins That Silently Discards Data

- **Symptom.** A field reverts. A customer's update "did not save". Nobody can reproduce it, because the overwrite is not an error — it is the conflict policy.
- **Cause.** LWW was chosen by omission, not by decision (§3.7, §7.1). With no decision on the record, the system's default became the policy. Under concurrent writes and clock skew (§3.2), LWW loses the update whose timestamp is lower — which is frequently the *later* real-world edit, because the clock that stamped it was behind.
- **Guardrail.** State which fields accept loss and which do not. The ones that do not need either a single writer per key, a version check, or a conflict-aware merge — not a timestamp.

### 13.3 The Timeout Mis-set in Both Directions

- **Symptom.** Either flapping failovers and duplicated work, or a clean dashboard during a total stall.
- **Cause.** §8.1: every timeout is wrong in one direction, and the direction is chosen by the setting. A timeout tuned from a mean RTT rather than a high percentile fails under load, exactly when it matters.
- **Guardrail.** Set the timeout from an observed high percentile under peak load; ensure a wrong answer is *safe* rather than trying to make it *correct* (§8.1 properties 1–3); and require the fence (§8.5) so that a false "dead" cannot produce a second writer.

### 13.4 The Retry Without Idempotency

- **Symptom.** A double charge, a double shipment, a duplicated ledger entry — always after an incident, never in testing.
- **Cause.** §9: at-least-once is what the transport gives you, so retries are a fact; the effect was not made idempotent, so a duplicate is a duplicate *effect*.
- **Guardrail.** An idempotency key enforced **where the effect happens**, not checked by the caller (§9.4); plus backoff with jitter so retries do not synchronise into a storm against the weakest node.

### 13.5 The Quorum Arithmetic That Accidentally Lost Fault Tolerance

- **Symptom.** The cluster fails at a *lower* failure count than the design assumed — a 4-node cluster goes read-only with two nodes down.
- **Cause.** §1.5 and §6.6: the majority is ⌊n/2⌋+1 of the *current* voter list, not of the intended one. Four nodes tolerate **one** failure, not two. And a five-node cluster with three voters in one datacentre has that datacentre as its single point of failure.
- **Guardrail.** Odd voter counts; voters placed by failure domain rather than availability zone; re-count after every membership change; and no configuration-file membership edits (§6.5).

### 13.6 The Clock Used for Ordering

- **Symptom.** An event appears to happen before its cause. A "later" write is overwritten by an "earlier" one. A certificate or report shows a timestamp that precedes the action it records.
- **Cause.** §3.1–§3.2: physical clocks cannot order distributed events — NTP steps, skew, and drift make the comparison meaningless, and a monotonic clock deliberately has no relation to wall time.
- **Guardrail.** Order with a logical or hybrid clock, or with the log's own order; use the wall clock only for human-readable display and never as a tie-breaker; audit every `ORDER BY timestamp` on a cross-node dataset (§3.7).

### 13.7 The Cross-Partition Transaction Smuggled In Later

- **Symptom.** A spring release adds a feature that "needs a bit more consistency". Writes to two partitions now happen without a coordinator, a compensation log, or a failure test.
- **Cause.** §7.6: an invariant over two partitions was discovered after the key was chosen, and the ad-hoc sequence of local writes is a saga that nobody called a saga — so nothing protects it.
- **Guardrail.** Re-run the invariant list whenever a requirement crosses a boundary; and treat "this spans two partitions" as a design trigger, not an implementation detail. The cheapest fix is usually to re-examine the partition key (§12.6).

### 13.8 The Leader Lease Without Fencing

- **Symptom.** A stale write lands after a failover. Nobody can explain it, because every component's behaviour was correct.
- **Cause.** §8.4–§8.5: the lease was treated as mutual exclusion, when it only defines a permission the holder cannot reliably evaluate. A paused process resumes and acts on a permission that expired.
- **Guardrail.** The three questions of §8.5, and specifically question 2: does **the resource** reject a stale token? A client-side check is not a fence.

### 13.9 The "We Have Strong Consistency" Claim That Was Never Tested Under Partition

- **Symptom.** The claim is true in the architecture diagram and false in an incident. Or true, but for one key rather than for the transactions everyone assumes.
- **Cause.** §5.5: the claim is scoped (single-key, single-shard, or `PC` above a weak baseline), or configured differently in one component (an asynchronous read replica), or simply unexercised.
- **Guardrail.** Substitute the specific guarantee from §5.1 for the phrase "strong consistency", write down which one your system actually provides and at what scope, then **inject the partition**. The experiment belongs to `technology/chaos_engineering_guide.md`; the assertion under test is the one in your §5.1 row.

## 14. The Claims Audit

**Verdict key:** ✅ **verified** this pass against a primary source; ⚠ **flagged** — asserted but not verified, or verified only through a secondary path; ❌ **rejected** — asserted somewhere and found false or unsupported. **Claim classes:** *R* = proven result (published theorem), *C* = engineering convention, *V* = vendor claim about its own product.

**Verification method and its limits, stated honestly.** This pass used **`web_extract` against primary URLs** — publisher and author records, conference proceedings pages, the papers' own PDFs, and product documentation. **`web_search` returned empty result sets for every query attempted**, so it was treated as unavailable and every verification below rests on a directly fetched URL; the empty searches are recorded as *empty*, not as negative evidence. The **ACM Digital Library returned HTTP 500 to the extractor on every attempt** (five URLs), so ACM-published venue/page details were checked against **author publication records or the paper's own PDF** wherever possible, and left ⚠ where not.

### 14.1 The Impossibility Results — the highest-risk class

| Claim | Class | Source | Quality | Date/venue | Verdict |
| --- | --- | --- | --- | --- | --- |
| Consensus cannot be *guaranteed to terminate* in a fully asynchronous system with ≥1 crash fault; no Byzantine faults, reliable message system, no clocks, no failure detector | R | Fischer, Lynch & Paterson, *Impossibility of Distributed Consensus with One Faulty Process* | Quoted from the paper's own model section | JACM 32(2):374–382, Apr 1985; PODS Mar 1983 | ✅ |
| Consensus *is* solvable under partial synchrony (eventually synchronous, or synchronous with unknown bounds) | R | Dwork, Lynch & Stockmeyer, *Consensus in the Presence of Partial Synchrony* | Publisher record | JACM 35(2):288–323, Apr 1988; PODC 1984 pp. 103–118 | ✅ |
| Atomic (linearizable) consistency and availability cannot both hold during a partition, in the asynchronous network model, even with no message loss | R | Gilbert & Lynch, *Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services* | Paper PDF; definitions and the Corollary quoted directly | ACM SIGACT News 33(2):51–59, Jun 2002 | ✅ |
| "Pick 2 of 3" is **Brewer's own** framing, from the 2000 PODC keynote | C | Brewer, *Towards Robust Distributed Systems*, PODC keynote slides | Slides fetched and quoted directly | PODC, 19 Jul 2000 | ✅ |
| The "2 of 3" framing is misleading; the trade-off binds only during a partition; partition handling must be designed explicitly | C | Brewer, *CAP Twelve Years Later* | **Not retrieved** — InfoQ returned a bot challenge, IEEE page not extracted | IEEE Computer 45(2):23–29, Feb 2012 | ⚠ |
| Availability as a continuum (*harvest* / *yield*) | R | Fox & Brewer, *Harvest, Yield, and Scalable Tolerant Systems* | Not retrieved this pass | HotOS 1999 | ⚠ |
| PACELC; the `ELC` half is driven by replication, which exists *because* failure is possible; `PC` does not mean strongly consistent | C | Abadi, *Consistency Tradeoffs in Modern Distributed Database System Design* | Quoted directly | IEEE Computer 45(2):37–42, Feb 2012 | ✅ |
| Serializable transactions are not achievable with high availability under partition; SI and Repeatable Read are not HAT-compliant | R | Bailis et al., *Highly Available Transactions: Virtues and Limitations* | Quoted directly | PVLDB 7(3), VLDB 2014 | ✅ |
| Coordination avoidance / invariant confluence | R/C | Bailis et al., *Coordination Avoidance in Database Systems* | **Not verified** | PVLDB 2015 | ⚠ |
| Agreement withstanding `f` Byzantine processes requires `n ≥ 3f+1` | R | Lamport, Shostak & Pease, *The Byzantine Generals Problem* | Publisher record | ACM TOPLAS 4(3):382–401, 1982 | ✅ |
| Byzantine quorum systems | R | Malkhi & Reiter, *Byzantine Quorum Systems* | Not verified this pass | 1998 | ⚠ |
| End-to-end "exactly-once delivery" over a lossy channel is impossible (coordinated attack / two generals) | R | Commonly credited to Akkoyunlu, Ekanadham & Huber (SOSP 1975), or Gray's *Notes on Data Base Operating Systems* (1978) | **Attribution NOT verified**; the argument in §9.2 is self-contained and does not depend on it | — | ⚠ |

### 14.2 The Consistency Spectrum and Isolation

| Claim | Class | Source | Quality | Date/venue | Verdict |
| --- | --- | --- | --- | --- | --- |
| Linearizability: a total order respecting real time, each operation atomic between invocation and response | R | Herlihy & Wing, *Linearizability: A Correctness Condition for Concurrent Objects* | Definition quoted | ACM TOPLAS 12(3):463–492, Jul 1990 | ✅ |
| Serializability is equivalence to *some* serial order, with no real-time constraint | R | Papadimitriou, *The Serializability of Concurrent Database Updates* | Not verified this pass | JACM 26(4), 1979 | ⚠ |
| ANSI SQL-92's phenomenon-based level definitions are inadequate; **Snapshot Isolation is defined here**, avoids the ANSI phenomena "but is not serializable" | R | Berenson, Bernstein, Gray, Melton, O'Neil & O'Neil, *A Critique of ANSI SQL Isolation Levels* | Definitions and quotes verified | Proc. ACM SIGMOD 1995, pp. 1–10, San Jose | ✅ |
| Generalized isolation-level definitions | R | Adya, Liskov & O'Neil, *Generalized Isolation Level Definitions* | Not verified this pass | ICDE 2000 | ⚠ |
| Session guarantees (read-your-writes, monotonic reads, monotonic writes, writes-follow-reads) | R | Terry, Demers, Petersen, Spreitzer, Theimer & Welch, *Session Guarantees for Weakly Consistent Replicated Data* | Venue verified | PDIS 1994 | ✅ |
| Eventual Consistency, Strong Eventual Consistency, and the CRDT convergence result | R | Shapiro, Preguiça, Baquero & Zawirski, *Conflict-free Replicated Data Types* | Quoted ("equivalent state immediately") | SSS 2011, pp. 386–400 | ✅ |
| Spanner's "external consistency" is equivalent to linearizability; `TrueTime` bounds make lease intervals disjoint; leader leases default to 10 s; Paxos is pipelined | V | Corbett et al., *Spanner: Google's Globally-Distributed Database* | Paper PDF; quoted | OSDI 2012 | ✅ |

### 14.3 Time, Causality and Consensus

| Claim | Class | Source | Quality | Date/venue | Verdict |
| --- | --- | --- | --- | --- | --- |
| Happens-before; logical clocks; physical clocks cannot order distributed events | R | Lamport, *Time, Clocks, and the Ordering of Events in a Distributed System* | Author's own publication record | CACM 21(7):558–565, 1978 | ✅ |
| Vector clocks | R | Fidge (1988) / Mattern (1988) | **Not verified** this pass — the guide states the mechanism, not a venue claim | — | ⚠ |
| Causal ordering (ISIS) | R | Birman & Joseph | **Not verified** this pass | 1987 | ⚠ |
| Paxos (Synod); the paper "was first submitted in 1990, setting a personal record for publication delay" | R | Lamport, *The Part-Time Parliament* | Author's publication record; quote verified | ACM TOCS 16(2):133–169, May 1998 | ✅ |
| Paxos Made Simple, written because Lamport "got tired of everyone saying how difficult it was to understand" | C | Lamport | Author's publication record; quote verified | ACM SIGACT News 32(4), Whole No. 121, pp. 51–58, Dec 2001 | ✅ |
| Fast Paxos: two message delays at the cost of more processes; 3 delays on conflict | R | Lamport | Author's publication record; quotes verified | Distributed Computing 19(2):79–103, Oct 2006 | ✅ |
| Cheap Paxos: `f+1` main + `f` auxiliary processors; needs a configuration master | R | Lamport & Massa | Author's record; quote verified | DSN 2004, Florence | ✅ |
| Vertical Paxos: reconfiguration inside the algorithm, with an external configuration master | R | Lamport, Malkhi & Zhou | Author's PDF (dated 9 Feb 2009; corrected 26 Aug 2009, 10 Mar 2019) | PODC 2009 | ✅ |
| Paxos Commit reframes commit as consensus instances; 2PC is "the trivial version of Paxos Commit that tolerates zero faults" | R | Gray & Lamport, *Consensus on Transaction Commit* | Quote verified via Lamport's own record | ACM TODS 31(1):133–160, 2006 | ✅ |
| Viewstamped Replication as an independent arrival at the same result | R | Oki & Liskov | **Not verified** this pass; treated as historically parallel | PODC 1988 | ⚠ |
| Raft is "equivalent to (multi-)Paxos" and "as efficient", but more understandable; overlapping majorities for membership change; user study as evidence | R | Ongaro & Ousterhout, *In Search of an Understandable Consensus Algorithm* | USENIX proceedings record — **Best Paper** verified | USENIX ATC 2014, pp. 305–319 | ✅ |
| PBFT: practical BFT in asynchronous environments, "only 3% slower than a standard unreplicated NFS"; names *software errors* alongside attacks | R | Castro & Liskov, *Practical Byzantine Fault Tolerance* | USENIX OSDI 99 record; quotes verified | OSDI 99, New Orleans, Feb 1999 | ✅ |
| PBFT extended TOCS version — volume and pages | R | Castro & Liskov | Not verified (recalled) | ACM TOCS 2002 | ⚠ |

### 14.4 Failure Detection, Membership, Leases and Fencing

| Claim | Class | Source | Quality | Date/venue | Verdict |
| --- | --- | --- | --- | --- | --- |
| Failure detectors as first-class abstractions with classes P / ◇P / S / ◇S; ◇S suffices for consensus | R | Chandra & Toueg, *Unreliable Failure Detectors for Reliable Distributed Systems* | Corroborated through Kleppmann's and SWIM's reference lists | JACM 43(2):225–267, Mar 1996 | ✅ |
| Accrual / heartbeat failure detector (output as a continuous suspicion level) | R | Aguilera & Toueg | **Not verified** this pass | 1997 | ⚠ |
| Epidemic algorithms for replication; origin of anti-entropy and rumour-mongering | R | Demers, Greene, Hauser, Irish & Larson | Both venues confirmed | PODC 1987, pp. 1–12; also OSR 22(1):8–32, 1988 | ✅ |
| SWIM: separates detection from dissemination; randomised probing; per-member cost independent of group size; suspect-before-declare; piggybacked dissemination; membership is *weakly* consistent | R | Das, Gupta & Motivala | Paper PDF extracted and quoted; **the DSN 2002 venue attribution is flagged** | DSN 2002 | ⚠ (PDF ✅ / venue ⚠) |
| Leases as a bounded lock for cache consistency | R | Gray & Cheriton, *Leases: An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency* | Verified via Kleppmann's citation of the paper for the lease concept | SOSP 1989 | ✅ |
| The two-generations-of-leader failure; fencing tokens; "you cannot fix this problem by inserting a check on the lock expiry just before writing back to storage"; a documented 90-second packet delay; HBase GC pauses of "several minutes" | C | Kleppmann, *How to do distributed locking* | Post fetched; quoted verbatim | 8 Feb 2016 | ✅ |

### 14.5 Delivery Semantics, Exactly-Once and Transactions — the second highest-risk class

| Claim | Class | Source | Quality | Date/venue | Verdict |
| --- | --- | --- | --- | --- | --- |
| The three modes, in the vendor's own words; and the warning that "many systems claim to provide 'exactly-once'... these claims are misleading" | V | Apache Kafka documentation, *Message Delivery Semantics* | Fetched and quoted verbatim | kafka.apache.org/documentation, extracted **2026-09-18** | ✅ |
| Kafka's exactly-once is scoped to Kafka topics; "exactly-once delivery for other destination systems generally requires cooperation with such systems" | V | Apache Kafka documentation | Same page, quoted verbatim | same | ✅ |
| "Note that many systems claim to provide exactly once delivery semantics, but this might not always what you think it is… cases where a consumer or producer **outside of the system** has failed" | V | Confluent documentation, *Message Delivery Guarantees for Apache Kafka* | Fetched and quoted verbatim (typo reproduced as published) | docs.confluent.io, extracted **2026-09-18** | ✅ |
| Idempotent producer deduplicates on producer ID + sequence number, preventing duplicate log entries; committed once all ISR replicas have applied it | V | Apache Kafka / Confluent documentation | Fetched | same | ✅ |
| Flink's exactly-once is checkpoint-and-replay: restore state and replay from the checkpoint; requires a **rewindable source**; skipping alignment produces duplicates on restore; mechanism derives from Chandy–Lamport | V | Apache Flink documentation, *Stateful Stream Processing* | Fetched and quoted verbatim | Flink 1.19 docs, extracted **2026-09-18** | ✅ |
| Kafka's transactional boundary is the producer, not the consumer; the offset write rides in the producer's transaction | V | Apache Kafka documentation, *Using Transactions* | Fetched and quoted verbatim | same | ✅ |
| Idempotency keys are retained ~24 h and pruned; "We generate a new request if a key is reused after the original is pruned"; parameters are compared and mismatches error | V | Stripe API reference, *Idempotent requests* | Fetched and quoted verbatim | docs.stripe.com, extracted **2026-09-18** | ✅ |
| 2PC "can easily block when nodes are unavailable"; global-serializability platforms likened to the Maginot Line; scaling implies an "entity" abstraction updated within but never across entities | C (**position paper**) | Helland, *Life beyond Distributed Transactions: an Apostate's Opinion* | Paper PDF fetched; quoted; the paper's own disclaimer noted ("the positions expressed in this paper are personal opinions") | CIDR 2007, 7–10 Jan, Asilomar, pp. 132–141 | ✅ |
| Sagas: all sub-transactions complete or compensating transactions run; compensation is semantic not physical; **no isolation — transactions may see partial saga results and are not notified when compensated** | R | Garcia-Molina & Salem, *Sagas* | Paper PDF fetched; all three limits quoted verbatim; ACM notice `0-89791-236-5/87/0005/0249` visible | Proc. ACM SIGMOD 1987, pp. 249–259 | ✅ |
| Three-phase commit as the non-blocking refinement | R | Skeen | **Not verified** this pass | 1981 | ⚠ |

### 14.6 Repository-boundary claims (verified by inspection of the working tree, 2026-09-18)

| Claim | Verdict |
| --- | --- |
| No distributed-systems discipline guide existed before this one; the five system-design meta-guides are interview- or book-framed | ✅ — `grep -rc -i "distributed system"` over all `.md`, plus filename scan; the five guides and their line counts (705/716/721/708/709) read directly from the working tree |
| `vector clock` had **zero** files' coverage | ✅ — `grep -rli` returned 0 files |
| `crdt` 2, `cap theorem` 3, `byzantine` 2, `consistent hashing` 7, `two-phase commit` 11, `linearizab*` 9, `raft` 22, `paxos` 10, `quorum` 36 | ✅ — measured on the working tree |
| `technology/temporal_guide.md` does not exist (the file is `technology/temporal_workflow_guide.md`) | ✅ — confirmed by `ls` before citing |
| `technology/apache_seata_guide.md` owns the saga / TCC / AT pattern space | ✅ — the file exists and was revised in a prior window against SIGMOD 1987 |

## 15. What Could Not Be Verified

Listed explicitly so that nothing above is read as more settled than it is. **A reader who needs any of these load-bearing should verify them, not cite this guide.**

1. **Brewer, *CAP Twelve Years Later* (IEEE Computer 45(2):23–29, 2012)** — the article itself could not be retrieved (InfoQ served a bot challenge; the IEEE page was not extracted). The *substance* of the self-correction is stated at §4.3 as ⚠ and **not quoted**. The phrase "2 of 3 is misleading" is therefore a paraphrase of a widely reported correction, not a verified quotation.
2. **Fox & Brewer, *Harvest, Yield, and Scalable Tolerant Systems* (HotOS 1999)** — not retrieved. The harvest/yield framing in §4.3 is anchored instead on Brewer's **verified** 2000 keynote slide ("Capacity × Completeness == Constant" ✅), which is a genuine primary artifact.
3. **The coordinated attack / two-generals attribution** — the Akkoyunlu–Ekanadham–Huber SOSP 1975 record and Gray's 1978 attribution are both **unverified**. §9.2's argument stands independently; only the attribution is flagged. **This is the single most important flag in this guide**, because the claim is used in vendor-facing arguments where the citation is routinely mis-stated.
4. **Aguilera & Toueg's heartbeat failure detector (1997)** — the accrual *mechanism* in §8.2 rests on the verified Chandra–Toueg abstraction; the 1997 paper's venue and title are from recollection.
5. **Skeen's three-phase commit (1981)** — §10.2's description of 3PC is the standard textbook account, but the primary source was not verified this pass, and the synchrony caveat is stated as a consequence of FLP rather than as a quotation from that paper.
6. **Viewstamped Replication (Oki & Liskov, PODC 1988)** — treated as a historically parallel result; no specific claim is attributed to it.
7. **Castro & Liskov, PBFT extended version (ACM TOCS, 2002)** — volume and page numbers recalled, not verified. The OSDI 99 record **is** verified.
8. **Papadimitriou (1979), Adya/Liskov/O'Neil (2000), Malkhi & Reiter (1998), Birman & Joseph (1987), Fidge and Mattern (1988), Bailis et al. coordination avoidance (2015)** — cited as the origin of a concept; venue details not verified this pass. Links to the *concepts* are sound; the bibliographic strings should be checked before use in formal work.
9. **SWIM's DSN 2002 venue** — the paper PDF was retrieved and quoted, but the conference attribution is flagged.
10. **ACM Digital Library** — returned HTTP 500 to the extractor on all five attempts, so no ACM-hosted page was read directly. Where ACM venue details are marked ✅ they were confirmed via the author's own record or the paper's own PDF.
11. **`web_search` returned empty for every query attempted** this pass. This is recorded as a **tool limitation, not as evidence of absence** — no claim in this guide rests on the absence of a search result.

## 16. Glossary

| Term | Meaning as used in this guide |
| --- | --- |
| **accrual failure detector** | A detector whose output is a continuous suspicion level rather than a boolean, so the caller picks the threshold and therefore picks which error it tolerates |
| **assumption ledger** | §2.5: the table of what a design silently assumes about network, clocks and machines, and what breaks when each assumption fails |
| **atomic commit** | A protocol making a multi-participant update all-or-nothing; 2PC is the classical form, with a blocking failure mode |
| **blocking (2PC)** | The state a participant enters after voting yes and before learning the outcome; it cannot decide alone, so it waits — indefinitely |
| **Byzantine fault** | A faulty component that continues executing while behaving arbitrarily (attacks *or* software errors), not merely stopping |
| **CAP** | The Gilbert–Lynch theorem: atomic (linearizable) consistency and availability cannot both be guaranteed during a partition |
| **causal consistency** | Causally related operations are seen in the same order everywhere; concurrent ones need not be |
| **compensation** | A new transaction with the opposite intent; it undoes a *semantic* effect and cannot restore the prior state, fees, rates or external effects |
| **consistent hashing** | Placing keys and nodes on a ring so that one node's departure moves only its neighbour's keys; virtual nodes smooth the distribution |
| **consensus** | Agreement by non-faulty processes on a single proposed value, with integrity and agreement |
| **coordination** | Any operation requiring nodes to communicate before deciding — the source of the latency, availability and complexity costs in §11.3 |
| **coordination-free / I-confluent** | An invariant that can be preserved by merging divergent states without communication |
| **CRDT** | A replicated data type whose merge is commutative/associative/idempotent, so replicas converge without coordination |
| **crash-recovery** | A failure model where a process may halt and later restart with persisted state |
| **deduplication window** | The period over which a consumer remembers processed message IDs; **a correctness parameter**, not a tuning knob |
| **epoch / term** | A monotonically increasing leadership incarnation number; higher wins |
| **eventual consistency** | If updates stop, replicas eventually converge — with **no bound** on when |
| **exactly-once (delivery)** | Not achievable end to end (§9.2); what is achievable is exactly-once *effects* |
| **fencing token** | A monotonically increasing number attached to writes that **the resource** validates and rejects if stale; a client-side check is not a fence |
| **FLP** | The Fischer–Lynch–Paterson result: no deterministic consensus protocol guarantees both agreement and termination in a fully asynchronous system with one crash fault |
| **failure detector** | An abstraction reporting which processes have failed; it may be wrong, and its error modes are the design question |
| **gossip / epidemic protocol** | Probabilistic dissemination of membership and state, with per-node cost independent of group size |
| **grow-only / semi-lattice** | A structure whose merge is a join, giving convergence without ordering |
| **happens-before** | Lamport's partial order: causally related events are ordered; concurrent ones are not |
| **idempotency key** | A client-generated unique key letting a server return the stored result of a repeated request — subject to a retention window |
| **in-doubt transaction** | A participant that has voted but cannot learn the outcome; the hallmark of 2PC's blocking window |
| **ISR** | Kafka's in-sync replicas: the set whose members must have applied a message for it to be *committed* |
| **invariant** | A property the system must always preserve; the input to every decision in this guide |
| **leader lease** | A time-bounded grant of leadership; it defines who *should* act, not who *can* |
| **linearizability** | A total order over operations respecting real time, each appearing atomic between invocation and response |
| **logical clock** | A counter advanced by communication rather than by time, giving an order consistent with causality |
| **majority quorum** | ⌊n/2⌋+1 of the current voter list; different in kind from leaderless `R + W > N` |
| **partial failure** | The defining condition: some component is broken and you usually cannot tell which |
| **partial synchrony** | The assumption that the system is synchronous eventually, or synchronous outside unknown bounds — FLP's escape hatch |
| **PBFT** | Practical Byzantine Fault Tolerance: consensus tolerating arbitrary faults with `n ≥ 3f+1`, viable in asynchronous environments |
| **quorum intersection** | The property that any two quorums overlap, which is what makes at-most-one-leader-per-term true |
| **read-your-writes** | A per-session guarantee; achievable without coordination, unlike global linearizability |
| **replica** | A participant holding a copy of the data, and usually a vote in a quorum |
| **Raft** | A consensus algorithm for replicated logs; equivalent to multi-Paxos, restructured for implementability |
| **saga** | A long-lived transaction decomposed into sub-transactions with compensators; provides atomicity of outcome but **no isolation** |
| **serializability** | Equivalence to *some* serial execution, with no real-time constraint |
| **session guarantee** | A per-client guarantee (read-your-writes, monotonic reads/writes, writes-follow-reads) |
| **shard / partition** | The unit of independent consensus and placement; cross-shard means no shared order |
| **snapshot isolation** | Each transaction reads a consistent snapshot; commits iff no concurrent committed transaction wrote its write set — **not serializable** (write skew) |
| **split brain** | Two nodes each believing they lead the same term; usually prevented for the log and **not** for external side effects |
| **state machine** | A deterministic function of an ordered command sequence; replication replicates the *order*, not the state |
| **strict serializability** | Serializability plus linearizability's real-time ordering |
| **SWIM** | A weakly-consistent membership protocol using randomized probing, suspect-before-declare, and piggybacked dissemination |
| **two-generations-of-leader** | The failure where a paused old leader resumes and writes after a new leader has been elected; the reason fencing exists |
| **write skew** | Two transactions read overlapping data, write disjoint data, and both commit, violating a cross-item invariant |
| **witness** | A voting member that holds votes but not data |
| **vector clock** | A per-process counter vector giving causality and concurrency detection — including the ability to say *two events are concurrent* |

## 17. Cross-References and Further Reading

**Within this repository — the guides that own their own subjects.** This guide cross-references rather than re-derives, and the boundary is declared in §1.4.

| Guide | What this guide defers to it |
| --- | --- |
| `technology/ddia_study_companion_guide.md` | The closest existing treatment of the theory, in book-companion form. This guide cites the primary papers directly and does **not** duplicate its chapter map |
| `technology/nalsd_system_design_guide.md`, `technology/grokking_system_design_companion_guide.md`, `technology/system_design_interview_insiders_guide.md`, `technology/google_system_design_interview_guide.md` | The interview and book-companion frame — structure, estimation, rubrics, case study catalogues |
| `technology/cockroachdb_guide.md` | CockroachDB as an answer set: Raft per range, serializable default, HLC timestamps |
| `technology/polardb_vs_oceanbase_guide.md` | Shared-storage vs shared-nothing, Paxos-based replication as a concrete contrast |
| `technology/oracle_sharding_guide.md` | Sharding mechanics, shard directors, cross-shard queries — the operational cost of partitioning |
| `technology/kafka_virtualization_guide.md`, `technology/kafka_alternatives_guide.md`, `technology/event_stream_processing_guide.md` | Kafka's ISR/`acks`/idempotent-producer/transaction machinery, and the alternatives |
| `technology/apache_seata_guide.md` | **The saga / TCC / AT pattern space in depth**, including isolation countermeasures — §10.3 defers to it by name |
| `technology/temporal_workflow_guide.md` | Durable execution; compensation expressed as code with deterministic replay |
| `technology/dds_guide.md` | DDS QoS durability and reliability profiles as an answer set to §9's delivery question |
| `technology/cephfs_alternatives_guide.md` | Placement and rebalancing at the storage layer |
| `technology/monolith_to_microservices_guide.md` | Service decomposition and boundaries; the transactional-outbox pattern |
| `technology/chaos_engineering_guide.md` | **The testing techniques** — fault injection, experiment design, blast radius. §11.5 and §13 cross-reference it for *how you would find out* |
| `technology/deterministic_engineering_guide.md` | Determinism, replay and deterministic simulation testing — the mechanism behind §9.4's last row |
| `technology/distributed_rate_limiter_guide.md`, `technology/distributed_auth_guide.md`, `technology/zero_downtime_system_design_guide.md` | Adjacent applied distributed-systems designs; each is an instance of the questions in §1.1 |

**Primary sources — the canon, as verified in §14.** Fischer–Lynch–Paterson (JACM 1985) · Dwork–Lynch–Stockmeyer (JACM 1988) · Gilbert & Lynch (SIGACT News 2002) · Brewer (PODC 2000 keynote; IEEE Computer 2012) · Abadi (IEEE Computer 2012) · Herlihy & Wing (TOPLAS 1990) · Berenson et al. (SIGMOD 1995) · Lamport (CACM 1978; TOCS 1998; SIGACT News 2001; DC 2006) · Lamport & Massa (DSN 2004) · Lamport, Malkhi & Zhou (PODC 2009) · Gray & Lamport (TODS 2006) · Ongaro & Ousterhout (USENIX ATC 2014) · Castro & Liskov (OSDI 1999) · Lamport, Shostak & Pease (TOPLAS 1982) · Chandra & Toueg (JACM 1996) · Demers et al. (PODC 1987) · Das, Gupta & Motivala (DSN 2002) · Gray & Cheriton (SOSP 1989) — and, for the applied side, Helland (CIDR 2007), Garcia-Molina & Salem (SIGMOD 1987), Shapiro et al. (SSS 2011), Corbett et al. (OSDI 2012), Kleppmann (2016), Bailis et al. (VLDB 2014). **Sources marked ⚠ in §14 are cited for the concept only and their bibliographic strings should be checked before formal use — do not treat this list as a bibliography.**

**Product documentation consulted and dated in §14:** Apache Kafka documentation (extracted 2026-09-18) · Confluent documentation (2026-09-18) · Apache Flink 1.19 documentation (2026-09-18) · Stripe API reference (2026-09-18).

**The one book worth cross-reading rather than a paper:** Kleppmann's *Designing Data-Intensive Applications* remains the best single narrative through §§3–7 of this guide; read it alongside `technology/ddia_study_companion_guide.md`, and read the papers above when a claim has to be defended.

## 18. Closing Summary

The discipline is smaller than its surface area suggests. It is a set of hard questions — what am I assuming about failure, how do I know the order of events, what is provably impossible, which guarantee does this process actually need, am I willing to pay coordination for it, how is state placed, how do I know a node is dead, and what happens if this is processed twice — and a well-settled body of theory about which answers are compatible. Every product in this repository's distributed cluster, and every design a reader will review next quarter, is a particular set of answers to those questions. The failures that reach production are, overwhelmingly, questions that were answered by nobody.

Three findings are worth carrying out of the guide. **First, the impossibility results are narrower and more useful than their folklore**: FLP forbids guaranteed termination, not consensus; CAP binds only during a partition and does not offer "CA"; and the escape hatches — partial synchrony, choosing CP or AP, weaker-than-serializable isolation — are all priced, which means they are all decidable. **Second, the consistency vocabulary is mostly a claim about scope**: "strong consistency" resolves to one of eight specific guarantees, at one of several scopes, and the phrase hides which; the conflated pairs in §5.2 are the short list of confusions that produce incidents. **Third, exactly-once is a boundary, not a guarantee**: delivery is not achievable end to end and the vendors' own documentation says so; what is achievable is an idempotent effect, and it comes from idempotency keys, transaction design and a deduplication window you actually measured — not from a transport setting.

Everything else follows from writing things down. An invariant list with a failure column, a declared failure model, a declared consistency level per operation, an explicit conflict policy, a lease with a fence the resource validates, a deduplication window wider than the retry budget, and a partition that was actually injected make a system reviewable. None of them is an algorithm.

Nothing in a distributed system is ever fully knowable from inside it. Every design is a set of decisions about what to do when you cannot tell — and the discipline is the accumulated answer to how to choose well when the only thing you can be certain of is the partial failure.
