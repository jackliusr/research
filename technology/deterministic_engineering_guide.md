# Deterministic Engineering: Getting the Same Answer Twice

**Determinism as an engineering property — where it pays for itself, where it comes from, where it leaks away, and the toolkit that puts it back**

> **Jack Liu Shurui, Solution Architect**
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Engineering Foundations · **Topic:** Determinism, deterministic replay, simulation testing, reproducible builds, numerical reproducibility
> **Focus:** Banking & regulated industries (Singapore, EU, global)
> **Companion Guides:** [Temporal Workflow](temporal_workflow_guide.md) · [Durable AI Agent Workflows](durable_ai_agent_workflows_guide.md) · [Agents Work Fall Apart](ai_llm/agents_work_fall_apart_guide.md) · [DDIA Study Companion](ddia_study_companion_guide.md) · [Test Orchestration](test_orchestration_guide.md) · [Interest Calculation Engine](../banking/interest_calculation_engine_guide.md)
> **Last Updated:** September 2026

---

## Table of Contents

1. [Overview: The Term's Status and the Boundary](#1-overview-the-terms-status-and-the-boundary)
2. [Why Determinism Is Worth Paying For](#2-why-determinism-is-worth-paying-for)
3. [The Sources-of-Nondeterminism Taxonomy](#3-the-sources-of-nondeterminism-taxonomy)
4. [Toolkit I: Deterministic Replay for Debugging](#4-toolkit-i-deterministic-replay-for-debugging)
5. [Toolkit II: Deterministic Simulation Testing](#5-toolkit-ii-deterministic-simulation-testing)
6. [Toolkit III: Durable Execution and Workflow Replay](#6-toolkit-iii-durable-execution-and-workflow-replay)
7. [Toolkit IV: Reproducible Builds and Hermeticity](#7-toolkit-iv-reproducible-builds-and-hermeticity)
8. [Toolkit V: Deterministic Arithmetic and Numerical Reproducibility](#8-toolkit-v-deterministic-arithmetic-and-numerical-reproducibility)
9. [Determinism in AI and LLM Systems](#9-determinism-in-ai-and-llm-systems)
10. [The Design Decisions: When to Demand Determinism](#10-the-design-decisions-when-to-demand-determinism)
11. [Testing as a Determinism Practice](#11-testing-as-a-determinism-practice)
12. [The Regulated-Institution Angle](#12-the-regulated-institution-angle)
13. [Worked Example: Cymbal Bank's Nightly Interest Run](#13-worked-example-cymbal-banks-nightly-interest-run)
14. [Anti-Patterns](#14-anti-patterns)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified](#16-what-could-not-be-verified)
17. [Glossary](#17-glossary)
18. [Cross-References and Further Reading](#18-cross-references-and-further-reading)
19. [Closing Summary](#19-closing-summary)

---

## 1. Overview: The Term's Status and the Boundary

### 1.1 What "Determinism" Means as a Property

A computation is **deterministic** when the same inputs, in the same order, on the same system, produce the same outputs *and* pass through the same sequence of states. The second clause is the one people forget. Antithesis's engineering blog quotes the standard definition and then underlines it: "it's not just the result that is the same, but any underlying state as well — even hidden behavior that only manifests as obscure side-effects" ([antithesis.com, 2024-03-20](https://antithesis.com/blog/deterministic_hypervisor/)). That distinction — output-equality versus state-equality — is the whole technical difference between a system that *looks* reproducible and one that *is*. A nightly batch job can print the same total twice while writing different rows in different orders, leaving a downstream consumer to fail on the third run. Determinism as an engineering property is therefore always stated relative to three things:

| Axis | The question | Consequence if unspecified |
|---|---|---|
| **Input set** | Inputs, ordering, and the environment count as inputs | "Deterministic" claims that silently exclude the clock, locale or host |
| **Platform** | Same binary? Same hardware? Same library versions? | Cross-platform reproducibility claims that were never tested |
| **Granularity** | Output, state, or timing? | Bit-identical results with wildly different latency |

### 1.2 The Identity Finding: This Is an Umbrella, Not a Discipline

Before writing a guide about a term, it is worth asking whether the term is a settled thing. For **chaos engineering** there is a Principles of Chaos Engineering document; for **FAPI** there is a specification maintained by an open standards body. For **"deterministic engineering" there is no equivalent first-party source.** The honest finding from primary evidence, checked 17 September 2026, is **(b)/(c) combined: an umbrella label that different communities attach to different practices, with no canon, no principles document, no specification, and no community of its own.**

The evidence, in order of weight:

1. **The literature that exists is per-technique and separately owned.** Deterministic simulation testing belongs to the database and systems community and is documented in the *project* documentation of FoundationDB and TigerBeetle. Record-and-replay belongs to debuggers and is documented at [rr-project.org](https://rr-project.org/). Reproducible builds belongs to the [Reproducible Builds](https://reproducible-builds.org/) project and to Debian. Deterministic-algorithm modes belong to PyTorch and TensorFlow. Nobody publishes "deterministic engineering" as the name of a field.
2. **The phrase is used in the wild in senses that do not reference each other.** A hardware/PLM tooling vendor publishes "The Rise of Deterministic Engineering" (2026) about toolchain determinism in chip and product design. A consultancy publishes "The Collapse of Deterministic Software Engineering in the Age of Probabilistic AI" (walturn.com, 5 March 2026), where "deterministic engineering" means the *pre-AI paradigm* of reproducible, binary-correct software. Several AI-marketing posts use the phrase to mean "AI-assisted engineering with predictable output". These are three unrelated usages of one label.
3. **Reference-works coverage does not exist.** Wikipedia had no article for "Deterministic simulation testing" when checked on 17 September 2026 — the technique is named, in detail, only in the documentation of the projects that practise it.
**Therefore this guide does not invent a canon.** It treats "deterministic engineering" as a *convenient umbrella* for one real engineering concern — controlling the sources of variation in a system so that behaviour is reproducible — and organises itself around the two things that actually exist: **a taxonomy of the sources of nondeterminism (§3)** and **a toolkit per layer (§4–§8)**. If you came here looking for a manifesto, there isn't one, and manufacturing one would be the first anti-pattern in §14.

### 1.3 The Boundary: What Other Guides Own

Determinism already appears in this repository in four places, in each case as *someone else's subject*. This guide cross-references them and deliberately does not re-derive them:

| Guide | What it owns | Boundary here |
|---|---|---|
| [temporal_workflow_guide.md](temporal_workflow_guide.md) §4 | Temporal's own determinism model, the deterministic constraints table, `NonDeterminismError`, `getVersion`/`patch`, worker versioning | §6 below states *why* durable engines need determinism and where it breaks; it does not restate Temporal's versioning API |
| [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) §2 | Non-deterministic LLM calls as the core durable-execution problem; engine landscape; retries and cost model | §9 covers the *mechanisms* of LLM nondeterminism; the engine comparison stays there |
| [ai_llm/agents_work_fall_apart_guide.md](ai_llm/agents_work_fall_apart_guide.md) | The deterministic → assisted → autonomous viability spectrum; the should-this-be-an-agent test | §9.7 references the spectrum; it does not re-derive it |
| [ddia_study_companion_guide.md](ddia_study_companion_guide.md) | Distributed-systems fundamentals: consistency, consensus, replication, ordering | §10 assumes this vocabulary; it does not teach it |

Additional boundaries: build/hermeticity tooling detail belongs to [configuration_management_languages_guide.md](configuration_management_languages_guide.md), [ansible_vs_ansible_tower_guide.md](ansible_vs_ansible_tower_guide.md), [jib_container_builder_guide.md](jib_container_builder_guide.md) and [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md) — §7 stays at discipline level. General test strategy belongs to [test_orchestration_guide.md](test_orchestration_guide.md) — §11 covers only determinism-specific testing (frozen clocks, seeded randomness, ordered collections, simulation harnesses). Money arithmetic context belongs to [interest_calculation_engine_guide.md](../banking/interest_calculation_engine_guide.md) and [posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) — §8 covers the numerical mechanism, not the banking product. Regulated-institution context belongs to [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [risk_management_models_guide.md](../banking/risk_management_models_guide.md) and [ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) — §12 is a condensed angle, not a compliance treatment.

### 1.4 Decoder Table

Determinism vocabulary is used loosely and the looseness causes real arguments. This table is the one to keep open while reading the rest:

| Term | Definition | What it does *not* mean |
|---|---|---|
| **Determinism** | Same inputs and same platform → same output *and* same state sequence | Not the same as "correct"; a deterministic system can be deterministically wrong |
| **Idempotency** | Applying an operation twice has the same effect as applying it once | Orthogonal: an idempotent operation may be non-deterministic run to run (e.g. it may return a different but equivalent acknowledgement) |
| **Reproducible build** | Same source + environment + instructions → bit-identical *artefact* | Says nothing about how the artefact behaves at runtime |
| **Deterministic replay** | Re-executing a recorded run reproduces its control flow and data | Requires a recording; does not make a *fresh* run reproducible |
| **Simulation testing** | Running the real system against a simulated, seeded, controllable environment | Not mocking: the production code path runs, with time/network/disk stubbed |
| **Hermeticity** | A build/run closes over all its inputs; nothing is read from ambient state | Hermetic builds are usually reproducible; reproducible builds need not be hermetic |
| **Seed** | The input to a pseudo-random generator that fixes its output sequence | Fixes *the generator*, not the order in which its outputs are consumed |
| **Nondeterminism source** | A mechanism by which identical inputs yield different states | Not automatically a bug — some sources (real network timing) are irreducible |

---

## 2. Why Determinism Is Worth Paying For

Determinism costs something: ordering constraints cost throughput, coordination costs latency, replay recording costs storage and runtime. The case for paying must therefore be concrete. Five returns, each with the named class of failure it prevents:

| Return | The concrete argument | Named class of failure it prevents |
|---|---|---|
| **Reproducibility of a defect** | The oldest argument and still the strongest. rr's stated motivation is that intermittent failures "are hard to debug because any given program run may not show the failure", so you record executions until you catch one and then replay the failing execution until it is understood ([rr-project.org](https://rr-project.org/)). The counterfactual is not slower debugging; it is a class of defects that becomes *permanently unfixable* and is closed as "cannot reproduce" — the technical debt that never amortises | The bug you cannot reproduce is the bug you cannot fix |
| **Testability** | A test suite's value is its credibility. Once engineers learn that a red build might mean "flaky", the suite stops gating releases, and no amount of added coverage restores the signal. Flakiness is, in practice, the cheapest available detector of nondeterminism: a test that passes 98% of the time points at a real uncontrolled source in the code or the harness (§11.1) | The flaky test that erodes a suite's credibility |
| **Auditability and reconstruction** | An institution that must answer "why was this decision made" has to reconstruct the decision from recorded inputs and get the *same* answer (§12). Without it, audit is reduced to reading logs and hoping they captured the inputs. The reconstruction test is brutal and useful: re-perform the control from the recorded inputs and compare the output | The audit question you cannot answer |
| **Debuggability under distribution** | A distributed system's failure is a *particular interleaving*, and interleavings are not reproducible by re-running. This is why simulation testing exists: FoundationDB states its simulation "has been vital to our engineering team", adding "it seems unlikely that we would have been able to build FoundationDB without this technology" ([FoundationDB 7.4.7 docs](https://apple.github.io/foundationdb/testing.html)). That is a project's own claim about its own engineering, not an independent measurement — but it is the strongest available statement of class | The intermittent production incident, and the unreproducible customer complaint |
| **Confidence that permits change** | Every change to a system you cannot reproduce reliably is an act of faith. The practical consequence is a codebase that only accretes: nobody dares restructure the interest calculation because "we don't know why those numbers moved last time". Determinism is what turns refactoring from gambling into engineering | The system nobody dares change |

### The Expensive Failures, Named

Each class above has a budget attached and maps to a part of the toolkit: the intermittent production incident to §4 replay and §5 simulation; the unreproducible customer complaint to §5 and §8; the audit question you cannot answer to §7 hermeticity and §12; the flaky test to §11; the batch job whose totals differ run to run to §3, §8 and §13; the "worked on my machine" build to §7; and the model output that changed without a model change to §9.

The common thread: nondeterminism does not usually cause the first failure. It causes the *second* failure — the same failure recurring with no explanation, which is the expensive kind.

---

## 3. The Sources-of-Nondeterminism Taxonomy

This is the analytical core of the guide and the section worth keeping. Everything downstream — the toolkit, the decision table, the worked example — is an application of this catalogue. Each entry gives **the mechanism**, **how it manifests**, and **its containment class**:

- **REMOVABLE** — the source can be eliminated from the code entirely. Cheapest fix, and the one most often skipped.
- **CONTAINABLE** — cannot be eliminated (it is how the world works), but can be pushed behind a boundary, injected deterministically, or recorded for replay.
- **IRREDUCIBLE** — cannot be removed or fully contained; the engineering response is to bound its effect, detect it, and choose *not* to depend on it.

### 3.1 The Catalogue

| # | Source | Mechanism | How it manifests | Class |
|---|---|---|---|---|
| 1 | Wall-clock time & timers | Reading a clock or scheduling on a timer returns a value that differs per run | Same request produces different timestamps, batch windows, TTLs, ordering keys | CONTAINABLE (injected clock) |
| 2 | Randomness & UUID generation | Unseeded PRNG, `/dev/urandom`, MAC+time UUIDv1, random tie-breaks | Different IDs, different sampling, different jitter; entities appear/disappear | REMOVABLE (seed) / CONTAINABLE |
| 3 | Concurrency & scheduling | OS thread/goroutine scheduling, work-stealing, lock acquisition order | Race conditions, lost updates, different emit order; the classic heisenbug | CONTAINABLE (single-core sim) |
| 4 | Iteration order | Hash-map/set ordering across versions, pointers, hash seeds; directory listing order | Output byte-differences, unstable API responses, different reduction order | REMOVABLE (sort) |
| 5 | Floating-point & reduction order | IEEE 754 addition is not associative; parallel reductions regroup | Totals differ in the last bits, then in the first digits after accumulation | CONTAINABLE / partly IRREDUCIBLE |
| 6 | Network delivery order, retries, duplication | Packet reordering, at-least-once delivery, timeout-triggered retry | Duplicate side effects, out-of-order state, split brain | CONTAINABLE (idempotency + sim) |
| 7 | GC & allocation timing | Collector pauses, compaction, finaliser scheduling | Latency spikes, timeouts, ordering changes under memory pressure | CONTAINABLE (bounded) |
| 8 | Memory layout & ASLR | Address-space randomisation, allocator addresses, hash-of-pointer | Pointer-order iteration, address-dependent logging, crashes that don't recur | REMOVABLE (never order by pointer) |
| 9 | Environment: locale, timezone, env vars, hostname | Ambient state read implicitly by libraries and toolchains | Decimal separators, date parsing, sorting rules, build metadata | REMOVABLE (pin/freeze) |
| 10 | External dependencies | Third-party service, registry, DNS, clock skew, remote ranking | Same call, different answer; upstream change you did not make | CONTAINABLE (record/replay, pin) |
| 11 | The input stream itself | Arrival order, batch composition, timing of inputs | Different grouping changes downstream results (see §9 for the GPU case) | CONTAINABLE (canonicalise order) |
| 12 | Consumption order of a fixed RNG | A seed *fixes* a sequence, but not how many draws each path consumes | Same seed, different output, because code paths consumed different numbers of draws | REMOVABLE (per-purpose streams) |

### 3.2 Entries in Detail

**Time, randomness and the environment (rows 1, 2, 9).** Wall-clock failure is not the timestamp itself but its *use*: a batch window boundary computed from `now()`, a lease expiry, a cache TTL, an ordering key of `created_at` under sub-millisecond resolution. Injected clocks make this removable in practice — the code still reads a clock, but the clock is a dependency that the test or simulator sets. Antithesis calls time "the biggest core constraint" and the thing that "haunts pretty much anyone who's ever attempted to write a deterministic execution environment", because "time nondeterminism can easily translate into data nondeterminism through a variety of means like racing transactions, interleaved event delivery, or simply reading the system clock" ([antithesis.com, 2024](https://antithesis.com/blog/deterministic_hypervisor/)). Randomness splits into randomness that *matters* (sampling, tie-breaks, shard selection, jitter) and randomness that does not (a request ID nobody re-computes); only the first needs a seed, and seeding a generator in a multi-threaded process does not fix *which thread draws when* — per-thread or per-purpose streams are the containment. The environment class has the best-developed playbook in the whole taxonomy, because the Reproducible Builds project has spent a decade on it: "Ensure stable inputs. Ensure stable outputs. Capture as little as possible from the environment" ([reproducible-builds.org](https://reproducible-builds.org/docs/deterministic-build-systems/)) — a prescription that applies verbatim to a test harness, a batch job and a container.

**Concurrency, iteration order, GC and memory layout (rows 3, 4, 7, 8).** Concurrency nondeterminism is not a defect in itself; it is the substrate of performance. The engineering move is not to remove concurrency but to remove *dependence on its ordering*: make state transitions commutative where possible, and where they are not, impose a total order at a single point. rr's design note is instructive — it "emulates a single-core machine. So, parallel programs incur the slowdown of running on a single core. This is an inherent feature of the design" ([rr-project.org](https://rr-project.org/)): serialisation is what buys determinism, and the bill is throughput. Iteration order is the most under-estimated source because it is invisible in single-run testing: hash-map iteration can differ between processes of the same program (hash seeds), between runtime versions, and after a benign-looking refactor — changing build bytes, response bytes, cache keys, or the arithmetic order. The fix is cheap and near-universal: **never iterate a set or map whose order matters — sort it, or use an ordered structure.** GC is the canonical IRREDUCIBLE source for a managed runtime but bounded in effect: you cannot make a collector deterministic without destroying its purpose, so you pin the heap configuration, avoid *depending* on latency, and record pause-induced ordering for replay. Memory layout and ASLR merge two mechanisms under one heading — ASLR changes addresses, which matters only if something orders by address (always a bug) or logs addresses, while allocator addresses determine which object wins a lock-free push, which is how pointer-order bugs appear once per thousand runs. Same remedy: never let an address be an ordering key.

**Floating point and reduction order (row 5), network delivery (row 6), external dependencies (row 10).** Floating point is covered fully in §8; the headline is that `(a + b) + c ≠ a + (b + c)` in binary floating point, so the summation order is part of the specification whether you wrote it down or not. Network timing turns into *data* nondeterminism, and the containment pattern is old and reliable: idempotency keys plus a recorded, replayable message log. TigerBeetle documents exactly that discipline applied to money movement — "each transfer has a unique client-generated `u128` id, and each transfer is processed at most once, even in the presence of intermediate retry loops" ([docs.tigerbeetle.com](https://docs.tigerbeetle.com/concepts/safety/)). A downstream dependency is a nondeterminism source you did not write and cannot fix: an upstream service that re-ranks results, a registry serving a different patch version, a DNS answer, a vendor model whose weights changed silently. The containment pattern is a **recorded boundary** — capture the interaction once, replay it thereafter, pin the versions — and note its boundary condition: a recording makes the *consumer* deterministic and the *world* frozen; it does not make the world deterministic.

**The input stream and RNG consumption order (rows 11, 12).** Inputs arriving in a different order, or grouped into different batches, can change outputs even when every individual operation is deterministic. This is the least obvious entry in the table and the one that most often defeats "we set a seed, why did it change?" — because the seed fixed the generator and nothing fixed the batching; §9 gives the sharpest published example. A related subtlety is that a seed fixes the *sequence* but not the *mapping* from sequence to purpose: two code paths that draw in different orders, or a refactor that adds one draw, produce completely different downstream values from the same seed. The pattern that fixes it is to derive a named, independent stream per purpose (request ID, jitter, sampling), so that adding a draw in one place cannot move another. Treat this last row as engineering practice rather than a referenced finding.

### 3.3 Using the Taxonomy

Three practical rules fall out of the table.

1. **Classify before you fix.** Most "make it deterministic" requests are answered badly because a REMOVABLE source (map order, ambient locale) was treated as IRREDUCIBLE and the team built ceremony around it. Sorting a list is cheaper than a simulation.
2. **Every IRREDUCIBLE source needs a named containment boundary.** Not "we accept it" — a boundary: which component may read the wall clock, which component may generate random values, which component may retry. Nondeterminism that is merely tolerated is nondeterminism that will be discovered by an auditor or an incident.
3. **Audit the taxonomy against your own incidents.** The taxonomy's real value is diagnostic: when totals differ run to run (§13), walking rows 1–12 in order resolves the cause faster than any tool.

---

## 4. Toolkit I: Deterministic Replay for Debugging

### 4.1 What Record-and-Replay Accomplishes

Record-and-replay makes an *already-executed* run re-executable. It does not make future runs reproducible; it makes the past one repeatable, forever, under a debugger. The value proposition is stated most crisply by rr: record a failure once, "then debug the recording, deterministically, as many times as you want. The same execution is replayed every time", with the promise that "the replayed execution's address spaces, register contents, syscall data etc are exactly the same in every run" ([rr-project.org](https://rr-project.org/)). The second-order benefit is subtler and more valuable than repeatability: **debugging knowledge becomes monotonic.** With a live process, everything you learn — the address of the object of interest, the ordering of events — is destroyed the moment you restart. With replay it accumulates, because the run is frozen.

### 4.2 Mechanisms

| Mechanism | How it works | Cost profile |
|---|---|---|
| **Syscall / scheduler-level record** | Intercept every input to the process from the kernel (syscall results, signals, timing) plus the few nondeterministic CPU effects, and log them; replay by feeding the log back and forcing the same scheduling decisions | Low overhead, needs deep syscall support; the rr approach |
| **Instruction-level / hardware-assisted** | Record at instruction granularity using hardware virtualisation or performance counters; hypervisor-level variants replay a whole machine | Highest fidelity; heaviest machinery; can require privileged access or a VM |
| **Virtualised environment / whole-system** | Run the system inside a deterministic hypervisor or simulator that owns time, I/O and scheduling | Very high fidelity and reproducibility; heavy operational cost; often a product |
| **Application-event level** | Record at the application's own boundary events (requests, RPCs, DB calls) and replay those, mocking the interior | Cheapest and most portable; weakest fidelity — cannot reproduce interior races |

### 4.3 The Live Tooling (status checked 17 September 2026)

| Tool | Who runs it | First-party or commercial | Platform coverage | Status on 2026-09-17 |
|---|---|---|---|---|
| **rr** — record and replay for Linux user-space | Community project, "development sponsored by [Pernosco](https://pernos.co/) and was originated by Mozilla" | First-party open source (Linux) | Linux only. Intel Nehalem+ (2010) or later; certain AMD Zen+; certain AArch64 (ARM Neoverse N1, Apple Silicon M-series). Emulates a single-core machine; cannot record processes sharing memory outside the recording tree | **Active.** Latest release **5.9.0, published 2025-02-13**; last commit **2026-09-15**; 7,266 commits; ~10.7k stars ([github.com/rr-debugger/rr](https://github.com/rr-debugger/rr)) |
| **Shadow** — discrete-event network simulator that executes real application code | Academic/community (Tor-project-motivated) | First-party open source | Linux. Interposes at the syscall API: "implements over 150 functions from the system call API" | **Active.** Latest release **v3.3.0, 2025-10-16**; last commit **2026-09-14** ([github.com/shadow/shadow](https://github.com/shadow/shadow)) |
| **Antithesis** — deterministic hypervisor with time-travel debugging | Vendor (Antithesis) | **Commercial product.** Its FAQ states there is no free or open-source version: "Sadly, no." | x86 binaries; the whole system runs on a single CPU core inside a custom hypervisor | **Active (vendor claim).** Platform marketed as "verify every pull request"; latest Java SDK tag **v1.7.0, 2026-09-02** ([antithesis.com](https://www.antithesis.com/product/)) |
| **jvm-sandbox-repeater** — JVM record/playback ("录制/回放") for Java server-side traffic | Alibaba | First-party open source (Apache-2.0) | JVM only; plugins for HTTP, Dubbo, iBatis, MyBatis, Java, Redis, Hibernate, JPA | **Dormant.** Last commit **2022-06-10**; 68 commits total; plugin contributions dated 2019-06 to 2019-10; CI badge points at Travis ([github.com/alibaba/jvm-sandbox-repeater](https://github.com/alibaba/jvm-sandbox-repeater)) — roughly four years without a commit as at the date checked |

**Named but not verified in this pass** (see §16 for the full list): Meta's `facebookexperimental/hermit` record-and-replay engine (search evidence includes a record/replay compatibility-matrix experiment directory dated 2026-07-21, which suggests activity, but no first-party page was read); the original DetTrace reproducible-container work; Microsoft's time-travel debugging (TTD) in WinDbg; and the commercial host-side replay vendors. Do not put any of these in a design without checking their status yourself on the day you need them — that is the whole point of the maintenance-status column.

### 4.4 Trade-offs

**Recording overhead, platform coverage and storage.** rr's own published numbers are a useful anchor for what "low overhead" means in this class: on Firefox test suites, "we see slowdowns down to ≤ 1.2x", with the honest caveat that "overhead can vary dramatically depending on the workload" and that mostly-single-threaded programs fare best ([rr-project.org](https://rr-project.org/), project's own measurement). Note the direction of the trade: this is *record* overhead on a single-core-emulated machine, so parallel programs pay the serialisation cost too. Platform coverage is the real currency — rr is Linux-only and architecture-sensitive despite being the most portable serious option, and a tool that exists for one OS/arch combination is a debugging speciality for teams that standardised on that combination, not a general capability. Storage, meanwhile, is a policy question rather than a technical one: rr's traces are "durable" and "compact" and can be "ported between machines" ([rr-project.org](https://rr-project.org/)), which is exactly what makes a recording attachable to a defect ticket — and a replayable trace is evidence, and evidence has a retention schedule.

**Fidelity limits, and the external world.** rr "requires knowledge of every system call executed by the recorded processes" and "sometimes needs to be updated in response to kernel changes, updates to system libraries, or new CPU families"; it cannot record processes that share memory with processes outside the recording tree, and it auto-disables features such as X shared memory to avoid it. More fundamentally, replay reproduces the *process's* view of the world, frozen: if the defect depended on a remote service's behaviour that you did not record, replay will faithfully reproduce a run that no longer explains the incident.

### 4.5 Where It Is Production-Viable

**Production-viable:** recording in CI on failure (record every test run, keep the trace only when the test fails); recording at the edge of a single-service production incident when a debugger can be attached and the platform is Linux with a supported CPU; using whole-system simulators (§5) for pre-merge verification of distributed logic. **A debugging speciality, not a production posture:** continuous always-on recording of every production request at syscall granularity, or trying to replay a native application on an unsupported platform. Where the runtime cannot be recorded deterministically, the fallback is application-event replay (§4.2) or reconstructing inputs from an audit log — which is a different engineering property, closer to §12.

---

## 5. Toolkit II: Deterministic Simulation Testing

### 5.1 What It Is

Deterministic simulation testing (DST) runs the **real production code** against a **simulated, seeded, controllable environment**. Time, network, disk and process faults are all supplied by the simulator rather than the OS. Because every input is generated from a seed, every run is reproducible from that seed, and because the simulator owns time, it can run far faster than reality. This is the single most under-used technique in this guide's toolkit and the one most likely to be new to a reader from an application-development background.

The technique is emphatically **not mocking**. Mocking replaces the system under test; simulation keeps it and replaces its *environment*. Antithesis states its unit of reproducibility as "the state of the entire system/experiment/workload as an interconnected whole, not any single process or server within the system", which "reduces the need for complicated, domain-specific mocks and test harnesses, since you can just run (for example) both your client and server software together in the same bubble of determinism" ([antithesis.com](https://antithesis.com/blog/deterministic_hypervisor/)).

### 5.2 Lineage: The Canonical Source

The canonical articulation is a conference talk, not a paper or a specification: **Will Wilson, "Testing Distributed Systems w/ Deterministic Simulation", Strange Loop 2014** ([conference page](https://thestrangeloop.com/2014/testing-distributed-systems-w-slash-deterministic-simulation.html); recording widely mirrored). Wilson was a FoundationDB engineer, and the technique as practised today traces to FoundationDB's implementation — TigerBeetle's own VOPR documentation states its approach "was heavily inspired by the work of FoundationDB and Antithesis".

The FoundationDB documentation is the primary written source: simulation "is enabled by and tightly integrated with [Flow](https://apple.github.io/foundationdb/flow.html), our programming language for actor-based concurrency", and it "is able to conduct a *deterministic* simulation of an entire FoundationDB cluster within a single-threaded process. Determinism is crucial in that it allows perfect repeatability of a simulated run, facilitating controlled experiments to home in on issues" ([FoundationDB 7.4.7 docs](https://apple.github.io/foundationdb/testing.html)). The project reports that it "runs tens of thousands of simulations every night, each one simulating large numbers of component failures", that "our simulations usually have about a 10-1 factor of real-to-simulated time", and that "we estimate that we have run the equivalent of roughly one trillion CPU-hours of simulation on FoundationDB" — the project's own estimate, not an independent measurement (flagged as such in §15). Its simulator models the number and type of machines, drive performance and drive-full conditions, and the network at packet-delivery granularity, and it can fail machines at short intervals; the team even ran an internal competition for the nastiest fault schedule, whose winner is named "swizzle-clogging" — clogging a random subset of network connections one by one, then unclogging them in random order.

### 5.3 Current Implementations, By Name (status checked 17 September 2026)

| Implementation | Who runs it | Kind | Status on 2026-09-17 |
|---|---|---|---|
| **FoundationDB Simulation** | Apple (FoundationDB) | First-party, in-tree; coupled to the Flow language | **Active.** Docs at release **7.4.7 (published 2026-09-01)**; repo last commit **2026-09-17**; ~16.7k stars |
| **VOPR** (Viewstamped Operation Replicator) in TigerBeetle | TigerBeetle (independent company) | First-party, in-tree; the project's name for its deterministic simulator | **Active.** Release **0.17.9 (2026-07-06)**; repo last commit **2026-08-31**; ~17.0k stars. TigerBeetle's docs claim the VOPR runs "at 1000x speed" and "24/7 on 1024 cores" — *project claims, not independently verified* |
| **Antithesis** | Antithesis (commercial) | **Product**, built on a custom deterministic hypervisor they call **the Determinator**, a fork of FreeBSD's **bhyve** | **Active, commercial.** No open-source version (vendor FAQ). Open-source adjacent tools from the same team: Hegel (property-based testing) and Bombadil (UI PBT) |
| **Shadow** | Academic/community | First-party network simulator executing real application code via syscall interposition | **Active.** v3.3.0 (2025-10-16); last commit 2026-09-14 |
| **dhyve** | Open-source, third-party | Deterministic hypervisor built on bhyve, described by Antithesis as "inspired by Antithesis" | Reported; not independently verified here (§16) |
| **batch-invariant kernels** (Thinking Machines Lab) | Research lab, open source | Not a simulator: batch-invariant operator library used to make *inference* deterministic — see §9 | **Active.** MIT-licensed companion library to a published blog post; 1.1k stars, 7 commits ([github.com/thinking-machines-lab/batch_invariant_ops](https://github.com/thinking-machines-lab/batch_invariant_ops)) |

### 5.4 What DST Uniquely Buys

1. **Deterministic reproduction of rare interleavings.** A failure discovered under seed 0x5f3a is reproducible by re-running seed 0x5f3a. TigerBeetle: "Because our simulator is deterministic based on a *seed* number and the Git commit, we can perfectly reproduce any bugs discovered in testing for easy local debugging" ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)). **Time compression** compounds it: simulation can advance time faster than reality (FoundationDB reports ~10:1; TigerBeetle claims up to 1000x), which converts "a rare bug seen once in production" into "a bug seen in the first minute of simulation".
2. **A bug-finding rate production cannot provide.** Production only exercises the interleavings that happen; a simulator with fault injection *searches* for them. Antithesis's own framing is that this exploration "synthesizes unusual input data and injects faults to actively increase test coverage" and reaches "hard-to-reach code paths like exception handling and leader election" — a vendor claim, but the mechanism (guided exploration + fault injection + determinism) is documented. And because every run is reproducible, **assertions become an amplifier**: a single violated invariant is immediately actionable. TigerBeetle runs with assertions on *in production* on the argument that "it is far better to stop operating than to continue operating in an incorrect state", and keeps extra checkers in the simulator — for example verifying that replicas' data files are byte-for-byte identical across caught-up nodes, which is itself a determinism property.

### 5.5 Design Requirements It Imposes on the System Under Test

DST is not a test harness you bolt on; it is an architectural constraint. Five requirements, in dependency order. First, **a single narrow I/O boundary** — every read or write of the outside world (clock, network, disk, random, environment) must go through one place; TigerBeetle: "In the simulator, all non-deterministic parts of the system are stubbed out. This includes the clock, network, and disk operations." If time is read in fifty places, none of this works. Second, **deterministic core code** — given the same inputs, the core must produce the same state, which means no unordered iteration, no dependence on scheduler order and no reads of ambient state (§3 rows 4, 8, 9). Third, **injectable time, randomness and storage** — a clock interface, a seeded generator, and a storage interface the simulator can corrupt and slow down.

Fourth, **fault injection as a first-class, seeded parameter** — the simulator's fault schedule must itself be derived from the seed, so that a failure is replayable; TigerBeetle's VOPR "uses a random seed to tune parameters for injecting different types of faults", and when a simulation fails, "the seed and Git commit hash can be used to replay back the exact simulation and bug". Fifth, **invariants you can state** — property-based oracles. Antithesis's onboarding step 1 is "outline your invariants", because a simulator that finds a violation still needs someone to have written down what counts as a violation.

### 5.6 The Costs, Including the One People Forget

**Simulation complexity is a product you now own.** A fault-injecting, time-compressing simulator is a significant software system. FoundationDB's is coupled to a bespoke actor language (Flow); TigerBeetle's required Zig and a discipline of stubbing every side effect. This is why DST is common in databases and rare in CRUD services. **And the simulator itself becomes a system to trust** — the honest point. Everything DST tells you is conditional on the simulator being faithful: if the simulated network never reorders in the way the real one does, DST proves a property of the simulator. Simulator bugs are real bugs, and the mitigation is layered: run representative workloads against the real environment too, validate the simulator against observed production failures, and keep the simulated model as narrow as possible.

Two further costs are organisational rather than technical. **A false sense of coverage** is easy to buy: a green run means "no violation found for these invariants under this fault model", not "correct" — and if your invariants are weak, DST will happily confirm a broken system for hours. And **the fault model carries sunk cost**: models drift from reality (a kernel version changes what "slow disk" means), so treat the fault model as a maintained artefact with an owner.

### 5.7 Trade-offs Versus Real-Environment Testing

| Dimension | Deterministic simulation | Real-environment testing (incl. chaos) |
|---|---|---|
| Reproducibility of a found bug | Perfect, from seed + commit | Effectively none for interleavings |
| Bug-finding rate for concurrency/fault bugs | High — searches deliberately | Low — you must happen to hit it; chaos engineering injects faults but does not reproduce the resulting interleaving |
| Time-to-signal | Seconds to minutes of simulated time | Real time; rare bugs take months |
| Fidelity | Bounded by the simulated model | Ground truth |
| Cost to build | High, and permanent | Low to start, high to maintain as flakiness |
| Best used for | Pre-merge verification of protocols, storage, consensus, recovery paths | Validating the fault model; finding integration and environment bugs that no one modelled |

The mature position is **both, in this order**: simulation to search and reproduce; real-environment testing (including the techniques covered elsewhere in this repository's testing guides) to keep the simulator honest.

---

## 6. Toolkit III: Durable Execution and Workflow Replay

This section is deliberately bounded. Temporal's determinism model — its constraints table, `NonDeterminismError`, `getVersion`/`patch` and worker versioning — is owned by [temporal_workflow_guide.md](temporal_workflow_guide.md) §4, and the durable-execution engine landscape and retry/cost model by [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) §2. What follows is the determinism *engineering* framing that generalises across engines.

### 6.1 Why Durable Engines Require Deterministic Workflow Code

A durable engine survives crashes by **replaying** a recorded event history through the same code. This is a determinism trade in the opposite direction from recording a run: instead of capturing the run, you capture the *inputs* and re-derive the run. It works only if re-derivation is a pure function of the history. The moment workflow code would produce a different event sequence on replay than the history records, the platform cannot know whether it is looking at a bug or an intended change — so it fails the execution rather than guess.

The payoff is large: effectively exactly-once execution without application-level checkpointing code, because the logic is *re-derived* from recorded facts rather than blindly retried.

### 6.2 What the Constraint Forbids

Inside the deterministic part of a workflow: no direct wall-clock reads, no direct randomness, no direct I/O, no global mutable state, no uncontrolled threads or goroutines, and no iteration over unordered collections. The rule everyone converges on is a single sentence: **if it touches the outside world or the current moment, it belongs behind the boundary.** These six prohibitions are the §3 taxonomy rows 1, 2, 3, 4 and 9, restated as a coding rule — durable execution is the clearest proof in this guide that the taxonomy is not academic.

### 6.3 How Nondeterminism Is Handled at the Boundary

The boundary has a standard vocabulary, and it is worth recognising it in any engine:

- **Activities / side-effect steps** — the only place I/O happens; their results are recorded in the history, so replay consumes the recorded result rather than re-performing the call. Retry, timeout and heartbeat semantics attach here.
- **Side-effect markers** — a recorded value injected into the deterministic code so that a random or time-derived value used *inside* the workflow is stable across replay.
- **Deterministic time and randomness APIs** — engine-supplied replacements whose values are derived from history rather than the machine.
- **Signals/queries** — external input that is itself appended to history, which is what makes "external" inputs deterministic: they become recorded facts.

### 6.4 The Versioning Problem

The hard case is not a violation but a *legal change*: workflow code is edited while histories are live. Adding, removing or reordering a step makes new code incompatible with old histories. Every durable engine solves this with versioned, backward-compatible change — the discipline is that **workflow code changes ship like schema migrations, never as in-place edits**. The specific APIs (patch markers, build/worker versioning) are documented in [temporal_workflow_guide.md](temporal_workflow_guide.md) §4.4 and are not re-derived here. The generalisable rule: *any change to durably-recorded code is a migration with a compatibility window.*

### 6.5 Failure Modes When the Constraint Is Violated

| Violation | Symptom |
|---|---|
| A direct `now()` in workflow code | Replay computes a different branch; history mismatch; non-determinism error at replay, often days later during recovery |
| An unseeded random draw | Same as above, intermittently — depends on the value drawn |
| Direct I/O in the deterministic part | Duplicated side effects on replay (money moved twice) or divergent paths |
| A benign-looking refactor (reordering two statements) | Executions that ran fine for months begin failing on the next worker deploy |
| Reading an environment variable in workflow code | Executions replay correctly on one host and fail on another after a config change |

The last two rows are the ones that make this section worth writing: the constraint is violated most often by a change that looked harmless, and it surfaces at the worst possible time — during recovery or a deploy, not during development.

---

## 7. Toolkit IV: Reproducible Builds and Hermeticity

### 7.1 The Definition, and Why It Is Deliberately Strict

The Reproducible Builds project states the bar precisely: "A build is **reproducible** if given the same source code, build environment and build instructions, any party can recreate bit-by-bit identical copies of all specified artifacts." Verification is "by bit-by-bit comparison… usually performed using cryptographically secure hash functions" ([reproducible-builds.org/docs/definition](https://reproducible-builds.org/docs/definition/)).

Two deliberate choices in that wording matter. First, **"any party"** — self-reproducibility on the author's machine is not reproducibility; that is the anti-pattern in §14. Second, **the authors define the perimeter**: the build environment, the instructions and the artefact set are declared by the producer, and "it is preferable to reduce this set of attributes". Reproducibility is therefore always relative to a declared perimeter, which is why a reproducibility claim without a stated perimeter is unfalsifiable.

### 7.2 What Bit-for-Bit Reproducibility Requires

Reduced to the minimum, the property needs four things. The Reproducible Builds project's summary of the issue is short enough to quote as the checklist: **"Ensure stable inputs. Ensure stable outputs. Capture as little as possible from the environment"** ([deterministic build systems](https://reproducible-builds.org/docs/deterministic-build-systems/)).

| Requirement | What it means concretely | Typical leaks |
|---|---|---|
| **Closure over inputs** | Everything the build reads is either declared or inside the perimeter. No network fetch of "latest", no ambient path lookup | Floating dependency versions, unpinned registries, `PATH`-resolved tools |
| **Pinned toolchain** | Compiler and build tools are part of the declared environment (different compiler versions are *expected* to differ) | Unpinned container base images, "latest" tags |
| **No embedded timestamps or volatile metadata** | Build time, file mtimes, archive entry timestamps, version strings carrying a date | `__DATE__`, ZIP/tar mtimes, OCI image `created` |
| **No path or ordering leakage** | The build directory, absolute paths, filesystem enumeration order, locale-dependent sort | Build-path strings in debug info or generated headers; unsorted archive entries |
| **No environmental leakage** | Locale, timezone, hostname, environment variables are not read (or are frozen) | `LANG`/`LC_*` changing sort or number formatting; `TZ` |

The mechanism for the timestamp class has a standard: **`SOURCE_DATE_EPOCH`**, "a standardised environment variable that distributions can set centrally and have build tools consume this in order to produce reproducible output", specifying the last modification time of the source in seconds since the Unix epoch ([reproducible-builds.org/docs/source-date-epoch](https://reproducible-builds.org/docs/source-date-epoch/)). The page documents per-language recipes and, usefully, the state of adoption: CMake respects it only since **version 3.8.0 (April 2017)**; Docker's Buildx propagates it automatically "since Docker Buildx v0.10", where it drives the OCI image config `created` timestamp, the history timestamps and the `org.opencontainers.image.created` annotation. Two honest gaps it also records: `apt-get` does not consume `SOURCE_DATE_EPOCH` to install packages from a historical snapshot, and `rewrite-timestamp=true` is **not** the default for image layers "due to the overhead of rewriting image layers".

### 7.3 The Tooling Families and Their Status

| Family | Examples | Status / note |
|---|---|---|
| **Verification infrastructure** | The [Reproducible Builds](https://reproducible-builds.org/) project's tooling; Debian's continuous-rebuild CI at reproducible.debian.net | Active. This is the layer that makes reproducibility *falsifiable* rather than aspirational |
| **Standardised variance controls** | `SOURCE_DATE_EPOCH` (published spec), `BUILD_PATH_PREFIX_MAP` (listed as WIP) | `SOURCE_DATE_EPOCH` is the mature one; wide tool support must be checked per tool |
| **Distro-scale programmes** | Debian, and other distributions operating in the same project | Active; see §7.4 for the honest status |
| **Build-metadata capture** | `*.buildinfo` files recording the build environment; `deb-buildinfo` | Active in Debian, and load-bearing — without recorded environment, a rebuild cannot be attempted at all |
| **Declarative/hermetic build ecosystems** | Nix, Guix, Bazel-style hermetic sandboxes | **Not independently verified in this pass** (§16). Treat the claim "Nix/Bazel gives you reproducibility" as a design hypothesis to test on your own artefact, not a fact. Build-tooling detail is owned by [configuration_management_languages_guide.md](configuration_management_languages_guide.md) and the container-build guides |

### 7.4 The Standards and Attestation Angle

Reproducibility and supply-chain attestation are related but distinct, and conflating them causes bad designs.

**SLSA** is the current specification for describing supply-chain security; **version 1.2 is the current version** and organises guarantees into **tracks** with levels. The Build track runs L0 (no guarantees) → L1 (provenance exists) → L2 (signed provenance from a hosted build platform) → L3 (hardened build platform preventing runs from influencing one another and protecting the signing material) ([slsa.dev/spec/v1.0/levels](https://slsa.dev/spec/v1.0/levels), [slsa.dev/spec/v1.2](https://slsa.dev/spec/v1.2/)). Version 1.0 removed the Source aspects to focus on Build; version 1.2 adds a Source track.

The important nuance for this guide, and it is stated in the spec itself: provenance describes what was built from what, which catches mistakes and deterrence-resistant tampering — but **"whether reproducible builds are used" is left to each ecosystem or organisation to define**, alongside how expectations are formed and what happens on verification failure. In other words, **SLSA attestation does not, by itself, give you reproducibility; reproducibility is what lets you *check* that the attestation is true about the artefact.** The two together are much stronger than either alone: provenance tells you which inputs produced the artefact; a reproducible build lets an independent party confirm it from those inputs.

Related attestation machinery — in-toto, Sigstore, and SBOM formats — is named here as adjacent but **was not verified in this pass** (§16). The load-bearing distinction to hold on to: **an SBOM describes composition, provenance describes process, reproducibility describes whether the process is repeatable.** A bank's software supply-chain control needs to know which of the three it is buying.

### 7.5 Practical Limits

**Toolchains that resist reproducibility.** The project's own documentation is candid that "Not all problems have solutions currently" and keeps a list of issues found while investigating reproducibility "in its 22,000+ source packages" (a project-reported figure). Known hard cases documented publicly include CMake's RPATH handling (the build-id section differs even when the RPATH is stripped at install time), Qt's `rcc` embedding the modification time of resources since Qt 5.8 (mitigations: `--format-version 1` from Qt 5.9, `QT_RCC_SOURCE_DATE_OVERRIDE` from 5.11, `SOURCE_DATE_EPOCH` from 5.13), and CMake `AUTOMOC` embedding relative paths — workaroundable only by keeping build and source directories fixed across builds.

**Embedded build metadata is a design tension, not a bug, and distro-scale status is real but is not "done".** Teams embed git SHAs, build dates and version strings because operators need them; every one is a reproducibility leak unless it is derived from a declared input, and the resolution is to derive it deterministically (from the commit, not from the clock) — which is precisely what `SOURCE_DATE_EPOCH` formalises. On scale, Debian's own project page is blunt: "Reproducible builds of Debian as a whole is still not a reality, though individual reproducible builds of packages are possible and being done." It reports that "most packages built in sid today are reproducible under a fixed build-path and environment", that `*.buildinfo` records the build environment, and that a continuous-integration platform "builds and immediately rebuilds packages" to detect problems "related to timestamps, file ordering, CPU usage, (pseudo-)randomness and other things" ([wiki.debian.org/ReproducibleBuilds](https://wiki.debian.org/ReproducibleBuilds)). Note the qualifier holding the whole claim up: **"under a fixed build-path and environment."** Some of the page's figures are explicitly dated (for example, a note that as of 2018 there were "more than 1250 unreproducible packages in Buster") — read dated figures as historical.

### 7.6 Why It Matters Beyond Supply-Chain Security

The security case (tamper detection, verifiable provenance) is well known. The underrated case is different and matters more day to day. First, **the artefact is not the execution**: reproducing a binary bit-for-bit says nothing about whether two *runs* of that binary agree (§8 and §9 are entirely about the second property), and confusing the two produces a team that has achieved reproducible builds and still cannot reproduce its own totals. Second, **reproducibility is the precondition for forensic rebuild**: when a released artefact is implicated in an incident, the question "what code produced this, and can I rebuild it?" is answerable only if the build is reproducible — without it, you have a binary and a guess.

Third, **reproducibility disciplines the build**: the leaks that break it — ambient env vars, unordered file enumeration, unpinned dependencies — are the same leaks that break build-time determinism in tests and CI (§3 rows 4 and 9), so fixing one usually fixes the other. Fourth, **it is a check, not a promise**: a reproducibility claim that has never been independently exercised is a claim about intent, and Debian's continuous rebuild is the model — rebuild the released artefact from its recorded environment and compare hashes. Anything less is aspiration (§14).

---

## 8. Toolkit V: Deterministic Arithmetic and Numerical Reproducibility

### 8.1 The Root Cause: Floating-Point Addition Is Not Associative

The Berkeley ReproBLAS documentation states the mechanism more cleanly than most textbooks: "Floating point summation is not associative because of roundoff errors, so the computed sum depends on the order of summation. Modern processors, which may dynamically assign resources, such as variable numbers of processors, do not guarantee the same order of summation from run-to-run of the same program or subroutine" ([bebop.cs.berkeley.edu/reproblas](https://bebop.cs.berkeley.edu/reproblas/)).

The consequences are not theoretical. Reproduced locally for this guide on 17 September 2026 — Python 3.11, binary64, `random.seed(7)` and 10,000 values drawn from `random.uniform(-1e7, 1e7)` — so that every figure below is re-runnable verbatim:

```
A. 0.1 + 0.2 = 0.30000000000000004   == 0.3 ?  False

B. the six values {1e16, 1.0, -1e16, 1.0, 1.0, 1.0}:
   as written  -> 3.0
   reversed    -> 4.0
   sorted      -> 0.0
   math.fsum   -> 4.0

C. the same 10,000 values, summed forward vs reversed:
   forward  -> -309718565.9115451
   reversed -> -309718565.91154855     identical? False
   fsum     -> -309718565.91154873     stable under reordering: True

E. distinct sums across 200 shuffles of the SAME 10,000 values:  77
   distinct sums using math.fsum over 50 shuffles:                1
```

Row E is the finding worth internalising: **the same data, in the same program, produced 77 different totals depending only on summation order** (the exact count moves with the data set — a different seed gives a different number, but never 1, and never 0). A nightly batch job that sums account balances with a parallel reduction across a worker pool will produce this behaviour, and no amount of re-running will tell you which total is right.

ReproBLAS enumerates exactly why the order varies between runs, and the list is a checklist for anyone debugging a parallel total:

- data partitioning across processors;
- the number of processors available;
- the choice of instructions (for example two doubles per SSE instruction versus four per AVX);
- data alignment in memory, which can affect load order into multi-word registers;
- the shape of the reduction tree, whether scheduled statically or dynamically;
- the input order itself.

### 8.2 The Approaches, Named

| Approach | What it does | Cost / caveat |
|---|---|---|
| **Naive sequential summation** | `sum()` | Order-dependent by construction; the baseline problem |
| **Compensated (Kahan) summation** | Carries a running correction term so low-order bits are not lost | Named after W. Kahan's 1965 *Communications of the ACM* note (cited in the ReproBLAS bibliography). Reduces error; does **not** by itself make a parallel sum order-independent |
| **Reproducible / binned summation** | Pre-rounds summands into a fixed set of bins keyed by exponent, so the sum is independent of order; the accumulator is a fixed-size data structure | This is the **ReproBLAS** approach. It assumes only binary floating point per IEEE 754, round-to-nearest, and gradual underflow. Its default double-precision accumulator carries at least 80 bits, occupying six 64-bit words, and it can be reduced in one pass with one reduction — which is what allows it to be used inside BLAS-3 tiling |
| **Correctly-rounded / exact summation** | Computes the correctly rounded result of the exact sum | Higher cost; see the papers in ReproBLAS's bibliography (Kornerup–Lefèvre–Louvet–Muller 2012; Lefèvre 2017) — cited here as bibliography entries, not read in full (§16) |
| **Accurate summation utilities** | Library helpers | Python's `math.fsum` is documented as returning "an accurate floating-point sum of values in the iterable. Avoids loss of precision by tracking multiple intermediate partial sums." Note the documented caveat: accuracy "depends on IEEE-754 arithmetic guarantees and the typical case where the rounding mode is half-even", and on some non-Windows builds the C library's extended-precision addition "may occasionally double-round an intermediate sum causing it to be off in its least significant bit" ([docs.python.org](https://docs.python.org/3/library/math.html)) |
| **Exact decimal arithmetic** | Represent money as decimal with defined scale, not binary float | See §8.4. The right answer for ledgers; the wrong performance profile for scientific kernels |

The performance numbers, as reported by the ReproBLAS project itself: reproducibly summing *n* values takes "approximately 9n floating point operations", and the measured slowdown versus an optimised non-reproducible dot product is **4x on a single Intel "Sandy Bridge" core** — but "less than 1.2x" on a system of more than 512 Intel "Ivy Bridge" cores at NERSC, for the summation of 1,000,000 doubles. *These are the project's own figures; the page's last update was 17 August 2018, so treat both the numbers and the library's currency accordingly — this is a documented technique, not a maintained product.*

### 8.3 The Reduction-Ordering Problem in Parallel and GPU Computation

Three properties of accelerated hardware make this worse than on a CPU:

1. **Reduction shape depends on batch shape.** A kernel's block/thread decomposition, and therefore the tree over which partial results are combined, is chosen from the *input sizes*. Different problem size → different tree → different rounding. This is the exact mechanism the inference engines name as the root cause of LLM nondeterminism (§9.3).
2. **Atomic adds and their non-deterministic completion order.** Reduction via atomics into shared memory has an arrival order that varies run to run.
3. **Kernel and algorithm selection.** Library heuristics and autotuning pick different algorithms per shape and per machine; PyTorch documents precisely this for cuDNN convolutions: benchmarking "can be a source of nondeterminism across multiple executions", and "due to benchmarking noise and different hardware, the benchmark may select different algorithms on subsequent runs, even on the same machine" ([pytorch.org/docs/stable/notes/randomness.html](https://pytorch.org/docs/stable/notes/randomness.html)).

**The engineering rule that follows:** in any parallel numerical path whose output is *used for a decision* (a total posted to the ledger, a fee, a limit, a risk number), the reduction must be **order-independent by construction**, not merely *usually* the same. That means reproducible summation (§8.2), a fixed and canonical reduction tree, per-shard accumulation with a deterministic merge, or exact arithmetic (§8.4).

### 8.4 The Financial-Computing Discipline: Exact Decimal for Money

Binary floating point cannot represent most decimal fractions exactly — `0.1 + 0.2` evaluates to `0.30000000000000004` on any IEEE 754 binary64 implementation, as demonstrated above. Accumulating 100 additions of `0.01` in binary float yields `1.0000000000000007`, not `1.0`. Neither is a bug in Python or in the processor; both are the definition of the number format.

The discipline for money is therefore: **represent monetary amounts as exact quantities** — integer minor units (cents, pence, sen) or a decimal type with an explicit scale, never binary float; **define the scale and the rounding mode explicitly**, and round at defined points only (half-even is the common default, but the choice is a business decision that must be written down and versioned like any other rate table); **never let a float touch a ledger boundary** (a float may exist inside an analytical or pricing calculation, but it must not be the type in which a posting amount is stored, transmitted or compared); and **treat a rounding-mode change as a migration**, because changing where or how you round changes customer balances — it is a versioned change with reconciliation, not a patch.

The repository's banking guides own the product-level treatment: see [interest_calculation_engine_guide.md](../banking/interest_calculation_engine_guide.md) for the calculation engine and day-count/rounding context, and [posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md) for how exact amounts move through a posting engine. This section supplies only the numerical mechanism: **exact arithmetic removes the reduction-order class of nondeterminism (§3 row 5) from the money path entirely**, because exact addition is associative. That is a much stronger guarantee than "we sum in a fixed order", and it is why a ledger that uses decimal amounts is deterministic by construction while one that uses float requires perpetual vigilance.

### 8.5 The Mixed Case

Most real systems are mixed: a decimal ledger fed by float-derived analytics. The boundary is where the determinism work goes. Decide, per boundary, whether the number crossing it is a *decision input* (must be reproducible and exact) or an *estimate* (may vary, provided the variance is quantified and the consumer tolerates it). Naming that boundary explicitly is the single highest-value output of this section.

---

## 9. Determinism in AI and LLM Systems

This is the most commonly misstated part of the subject, so it is handled from primary sources only. The short version: **"set temperature to 0 and you get the same answer" is false as a general claim**, and the inference engines themselves say so.

### 9.1 The Honest Statement of the Seed and Temperature Question

SGLang's documentation is unambiguous: "Even with `temperature=0`, standard LLM inference can produce different outputs due to dynamic batching and varying reduction orders in GPU kernels" ([docs.sglang.ai, deterministic inference](https://docs.sglang.ai/advanced_features/deterministic_inference.html)). vLLM's documentation is equally direct: "vLLM does not guarantee the reproducibility of the results by default, for the sake of performance" — and even when its mechanisms are enabled, the guarantee is bounded, because "vLLM only provides reproducibility when it runs on the same hardware and the same vLLM version" ([docs.vllm.ai, reproducibility](https://docs.vllm.ai/en/latest/usage/reproducibility.html)). A seed is a real mechanism and does real work: it fixes the random state. It does not fix the *scheduling* or the *reduction order*, which is where the remaining divergence lives. That distinction is the entire content of this section.

### 9.2 The Documented Sources of LLM Nondeterminism

| Source | Mechanism | Where it is documented |
|---|---|---|
| **Sampling vs greedy decoding** | Sampling with `temperature > 0` draws from a distribution by definition. Greedy (`temperature=0`) removes the *draw*, not the arithmetic | SGLang, vLLM docs; general |
| **Inference-server batching and scheduling** | Dynamic/continuous batching means the set of sequences co-resident in a forward pass, and their positions, depend on arrival timing and scheduling policy. Batch size then changes kernel decomposition | SGLang docs name "dynamic batching"; vLLM's offline mode offers deterministic scheduling via `VLLM_ENABLE_V1_MULTIPROCESSING=0` |
| **GPU kernel reduction order** | "Different batch sizes cause GPU kernels to split reduction operations differently, leading to different addition orders. Due to floating-point non-associativity (`(a + b) + c ≠ a + (b + c)`), this produces different results even for identical inputs" | SGLang docs — the clearest primary statement available |
| **Autotuning and framework op nondeterminism** | Kernel/algorithm selection varies by shape and machine; inside GPU ops, asynchronous threads change the order in which floating-point numbers are added | PyTorch docs (cuDNN benchmarking); TensorFlow docs (§9.5) |
| **Mixture-of-Experts routing** | Routing is per-token, so the *set* of tokens gathered per expert — and therefore the grouping of its reductions — changes with batch composition. This is why engines validate batch invariance specifically on MoE models: vLLM's validated list includes DeepSeek-V3/R1/V3.1, `Qwen3-30B-A3B`, `gpt-oss-20b/120b` and Granite MoE; SGLang documents an MoE example (`Qwen/Qwen3-30B-A3B`) | Engine docs; mechanism as the engines describe it |
| **Preemption and recomputation under paged attention** | A preempted sequence may be recomputed, which changes what arithmetic is performed and in what grouping | **Not verified in a primary source in this pass** (§16) — treat as a hypothesis until you find the engine's own statement |

### 9.3 Batch Invariance: The Concept, and the Measured Result

**Batch invariance** is the property that a request's output does not depend on the batch size it was served in, nor on the order of requests within the batch. It is the specific property an LLM serving stack needs — and does not have by default — for reproducibility under load. vLLM's documentation describes it exactly so: "Batch invariance ensures that the output of a model is deterministic and independent of the batch size or the order of requests in a batch" ([docs.vllm.ai, batch invariance](https://docs.vllm.ai/en/latest/features/batch_invariance/)). The most useful *measured* number in this whole area comes from the Thinking Machines Lab engineering work and its companion library: the repository README reports that, running vLLM without upstreaming batch-invariant kernels, "out of 1000 random length 100 completions we see 18 unique samples. After the upstream PR, there is only one unique sample" ([github.com/thinking-machines-lab/batch_invariant_ops](https://github.com/thinking-machines-lab/batch_invariant_ops)). Eighteen distinct outputs from one thousand nominally identical requests is the concrete shape of the problem: not chaos, but a small number of recurring variants — exactly the failure mode that survives testing and appears in production.

### 9.4 What the Inference Engines Actually Document

| Engine | What it documents | Status on 2026-09-17 |
|---|---|---|
| **vLLM** | Reproducibility page: no default guarantee. Offline: set `VLLM_ENABLE_V1_MULTIPROCESSING=0` for deterministic *scheduling*, or enable batch invariance. Online: batch invariance is the only option. Seed defaults to 0 in V1 and is set per worker, so results are consistent per run "even if `temperature > 0`". Batch invariance is **beta**, requires NVIDIA compute capability 8.0+ (or Intel XPU with Triton), uses deterministic kernel implementations, disables optimisations such as custom all-reduce in tensor-parallel mode, and **costs performance** | Release **v0.29.0 (2026-09-09)**; batch-invariance tracking issue #27433; active |
| **SGLang** | Deterministic inference via `--enable-deterministic-inference`, built on Thinking Machines Lab's batch-invariant operators; supported backends are FlashInfer, FlashAttention 3 and Triton, with a compatibility matrix across CUDA graphs, chunked prefill, radix cache and non-greedy sampling; sampling seed defaults to **42**; ships determinism self-tests (`test_deterministic.py`) expecting "Unique samples: 1" | Release **v0.5.19 (2026-09-05)**; active |
| **Hugging Face TGI** | — | **Archived.** Last release **v3.3.7 (2025-12-19)**; last commit **2026-03-21**. Do not specify it for new work without checking |
| **Hugging Face Transformers** | — | Active; release **v5.17.0 (2026-09-09)** |
| **llama.cpp** | — | Active; latest build tag **b11026 (2026-09-17)** |
| **NVIDIA TensorRT-LLM** | — | Active; release **v1.3.0rc26 (2026-09-09)** |

### 9.5 What the ML Frameworks Document

Both major frameworks document determinism as an opt-in mode with explicit caveats — and the caveats are the point.

**PyTorch** states flatly: "Completely reproducible results are not guaranteed across PyTorch releases, individual commits, or different platforms. Furthermore, results may not be reproducible between CPU and GPU executions, even when using identical seeds." It offers `torch.use_deterministic_algorithms()`, which makes ops use deterministic implementations "where available" and **throws an error if an operation is known to be nondeterministic and has no deterministic alternative** — its documented example is CUDA `index_add_`. It documents the performance trade ("Deterministic operations are often slower than nondeterministic operations, so single-run performance may decrease"), and it handles undefined memory by default: `torch.utils.deterministic.fill_uninitialized_memory` defaults to `True` when deterministic algorithms are enabled, "which will fill the uninitialized memory with a known value" — because a tensor of uninitialised memory makes the output nondeterministic no matter how careful the arithmetic is. Page created/updated **14 May 2026** ([pytorch.org](https://pytorch.org/docs/stable/notes/randomness.html)).

**TensorFlow** documents `tf.config.experimental.enable_op_determinism()`, off by default: "so ops might return different results when run with the same inputs. These differences are often caused by the use of asynchronous threads within the op nondeterministically changing the order in which floating-point numbers are added." It requires the same hardware and software environment every run, notes that "determinism is not guaranteed across different versions of TensorFlow", and records performance costs including `Dataset.map` becoming "several orders of magnitude slower" when stateful random ops prevent parallel execution. A detail worth copying into any determinism design review: "latency, memory consumption, throughput, and other performance characteristics are *not* made deterministic by enabling op determinism. Only op outputs and side effects are made deterministic" — and a model may nondeterministically raise `ResourceExhaustedError` because memory consumption is not deterministic ([tensorflow.org, page last updated 2024-04-26](https://www.tensorflow.org/api_docs/python/tf/config/experimental/enable_op_determinism)).

### 9.6 Research Findings on ML Reproducibility

| Work | Finding | Date |
|---|---|---|
| Gundersen, Coakley, Kirkpatrick, Gil — *Sources of Irreproducibility in Machine Learning: A Review* (arXiv:2204.07610) | Opens from the premise that "Many published machine learning studies are irreproducible", attributes much of it to methodology and to failing to account for variation introduced by algorithms and their implementations, and builds a framework of factors organised by stage of the scientific method | v1 2022-04-15; v2 2023-04-14 |
| Henderson, Islam, Bachman, Pineau, Precup, Meger — *Deep Reinforcement Learning that Matters* (AAAI 2018; arXiv:1709.06560) | "non-determinism in standard benchmark environments, combined with variance intrinsic to the methods, can make reported results tough to interpret"; argues for significance metrics and tighter standardisation of reporting | v1 2017-09-19; v3 2019-01-30 |
| Picard — *Torch.manual_seed(3407) is all you need* (arXiv:2109.08203) | Scanned up to 10⁴ seeds on CIFAR-10 (and fewer on ImageNet with pre-trained models): "even if the variance is not very large, it is surprisingly easy to find an outlier that performs much better or much worse than the average" | v1 2021-09-16; v2 2023-05-11 |

The engineering translation of these three findings is the same one the taxonomy gives: in ML, seed choice is a *hyperparameter with measurable effect on the result*, so a single run is not evidence and a "reproduced" number from one seed is not a reproduction.

### 9.7 Is There Any Vendor Guarantee of Bit-Reproducible Inference?

**None was found in this pass.** The strongest documented positions are the *bounded* guarantees above: vLLM will give reproducibility if you disable scheduling nondeterminism or enable batch invariance, **and** you stay on the same hardware and version; SGLang offers deterministic inference under a specific set of attention backends with a documented feature-compatibility matrix. Neither is a vendor statement that inference is bit-reproducible in general, and both attach conditions that a bank's control document must reproduce verbatim. Anyone asserting an unconditional guarantee should be asked for the primary source (§14's seed-as-proof fallacy).

### 9.8 The Engineering Response: A Deterministic Surface Around a Probabilistic Core

Given that the probabilistic core cannot be made unconditionally reproducible, the design pattern is to stop trying and instead **build a deterministic surface around it**:

1. **Constrain the output space**, then **verify rather than trust.** Structured/constrained decoding and a schema (JSON Schema, grammar-constrained generation) reduce the output from free text to a validated shape; this does not make generation deterministic, it makes the *interface* deterministic — the consumer sees a value from a finite, checked domain. Then put a deterministic checker after the model: schema validation, range checks, reference-data lookups, arithmetic re-computation, a rule engine. The model proposes; the checker disposes, converting "the model might be wrong" from an unbounded risk into a bounded, logged rejection.
2. **Orchestrate deterministically, and record the inputs.** Put the probabilistic steps *inside* a deterministic control flow — exactly the durable-execution pattern from §6, where the model call is a recorded activity and the planning loop around it is deterministic workflow code (the mechanics are owned by [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) §2 and [temporal_workflow_guide.md](temporal_workflow_guide.md) §4). A model call that cannot be reconstructed is an audit problem (§12), so persist the model identifier, the seed, the sampling parameters, the prompt and the response, so a re-performance is at least *attemptable*.
3. **Fix the serving configuration where it matters, and separate the two questions.** If a decision depends on an LLM output, pin the model version, the engine version and the hardware class — the same discipline as a rate table — and know that if the workload is served online through dynamic batching, batch invariance is off by default (§9.3). Above all, keep "does the model give the same answer twice?" separate from "does the system give the same decision twice?" — the second is answerable with the techniques in this guide even when the first is not.

For where on the deterministic-to-autonomous spectrum a task should sit — and whether it should involve a model at all — see [ai_llm/agents_work_fall_apart_guide.md](ai_llm/agents_work_fall_apart_guide.md). This guide's contribution is narrower: the determinism *surface* and the honest statement of what cannot be guaranteed.

---

## 10. The Design Decisions: When to Demand Determinism

### 10.1 The Honest Starting Point

**A fully deterministic distributed system is not achievable, and no amount of engineering will make it so.** Wall-clock time, real network timing, hardware faults and ambient entropy are not removable. The engineering question is therefore never *"how do we abolish nondeterminism?"* but **"where do we contain it, and what does containment cost?"** Any design review that opens with the first question will spend its budget badly.

This is also where determinism must be separated from its neighbours in the repository's distributed-systems material ([ddia_study_companion_guide.md](ddia_study_companion_guide.md)):

- **Consistency** is about what *observers* may see (linearizability, serializability). A system can be linearizable and nondeterministic: the operations are totally ordered, but which order you got depended on network timing.
- **Determinism** is about whether the same inputs produce the same state sequence. A system can be deterministic and inconsistent: the same inputs always produce the same, wrong, order.
- **Idempotency** is about repeated application. It is the tool you use at a nondeterministic boundary (a retrying client) to make duplication harmless.

Conflating these produces two classic errors: claiming determinism as a substitute for consistency (it is not — you still need a consensus protocol), and demanding consistency where determinism was needed (coordination cost paid, reproducibility not gained).

### 10.2 The Cost Side

| Cost | Mechanism | Mitigation |
|---|---|---|
| **Serialisation / throughput penalty** | Enforcing a total order means not running in parallel, or inserting synchronisation; rr's single-core emulation is the extreme case, and its design note calls it "an inherent feature of the design" | Contain ordering to the point where it is needed; keep parallel paths order-independent rather than serialised |
| **Computation penalty** | Deterministic algorithms are slower: PyTorch documents slower ops; vLLM documents that batch invariance "may impact performance compared to the default non-deterministic mode. This trade-off is intentional to guarantee reproducibility"; TensorFlow documents `Dataset.map` becoming "several orders of magnitude slower". Reproducible summation costs ~9n operations (ReproBLAS) | Apply only on decision paths; measure the real delta on your workload rather than accepting a vendor's order-of-magnitude |
| **Coordination cost** | A shared ordering service, a global clock, a sequencer adds latency and a new failure mode | Prefer per-aggregate ordering (a key, an account) over global ordering |
| **Availability cost** | Stronger coordination can mean less availability under partition (the classical trade) | Decide which paths genuinely need it; a reconciliation total usually does not need to be available during a partition |
| **Engineering and cognitive cost** | Determinism is a constraint every developer must respect forever; violated by benign-looking refactors (§6.5) | Encode it in the toolchain: injected clocks, no direct ambient reads, lint rules, and a test that fails when a wall clock is called |
| **Simulator/harness ownership cost** | A simulator is a system with its own backlog (§5.6) | Fund it explicitly, or do not start |

### 10.3 The Scoping Principle: Where It Earns Its Keep

Four paths justify the cost. Everything else should be evaluated against them:

| Path | Why determinism earns its keep | What "done" looks like |
|---|---|---|
| **The money path** | Exactness is a regulatory and reputational requirement, not an optimisation. Posting, interest, fees, limits, settlement | Exact arithmetic (§8.4); reproducible reduction order; a reconciliation that ties to the penny |
| **The audit path** | A decision must be reconstructable from recorded inputs and re-performable (§12) | Recorded inputs, versions and rounding modes sufficient to re-perform; the re-performance is *actually executed* periodically |
| **The replay path** | If a workflow orchestrates side effects, it must replay correctly or it will duplicate money (§6) | Deterministic workflow core; every world-touching step behind an activity |
| **The test path** | A suite whose failures are not reproducible has no signal (§11) | Frozen clock, seeded randomness, pinned locale/timezone, ordered collections, zero unexplained flakiness |

### 10.4 The Decision Table

Apply per computation, not per system. The first column is the question that decides it:

| If the output… | Determinism needed? | Containment to apply |
|---|---|---|
| Is a *posted monetary amount* or affects a customer balance | **Yes — mandatory** | Exact decimal/integer arithmetic; canonical ordering; versioned rounding; reconciliation |
| Is *used to make a decision about a person or an account* (limit, risk score, eligibility) | **Yes for the decision, not for the score's derivation** | Record inputs + model/rule version so the decision is reconstructable; verify with a deterministic checker (§9.8) |
| Must be *shown to a supervisor as a re-performable control* | **Yes — and it must be exercised** | Recorded inputs; periodic re-performance test; evidence of the re-performance |
| Is a *generated artefact* that is released (binary, image, package) | **Yes for the artefact** | Reproducible + hermetic build (§7); independent rebuild verification |
| Is a *protocol or storage state machine* (consensus, recovery, replication) | **Yes — and simulation is the practical way to test it** | Deterministic core, stubbed I/O, seeded fault injection (§5) |
| Is an *estimate, ranking, or recommendation* with a human in the loop | **No — bound the variance instead** | Quantify run-to-run variance; do not compare single runs; log the variant |
| Is a *cached value or a performance optimisation* | **No** | Order-independence or idempotency is usually enough; accept divergence |
| Is a *metrics/trace label* (request ID, timestamp) | **No** | Do not let it become an ordering key (§3 rows 1, 2) — this is where nondeterminism turns into bugs |
| Is an *exploratory analysis* | **No — reproducibility of the *method* is enough** | Pin the input snapshot and the code version; the seed is allowed to vary |

The asymmetry to notice: the cost of determinism is paid in **throughput and engineering discipline**; the cost of nondeterminism is paid in **incident MTTR, audit findings and reconciliation disputes**. The first is budgetable; the second is not. When in doubt on the money, audit or replay path, pay for determinism. When in doubt elsewhere, quantify the variance and move on.

---

## 11. Testing as a Determinism Practice

This section covers only what determinism changes about testing. General test strategy — levels, pyramid, orchestration, environments — belongs to [test_orchestration_guide.md](test_orchestration_guide.md).

### 11.1 The Flaky Test Is a Detector, Not a Nuisance

A flaky test is the cheapest nondeterminism instrument you own: it is a controlled experiment that sometimes produces different results from identical inputs, which is the definition of a nondeterminism source (§3). The engineering response has three parts:

1. **Treat a flaky test as a defect report against the code or the harness — never as a reason to retry.** Automatic retry-on-failure is the single most destructive practice in this area: it converts a reliable signal into a permanently suppressed one, and it removes the only instrumentation that would have found the source.
2. **Diagnose it with the taxonomy.** §3's rows are a diagnostic order: is it a clock (row 1), an unseeded or differently-consumed RNG (rows 2, 12), a concurrency order (row 3), an unordered collection (row 4), a float reduction (row 5), an external dependency (row 10), or an input/batch composition (row 11)? Most flakes resolve on row 4 or row 1.
3. **Measure the rate, not just the presence.** "Flaky" is a rate. A test that fails once in 200 runs is a different problem from one that fails one in three, and the rate is the evidence that a fix worked. Recording the rate also distinguishes a genuine source from an environment defect.

The flip side is worth stating: **a suite with 100% pass stability is not proof of determinism** — it may simply never exercise the interleavings (which is exactly the gap DST closes).

### 11.2 Property-Based and Model-Based Testing

These two families are the mechanisms that *surface* nondeterminism rather than assuming it away:

- **Property-based testing (PBT)** asserts invariants over *generated* inputs rather than asserting outputs for hand-written examples. Generators reach states you would not have written down, and a failing case comes with a seed, which makes it reproducible by re-running the generator with that seed. This is why PBT pairs naturally with simulation (§5): the generator becomes the fault/invariant space, and the seed becomes the reproduction handle. Antithesis's onboarding explicitly starts with "outline your invariants" and its open-source PBT tools (Hegel, Bombadil) are named in §5.3; classic PBT lineage traces to QuickCheck (named here for orientation, not verified in this pass — §16).
- **Model-based testing** runs a simplified reference model in parallel with the implementation and asserts that they agree on observable behaviour. TigerBeetle's storage checkers are a variant of this: the simulator checks that replicas' data files are byte-for-byte identical across caught-up nodes.

Both are how you get *invariants you can actually assert* — which §5.5 identified as a prerequisite for a simulation to be useful. A simulator with no oracles is a random number generator.

### 11.3 Test-Environment Determinism Requirements

A test environment must supply what the taxonomy says the code should not read:

| Requirement | What it means | Failure it prevents |
|---|---|---|
| **Frozen clock** | Time is injected, not read. Advance it explicitly; assert against the injected value | Date-dependent test failures (month ends, leap years, DST transitions, quarter boundaries) |
| **Seeded randomness** | Every generator seeded from the test's own seed, and the seed printed on failure. Per-purpose streams so one code path's draws cannot shift another's (§3 row 12) | Irreproducible failing cases; tests that pass alone and fail in suite |
| **Controlled timezone and locale** | A pinned `TZ` and locale in the harness, plus at least one test that runs under a *second* timezone and locale | "Passed in Singapore, failed in London" — date parsing, decimal separators, collation |
| **Ordered collections** | Never assert against map/set iteration order; sort before comparing; prefer ordered fixtures | Test failures that depend on hash seeds or runtime version |
| **Pinned dependencies and reference data** | Frozen rate tables, fixture snapshots, pinned library versions | Tests that fail because a third party changed (row 10) |
| **No ambient network or filesystem** | The harness provides the boundary; the test fails loudly rather than reaching out | Tests that pass only on the developer's machine |
| **The simulation harness as the environment** | For distributed logic, the test environment *is* the deterministic simulator, with faults as a test parameter | The interleaving-dependent bug that no unit test can reach |

### 11.4 What to Do With the Seeds

A practical habit worth institutionalising: **a failing test must print the reproduction handle.** For a seeded test that is the seed; for a simulation run it is seed + commit; for a replayed run it is the trace. If the CI log does not contain the handle, the failure is not yet actionable, and the team will re-run the test until it passes — the anti-pattern in §14.

---

## 12. The Regulated-Institution Angle

Condensed, because the regulatory context is owned elsewhere in this repository: [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) for resilience and third-party arrangements, [risk_management_models_guide.md](../banking/risk_management_models_guide.md) for model risk and validation, [ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md) for AI-specific obligations. What follows is what determinism contributes to those agendas.

**I am not asserting specific supervisory rule text here.** The clauses, article numbers and supervisory expectations of any given regime must be read from the regulator's own publication for the jurisdiction in question; §16 records that I did not verify rule text for this guide. What is stated below is the *structural* requirement that recurs across supervisory regimes in different wording, because it follows from the nature of supervision rather than from any one rulebook.

### 12.1 The Reconstruction Requirement

The recurring structural requirement is this: **a regulated institution must be able to take a past decision, balance or trade, and re-derive it from recorded inputs, producing the same result.** Supervisors ask for this because oversight is impossible if the institution can only *describe* what it did and cannot *re-demonstrate* it. Three properties make reconstruction possible: the **inputs are recorded, not inferred** (the reference data, rates, parameters, rounding mode, versions and ordering used at the time); **the computation is a function of those inputs** (no dependence on the current clock, the current configuration, or the current library version); and **the reconstruction is exercised** — a re-performance capability that has never been run is an assertion, not a control.

That third property is the determinism discipline's hardest gift and its most-often-skipped one. Re-performing a control from recorded inputs on a schedule — and comparing outputs exactly — is a piece of evidence most institutions could produce and very few do.

### 12.2 The Audit Trail as a Determinism Artefact

An audit trail is not (only) a log. For reconstruction purposes it must be **sufficient to re-run the computation**: inputs, ordering, parameters, versions, rounding, and the outcome. A trail that records inputs and outputs but not the *versions and parameters* lets you explain what happened but not re-perform it. The engineering specification is therefore: *the trail must be a complete, minimal input set for a deterministic function.*

Log hygiene interacts with §3 here. Timestamps and generated IDs in a trail are correct and necessary — but they must never be *inputs* to the computation being audited, or the trail becomes an input to its own audit. Keep generated identifiers and ordering keys distinct.

### 12.3 Exactness in Monetary and Regulatory Calculation

Regulatory capital, interest, fees, limits and reported figures must reconcile exactly. The numerical discipline in §8.4 is the mechanism: exact decimal or integer-minor-unit arithmetic on the money path; explicitly versioned scaling and rounding; reproducible reduction order for any parallel aggregation that feeds a reported figure. A reported number that differs between runs cannot be defended, and the defence "the difference is only in the last digits" does not survive a supervisor asking whether the same question gets the same answer twice.

### 12.4 The Evidence a Supervisor Expects for a Re-Performable Control

Stated as an evidence set you could assemble today:

| Evidence | What it demonstrates |
|---|---|
| Recorded inputs and parameters for the control period | The reconstruction is possible |
| Version identifiers for code, reference data, models and rules | The reconstruction is of the *same* subject |
| A re-performance run with an exact comparison to the original result | The reconstruction actually reproduces |
| A dated log of re-performances, including failures and their resolution | The capability is a control, not a claim |
| For parallel aggregations, evidence of order-independence or a canonical order | The result does not depend on how the work was distributed |
| For any model in the path, version and configuration of the model | Model reproducibility expectation (below) |

### 12.5 Model Reproducibility for a Regulated Model

A regulated model must be reconstructable to a versioned artefact with versioned inputs — this is the classical model-risk expectation, and it is a *weaker* requirement than bit-reproducible inference. The distinction matters enormously when the model is an LLM or a GPU-trained ML model. **For a trained model**, the training run need not be bit-reproducible, but the *model artefact*, its inputs, its version and its evaluation results must be pinned and reproducible — and the frameworks' documented limits (§9.5) mean cross-version and cross-platform bit-reproducibility should not be assumed. **For an LLM in a process**, design for *decision* reproducibility, not generation reproducibility (§9.8): record the model version, prompt, sampling parameters, seed, engine version and the response, and verify the output with a deterministic checker, so the *decision* is reconstructable even though the generation is not bit-for-bit repeatable. And **be explicit in the documentation about which of the two you are claiming** — a validation document that claims "the model is reproducible" without saying which is a finding waiting to happen.

### 12.6 The Honest Limits

Where reconstruction is genuinely impossible, say so and compensate:

| Limit | Why it is irreducible | What institutions do instead |
|---|---|---|
| **Human judgement and unstructured inputs** | The "input" was a conversation, a document, a decision-maker's reasoning | Record the rationale as text and the decision as an event; reconstruct the *decision*, not the reasoning |
| **Sampled model output at temperature > 0** | Sampling is definitionally not reproducible without the seed, and even with a seed the engine conditions apply (§9.4) | Record the output itself as the fact; verify with a deterministic checker; do not claim bit-reproducibility |
| **Public data sources that have changed** | The upstream source no longer serves the historical value | Snapshot and date-stamp the input; store the snapshot as the system of record |
| **Third-party processes you do not control** | Their internal nondeterminism is out of reach | Contractual pinning, recorded responses, reconciliation of *outcomes* rather than re-execution of their computation |
| **Hardware faults and rare hardware-dependent arithmetic** | The machine that produced the result is gone or nondeterministic at the bit level | Verify outcomes by invariant and reconciliation rather than bit equality; keep the recorded inputs as the reproduction handle |

The pattern in the right-hand column is consistent: **when you cannot reproduce the computation, record its output as a fact and verify its consistency by a different route.** That is a legitimate engineering answer, provided it is written down as a limit rather than discovered during an examination.

---

## 13. Worked Example: Cymbal Bank's Nightly Interest Run

**The bank is fictional. Cymbal Bank is a persona used to illustrate architecture, and the figures in §13.6 are illustrative engineering estimates produced for this guide, not measurements from any real institution, vendor benchmark or production system. Do not cite them as evidence.**

### 13.1 The Incident

Cymbal Bank runs a nightly interest accrual and fee assessment batch across deposit and loan portfolios. The job: read balances and rate tables as of the value date, compute per-account daily interest and fees, aggregate, and post. It runs on a distributed worker pool and writes a control total per product line that Finance reconciliation uses the next morning.

On a Monday, the daily control total for one product line differed from Sunday's expectation by a small amount despite no rate change, no balance movement in the affected accounts, and no code deploy. Operations re-ran the job for the same value date: **the total changed again**, by a different small amount. A third run produced a third total. None of the three was explicable from the inputs. Finance held the reconciliation pending, which pushed a regulatory reporting deadline into a two-day variance.

The incident was not a failed run. Every run finished green and posted internally consistent entries. The failure was that *the same inputs produced different answers*, which meant no one could say which of the three was correct.

### 13.2 Diagnosis Using the §3 Taxonomy

The team walked the taxonomy deliberately, top to bottom, and reported what they found at each row. The value of the table was that it stopped the investigation from becoming "let's add logging and re-run":

| §3 row | Source | Finding |
|---|---|---|
| 1 | Wall-clock / timers | Partition assignment used a wall-clock scheduler; partitions (and therefore the grouping of accounts) differed between runs. **Confirmed contributor.** The value date itself came from a separately-read clock, which was stable |
| 3 | Concurrency | Worker pool with work stealing; assignment order varied. **Confirmed contributor** — and it is what made row 1 bite |
| 4 | Iteration order | Fee assessment iterated a hash map of fee rules keyed by product code; iteration order differed between JVMs. **Confirmed contributor** — it changed nothing arithmetically, but it changed the order in which accrual adjustments were applied |
| 5 | Floating-point / reduction order | Per-account interest was exact decimal, but the **aggregation to the control total used `double`** with a parallel reduction over partitions. **Root cause of the differing totals.** Evaluated locally for this guide, 200 shuffles of the same 10,000 values produced 77 distinct sums (§8.1) — exactly this failure mode |
| 6 | Network / retries | At-least-once retry on the posting call; no duplicate postings observed, because the posting engine used idempotency keys. Not a contributor |
| 9 | Environment | `TZ` differed between two workers. Not implicated here because value dates were passed explicitly — but it had caused an unrelated incident the prior quarter |
| 10 | External dependencies | The rate table service returned a `lastUpdated` field that one code path used for a tie-break. **Minor contributor** |
| 11 | Input stream | The account list was fetched in pages; page boundaries varied with the fetch window. **Contributed to partition composition** |
| 2, 7–8, 12 | Randomness / UUID, GC and memory layout, RNG consumption order | **Ruled out.** No unseeded randomness in the calculation (event IDs were cosmetic), no GC or layout effects beyond latency, and no RNG in the calculation path |

Three findings mattered, and the ordering is the lesson: **row 5 produced the differing total, but rows 1/3/4/11 produced the varying reduction order that made row 5 visible.** Fixing only the float aggregation would have hidden the symptom while leaving a control total that still depended on partition composition.

### 13.3 The Chosen Interventions, by Layer

| Layer | Intervention | Taxonomy rows addressed |
|---|---|---|
| **Arithmetic** | Replace `double` accumulation with exact decimal amounts at a defined scale for the control total; define the rounding mode explicitly and version it. Per-account amounts were already exact | 5 |
| **Aggregation** | Make the aggregation order-independent: accumulate per partition in exact form, then merge with a canonical, documented partition ordering (sorted by partition key) | 5, 11 |
| **Scheduling** | Replace wall-clock partition assignment with a deterministic function of the account key (stable hashing), so the same input set always produces the same partition assignment | 1, 3, 11 |
| **Iteration** | Sort fee-rule iteration by product code; forbid iterating the rule map directly. A lint rule was added | 4 |
| **External input** | Remove the `lastUpdated` tie-break from the calculation path; capture rate-table versions in the run record instead | 10, and the audit path in §12 |
| **Reconciliation** | Add a second, independent total computed by a different method (sequential exact summation over the reconciled account set) and compare. Divergence fails the batch rather than posting | Detection of any residual source |

Note what was *not* done: the worker pool was not serialised. Parallelism was kept; only the *order-dependence* was removed. That is the general shape of good determinism work — remove the dependence, keep the concurrency.

### 13.4 The Design Change That Contained It

The structural change was to introduce a single, explicit **deterministic aggregation boundary** in the batch. Each worker emits, per partition, an exact per-account result set plus a partition key; the reducer takes the full set of partition results and orders them by partition key — a total order derived from the data, not from arrival; reduction is then performed in exact decimal on that canonical order; and the control total is derived from the exact figure and rounded once, at the end, with the versioned rounding mode.

Everything outside that boundary — the worker pool, the scheduling, retries, parallel I/O — remains concurrent and, in principle, nondeterministic. Everything inside it produces the same total for the same inputs. This is §10.2's "contain, don't abolish" made concrete: the containment boundary is four lines of specification, and it is written down and versioned.

### 13.5 The Testing Change That Would Have Caught It

The incident was detectable by a test Cymbal did not have. Four tests were added, in increasing order of value. A **permutation test** — take a fixed input set, run the aggregation with N randomised partition orderings and shuffle seeds, and assert that the control total is *byte-identical* across all runs — is the test that fails loudly on the original code and the direct analogue of the local demonstration in §8.1. A **value-date replay test** re-runs a stored historical batch input and asserts equality with the stored output, which is the reconstruction requirement (§12.1) enforced in CI.

The third test is a **summation-order property test**: for generated sets of amounts, assert that the aggregation equals a reference exact computation regardless of input order. The fourth is a **two-method reconciliation test**, asserting that the parallel aggregation and an independent sequential exact computation agree on every batch — the production guard from §13.3, asserted in test. Test 1 alone would have caught the incident before release; test 4 is what makes the class of failure self-detecting rather than dependent on someone writing the right test.

### 13.6 The Performance Cost (Illustrative Figures)

**The following figures are illustrative engineering estimates for this worked example. They are not measurements from any real system, and they must not be cited as benchmarks.** They are stated so the *shape* of the trade is visible:

| Change | Illustrative cost | Comment |
|---|---|---|
| Exact decimal accumulation instead of `double` for aggregation | Order of 2–5× slower on the aggregation step only | Aggregation was a small fraction of batch wall-clock in this scenario; the arithmetic change was not the bottleneck |
| Canonical ordering of partition results | Negligible (a sort of a few thousand keys) | Sorts are cheap; the earlier problem was ordering that *was not chosen* |
| Stable hashing for partition assignment instead of wall-clock scheduling | Neutral to slightly better | Removed a scheduler read, not a computation. Reported as neutral |
| Second independent reconciliation total | Order of +10% end-to-end batch time | The largest *end-to-end* cost, and purely additive: independent verification is not free |
| Removal of the external tie-break | Slightly faster (one fewer call) | The simpler design was the faster design — a common outcome |

The honest summary for this example: **the determinism work was cheap; the independent verification was the expensive part, and it was the part worth buying.**

### 13.7 The Reconciliation and Audit Implications

**Reconciliation changed from a comparison to a contract.** Finance receives the control total plus evidence of the two-method agreement, and a divergence fails the batch instead of producing a number to argue about. **The run record became an audit artefact**, with storage changed to persist per-partition exact results, the partition key ordering, rate-table versions, the code version and the rounding-mode version — the complete input set for a deterministic function (§12.2).

**Re-performance became possible and was scheduled**: a quarterly exercise re-runs a sample of historical value dates from the run record and compares exactly — the control that, before the incident, existed only as an assertion. **And the lesson was recorded:** the control total had *looked* reproducible for years because nothing had changed the partition composition. Stability under an unchanged environment is not determinism; it is luck with a long expiry.

### 13.8 Rollout

The changes shipped in four steps, deliberately in this order, because the fastest value came from the ordering fixes and the risk sat in the arithmetic:

1. **Observer first.** Ship the second-method reconciliation total in *detect-only* mode — log divergences, do not fail the batch, do not change the posted figures. This quantified the real divergence rate before anything was changed, and gave the team evidence to size the problem.
2. **Ordering fixes.** Deterministic partition assignment and sorted fee-rule iteration. No arithmetic change, so the posted figures were unchanged; divergence rate fell measurably.
3. **Arithmetic change with a shadow period.** Exact aggregation, run in shadow against the legacy total for a defined period; every difference investigated to the cent before cutover. Cutover was a versioned change with Finance sign-off and a reconciliation statement, not a patch (§8.4, item 4).
4. **Re-performance control.** Enable fail-on-divergence, and schedule the quarterly re-performance. Documentation updated to state which determinism property is claimed and which is not.

The ordering principle generalises: **make the problem visible and measured before you change behaviour on the money path.**

### 13.9 What Remains Nondeterministic, and Why That Is Now Acceptable

Being honest about the residual is the point of the exercise:

| Still nondeterministic | Why | Why acceptable |
|---|---|---|
| **Wall-clock timestamps** on postings and events | Genuinely cannot be fixed — the bank does not control time | They are recorded facts, and they are not inputs to a calculation. §12.2's rule holds: generated values are never calculation inputs |
| **Execution schedule** — which worker processed which account, how long the batch took | Concurrency and the OS scheduler remain uncontrolled. Serialising would have cost the throughput that makes the window feasible | Partition *assignment* is now a deterministic function of the key, so the schedule cannot change the *result*. Which worker did it is recorded, not depended on |
| **Retry counts and transient call failures** | The world is unreliable | The boundary is idempotent, so a retry changes nothing observable in the outcome |
| **Reconciliation timing and page boundaries of the source fetch** | Page composition still varies | No longer reachable from the control total, because ordering and partitioning are canonical and the tie-break source was removed |
| **Full bit-level reproducibility of the underlying runtime** | Not a stated property of the platform, and not worth buying (see §9.5's framework caveats) | The claimed property is narrower and testable: for a given input set and version set, the control total is reproducible. That is the property the control needs |
| **The rounding-mode version's correctness** | Structural: the bank chooses a rounding convention; determinism guarantees the choice is applied consistently, not that the choice is right | Correctness is owned by Finance and validation; determinism only guarantees *repeatability* (§1.1 — deterministic and wrong is still wrong) |

The framing to carry away: **Cymbal did not make the batch deterministic. Cymbal moved the nondeterminism to places where the outcome does not depend on it, and wrote down what remains.** That is the achievable version of the whole discipline.

---

## 14. Anti-Patterns

Eight in the "too much / misplaced trust" direction and one in the opposite direction — each with symptom, cause and guardrail.

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 14.1 | Chasing total determinism where it cannot exist | A design review that cannot close because "the system is still not fully deterministic"; serialisation creeping into the request path; a wall-clock ban no one can comply with | Determinism treated as a binary property of a system rather than a scoped property of a computation (§10.1) | Require every determinism requirement to name its *path* and its *scope*: which output, at which boundary, over which input set |
| 14.2 | The seed-as-proof fallacy | "We set the seed, so it's deterministic"; a number reproduced from one seed treated as validated; an LLM pipeline declared reproducible because `temperature=0` | Confusing *fixing the generator* with *fixing the computation*: taxonomy rows 11 and 12 are invisible to a seed (§3.2), and the engines document that temperature 0 is not sufficient (§9.1) | Never accept a seed as evidence; ask for the test that varies order, batch composition, worker count and hardware class and asserts equality (§13.5) |
| 14.3 | The reproducible-build claim that reproduces only on the author's machine | The claim passes locally; an independent rebuild fails — "works on my machine" wearing a compliance badge | The definition requires that **"any party"** recreate bit-identical artefacts; reproducing twice on one machine exercises no path, locale, ordering or timestamp leakage | Independence is the test: a different machine, directory, locale, user — ideally a different organisation (§7.6) |
| 14.4 | The simulation that tests the simulator | Green simulation runs for months, then production finds a fault class the simulator never modelled | The simulator is a system with its own bugs and fault model (§5.6), never validated against reality; green means "no violation under this model", not "correct" | Keep the model narrow and documented; try to reproduce real incidents in it — failure to reproduce is a simulator gap, not a null result — and keep real-environment tests |
| 14.5 | The determinism constraint violated by a benign-looking refactor | A deploy or recovery — not a test — fails with a non-determinism error, on executions that ran for months; or histories replay wrongly and side effects duplicate | Reordering statements, moving an I/O call, adding a draw or reading an env var in the deterministic part of a durably-recorded workflow (§6.5); invisible until an *existing* history is replayed | A constraint is only as strong as its cheapest violation: injected clocks and randomness, lint rules against direct time/IO in the deterministic layer, and a replay test before deploy |
| 14.6 | The flaky test suppressed rather than diagnosed | Retry-on-failure configured; the suite goes green; the race ships; the failure later appears in production where there is no retry button | Flakiness treated as an inconvenience rather than as the cheapest nondeterminism detector available (§11.1); retrying destroys the instrument | Flakiness has an owner and a defect budget; a failure must print its reproduction handle (seed, commit, trace) before quarantine; quarantine expires |
| 14.7 | The replay tooling nobody uses in an incident | Replay is deployed and recording; in the next incident nobody reaches for it and the team re-runs the scenario by hand | A tool is not a capability: not in the runbook, never used under pressure, absent from the incident template | Put replay in the incident process: named in the runbook, exercised in a game-day, the trace handle an incident artefact alongside logs and metrics |
| 14.8 | The audit claim of reconstruction never tested by re-performing it | Documentation states decisions can be reconstructed; nobody has run one; an examination reveals the recorded inputs are insufficient | A re-performance capability treated as documentation rather than as a control with an execution record (§12.4) | Schedule it: a quarterly re-performance with exact comparison and a logged result — including failures — is the difference between a claim and evidence |
| 14.9 | The over-constrained system that bolted down determinism where it bought nothing | A request path serialised "for determinism" where nobody compares runs; a throughput ceiling defended by a requirement no one can trace to a decision; developers working around it | A determinism requirement inherited rather than derived: the cost paid once (§10.2) and then forever, while the benefit accrued to nobody because no path needed it (§10.3) | Every determinism constraint gets a named consumer and a review date; if no path in §10.3 depends on it and no incident motivated it, remove it |

The symmetry between §14.8 and §14.9 is the point: **under-constraining and over-constraining are both failures of the same discipline — naming which property you need, on which path, for which consumer, and then measuring whether you got it.**

---

## 15. The Claims Audit

Every factual claim in this guide, with its status, source, source quality and the date checked. **All checks were performed on 17 September 2026** unless the source itself carries an older date, which is stated separately.

Status key: **VERIFIED** = read from the primary source named; **FLAGGED** = self-reported by an interested party, or read from a secondary citation, so treat as directional; **REJECTED** = a claim commonly made that the evidence did not support.

| # | Claim | Status | Source (quality) |
|---|---|---|---|
| 1 | "Deterministic engineering" is a settled discipline with a canon, principles document or specification | **REJECTED** | No such artefact found via primary sources; per-technique literature is separately owned; Wikipedia had no "Deterministic simulation testing" article (primary: absence, checked 2026-09-17) |
| 2 | The phrase is used in the wild in mutually unrelated senses (hardware/PLM toolchain determinism; the pre-AI software paradigm; AI engineering marketing) | **VERIFIED** | koddex.io blog post; walturn.com article dated 2026-03-05; LinkedIn post (low quality: vendor and thought-leadership sources) |
| 3 | FoundationDB's Simulation conducts a deterministic simulation of an entire FDB cluster within a single-threaded process; integrated with the Flow language | **VERIFIED** | [apple.github.io/foundationdb/testing.html](https://apple.github.io/foundationdb/testing.html), FDB 7.4.7 docs (primary, project) |
| 4 | FDB: ~10:1 real-to-simulated time; "tens of thousands of simulations every night"; "roughly one trillion CPU-hours of simulation" | **FLAGGED** | Same FDB page — the project's own estimates, not independently measured |
| 5 | Will Wilson's "Testing Distributed Systems w/ Deterministic Simulation", Strange Loop 2014, is the canonical articulation of DST | **VERIFIED** | [thestrangeloop.com/2014](https://thestrangeloop.com/2014/testing-distributed-systems-w-slash-deterministic-simulation.html) (primary conference page; read via search result, video recording exists) |
| 6 | TigerBeetle's simulator is the VOPR (Viewstamped Operation Replicator); seeded by a seed number plus the Git commit; clock, network and disk are stubbed out; "heavily inspired by" FoundationDB and Antithesis | **VERIFIED** | [docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) (primary, project) |
| 7 | VOPR runs "at 1000x speed" and "24/7 on 1024 cores"; "one minute of VOPR time is equivalent to days of real-world testing" | **FLAGGED** | [docs.tigerbeetle.com/concepts/safety](https://docs.tigerbeetle.com/concepts/safety/) and the VOPR doc — project claims, not independently measured |
| 8 | Antithesis is a commercial product with no open-source version, built on a custom deterministic hypervisor ("the Determinator") forked from FreeBSD's bhyve | **VERIFIED** | [antithesis.com/product](https://www.antithesis.com/product/) and [the hypervisor blog, 2024-03-20](https://antithesis.com/blog/deterministic_hypervisor/) (primary, vendor — labelled as vendor claims where they concern results) |
| 9 | rr is active: release 5.9.0 (2025-02-13), last commit 2026-09-15, ~10.7k stars; development sponsored by Pernosco, originated by Mozilla | **VERIFIED** | GitHub API + [rr-project.org](https://rr-project.org/) (primary) |
| 10 | rr is Linux-only, x86 Nehalem+/some AMD Zen/some AArch64, emulates a single-core machine, cannot record processes sharing memory outside the tree | **VERIFIED** | [github.com/rr-debugger/rr](https://github.com/rr-debugger/rr) README (primary) |
| 11 | rr recording overhead "down to ≤ 1.2x" on Firefox test suites | **FLAGGED** | rr-project.org — the project's own benchmark; the page itself says overhead "can vary dramatically" |
| 12 | jvm-sandbox-repeater is dormant: last commit 2022-06-10, 68 commits, plugin work dated 2019 | **VERIFIED** | [GitHub commits page](https://github.com/alibaba/jvm-sandbox-repeater/commits/master) (primary) |
| 13 | Shadow is active (v3.3.0, 2025-10-16; last commit 2026-09-14); it executes real application code via syscall interposition and supports deterministic replication | **VERIFIED** | [shadow.github.io/docs](https://shadow.github.io/docs/guide/) + GitHub API (primary) |
| 14 | Hugging Face TGI is archived: last release v3.3.7 (2025-12-19), last commit 2026-03-21 | **VERIFIED** | GitHub API repository metadata (primary) |
| 15 | PyTorch: "Completely reproducible results are not guaranteed across PyTorch releases, individual commits, or different platforms… may not be reproducible between CPU and GPU executions, even when using identical seeds"; `torch.use_deterministic_algorithms()` throws on nondeterministic ops | **VERIFIED** | [pytorch.org/docs/stable/notes/randomness.html](https://pytorch.org/docs/stable/notes/randomness.html), page created/updated 2026-05-14 (primary) |
| 16 | TensorFlow: op determinism is off by default; nondeterminism is "often caused by the use of asynchronous threads within the op nondeterministically changing the order in which floating-point numbers are added"; determinism not guaranteed across TF versions | **VERIFIED** | [tensorflow.org API docs](https://www.tensorflow.org/api_docs/python/tf/config/experimental/enable_op_determinism), page dated 2024-04-26 (primary) |
| 17 | vLLM "does not guarantee the reproducibility of the results by default, for the sake of performance"; reproducibility only on the same hardware and same vLLM version; seed defaults to 0 in V1 | **VERIFIED** | [docs.vllm.ai reproducibility](https://docs.vllm.ai/en/latest/usage/reproducibility.html) (primary, project) |
| 18 | vLLM batch invariance is beta, needs NVIDIA compute capability 8.0+ (or Intel XPU + Triton), uses deterministic kernels, disables custom all-reduce in tensor parallel, and costs performance | **VERIFIED** | [docs.vllm.ai batch invariance](https://docs.vllm.ai/en/latest/features/batch_invariance/) (primary) |
| 19 | SGLang: "Even with `temperature=0`, standard LLM inference can produce different outputs due to dynamic batching and varying reduction orders in GPU kernels"; root cause is batch-size-dependent reduction splitting plus floating-point non-associativity; backends FlashInfer/FA3/Triton; default sampling seed 42 | **VERIFIED** | [docs.sglang.ai deterministic inference](https://docs.sglang.ai/advanced_features/deterministic_inference.html) (primary, project) |
| 20 | Without batch-invariant kernels, 1000 random length-100 completions yielded 18 unique samples; with the upstream PR, 1 unique sample | **FLAGGED (as a measurement)** | [thinking-machines-lab/batch_invariant_ops](https://github.com/thinking-machines-lab/batch_invariant_ops) README — the authors' own reported experiment; the mechanism is independently documented by SGLang |
| 21 | No vendor guarantees bit-reproducible inference | **VERIFIED BY ABSENCE** | Engine and framework documentation reviewed (rows 15–19); both engines attach hardware/version conditions rather than offering an unconditional guarantee |
| 22 | IEEE 754 binary64 addition is not associative; the same 10,000 values produced 77 distinct sums across 200 shuffles, and `math.fsum` produced 1 | **VERIFIED (local computation)** | Python 3.11 on 2026-09-17; the seed (`random.seed(7)`) and generation parameters are stated in §8.1, so the figures are re-runnable verbatim (primary/local) |
| 23 | ReproBLAS provides order-independent summation with a ≥80-bit default accumulator; ~9n floating-point operations; 4x slowdown on one Sandy Bridge core, <1.2x on >512 Ivy Bridge cores | **VERIFIED (project page) — but currency FLAGGED** | [bebop.cs.berkeley.edu/reproblas](https://bebop.cs.berkeley.edu/reproblas/) — page states "Last update August 17, 2018", so treat the library as documented-technique rather than maintained product |
| 24 | Kahan's compensated-summation note: *Communications of the ACM* 8(1), 1965 | **FLAGGED** | Read only as a bibliography entry inside the ReproBLAS references, not from the original (secondary citation) |
| 25 | A build is reproducible when "any party" can recreate bit-by-bit identical artefacts from the same source, environment and instructions | **VERIFIED** | [reproducible-builds.org/docs/definition](https://reproducible-builds.org/docs/definition/) (primary, standards project) |
| 26 | `SOURCE_DATE_EPOCH` is a standardised environment variable; CMake respects it since 3.8.0 (April 2017); Docker Buildx propagates it since v0.10; `apt-get` does not consume it | **VERIFIED** | [reproducible-builds.org/docs/source-date-epoch](https://reproducible-builds.org/docs/source-date-epoch/) (primary; the CMake and Buildx version dates are from that page) |
| 27 | "Reproducible builds of Debian as a whole is still not a reality"; most packages in sid are reproducible "under a fixed build-path and environment"; `.buildinfo` records the build environment; CI rebuilds packages to detect variance | **VERIFIED** | [wiki.debian.org/ReproducibleBuilds](https://wiki.debian.org/ReproducibleBuilds) (primary; the page mixes undated and explicitly-dated figures, e.g. a 2018 note on unreproducible Buster packages — read dated figures as historical) |
| 28 | SLSA v1.2 is current; the Build track is L0–L3; "whether reproducible builds are used" is defined by each ecosystem or organisation, not by the specification | **VERIFIED** | [slsa.dev/spec/v1.0/levels](https://slsa.dev/spec/v1.0/levels), [slsa.dev/spec/v1.2](https://slsa.dev/spec/v1.2/) (primary, standards body) |
| 29 | Gundersen et al.: "Many published machine learning studies are irreproducible" | **VERIFIED** | [arXiv:2204.07610](https://arxiv.org/abs/2204.07610), v1 2022-04-15, v2 2023-04-14 (primary) |
| 30 | Henderson et al.: non-determinism in benchmark environments plus intrinsic method variance makes reported RL results hard to interpret (AAAI 2018) | **VERIFIED** | [arXiv:1709.06560](https://arxiv.org/abs/1709.06560) (primary) |
| 31 | Picard: scanning up to 10⁴ seeds, outliers performing much better or worse than average are "surprisingly easy" to find | **VERIFIED** | [arXiv:2109.08203](https://arxiv.org/abs/2109.08203), v1 2021-09-16 (primary) |
| 32 | Temporal's determinism model, variable-versioning API and non-determinism error are owned by the sibling guide | **VERIFIED (local)** | [temporal_workflow_guide.md](temporal_workflow_guide.md) §4 read directly during preparation (primary/local) |

**Highest-risk classes, restated.** Two fact categories in this space decay fastest and are the most often asserted from memory: **tool maintenance status** (rows 9–14 — several of these change within months, and one of them, TGI, moved from flagship to archived while this area was being written about) and **LLM-determinism claims** (rows 17–21 — where the correct answer is always a conditional one, and the condition varies by engine version). Both should be re-checked on the day they are relied upon, not cited from this table.

---

## 16. What Could Not Be Verified

Listed explicitly so no reader mistakes silence for confirmation. None of these is asserted as fact anywhere in this guide.

1. **Supervisory rule text for reconstruction, re-performance and model reproducibility.** §12 states the *structural* requirement and deliberately cites no clause, article or supervisory publication. No regulator's text was read for this guide, so no article number may be attributed to this guide's authority.
2. **Microsoft's time-travel debugging (TTD) in WinDbg.** Named in §4.3 as an unverified adjacent capability; its current maintenance status and platform coverage were not established.
3. **Meta's `facebookexperimental/hermit`.** Search evidence showed a record/replay compatibility-matrix experiment directory dated 2026-07-21, which suggests activity, but no first-party page was read. Status unresolved.
4. **The original DetTrace reproducible-container work.** The search surfaced a repository using the name, not the original research artefact. Paper, venue and status unverified; omitted from the toolkit tables.
5. **Undo, Replay.io and the commercial host-side replay vendors.** Not checked at all.
6. **Preemption and recomputation under paged attention as an LLM nondeterminism source** — the mechanism is plausible and widely repeated, but no engine's own documentation stating it was found. §9.2 marks it as a hypothesis.
7. **Nix, Guix and Bazel reproducibility claims.** Named in §7.3 as a family and explicitly not verified; no first-party source was read, and no claim is made about which of them delivers reproducibility on any given artefact.
8. **in-toto, Sigstore and SBOM formats.** Named in §7.4 as adjacent attestation machinery; no specification was read, and no status or capability is asserted.
9. **Debian's current reproducibility percentage.** §7.5 quotes the project's own qualitative statements and one explicitly 2018-dated figure rather than a live percentage, because the live figures are published as continuously-updated charts which were not captured at a fixed date here.
10. **The exact wording of `Kahan (1965)`.** Cited only via the ReproBLAS bibliography (row 24 of §15); the original paper was not read.
11. **SGLang's and vLLM's longevity and version-to-version stability of their determinism features.** Both document their features as current and (for vLLM's batch invariance) beta; neither claims stability across versions, and no longitudinal data was gathered.
12. **Any numerical figure in §13.** The Cymbal Bank worked example is fictional and every performance figure in §13.6 is an illustrative estimate produced for this guide. There is nothing here to verify because nothing here is a measurement.
13. **The scope of the "deterministic engineering" identity finding as a searched negative.** The assessment in §1.2 rests on primary sources plus the absence of a canon; general web search was intermittent during preparation (it returned empty results for a period and blocked direct query extraction), so the negative finding is strong on *absence of a specification in the technology literature* and weaker as a claim about every corner of the web.

---

## 17. Glossary

| Term | Definition |
|---|---|
| **Batch invariance** | The property that a model's output does not depend on the batch size it was served in, nor on the ordering of requests within the batch. Documented by vLLM as a beta feature requiring specific hardware. |
| **Binned summation** | A reproducible-summation technique that pre-rounds summands into fixed exponent-keyed bins so the result is independent of summation order. The ReproBLAS approach. |
| **Containment class** | This guide's classification of a nondeterminism source as REMOVABLE (can be eliminated), CONTAINABLE (must be pushed behind a boundary or injected) or IRREDUCIBLE (can only be bounded and detected). |
| **Determinism** | The property that the same inputs on the same platform produce the same output *and* the same sequence of states. |
| **Deterministic replay** | Re-execution of a recorded run reproducing its control flow and data. Requires a recording; makes the past repeatable, not the future. |
| **Deterministic simulation testing (DST)** | Running the real system under test against a simulated, seeded environment in which time, network, disk and faults are provided by the simulator. |
| **Event history** | In durable execution, the append-only record of decisions and results that workflow code is replayed against. |
| **Flaky test** | A test that produces different results from identical inputs. Treated in this guide as a nondeterminism detector, not a nuisance. |
| **Hermeticity** | The property that a build or run closes over all its inputs and reads nothing from ambient state. |
| **Idempotency** | The property that applying an operation twice has the same effect as applying it once. Distinct from determinism. |
| **Injected clock** | A time source supplied as a dependency, so tests and simulations control time instead of reading the machine clock. |
| **Non-determinism error** | The failure a durable-execution engine raises when replayed code would produce a different event sequence from the recorded history. |
| **Reduction order** | The order in which a set of values is combined by a summation or reduction. In floating point, the order is part of the specification whether written down or not. |
| **Reproducible build** | A build in which any party, given the same source, environment and instructions, can recreate bit-identical artefacts. |
| **Reproducible summation** | Any scheme that makes a sum independent of summation order; for example binned accumulation. |
| **Seed** | The input that fixes a pseudo-random generator's output sequence. Fixes the generator, not the consumption order or the reduction order. |
| **Simulation harness** | The test environment for distributed logic, in which the deterministic simulator supplies time, I/O and faults as parameters. |
| **SOURCE_DATE_EPOCH** | The standardised environment variable that supplies a build's reference timestamp so tools produce reproducible output. |
| **Time compression** | A simulator's ability to advance simulated time faster than real time, measured as a real-to-simulated ratio. |
| **VOPR** | TigerBeetle's deterministic simulator, the Viewstamped Operation Replicator. |

---

## 18. Cross-References and Further Reading

### 18.1 Inside This Repository

- **Determinism as someone else's subject:** [temporal_workflow_guide.md](temporal_workflow_guide.md) §4 (the determinism model, versioning), [durable_ai_agent_workflows_guide.md](durable_ai_agent_workflows_guide.md) §2 (nondeterministic LLM calls in durable execution), [ai_llm/agents_work_fall_apart_guide.md](ai_llm/agents_work_fall_apart_guide.md) (the deterministic-to-autonomous spectrum), [ddia_study_companion_guide.md](ddia_study_companion_guide.md) (consistency, consensus, replication)
- **Testing:** [test_orchestration_guide.md](test_orchestration_guide.md) (general test strategy; this guide adds only the determinism-specific requirements)
- **Build and hermeticity tooling:** [configuration_management_languages_guide.md](configuration_management_languages_guide.md), [ansible_vs_ansible_tower_guide.md](ansible_vs_ansible_tower_guide.md), [jib_container_builder_guide.md](jib_container_builder_guide.md), [docker_image_comparison_tools_guide.md](docker_image_comparison_tools_guide.md)
- **Money and arithmetic context:** [interest_calculation_engine_guide.md](../banking/interest_calculation_engine_guide.md), [posting_engine_core_banking_guide.md](../banking/posting_engine_core_banking_guide.md)
- **Regulated-institution context:** [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [risk_management_models_guide.md](../banking/risk_management_models_guide.md), [ai_genai_banking_compliance_guide.md](../banking/ai_genai_banking_compliance_guide.md)
- **Repository root:** [readme.md](../readme.md)

### 18.2 Primary Sources Referenced

- **Simulation testing:** [FoundationDB — Simulation and Testing](https://apple.github.io/foundationdb/testing.html) · [TigerBeetle VOPR](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) · [TigerBeetle safety concepts](https://docs.tigerbeetle.com/concepts/safety/) · Will Wilson, *Testing Distributed Systems w/ Deterministic Simulation*, Strange Loop 2014 · [Antithesis — writing a deterministic hypervisor](https://antithesis.com/blog/deterministic_hypervisor/)
- **Replay:** [rr](https://rr-project.org/) and its [repository](https://github.com/rr-debugger/rr) · [Shadow](https://shadow.github.io/docs/guide/) · [jvm-sandbox-repeater](https://github.com/alibaba/jvm-sandbox-repeater)
- **Numerical reproducibility:** [ReproBLAS](https://bebop.cs.berkeley.edu/reproblas/) · Python [`math.fsum`](https://docs.python.org/3/library/math.html)
- **Reproducible builds and provenance:** [reproducible-builds.org — definitions](https://reproducible-builds.org/docs/definition/), [deterministic build systems](https://reproducible-builds.org/docs/deterministic-build-systems/), [SOURCE_DATE_EPOCH](https://reproducible-builds.org/docs/source-date-epoch/) · [Debian ReproducibleBuilds](https://wiki.debian.org/ReproducibleBuilds) · [SLSA v1.2](https://slsa.dev/spec/v1.2/) and [v1.0 levels](https://slsa.dev/spec/v1.0/levels)
- **ML/LLM determinism:** [vLLM reproducibility](https://docs.vllm.ai/en/latest/usage/reproducibility.html) and [batch invariance](https://docs.vllm.ai/en/latest/features/batch_invariance/) · [SGLang deterministic inference](https://docs.sglang.ai/advanced_features/deterministic_inference.html) · [Thinking Machines Lab batch-invariant ops](https://github.com/thinking-machines-lab/batch_invariant_ops) · [PyTorch reproducibility](https://pytorch.org/docs/stable/notes/randomness.html) · [TensorFlow op determinism](https://www.tensorflow.org/api_docs/python/tf/config/experimental/enable_op_determinism) · [arXiv:2204.07610](https://arxiv.org/abs/2204.07610) · [arXiv:1709.06560](https://arxiv.org/abs/1709.06560) · [arXiv:2109.08203](https://arxiv.org/abs/2109.08203)

---

## 19. Closing Summary

"Deterministic engineering" is not a discipline and this guide has not pretended otherwise. It is an umbrella over five separately-owned bodies of practice — deterministic replay, deterministic simulation testing, durable-execution replay, reproducible builds, and numerical reproducibility — plus one that is still being worked out in public, LLM inference reproducibility. What unites them is a single engineering property: **the same inputs should produce the same answer, and where they cannot, the difference should be a decision you made rather than a surprise you received.**

The reusable asset is §3's taxonomy. Every technique in §§4–9 is an answer to one of its twelve rows, and every anti-pattern in §14 is a taxonomy row that was ignored, mis-classified, or over-classified. Twelve rows, three containment classes — REMOVABLE, CONTAINABLE, IRREDUCIBLE — and one rule that resolves most arguments before they start: **classify the source before you spend money on it.** Sorting a list is cheaper than a simulator; a lint rule is cheaper than an incident.

Three honest positions are worth carrying away. First, **a fully deterministic distributed system is not achievable**, so the question is always where to contain the nondeterminism, never how to abolish it — and the containment boundary should be a small, written, versioned thing, as it was in §13. Second, **the simulators and the tools are systems too**: a simulator has a fault model that can be wrong, a replay trace has an external world it did not capture, and a reproducibility claim that has never been independently exercised is a statement of intent. Third, **the seed is not proof** — not in a test, not in a build, and above all not in an LLM, where the engine vendors themselves decline the unconditional version of the claim.

The economics are asymmetric and that is the whole argument. Determinism is paid for in throughput, latency and permanent discipline. Nondeterminism is paid for in incident MTTR, unreproducible customer complaints, reconciliation disputes and audit findings you cannot re-perform. The first bill is budgetable and negotiable; the second arrives unannounced. So scope it, name the consumer, pay on the money path, the audit path, the replay path and the test path — and for everything else, quantify the variance and move on.

None of this makes a system correct. Determinism only guarantees that when you ask the same question again, you get the same answer twice.

