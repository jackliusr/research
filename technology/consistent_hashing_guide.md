# Consistent Hashing — the Ring, the Virtual Nodes, and Why the Technique Is a Misnomer

**The Technique Layer — Why Modulo Hashing Fails and Why the Failure Is Catastrophic Rather Than Gradual, the Ring Stated Precisely, the Arc Arithmetic Derived Rather Than Asserted, Virtual Nodes and the 1/√R Mechanism, Replica Placement Across Failure Domains, Weighting and Heterogeneous Fleets, the Four Places It Genuinely Cannot Help, Rendezvous and Jump and Maglev as Alternatives, Nine Implementations Traced to Their Own Documentation, Client-Side Versus Server-Side Rings, Operations and the Monitoring That Reveals an Unbalanced Ring, the Regulated-Institution Angle, a Cymbal Bank Worked Example, the Anti-Patterns, the Claims Audit, and What Could Not Be Verified**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Byline:** Jack Liu Shurui, Solution Architect
> **Series:** Technology / Distribution — the one algorithm almost every distributed system uses and almost nobody states precisely.
> **Audience:** architects, platform and data engineers who are choosing a partitioning scheme, reviewers who have to decide whether a proposed ring is balanced enough to survive a node failure, and anyone who has watched a cache tier evaporate when a node count changed.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026
> **Integrity convention:** ✅ = verified this pass against the cited primary source (paper PDF, publisher/conference record, or the product's own documentation); ⚠ = flagged — asserted, plausible, or dependent on a secondary source; ❌ = rejected (asserted somewhere, found false or unsupported). Used in every table that makes a factual claim.
> **Claim classes:** **PROVEN RESULT** (a theorem with a published proof, or arithmetic derived in this guide line by line), **ENGINEERING CONVENTION** (a widely used practice with no theorem behind it — true because everyone agreed), **VENDOR DOCUMENTATION** (a product's claim about itself, or its own specification; treated as authoritative only for what that product does).
> **Illustrative figures:** Every number attributed to **Cymbal Bank** (a **fictional** institution, and the only bank persona used in this repository) is **explicitly illustrative and fictional** — a worked shape for a decision, not a benchmark, not a survey, and not a claim about any real institution. No real bank is asserted to use any scheme discussed here.
> **Verification method, stated up front:** Every ✅ below was verified this pass with live web access, using **`web_extract` against a primary URL** — the Dynamo SOSP'07 PDF at `allthingsdistributed.com`, the arXiv abstract pages for jump consistent hash and bounded loads, the USENIX NSDI '16 record for Maglev, `redis.io` cluster specification, `cassandra.apache.org` architecture documentation, `envoyproxy.io` load-balancer documentation, `docs.couchbase.com` and the `libketama` README. `web_search` was **not** used as a source of verification — its two queries this pass were used only to locate two documents, and that behaviour is recorded in §16.1 item 11. Where a source could not be reached (the ACM Digital Library, the Microsoft Research tech-report PDF for HRW, the JavaScript-driven Riak documentation site), it is recorded in §16 as a tool limitation, **not** as evidence of absence. Where claims could not be confirmed, they are in §16.

**How this guide is organised.** §1 fixes the boundary and the thesis. §2 derives, from scratch, why modulo hashing is catastrophic rather than merely inefficient. §3 builds the ring and derives what moves. §4 is the mechanism behind virtual nodes. §5 is replication on the ring and why the naive walk is a trap. §6 is weighting. §7 is the section that matters most: the four things the technique cannot do, stated as mechanisms. §8 is the alternatives, with a properties table. §9 traces nine implementations to their own documentation. §10 is the stale-ring problem. §11 is operations. §12 is the regulated-institution angle. §13 is a fictional worked example. §14 is the anti-patterns. §15 is the claims audit. §16 is what could not be verified, the glossary, cross-references, and the closing.

**Table of Contents**

1. [Overview, the Decoder, the Thesis and the Boundary](#1-overview-the-decoder-the-thesis-and-the-boundary) — 1.1 the problem in one sentence · 1.2 the decoder, extended · 1.3 the thesis · 1.4 what this guide owns and what it does not
2. [The Problem It Solves: Modulo Hashing and Its Failure](#2-the-problem-it-solves-modulo-hashing-and-its-failure) — 2.1 the scheme · 2.2 the derivation · 2.3 worked values · 2.4 the removal case and the general fact · 2.5 why a cache tier turns a 91% remap into an outage
3. [The Ring, Constructed Precisely](#3-the-ring-constructed-precisely) — 3.1 the construction · 3.2 the lookup · 3.3 what moves on add and on remove, derived · 3.4 the comparison against modulo · 3.5 lookup and memory cost
4. [Balance and Virtual Nodes](#4-balance-and-virtual-nodes) — 4.1 why few tokens give an uneven ring · 4.2 the 1/√R mechanism · 4.3 what Dynamo measured · 4.4 why many small arcs beat one big arc · 4.5 what vnodes cost
5. [Replication on the Ring](#5-replication-on-the-ring) — 5.1 the naive clockwise walk · 5.2 what it gets wrong · 5.3 failure-domain-aware placement · 5.4 the preference list
6. [Weighting and Heterogeneous Nodes](#6-weighting-and-heterogeneous-nodes) — 6.1 the lever · 6.2 drift · 6.3 weight changes are data movement
7. [Where It Fails](#7-where-it-fails) — 7.1 hot keys · 7.2 cold cache and stampede · 7.3 churn and flapping · 7.4 multi-key and cross-partition operations
8. [The Alternatives](#8-the-alternatives) — 8.1 rendezvous/HRW · 8.2 jump consistent hash · 8.3 Maglev · 8.4 bounded loads · 8.5 the properties table
9. [The Implementations, Verified One by One](#9-the-implementations-verified-one-by-one) — 9.1 memcached/ketama · 9.2 Dynamo and Cassandra · 9.3 Redis Cluster, which is not a ring · 9.4 Kafka · 9.5 Envoy · 9.6 Riak · 9.7 Couchbase
10. [Client-Side Versus Server-Side Rings](#10-client-side-versus-server-side-rings) — 10.1 where the ring lives · 10.2 stale rings · 10.3 membership propagation and versioning
11. [Operations](#11-operations) — 11.1 the procedure · 11.2 what actually moves · 11.3 the monitoring · 11.4 the honest note
12. [The Regulated-Institution Angle](#12-the-regulated-institution-angle) — 12.1 where it appears in a bank estate · 12.2 the partition map as a versioned asset
13. [A Cymbal Bank Worked Example](#13-a-cymbal-bank-worked-example) — 13.1 the starting state · 13.2 the migration arithmetic · 13.3 the vnode choice · 13.4 the hotspot the ring does not solve · 13.5 the rollback position
14. [Anti-Patterns](#14-anti-patterns) — symptom / cause / guardrail
15. [Claims Audit](#15-claims-audit)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing](#16-what-could-not-be-verified-glossary-cross-references-and-closing)

## 1. Overview, the Decoder, the Thesis and the Boundary

### 1.1 The problem in one sentence

A client has a set of keys and a set of servers. It needs a rule that answers "which server owns key *k*?" that (a) spreads keys evenly, (b) is computable independently by every client without asking anyone, and (c) does not reshuffle the whole keyspace when the server set changes. Only (c) is hard, and only (c) is what the technique described in this guide actually buys.

### 1.2 The decoder, extended

The repository's house decoder (§1 of `distributed_systems_engineering_guide.md`) names the distributed-systems terms. This guide extends it with the vocabulary of the partitioning layer. Every one of these terms is used informally in production documentation and precisely here.

| Term | The precise meaning | The confusion it is meant to prevent |
| --- | --- | --- |
| **Hash ring** | A circular, ordered identifier space — in practice the integers `[0, 2^n)` — onto which both *nodes* and *keys* are mapped by a hash function. "Ring" is a statement about the ordering, not a physical topology. | That the ring is a data structure you can be given a handle to. It is a convention plus a sorted list of tokens. |
| **Token** | A single position on the ring, i.e. one hash value, that a physical node claims. A node with R tokens claims R positions. Cassandra's own documentation defines a token exactly this way — "a single position on the Dynamo-style hash ring" ✅. | "Token" in the security sense. The two senses coexist in the same documents. |
| **Virtual node (vnode)** | One token of a physical node when a node holds more than one. It is not a process, not a container, and not a replica: it is a *claim on an arc*. ✅ (Cassandra: "a token on the hash ring owned by the same physical node".) | That a vnode is a node. It has no memory, no CPU and no failure mode of its own — but it does multiply the node's ring neighbours, and that has availability consequences (§4.5). |
| **Replica placement** | The rule that turns "the owner of *k*" into "the set of nodes that hold *k*", normally by continuing clockwise to the next distinct *physical* nodes. ✅ (Cassandra: replicas "are always chosen such that they are distinct physical nodes which is achieved by skipping virtual nodes if needed".) | That replication is a separate subsystem. On a ring it is part of the same walk. |
| **Rebalancing** | The physical movement of data or traffic after the ownership map changes. It is *not* the recomputation of the map; the map recomputation has to be cheap by design, and it is. | That "consistent hashing is consistent". The technology makes the *map* cheap to change and the *data* just as expensive to move as it ever was (§11.4). |
| **Lookup** | Locating the owning node for a key: hash the key, then find the first token clockwise. `O(log N)` over a sorted token array, or `O(1)` over a precomputed table. | That lookup on the ring is free at high node counts if you reimplement it naively per request. |
| **Origin** | In a cache tier, the system of record behind the cache. The ring's job is to make cache-miss behaviour survivable; the origin is what absorbs the misses when it fails. | That cache-tier partitioning is a cache problem. It is an origin-capacity problem (§2.5). |
| **Client-side partitioning** | The placement decision is made by the *caller*, holding its own copy of the ring. | That the server enforces consistency. It usually does not, and that is the single most common incident cause in this area (§10). |
| **Server-side partitioning** | The placement decision is made centrally (a coordinator, a config service, a metadata service) and pushed to clients; clients may still cache it. | That server-side means there is no ring. Redis Cluster is server-side and slot-based (§9.3). |

### 1.3 The thesis

**Consistent hashing is a misnomer.** It does not make hashing consistent. The hash function is not made more consistent by the ring; the same key hashed by the same function gives the same value before and after. What the technique makes cheap is **rebalancing**: the *fraction of keys whose owner changes* when the node set changes, nothing else.

That distinction is not pedantry. It is the whole guide. Almost every real misunderstanding in this area — a team expecting a balanced ring to fix a hot key (§7.1), a team expecting a ring to avoid data movement (§11.4), a team benchmarking a ring without measuring imbalance (§14) — comes from reading the name as a promise about hashing rather than a claim about remapping volume. The closing sentence of this guide repeats the correction in one line.

### 1.4 What this guide owns and what it does not

**This guide owns:** the consistent-hashing technique itself — the ring and its construction, virtual nodes and the balance mechanism, replica placement on the ring (including failure-domain awareness), weighting, the failure modes of the technique, the alternative placement schemes (rendezvous/HRW, jump, Maglev, bounded loads), the production implementations, client-side versus server-side rings, and the operational consequences.

**This guide does not cover, and will not re-derive:**

| Owned by | Filename | What it owns |
| --- | --- | --- |
| The study route for interviews | [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md), [ddia_study_companion_guide.md](ddia_study_companion_guide.md), [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) | How the technique is likely to be asked about, how it is framed for interview preparation, and the textbook framing around it (DDIA's chapter on partitioning, the Grokking design chapters). Read those for the exam-shaped version of this material; read this for the mechanism. |
| The distributed-systems discipline | [distributed_systems_engineering_guide.md](distributed_systems_engineering_guide.md) | Replication and partitioning *as a topic area* — the three topologies, synchronous versus asynchronous replication, the partition/rebalance framework, secondary indexes, and the cross-partition operation as a design problem. Its §7.4 already states the key distinction — *virtual nodes fix hot nodes, not hot keys* — and shows a three-row comparison of modulo versus ring versus virtual nodes. **This guide does not contradict that; it is the deeper treatment behind those three rows.** |
| Consistency models | [tunable_consistency_databases_guide.md](tunable_consistency_databases_guide.md) | What a quorum means, the consistency spectrum, and shard-per-core architecture. Where this guide says "the replicas are the preference list", that file says what the preference list can then promise. |
| Oracle's product | [oracle_sharding_guide.md](oracle_sharding_guide.md) | Oracle's sharding product: its own partitioning mechanisms and operational model. Where this guide needs an RDBMS sharding contrast, it points there rather than re-deriving it. |
| Analytical engines | [clickhouse_guide.md](clickhouse_guide.md) §4.2, [spark_tuning_guide.md](spark_tuning_guide.md) §11 | Sharding in ClickHouse and partitioning in Spark — both are partition-assignment problems, both solved with hash mod or explicit keys, neither is a ring. Read those for the batch/analytical side. |
| Large-scale architecture framing | [architecture/billion_user_system_arch.md](architecture/billion_user_system_arch.md) | Where partitioning sits in a billion-user design; the technique appears there as one component of a larger answer. |

Everything in the table above is cross-referenced by filename; nothing in it is repeated here.

## 2. The Problem It Solves: Modulo Hashing and Its Failure

### 2.1 The scheme

The naive placement rule is one line: `owner(k) = h(k) mod N`, where `N` is the number of buckets and `h` is a uniform hash function. It has no state, `O(1)` time per lookup and `O(1)` memory. It satisfies two of the three requirements in §1.1 — it spreads keys evenly, and every client can compute it independently. It fails the third one completely, and the size of the failure is the whole reason this technique exists.

### 2.2 The derivation: what changes when N becomes N+1

**Setup.** `N` buckets; change the bucket count to `N+1`. A key keeps its bucket iff its hash satisfies both moduli the same way:

```
key k is unchanged  ⟺  h(k) mod N  =  h(k) mod (N+1)
```

Write `x = h(k)` and consider `x` over one full period `P = N(N+1)`.

**The counting argument.** Because `gcd(N, N+1) = 1`, the Chinese Remainder Theorem gives a bijection between `x ∈ {0, 1, …, N(N+1) − 1}` and the pairs `(r, s) ∈ ℤ_N × ℤ_{N+1}`: each residue pair occurs **exactly once** per period. The condition `r = s` holds only for the pairs `(0,0), (1,1), …, (N−1,N−1)`. It stops at `N−1` and not at `N`, because a residue equal to `N` is not a legal value in `ℤ_N` (it is `0` there, and then `s = N ≠ r = 0`). So exactly **N** pairs out of the `N(N+1)` pairs satisfy the condition.

**The result.**

```
keys that keep their bucket     = N / (N(N+1)) = 1/(N+1)
keys that are remapped          = 1 − 1/(N+1)  = N/(N+1)
```

This is not an approximation and not a rule of thumb. Under the model that `h` behaves as a uniform random function, and with a key population large relative to `N(N+1)`, it is exact.

### 2.3 Worked values, add and remove

Adding one bucket, `N → N+1`, derived per the line above:

| Change | Fraction that keeps its owner | Fraction remapped (formula) | Fraction remapped (decimal) |
| --- | --- | --- | --- |
| 2 → 3 | 1/3 = 33.33% | 2/3 | 0.666667 = 66.67% |
| 10 → 11 | 1/11 = 9.09% | 10/11 | 0.909091 = 90.91% |
| 100 → 101 | 1/101 = 0.990% | 100/101 | 0.990099 = 99.01% |
| 1000 → 1001 | 1/1001 ≈ 0.0999% | 1000/1001 | 0.999001 = 99.90% |

Removing one bucket, `N → N−1`: the same argument with moduli `N` and `N−1` (also coprime). The condition `r = s` holds for `v ∈ {0, …, N−2}`, i.e. `N−1` pairs out of a period of `N(N−1)`:

```
keys that keep their bucket = (N−1) / (N(N−1)) = 1/N
keys remapped               = 1 − 1/N          = (N−1)/N
```

Verified numerically: `N=2 → 1/2 = 50%`; `N=3 → 2/3 ≈ 66.67%`; `N=10 → 9/10 = 90%`; `N=100 → 99/100 = 99.00%`.

**The general fact.** For coprime bucket counts `a → b`, the fraction that stays is `min(a,b)/(a·b) = 1/max(a,b)`, because there are exactly `min(a,b)` residues common to both moduli and one residue-pair period has length `lcm(a,b) = a·b` when `a` and `b` are coprime. So:

```
fraction remapped = 1 − 1/max(a,b)
```

Check it against both cases above: `a=N, b=N+1` gives `1 − 1/(N+1) = N/(N+1)` ✅; `a=N, b=N−1` gives `1 − 1/N = (N−1)/N` ✅. Consecutive integers are always coprime, so the two cases that actually occur in production — add one node, lose one node — are both in the exact regime.

**Two honest caveats.** (i) The result assumes a uniform random hash function; a hash with poor avalanche changes the constants and not the conclusion, but it can make them worse than the formula predicts. (ii) The formula describes the *expected* fraction; with a small key population the realised fraction is noisy around it. Neither caveat is the interesting one. The interesting one is what `N/(N+1)` does to a cache tier.

### 2.4 Why a cache tier experiences this as an outage, not a slowdown

Take a cache tier of `N = 10` nodes in front of an origin, warm, with a 99% hit ratio. Now change the node count — either deliberately (capacity growth) or involuntarily (a node dies).

- **Deliberate add, `10 → 11`.** By §2.3, 90.91% of keys are remapped. Those keys now hash to a node that does not hold them, so they miss. The 9.09% that stayed still hit. Absent any migration, the hit ratio is not "slightly degraded" — it is `1/(N+1) = 1/11 ≈ 9.09%`.
- **Involuntary loss, `10 → 9`.** `(N−1)/N = 9/10 = 90%` of keys are remapped. The failure of **one** cache node silently invalidates 90% of the cache. This is the more common production trigger: not the growth event that was planned and rehearsed, but the node death that was not.
- **Origin load.** Miss traffic rises from `1 − h ≈ 1%` to `N/(N+1) ≈ 90.9%` of the cache's request rate. The ratio of miss traffic before to after is `0.909091 / 0.01 ≈ 90.9` — a **90.9× increase in origin load in one step**, with no ramp, on the deploy or failover event itself.
- **The new node is cold too.** Its `1/(N+1)` arc of keys was previously owned by someone else and is not resident on it. So even the fraction of the keyspace that "should" now be served by the new node misses on first touch.
- **The feedback loop.** The origin saturates, latency rises, client requests time out, clients retry, retries add load to an already-saturated origin, and the cache cannot re-warm because re-warming requires successful origin reads. A saturated origin never warms the cache back; a cold cache keeps the origin saturated. The system sits in that state until load drops or a human intervenes.

The correct framing is the one this guide keeps returning to: **this is a capacity event, not an efficiency event.** Removing 90.9% of a caching layer is not "10% slower". It is the near-total removal of a tier, executed instantaneously and automatically by an arithmetic rule that no one had to deploy, triggered by a node count that a client library silently recomputed. And it is symmetric: modulo hashing punishes *both* directions, so the technology that is supposed to absorb failure amplifies it.

## 3. The Ring, Constructed Precisely

### 3.1 The construction, step by step

1. **Pick a hash function `H` with a large range** — in practice `2^32`, `2^64`, or a 160-bit digest. `H` must be uniform and must avalanche (a one-bit input change should randomise the output). Nodes and keys are hashed by the **same** `H` into the **same** space.
2. **Give every node one or more tokens.** For token `j` of node `i`, compute `H(identity(i,j))`. The identity string must be stable and unique per token — the canonical pattern is `host:port#replica_index`. Ketama's own README describes hashing each server string (`1.2.3.4:11211`) to several unsigned ints and placing those on a circle from `0` to `2^32` ✅.
3. **Sort the tokens ascending.** That sorted list *is* the ring. Each token owns the arc from the previous token (exclusive) to itself (inclusive) — Cassandra's documentation states the ownership range in exactly this half-open form, `(t1, t2]` ✅.
4. **Look up a key:** compute `t = H(key)`, find the first token `≥ t`, and walk to that token's node. If no token is `≥ t`, wrap to the smallest token. Ketama documents the wrap rule explicitly: a key hashing near `2^32` with no greater point on the continuum "return[s] the first server in the continuum" ✅.
5. **Replication** starts from that token and continues clockwise (§5).

Three non-obvious consequences:

- **The ring is a derived artifact.** There is no ring process, no ring state, and nothing to install. The "ring" is a sorted array of `(token, node)` pairs that any client can rebuild from the membership list and the same hash function.
- **Tokens must be unique, and collisions need a documented tie-break.** Two nodes can hash to the same token — the probability is small per pair but not zero at millions of tokens in a `2^32` space. Whatever the tie-break is, it must be in the specification, because every client must agree.
- **A different hash function is a different ring.** Two clients using different hash functions, or the same function with a different digest length or a different token identity string, produce different owners for the same keys and will not detect it. This is the mechanism behind §10.

### 3.2 Lookup cost, build cost and memory

| Structure | Lookup | Build | Memory | Notes |
| --- | --- | --- | --- | --- |
| Sorted token array + binary search | `O(log T)` | `O(T log T)` per membership change | `O(T)` | `T = N·R` tokens. At `N=100, R=256`, `T = 25,600`: 7–15 comparisons per lookup, ≈400 KB at 16 bytes per entry (`2^64` token + node index + overhead). |
| Precomputed lookup table | `O(1)` | `O(table size)` per membership change | `O(table size)`, independent of `N` | Maglev's table is filled by placing each host in proportion to its weight; Envoy's documentation of the same algorithm gives a concrete fixed total of **65,537 entries** ✅. |
| Per-client memoisation of key → node | `O(1)` amortised | — | `O(distinct keys)` | What a client-side "ring cache" degrades into; see §10 for why it is dangerous. |

The honest observation about `O(1)` versus `O(log N)`: at the node counts most estates actually run, this is not where the performance is. At `N=100` and `R=256`, a binary search is about fifteen comparisons over 400 KB of resident read-only memory. Teams that replace a sorted array with a custom table to save those comparisons while shipping a ring whose hot spot is 5× the average load (§4.1) have optimised the wrong term.

The token count is, however, a genuine operational cost, and Amazon's own production note says so: moving from "`T` random tokens per node" to "`Q/S` tokens per node" **"reduces the size of membership information maintained at each node by three orders of magnitude"**, and Dynamo gossips that membership information periodically, "and as such it is desirable to keep this information as compact as possible" ✅ (Dynamo SOSP'07, §6.2, read from the paper's own PDF at `allthingsdistributed.com`). Token count is not merely a balance knob; it is bytes on the wire, in every gossip round, forever.

### 3.3 What actually moves, derived

Work the unit chain. Total keyspace = `1.0`; `R` tokens per node; `T = N·R` tokens total.

- **Add one node to a ring of `N` (now `N+1`).** Idealised equal arcs: each node owns `1/(N+1)` of the circle after the change. Only the arcs of the new node change ownership: the keys falling in the new node's `R` arcs were, before the change, owned by whichever node previously owned those arcs. Every other token's predecessor relationship is untouched, so nothing else moves.

  ```
  keys moved per add = R × 1/((N+1)·R) = 1/(N+1)
  ```
- **Remove one node from a ring of `N`.** The departing node owns `R` arcs totalling (in expectation) `R/(N·R) = 1/N`. Its arcs are absorbed by the successors of its tokens.

  ```
  keys moved per remove = 1/N
  ```
- **The ratio against modulo hashing.**
  - Add: `modulo = N/(N+1)`, `ring = 1/(N+1)` → ratio `(N/(N+1)) ÷ (1/(N+1)) = N`.
  - Remove: `modulo = (N−1)/N`, `ring = 1/N` → ratio `((N−1)/N) ÷ (1/N) = N−1`.

  So the technique is a **factor-of-~N reduction in keys that move**, not the elimination of movement. Both halves of that sentence matter (§11.4).

| Change | Modulo: fraction remapped | Ring: fraction remapped | Ratio |
| --- | --- | --- | --- |
| `N=10 → 11` (add one node) | 10/11 = 90.91% | 1/11 = 9.09% | 10× |
| `N=100 → 101` (add one node) | 100/101 = 99.01% | 1/101 = 0.990% | 100× |
| `N=100 → 99` (one node dies) | 99/100 = 99.00% | 1/100 = 1.00% | 99× |

The `N=100 → 101` row is the number to remember: **99.01% versus 0.99%**, exactly a factor of 100, both derived in §2.3 and §3.3 and neither asserted. Note also what the table does *not* say: moving 0.99% of a 100 TB keyspace is still 0.99 TB to physically relocate, and for a *cache* the operationally relevant quantity is not "how many keys change owner" but "what fraction of the live request stream now misses" (§7.2).

### 3.4 The naive one-token-per-node ring is correct and useless

With `R = 1` — one token per node, the form in the original description of the technique — the arcs are the spacings between `N` uniform random points on a circle. Those spacings have mean `1/N` and are distributed as `Beta(1, N−1)`, which for large `N` is close to exponential. The **largest** of `N` such spacings has expected value `H_N / N`, where `H_N` is the `N`-th harmonic number `≈ ln N + 0.5772`. The unluckiest node therefore carries `H_N` times the average load:

| `N` (one token each) | `H_N` (max arc ÷ average arc) | Meaning |
| --- | --- | --- |
| 10 | 2.929 | worst node ≈ 2.9× the fair share |
| 100 | 5.187 | worst node ≈ 5.2× the fair share |
| 1000 | 7.485 | worst node ≈ 7.5× the fair share |

A one-token-per-node ring is *arithmetically correct* — every key is found, every client agrees, and the ring's remapping behaviour is exactly as derived above — and simultaneously unusable, because the node that owns the largest arc serves several times the average load no matter how uniform the request distribution is. That is the problem §4 solves, and it is the reason virtual nodes are not an optimisation but a precondition.

## 4. Balance and Virtual Nodes

### 4.1 Why a small number of tokens gives a badly uneven ring

The ring does not distribute *keys* by any rule. It distributes *arcs*, and the arcs are whatever the hash function produced. Load per node is proportional to arc length, so the load distribution is the **arc-length distribution** — a random object that nobody chose. With `N` tokens on the circle the `N` arcs sum to `1` and have mean `1/N`, but their *spread* is large relative to that mean: the largest of them has expected length `H_N/N` (§3.4), so the unluckiest node carries `H_N` times the average.

The consequence is stated most starkly in the single-token case at `N = 100`: the average node carries 1% of the keyspace, and the worst node is expected to carry `H_100/N = 5.187/100 = 5.187%`. No traffic pattern caused this. It is a property of how 100 random numbers happened to land.

Two things follow, and both are operational:

- **Ring imbalance and request imbalance are different animals.** Arc imbalance is a *node* problem that the ring itself introduces; request imbalance from popularity is a *key* problem the ring cannot touch (§7.1). Teams routinely measure the second and blame the first, or fix the first and expect the second to improve.
- **Imbalance is measurable in advance.** From the token list alone — no traffic, no load test — you can compute every arc length and therefore the exact *static* share of the keyspace that each node owns. That computation belongs in the deployment pipeline, and §11.3 makes it the primary ring metric.

### 4.2 The mechanism: how virtual nodes fix it, and the 1/√R relation

Give each node `R` tokens instead of one. Total tokens `T = N·R`; the average arc is `1/T`; a node's share is the **sum of `R` arcs** it owns. The arcs are spread around the circle, so the R arcs belonging to one node are (to a good approximation) independent draws from the spacing distribution. That gives the whole mechanism in three lines — the unit chain matters, so it is written out:

```
Model: T = N·R arcs, approximately i.i.d. exponential with mean 1/T
       (the spacings of T uniform random points are Dirichlet(1,…,1), and each
        marginal is Beta(1, T−1), which for large T behaves like Exp(mean 1/T))

Mean share of one node   E[X] = R × (1/T) = R × 1/(N·R) = 1/N          ← unchanged: still the fair share
Variance of one arc      Var = (1/T)²
Variance of a node's share Var(X) = R × (1/T)² = R/(N·R)² = 1/(N²·R)
Standard deviation       sd(X) = √(1/(N²·R)) = 1/(N·√R)
Relative spread          sd(X)/E[X] = (1/(N√R)) / (1/N) = 1/√R
```

**The node's share is on average exactly the fair share `1/N`, and its relative dispersion falls as `1/√R`.** That is the entire physics of virtual nodes in one expression, and it is why the mechanism is a *variance* reduction rather than a redistribution: adding tokens does not change the mean, it shrinks the standard deviation around it.

| `R` (tokens per node) | Relative spread `1/√R` | Read as: typical deviation of a node's load from its fair share |
| --- | --- | --- |
| 1 | 1.000 (100%) | a node's load is essentially unconstrained |
| 10 | 0.316 (31.6%) | ±1/3 of the fair share is routine |
| 100 | 0.100 (10.0%) | ±10% is routine |
| 256 | 0.0625 (6.25%) | ±6% is routine |
| 1000 | 0.0316 (3.16%) | ±3% is routine |

Worked at a realistic size. `N = 100` nodes, `R = 256` tokens per node, `T = 25,600` tokens: fair share `1/N = 1%` of the keyspace; relative spread `1/√256 = 1/16 = 6.25%`; so the standard deviation of a node's share is `1% × 0.0625 = 0.0625` percentage points of the keyspace. Because the distribution is near-normal in the aggregate, a node running `3σ` above the mean is to be expected somewhere in a fleet of 100: that is `3 × 6.25% = 18.75%` above its fair share. **A node at 118% of fair share at `R=256` is not a bug; it is the arithmetic.** The bug is a node at 200% of fair share, which is what `R=1` produces on a routine basis.

Three honest caveats, because this model is a model:

1. It assumes the hash function distributes tokens like uniform random points. A weak hash — or one applied to identity strings that share long prefixes — violates this directly, and no amount of `R` repairs it.
2. It assumes the `R` arcs of a node are independent. They are not exactly (they are draws from the same circle without replacement), but the correlation is `O(1/T)` and vanishes as `T` grows. The `1/√R` scaling is therefore a good description and not a bound.
3. It says nothing about *dynamics*. The dispersion above is a static property of a settled ring; the failure-and-recovery path has its own behaviour (§7.2, §7.3).

### 4.3 What Dynamo actually reports

Dynamo is where the industry learned this mechanism, and its authors state the problem and the fix in their own words. Quoted verbatim from the SOSP'07 PDF (`allthingsdistributed.com`, read 2026-09-28):

> "The basic consistent hashing algorithm presents some challenges. First, the random position assignment of each node on the ring leads to non-uniform data and load distribution. Second, the basic algorithm is oblivious to the heterogeneity in the performance of nodes. To address these issues, Dynamo uses a variant of consistent hashing … instead of mapping a node to a single point in the circle, each node gets assigned to multiple points in the ring. To this end, Dynamo uses the concept of 'virtual nodes'." ✅

and the three advantages it claims for virtual nodes, verbatim:

> "• If a node becomes unavailable (due to failures or routine maintenance), the load handled by this node is evenly dispersed across the remaining available nodes.
> • When a node becomes available again, or a new node is added to the system, the newly available node accepts a roughly equivalent amount of load from each of the other available nodes.
> • The number of virtual nodes that a node is responsible can decided based on its capacity, accounting for heterogeneity in the physical infrastructure." ✅ (the second bullet reads exactly as printed; the missing "be" is the paper's)

On measurement, Dynamo defines its own metric verbatim — "Load balancing efficiency is defined as the ratio of average number of requests served by each node to the maximum number of requests served by the hottest node" ✅ — and reports the comparison of its three partitioning strategies at `S=30` nodes and `N=3` replicas, concluding that strategy 3 "achieves the best load balancing efficiency and strategy 2 has the worst" ✅, with strategy 3 reducing membership information "by three orders of magnitude" relative to strategy 1 ✅.

**Flagged, not quoted:** Dynamo's load-balancing-efficiency results are presented as **Figure 8**, a plot of efficiency against membership size as `T` and `Q` vary. The figure is a graphic; its per-parameter values could not be read as text from the PDF this pass, so **this guide does not state numeric per-`T` values for Dynamo's experiments**. Anyone needing those numbers must read the figure. See §16.

### 4.4 Why many small arcs beat one big arc

The benefit of vnodes is not only statistical balance; it is *how a departure is absorbed*. Take a node `D` out of a ring of `N` nodes and follow the keyspace it owned.

**Case `R = 1` (one arc).** `D` owns one arc of expected length `1/N`. Its successor inherits the whole arc. That successor was already carrying its fair share of `1/N`; it now carries `1/N + 1/N = 2/N`. In unit terms, using `N = 100`:

```
fair share per node      = 1/100 = 1% of the keyspace
arc inherited by the one successor = 1% 
successor's new load     = 1% + 1% = 2% = 200% of fair share
```

So the loss of one node with `R=1` is expected to **double** the load on one surviving neighbour. In a cache tier that neighbour is now hot, starts evicting under pressure, and its hit ratio falls — the loss propagates.

**Case `R = 256` (many arcs).** `D` owns 256 arcs, each of expected length `1/(N·R) = 1/25,600 ≈ 0.0039%` of the keyspace. Each arc is absorbed by *its own* successor. Those successors are (approximately) distinct nodes, so the `1%` total is divided across the survivors:

```
total absorbed = 1/N = 1% of the keyspace
successors that absorb it ≈ min(R, N−1) = 99 distinct surviving nodes
extra load per affected survivor = 1% / 99 ≈ 0.0101% of the keyspace
relative increase on each survivor = 0.0101% / 1% ≈ 1.01%
```

Against the `R=1` case's `+100%` on a single node. The keyspace that moves is **the same `1/N` in both cases** — vnodes do not reduce how much data moves when a node dies; they change *how many nodes receive it*. This is precisely Dynamo's claim above ("the load handled by this node is evenly dispersed across the remaining available nodes") ✅ and Cassandra's ("when a node is decommissioned, it loses data roughly equally to other members of the ring, again keeping equal distribution of data across the cluster" ✅). The same reasoning applies in reverse on join: a new node with many tokens "steals" many small pieces from many peers rather than one large piece from one peer, which is why the streaming load is spread and the ring re-balances in one pass.

### 4.5 What virtual nodes cost

Vnodes are not free, and the costs are documented — by the products that use them, in their own words.

1. **More membership metadata.** `T = N·R` token entries, gossiped. Dynamo's own operational response was to move to a scheme where membership information shrank "by three orders of magnitude" ✅. Doubling `R` doubles the metadata.
2. **More ring neighbours, and therefore a larger availability blast radius.** Quoted verbatim from `cassandra.apache.org`'s architecture documentation (5.0 docs page, read 2026-09-28):

   > "Every token introduces up to `2 * (RF - 1)` additional neighbors on the token ring, which means that there are more combinations of node failures where we lose availability for a portion of the token ring. The more tokens you have, the higher the probability of an outage." ✅

   This is the cost most often missed in ring designs: `R` is not merely a balance knob, it is an **availability** knob, and it moves in the wrong direction. The same documentation lists two further costs verbatim: "as the number of tokens per node is increased, the number of discrete repair operations the cluster must do also increases" ✅ and "Performance of operations that span token ranges could be affected." ✅
3. **Membership changes become multi-partner operations.** A joining node with `R` tokens streams `R` separate ranges from `R` different peers; with `R = 1`, one range from one peer. Bootstrapping logic, throttling and progress reporting all get harder, and a partially-completed bootstrap across 256 ranges is a worse state to be in than one across 1.
4. **Token placement must be coordinated, which is a new dependency.** Random token selection is what forces high `R`; deterministic allocation is what allows low `R`. Cassandra states the history verbatim: "Note that in Cassandra `2.x`, the only token allocation algorithm available was picking random tokens, which meant that to keep balance the default number of tokens per node had to be quite high, at `256`. This had the effect of coupling many physical endpoints together, increasing the risk of unavailability. That is why in `3.x +` a new deterministic token allocator was added which intelligently picks tokens such that the ring is optimally balanced while requiring a much lower number of tokens per physical node." ✅ — and the shipped configuration in the project's own `conf/cassandra.yaml` (4.1 branch, read 2026-09-28) sets `num_tokens: 16`, with the explanatory comment that "The load assigned to each node will be close to proportional to its number of vnodes", and `allocate_tokens_for_local_replication_factor: 3` noted as "Only supported with the Murmur3Partitioner" ✅. Note the shape of that history: **two very different token counts — 256 and 16 — are both "the answer"**, depending on whether token placement is random or allocated. A vnode count copied from another system's documentation is therefore not a configuration decision, it is a guess (§14).

## 5. Replication on the Ring

### 5.1 The naive clockwise walk

The standard rule: hash the key to a token, walk **clockwise** to the first token, and that node is the primary. For replicas, continue clockwise and take the next distinct *physical* nodes until the replication factor is met. Cassandra's documentation states the mechanism verbatim, including the worked example:

> "For example, if we have an eight node cluster with evenly spaced tokens, and a replication factor (RF) of 3, then to find the owning nodes for a key we first hash that key to generate a token (which is just the hash of the key), and then we 'walk' the ring in a clockwise fashion until we encounter three distinct nodes, at which point we have found all the replicas of that key." ✅

Dynamo calls the result the **preference list**, and adds the crucial vnode corollary verbatim:

> "To account for node failures, preference list contains more than N nodes. Note that with the use of virtual nodes, it is possible that the first N successor positions for a particular key may be owned by less than N distinct physical nodes (i.e. a node may hold more than one of the first N positions). To address this, the preference list for a key is constructed by skipping positions in the ring to ensure that the list contains only distinct physical nodes." ✅

So the walk is a **distinctness** rule: it guarantees the replicas are different *nodes*. It says nothing at all about where those nodes physically are.

### 5.2 What naive clockwise replication gets wrong

**Distinctness is a set property. Independence is a physical property. They coincide only when a design makes them coincide.** The clockwise walk operates on token order, which is a hash artifact. Two tokens adjacent in the ring are adjacent for no reason that has anything to do with racks, hosts, power feeds, switches or availability zones.

Concretely, the naive walk can produce:

- **Three replicas inside one failure domain.** `node-a`, `node-b` and `node-c` can be three virtual machines on one hypervisor in one rack behind one top-of-rack switch. Hashing them into a ring and walking clockwise produces a replica set of three — which a single rack failure removes in its entirety. The ring did not make an error; it answered the question it was asked ("distinct nodes?") and the design asked the wrong question.
- **Accidental, unverifiable independence.** With vnodes, a node's tokens are scattered, so the "next three distinct nodes" are effectively random choices among survivors. That *tends* to spread replicas around, which is why ring systems often look robust in practice. But the operator cannot *read* the ring and see the safety margin: co-location is invisible in token order. Robustness that cannot be demonstrated is robustness that cannot be reviewed — which is precisely the problem in a regulated estate (§12).
- **Small clusters with a large coverage footprint.** With low `R` and few nodes, each replica set covers a large contiguous stretch of keyspace owned by nodes that are likely to be co-located. The blast radius of one rack loss is a large fraction of the keyspace rather than a fraction equal to that rack's share.
- **Replica sets that shift as tokens change.** Because replicas are "the next `RF` distinct nodes from this token", any change in token placement rewrites replica sets for the affected arcs — which means the availability argument has to be re-checked after every topology change, not just after the initial design.

### 5.3 Making placement failure-domain aware

The fix is to make the placement rule *topology-aware*, and the products that learned this document it. Cassandra's own documentation, verbatim:

> "Replicas are always chosen such that they are distinct physical nodes which is achieved by skipping virtual nodes if needed. Replication strategies may also choose to skip nodes present in the same failure domain such as racks or datacenters so that Cassandra clusters can tolerate failures of whole racks and even datacenters of nodes." ✅

Three patterns, in increasing order of explicit guarantee:

1. **Skip-on-walk (rack-aware walk).** Keep the clockwise walk, but skip any candidate whose failure domain already appears in the replica set. This is the minimal change and it is what the quoted behaviour describes: replicas are distinct nodes, *and* the strategy may skip nodes in the same rack or datacenter. The property is: "no two replicas share a failure domain as long as enough distinct failure domains hold tokens." That last condition is the catch — the walk can only find a distinct rack if a distinct rack owns a token on the path.
2. **Per-failure-domain arcs.** Assign tokens so that each failure domain owns a set of arcs, and place one replica per failure domain by construction. Placement becomes a two-level decision — pick the set of failure domains, then pick a node within each — which is readable, testable, and has a coverage guarantee that does not depend on the hash landing favourably.
3. **Verify against inventory, not against the ring.** Whatever the rule, assert the invariant against the *actual* topology inventory (rack/zone/host mapping from the CMDB or cloud API) and fail the deployment if any replica set violates it. This is the only version that survives a node being *moved* between racks without a token change.

**State plainly what naive replication gets wrong, in one sentence:** it guarantees that replicas are different processes on the ring and offers no guarantee whatsoever that they are in different failure domains, so a design that equates "RF=3" with "survives a rack loss" — without checking that the three replicas are in three racks — has a single point of failure wearing a replica's uniform.

### 5.4 The preference list and the arithmetic of replica count

Three consequences follow from the walk that belong in every ring design review:

- **The preference list is longer than the replica count.** Dynamo: "preference list contains more than N nodes" ✅ — the extra successors are the handoff targets for hinted handoff and for replica recovery. Removing a node from the ring does not lose its data only because these successors already know to accept the writes.
- **Neighbour count scales with `RF` and token count.** Cassandra's "up to `2 * (RF - 1)` additional neighbors" ✅ per token is the availability arithmetic: the more replicas and the more tokens, the more failure combinations that remove a replica for some part of the keyspace.
- **`RF` versus failure-domain count is a hard constraint, not a tuning choice.** With 3 availability zones and `RF = 3`, one replica per zone is the only arrangement that keeps a quorum after any single-zone loss. With `RF = 2` in a 3-zone cluster, any key whose two replicas happen to be in the same two zones is unavailable if one of those zones is lost — and the number of keys in that position is a placement outcome, not a number you chose. For what a quorum then promises, see `tunable_consistency_databases_guide.md`; the ring's job is only to decide *which* nodes are in the set.

## 6. Weighting and Heterogeneous Nodes

### 6.1 The lever: token count is the weight

Nothing about a ring requires nodes to be identical. Capacity differences are expressed through **token count**, because token count is proportional to expected arc length, which is proportional to expected load. Every implementation that supports weighting does it this way:

- **Ketama** puts the weight in the server file and documents it verbatim: the file lines are "ip:port and weighting, `\t` separated, `\n` line endings. Just use the number of megs allocated to the server as the weight. The weightings are realised by adding more or less points to the continuum." ✅
- **Dynamo** states the intent verbatim: "The number of virtual nodes that a node is responsible can decided based on its capacity, accounting for heterogeneity in the physical infrastructure." ✅
- **Cassandra** states the arithmetic and the default policy verbatim in its own configuration file (4.1 branch): "This defines the number of tokens randomly assigned to this node on the ring. The more tokens, relative to other nodes, the larger the proportion of data that this node will store. You probably want all nodes to have the same number of tokens assuming they have equal hardware capability." ✅, and for the deterministic allocator: "The load assigned to each node will be close to proportional to its number of vnodes." ✅

The unit chain for a weighted ring is short:

```
tokens(node i) = R_i ;  T = Σ R_i
expected share of node i = R_i / T   ≈  keyspace fraction owned  ≈  request share (if cost per key is uniform)
weight ratio  R_i / R_j = 2  ⇒  node i is expected to carry ≈ 2× the load of node j
```

The last clause is where weighting goes wrong. Token count is proportional to **the number of keys**, not to the **cost of serving them**. If node `i` carries the tokens for a tenant whose reads are 10× more expensive, then doubling its token count does not make it twice as useful — it makes it twice as loaded for the same throughput. Weighting is keyspace weighting.

### 6.2 Drift: the failure is a stale weight, not a wrong one

Weighting is set once, at commissioning, under a policy, and then reality moves underneath it. The recurring mechanisms:

| Mechanism | What happens | Why it is easy to miss |
| --- | --- | --- |
| Hardware refresh | New hosts are brought in with a default token count while legacy hosts keep the old one; the intended capacity ratio is never re-expressed | The ring remains *valid* — every key has an owner — so no alert fires |
| Emergency replacement | A failed node is replaced from a stock image with the shipped default `num_tokens`, not the fleet's intended value | The incident is over; the imbalance is permanent and small enough to be attributed to "noise" |
| Role change | A node's workload changes (a heavier tenant lands on it) without a corresponding token change | Request cost per key changed, which token weighting does not model (§6.1) |
| Config copied between environments | Token counts or weight tables are part of a config file (ketama's server file with its weight column ✅), and configs get cloned | Works fine in test; silently mis-weights production |
| Never revisited | The capacity ratio is decided once and the fleet is scaled by adding identically-weighted nodes | Correct at the time, wrong two years later, never wrong enough to trigger an alert |

**The honest symptom** of a drifted ring is a persistent, per-node imbalance that survives restarts and does not correlate with request-rate variance — a node that is hot at low load, or a node that is cold while its neighbours throttle. That is not a hot key (§7.1) and not a hot node from arc variance (§4.1): it is a declared weight that no longer matches the machine underneath it.

**The guardrail** is to make the token count a *derived* value: declare each node's capacity class, derive the token count from the class and a documented ratio, store the derivation under version control, and add a check that compares declared share against observed share continuously (§11.3) rather than relying on someone reading a YAML diff.

### 6.3 Weight changes are themselves data movement

There is no free re-weighting. Changing a node's token count changes which arcs it owns, and every arc that changes owner means data moving between nodes — the same movement as adding or removing a node, in smaller parcels.

The arithmetic, on a ring of `T = N·R` tokens where the keyspace is `1.0`:

```
one token added to one node     takes an expected 1/T of the keyspace
at N = 100, R = 256, T = 25,600:  1/25,600 ≈ 0.0039% of the keyspace
```

| Tokens added to one node | Keyspace moved (expected) | Derivation |
| --- | --- | --- |
| 1 | 0.0039% | 1/25,600 |
| 32 | 0.125% | 32/25,600 |
| 256 | 1.00% | 256/25,600 = 1/100 — one full node's fair share, as it must be |

Two operational notes, both derived from the construction in §3.1 and both easy to get wrong:

1. **Adding tokens is bounded; renumbering tokens is not.** The position of a token is `H(identity)`. Adding new tokens with *new* identities moves only the arcs those tokens claim — bounded by the table above. Renumbering existing tokens, or changing the identity string (for example from `host:port#3` to a renumbered scheme), moves every affected token to a fresh position, which is a *rehash*: it can move an arbitrary and potentially very large fraction of the keyspace. "Rebalancing" that renumbers tokens is a ring replacement, not a rebalance.
2. **A fleet-wide re-weight is a fleet-wide migration.** Recomputing every node's token count to match a new capacity profile is bounded by the *sum* of the changes, and it is best executed as a sequence of bounded steps with the capacity headroom of §11.1 — never as one coordinated change.

## 7. Where It Fails

The section order in this guide is deliberate. Six sections have explained what the technique does; this one explains what it cannot do, because the second list is the one that decides whether the design survives. Each failure below gets a mechanism and a verdict: **the ring helps**, **the ring is neutral**, or **the ring is irrelevant** — and "irrelevant" is the dangerous one, because irrelevant techniques get deployed as if they were fixes.

### 7.1 Hot keys — the ring is irrelevant

**The mechanism in one sentence:** the ring distributes **nodes** across arcs; it says nothing about the distribution of **popularity** across keys, and a hot key lands on exactly one node no matter how well balanced the ring is.

Work the unit chain. Let one key account for a fraction `p` of the request stream. Its owner receives `p` of the tier's requests from that key alone, while its *fair* share of the stream is `1/N`:

```
N = 100 nodes, p = 0.10 (one key absorbing 10% of requests)
the owning node's fair share          = 1/100 = 1%
the same node's load from this key    = 10%
ratio to fair share                   = 10×  (from one key, before any traffic on the node's arcs)
```

Now change `R`, the token count, and the ratio does not move. `R` controls the *variance of arc lengths* — a property of the keyspace partition (§4.2). A hot key's arc is one arc; adding tokens adds arcs to nodes, not splits to keys. `R = 1` and `R = 10,000` place that key on the same node with the same probability distribution: one node, one arc, all of the requests. This is the mechanism behind the sentence in `distributed_systems_engineering_guide.md` §7.4 — *virtual nodes fix hot nodes, not hot keys* — and this guide does not contradict it.

The same reasoning makes the mirror-image mistake visible. A team that sees one node hot has two independent hypotheses — that node's **arc** is too large (a ring problem, fixable by re-weighting or more tokens), or a **key** is too popular (not a ring problem at all, because the load is intrinsic to the key). The discriminator is the *arc histogram* versus the *request histogram by key*: if the node's arc is normal-sized and its request rate is high, the ring is not the cause and no ring change will be the cure.

**What actually fixes a hot key** (none of it is the ring):

| Fix | What it does | What it costs |
| --- | --- | --- |
| Dedicated partition / pinned route | A single key (or key family) gets its own node or its own capacity, bypassing the ring for that key | Node count stops being uniform; capacity becomes a per-key decision |
| Key splitting / aggregation | Rewrite one logical key into `k` physical keys (`key#0 … key#k−1`) so `k` nodes share the load; the reader aggregates | Every reader must know `k` and aggregate; writes must fan out; the ring is unchanged but the *data model* changed |
| Client-side / local caching | The hottest keys are served from the caller's own process, removing those requests from the tier entirely | Cache invalidation moves to the client; correctness depends on TTL discipline |
| Request coalescing (single-flight) | `m` concurrent identical misses become one origin read; the other `m−1` wait for its result | Needs a per-key in-flight map; adds a small wait, removes a large amplification |
| Load shedding / admission control | Under overload, the tier refuses or degrades a controlled fraction of traffic rather than collapsing | Requires the business to accept partial unavailability as a feature |
| Bounded-load placement | Cap the per-node load and let keys spill to the next node on the ring | Placement stops being a pure function of the key; see §8.4 |

The last row is the only one that is a change to the *placement technique* rather than a change to the data model or the client. Google's formulation of it — "Consistent Hashing with Bounded Loads" — states the cap verbatim: "we take an arbitrary user specified balancing parameter `c = 1+ε > 1`. With `m` balls and `n` bins in the system, we want no load above `⌈cm/n⌉`" ✅, and the headline result verbatim: "with `n` clients and `n` servers, we get a guaranteed max-load of 2 while only moving an expected constant number of clients for each update" ✅ (Mirrokni, Thorup, Zadimoghaddam, arXiv:1608.01350; v1 3 Aug 2016, v3 27 Jul 2017, verified from the arXiv abstract page). Note what it buys and what it charges: it *caps* node load, which is a genuine answer to a hot **node**; for a single hot **key** the cap does not help either, because the spilling decision applies to which node receives a key, and a key that is 10% of traffic still has to go somewhere. Bounded loads raises the priority of a better answer to a hot key: making the hot key's load *someone else's problem* by splitting it.

### 7.2 Cold cache and cold start — the ring is partly helpful, and the help is smaller than it looks

**The mechanism:** ownership changes assign a node responsibility for keys whose data it does not physically hold. Every changed key is a **miss on first touch** when the layer is a cache.

The fractions are §3.3's, and they are not the interesting part. The interesting part is the **rate**:

```
adding one node to a ring of N=100 moves 1/101 ≈ 0.99% of the keyspace
at 200,000 requests/second on the tier, that is ≈ 1,980 extra cache misses per second,
delivered to the origin at whatever rate the request stream delivers them
```

The misses do not trickle — the arcs are remapped *at once*, so the origin sees a step increase proportional to the moved fraction immediately, with no ramp. Three amplifiers follow:

- **Cache stampede / thundering herd on a single key.** If `m` concurrent requests for the *same* key all miss, all `m` go to the origin unless coalesced. This is the same failure as §7.1's hot key, triggered by a cold start rather than by popularity.
- **The new node is cold for its whole arc.** It has none of its `1/(N+1)`; the first request for each of those keys is a miss *and* a write to the new node. So the moved fraction of the keyspace generates both read misses and write amplification.
- **Re-warming competes with serving.** The origin must absorb the miss traffic *and* the re-warming reads; if the origin saturates, the cache never re-fills and the state persists (§2.4's feedback loop).

**What mitigates it, in order of effectiveness:**

1. **Prewarm before the node takes traffic.** Because the ring is a derived artifact (§3.1), the exact arcs the new node will own are *computable in advance* from the token list. For a storage system that means streaming those ranges before announcing the node; for a cache it means reading the arcs from the origin — earlier, deliberately, at a controlled rate, instead of all at once the moment the ring changes. The ring makes the *target set* knowable, which is why prewarming is even possible; that is the "ring is partly helpful" claim, and it is the whole of the help.
2. **Slow ramp.** Do not let the new node take 100% of its arc immediately. Envoy documents the general mechanism for endpoints verbatim: "Slow start … allowing new or recovered endpoints to ramp up traffic gradually" ✅ (from its client-side weighted round-robin documentation, read 2026-09-28). On a ring this is a re-weighting ramp (§6.3) — its cost is precisely the data movement it defers.
3. **Serve-stale / stale-while-revalidate.** Return the last known value past its TTL instead of a miss, when the business tolerates it. This converts an origin-load event into a freshness event, which is usually the better trade — but it must be a *decision*, not a default.
4. **Coalesce requests** per key so one miss serves many callers (single-flight).
5. **Hold origin headroom for the duration of the change.** This is a capacity-planning commitment: `N/(N+1)` extra origin load in the modulo case, `1/(N+1)` in the ring case, for the duration of the migration or failure.

**The honest summary:** the ring reduces the moved fraction from `N/(N+1)` to about `1/(N+1)`, and therefore reduces the cold-start miss spike by a factor of `N` — the same factor as everywhere else. It does not eliminate it. A ring-based cache tier that has no prewarm, no coalescing and no origin headroom still has an outage; it just has a smaller one, and it has it at a smaller, more dangerous scale where nobody notices until the fleet grows.

### 7.3 Node churn and flapping membership — the ring makes the churn cheap to compute, which is not the same as cheap

**The mechanism:** every membership change is a new ring version. The *control plane* recomputes ownership cheaply — that is the design goal — but the *data plane* still has to move data and the *clients* still have to receive and adopt the new version. Churn multiplies all three.

- **Movement that never completes.** A node that joins, fails and rejoins repeatedly causes repeated ownership changes. Each change moves `1/(N+1)` (join) or `1/N` (leave) of the keyspace, and each movement takes real time — streaming gigabytes takes minutes to hours. If flapping happens faster than a movement completes, the system spends its life in a partially-moved state: lower hit ratios, split key ownership, and permanent background I/O. The arithmetic is a race between two clocks, and the ring does not lengthen either one.
- **Overlapping rebalances.** Starting rebalance #2 before #1 completes means two ownership changes in flight against one dataset; the correct discipline is to serialize them, which in turn means a long tail of small, slow corrections rather than one decisive operation.
- **Headroom at the destination.** A node that inherits an arc must hold its own data *and* the incoming arc during movement, so the destination needs transient capacity above steady state. A rebalance started with no headroom turns a capacity event into a node failure.
- **Adverse interaction with health checking.** A heavily loaded node misses a heartbeat; membership declares it gone; its arcs are reassigned and movement begins; it comes back healthy and its arcs are reassigned back. Each cycle costs movement and re-warming. The ring is *more* sensitive to this than a static modulo assignment, because under modulo a member-set change is already catastrophic whereas a ring change is — deliberately — small enough to *seem* harmless.

**Guardrails, stated as ring-specific decisions:** use a slower failure-detection window for *ring membership* than for *request routing*, so a momentarily slow node is not removed from the ring; require a completed-movement barrier before the next rebalance; require measured headroom before any membership change; and cap the rate of membership changes per hour as an explicit operational limit, since the ring has removed the natural friction that used to prevent it.

### 7.4 Multi-key and cross-partition operations — the ring is structurally irrelevant

**The mechanism:** the ring partitions by a single hash of a single key. Any operation that must touch more than one key has to be decomposed by the caller, and the ring offers no assistance whatsoever — it is not that cross-partition work is hard *on a ring*; it is hard on *any* key-hash partitioning, and the ring's inability to place related keys together is a direct consequence of hashing keys.

| Operation | What the ring does to it |
| --- | --- |
| Multi-get of `k` keys | Fan-out to up to `k` nodes; the latency is the slowest response, and the failure probability compounds with `k`. Coalescing partial results across a node failure is the caller's problem. |
| Transactions spanning keys | Not provided. Cassandra's documentation states the lineage plainly: "Cassandra, like Dynamo, chooses not to provide cross-partition transactions that are common in SQL Relational Database Management Systems (RDBMS)" ✅. |
| Joins | Nothing at the ring level. The analytical equivalents are the shuffle/co-partitioning problems of batch engines — see `spark_tuning_guide.md` §11 — and they arrive at the same conclusion by a different route. |
| Secondary-attribute lookups | A secondary index is keyed by a different attribute, so the ring's partition key carries no information about it; the lookup is either a scatter-gather across all partitions or a second, independently partitioned index (with the same hot-key exposure, §7.1). |
| Multi-key commands | Only work if the keys happen to co-locate. Redis Cluster states both the mechanism and the limit verbatim: "Redis Cluster implements a concept called **hash tags** that can be used to force certain keys to be stored in the same hash slot. However, during manual resharding, multi-key operations may become unavailable for some time while single-key operations are always available." ✅ |

**The trade, stated once:** the ring buys *locality per key* and gives up *everything that spans keys*. The mechanisms that buy multi-key support back — hash tags, explicit co-location, a single-slot keyspace — do so by *deliberately creating a hot spot*: keys accessed together are placed together, so their load lands together. That is not a bug in those mechanisms; it is the unavoidable price of locality, and it should be a documented decision rather than an accident (§14).

## 8. The Alternatives

Four alternatives are in serious production use. Each is described by what it actually does, what it trades away, and its primary source with its authors and year. Two of them are not rings at all, and the honest conclusion is that "consistent hashing" is a *family* of placement schemes with different engineering trade-offs, not one algorithm.

### 8.1 Rendezvous hashing (highest-random-weight, HRW)

**What it does.** For a key `k` and nodes `1..N`, compute `w_i = H(k, node_i)` for **every** node, and choose the node with the highest (or lowest, consistently) value. There is no ring, no token, and no partition of the keyspace: the "placement" is the argmax of `N` independent hashes.

**Why it behaves like a ring, without being one.** Each key independently picks its own winner. Remove node `j`: only the keys for which `j` was the argmax change, which is `≈ 1/N` of keys — the same fraction a ring moves. Add a node: each key independently has probability `1/(N+1)` of having the new node as its argmax, so `≈ 1/(N+1)` of keys move — again the same fraction. So rendezvous has the ring's cardinal property (minimal movement) with **no ring state to maintain at all**.

**What it trades away.**

- **Lookup cost is `O(N)` hashes per key**, against `O(log T)` for a ring array. At `N = 100` that is 100 hash evaluations per lookup instead of ~7 comparisons; at `N = 5,000` per lookup it is the dominant cost of the hot path. This is the single reason rings (and tables) are used in large fleets.
- **No tuning knob for balance or weighting.** There are no vnodes to add, so the balance is whatever the argmax distribution gives (the same `Θ(log N / N)` maximal-load behaviour as a one-token-per-node ring, with no `R` to reduce it). Weighted variants exist that alter the comparison — a weight is applied as an exponent or as repeated entries — but the mechanics are outside what this pass verified, so they are flagged rather than described (§16).
- **The node list must be complete and identical on every client.** Rendezvous has no partial view: a client that is missing one node computes a *different* argmax for the keys that node would have won.

**Primary source.** David G. Thaler and Chinya V. Ravishankar, "Using Name-Based Mappings to Increase Hit Rates" — Microsoft Research technical report MSR-TR-98-14, and the journal version in *IEEE/ACM Transactions on Networking*, vol. 6, no. 1 (February 1998). The title-page author list was confirmed this pass from the paper's own first page ("David G. Thaler, Student Member, IEEE, and Chinya V. Ravishankar, Member, IEEE") via two independent renderings ⚠ — the MSR PDF and the IEEE Xplore page were both unreachable directly; the venue and volume are therefore flagged in §16 rather than ✅.

### 8.2 Jump consistent hash

**What it does.** Not a ring and not a table: a 5-line function of the key and the bucket count that returns a bucket index in `[0, n)`, with the property that increasing `n` by one moves only the keys belonging to the new bucket. Lamping and Veach's abstract makes both the properties and the limitation explicit, verbatim:

> "We present jump consistent hash, a fast, minimal memory, consistent hash algorithm that can be expressed in about 5 lines of code. In comparison to the algorithm of Karger et al., jump consistent hash requires no storage, is faster, and does a better job of evenly dividing the key space among the buckets and of evenly dividing the workload when the number of buckets changes. Its main limitation is that the buckets must be numbered sequentially, which makes it more suitable for data storage applications than for distributed web caching." ✅

Verified from the arXiv abstract page (arXiv:1406.2294, submitted 9 June 2014) ✅.

**The trade.** It wins outright on memory (`no storage` — there is no token array at all, so no metadata to gossip, no ring to distribute, no consistency problem in the ring itself) and on balance quality (the authors' claim that it divides the key space and the workload *better* than Karger et al. is the abstract's own, ✅, and it is consistent with the `1/√R` analysis of §4.2: a scheme that spreads every key independently has no `1/√R` penalty to pay). It loses on **structure**:

- **Buckets must be numbered sequentially**, so the bucket set is `{0, 1, …, n−1}`, not an arbitrary set of nodes. Adding capacity means appending bucket `n`. Any change that is not an append — replacing the node in bucket 4, retiring bucket 7 out of the middle — either cannot be expressed or requires renumbering, and renumbering changes the function's input for every key that mapped to a numbered bucket. A mid-set removal is therefore a near-total remap: a derived consequence of the numbering requirement, not a quoted result.
- **No weighting.** There is no weight parameter and no token count; a bucket is a bucket.
- **Node identity is positional.** Bucket `i` means "whoever is in position `i`", so the placement depends on an external ordered list that must be kept identical everywhere.

That combination is exactly right for a shard index in a storage system (append-only, numbered, no weighting) and exactly wrong for a request-routing layer over a changing set of hosts, which is what the authors say it is.

### 8.3 Maglev hashing

**What it does.** A precomputed lookup table of fixed size maps every entry to a backend; a lookup is a single table index, so it is `O(1)` with no search. The table is built by placing each backend into the table, weighted by its configured share, until the table is full. Verified from Google's own paper record: the USENIX NSDI '16 abstract states verbatim that "Maglev is also equipped with consistent hashing and connection tracking features, to minimize the negative impact of unexpected faults and failures on connection-oriented protocols", and that "Maglev has been serving Google's traffic since 2008" ✅. Authors verified at the source: Danielle E. Eisenbud, Cheng Yi, Carlo Contavalli, Cody Smith, Roman Kononov, Eric Mann-Hielscher, Ardas Cilingiroglu, Bin Cheyney (Google), Wentao Shang (UCLA), Jinnah Dylan Hosein (SpaceX); *13th USENIX Symposium on Networked Systems Design and Implementation (NSDI 16)*, 2016, pp. 523–535 ✅.

**The trade, in Envoy's own words** (Envoy implements both schemes, so its comparison is a first-class operational statement, quoted verbatim from `envoyproxy.io`, read 2026-09-28):

> "In general, when compared to the ring hash ('ketama') algorithm, Maglev has substantially faster table lookup build times as well as host selection times (approximately 10x and 5x respectively when using a large ring size of 256K entries). While Maglev aims for minimal disruption, it is not as stable as ring hash when upstream hosts change. More keys will move position when hosts are removed (simulations show approximately double the keys will move). The amount of disruption can be minimized by increasing the `table_size`." ✅

So: `O(1)` lookup against `O(log T)`, substantially faster to build, at the price of moving roughly **twice** the keys on a host removal — and the price is tunable by growing the table. On the table itself, Envoy's documentation of the filling algorithm gives a concrete worked example, verbatim: "if host A has a weight of 1 and host B has a weight of 2, then host A will have 21,846 entries and host B will have 43,691 entries (totaling 65,537 entries)" ✅ — which is where this guide's "65,537 entries" figure comes from. Maglev's own paper is the authority on the table size; the paper PDF was not readable as text this pass, so the paper's own statement of the default is **not quoted here** (§16). Weighting on Maglev is therefore expressed as *table share*, which is the same keyspace-weighting semantics as ring tokens (§6.1).

### 8.4 Consistent hashing with bounded loads

**What it does.** Keeps the ring, and adds a cap. Rather than sending a key to the first node clockwise unconditionally, the placement respects a per-node load limit `⌈cm/n⌉` for user-chosen `c = 1+ε`, so a node that is already at its cap passes the key along rather than absorbing it. Verified verbatim from the arXiv abstract (Mirrokni, Thorup, Zadimoghaddam, arXiv:1608.01350) ✅:

> "The most popular solution for our dynamic settings is Consistent Hashing. However, the load balancing of consistent hashing is no better than a random assignment of clients to servers, so with `n` of each, we expect many servers to be overloaded with `Θ(log n / log log n)` clients… we get a guaranteed max-load of 2 while only moving an expected constant number of clients for each update." ✅

That first sentence is a useful corrective to §4: **plain consistent hashing's maximal load is `Θ(log n / log log n)`** — the same order as throwing balls at random — and the only fix in the classical scheme is more tokens (`1/√R`, §4.2), which does not *guarantee* a bound. Bounded loads does guarantee one. The paper also quantifies the price of the cap verbatim: moves increase "only by a multiplicative factor `O(1/ε²)` for `ε ≤ 1`" and "by a factor `1+O(log c / c)` for `ε ≥ 1`" ✅.

**The trade, stated precisely:** the guarantee requires the placement decision to depend on **current load**, not only on the key. That means the ring is no longer a pure function of `(key, membership)`: participants need load information, they need it fresh, and two participants that disagree about load can disagree about placement. It is a genuine answer to node overload — including the overload caused by arc imbalance — and it does not answer §7.1's hot key, because the key still has to be placed somewhere. The exact spill rule (which node is tried next, and what happens at the extreme) is a detail of the paper's algorithm that this pass verified only at abstract level; see §16.

### 8.5 The comparison that matters

| Property | Modulo | Ring, 1 token/node | Ring, `R` vnodes | Rendezvous (HRW) | Jump hash | Maglev | Bounded loads |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Object** | none | ring | ring | none | none | fixed table | ring + cap |
| **Balance quality** | uniform on keys, catastrophic on change | worst node ≈ `H_N`× average (§3.4) | relative spread `1/√R` (§4.2) | `Θ(log N / N)` max load, no knob | paper claims better than Karger et al. ✅ | weighted by table share | guaranteed max load 2 at `n=n` ✅ |
| **Lookup cost** | `O(1)` | `O(log T)` | `O(log T)` | `O(N)` hashes | `O(ln n)` expected iterations ⚠ | `O(1)` ✅ | `O(log T)` + load check |
| **Memory per client** | none | `O(N)` | `O(N·R)` | `O(N)` node list | **none** ✅ | `O(table size)` | `O(N·R)` |
| **Movement on add** | `N/(N+1)` | `1/(N+1)` | `1/(N+1)` | `1/(N+1)` | `1/(n+1)` | table rebuild, ≈ ring-ish | `O(1/ε²)` × ring ✅ |
| **Movement on remove** | `(N−1)/N` | `1/N` | `1/N` | `1/N` | not expressible (mid-set) | ≈ 2× ring hash ✅ | `O(1/ε²)` × ring ✅ |
| **Weighting** | no | no | yes — token count ✅ | with weighted variants ⚠ | no | yes — table share ✅ | via the cap |
| **Operational complexity** | trivial, and fatally fragile | trivial | token allocation is a real discipline (§4.5) | trivial; no ring to distribute | requires a stable numbered list | table must be identical everywhere; rebuild on change | needs a load channel between participants |

Read the table for the decision, not for a winner: **rendezvous and jump both remove the ring entirely and keep the movement property**, at the cost of lookup cost (`O(N)` hashes) or structural rigidity (numbered buckets). **Maglev buys constant-time lookup at the price of ~2× the movement and a table that must be rebuilt identically everywhere.** **Bounded loads is the only one that bounds the worst node**, and it does so by giving up determinism. And the ring remains the default because it is the only scheme that combines tunable balance (`R`), tunable weighting (§6), `O(log T)` lookup and `1/(N+1)` movement in one object.

## 9. The Implementations, Verified One by One

The rule for this section: **every scheme named here is the scheme that system's own documentation (or its own source) states.** Where a system's documentation could not be read this pass, the entry says so and no claim is made. Several popular beliefs about who uses what do not survive that discipline.

### 9.1 memcached: ketama, and it lives in the client

**What it actually is.** There is no consistent hashing in memcached's server. The ring lives in the **client library**, and the canonical one is `libketama` (and its language ports), whose own README describes both the problem and the scheme verbatim:

> "We wrote ketama to replace how our memcached clients mapped keys to servers. Previously, clients mapped keys->servers like this: `server = serverlist[hash(key)%serverlist.length];` **This meant that whenever we added or removed servers from the pool, everything hashed to different servers, which effectively wiped the entire cache.**" ✅

That sentence is §2 of this guide, written by the library's own authors about their own production cache — and it is a better statement of §2.4 than most internal design documents manage. The README continues with the mechanism, verbatim:

> "• Take your list of servers (eg: 1.2.3.4:11211, 5.6.7.8:11211, 9.8.7.6:11211)
> • Hash each server string to several (100-200) unsigned ints
> • Conceptually, these numbers are placed on a circle called the continuum. (imagine a clock face that goes from 0 to 2^32)
> • Each number links to the server it was hashed from, so servers appear at several points on the continuum, by each of the numbers they hashed to.
> • To map a key->server, hash your key to a single unsigned int, and find the next biggest number on the continuum. The server linked to that number is the correct server for that key.
> • If you hash your key to a value near 2^32 and there are no points on the continuum greater than your hash, return the first server in the continuum." ✅

Three details in that quote are worth extracting because they are the operational reality of every client-side ring:

- **`100-200` points per server** is the vnode count in the scheme that named the industry's default. It is a range, not a number, and it is what `R` is in this guide.
- **The token identity is the server string** (`1.2.3.4:11211`), hashed to *several* unsigned ints — the §3.1 rule that the identity must be stable, unique and identical on every client, in the wild.
- **Client-side state has a lifecycle.** `libketama`'s README describes the continuum as "created and stored in shared memory for future access. If the file modification time changes, the continuum is regenerated and shared memory is updated." ✅ A file mtime is the membership-update trigger. §10 is about what happens when that file is not the same everywhere.

Envoy's documentation refers to ring-hash as "commonly known as 'Ketama' hashing" ✅, so the name has become generic for "token ring with multiple points per host".

### 9.2 Dynamo and Cassandra: the lineage that popularised it

**Dynamo.** Consistent hashing is not Dynamo's invention and Dynamo's own text attributes it: the paper's reference [10] is "Karger, D., Lehman, E., Leighton, T., Panigrahy, R., Levine, M., and Lewin, D. 1997. Consistent hashing and random trees … STOC '97 … 654-663" ✅. What Dynamo did was *popularise* it in industry: the paper's own summary of the design states, verbatim, "Data is partitioned and replicated using consistent hashing [10]" ✅, with virtual nodes, a preference list built by skipping positions for distinct physical nodes, N replicas, and a gossip-based membership protocol ✅ (all quoted in §4.3 and §5.1 and verified from the SOSP'07 PDF). **Attribute the technique to Karger et al. 1997 — verified at the paper's own title page — and attribute the industrial pattern (vnodes + preference list + gossip membership) to Dynamo.**

**Cassandra.** Cassandra's documentation is explicit that it is following Dynamo: "Apache Cassandra relies on a number of techniques from Amazon's Dynamo distributed storage key-value system", listing "Dataset partitioning using consistent hashing" ✅, and it spells out its ring in its own terms — tokens, endpoints, hosts, vnodes, the token map, and the `(t1, t2]` ownership range ✅ (quoted in §3.1, §4.5, §5.1). Verified specifics:

| Item | What Cassandra's own documentation/source says | Where read |
| --- | --- | --- |
| Partitioning | "Cassandra partitions data over storage nodes using a special form of hashing called consistent hashing" ✅; ownership by "walking" the ring clockwise, "similar to the Chord algorithm" ✅ | `cassandra.apache.org` architecture docs, 5.0 docs page, read 2026-09-28 |
| Replica selection | "walk the ring in a clockwise fashion until we encounter three distinct nodes" (worked RF=3, 8-node example) ✅ | same |
| Replica distinctness / failure domains | "Replicas are always chosen such that they are distinct physical nodes which is achieved by skipping virtual nodes if needed. Replication strategies may also choose to skip nodes present in the same failure domain such as racks or datacenters" ✅ | same |
| vnodes | `num_tokens: 16` as shipped in the 4.1 configuration file ✅; 2.x's random-token default of **256** per node as stated in the architecture docs ✅; the deterministic allocator added in 3.x+ "requiring a much lower number of tokens per physical node" ✅ | `conf/cassandra.yaml` (4.1 branch) and the architecture docs |
| vnode downside | "Every token introduces up to `2 * (RF - 1)` additional neighbors … The more tokens you have, the higher the probability of an outage" ✅ | architecture docs |

**What is deliberately not claimed:** the exact default partitioner class and its hash function are not asserted here from a page this pass did not read end to end. The configuration file's own comment ties the deterministic allocator to the `Murmur3Partitioner` ✅, and that is the limit of what this guide states.

### 9.3 Redis Cluster is a hash-slot system, not a ring

This is the most useful contrast in the guide, because Redis Cluster is frequently described as "consistent hashing" and its own specification describes something structurally different. Verified verbatim from the Redis cluster specification (`redis.io`, read 2026-09-28):

- **Slot count and the hash function:** "The cluster's key space is split into **16384** slots, effectively setting an upper limit for the cluster size of 16384 master nodes (however, the suggested max size of nodes is on the order of ~ 1000 nodes)" ✅ and "`HASH_SLOT = CRC16(key) mod 16384`" ✅.
- **The exact CRC:** the specification gives the full parameter set ✅ — "Name: XMODEM (also known as ZMODEM or CRC-16/ACORN) · Width: 16 bit · Poly: 1021 … · Reflect Output CRC: False · Xor constant to output CRC: 0000 · Output for `"123456789"`: 31C3", and notes "14 out of 16 CRC16 output bits are used (this is why there is a modulo 16384 operation in the formula above)" ✅.
- **Hash tags:** "Redis Cluster implements a concept called **hash tags** that can be used to force certain keys to be stored in the same hash slot" ✅, computed by hashing only the substring between the first `{` and the first following `}` when that substring is non-empty ✅.
- **Where the map lives:** "In Redis Cluster, nodes are responsible for holding the data, and taking the state of the cluster, including mapping keys to the right nodes" ✅; nodes do not proxy ("nodes don't proxy commands to the right node in charge for a given key, but instead they redirect clients to the right nodes" via `-MOVED`/`-ASK`) ✅, and "clients that are able to cache the map between keys and nodes can improve the performance in a sensible way" ✅.

**Why it is not a ring, stated precisely:**

1. **The keyspace is partitioned before nodes exist.** `CRC16(key) mod 16384` divides the keyspace into a fixed 16,384 slots; nodes are then assigned *sets of slots*. There are no tokens, no arcs, and no clockwise walk — a lookup is a modulo and a map lookup, not a search.
2. **Rebalancing is an explicit, operator-chosen migration of whole slots**, not an automatic consequence of a membership change. The specification distinguishes stability from reconfiguration ("the cluster is **stable** when there is no cluster reconfiguration in progress (i.e. where hash slots are being moved from one node to another)") ✅, which is a statement no ring ever makes: on a ring, ownership follows from the token set immediately, and there is no "in progress" state in the map itself.
3. **The minimum unit of movement is a slot**, so the granularity of a rebalance is chosen by whoever moves the slots — a different operational object from "1/(N+1) of the keyspace moves automatically".

The practical consequence: a Redis operator *controls* movement (and therefore must plan it), while a ring operator *predicts* movement (and must plan the consequences). Both are legitimate; conflating them produces designs that assume automatic rebalancing that will never happen.

### 9.4 Kafka: partition assignment by modulo, and not a ring

Kafka is often cited as an example of consistent hashing. Its own source says otherwise. Verified from the Apache Kafka source tree (3.7 branch, read 2026-09-28), the built-in partitioner's key function is:

```java
public static int partitionForKey(final byte[] serializedKey, final int numPartitions) {
    return Utils.toPositive(Utils.murmur2(serializedKey)) % numPartitions;
}
```

That is **modulo hashing**, with a murmur2 hash and a positive-modulo fix — the very rule §2 derives the failure of, applied to partitions rather than nodes. Kafka's documentation states the client-visible guarantee in its own words: "Events with the same event key (e.g., a customer or vehicle ID) are written to the same partition" ✅, which is all the ordering guarantee requires. What makes Kafka workable is that **the partition count does not change**: partitions are a fixed property of a topic, and adding brokers is handled by moving whole partitions between brokers at the controller's discretion — not by rehashing keys. Increasing a topic's partition count *does* change `numPartitions` and *does* remap keys, which is the same object lesson as Redis: an explicit, planned reconfiguration instead of an arithmetic side effect. This guide makes no claim about the specific algorithm Kafka's controller uses to place partitions on brokers, beyond that it is a placement decision over a fixed partition set.

### 9.5 Envoy: ring hash and Maglev, side by side

Envoy is the rare production system whose documentation defines both schemes, which makes it the best single source for the trade in §8.3. Verified verbatim (`envoyproxy.io`, read 2026-09-28):

- **Ring hash:** "The ring/modulo hash load balancer implements consistent hashing to upstream hosts. Each host is mapped onto a circle (the 'ring') by hashing its address; each request is then routed to a host by hashing some property of the request, and finding the nearest corresponding host clockwise around the ring. This technique is also commonly known as 'Ketama' hashing, and like all hash-based load balancers, it is only effective when protocol routing is used that specifies a value to hash on." ✅
- **The token identity is configurable:** "If you want something other than the host's address to be used as the hash key (e.g. the semantic name of your host in a Kubernetes StatefulSet), then you can specify it in the `"envoy.lb"` `LbEndpoint.Metadata`" ✅ — the §3.1 stability rule again, this time as a configuration surface.
- **Maglev:** the table is filled "until the table is completely filled", with entries proportional to weight (the 21,846/43,691/65,537 example) ✅, and the comparison to ring hash is the `10x / 5x / ~2× keys move` statement quoted in §8.3 ✅.

Envoy also documents the operational guardrail for Maglev's weighting skew, verbatim: "Best practice is to monitor the `min_entries_per_host` and `max_entries_per_host` gauges to ensure no hosts are underrepresented or missing." ✅ That is §11.3's per-node share check, implemented as product telemetry.

**What Maglev's own paper states about the table** (verified from the NSDI '16 PDF, read 2026-09-28): `M` is the table size; "`M` must be a prime number so that all values of `skip` are relatively prime to it"; each backend's placement is driven by a permutation generated from `offset = h1(name[i]) mod M` and `skip = h2(name[i]) mod (M−1) + 1`, with `permutation[i][j] = (offset + j × skip) mod M` ✅; the worst-case build is `O(M²)` and the average is `O(M log M)` ✅; and "in practice, we choose `M` to be larger than `100×N` to ensure at most a 1% difference in hash space assigned to backends", because each backend takes `⌊M/N⌋` or `⌈M/N⌉` entries, so "the number of entries occupied by different backends will differ by at most 1" ✅. On weighting, the paper is explicit about its own limits, verbatim: heterogeneous backend weights "can be achieved by altering the relative frequency of the backends' turns; the implementation details are not described in this paper" ✅ — so the weight-proportional table filling quoted above is **Envoy's documented implementation of the same scheme**, and Maglev's paper does not describe its own. The paper's own statement of the default `M` value was not located in the text this pass, which is why §8.3 cites Envoy's `65,537` example rather than a Maglev default (§16).

### 9.6 Riak: not verified this pass

Riak KV is widely described as using consistent hashing with vnodes. **This guide does not state that as fact**, because the attempt to verify it failed: `docs.riak.com` served a JavaScript product-selector page for both the direct URL and the `index.html` variant, and the Wayback Machine reported that the specific page under `docs.riak.com/riak/kv/2.2.3/learn/concepts/consistent-hashing/` had not been archived. That is a tooling and site-architecture limitation, **not evidence about Riak** — recorded in §16 as unverified rather than as refuted.

### 9.7 Couchbase: vBuckets, which are pre-computed slots

Verified from Couchbase's own documentation (`docs.couchbase.com`, current, read 2026-09-28):

- **What they are:** "vBuckets are virtual buckets that break bucket data into smaller pieces to make distributing and replicating data across multiple nodes easier", and they are **a fixed number per bucket set at creation**: "Once created, the number of vBuckets in a bucket does not change." ✅
- **The counts:** macOS 64; Couchstore on Linux and Windows 1024; Magma "can use either 128 or 1024 vBuckets on Linux and Windows" ✅ — with the number chosen at bucket creation.
- **The hash:** "Couchbase Server uses a CRC32 hashing algorithm to map items to vBuckets. It hashes the item's key, to determine which vBucket stores the item." ✅
- **The map and its distribution:** "The Cluster Manager tracks which nodes contain each vBucket … When the mapping changes, the Cluster Manager updates the vBucket map and notifies clients of the change." ✅ And for replicas: "The active vBucket and its replicas are always on different nodes in the cluster to protect against data loss from node failovers." ✅
- **Rebalance:** "During a rebalance, Couchbase Server redistributes active and replica vBuckets across the available nodes." ✅

**The structural verdict:** vBuckets are *slots*, in the same family as Redis's slots, not tokens on a ring — a fixed pre-computed partition set with an explicit map from partition to node, a client-visible map version, and an operator-initiated rebalance. The vocabulary ("virtual buckets") invites confusion with "virtual nodes"; the mechanics are the opposite kind of object. A vnode is a point on a continuum whose ownership follows from a hash; a vBucket is a numbered slice of a bucket whose owner is recorded in a map.

### 9.8 The summary table, with verification status

| System | Scheme its own documentation/source states | Ring? | Verified at | Status |
| --- | --- | --- | --- | --- |
| memcached clients (`libketama`) | token ring, `100-200` points per server, `0…2^32` continuum | yes | `github.com/RJ/ketama` README | ✅ |
| Dynamo | consistent hashing with virtual nodes + preference list | yes | SOSP'07 PDF (`allthingsdistributed.com`) | ✅ |
| Cassandra | token ring, vnodes, distinct-node/failure-domain replica selection | yes | `cassandra.apache.org` docs + `conf/cassandra.yaml` (4.1) | ✅ |
| Redis Cluster | 16,384 hash slots, `CRC16(key) mod 16384`, hash tags | **no** | `redis.io` cluster spec | ✅ |
| Kafka (producer key routing) | `murmur2(key) % numPartitions` | **no** | Kafka source, 3.7 branch | ✅ |
| Envoy | ring hash ("ketama") **and** Maglev, both documented | ring, and a table | `envoyproxy.io` docs | ✅ |
| Maglev (Google) | fixed lookup table of size `M` (prime), permutation-based fill | no ring; a table | NSDI '16 paper PDF + USENIX record | ✅ (weighting implementation: paper says it is not described ✅) |
| Couchbase | 1024 (Couchstore) / 128 or 1024 (Magma) vBuckets, CRC32, cluster-managed map | **no** | `docs.couchbase.com` | ✅ |
| Riak KV | not verified | unknown | documentation unreachable | ❌ unverifiable this pass |

## 10. Client-Side Versus Server-Side Rings

### 10.1 Where the ring lives, and the three real arrangements

| Arrangement | Who computes the owner | Examples, as their own docs state | The failure it is exposed to |
| --- | --- | --- | --- |
| **Client-side ring** | The caller, holding its own token array | `libketama`: the continuum is built from a server file, stored in shared memory, "regenerated and shared memory is updated" when the file mtime changes ✅ | The client's server file is stale or different from its peers' |
| **Server-side map, client-cached** | The cluster; clients cache a versioned map | Redis Cluster: nodes hold the slot map, clients "that are able to cache the map between keys and nodes can improve the performance in a sensible way" ✅; Couchbase: "the Cluster Manager updates the vBucket map and notifies clients of the change" ✅ | The cached map is stale; the cluster must detect and push |
| **Server-side, transparent** | Any node can answer for any key | Dynamo: "every node in the system can determine which nodes should be in this list for any particular key" ✅; Cassandra: a coordinator routes to the replica set and the client need not know the ring at all | The cluster's own membership view lags reality (gossip delay), so a *server* can also route on a stale ring |

The important structural fact is that the third row does not eliminate the stale-view problem; it *moves* it from the client to the cluster. Karger et al.'s original motivation was exactly this: on the Internet, "information about what machines are functional propagates slowly through the network, so that clients may have incompatible 'views' of which machines are available to replicate data" ✅ — read from their paper's own text (Brown University copy of the STOC'97 paper, title page and introduction verified; see §15). Two of the four properties the original paper proved for its hash family — **spread** and **load** — are described by Google's jump-hash paper as the ones that "measure the behavior of the hash function under the assumption that each client may see a different arbitrary subset of the buckets" ✅, and it observes that where "all clients see the same set of buckets", those two properties are unnecessary ✅. That is the cleanest available statement of the trade in this section: **the ring's tolerance of partial views is a feature you pay for, and systems that guarantee a single consistent view do not need it.**

### 10.2 What a stale ring actually does

The failure is not "some requests go to the wrong place". It depends on what the layer is:

1. **On a cache:** wrong-node requests are misses. The tier's hit ratio drops by the size of the disagreement; the origin sees extra load. Annoying, self-healing, and mostly invisible until §7.2's amplifiers are in play.
2. **On a store with an ownership assumption:** two clients that disagree can route the same key to two different nodes. Neither node sees a conflict, because each believes it is authoritative. The result is not a latency problem but a *correctness* problem — divergent replicas, lost updates, and (for deletes) the resurrect-class of anomalies. This is why ring operators must be able to answer "which node owns key K at time T" from a versioned record, not from a running process (§12).
3. **During a migration:** the window in which two versions of the ring are simultaneously live is the window in which double-ownership is possible. The specification must say who wins during the window (the newer version, the higher epoch, or "reads tolerate, writes fence").

### 10.3 Membership propagation and the versioning that makes it survivable

The minimum apparatus for a client-side ring that stays correct:

- **A monotonic ring version (epoch)**, incremented on every membership change, carried on every request or on every client's cached copy. Without it, a stale client cannot even *detect* that it is stale.
- **A fail-closed or fail-safe rule for version mismatch**: the honest options are "reject and refresh" (safe, adds a latency spike at the moment of change) or "proceed and accept the miss" (fine for caches, wrong for stores). It has to be a *stated decision*, because the default in most implementations is the second.
- **A bound on propagation lag, monitored**: the number of distinct ring versions live in a fleet, and the age of the oldest, measured continuously. This is cheap to instrument if the version is in the request path, and impossible to reason about if it is not.
- **A change protocol that never requires two incompatible changes at once**: add-then-remove, one at a time, with the completion of movement as a barrier (§11.1).
- **The hash function and the token identity strings are part of the version.** Changing the hash function, the digest length, or the token identity format creates a *different ring*, and it must be deployed as a ring migration with a cutover, never as a library upgrade. Nothing in the protocol detects the mismatch; the symptom is a fraction of keys silently living on the wrong node.

**The claim to take away from this section:** in practice, the incidents attributed to "consistent hashing" are overwhelmingly incidents of **ring-version disagreement** — a client on an old ring, a server on a new one, a hash function changed in one library version — rather than incidents of an unbalanced ring. The arithmetic in §2–§4 explains why a ring redesign is rare; the mechanics in this section explain why a *rolled-out* ring redesign is common.

## 11. Operations

### 11.1 The procedure for adding or removing a node

The order of operations is the whole of the operational advice; done out of order, each step turns a bounded movement into an incident. The pattern below assumes a system that exposes membership changes as explicit operations (the ring itself never requires them in an order — it requires them one at a time and completely):

1. **Compute the new ring offline and diff it.** From the current token list and the proposed one, compute every arc and therefore the exact set of keys whose owner changes. This is arithmetic on a sorted array — no traffic needed — and it produces the moved fraction *before* anything is changed. If the number is not what you expect from `1/(N+1)` or `1/N` (§3.3), stop: someone is changing token identities, and the movement is not bounded (§6.3).
2. **Check headroom at both ends.** The receiving nodes must hold their own data plus the incoming arc; the origin (if this is a cache tier) must absorb the moved fraction as misses (§7.2). Headroom is a precondition, not a hope.
3. **Prewarm or pre-stream.** For storage, stream the arcs before announcing the node. For caches, warm at a controlled rate ahead of the cutover. The target set is computable from step 1, which is what makes this possible at all.
4. **Announce one membership change, with a new ring version.** One change at a time. Never add and remove simultaneously.
5. **Ramp traffic if the mechanism allows it** (weight ramp / token ramp / slow start, §7.2), watching origin latency and per-node load, with an automatic halt threshold.
6. **Wait for movement completion before the next change.** Completion is a measurable state, and "mostly complete" is not a state to start from.
7. **Verify against the invariants after the change**: per-node share within the expected band (§4.2), replica sets still satisfying the failure-domain rule (§5.3), hit ratio returned to baseline, origin load back to baseline. Only then close the change record.
8. **Decommission the old node last, and only when it owns nothing.** With vnodes the decommission is a multi-partner operation (§4.5) and is not complete until every arc has been handed off.

### 11.2 How much data actually moves, in one table

All figures derived in §3.3, §4.4 and §6.3; the "time" column is deliberately absent because wall-clock movement time depends on bytes, bandwidth and throttling, none of which are properties of the technique:

| Event | Keyspace whose owner changes | Where it lands |
| --- | --- | --- |
| Add one node to `N` (ring) | `1/(N+1)` | the new node's arcs |
| Add one node to `N` (modulo) | `N/(N+1)` | everywhere |
| Remove one node from `N` (ring) | `1/N` | the successors of its arcs |
| Remove one node from `N` (modulo) | `(N−1)/N` | everywhere |
| Add one token to a ring of `T` tokens | `1/T` | the node that owns the new token |
| Add 256 tokens to a ring of `T = 25,600` | `256/25,600 = 1%` | one node's fair share, as expected |
| Renumber or re-identify tokens | unbounded — a rehash | not a rebalance (§6.3) |

### 11.3 The monitoring that reveals an unbalanced ring

Six signals, in the order they should be built. The first two are computed from the token list with no traffic at all, which means they can be *asserted in CI* rather than paged on.

| Signal | What it reveals | How to set the threshold |
| --- | --- | --- |
| **Static arc histogram** (per-node share of the keyspace, computed from the token list) | An unbalanced ring, before any request is served | Derive the expected band from `1/√R` (§4.2): at `R=256` the relative spread is `1/√256 = 6.25%`, so a node more than ~3σ (`≈19%`) above its fair share is anomalous *for that `R`*; a node at 200% means `R≈1`. The threshold must be stated with the `R` it assumes. |
| **Declared vs observed share** | Weight drift (§6.2): a node whose token count no longer matches its capacity, or an emergency replacement running a default `num_tokens` | Compare the *declared* share (`R_i/T`) against measured request or byte share per node; a persistent gap that survives restarts is drift, not noise |
| **Hit ratio per node and fleet-wide** | Cold nodes (§7.2) — a newly added node must climb to the fleet's hit ratio as it warms; a node that never climbs is receiving keys it cannot serve | Baseline versus current, with the change time annotated; the gap is bounded by the moved fraction |
| **Origin load versus baseline** | The direct business consequence of a cache ownership change, and the metric that decides whether a ramp continues | Provision headroom for `1/(N+1)` (ring) or `N/(N+1)` (modulo) of the tier's request rate; alarm on the step, not on the average |
| **Top-N key request histogram** | Hot keys, which no ring metric can see (§7.1) | If the top key is more than ~a few times `1/K_total` of traffic, the answer is §7.1's table and not a ring change |
| **Ring-version census** | Stale clients (§10), the most common real cause of ring incidents | Count distinct live ring versions and the age of the oldest; a version older than the propagation budget is an incident, not a warning |

### 11.4 The honest note

**The technique's operational cost is exactly the data movement it does not eliminate.** The ring makes the *decision* cheap — a sorted array, a binary search, an arithmetic guarantee about which keys change owner — and it leaves the *work* entirely intact. `1/(N+1)` of the keyspace still has to be streamed, warmed, rebuilt or re-fetched; 0.99% of 100 TB is still 0.99 TB; the receivers still need headroom, the movement still needs a queue and a completion barrier, and the origin still needs to absorb the misses that the movement creates.

What the technique changes is the *shape* of the operational burden. Without it, a node count change is an unplannable event: `N/(N+1)` of everything moves and no amount of preparation makes it bounded. With it, the event is bounded, plannable, precomputable and — crucially — **small enough to be dangerous**, because a change that moves 1% of the keyspace looks harmless right up to the moment the origin saturates on the 1% (§7.2). The ring converts an outage into a procedure, and a procedure into a discipline; it does not convert the work into nothing.

## 12. The Regulated-Institution Angle

This section does not re-derive any infrastructure guide. It names where the technique actually shows up in a bank estate and then makes one argument about how the ring's artefacts should be governed. **No real bank is asserted to use any scheme described in this guide.** Cross-references below are to files that exist in this repository; the domain context lives in them, the partitioning technique lives here.

### 12.1 Where the technique actually appears in a bank estate

| Layer | How the ring appears | What breaks, and which section explains it | Domain context lives in |
| --- | --- | --- | --- |
| **Cache tier in front of the system of record** | Client-side ketama-style ring, or a ring-hash load balancer distributing requests across cache nodes | §2.4: a node count change turns `N/(N+1)` of the tier into misses and attacks the origin; §7.2: the cold start | `tunable_consistency_databases_guide.md` for what the store behind the cache can promise |
| **Session, affinity and connection layers** | Ring-hash (ketama) or Maglev policies in the load balancer, hashing on a session or connection identifier to keep long-lived sessions on one endpoint | §10: two balancers on different ring versions split a session's traffic; §5.2 if the "affinity" was assumed to also give resilience | `../banking/fix_protocol_guide.md` for the session-oriented protocol context |
| **Sharded operational stores** | Token ring with vnodes and a preference list (the Dynamo/Cassandra lineage), or slot-based engines | §5: replica sets that are distinct nodes but not independent failure domains; §7.1: hot tenant keys | `oracle_sharding_guide.md` for the RDBMS product's own sharding mechanisms, `architecture/billion_user_system_arch.md` for where partitioning sits in a large design |
| **Analytical and warehouse layers** | Hash partitioning (usually a key hash mod `N`, sometimes a ring), then a shuffle when the query does not respect the partition key | §7.4: cross-partition work; the analytical engines solve it by moving data between stages | `clickhouse_guide.md` §4.2, `spark_tuning_guide.md` §11 |
| **Messaging and transfer layers** | Partitioned messaging with key-based routing (modulo on the partition count in the case verified in §9.4), and partitioned consumers | §9.4: the partition count is the sensitive parameter; changing it rehashes keys | `../banking/swift_alliance_access_guide.md` and `../banking/market_data_consumption_guide.md` for the messaging and market-data domains respectively |
| **Payment and fraud pipelines** | Keyed fan-out over sharded workers, with the partition key chosen for co-location of an entity's events | §7.1: the entity that dominates traffic becomes the hot partition | `../banking/financial_fraud_detection_at_scale_guide.md` |

The pattern across those six rows is that **the ring is almost never the interesting layer** — it is the component that makes the interesting layer's failure modes bounded and plannable rather than unbounded and surprising. That is also why ring questions get escalated: they surface as availability and capacity incidents, not as engineering detail.

### 12.2 The partition map is a versioned operational asset, not an implementation detail

The single governance conclusion of this guide: **the artefact that decides which node holds which key is a control, in the same class as the schema and the key inventory.** It has the properties that make that true:

- **It is configuration with a version.** The ring version, the token list, the hash function and the token identity format together define the mapping. Any of them changing changes the placement of data (or of requests), and none of them is visible in application code.
- **It has a diff, and the diff is computable before the change.** §11.1 step 1 turns "we are adding a node" into "these key ranges change owner, that is `1/(N+1)` of the keyspace, and here is the list". A change record that cannot state the movement is a change record that has not been analysed.
- **It has to be reconstructible for a past timestamp.** "Which node held customer *X*'s record on 14 March" is a question asked during incident review, dispute handling and audit. If the answer depends on a running process's current view, it is unanswerable; if it depends on a versioned token file with a change history, it is a lookup. Keep the history.
- **A restore plan that omits the ring version is not a restore plan.** Restoring a node's data to a node that no longer owns those arcs either does nothing useful or creates a second owner. Recovery runbooks must carry the ring version alongside the backup generation, and the two must be checked together.
- **The failure-domain invariant must be evidenced, not asserted.** §5.2 explained why `RF=3` does not mean three racks. In an estate with documented resilience expectations — see `../banking/mas_regulations_guidelines_guide.md` for the regulatory-context material — the evidence has to come from the topology inventory: enumerate the replica set for the arcs that matter, resolve each node to its rack, zone and host, and show that the failure domains are distinct. Then re-run it after every topology or membership change, because §5.2's last point is that replica sets shift when tokens do.
- **Decommissioning requires proof of emptiness.** A node with `R` tokens is not decommissioned until every one of its arcs has been handed off (§4.5), and the proof is per-arc, not "the cluster looks balanced".
- **Weights are a policy with an owner.** §6.2's drift table is a governance failure as much as an engineering one: a token count is a declared capacity claim, and something has to own the claim, review it when hardware changes, and reconcile it against observed share (§11.3).

Put differently: the ring itself is a disposable computation — you can rebuild it from a token list in milliseconds — and the *decisions recorded in it* are not. The reason a sharded estate is governable is that those decisions are written down, versioned, diffable and reconstructible; the reason sharded estates have incidents is that they usually are not.

## 13. A Cymbal Bank Worked Example

> **Every number in this section is illustrative and fictional.** Cymbal Bank is the only bank persona in this repository; the figures below are a worked *shape* for a decision, not a benchmark, not a measurement, and not a claim about any real institution.

### 13.1 The starting state

Cymbal Bank's digital channel runs a session/cache tier in front of its session store. Illustratively:

| Parameter | Value | Note |
| --- | --- | --- |
| Cache nodes | 12 | `N = 12`, and the client libraries place keys with `hash(key) mod 12` |
| Cached keys | 240 million | session and device-registration records |
| Average value size | 2 KB | ≈ 480 GB of cached state in total |
| Peak request rate | 120,000 requests/second | across the tier |
| Hit ratio today | 99% | so the origin sees ≈ 1,200 rps at peak |
| Origin spare capacity | 15,000 requests/second | the headroom available for a migration, illustratively |

The trigger is §2.4: the last capacity change (12 nodes from 10) took the tier's hit ratio to `1/(N+1) = 1/11 ≈ 9%` for the duration of the re-warm, and the origin survived it by luck and by a quiet Sunday. The team decides to move to a ring *before* the next node is added, not after.

### 13.2 The migration arithmetic (this is the part that surprises people)

**A modulo-to-ring migration is not a bounded movement.** The two placements have nothing to do with each other: the old owner is `hash(key) mod 12`, the new owner is the first token clockwise. Under the model that the ring's placement is uniform and independent of the modulo placement, the probability that a key's new owner happens to be its old owner is `1/N`, so:

```
N = 12
keys that keep their owner by accident  = 1/12  = 8.33%
keys whose owner changes                = 11/12 = 91.67%
```

The `8.33%` is a coincidence, not structure — and note that `91.67%` is exactly the same fraction as §2.3's node-removal case `(N−1)/N`. **The first migration to a ring costs as much as the worst change modulo hashing can inflict.** The benefit arrives with the *next* change:

| Event | Modulo (before) | Ring (after) | Ratio |
| --- | --- | --- | --- |
| Add a 13th node | 12/13 = 92.31% of keys move | 1/13 = 7.69% | 12× |
| Lose one of 13 nodes | 12/13 = 92.31% | 1/13 = 7.69% | 12× |
| Lose one of 12 nodes | 11/12 = 91.67% | 1/12 = 8.33% | 11× |

**Turning 91.67% into a plan.** A staged cutover: route a fraction `p` of keys to the new ring, so the origin absorbs `p × 91.67% × 120,000` rps:

```
p = 10%  →  0.10 × 0.9167 × 120,000 ≈ 11,000 rps   (within the 15,000 rps of spare capacity)
p = 20%  →  0.20 × 0.9167 × 120,000 ≈ 22,000 rps   (beyond it)
largest safe stage ≈ 15,000 / (0.9167 × 120,000) ≈ 13.6% of the keyspace
```

So the migration runs as roughly eight stages of ~12% each, each stage verified against the origin's latency before the next begins — or, if the business can accept a 91.67% cache-cold window overnight, as one planned drain-and-refill in the quiet window. Both are defensible; what is not defensible is doing it during peak load and discovering the arithmetic in the incident channel.

### 13.3 The vnode choice, and its metadata cost

`N = 12` is a small fleet, so the `1/√R` relation from §4.2 matters more here than it would at `N = 100`:

| `R` | Tokens (`N·R`) | Relative spread `1/√R` | 3σ deviation of a node's share | Worst node vs fair share (≈) |
| --- | --- | --- | --- | --- |
| 1 | 12 | 100% | 300% | ~400% — the §3.4 problem, in a 12-node fleet |
| 64 | 768 | 12.5% | 37.5% | ~137% |
| 128 | 1,536 | 8.84% | 26.5% | ~127% |
| 256 | 3,072 | 6.25% | 18.75% | ~119% |

Derivation of one row, so the table is not a set of assertions: at `R = 128`, the relative spread is `1/√128 ≈ 0.0884 = 8.84%`; a node's fair share is `1/12 = 8.33%` of the keyspace; so the standard deviation of a node's share is `8.33% × 8.84% ≈ 0.74` percentage points of the keyspace, and a 3σ node sits at `8.33% + 3 × 0.74% ≈ 10.5%` — about 127% of its fair share, which at 120,000 rps peak is a node at ~12,600 rps instead of 10,000.

Cymbal chooses `R = 128` (illustratively), because the marginal balance gain from 256 does not justify the extra ring neighbours and the doubling of every membership message. The metadata cost is trivial at this scale and worth stating precisely: `12 × 128 = 1,536` tokens, at roughly 16 bytes per `(token, node)` entry, is ≈ 24 KB per client — the point being that **vnode metadata is a fleet-scale cost, not a small-fleet cost**, and the reason Dynamo's own operational note was about membership information shrinking by three orders of magnitude (§4.5) is that Dynamo's fleets were three orders of magnitude larger than Cymbal's.

### 13.4 The hotspot the ring does not solve, and what Cymbal does instead

During the design review, the telemetry shows that one key family — the app-configuration record that every client fetches on start-up and re-checks periodically — accounts for **≈18% of the tier's request rate** (illustratively). Its owner node is at ~1,300% of its fair share from that single key.

Applying §7.1: the ring cannot help. `R = 128` versus `R = 1,000,000` place that key on one node either way; re-weighting moves the *arcs*, not the key's popularity. What Cymbal does instead:

1. **Client-side caching with a short TTL** — the application holds the configuration record locally for 5 seconds, which removes most of the 18% from the tier entirely. This is the highest-leverage fix and it is a client change, not a ring change.
2. **Request coalescing (single-flight)** for the remaining fetches, so `m` concurrent misses on the same key produce one origin read.
3. **Key splitting** — the configuration is partitioned into `config#0 … config#7`, distributed across eight owners, with the client merging the eight parts. This costs a fan-out of 8 on a cold read and buys an 8× reduction in the hot key's per-node load.
4. **A documented decision not to touch the ring.** The review records explicitly that ring imbalance was ruled out as the cause (the arc histogram is normal for `R = 128`) and that no ring change is part of the remedy.

### 13.5 The rollback position

Stated before the change begins, in the change record:

- **The previous placement is a file, versioned and deployable.** Rolling back is a configuration flip back to the previous token/`mod 12` placement, not a code revert.
- **The old nodes keep running and keep their caches warm** for 24 hours after the cutover (illustratively), so the rollback has a warm target rather than a cold one. Rollback is only cheap while the old placement's data still exists; after the old nodes are decommissioned, the rollback position is the *origin's* capacity, not a data position.
- **The ramp has an automatic halt threshold**, e.g. origin p99 latency above a set multiple of baseline for five consecutive minutes (illustratively), which stops routing more keys to the new ring and returns the current stage to the previous placement. Because the ramp is expressed as token weights (§6.3), halting it is a config change and not a data movement.
- **The rollback position degrades over time, and that is stated.** Each completed stage warms more of the new ring and ages the old one; the plan records the point beyond which rollback is no longer a config flip but a re-warm — and therefore the point of no return.

The honest framing of the whole exercise: Cymbal spends a 91.67%-movement migration once, in order to make every subsequent capacity change an ~8% movement. That is the entire investment case, and it only pays if the fleet's node count actually changes again — which, for a tier that scales with customer growth, it will.

## 14. Anti-Patterns

Each row is a real failure with a symptom you can observe, a cause you can name, and a guardrail that prevents it. The meta-pattern is the same in every case: **the ring was asked a question it does not answer.**

| # | Anti-pattern | Symptom | Cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **Assuming a balanced ring fixes a hot key** | One node hot while the static arc histogram is normal for its `R`; re-weighting or adding vnodes changes nothing | §7.1: the ring distributes nodes, not popularity; a key lands on one node at any `R` | Track a top-N key request histogram as a first-class metric; when the top key's share exceeds a stated bound, use §7.1's fix table (split, coalesce, cache locally, shed), not a ring change |
| 2 | **Copying a default vnode count** | `num_tokens` (or "100–200 points") inherited from another system's documentation, with no derivation and no recorded reasoning | The right `R` depends on whether tokens are chosen randomly or allocated (§4.5), on the target spread `1/√R` (§4.2), and on the availability cost of extra ring neighbours (§4.5) — none of which travels with the number | Choose `R` from a stated target relative spread and a neighbour-count budget; record the derivation next to the config; re-derive it when the allocator or fleet size changes |
| 3 | **Two clients (or a client and a server) holding different rings** | A fraction of keys landing on nodes the design says they should not; hit ratio below design; "impossible" routing traces | §10: stale membership file or slot map, a hash function changed in one library version, a token-identity format change, a ring version that never propagated | Put the ring version in the request path; monitor the census of live versions and the age of the oldest; deploy hash-function and token-identity changes as migrations with a cutover, never as library upgrades |
| 4 | **Rebalancing without capacity headroom** | The destination node saturates mid-movement; the origin saturates on the moved fraction; a rebalance "causes" an outage | Movement is *additive* load: the receiver holds its own data plus the incoming arc, and a cache tier sends the moved fraction to the origin as misses (§7.2, §7.3) | Pre-flight the headroom check from the computable diff (§11.1 step 1); size origin headroom at `1/(N+1)` of the request rate; ramp with an automatic halt threshold |
| 5 | **Treating a hash-slot system as a ring** | An expectation of automatic minimal movement, followed by the discovery that nothing moved until someone moved slots; a "rebalance" that is a manual slot-migration project | §9.3/§9.7: slot maps (16,384 slots; 1,024 vBuckets) and token rings are different objects, and the vocabulary collides ("vBuckets" vs "vnodes") | Name the object in the design document — *token ring* or *slot map* — and plan movement explicitly for slot systems, where the operator, not the membership change, chooses what moves |
| 6 | **Benchmarking a ring without measuring imbalance** | A load test that reports healthy throughput and per-node averages, in a production system whose p99 is driven by one skewed node | The benchmark used a synthetic uniform key distribution, ran at one node count, and never printed the arc histogram, the per-node share or the top-K key concentration | Make the benchmark print max/mean per-node share and top-K key concentration, compare them against the `R`-dependent band from `1/√R`, and re-run at two node counts (for example `N` and `N+1`) to exercise the movement path |

The same discipline applies to all six: **state the question the ring is answering.** "Which node owns this key, and what fraction of the keyspace changes owner when the membership changes?" is a question the ring answers exactly, and §3 gives the answer. "Is my traffic evenly spread?" and "will one machine fall over?" are different questions, answered by the request histogram, the origin's headroom, and the load-capping schemes in §8 — and no amount of ring tuning substitutes for them.

## 15. Claims Audit

Every factual claim in this guide that is attributed to a source, with the source's date, the authority of the source, and the result. ✅ = verified this pass against the primary source; ⚠ = flagged (secondary attribution, or a detail not fully confirmed); ❌ = rejected, or unverifiable and therefore not relied upon.

### 15.1 The papers

| Claim | Source, with date | Quality | Note |
| --- | --- | --- | --- |
| Consistent hashing was introduced by **Karger, Lehman, Leighton, Levine, Lewin and Panigrahy**, in *Proceedings of the 29th ACM Symposium on Theory of Computing (STOC)*, pp. 654–663, 1997 | The paper's own title page, read via the Brown University course copy (`cs.brown.edu/courses/cs296-2/papers/consistent.pdf`) | ✅ | The ACM Digital Library record (`doi 10.1145/258533.258660`) was **not** readable this pass — the extractor returned HTTP 500 and a JavaScript-gated page on every attempt. The author list was instead read from the paper's own title page and cross-checked against two independent bibliographies: Lamping & Veach 2014, reference [1]; Maglev, NSDI '16, reference [28]. Note that Dynamo's own bibliography lists the same six authors in a **different order** — a citation-order variation, not a disagreement about authorship. |
| The original definition: "a consistent hash function is one which changes minimally as the range of the function changes" | Karger et al. 1997, abstract, quoted verbatim | ✅ | This is the sentence that supports §1's thesis. The original definition is about the **assignment** changing minimally as the bucket set changes — i.e. about remapping volume — not about the hash function's output being stable. The name has been read the wrong way ever since. |
| Karger et al.'s experiments used **1000 points per bucket** to reach a standard deviation of **3.2%** in the number of keys assigned to different buckets | Reported verbatim by Lamping & Veach, arXiv:1406.2294 (9 June 2014), related-work section | ✅ as a secondary attribution | This is a genuine independent check of §4.2's derivation: `1/√1000 = 3.16%`, against the reported 3.2%. Two independent lines — a first-principles variance model and a 1997-era experimental measurement — agreeing to one significant figure is the strongest available evidence that the `1/√R` scaling is the right model. |
| Consistent hashing has four properties — **balance, monotonicity, spread, load** — of which spread and load concern clients seeing different subsets of the buckets | Lamping & Veach 2014, describing the Karger et al. paper | ✅ as a secondary attribution | Useful because it shows the original design *expected* inconsistent client views (§10) and priced the tolerance for them. |
| **Rendezvous hashing**: for each candidate bucket, compute `h(key, bucket)` and return the bucket with the highest value; the original version "requires time proportional to the number of buckets" | Lamping & Veach 2014, describing Thaler & Ravishankar | ✅ as a secondary attribution | The primary text was read only at title-page level (§15.2). |
| **Thaler & Ravishankar**, "Using Name-Based Mappings to Increase Hit Rates" | Title page rendering of the paper ("David G. Thaler, Student Member, IEEE, and Chinya V. Ravishankar, Member, IEEE"); journal version *IEEE/ACM Transactions on Networking*, vol. 6, no. 1, pp. 1–14 | ✅ for authors and title; ⚠ for the year and volume | The MSR technical-report PDF (`tr-98-14`, and the `2017/02/HRW98.pdf` path) failed to load, and IEEE Xplore returned nothing. Two bibliographies disagree on the year — Lamping & Veach cite "Volume 6, pages 1–14, **1997**", while Maglev (NSDI '16) cites "6(1):1–14, **1998**" — and a search-result rendering of the paper's own header shows "VOL. 6, NO. 1, **FEBRUARY 1998**". This guide therefore uses volume 6, number 1, pp. 1–14, February 1998, and records the discrepancy. |
| **Jump consistent hash**: Lamping & Veach, Google, arXiv:1406.2294, submitted 9 June 2014; "about 5 lines of code"; "requires no storage, is faster, and does a better job of evenly dividing the key space among the buckets"; "Its main limitation is that the buckets must be numbered sequentially" | arXiv abstract page and the full PDF, both read | ✅ | Additional verbatim facts from the PDF: it is converted from linear to **logarithmic time** by computing only jump destinations; the balance requirement is stated as `ch(k, n+1)` staying the same for `n/(n+1)` of keys and jumping to `n` for the other `1/(n+1)`; Karger et al.'s approach "needs thousands of bytes of storage per candidate shard", which becomes "megabytes of memory" per client at thousands of shards. |
| **Maglev**: Eisenbud, Yi, Contavalli, Smith, Kononov, Mann-Hielscher, Cilingiroglu, Cheyney (Google), Shang (UCLA), Hosein (SpaceX), *13th USENIX NSDI*, 2016, pp. 523–535; "Maglev is also equipped with consistent hashing and connection tracking features"; "has been serving Google's traffic since 2008" | USENIX conference record and the paper PDF | ✅ | Table mechanics verified from the PDF: `M` is the table size and must be prime; `offset = h1(name) mod M`, `skip = h2(name) mod (M−1) + 1`, `permutation[i][j] = (offset + j×skip) mod M`; worst-case build `O(M²)`, average `O(M log M)`; `M` chosen "larger than `100×N` to ensure at most a 1% difference in hash space"; entries per backend differ by at most 1; **"heterogeneous backend weights… the implementation details are not described in this paper"**. |
| Maglev's own default table size | — | ⚠ | Not located in the paper text this pass. The only figure this guide quotes is Envoy's documented example total of **65,537** entries, which is Envoy's implementation detail, not a Maglev default. |
| **Consistent Hashing with Bounded Loads**: Mirrokni, Thorup, Zadimoghaddam; arXiv:1608.01350, v1 3 August 2016, v3 27 July 2017; cap of `⌈cm/n⌉`; "with `n` clients and `n` servers, we get a guaranteed max-load of 2"; plain consistent hashing's expected overload is `Θ(log n / log log n)`; the cap costs a multiplicative move factor `O(1/ε²)` (`ε ≤ 1`) or `1+O(log c/c)` (`ε ≥ 1`) | arXiv abstract page | ✅ at abstract level | The algorithm's exact spill rule (which node is tried when a node is at its cap) is not stated in the abstract and was **not** verified; §8.4 therefore describes the mechanism only at the level the abstract supports. |

### 15.2 The implementations

| Claim | Source, with date | Quality | Note |
| --- | --- | --- | --- |
| `libketama` replaced `serverlist[hash(key) % serverlist.length]` because any add or removal "effectively wiped the entire cache"; it hashes each server string to "several (100-200) unsigned ints" onto a `0…2^32` continuum; a key maps to the "next biggest number"; a key near `2^32` wraps to the first server; weights are "realised by adding more or less points to the continuum"; the continuum lives in shared memory and is regenerated when the server file's mtime changes | `github.com/RJ/ketama` README, read 2026-09-28 | ✅ | The README carries **no date**, so no date is attributed to it; the commit history shows 33 commits. A separate section of the README marks the project as seeking a new maintainer. |
| Envoy **ring hash** "implements consistent hashing to upstream hosts", hashes each host's address onto a circle and finds "the nearest corresponding host clockwise", is "commonly known as 'Ketama' hashing", and is only effective when a protocol specifies a value to hash on; the hash key can be overridden via `envoy.lb` endpoint metadata | `envoyproxy.io` load-balancer documentation, version string `1.40.0-dev`, read 2026-09-28 | ✅ | Product documentation about its own implementation. |
| Envoy **Maglev** vs **ring hash**: "approximately 10x and 5x respectively" faster table build and host selection at a 256K ring; "not as stable as ring hash when upstream hosts change"; "simulations show approximately double the keys will move"; disruption is tunable via `table_size`; weights are realised as entries (21,846 / 43,691 / 65,537); monitor `min_entries_per_host` and `max_entries_per_host` | same | ✅ | These are **Envoy's own simulations**, not Maglev paper results. Attributed accordingly. |
| Redis Cluster: **16,384** slots, `HASH_SLOT = CRC16(key) mod 16384`, CRC16 named XMODEM/ZMODEM/CRC-16/ACORN with poly 1021, init `0000`, no input/output reflection, xor `0000`, check value `31C3` for `"123456789"`; hash tags defined by the first `{`…`}` substring with non-empty content; suggested maximum "on the order of ~ 1000 nodes"; `-MOVED`/`-ASK` redirection, no proxying; clients may cache the slot map | `redis.io` cluster specification, read 2026-09-28 | ✅ | The specification also notes it is "a work in progress as it is continuously synchronized with the actual implementation" and that what it describes "is implemented in Redis 3.0 or greater". |
| Cassandra: partitions with consistent hashing; clockwise walk to find replicas; replicas distinct physical nodes by skipping vnodes; strategies "may also choose to skip nodes present in the same failure domain such as racks or datacenters"; `(t1, t2]` ownership ranges; tokens/endpoints/hosts/vnodes/token map vocabulary; multi-token benefits and costs; `num_tokens: 16` shipped in the 4.1 configuration file; "the default number of tokens per node had to be quite high, at `256`" in the 2.x random-token era; deterministic allocator in 3.x+; "Every token introduces up to `2 * (RF - 1)` additional neighbors … the higher the probability of an outage"; "The load assigned to each node will be close to proportional to its number of vnodes"; the allocator is "Only supported with the Murmur3Partitioner" | `cassandra.apache.org` architecture documentation (5.0 docs page) and `conf/cassandra.yaml` from the Cassandra 4.1 branch, both read 2026-09-28 | ✅ | Cassandra's default partitioner class and its hash function are **not** asserted here beyond what the configuration file's own comment states. |
| Couchbase vBuckets: fixed count per bucket, set at creation and never changed; 64 (macOS), 1024 (Couchstore on Linux/Windows), 128 or 1024 (Magma); CRC32 hashing to map keys to vBuckets; the Cluster Manager owns the vBucket map and notifies clients on change; active and replica vBuckets always on different nodes; rebalance redistributes both | `docs.couchbase.com`, current, read 2026-09-28 | ✅ | |
| Kafka's built-in keyed partitioner: `Utils.toPositive(Utils.murmur2(serializedKey)) % numPartitions` | Apache Kafka source, 3.7 branch, `BuiltInPartitioner.java`, read 2026-09-28 | ✅ | Read from the project's own source tree. The same file notes the adaptive sticky partitioning described in KIP-794. |
| Kafka's ordering guarantee: "Events with the same event key … are written to the same partition" | `kafka.apache.org` documentation, read 2026-09-28 | ✅ | |
| Dynamo (DeCandia, Hastorun, Jampani, Kakulapati, Lakshman, Pilchin, Sivasubramanian, Vosshall, Vogels; SOSP'07, October 2007): "Data is partitioned and replicated using consistent hashing [10]"; virtual nodes and the three claimed advantages; "preference list contains more than N nodes"; the skip-for-distinct-physical-nodes rule; the load-balancing-efficiency definition; `S=30`, `N=3` experiments; "three orders of magnitude" smaller membership information; "a new node that joins … has taken almost a day" under strategy 1 | `amazon-dynamo-sosp2007.pdf` at `allthingsdistributed.com`, read 2026-09-28 | ✅ | Consistent hashing is attributed by Dynamo to Karger et al.; Dynamo is credited with popularising it in industry, not with inventing it. |
| Dynamo's per-`T` load-balancing-efficiency numbers | — | ❌ not relied upon | Presented as Figure 8, a plot; the per-parameter values could not be read from the text layer. **No number from Dynamo's Figure 8 appears anywhere in this guide.** Anyone needing them must read the figure. |
| Riak KV uses consistent hashing with vnodes | — | ❌ unverifiable this pass | `docs.riak.com` served a JavaScript product-selector page for the documentation URL and its `index.html` variant; the Wayback Machine reported the specific path as not archived. Recorded as a limitation, **not** as evidence against the claim. |
| Cymbal Bank figures throughout §13 | — | ❌ by construction | Fictional and illustrative. Cymbal Bank is the only bank persona in this repository, and no real bank is asserted to use any scheme in this guide. |

### 15.3 The arithmetic

Every number in §2, §3, §4, §6, §7, §11 and §13 was derived in place and recomputed with a calculator during this pass; none was taken from a secondary summary. The load-bearing results, restated with their derivations attached:

| Result | Value | Derivation |
| --- | --- | --- |
| Modulo, add a bucket, `N → N+1` | remapped `N/(N+1)` | CRT period argument: `N` of `N(N+1)` residue pairs satisfy `r = s` |
| Modulo, remove a bucket, `N → N−1` | remapped `(N−1)/N` | same argument over a period of `N(N−1)`, `N−1` matching pairs |
| Modulo, general coprime `a → b` | remapped `1 − 1/max(a,b)` | `min(a,b)` common residues per period of `ab` |
| `N=10 → 11` | 10/11 = 90.91% | product: `0.909090…` |
| `N=100 → 101` | 100/101 = 99.01% | product: `0.990099…` |
| Ring, add one node out of `N` | `1/(N+1)` moves | the new node's `R` arcs, each `1/((N+1)R)` |
| Ring, remove one node out of `N` | `1/N` moves | its `R` arcs, each `1/(NR)` |
| Ring vs modulo, add one node at `N=100` | 0.99% vs 99.01%, ratio 100× | `(N/(N+1)) ÷ (1/(N+1)) = N` |
| Worst arc with one token per node | `H_N ×` the average | `E[max of N Dirichlet(1,…)] = H_N/N`; `H_100 = 5.187`, `H_1000 = 7.485` |
| Vnode dispersion | relative spread `1/√R` | `E[X]=1/N`, `Var(X)=R(1/T)² = 1/(N²R)`, `sd/E = 1/√R` |
| Cross-check of `1/√R` | `1/√1000 = 3.16%` vs Karger et al.'s reported 3.2% | §15.1 |
| Departure absorbed by many arcs | `+100%` on one node at `R=1`; `≈+1%` each across `N−1` nodes at `R=256`, `N=100` | `1/N` divided by `min(R, N−1)` successors |
| Weight change | `1/T` of the keyspace per token added | one token = one arc |

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing

### 16.1 What could not be verified

Stated plainly, because a guide that hides its gaps is worth less than one that names them:

1. **The ACM Digital Library.** Every attempt to read an ACM record this pass failed: `dl.acm.org/doi/10.1145/258533.258660` returned HTTP 500 from the extractor, and `dlnext.acm.org` appeared only in search results. The STOC'97 paper was therefore read from a **course-site copy at Brown University** (`cs.brown.edu/courses/cs296-2/papers/consistent.pdf`), and the author list was cross-checked against two peer-paper bibliographies. **The copy read for this guide is the Brown University one**, and the attribution is flagged accordingly in §15.1.
2. **A stable PDF of the Karger paper at the authors' institution or at Princeton** — `people.csail.mit.edu/karger/Papers/consistent-hash.pdf` returned 404 and `cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf` timed out. Neither failure is evidence about the paper.
3. **Microsoft Research's copies of `tr-98-14` / `HRW98.pdf`** — the modern paths returned 404 or failed to load, and IEEE Xplore (`document/663365`, `document/663936`) was blocked. The HRW paper is therefore verified for **authors and title** (from renderings of its own first page) and **flagged** for the exact volume/year (§15.1).
4. **Dynamo's Figure 8 values.** The load-balancing-efficiency results are a plot; the per-`T` numbers are not in the text layer. **No figure from it is quoted.** The paper's *definitions* of the metric and its strategy configurations are quoted verbatim.
5. **Maglev's own default table size.** Not located as text in the paper this pass; the paper's implementation of heterogeneous weights is explicitly *not described in the paper* ("the implementation details are not described in this paper"), and the weight-proportional filling quoted in §9.5 is Envoy's documented implementation.
6. **The exact spill rule in Consistent Hashing with Bounded Loads.** Verified at abstract level only. The paper's algorithm detail — which node is consulted when a node is at its cap — is not asserted here.
7. **Riak.** Unverifiable this pass: the documentation site is JavaScript-driven, and the archived copy of the consistent-hashing concept page was not captured. No claim is made.
8. **Kafka's controller-side partition-to-broker placement.** The producer-side keyed routing is verified from source; the controller's placement algorithm is not, and this guide states no rule for it.
9. **Weighted rendezvous variants.** Described as existing, not described mechanically — the mechanics were not verified, so they are flagged rather than explained (§8.1).
10. **Any per-vendor "recommended vnode count" other than the two Cassandra values read from Cassandra's own documentation and configuration file** (`256` in the 2.x random-token era; `num_tokens: 16` shipped in 4.1) and ketama's `100-200` points per server. No other default is quoted anywhere in this guide.
11. **`web_search` behaviour, recorded honestly.** The dispatcher's briefing for this pass said `web_search` is unreliable on this host and returns empty result sets. **That did not happen this pass:** two of two `web_search` queries returned results, and they were used to locate a stable copy of the Karger paper and the HRW paper's title page. Every *verification* statement in this guide still rests on `web_extract` against a primary URL, with the two search-derived facts (the HRW title page rendering, and the location of the Brown copy) labelled as such.

### 16.2 Glossary

| Term | Meaning as used in this guide |
| --- | --- |
| **Arc** | The interval of the ring between one token (exclusive) and the next (inclusive). Load on a node is proportional to the total length of its arcs. |
| **Balance** | The property that keys are spread evenly across buckets. One of the four properties the original paper discusses. |
| **Bounded loads** | Consistent hashing with a per-node load cap, spilling keys when a node is full (Mirrokni, Thorup, Zadimoghaddam, 2016). |
| **Bucket** | The general term for a destination (a node, a shard, a partition). Modulo hashing counts buckets; a ring counts tokens. |
| **Cache stampede / thundering herd** | Many concurrent requests for the same missing key all reaching the origin, uncoalesced. |
| **Consistent hashing** | A placement family in which a small change in the bucket set induces only a small change in the key→bucket assignment. **Not** a statement about the hash function's stability. |
| **Continuum** | Ketama's name for the ring (`0…2^32`). |
| **Epoch / ring version** | A monotonic identifier for a membership state; the only way a stale client can detect that it is stale. |
| **Failure domain** | A set of components that fail together (rack, zone, hypervisor, power feed). Distinctness of nodes does not imply distinctness of failure domains. |
| **Hash tag** | Redis's `{…}` mechanism forcing keys into the same slot, buying multi-key operations at the cost of co-located load. |
| **Hot key** | A key whose popularity makes its owner hot regardless of ring balance. The ring cannot fix it. |
| **Hot node** | A node whose *arc* is too large. The ring's own problem; vnodes and weighting fix it. |
| **HRW / rendezvous hashing** | Placement by argmax over `h(key, node)` for every node; ring-free, `O(N)` per lookup (Thaler & Ravishankar). |
| **Jump consistent hash** | A 5-line `O(log n)` function of key and bucket count; no storage; buckets must be numbered sequentially (Lamping & Veach, 2014). |
| **Ketama** | The original token-ring client for memcached (`libketama`), and the generic name for a ring with multiple points per host. |
| **Load (Karger's sense)** | The property that a single bucket is not overloaded when the buckets have different views; one of the original four properties. |
| **Maglev** | Google's table-based consistent hashing: fixed prime-size table, `O(1)` lookup, permutation-based fill. |
| **Monotonicity** | The property that adding buckets moves keys only from old buckets to new ones. |
| **Movement** | The physical relocation of data or the failure of cache entries, following an ownership change. Distinct from recomputing the map. |
| **Origin** | The system of record behind a cache. The ring decides what misses; the origin pays for it. |
| **Preference list** | Dynamo's ordered replica set for a key: the first `N` distinct physical nodes clockwise, plus successors for handoff. |
| **Rebalancing** | The process of moving data after an ownership change. |
| **Replication factor (RF)** | The number of replicas per key. On a ring, replicas are chosen by continuing the clockwise walk to distinct physical nodes. |
| **Ring / hash ring** | The circular ordered identifier space onto which nodes and keys are hashed. A derived artefact, not a component. |
| **Slot** | A pre-computed partition of the keyspace (16,384 in Redis Cluster; 1,024 vBuckets in Couchbase). Assigned to nodes by a map, not derived from tokens. |
| **Spread (Karger's sense)** | The property that no key is assigned to a disproportionate number of buckets across differing client views. |
| **Token** | A single position on the ring claimed by a node. A node with `R` tokens claims `R` positions. |
| **Virtual node (vnode)** | One token of a node that holds more than one. Not a process and not a replica; a claim on an arc. |
| **`1/√R`** | The relative spread of a node's share under `R` vnodes: the mechanism of virtual nodes in one expression. |

### 16.3 Cross-references

Within `technology/`:

- [distributed_systems_engineering_guide.md](distributed_systems_engineering_guide.md) — the discipline: replication and partitioning as topic areas, the failure model, and §7.4's three-row comparison and the sentence *virtual nodes fix hot nodes, not hot keys* that §7.1 of this guide derives the mechanism behind. **Nothing in this guide contradicts it.**
- [tunable_consistency_databases_guide.md](tunable_consistency_databases_guide.md) — what a quorum can promise once a ring has chosen the replica set; consistency models; shard-per-core.
- [oracle_sharding_guide.md](oracle_sharding_guide.md) — Oracle's own sharding product, for the RDBMS-side contrast that this guide deliberately does not re-derive.
- [clickhouse_guide.md](clickhouse_guide.md) §4.2 and [spark_tuning_guide.md](spark_tuning_guide.md) §11 — the analytical-engine side of partitioning and shuffle.
- [system_design_interview_insiders_guide.md](system_design_interview_insiders_guide.md), [ddia_study_companion_guide.md](ddia_study_companion_guide.md), [grokking_system_design_companion_guide.md](grokking_system_design_companion_guide.md) — the interview and textbook framing of this material; the study route, not the mechanism.
- [architecture/billion_user_system_arch.md](architecture/billion_user_system_arch.md) — where partitioning sits in a large-scale design.
- [capacity_sizing_guide.md](capacity_sizing_guide.md) — the headroom arithmetic that §7.2, §11.1 and §13.5 depend on.

From the repository root:

- [../banking/financial_fraud_detection_at_scale_guide.md](../banking/financial_fraud_detection_at_scale_guide.md), [../banking/fix_protocol_guide.md](../banking/fix_protocol_guide.md), [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md), [../banking/market_data_consumption_guide.md](../banking/market_data_consumption_guide.md), [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) — the banking domains in which §12's layers sit, and the regulatory context for §12.2's evidence requirement.

### 16.4 Closing summary

Sixteen sections, one technique, and one correction. The ring is a derived artefact — a sorted list of tokens, rebuildable by anyone holding the same hash function and the same membership list. Virtual nodes are a variance-reduction mechanism with an exact scaling law, `1/√R`, confirmed to one significant figure by a measurement published in 1997. Replication on the ring is a distinctness rule that says nothing about failure domains until a designer makes it. Weighting is keyspace weighting, and it drifts. The technique's four genuine failure modes are hot keys, cold starts, churn and cross-partition work, and only the second of those is one the ring partly helps with. The alternatives — rendezvous, jump, Maglev, bounded loads — each give up something the ring keeps, which is why the ring is still the default. The implementations divide cleanly into rings (ketama, Dynamo, Cassandra, Envoy's ring hash) and slot maps (Redis Cluster, Couchbase), with Kafka's keyed routing sitting at modulo. The real incidents come from stale rings, not from unbalanced ones. And the operational cost of the technique is precisely the data movement it does not eliminate.

Which returns the thesis to where it started, and to the sentence this guide exists to make unavoidable: **consistent hashing does not make hashing consistent; it makes rebalancing cheap.**
