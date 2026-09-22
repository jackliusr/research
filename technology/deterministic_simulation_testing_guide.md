# Deterministic Simulation Testing: Building the Environment You Get Wrong

**The construction-and-practice half of DST — how the simulator is actually built, what it costs to own, and the three places it stops working.**

> **Jack Liu Shurui, Solution Architect**
> **Repo:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Deterministic Engineering · **Topic:** Deterministic simulation testing — construction and practice
> **Focus:** The architectural inversion that makes a system simulatable; the I/O boundary and what to do about dependencies you do not control; simulated time, network, storage and crash model; the deterministic scheduler and the seed discipline; fault injection as a seeded parameter; assertions and invariant probing; the seed→replay CI workflow; the cost of ownership with arithmetic; the three hard limits; alternatives and complements; and a worked Cymbal Bank ledger example
> **Companion Guides:** [Deterministic Engineering](deterministic_engineering_guide.md) §5 · [Antithesis: The Product](antithesis_guide.md) · [Chaos Engineering](chaos_engineering_guide.md) · [Turso Database](turso_database_guide.md)
> **Last Updated:** September 2026

**Source convention:** every mechanism in this guide is attributed in prose and tabulated in the claims audit (§15.3) with its source, the date checked, and its quality — **first-party** (the project documenting its own system), **project claim** (a first-party performance or capability assertion), **conference talk** (an abstract or talk, which is not a specification), or **secondary**. Project claims are labelled where they appear and are not restated as measured facts; anything that could not be verified is recorded in §16.1 rather than asserted. Nothing here is written from memory.

---

## Table of Contents

1. [The Overview, the Boundary and the Decoder](#1-the-overview-the-boundary-and-the-decoder)
2. [The Architectural Inversion](#2-the-architectural-inversion)
3. [The I/O Boundary](#3-the-io-boundary)
4. [Simulated Time](#4-simulated-time)
5. [The Simulated Network](#5-the-simulated-network)
6. [Simulated Storage and the Crash Model](#6-simulated-storage-and-the-crash-model)
7. [The Deterministic Scheduler and the Seed Discipline](#7-the-deterministic-scheduler-and-the-seed-discipline)
8. [Fault Injection as a First-Class, Seeded Parameter](#8-fault-injection-as-a-first-class-seeded-parameter)
9. [Writing Tests That Find Bugs](#9-writing-tests-that-find-bugs)
10. [The Seed-to-Replay Workflow: CI and Debugging](#10-the-seed-to-replay-workflow-ci-and-debugging)
11. [What It Costs to Own](#11-what-it-costs-to-own)
12. [Where It Stops Working](#12-where-it-stops-working)
13. [The Alternatives and the Complements](#13-the-alternatives-and-the-complements)
14. [The Cymbal Bank Worked Example](#14-the-cymbal-bank-worked-example)
15. [The Anti-Patterns and the Claims Audit](#15-the-anti-patterns-and-the-claims-audit)
16. [What Could Not Be Verified](#16-what-could-not-be-verified)

---

## 1. The Overview, the Boundary and the Decoder

**The thesis in one line: a deterministic simulator keeps the system and replaces its environment, so the environment becomes the thing you build, maintain and get wrong.**

That single sentence is the whole guide. Everything that follows is either the mechanics of "replacing the environment" or the consequences of "getting it wrong". The technique is not a test harness and it is not a framework selection — it is an engineering project whose deliverable is a second implementation of the world your system runs in, and whose maintenance lifecycle is as long as the system's own.

### 1.1 What This Guide Does Not Own (and Why That Declaration Matters)

This guide is deliberately the second half of a two-part treatment. The first half already exists and must be read first.

[**deterministic_engineering_guide.md**](deterministic_engineering_guide.md) **§5, "Toolkit II: Deterministic Simulation Testing" (lines 194–249), owns the whole of the following, and none of it is re-derived here:**

- **The definition of DST** — running the real production code against a simulated, seeded, controllable environment.
- **The "this is not mocking" distinction** — mocking replaces the system under test; simulation keeps it and replaces its environment.
- **The canonical source** — Will Wilson's Strange Loop 2014 talk, and the FoundationDB documentation as the primary written source.
- **The implementations landscape table** — FoundationDB Simulation/Flow, VOPR in TigerBeetle, Antithesis/Determinator, Shadow, dhyve — with dated statuses as at 2026-09-17.
- **The two benefits of DST** — deterministic reproduction of rare interleavings, and a bug-finding rate production cannot provide.
- **The five architectural requirements, in dependency order** — (1) a single narrow I/O boundary; (2) deterministic core code; (3) injectable time, randomness and storage; (4) fault injection as a first-class, seeded parameter; (5) invariants you can state.
- **The line that matters most for budgeting** — "simulation complexity is a product you now own".
- **The comparison table** of deterministic simulation versus real-environment testing.
- **Its claims audit, which already flags VOPR's "1000x" and "1024 cores" figures** as project claims rather than independently verified measurements.

This guide does **not** restate any of that. It assumes a reader who has read §5 and now has to *build* the thing.

**A sibling guide, [`antithesis_guide.md`](antithesis_guide.md), is being written in parallel and owns the Antithesis product, company, pricing, market positioning and history.** Antithesis appears in this guide only as one named implementation among several — a commercial example of the same architectural shape, built on a custom deterministic hypervisor they call the Determinator. No pricing, no market analysis, no product deep-dive, no company history appears below. Where Antithesis's own documentation is the clearest statement of a mechanism, it is cited as a vendor claim, in one sentence, and the reader is pointed to the companion guide.

The division of labour, stated as a table, is the fastest way to see the boundary:

| Question | Owner |
|---|---|
| What is DST? Is it mocking? Who invented it? Which implementations exist? | [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5 |
| What five properties must the system under test have? | [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.5 |
| Does DST beat real-environment testing? | [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.7 |
| How do I actually restructure a codebase so it can be simulated? | This guide §2, §3 |
| How do the clock, network and disk get modelled, exactly? | This guide §4, §5, §6 |
| How does the seed reach every random draw? | This guide §7 |
| What do I put in the fault catalogue, and what do I assert? | This guide §8, §9 |
| How do I run this in CI and reproduce a failure on my laptop? | This guide §10 |
| What does owning this cost, and when is it the wrong choice? | This guide §11, §12, §13 |
| Is Antithesis worth buying? What does it cost? | [antithesis_guide.md](antithesis_guide.md) |

### 1.2 The Decoder: Ten Terms, Defined for Construction

The vocabulary of DST is small and each term has a precise operational meaning. Misusing any one of them is how a project ends up building an elaborate mock framework and calling it a simulator.

| Term | What it means in construction terms | What it is often confused with |
|---|---|---|
| **I/O boundary** | The single, narrow interface through which the system touches the outside world — clock, network, disk, entropy, environment. One interface, two implementations: real and simulated. | "We have a repository layer", which is usually a hundred leaks pretending to be one |
| **Deterministic executor** | The component that runs the system's tasks in a defined order, in one thread, with no reliance on the OS scheduler. It owns *when* code runs, not *what* code runs | A thread pool with a mutex; a real event loop, which is at the mercy of the OS |
| **Simulated time** | A counter the simulator advances. Timers, sleeps and timeouts are scheduled against this counter and fire in simulated order, not real order | Wall-clock speedup; "the tests get faster" |
| **Simulated network** | A data structure holding in-flight messages, with delivery, delay, reorder, duplication, partitioning and loss expressed as parameters the simulator chooses | A local socket pair, or a toxiproxy-style latency shim over a real socket |
| **Fault schedule** | The concrete, ordered sequence of faults a particular run will experience, *derived from the seed* — not a probability table consulted at random | A random fault injector; a chaos experiment definition |
| **Seed** | A single integer that, together with the code revision and the configuration, fully determines the run. It is an input, threaded explicitly through the system | A value printed in a log line and never used again |
| **Replay** | Re-running the same code revision with the same seed and configuration and getting the identical execution — the same interleaving, the same faults, the same failure | Record-and-replay, which reproduces a *past* run from captured inputs rather than re-deriving it |
| **Shrinking / minimisation** | Reducing a failing case to a smaller one that still fails, so the report is a thirty-line trace rather than a three-hour one | Something DST generally does well; see §10.5 — this is an OPEN problem here |
| **Model-based checker** | A simplified reference implementation (or reference invariant) run alongside the real one, asserting that the two agree on observable behaviour | A unit test with expected values hard-coded |
| **Invariant** | A property that must hold in every state the system passes through, written down by a human, and asserted continuously by the simulator | A test assertion at the end of a scenario |

Two of these deserve a second sentence, because they are the ones that get built wrong.

A **fault schedule** is not a coin flip. If the simulator called `rand()` at the moment it needed to decide whether to drop a packet, the run would be reproducible only if the *number and order* of those calls were fixed — and the number and order depend on the execution path, which depends on the faults already injected. The usual construction is to derive the whole schedule up front, or to draw from a seeded generator whose draw sequence is itself determined by an explicitly ordered walk of simulated time. §7 and §8 return to this.

A **model-based checker** is where corruption and split-brain bugs are caught, and it is the part most teams omit in a first attempt because it requires writing the same logic twice. §6.5 covers the approach.

### 1.3 What This Guide Does Not Cover

This guide does not cover the definition, lineage, landscape or benefit case for DST — that is §5 of [deterministic_engineering_guide.md](deterministic_engineering_guide.md), and this guide assumes it. It does not cover the Antithesis product, its pricing, its market or its company — that is [antithesis_guide.md](antithesis_guide.md). It does not cover the determinism taxonomy in general (ambient locale, map iteration order, unordered collections), which the first guide's §3 owns and which is a *prerequisite* discipline rather than a DST-specific one: you cannot simulate a system that is nondeterministic for removable reasons, because the simulator's replay guarantee would be destroyed by a source of variation no seed can control. It does not cover record-and-replay debugging tools (that guide's §4) or durable-execution workflow replay (its §6), which are determinism techniques aimed at *reproducing a past run*, not at *searching a modelled environment*. And it does not cover chaos engineering, which is this guide's complement rather than its competitor — see §13.1 and [chaos_engineering_guide.md](chaos_engineering_guide.md).

---

## 2. The Architectural Inversion

### 2.1 What the Inversion Actually Is

Ordinary testing wraps the system: a harness starts it, feeds it inputs, observes outputs, and the environment — the operating system, the clock, the network, the disk — is taken as given. DST inverts that relationship. Instead of the test living inside the environment and observing the system, the **system lives inside the test** and the environment is supplied by it.

The Strange Loop 2014 talk abstract states the inversion precisely: deterministic simulation is where "the hardware components of the system — including hard disks, network links, and the machines themselves — are replaced in testing with software which fulfills the contracts of those systems, but whose state is completely transparent to the developer" ([thestrangeloop.com](https://thestrangeloop.com/2014/testing-distributed-systems-w-slash-deterministic-simulation.html), conference talk abstract, checked 2026-09-22). Two things are doing work in that sentence. "Fulfills the contracts" is the fidelity requirement — the simulated disk must behave like a disk as the *system* understands it, not as a mock understands it. "Completely transparent" is the capability requirement — the simulator can look inside a disk or a network link at any instant, which is exactly what a real one never permits. §5.3 and §6.4 build on that second clause; it is the most under-appreciated reason to do this work at all.

Notice what is *not* replaced. The system's own logic — the consensus protocol, the ledger, the recovery path — is the production code, compiled and run unmodified in the simulated configuration. A FoundationDB release is described as having simulation "enabled by and tightly integrated with Flow, our programming language for actor-based concurrency", with Flow both "generat[ing] efficient production code" and working "with Simulation for simulated execution" ([FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22). One codebase, two execution contexts — that is the whole design goal.

### 2.2 What the Inversion Forces: Entry Point, Time, Lifecycle

Three things change shape as soon as the environment is injected.

**The entry point becomes an argument, not a fact.** A production `main()` acquires its world from the OS: it opens sockets, reads configuration from files and environment variables, gets the current time, spawns threads. None of that can remain. In a simulatable system the composition root takes an *environment* (or a bundle of interfaces: clock, network, storage, entropy, scheduler) and every subsystem receives its dependencies from it. The practical test is blunt: **if you cannot construct the system twice in the same process, with two different environments, and have them not interfere, you have not made the inversion.** That single property explains why hidden global state is fatal — a global connection pool, a global logger with its own timer, a lazy-initialised singleton clock — and why the FoundationDB documentation describes the cluster as running inside *a process the simulator owns* rather than as a program the simulator merely starts.

**Time stops being ambient.** Every timeout, every lease, every retry backoff, and every "how long has this been down" question must read a clock drawn from the environment. §4 is devoted to this, but the architectural consequence belongs here: the system's runtime becomes a *state machine over a virtual timeline* rather than a sequence of blocking waits. In FoundationDB's case the language exists partly to force this — Flow "is implemented as a compiler which analyzes an asynchronous function (actor) and rewrites it as an object with many different sub-functions that use callbacks to avoid blocking" ([flow.html](https://apple.github.io/foundationdb/flow.html), checked 2026-09-22). Non-blocking is not a performance choice in this design; it is what lets the simulator advance time when all tasks are waiting.

**Lifecycle becomes explicit and observable.** Machines come up, go down, come back, and are wrong about state when they return. That is not something you bolt on to a `main()`; it is a lifecycle the simulator drives. FoundationDB states it simulates "failures modes at the network, machine, and datacenter levels, including connection failures, degradation of machine performance, machine shutdowns or reboots, machines coming back from the dead, etc." ([testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22) — "machines coming back from the dead" is the phrase worth dwelling on, because it is the state that a test suite built out of fixtures and teardowns handles worst.

### 2.3 Why Determinism Pushes Toward a Single-Threaded Executor

The reason is not stylistic. **If two pieces of the system run at genuinely the same instant, then the order in which their effects become visible is decided by the OS scheduler and the hardware, and no seed can control either.** Determinism therefore requires that exactly one thing be runnable at a time and that the simulator decides which.

FoundationDB's documentation states the result without hedging: Simulation "is able to conduct a *deterministic* simulation of an entire FoundationDB cluster within a single-threaded process. Determinism is crucial in that it allows perfect repeatability of a simulated run, facilitating controlled experiments to home in on issues" ([testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22). "An entire cluster within a single-threaded process" is the load-bearing claim in this whole guide: not one server, not one node, a cluster.

The interface that makes this work is cooperative: Flow actors wait on futures `wait()` "allow[s] the calling actor to pause execution until the value of the future is set", during which "other actors can continue execution, providing asynchronous concurrency within a single process", and `choose … when` "allow[s] an actor to wait for multiple futures at once in a ordered and predictable way" ([flow.html](https://apple.github.io/foundationdb/flow.html), checked 2026-09-22). Held against the single-threaded claim, the phrase "ordered and predictable" is a specification of the executor, not an aspiration about the code.

### 2.4 What a Single-Threaded Deterministic Executor Forbids

Stated as a list, because each item is a class of bug or a class of technique that is now out of reach by construction. The first four follow directly from the documented single-threaded, non-blocking, ordered executor; the fifth is an inference from it and is labelled as such.

| Forbidden | Why | Consequence for the test suite |
|---|---|---|
| **Real parallelism inside the simulated system** | Two truly concurrent effects cannot be ordered by a seed | You cannot rely on a data race to surface as a failure; races must be *modelled* as interleavings the executor chooses |
| **Real OS scheduling as a source of variation** | The executor, not the kernel, decides what runs next | You cannot test the kernel's scheduling decisions, preemption, priority inversion or CPU affinity effects |
| **Blocking calls in the deterministic path** | A blocked thread stalls the whole simulation; the callback rewrite exists to avoid blocking ([flow.html](https://apple.github.io/foundationdb/flow.html)) | Every dependency that can block — a synchronous driver, a blocking syscall, a file read — must be behind the boundary, or it degrades the simulation to real-time |
| **Unsequenced completion callbacks** | Completion order must be produced by the executor's queue | Callback-style code whose ordering depends on a thread pool inverts the guarantee the whole exercise bought |
| **Blocking on a real lock as a source of variation** *(inference, not a documented claim)* | If the executor serialises tasks, a mutex is uncontended in the simulated run unless the simulator injects contention | Lock-contention and deadlock bugs are **not found in the same way** — see §2.5 |

### 2.5 Concurrency Is Modelled, Not Experienced

This is the sentence to take away from §2: **in a deterministic simulator the system's concurrency is modelled rather than experienced.** The code may still be *written* concurrently — FoundationDB's is actor-based, with promises, futures and streams traversing the network ([flow.html](https://apple.github.io/foundationdb/flow.html)) — but at execution time the executor serialises it and the simulator chooses the interleaving. The fine distinction matters and is routinely flattened in secondary write-ups:

- The *programming model* stays concurrent and message-passing; the *execution model* is one-runnable-task-at-a-time.
- Therefore an ordering bug is still reachable — indeed the simulator deliberately searches orderings — but only orderings the executor's model can express. A pathological OS-level interleaving that depends on two CPUs touching the same cache line is not in the search space.
- Therefore lock-contention, deadlock and priority-inversion bugs must be **modelled explicitly**: the simulator injects contention, delays a holder, delays a waiter, or asserts a lock-ordering invariant. They do not arrive for free, and a team that assumes "the simulator will find the deadlock" has misunderstood what was built.

### 2.6 Implementations Differ Here — This Is Not One Design

It is tempting to describe "the" DST architecture. There are at least three distinct positions on *where determinism is imposed*, and they have very different costs:

| Position | Where the knife goes | Examples | Consequence |
|---|---|---|---|
| **Boundary built into the application** | The system is restructured so all I/O goes through interfaces the simulator implements | FoundationDB (with Flow); TigerBeetle (Zig, "all non-deterministic parts of the system are stubbed out. This includes the clock, network, and disk operations" — [docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22) | Highest fidelity and control, highest rewrite cost, total dependence on developer discipline |
| **Determinism imposed below the application** | The application is unchanged; an interposer or hypervisor owns the machine's I/O and scheduling | Shadow (syscall interposition, per the landscape table in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.3); Antithesis's hypervisor, which per the same table runs the whole system on a single CPU core inside a custom hypervisor | No rewrite of the system under test; less control over what "time" means inside the application; the tooling is a platform, not a library (product details: [antithesis_guide.md](antithesis_guide.md)) |
| **Determinism confined to a component or test harness** | A library or executor makes one component deterministically testable | The general property-testing ecosystem; the durable-execution engines in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §6 | Cheap to adopt; cannot reproduce a *cluster* interleaving, because it does not own the environment the cluster shares |

The positions are complements, not competitors, and they are not equally suited to a given team. The middle column is what a bank buys when it has no appetite for rewriting a core system; the left column is what a database team builds when the system *is* the product. §13 returns to this.

### 2.7 The Honest Note on Cost

The restructuring is the expensive part, and it does not reverse for free. Once the system composes its world from injected interfaces, every subsequent feature pays a small tax — a new dependency must be expressed as an interface, a new timer must go through the clock, a new metric exporter must not touch the disk on its own — and a new engineer must learn the rule or silently violate it. There is no "we'll add DST later if we need it" path that avoids this: the interface design is the investment, and it is incurred whether or not the simulator is ever written. What you get for it is a system whose environment is a parameter — and, as §11 argues, that is a second engineering project with a maintenance schedule attached.

---

## 3. The I/O Boundary

### 3.1 Why One Boundary, and Why the Count Matters

Everything the system touches from outside must pass through one narrow interface. The reason is not tidiness. The simulator's reproducibility guarantee is a claim that **the system's entire input set is generated by the simulator**; every read of the outside world that does not pass through the boundary is an input the seed does not control, and therefore a hole in the guarantee.

This is why the sibling guide's first requirement — "a single narrow I/O boundary" — is listed first and in dependency order: with two boundaries, determinism holds only if both are driven from the same seed and the same logical clock, which is a strictly harder invariant to maintain than one. And it is why the count, not the elegance, is the metric that matters at review time. A codebase where time is read in fifty places cannot be simulated, no matter how good each individual call site looks.

FoundationDB's architecture shows the boundary at the language level: Flow actors "receive asynchronous messages from each other using a data type called a *future*", promises and futures "can traverse the network", and "many of the servers in FoundationDB expose their interfaces as a `struct` of promise streams — one for each request type" ([flow.html](https://apple.github.io/foundationdb/flow.html), checked 2026-09-22). Note the shape: the system's interface to everything external is a *typed set of streams*, which is exactly what an interface you can implement twice looks like.

TigerBeetle states the discipline as an inventory rather than a principle: "In the simulator, all non-deterministic parts of the system are stubbed out. This includes the clock, network, and disk operations." ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22). Three nouns — clock, network, disk. That is the boundary's minimum viable contents, and a useful first audit checklist for any team attempting this.

### 3.2 What Belongs at the Boundary

| Capability | Why it must be injected | Typical operations |
|---|---|---|
| **Time** | Ambient time makes the run unrepeatable and prevents time from being advanced | Read monotonic time; read wall time; arm a timer; sleep |
| **Network / messaging** | Delivery, delay, loss, duplication, reorder and partition are the fault surface | Send to a peer; receive; connect; accept; resolve a name |
| **Storage** | Durability, ordering and corruption are the fault surface; the crash model lives here | Open, read, append, flush/fsync, truncate, close |
| **Entropy** | A draw from the OS's RNG is unseeded variation by definition | Generate a random value; derive a key; get a UUID |
| **Environment / configuration** | Ambient reads make behaviour depend on the host, not the revision | Read a config value; read an environment variable; read a hostname |
| **Process lifecycle** | Crash, restart and "coming back from the dead" must be drivable | Kill self; begin startup; report readiness |
| **Introducing nondeterminism you accept** | Every permitted source should be *named* at the boundary even when it is not simulated, so the count is auditable | Thread spawn; real signal handling |
| **Observation** | Logging, metrics and tracing must not regress the simulation into real-time or real-I/O | Emit a log line; record a metric |

### 3.3 What Does *Not* Belong at the Boundary

The greater risk is over-broadening. Each of these is tempting and each is a mistake:

- **Business logic.** A boundary method named `postLedgerEntry()` is not I/O; it is the system. A boundary that knows the domain cannot be implemented by a simple simulator, and it stops the simulator from being able to say "the disk was slow" independently of what the code was doing.
- **Serialisation format.** Keep the boundary at the level of "here are the bytes". A boundary that accepts a typed domain object has hidden the wire format inside itself, which means the simulator cannot corrupt it — and corrupting messages is one of the highest-value faults (§8).
- **Retry policy.** Retries are system behaviour, and they are exactly the thing you want to exercise under compressed time. A boundary that retries internally has taken a testable behaviour and made it invisible.
- **Caching.** A cache in front of the boundary is a deterministic *bug surface* (staleness, invalidation order, eviction under pressure) that becomes invisible if the simulator cannot see the misses.
- **Scheduling of the system's own tasks.** The executor decides what runs next; the boundary only knows about external events.

### 3.4 One Interface, Two Implementations

The construction rule is a sentence: **define the interface, write two implementations (real and simulated), and forbid a third.** The properties that make it work are less obvious than the rule.

- **No hidden global state.** If the boundary is reachable via a global singleton, then two simulated clusters in one process share it, and §2.2's construction test fails. The mocked-up version of the anti-pattern is a `Time.now()` static that the test overrides — override ordering, not determinism, is then the thing under test.
- **No ambient reads.** The boundary is the *only* place `getenv`, `gettimeofday`, `rand`, `open` or `connect` may appear. Enforce it mechanically if you can: a lint rule, a module boundary, a CI grep. A rule enforced by culture alone decays — and it decays quietly, because the violation is invisible until reproducibility breaks months later (§7.3).
- **Configuration is an input, not an ambient fact.** Pass the config in. A simulator run must not depend on a machine's locale, DNS or `TZ` — those are the removably-nondeterministic sources the first guide's taxonomy already catalogues, and they defeat replay just as surely as an unseeded random draw.
- **The simulated implementation is written to be inspectable.** It should expose its internal state to the simulator (in-flight messages, pending writes, unflushed data) rather than pretending to be opaque. This is the "completely transparent to the developer" clause of §2.1, and it is the reason the exercise pays for itself.
- **Fail loudly on the unimplemented.** A simulated boundary that silently no-ops an operation it does not model converts a modelled-absence into a false pass. Prefer a hard failure on an unimplemented simulated operation over a silent success.

```
# ILLUSTRATIVE PSEUDOCODE — not a real API, not any project's actual interface.
interface Environment {
    clock:      Clock        # now(), sleep(d), set_timer(at, cb)
    net:        Network      # send(peer, bytes), receive()->(peer, bytes), connect, partition(a,b,on/off)
    disk:       Disk         # read(file,off,len)->bytes, write(file,off,bytes), flush(file), crash()
    entropy:    Rng          # u64(), bounded(n), derived_substream(label)  -- ALL seeded
    lifecycle:  Lifecycle    # kill_self(), reboot(), now_ready()
    observe:    Observer     # log(msg), metric(name, value)
}
# Two implementations: RealEnvironment (OS-backed) and SimEnvironment (simulator-backed).
# A third implementation is a bug: it means someone needed an exception to the boundary.
```

### 3.5 When You Do Not Control a Dependency — The Point Where Adoption Actually Fails

This is the single most common reason a DST adoption attempt fails, and it is worth being blunt about the shape of the failure. The team does §2 and §3 correctly for the code they own, then discovers that a *dependency* owns a channel to the outside world: a third-party database client that opens its own sockets, a TLS or DNS library, a managed message broker's driver with its own reconnection thread, a syscall nobody surfaced, an HSM client, a metrics agent that binds a port on import. The simulator cannot see any of it, the run is no longer deterministic, and the project stalls at the boundary rather than at the architecture.

The patterns that work, roughly in order of preference:

| Pattern | What it does | Cost / limit |
|---|---|---|
| **Replace the dependency behind the boundary** | Put the capability behind your interface and write a simulated implementation, so the library is only used by the *real* implementation | Best outcome; requires the capability to be expressible in your interface; may be impossible for a library that fuses protocol with I/O |
| **Interpose below the system** | Let a tool own the syscall layer so the dependency's real I/O becomes simulated automatically (the Shadow-style position, §2.6) | No application rewrite; you inherit the interposer's coverage and compatibility surface, and you cannot inject *application-level* faults |
| **Embed a model of the dependency** | Write a simulated peer that speaks the reduction of the protocol you actually rely on — acks, ordering, duplication, failure modes | Undermines "real code against simulated environment" for that dependency only; the model can drift from the real behaviour, and this is a fault catalogue gap (§8.4) |
| **Narrow the dependency to one call site** | If you cannot eliminate it, confine it so the nondeterminism it introduces is at least countable and its blast radius known | Acknowledged limitation, not a solution; must be recorded in the honesty inventory |
| **Exclude it from the simulation and test it another way** | Accept that a component is outside the modelled world and verify it with a different technique — black-box testing against the real deployment (§13.2), or formal specification | The honest choice; requires somebody to actually do the other technique (§14.7 shows this decision being made) |

Two anti-patterns are common enough to name. The first is **threading real time through a "temporary" escape hatch** — "this library sleeps 50 ms, so the simulator just uses real sleep here". One real sleep makes the run wall-clock bound and breaks the ordering guarantee; it is the leak described in §15.1. The second is **a simulated dependency that is not a model but a wish** — a fake broker that always acks, always orders, and never duplicates. That is a mock wearing a simulator's clothes, and it produces passing runs that prove nothing about the cases you were worried about.

The practical rule: **do the dependency inventory before the architecture, not after.** List every channel to the outside world the system has, including those owned by libraries and by the runtime (DNS, TLS session cache, entropy, logging sinks, thread pools), and classify each as *behind the boundary*, *below the boundary*, or *outside the model*. That list is also the honest-inventory artefact of §8.4 — the same document serves both purposes, which is a sign it is the right artefact.

---

## 4. Simulated Time

### 4.1 The Clock as a Controllable Input

Simulated time is the single highest-leverage piece of the construction, and the one whose meaning is most often overstated. The mechanism is simple: the system reads time from the injected clock, the simulator owns that clock, and when every task in the system is waiting on a timer, the simulator advances the clock to the earliest pending deadline and runs whatever was waiting. No real time passes. From the system's perspective, a forty-second lease expired; from the machine's perspective, nothing happened at all.

FoundationDB's documentation states both the mechanism and the reason: "The simulation steps through time, synchronized across the system, representing a larger amount of real time in a smaller amount of simulated time. In practice, our simulations usually have about a 10-1 factor of real-to-simulated time, which is advantageous for the efficiency of testing." ([FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22). **That 10:1 figure is the FoundationDB project's own estimate of its own ratio, not an independently measured benchmark, and it should be quoted that way.** It is also the only ratio on that page — the same page's "one trillion CPU-hours" is likewise a project estimate and is already flagged in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.

A second data point, labelled the same way: TigerBeetle's VOPR documentation says "Crucially, VOPR can speed up time arbitrarily. One minute of VOPR time is equivalent to days of real-world testing." ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22) — **project claim, no measured multiple given, and deliberately not converted into one here.** Note the projects differ in how they describe this: FoundationDB gives a ratio ("about a 10-1 factor") while TigerBeetle claims a capability ("arbitrarily"). The capability claim is the stronger and the less falsifiable of the two; treat ratios with a source and treat capabilities as marketing unless you measure them yourself.

### 4.2 What "the Sleep Is Instant" Actually Buys

It buys the ability to *exercise logic that is about time*, which is otherwise the most expensive bug class to reach in a real test environment:

| Behaviour | What compressed time lets you do | What it does not tell you |
|---|---|---|
| **Timeouts** | Fire a timeout deterministically at the boundary of its window, and again one tick later | Whether the timeout value is achievable on real hardware |
| **Leases and lock expiry** | Take a lease, lose the holder, and observe expiry, re-acquisition and the stale-holder's subsequent behaviour within one run | Whether clock drift between real nodes would have changed the outcome (§4.5) |
| **TTLs and reaping** | Age data past its TTL in seconds of wall time | Whether the reaper's throughput keeps up with production volume |
| **Retry backoff** | Walk an entire exponential backoff ladder, including jitter sequences and the exhaustion path, in one second | Whether the backoff's wall-clock shape overloads a recovering dependency |
| **Recovery windows** | Simulate a node down for "an hour" and check what its peers did | Anything about the durability of real storage under that hour |

The last column is the point of the next two subsections, and it is why the honest phrasing is **"we can reach time-dependent states far faster than reality", not "the code is fast".**

### 4.3 Monotonic vs Wall Clock Inside a Simulator

Two clocks exist in most systems and behave differently, and a simulator must decide what each means.

- **Monotonic time** — used for durations, timeouts, leases and ordering. This is the clock the simulator genuinely owns: it is a counter, it never goes backwards, and advancing it is the whole trick.
- **Wall time** — used for timestamps that humans, other systems or regulators will read. In a simulator, wall time is either (a) derived from simulated time by an offset from a fixed epoch, so runs are comparable, or (b) left as a real value, in which case it is an unseeded input and must be called out as such.

The discipline is to make the choice explicit at the boundary rather than letting each call site decide. The two failure modes are symmetric and both are common. Mixing the clocks — computing a duration by subtracting a real wall-clock reading from a simulated monotonic one — produces negative or absurd durations that only appear in simulated runs. Freezing wall time where the system genuinely means "the current real date" (a certificate validity window, a settlement cut-off date) produces a system that is confidently wrong in production while passing every simulation. Neither is a simulator bug; both are boundary-design bugs, and both are cheap to prevent with one rule: **every time-typed value in a signature says which clock it came from.**

### 4.4 An Illustrative Shape

The boundary's clock is small, and it is worth seeing how little it needs to carry — the difficulty is never the interface, it is the discipline of using only this and nothing else:

```
# ILLUSTRATIVE PSEUDOCODE — not a real API.
interface Clock:
    monotonic_ns() -> u64                 # the simulated timeline
    wall_ns()      -> u64                 # derived from an epoch offset, or flagged as real
    sleep(d)       -> void                # yields to the executor; simulator advances time
    set_timer(at, cb) -> TimerHandle      # fires when simulated time reaches 'at'
    cancel(h)      -> void
# The simulated implementation keeps a priority queue of deadlines.
# Advancing time = pop the earliest deadline, set now = that deadline, run its callback.
# There is no other way for time to pass.
```

### 4.5 What Time Compression Does **Not** Preserve

This is the crux of the section, and the reason a compressed-time test suite is not a performance test. When the simulator advances a counter, the following classes of behaviour are unaffected — and therefore untested:

- **Real network RTT distributions.** Simulated latencies are the ones you modelled. Real wide-area latency is multimodal, correlated with congestion and route changes, and occasionally has a 400 ms tail nobody wrote down.
- **Cache and memory-hierarchy effects.** Contention on a shared cache line, NUMA distance, TLB pressure and memory bandwidth all depend on real execution.
- **Garbage-collection pauses.** A managed runtime's pause profile — and therefore every timing assumption built on top of tail latency — is a property of the heap, not of the simulated timeline.
- **CPU starvation, scheduling and oversubscription.** The executor decides what runs; a real machine under load does not.
- **Thermal and turbo effects.** Clock frequency is a physical fact and has no place in a virtual timeline.
- **True lock contention and hardware-level races.** §2.5: modelled, not experienced.
- **Cryptography and I/O throughput.** Signature verification and disk flushing take real microseconds; compressed time does not make them cheap in practice (they still consume real wall time in your CI job) and does not make them slow in the model either.

The general statement, which is worth committing to memory because it defuses a whole family of false claims: **the simulator advancing time is not the same as the code being fast, and simulated "time" is an accounting dimension rather than a measurement.** You may legitimately claim "this invariant held across a simulated year"; you may not claim "this system handles a year of load per minute" without a real measurement, because the simulated year contained no real queues, no real bytes on a wire and no real CPU.

---

## 5. The Simulated Network

### 5.1 The Network as an Object the Simulator Owns

In a real system the network is a medium the system uses. In a simulator it is a data structure the *simulator* owns: a set of in-flight messages, each with a source, a destination, a payload and a notional delivery time. The system's `send` deposits a message; the system's receive path is driven by the simulator deciding when to deliver it. FoundationDB's documentation states the capability in one line: "Simulation also models the network, allowing a small amount of code to specify delivery of packets." ([testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22)

That "small amount of code" phrasing is the key architectural observation. The network model is *composable from the simulator's side* — new fault patterns are written as code against the simulated network, not as configuration of a middleware component. It is why the same team was able to hold an internal competition for the nastiest fault schedule (§5.3) and why the model could evolve faster than the system it tests.

### 5.2 The Parameters

| Parameter | What it models | What it actually tests |
|---|---|---|
| **Delivery** | Whether a message arrives at all (and whether the ack does) | Retry correctness, idempotency, at-least-once handling |
| **Delay** | Arrival time relative to send time, per message | Timeouts firing before/after replies; request/response reordering |
| **Duplication** | The same message delivered more than once | Idempotence; deduplication state; double-application of effects |
| **Reordering** | Arrival order differing from send order | Ordering assumptions; sequence-number gaps; staleness checks |
| **Partitioning** | Symmetric or asymmetric reachability changes between two endpoints | Quorum logic, view changes, split-brain avoidance — and, per §8.3, liveness |
| **Loss** | Silent drop with no error signalled | The hardest networking failure, because nothing tells the sender |
| **Corruption** | Payload altered in flight | Checksums, validation, and whether a bad value can be *used* |

The distinction between symmetric and asymmetric partition matters more than it looks, and there is a first-party account of why. TigerBeetle's own engineering blog describes splitting its simulator into "safety" and "liveness" modes precisely because **uniform probabilistic fault injection could not find a liveness bug**: "in safety mode, the simulator is guaranteed to restart each replica eventually, so that it might not notice a problem where a replica gets stuck until reboot", and a probabilistic asymmetric partition "would not have been permanent — at some point it would have healed and the vicious view change cycle would be broken (and undetected)" ([tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness), project blog, checked 2026-09-22 — *project account of its own simulator*). The construction lesson generalises: **a fault that is merely probable is a fault that will stop being present, and some bugs require the fault to be permanent.** §8.3 returns to this.

### 5.3 Fault Schedules: The Documented Example

FoundationDB's documentation describes both the general approach and a specific, named, prize-winning fault pattern. The general approach: "We use Simulation to simulate failures modes at the network, machine, and datacenter levels, including connection failures, degradation of machine performance, machine shutdowns or reboots, machines coming back from the dead, etc. We stress-test all of these failure modes, failing machines at very short intervals, inducing unusually severe loads, and delaying communications channels." And then the competition: "For a while, there was an informal competition within the engineering team to design failures that found the toughest bugs and issues the most easily. After a period of one-upsmanship, the reigning champion is called 'swizzle-clogging'. To swizzle-clog, you first pick a random subset of nodes in the cluster. Then, you 'clog' (stop) each of their network connections one by one over a few seconds. Finally, you unclog them in a random order, again one by one, until they are all up. This pattern seems to be particularly good at finding deep issues that only happen in the rarest real-world cases." ([testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22 — first-party project documentation, and note that "seems to be particularly good" is the project's own assessment, not a measurement)

Two things are worth extracting from that example. First, the fault is *structured and ordered* — a sequence, a duration, a random subset, a random order — not a per-message coin flip. It is a schedule, and the simulator is where schedules live. Second, the pattern is the kind of thing that arises from human imagination about real networks, not from a taxonomy. A fault catalogue is a starting point; the valuable faults are usually invented by someone who has been paged at 03:00 by a specific bizarre network state.

### 5.4 Inspectability: The Capability a Real Network Never Gives You

This is the advantage of the simulated network that is least often mentioned and disproportionately valuable. Because the network is a data structure the simulator owns, the simulator can — at any point in simulated time — enumerate the messages currently in flight, inspect their contents, delay one that was about to arrive, deliver one that was not due yet, or assert a property over the whole in-flight set. Real testing cannot do any of these things: on a real network you can observe traffic (see the network telemetry discussion in [network_packet_capture_guide.md](network_packet_capture_guide.md)) but you cannot *reach in and change what is about to happen* while also controlling everything else the system sees.

The Strange Loop abstract frames the same capability in general terms: the simulated hardware's "state is completely transparent to the developer" ([thestrangeloop.com](https://thestrangeloop.com/2014/testing-distributed-systems-w-slash-deterministic-simulation.html), talk abstract, checked 2026-09-22). It is also an underused one: a team that builds an inspectable simulated network and then only uses it to drop packets has built a hypothesis-testing instrument and is using it as a randomiser. Assertions over in-flight state — "no two replicas ever believe they hold the same lease", "no message is delivered after its sender was told the send failed", "the in-flight set never exceeds the modelled queue capacity" — are where the instrument earns its cost.

### 5.5 How Latency Distributions Are Expressed

The engineering question is: what shape do you give a message's delay? The constructions that appear in practice, in increasing order of sophistication:

1. **Constant plus jitter** — a fixed base delay with an additive random term, drawn from the seeded RNG. Cheap, and adequate for finding ordering and timeout bugs.
2. **Per-link or per-pair distributions** — separate parameters for intra-zone, cross-zone and cross-region links, so a partition that *should* be survivable is not made trivially survivable by uniform latency.
3. **Heavy-tailed and multimodal draws** — an occasional large delay to model a retransmission or a route change, which is how real wide-area latency actually looks.
4. **Correlated latency** — a shared underlying state (a "congested link", a loaded machine) that raises delays for many messages at once, which is closer to how real congestion behaves than independent per-message draws.
5. **Delay as a function of queue occupancy** — the simulated network models its own capacity, so delay grows as it fills and drops occur when it overflows.

Only the first is straightforward, and this is a place to be honest: **none of the sources consulted for this guide publishes the exact latency-distribution function its simulator uses.** FoundationDB documents *that* the network is modelled and that code specifies packet delivery; the probability distribution and per-pair parameterisation are implementation details that are not in the public documentation (recorded as an open verification item in §16). If you need a distribution that resembles your production network, measure it, and treat the model as an artefact with an owner.

### 5.6 The Honest Note

**The network you modelled is the network you test.** A simulator whose network is a constant 1 ms delay with 1% loss is a simulator whose verdicts are about that network. Two failure modes follow, and they are not symmetric. If the model is *kinder* than reality — no reordering, no asymmetric partitions, no long tails — the simulator will pass systems that fail in production, and no amount of seeds will fix it, because the fault is not in the search space. If the model is *harsher* than reality, the simulator will burn engineering time on states that cannot occur, and the team will start ignoring its verdicts, which is worse. Both are covered by the reviewer's question in §8.4: *show me the network model, and show me the production measurement it came from.*

---

## 6. Simulated Storage and the Crash Model

### 6.1 Why Storage Is Where Corruption Bugs Live

A network bug usually produces a wrong answer that is visible in the response. A storage bug produces a wrong answer that is *written down*, survives restart, and is discovered by an audit six weeks later. Storage faults also interact with the crash model, which means the interesting cases require the process to die at an arbitrary instant and come back — an operation a real test suite can barely arrange and a simulator can arrange thousands of times a minute. This is the section that finds corruption bugs, and it is the one whose absence in a "DST" implementation should make you doubt that implementation.

### 6.2 What the Filesystem Interface Must Expose

For a crash to mean anything, the simulated filesystem must model more than bytes-at-a-path. The minimum:

| Capability | Why the crash model needs it |
|---|---|
| **Read / write at an offset** | The system's actual durability logic is usually offset-based (log segments, pages) |
| **Flush / fsync as an observable operation** | Durability is defined by *what was flushed before the crash*, not by what was written |
| **Write ordering** | Two writes without an intervening flush may land in either order after a crash; modelling this is how you find "we assumed the log was durable before the index" bugs |
| **Truncation and partial writes** | A crash can leave a record half-written; the system must detect and discard it |
| **Arbitrary stop point** | The simulator can crash the process at *any* point in the operation stream, not only at operation boundaries |
| **Capacity and full-disk conditions** | Disk-full is a documented modelled failure in FoundationDB: Simulation "models drive performance on each machine, including drive space and the possibility of the drive filling up" ([testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22) |
| **Latency and degradation** | Slow storage changes timeout and election outcomes; FoundationDB likewise documents modelling drive performance |
| **Corruption of reads and writes** | TigerBeetle documents this fault class in its simulator: the VOPR "may drop and reorder packets, partition the network, or corrupt reads and writes to the 'disk'" ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22) |

### 6.3 Clean Shutdown Versus Crash

The distinction is the whole point of the section, and a test suite that only ever exercises one of them is testing half the durability story:

- **Clean shutdown** — the system is told to stop, flushes what it chooses, releases resources, exits. It tests that the *shutdown path* is correct and that a restart from a flushed state works.
- **Crash** — the process ceases at an arbitrary instant with no notification. Everything unflushed is lost; anything partially written may be torn. It tests whether the system's *invariant-detection and recovery* logic is correct, which is the property you actually care about.

A crash is the interesting case because it is the only one that can produce a state the system's authors did not anticipate. The construction requirement follows: the simulator must be able to kill a task *between* any two boundary operations, and the system must be able to start up from whatever the simulated disk contains. If the only way to stop the system is a graceful signal, the simulator cannot test crash recovery at all — it can only test restart.

**What a torn/partial write models.** A torn write is a write that landed incompletely: the first part of the bytes reached the media and the remainder did not, usually at a device block boundary rather than a byte boundary, so the result is a prefix (or a mix of old and new content), not a clean gap. It is the model of the case where the storage system does not guarantee atomicity for a multi-block write. Systems that assume "either the whole record is there or none of it is" are exactly wrong about real hardware, and the simulated torn write is how you find out. The related model worth having is a *lost but acknowledged* write: the write returned success and was not durable because someone forgot the flush. Both are found by a crash-and-restart test, and neither is found by any test that does not crash.

### 6.4 The Durability Contract Must Be Stated Before It Can Be Tested

A crash test has no oracle unless the system declares what survives. The contract is usually three sentences, and stating it is a design exercise that pays for itself independently of the simulator:

1. **What is guaranteed after a flush returns.** Typically: "after `flush(f)` returns successfully, every byte written to `f` before the call is durable and will be visible after a crash."
2. **What may be lost without a flush.** Typically: "unflushed writes may be lost, reordered or partially present after a crash; the system must not depend on them."
3. **What the system does about a violation.** Typically: "on startup, a partially written record is detected by its checksum and truncated; a record whose checksum passes is used."

Two of the three statements are testable by the simulator directly, and the third is the recovery logic you assert on after restart. Without a contract, a crash test can only ask "did it start?" — a question every implementation passes.

### 6.5 The Model-Based Storage Checker

The approach that catches corruption is a **model-based checker**: a simple, obviously correct reference implementation (or reference invariant) run in parallel with the real one, asserting that the two agree on observable behaviour. Where the reference model is a simple in-memory map and the real thing is a replicated, crashing, flushing log, "they agree on the observable contents" is a strong property that no per-operation test expresses.

TigerBeetle documents a concrete instance of exactly this shape. Its VOPR documentation says: "On top of the assertions in the code, the simulator also includes a variety of additional checkers that verify the correctness of the cluster's state. For example, TigerBeetle replicas' data files are designed to be byte-for-byte identical across caught-up nodes in the cluster. Some of the storage checkers verify that this is the case across simulations." ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22 — first-party project documentation of its own checkers). That check is a *replica-agreement* checker rather than a reference-model comparison, and it is worth naming both because they are different instruments:

| Checker shape | What it asserts | What it finds |
|---|---|---|
| **Replica / node agreement** | Two independently evolved states of the same logical data are identical (byte-for-byte, in TigerBeetle's documented case) | Divergence, non-determinism in the applied state, incomplete repair, split-brain residues |
| **Reference-model comparison** | The real implementation's observable behaviour matches a simple model's | Semantic bugs in the implementation's own logic — off-by-one, lost updates, wrong eviction |
| **Cross-check of independent computations** | Two different code paths computing the same quantity agree (a checksum recomputed from the log versus a counter maintained in memory) | Drift between maintained state and derivable state — a classic corruption symptom |
| **Invariant over the whole simulated cluster** | A global property holds across *all* nodes, not one (see §14.4) | Bugs invisible from any single node's perspective |

The general reference-model shape is the standard model-based-testing construction with a long lineage in property-based testing (QuickCheck and after; §9.3). Labelling is important here: **the "reference model run in parallel" phrasing is a generic technique, not a quotation from either project.** What is documented first-party is TigerBeetle's byte-for-byte data-file equivalence checkers and FoundationDB's disk models (drive performance, space, and the drive filling up). FoundationDB's public testing page does *not*, as far as this pass could verify, publish a torn-write or bit-corruption model for its simulated disk — recorded in §16 rather than asserted.

### 6.6 The Construction Consequence

The storage layer is where the boundary most often turns out to be too thin. A boundary of `read()` and `write()` is enough to simulate latency and loss but not crash semantics: to crash meaningfully you need flush visibility, write ordering, and a kill point that is not an operation boundary. Teams that build the network model first and the storage model second usually discover this late, and the fix is not a small one — it changes the interface the rest of the system is written against. Build the storage interface as if the crash model is the primary use case, because it is.

---

## 7. The Deterministic Scheduler and the Seed Discipline

### 7.1 Where Nondeterminism Can Enter

A deterministic simulator is a claim about a closed set of inputs. Enumerating the *openings* is therefore the primary design activity, and it is more productive to do it as an inventory than as a principle.

| Entry point | How variation gets in | Who must own it in a simulated run |
|---|---|---|
| **OS entropy / RNG** | A draw from the system's random source | The simulator (seeded RNG, §7.2) |
| **Clock reads** | Monotonic or wall time entering a decision | The simulator (simulated clock, §4) |
| **Network arrival order and timing** | The kernel's timing, not the system's logic | The simulator (simulated network, §5) |
| **Disk completion order** | Asynchronous I/O completion order | The simulator (simulated storage, §6) |
| **Task scheduling** | Which runnable task the runtime picks | The simulator's executor (§2.3) |
| **Unordered iteration** | Hash map/set iteration order; unordered query results | The **system**, by discipline — a removably-nondeterministic source ([deterministic_engineering_guide.md](deterministic_engineering_guide.md) §3). Not the simulator's job, and unfixable by seeding |
| **Uninitialised memory / addresses** | Reading padding or pointer values as data | The **system**, by discipline (and by memory safety) |
| **Locale, timezone, environment** | Ambient configuration changing behaviour per host | The **system**, by discipline — configuration must be an input (§3.4) |
| **Unordered callbacks / thread pools** | Completion ordering decided by a pool | The **system** must remove them from the deterministic path (§2.4) |
| **External processes** | A peer, a sidecar, a broker not inside the boundary | Out of scope — a gap in the model (§8.5) unless modelled |
| **Real concurrency inside the system** | Two effects visible simultaneously | Forbidden by construction (§2.4) |

The rule underneath the table is the one worth writing on a wall: **every source of variation must be either seeded or forbidden.** There is a third legitimate option — *captured*, i.e. recorded and replayed — but that is record-and-replay debugging, a different technique with a different purpose ([deterministic_engineering_guide.md](deterministic_engineering_guide.md) §4). What is *not* legitimate is a source of variation that is none of the three. That is the state of a codebase that "mostly" reproduces, which is the most expensive state to be in, because every failure report becomes uncertain.

### 7.2 How the Seed Is Plumbed Through the System

The mistake is to treat the seed as belonging to the test harness. It belongs to the *system under simulation*, and it must be an ordinary parameter of the system's construction. The discipline that holds in practice:

1. **The seed is a value passed at composition time**, alongside the environment: `Simulation.init(seed)` constructs the environment, and the environment constructs every subsystem with the random sources it is allowed to use.
2. **No global RNG.** There is no process-wide generator to call. The absence of the API is the enforcement mechanism; a global generator is silently shared state, and shared state is what two simulations in one process (§2.2) cannot survive.
3. **Sub-seeds are derived, deterministically, from the root seed.** Each subsystem gets its own stream — `rng_for("network")`, `rng_for("disk")`, `rng_for("fault_schedule")` — derived from the root seed by a stable, documented derivation. The reason is not elegance: **it makes the streams independent and stable.** If the network and the disk draw from one shared sequence, then a change that makes the network draw one extra value silently shifts every subsequent disk draw, and a seed that reproduced a bug yesterday stops reproducing it today for a reason nobody will enjoy debugging.
4. **The random source is a component with state, not a function.** It is passed explicitly, and it is inspectable: the simulator can report how many draws each stream has consumed, which is the first diagnostic when a run stops replaying.
5. **Only the injected randomness is "random".** System code must not call an RNG for anything behaviour-affecting — a key derivation, a jitter, a shard choice — except through the injected source. This is §3's boundary rule restated for entropy.

This plumbing is generic engineering discipline rather than a documented feature of either project, and it is labelled as such. What *is* documented is the seed's role at the small end: "The VOPR uses a random seed to tune parameters for injecting different types of faults into the simulation" ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22) — the seed governs the fault parameters, not merely test data. Neither FoundationDB's public testing page nor TigerBeetle's VOPR page publishes its internal RNG-derivation scheme (recorded in §16).

### 7.3 Why One Global Unseeded Random Call Is Enough to Destroy Replay

Be concrete, because the failure is not obvious from the phrase "nondeterminism". Suppose the seeded generator is a single stream shared by everything, and somewhere in a retry helper a developer writes `jitter = rand() % 100` against the *system* RNG instead of the injected one. Now trace it:

1. The stray call consumes one value from the shared stream (or, worse, from a genuinely unseeded source).
2. Every subsequent draw from that stream now returns the value that would have been drawn one position later.
3. The fault schedule derived from the stream is therefore different — a different packet is dropped, or the same packet is dropped at a different simulated instant.
4. A different fault at a different instant means a different execution path, which means a different number of draws, which means the streams diverge permanently.

The end state is the worst possible: the run no longer reproduces, so a real failure found under seed `S` is now unreproducible, and the bug is demoted from "found" to "rumoured". Two refinements make this worse than it first appears. First, the divergence may be silent and rare — if the stray call only occurs on a path that rare faults reach, most seeds replay fine and a few do not, and the team concludes the simulator is flaky rather than that a call site is wrong. Second, a stray call against a genuinely unseeded source is not even "shifted": it is a value no seed controls, so the *same* run can differ between two invocations with identical inputs. That is the difference between a bendable guarantee and a broken one.

The mitigation is mechanical, not cultural: make the unseeded API unreachable from the deterministic core. If the module graph does not permit the OS RNG inside the deterministic region, the class of bug is eliminated by construction rather than by review.

### 7.4 The Reproducible Failure Artifact

A run is reproducible when three things are pinned: **the seed, the code revision, and the configuration.** TigerBeetle states the first two as its unit of reproducibility: "Because our simulator is deterministic based on a *seed* number and the Git commit, we can perfectly reproduce any bugs discovered in testing for easy local debugging" ([docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22 — first-party project claim about its own simulator). The third — configuration — is the one that gets lost in practice, and it is not optional: node count, replication factor, cluster parameters, feature flags, workload shape, fault-probability settings and any simulated latency parameters are all inputs. A seed-and-commit pair with the wrong configuration reproduces a *different* run, and a team that has not recorded the configuration will spend a day discovering that the difference was in the parameters rather than in the code.

The practical artefact is therefore a triple, and the honest engineering decision is to make it impossible to run the simulator without producing it. A failing run should print one line that can be pasted into a bug report and pasted back into a command:

```
# ILLUSTRATIVE PSEUDOCODE — a failure artifact shape, not a real output format.
simulation failed
  seed:      0x5f3a...
  commit:    <git sha>
  config:    nodes=5, replicas=3, faults=<preset>, workload=ledger-ops
  assertion: cluster-level "sum(debits) == sum(credits)" at simulated t=...s
  replay:    ./sim --seed 0x5f3a... --config <config-id>
```

---

## 8. Fault Injection as a First-Class, Seeded Parameter

### 8.1 What §5 Already Established, and What This Section Adds

[deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.5's fourth requirement is that fault injection be a first-class, seeded parameter, quoting TigerBeetle's mechanism for it (the seed tunes fault parameters; on failure the seed and commit replay the exact simulation). This section is the *depth* behind that requirement: what to put in the catalogue, how the schedule is derived, and how to know what you left out.

### 8.2 The Catalogue: Fault Classes and What Each Actually Tests

| Fault class | Concrete injections | What it actually tests | Documented by |
|---|---|---|---|
| **Process crash and restart** | Kill at an arbitrary point; restart cold; restart with stale on-disk state; return after a long absence | Recovery logic, log replay, idempotency of recovery, re-election | TigerBeetle: crashes "a healthy replica", restarts "a downed replica" ([VOPR blog](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness), project blog) |
| **Message delay** | Per-message added latency; a reply delayed past its caller's timeout | Timeout sizing, ordering assumptions, stale-response handling | FoundationDB: "delaying communications channels" ([testing.html](https://apple.github.io/foundationdb/testing.html)) |
| **Message duplication** | Deliver a message twice; deliver a reply to a retried request twice | Idempotence and deduplication state | TigerBeetle documents drop/reorder ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)); duplication is the standard companion injection |
| **Reordering** | Deliver out of order within a link | Sequence handling, gap detection, repair triggering | TigerBeetle: VOPR "may drop and reorder packets" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)) |
| **Partitioning** | Symmetric or asymmetric; temporary or permanent; partial subsets | Quorum logic, view change, split-brain avoidance, **and liveness** | FoundationDB: "connection failures"; TigerBeetle: "partition the network", plus the permanence insight ([blog](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness)) |
| **Disk latency / degradation** | Slow reads and writes; degraded throughput for a window | Timeout interactions, election storms, throughput-dependent invariants | FoundationDB: "degradation of machine performance"; Simulation "models drive performance on each machine" ([testing.html](https://apple.github.io/foundationdb/testing.html)) |
| **Write corruption / torn writes** | Corrupt a read or a write; truncate a record; partial write at a block boundary | Checksums, validation, recovery, "we can trust this value" assumptions | TigerBeetle: VOPR "may … corrupt reads and writes to the 'disk'" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)) |
| **Disk full / space exhaustion** | Fill the drive; deny a write | Backpressure, admission control, graceful degradation | FoundationDB: "the possibility of the drive filling up" ([testing.html](https://apple.github.io/foundationdb/testing.html)) |
| **Machine shutdown / cold return** | Stop a machine; bring it back; return it after a long absence | Cold-start correctness, membership, resurrection handling | FoundationDB: "machine shutdowns or reboots, machines coming back from the dead" ([testing.html](https://apple.github.io/foundationdb/testing.html)) |
| **Clock skew between nodes** | Nodes whose clocks disagree; a clock that jumps forward or backward | Lease safety, timestamp-based ordering, monotonicity assumptions | **Not verified on either public documentation page consulted — flagged, §16.** A real gap in public documentation, not evidence that no project does it |
| **Resource exhaustion** | Fail an allocation; exhaust file descriptors; fill a bounded queue | Error paths, backpressure, whether "impossible" branches are implemented | Generic construction; not verified as a modelled fault in the public pages consulted (§16) |
| **Network flap** | Repeated short partitions; connection churn; clog-then-unclog cycles | Reconnection storms, retry amplification, leader churn | FoundationDB's *swizzle-clogging* is exactly this and won its fault-design competition ([testing.html](https://apple.github.io/foundationdb/testing.html)) |
| **Failure of the system's own dependencies** | Kill a peer, a coordinator, a metadata service | Dependency-loss handling, timeouts on a *synchronous* dependency, degraded modes | Generic construction; requires the dependency to be inside the model at all (§3.5) |

### 8.3 Deriving the Schedule From the Seed

The schedule must be a *function* of the seed, not merely a consumer of it. Two constructions are used in practice:

- **Scheduled up front.** The simulator expands the seed into an ordered list of `(simulated_time, fault, target)` events before the run starts. Advantages: the schedule is inspectable, printable, diffable and reviewable before it executes, and the fault stream cannot shift when the system's code path changes. Cost: the schedule cannot react to what the system does, so it cannot target the state that just became interesting.
- **Drawn lazily from a dedicated sub-stream.** The simulator draws fault decisions as simulated time advances, from a stream reserved for the purpose (§7.2, item 3). Advantage: it can be conditioned on the run — "after a node has been down for an hour, do X". Cost: the number of draws now depends on the execution path, so any behaviour change perturbs the schedule. That is acceptable when the revision changed, but it means schedules are not comparable across revisions.

TigerBeetle's documented construction sits in the second family: "Up to now, the simulator worked by injecting faults uniformly. That is, on every tick it rolled a dice and, with some probability, crashed a healthy replica, restarted a downed replica, dropped a packet from the network, or corrupted read or write disk I/O." ([tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness), project blog, checked 2026-09-22). The same blog documents why uniform probabilistic injection is not sufficient on its own, and what changed.

### 8.4 Fault Probability, and the Three Ways It Goes Wrong

Fault probability is a tuning problem with a narrow useful band, and both edges are bad. **Too low:** the run completes, nothing interesting happens, and the simulator does the job production already does at greater expense — the signal is a suite that has been green for months and has found nothing, which is not evidence of correctness (§9.4). **Too high:** the system never makes progress, so every run is spent in the "nearly everything is broken" branch; that branch is worth testing, but if it is all you test, you have tested one state thoroughly and missed everything between healthy and chaotic. **Too uniform, over time:** a fault that is merely probable is a fault that will heal, and some bugs require it to stay broken.

The third is the one with a first-party account. TigerBeetle's blog states that with uniformly injected faults, a probabilistic asymmetric partition "would not have been permanent — at some point it would have healed and the vicious view change cycle would be broken (and undetected)", and that "in safety mode, the simulator is guaranteed to restart each replica eventually, so that it might not notice a problem where a replica gets stuck until reboot". Their fix was to switch mid-run from safety mode to liveness mode: select a quorum to be fully available, heal all partitions within it, and **make every failure involving non-core replicas permanent** ([blog](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness), checked 2026-09-22 — project account of its own simulator).

Two consequences follow. First, **the useful design is a schedule with phases** — a phase that explores chaotic states probabilistically, followed by a phase that declares some set of things permanently true and requires the system to recover. Second, **per-run configuration randomisation is a real technique, not a gimmick**: rather than tuning to one "good" probability, randomise the parameters per run so different seeds explore different shapes of the fault space. The general form of this idea comes from the swarm-testing literature, which TigerBeetle's blog names among its major inspirations ([Swarm testing](https://users.cs.utah.edu/~regehr/papers/swarm12.pdf), Regehr et al. — general systems literature, checked 2026-09-22). This remains **OPEN practice**: no source consulted for this guide gives a defensible procedure for choosing the band, and the probabilities should be treated as an experiment the team owns rather than a setting copied from a blog post.

### 8.5 The Honest Inventory: What You Did Not Model

Every DST adoption produces a document more valuable than its own green runs: the list of things the simulator does not model. A reviewer's standard question should be *"show me that list"*, and a DST project without one should be treated as an unverified claim. A workable starting checklist, each line a candidate entry:

- **Real hardware** — device firmware behaviour, write caches that lie, silent bit rot, recoverable vs unrecoverable errors, disk-full semantics differing from the model.
- **Real timing** — CPU frequency scaling, NUMA placement, cache hierarchies, GC pauses, scheduling jitter (§4.5).
- **Real network** — the actual latency distribution, middleboxes, MTU and fragmentation, TLS renegotiation, DNS resolution and its failure modes, load balancers.
- **Real operational actors** — a human restarting the wrong node, a rolling deploy, a config-management push, a certificate expiry, a quota.
- **Dependencies outside the boundary** — the third-party client, the managed broker, the HSM, the settlement gateway (§3.5). Each is a named hole.
- **Security properties** — an adversary, malformed input from an untrusted peer, timing side channels, the behaviour of a compromised node.
- **Load and resource limits at real scale** — the model's numbers are not production's numbers.
- **The specification itself** — no simulator checks whether the property you asserted is the property you meant (§12.3).

---

## 9. Writing Tests That Find Bugs

### 9.1 A Test Is an Invariant Probe, Not an Outcome Check

A conventional test says: given this input, expect this output. That construction is what makes conventional suites brittle under refactoring and blind under concurrency. A DST assertion is different in kind — it is a statement that must hold in *every* state the system passes through, asserted continuously, whose failure is reported with the seed that produced it.

The consequence for how you write is direct. You do not write "the client receives ack 7". You write "for all nodes, at all points in simulated time, the sum of debits equals the sum of credits across the cluster", and you let the simulator's search find a state where it does not.

### 9.2 What to Assert

| Property class | Shape | Examples | Documented instance |
|---|---|---|---|
| **Safety** | Never happens | No two leaders in one term; no committed entry lost; no value used before validation | TigerBeetle's safety mode checks "strict serializability" under network, process and storage faults ([blog](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness)) |
| **Liveness** | Eventually happens, given assumptions | With a quorum healthy and every other failure permanent, the cluster keeps committing | TigerBeetle: "if there's a quorum of replicas available, then the cluster as a whole remains available, *regardless* of the behavior of the other replicas" ([blog](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness)) |
| **Linearizability / serializability** | The observed history is equivalent to some legal sequential history | Concurrent transactions produce a total order consistent with real time | TigerBeetle's documented safety check, above |
| **Model agreement** | The real implementation matches a simple reference model on observable state | The storage checkers of §6.5; a counter recomputed from the log equals the maintained counter | Byte-for-byte data-file agreement is TigerBeetle's documented storage checker ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)) |
| **Cross-cluster (global) invariants** | A property over *all* nodes simultaneously, not per node | Total of all balances equals total of all posted entries | The one teams nearly forget — see §14.4 |
| **Anti-properties** | Something must *not* be reachable | The recovery path must not run in a healthy run; a retry must not double-apply | Derived from the fault catalogue in §8.2 |

The last row inverts the usual instinct and deserves a sentence. Some of the highest-value assertions are that a state is *unreachable* — "in a run where no disk fault occurred, the recovery path must never execute". That assertion turns silent over-triggering recovery into a test failure, and it is only expressible because the simulator knows the entire fault schedule.

### 9.3 The Property-Based Testing Lineage

DST sits in a family whose roots are in property-based testing, and naming the lineage clarifies what is borrowed and what is new. The **QuickCheck** approach (Claessen and Hughes, ICFP 2000) introduced checking properties against randomly generated inputs and, crucially, *shrinking* a failing case to a minimal one. DST borrows the property-not-example framing and the random-input-search framing wholesale; what it adds is that the randomly varied input is the *environment* — faults, timing, interleavings — rather than only the data.

Two facts to keep straight. First, **shrinking did not come along for the ride**: DST failures are typically reported at full size, which is why §10.5 flags minimisation as an open problem for the technique rather than a solved one. Second, the ecosystems overlap — Antithesis publishes **Hegel**, a property-based testing library, from the same team that builds the commercial simulator, as named in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.3. Hegel is factually a property-based testing library; the product it sits beside, its pricing and its positioning are owned by [antithesis_guide.md](antithesis_guide.md) and are not discussed here. Model-based testing is the other adjacent family, and it is what §6.5's checkers actually are: a model of expected behaviour, exercised against the implementation, with agreement asserted. Neither family substitutes for DST, because neither owns the environment.

### 9.4 Why an Unasserted Simulator Finds Nothing

This deserves its own subsection because it is the most common way a technically correct DST implementation delivers no value. A simulator is a *search engine over executions*: it reaches a state and asks a question. If nobody wrote the question, the answer is always "no violation" — not because the system is correct, but because there was nothing to violate.

TigerBeetle's own style documentation states the multiplication factor and the density requirement plainly. Assertions "downgrade catastrophic correctness bugs into liveness bugs. Assertions are a force multiplier for discovering bugs by fuzzing", and "the assertion density of the code must average a minimum of two assertions per function" ([docs/TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md), checked 2026-09-22 — first-party, about its own practice). Its VOPR documentation adds the same point from the simulator's side: "Throughout the code base there are thousands of assertions checking that all manner of invariants hold true… If any assertion is broken under a specific set of circumstances, the simulation will crash and we debug that failure." ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22)

Note the second-order point, which is the honest one, and which that same style document makes against its own technique: "Assertions are a safety net, not a substitute for human understanding. With simulation testing, there is the temptation to trust the fuzzer. But a fuzzer can prove only the presence of bugs, not their absence." ([TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md), checked 2026-09-22). What generalises is the arithmetic: **search power × assertion density = bug-finding rate.** Raising one while the other is zero produces nothing. The production posture that document also describes — keeping assertions enabled in production on the argument that it is better to stop than to continue in an incorrect state — is a documented design position with real availability trade-offs, not a general recommendation.

### 9.5 The Search Framing

The mental model to adopt, and the one that justifies an engineering project rather than a test suite: **the simulator is not replaying the states production happened to hit; it is searching for the ones it did not.** Production samples a tiny, biased slice of the state space — the states reachable with healthy hardware, ordinary latency, correct configuration and no operator error. The fault schedule deliberately reweights the sample toward states production reaches only by accident, and the seed makes the reweighting repeatable and reviewable.

That framing also predicts the failure modes. A search is only as good as its coverage, which is why the two open problems in this technique (§8.4 on fault-parameter selection, §10.5 on minimisation) are both about *which states the search can reach* — and why §12's first limit, "a fault you did not model is a fault you cannot find", is a statement about the search space rather than about the tool.

---

## 10. The Seed-to-Replay Workflow: CI and Debugging

### 10.1 The Continuous Run and the Seed Sweep

A DST suite is not a suite you run once; it is a sweep you run continuously. Each seed is an independent exploration of the fault space, so throughput comes from running many seeds in parallel across cores — the opposite of a normal test suite, where the marginal value of the thousandth test is small.

The largest documented sweep is FoundationDB's, and it is a project statement about its own practice: Simulation "runs tens of thousands of simulations every night, each one simulating large numbers of component failures" ([FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html), checked 2026-09-22 — first-party project statement, quoted as such and not as an independently verified figure). Note what the sentence does *not* say: it does not give a wall-clock duration, a cost, or a bug count. Anything of that shape you see quoted elsewhere about DST should be checked against a dated source before it goes in a slide.

Three CI shapes are used in practice, and they are complements rather than alternatives:

| Shape | What runs | What it is for |
|---|---|---|
| **Seed sweep (nightly or per-merge)** | N seeds from a deterministic range or from a random starting point recorded in the log | Broad search across the fault space; the main bug-finding engine |
| **Pinned-seed regression set** | A fixed list of seeds that previously failed | Preventing regressions; §10.5 |
| **Directed scenario set** | Hand-written schedules for cases that matter and are not reachable by chance | Covering named scenarios cheaply; the same construction TigerBeetle documents using "the same deterministic simulation infrastructure" for "specific cases… that are hard or slow to replicate through random simulation" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22) |

### 10.2 What a Green Run Does and Does Not Mean

A green sweep means exactly one thing: **for these N seeds, under this fault model, with these assertions, no violation was found.** It does not mean the system is correct, and the reasons are already the guide's spine — the fault model bounds the search space (§12.1), the assertions bound what counts as a violation (§9.4), and the real environment is not in the loop at all (§12.2). The clearest statement of this in the sources consulted is the project that does the most of it, turned against its own technique: with simulation testing "there is the temptation to trust the fuzzer. But a fuzzer can prove only the presence of bugs, not their absence" ([docs/TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md), checked 2026-09-22).

The operational consequence is that a green sweep is a *hygiene signal*, not an acceptance criterion. The signals that should worry a team are the ones about the sweep itself: a sweep whose failure rate has been exactly zero for months is probably under-powered (§8.4), and a sweep whose failure rate has been high and stable for months means either the fault band is too aggressive or nobody is fixing what it finds.

### 10.3 The Failure Artifact

The unit of reproducibility is a triple — **seed + commit + configuration** — and the artifact should be produced automatically, printed on failure, and attached to the defect rather than retyped into a chat message (§7.4 shows the shape). Three properties make it usable: it is *complete* (nothing needed to reproduce is missing), it is *pasteable* (a single command reproduces the run), and it is *durable* (it survives the CI log retention window, because a defect ticket outlives a log). The most common omission is the configuration; the second most common is the workload or fault-preset identifier when those are referenced by name rather than inlined.

### 10.4 The Reproduction Protocol

This is the sequence to follow when a sweep fails, and it is deliberately mechanical. Steps 1–4 are the protocol; steps 5–7 are how the reproduction turns into an understanding.

1. **Obtain the triple.** Seed, commit hash, configuration identifier — from the failure artifact, not from memory. If the configuration is not captured, stop and fix the artifact before continuing; the rest of the protocol is not trustworthy without it.
2. **Check out the exact commit.** A branch HEAD is not the commit. If the failure came from a CI run on a merge commit, reproduce on the merge commit.
3. **Rebuild with the same build configuration.** Debug vs release matters in practice here — optimisations can change observable behaviour, and assertions kept in production may be compiled out of a release build. Use the same flags the sweep used.
4. **Re-run the single seed with the same configuration**, and confirm the *same* failure at the *same* simulated time and the same assertion. If it does not reproduce, you have a real finding of a different kind: a leak in the determinism claim (§15.1, §15.2), and the investigation should move to the boundary rather than to the bug.
5. **Follow simulated time, not breakpoints.** Because the simulator owns the clock, a breakpoint at a wall-clock point is meaningless; the useful debugging entry is a *condition on simulated time* or on the assertion's own state. Set the breakpoint where the invariant first becomes false, or one step earlier where you can watch it become false.
6. **Enable verbose tracing of the boundary.** The boundary is the whole input set, so the run's log of boundary operations — sends, deliveries, writes, flushes, timer fires, fault injections, in simulated order — is a complete record of what the system was told. Reading it is usually faster than stepping.
7. **Reduce, if you can, and if you cannot, record why.** §10.6.

The payoff of this protocol is the reason the whole investment exists, and it has a documented demonstration: TigerBeetle's VOPR documentation points at a recorded session in which an engineer live-debugs "a real simulator failure" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22, naming an IronBeetle episode). The significance is not the video; it is that the failure was debuggable *at all* — an interactive debugger attached to a distributed-system interleaving that occurred once, in simulation, under a seed. Contrast the alternative that every engineer has experienced: a production race that appears once, cannot be reproduced, and is closed as "cannot reproduce" (§13.2).

### 10.5 A Failed Seed Becomes a Permanent Regression Test

A seed that found a bug should never be dropped. It becomes a pinned case in the CI set and remains there indefinitely: it costs seconds or minutes, it re-verifies a fix, and — this is the part that justifies keeping it forever — it re-verifies the fix after every later change to the executor, the fault schedule or the boundary. Deterministic reproduction makes this unusually cheap, because the pinned case needs no scenario code at all: it is the same command with the same seed.

TigerBeetle documents exactly this bifurcation of its suite: random simulation for searching, plus specific cases run "using the same deterministic simulation infrastructure… that are hard or slow to replicate through random simulation" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md), checked 2026-09-22). A DST suite without a pinned-regression set loses its memory every night: the sweep finds a bug, the bug is fixed, and the only evidence the seed could ever find it again is a comment in a closed ticket.

**Why the pinned case must be a seed and not a rewritten test.** When a failure is found under a rich fault schedule, the tempting move is to hand-write a small test that injects just the one fault that mattered. That test is useful — it is faster and it documents the mechanism — but it is *not* the same test: it drops the interleaving, the unrelated concurrent faults, and the ordering that made the bug reachable. Keep both. The hand-written case explains; the pinned seed preserves.

### 10.6 Shrinking and Minimisation: OPEN

Conventional property-based testing minimises a failing case automatically (QuickCheck's shrinking, §9.3). **DST has no generally adopted equivalent, and this is genuinely OPEN.** The reason is structural: the input to a DST failure is not a data value that can be deleted field by field, it is a *schedule* — a sequence of faults and interleavings whose every element interacts with the program's control flow. Removing a fault from the middle of a schedule changes what runs afterwards, so the "smaller case" is usually a different execution that does not fail, and the search for a minimal failing schedule is itself an expensive search over schedules.

What teams do instead, all of which are partial:

- **Shorten the run** — find the earliest simulated time at which the invariant is already violated and stop there; often a large reduction for free.
- **Tune the fault presets down** — re-run the same seed with a narrower fault catalogue and bisect which fault class is necessary for the failure.
- **Bisect the schedule** — remove a contiguous block of faults and check whether the failure persists; a crude but usable approximation of shrinking.
- **Re-derive a hand-written minimal case** — express the mechanism you have understood as a directed test (§10.5), accepting that it is a different artefact.

Treat any stronger claim with suspicion: if a tool advertises automatic DST shrinking, ask what it is minimising (schedule length? simulated duration? number of distinct fault types?) and whether the minimised case is guaranteed to fail for the same reason.

### 10.7 The Three Open Questions in the Workflow

Stated plainly, because they are the questions a team will actually have and no source consulted here answers them:

1. **Seed-sweep policy.** How many seeds per night, from what range, with what retention and what failure-rate target? No defensible procedure was found in any source consulted. The honest position is that the sweep size is chosen by budget (§11.5) and the coverage it buys is unmeasured (§8.4).
2. **Minimisation.** §10.6.
3. **Fault-catalogue coverage.** How much of the real fault space a given catalogue reaches is unknown and, without production measurement, unknowable. The mitigations are the honesty inventory (§8.5) and the real-environment complement (§13.1).

---

## 11. What It Costs to Own

### 11.1 The Simulator Is a Product

The single most important budgeting fact, already stated in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.6 and repeated here only because it changes the shape of the business case: **simulation complexity is a product you now own.** A simulator has features, configuration, bugs, an owner, a backlog and a compatibility surface with the system under test. It will be maintained for as long as the system is. Nobody deletes it, because deleting it deletes the only reproducible evidence of the properties it checks.

What that means in staffing terms: the simulator needs a named owner (not a shared responsibility), a review path for changes to the fault catalogue, and a rule for what happens when the system's boundary changes — because a boundary change that is not reflected in the simulated implementation converts the simulation into a test of last year's system.

### 11.2 Coupling to a Language or a Runtime

Every serious implementation is coupled to something, and the honest examples are the most instructive.

**FoundationDB is coupled to a bespoke language.** Simulation there is "enabled by and tightly integrated with Flow, our programming language for actor-based concurrency", and "Flow also provides input to our simulation tool, which conducts deterministic simulations of the entire system, including its physical interfaces and failure modes" ([flow.html](https://apple.github.io/foundationdb/flow.html), checked 2026-09-22). Flow is "a new programming language that brings actor-based concurrency to C++11", implemented as a compiler that rewrites asynchronous functions into callback-based objects. The cost is visible in the same page's caveats: Flow files are preprocessed, "even though Flow code looks a lot like C++, it is not", local variables do not survive a `wait`, `state` variables follow non-standard scoping rules, and IDE support requires preprocessor shims. That is a real tax on every engineer who touches the codebase, paid whether or not a simulation is running — and it is the price of the tightest integration described in any public source.

**TigerBeetle is coupled to a language plus a discipline.** Its simulator requires Zig, and it is surrounded by whole-codebase constraints that exist to make the system predictable and testable: a "zero dependencies" policy "apart from the Zig toolchain" on the argument that dependencies "inevitably lead to supply chain attacks, safety and performance risk, and slow install times"; statically allocated memory only, with "no memory may be dynamically allocated (or freed and reallocated) after initialization"; explicitly-sized integer types rather than architecture-specific ones; and a minimum assertion density of two per function ([docs/TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md), checked 2026-09-22 — first-party documentation of its own style rules). These are not DST requirements as such, but they are the conditions under which "all non-deterministic parts of the system are stubbed out" ([vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md)) remains true of a growing codebase.

The generalisable warning: **if adopting DST requires a language, a dialect, or a whole-codebase discipline, budget for the fact that every new engineer pays the adoption cost again.** The hypervisor-based position (§2.6) avoids that tax by not touching the application — and pays for it by owning less of the application's semantics, and by being a platform relationship rather than a library.

### 11.3 Onboarding and Velocity Costs

- **Onboarding.** A new engineer must learn: the boundary and what may not cross it; the seed/RNG discipline and why `rand()` is banned; the clock rule and which clock to use; how to write an invariant rather than an assertion at the end of a scenario; and how to read a failure artifact. This is a real training cost, and the failure mode when it is skipped is silent: the first `rand()` call site looks harmless and destroys replay months later (§7.3).
- **Velocity.** Every feature that touches the outside world now needs an interface change, a simulated implementation, and probably a fault or a test. This is a per-feature tax, small individually, permanent in aggregate. It is worth stating honestly rather than selling: teams that adopt DST because "it finds bugs" and do not anticipate this tax tend to erode the boundary — adding one direct syscall "temporarily" — and the erosion is what kills the technique, not the technique's limits.
- **Latency of change.** A change to the boundary or the fault model is a change to the test environment for *everything*. It needs review by someone who understands the simulator, which makes the simulator a rate-limiting dependency on the critical path unless it has a real owner.

### 11.4 The CI Compute Cost, With the Arithmetic Shown

**Everything in this subsection is ILLUSTRATIVE arithmetic on assumed inputs. It is not a quoted price, not a vendor figure, and not a benchmark.** The point of showing it is to make the shape of the cost explicit: it scales linearly with seeds and run length, and it is usually *not* the dominant cost (§11.4 last row).

Assumptions, stated so they can be replaced: a core-hour rate of **US$0.04–0.05 per core-hour** (a generic cloud-CPU order of magnitude, illustrative); one seed = one core-minutes-scaled simulation run; CI runs once nightly.

| Seeds/night | Core-minutes per run | Core-hours/night | Cost/night (illustrative) | Cost/year (illustrative) |
|---|---|---|---|---|
| 500 | 10 | 83.3 | $4.17 @ $0.05 | ≈ $1,520 |
| 1,000 | 5 | 83.3 | $4.17 @ $0.05 | ≈ $1,520 |
| 2,000 | 8 | 266.7 | $10.67 @ $0.04 | ≈ $3,890 |
| 10,000 | 12 | 2,000.0 | $80.00 @ $0.04 | ≈ $29,200 |

**Engineering time is the dominant cost, not compute.** The same table puts a single full-time engineer at a fully loaded cost of roughly $180,000 per year on an illustrative assumption, so the "expensive" bracket of compute above is about one sixth of one engineer:

| Item | Illustrative annual cost |
|---|---|
| Simulator + simulation-related engineering (0.5 FTE) | ≈ $90,000 |
| Simulator + simulation-related engineering (1.0 FTE) | ≈ $180,000 |
| Simulator + simulation-related engineering (2.0 FTE) | ≈ $360,000 |
| Seed sweep at 10,000 seeds × 12 core-min/night | ≈ $29,200 |
| Seed sweep at the smallest illustrative setting | ≈ $1,520 |

The reading is not "compute is free" — it is that **the decision is a hiring decision, not a cloud bill**. A team that cannot name the engineer who owns the simulator has not budgeted for it, and the compute line will look fine right up until the sweep is quietly turned off. (All figures above: arithmetic performed 2026-09-22; inputs assumed and labelled as illustrative.)

### 11.5 The Honest Summary

This is a technique for a team with a system whose correctness is worth **a second engineering project** — a system where the failure mode is silent corruption, divergence, or an unreproducible outage, and where the organisation will fund a permanent team-level capability rather than a test-suite sprint. If the system's failure mode is a 500 error and a retry, the same money buys far more reliability elsewhere. §12.5 states the conditions for not adopting, and they are not rare.

---

## 12. Where It Stops Working

### 12.1 Limit One: You Test Against the Environment You Imagined

**The simulator tests the system you wrote against the environment you imagined. A fault you did not model is a fault you cannot find, at any seed count.** This is not a tuning problem; it is a property of a search over a fixed space. Ten thousand seeds explore the modelled fault space more thoroughly; they do not explore a fault that is not in the catalogue at all. The practical form of the failure is quiet: a system passes a dense sweep for a year and fails in production on a real network behaviour — asymmetric MTU blackholing, a certificate rotation, a broker's rebalance — that had no representation in the model.

The only defences are the ones already described: the honesty inventory (§8.5), real-environment testing to keep the model honest (§13.1), and treating production incidents as simulator specification bugs (§12.4).

### 12.2 Limit Two: No Evidence About Real Hardware or Real Timing

**A deterministic simulator gives you no evidence about real hardware or real timing.** Simulated time is an accounting dimension, not a measurement (§4.5). A run that "took a simulated year" says nothing about throughput, latency, cache behaviour, GC pauses, thermal behaviour, or how the system behaves when a disk is slow *because it is failing*. It also cannot observe a hardware bug, a firmware bug, or a kernel behaviour. A team that uses DST results as a performance or capacity argument has confused the two claims, and this confusion is the most likely way a DST programme generates a false confidence that reaches a decision-maker.

### 12.3 Limit Three: It Cannot Find a Specification Bug

**An implementation that faithfully matches a wrong specification passes everything a simulator can throw at it.** The simulator's oracles are the invariants humans wrote down; if the invariant encodes a misunderstanding — the wrong rule for who owns a balance, an incorrect interpretation of a settlement window, a mis-ordered reconciliation step — then a correct implementation of the wrong rule passes with flying colours, at every seed, for ever. The technique's precision is exactly what makes this failure so clean: unlike a fuzzy test, the simulator will faithfully confirm a wrong design.

This is the boundary between verification and validation, and it belongs in the guide because the two techniques are complementary in a specific way. A deterministic simulator verifies *this implementation against this design* exhaustively under modelled faults. It says nothing about whether the design is right. That question is answered by specification work and by adversarial review of the design itself, and the same source material that identifies "fundamental design errors, which are hard to find and expensive to correct in code" is the right tool for it: TLA+ is described by its creator as "a high-level language for modeling programs and systems — especially concurrent and distributed ones", useful precisely for "eliminating fundamental design errors" ([lamport.azurewebsites.net/tla/tla.html](https://lamport.azurewebsites.net/tla/tla.html), checked 2026-09-22; tools: the TLC model checker and the TLAPS proof system). §13.5 places model checking in the complement set, and §14.7 shows a worked decision to use a different technique on a component rather than simulate it.

### 12.4 The Simulator's Own Bugs

The simulator is a program, and its bugs are indistinguishable from findings. Four failure shapes recur:

- **A false positive** — the simulator's model is wrong and reports a violation the real system cannot produce. The team burns days on a non-bug, and — worse — the failure repeats under the same seed, so it looks like a genuine reproducible defect.
- **A false negative** — the model is *more permissive* than reality: it accepts a state the real system never reaches, or a fault is silently a no-op. This is the dangerous direction, because it manufactures confidence. The specific mechanism to fear is a simulated operation that returns success without modelling the failure it is supposed to model.
- **A determinism leak** — the simulator itself consumes nondeterminism, so a seed stops reproducing. The symptom is a flaky DST suite, which is a category error: this suite is supposed to be the one place flakiness cannot exist.
- **A drift** — the simulated implementation no longer matches the real one after a boundary change (§11.1), so the tested system is not the shipped system. This one is invisible without a deliberate comparison discipline, because everything keeps passing.

The mitigations are the same set as elsewhere in this guide, restated as simulator hygiene: keep the model as narrow as possible; validate the model against observed production behaviour; run representative workloads against the real environment too; treat every model change as a reviewed change with a test; and add assertions *about the simulator's own state* — in-flight message counts, fault-injection counts, draw counts per stream (§7.2) — so that a silent no-op becomes a visible failure.

### 12.5 When Not to Adopt

Stated plainly, as a list of conditions under which the honest answer is "no":

- **Correctness is not the binding constraint.** If the system's failures are degraded availability or slowness, and the business cost is a retry, DST is a poor use of the engineering budget.
- **The system is mostly I/O integration.** If the system is a set of adapters over third-party systems you do not control (§3.5), there is little deterministic core to simulate and the simulator would mostly be a fake of other people's software.
- **The boundary cannot be built — and the team has no appetite to build it.** Without §3 there is no DST; the boundary work is not optional and not retrofittable for free.
- **The organisation will not fund the owner.** A simulator with no named maintainer degrades into a green suite nobody trusts (§15.6).
- **The team already knows its specification is wrong.** A simulator will verify an implementation against a design the team already distrusts. Fix the design first (§12.3, §15.7).
- **Real-environment testing has not been tried.** For many teams, scripted fault injection in a staging environment plus black-box verification (§13.2) buys most of the available confidence at a fraction of the cost, and it tells you things about the real environment that DST cannot.
- **The system's lifetime is short.** The payback horizon of a second engineering project is years, not sprints.

---

## 13. The Alternatives and the Complements

### 13.1 Real-Environment and Chaos Testing — The Complement, Not the Competitor

Real-environment fault injection is the technique that keeps the simulator honest, and it is covered in this repository by [chaos_engineering_guide.md](chaos_engineering_guide.md) (real-environment fault injection is its subject; see its §13 on anti-patterns and §14 on claims). The division of labour is clean and it is not a rivalry:

| | Deterministic simulation | Real-environment / chaos testing |
|---|---|---|
| Environment | Modelled, seeded, owned by the simulator | Ground truth |
| Finds | Rare interleavings, modelled fault combinations, corruption and recovery bugs — reproducibly | Real-world environment bugs: hardware, kernel, network paths, real timing, real dependencies |
| Reproducibility | Perfect, given seed + commit + config | Effectively none for interleavings |
| Time to signal | Seconds of simulated time | Real time; rare bugs take months |
| Its failure mode | Tests the environment you imagined (§12.1) | Never manages to hit the state you care about |

A DST programme that never validates its model against the real environment is a programme that will slowly diverge from reality while reporting green. A chaos programme that never simulates cannot reproduce what it finds. Use simulation to search and reproduce, and real-environment testing to keep the model honest.

### 13.2 Black-Box Verification of Real Systems: The Jepsen-Style Approach

**DST and Jepsen-style testing are routinely confused, and the distinction is worth stating precisely because the two techniques are genuinely different instruments.** The Jepsen project's self-description is the anchor: it "aims to improve the safety of distributed databases, queues, consensus systems", maintains "an open source library for safety testing", and publishes "in-depth analyses of specific systems… In each analysis we explore whether the system lives up to its documentation's claims, file new bugs, and suggest recommendations for operators" ([jepsen.io](https://jepsen.io/), checked 2026-09-22).

The structural distinction:

| | DST | Jepsen-style black-box verification |
|---|---|---|
| What is under test | The system's own code, compiled into a simulated environment | A built artefact — often someone else's product — deployed for real |
| Where faults come from | A modelled environment owned by the simulator | The real environment: real partitions, real process kills, real clocks, real hardware |
| Persistence | Inside the system, usually through an API built for it | Outside; typically a client workload and an independent history check |
| Oracle | Invariants and model-agreement checks inside the system | A checker over the observed history — e.g. linearizability / serializability of the operations the client performed |
| Reproducibility of a finding | Perfect (seed + commit + config) | Poor; a failure is a property of an environment that no longer exists |
| What it finds that the other cannot | Internal corruption and recovery bugs, at will, in compressed time | Real-timing, real-hardware and real-dependency bugs — and it can be pointed at a system whose source you do not have |

The two are complements and the ecosystem has started to acknowledge it in code: Jepsen's own release notes describe a version "aimed at controllable entropy and support for running Jepsen inside Antithesis: a deterministic simulation testing environment", adding a pluggable random-value namespace so a test can "run with a deterministic seed, or… source entropy from an external system" ([jepsen.io release notes, Jepsen 0.3.10, dated 2025-12-01 on the project's news page, checked 2026-09-22](https://jepsen.io/)). Read that carefully: the black-box tool gained a seed so it could be hosted inside a deterministic simulator. That is the complementarity in practice — and the Antithesis product side of that story belongs to [antithesis_guide.md](antithesis_guide.md).

What black-box verification finds is not theoretical. A documented recent example from Jepsen's own analyses: MariaDB Galera Cluster 12.1.2 was found to have "two scenarios which led to the loss of committed transactions. First, under the recommended configuration settings it does not flush data to disk before acknowledgement; committed transactions can be lost when nodes crash in quick succession. Second, it occasionally loses committed transactions with process crashes and network partitions." ([jepsen.io](https://jepsen.io/), checked 2026-09-22 — the project's own finding about a third-party system.) Note the shape of that finding: it is about a *real deployment's* durability configuration, discovered from outside, against software whose source the tester did not restructure. No simulated environment could have produced the first half of it, because "the recommended configuration does not flush before acknowledgement" is a fact about the real system on real disks.

### 13.3 Fuzzing

Fuzzing searches for inputs that violate a property, and it overlaps with DST in mechanism (randomised search, seeded, reproducible, relies on assertions) and not in scope. A fuzzer varies *data*; a DST simulator varies *the environment*. They compose: a fuzz-generated malformed message delivered through a simulated network into a simulated cluster is a stronger test than either alone, and it is how the assertion-density argument of §9.4 pays off twice. The limit of each is the other's strength — a fuzzer cannot reach a rare interleaving deliberately, and a simulator does not search the space of malformed inputs it was not told to send.

### 13.4 Formal Methods

Model checking and specification languages work on the *design* rather than the implementation, which is precisely the gap in §12.3. TLA+ is described by its creator, Leslie Lamport, as "a high-level language for modeling programs and systems — especially concurrent and distributed ones… useful for eliminating fundamental design errors, which are hard to find and expensive to correct in code", with the TLC model checker and the TLAPS proof system as its tools ([lamport.azurewebsites.net/tla/tla.html](https://lamport.azurewebsites.net/tla/tla.html), checked 2026-09-22). The trade is the mirror image of DST's: a model checker explores the design's state space exhaustively *by construction*, so it can prove absence of a class of design error — but only relative to the model, which is not the code. A specification is not an implementation, and an implementation is not a specification; a programme with both has closed the loop, at the cost of maintaining two artefacts that must agree.

Light-weight specification — writing the protocol down precisely without a checker — is the cheap version: it will not prove anything, but it forces the disagreement about what the system is supposed to do to happen in a design review rather than in production.

### 13.5 Technique → Class of Bug It Finds

| Technique | Class of bug it finds | Class of bug it is blind to |
|---|---|---|
| **Deterministic simulation testing** | Rare interleavings, modelled fault combinations, corruption, recovery and liveness bugs; reproducibly, in compressed time | Unmodelled faults (§12.1); real timing and hardware (§12.2); specification errors (§12.3) |
| **Real-environment / chaos testing** ([chaos_engineering_guide.md](chaos_engineering_guide.md)) | Real-environment integration bugs; hardware, kernel, network-path, dependency and operator-triggered failures | Rare interleavings it cannot reproduce; anything requiring a specific fault schedule |
| **Black-box verification (Jepsen-style)** | Real-system durability and consistency violations, including in third-party products; configuration-dependent data loss | Internal corruption not observable through the client API; anything needing reproducibility to debug |
| **Fuzzing** | Malformed and boundary-value inputs; parser, validation and serialisation bugs; reachable assertion failures | Environment-dependent failures; interleavings |
| **Model checking / light-weight specification** | Fundamental design errors — the wrong rule, the missing state, the impossible-to-implement guarantee | Implementation bugs entirely; anything about the running code |
| **Assertions in production** | Detected incorrect states before they corrupt more state (a design posture with availability trade-offs, §9.4) | Anything the assertions do not state; and it is a detection mechanism, not a search |

---

## 14. The Cymbal Bank Worked Example

**This entire section is FICTIONAL AND EXPLICITLY ILLUSTRATIVE.** Cymbal Bank is a persona used across this repository to make consequences concrete. The component, the team, the findings, the dependency problems and every number below are invented for illustration. Nothing here is a claim about any real bank, system or vendor, and the arithmetic is labelled illustrative wherever it is arithmetic.

### 14.1 The Component

Inside Cymbal Bank, a team called **Ledger Core** owns a replicated **double-entry ledger service**. It accepts posting instructions (debit one account, credit another), applies them as an atomic double-entry transaction, and replicates the resulting log to a quorum of replicas so the ledger survives the loss of a node. It is the system of record behind the bank's internal ledgers; downstream systems (balances, statements, regulatory reporting) read from it.

Why this component is a plausible DST candidate and not a contrived one:

- **Its failure mode is silent corruption**, not a slow response. A ledger that loses a posting, double-applies one, or lets its replicas disagree is an incident even when every request returned 200.
- **Its bugs live in the seams** — recovery after a crash, replica rebuild, re-election, retry after an ambiguous failure — which are exactly the states a normal integration test cannot reach.
- **It has a deterministic core.** The posting rules, the double-entry application, the log and the replication protocol are Cymbal Bank's own code. That is the precondition from §3: something to simulate.

The concrete goal Ledger Core sets: *find ledger-corrupting and cluster-divergence bugs before they reach the shared test environment, reproducibly, without waiting for rare production states.*

### 14.2 The Boundary Design, and What Cannot Be Placed Behind It

The team rewrites the service's composition root so that a `LedgerEnvironment` is injected: a clock, a network (peer messaging), a durable store, a seeded entropy source, a process-lifecycle handle, and an observation sink. The posting rules, the log format, the replication protocol and the recovery logic are untouched production code compiled into the simulator.

The interesting part is the inventory of channels they *cannot* bring behind the boundary, because §3.5 predicts this is where the project either survives or dies.

| Channel | Can it go behind the boundary? | Decision, and what it costs |
|---|---|---|
| **Replica-to-replica messaging** | Yes — the team owns the protocol | Simulated network with modelled delay, duplication, reorder, partition and loss (§5.2) |
| **Local durable store** | Yes — the team owns the file format | Simulated filesystem with flush visibility, write ordering, arbitrary kill points, torn writes and corruption (§6.2) |
| **Clock and entropy** | Yes | Simulated clock and seeded RNG, one stream per subsystem (§7.2) |
| **Internal configuration** | Yes | Passed in as an input; nothing reads the ambient environment (§3.4) |
| **Third-party core-banking adapter** (the bank's transaction-processing system, reached through a vendor client library) | **No** — the library opens its own sockets and has its own reconnect thread | The team models a *reduction* of the protocol (submit, ack, reject, timeout) behind their own interface, and records in the honesty inventory that the vendor's real behaviour is not simulated. The adapter is exercised separately against a real sandbox |
| **External settlement/clearing gateway** (a partner system outside the bank's control) | **No** — a real external counterparty with its own file/batch semantics | Not simulated. The team pins the *interface* inside the model but does not model the counterparty. This is a named hole in the inventory, and §14.8 assigns it a different technique |
| **Hardware security module (HSM)** for signing postings | **No** — a physical device behind a vendor client | Model a signing oracle that can be slow, can fail, and can return an error the code must handle; the *cryptography* is not modelled and does not need to be |
| **Managed message broker** (the broker that carries posting submissions from upstream services) | **Partly** — the team can model delivery semantics, but not the broker's own behaviour | The broker's semantics (duplicate delivery, rebalance-induced redelivery, ordering per partition) are modelled from the team's documented understanding. The broker's *implementation* is not simulated, and an incident later shows why that matters (§14.6) |
| **Telemetry/logging sink** | Yes, but it must be modelled as a no-op-capable sink | Prevents logging from re-introducing real I/O into the deterministic path (§3.2) |

The two rows that took the most argument were the third-party adapter and the broker, and both were resolved by the same rule: **model the interface, record the gap, and verify the gap elsewhere.** What the team refuses to do is the third anti-pattern in §15.4 — declare the vendor library "simulated" by wrapping it in a fake that always succeeds.

### 14.3 The Seed and Fault-Schedule Design

- **One root seed per run**, with derived sub-streams for `network`, `disk`, `fault_schedule`, `workload` and `entropy` (§7.2). The workload generator draws from its own stream, so a change to the workload does not shift the fault schedule.
- **The workload** is a stream of posting instructions generated from the seed: valid transfers, deliberately unbalanced requests, requests with unknown accounts, duplicate submission IDs (simulating broker redelivery), and requests that are retried after an ambiguous timeout by the client model.
- **The fault schedule** is drawn lazily from the fault stream, with phase structure (§8.4): a first phase that injects crashes, restarts, delays, duplication, reordering, partitions and disk faults probabilistically; then a liveness phase in which a quorum is declared healthy, partitions within it are healed, and **every failure outside it is made permanent** — the pattern §8.4 describes from TigerBeetle's documented liveness mode. The second phase is what turned two of their three findings into failures.
- **Fault parameters** are randomised per run rather than fixed (§8.4), so the sweep explores different shapes of the fault space rather than the same shape a thousand times.

### 14.4 The Assertions — and the One They Nearly Forgot

The team writes the obvious ones first:

1. **Safety, per posting:** no posting is applied twice; every applied posting is a balanced double entry; a rejected posting has no effect on any balance.
2. **Safety, per replica:** a committed log entry is never lost, and the log is append-only.
3. **Safety, cluster-wide:** at most one leader per term.
4. **Liveness:** if a quorum is healthy and all other failures are permanent, the cluster continues to commit postings.
5. **Model agreement:** the balances derived from the committed log equal the balances the service reports.
6. **Anti-property:** in a run where no disk fault and no process kill occurred, the recovery path is never entered.

And the one they nearly forgot, which they added only after a reviewer asked what the ledger's *purpose* was: **a single global invariant over the whole simulated cluster — the sum of all debits equals the sum of all credits, across every replica and every account, at every point in simulated time.** Their earlier checks were all local: a node's log was append-only, a node's balances matched its own log, two replicas agreed. Every one of those can hold on every node while the cluster as a whole has conjured a posting into existence — because per-node agreement is a property of *replication*, not of *conservation*.

Two further refinements came from the same review:

7. **Mid-transaction safety:** a balance may be temporarily wrong only in states explicitly permitted by the design — which forced the team to write down, for the first time, exactly which intermediate states are legal. The invariant exposed a design ambiguity that had been tolerated for years.
8. **Global invariant checked at *every* simulated step, not at the end:** an end-of-run check would have missed every transient violation, and the postings that produce a transient violation are exactly the interesting ones.

### 14.5 What the Simulator Found

Three illustrative findings, each of the shape DST is supposed to produce — rare, deterministic under a seed, and located in a recovery or rebuild path nobody could reach on purpose:

1. **A replica rebuild that dropped a posting under a specific interleaving.** A recovering replica accepted a snapshot boundary and a trailing log segment in an order that left a gap; the per-replica append-only check passed (the log *was* append-only), the two-replica agreement check passed (both replicas had the same gap), and only the global conservation invariant fired. This finding is the reason §14.4's sixth assertion exists.
2. **A double-apply on ambiguous client retry.** Under a partition that made a submission's acknowledgement fail while the posting itself committed, the client model retried; the deduplication window had already been trimmed because a *later* posting had advanced it. This is a classic retry-idempotency bug, and it was reproducible from a seed in seconds of simulated time after the fault schedule reached the state.
3. **A liveness failure: permanent asymmetric partition stalled the cluster.** The cluster remained internally consistent — every safety assertion held — but it stopped committing, which only became visible once the schedule made a non-core replica's failure permanent (§14.3, phase two). Safety checks alone would have reported a green run.

### 14.6 One Bug It Cannot Find

The bank suffers an incident that the simulator was structurally incapable of finding. A managed-broker **rebalance** causes in-flight posting submissions to be redelivered; the broker also delivers a duplicate whose *ordering* relative to a later submission differs from anything the team modelled, because they modelled duplicate delivery as "the same message arrives twice, close together". The real interleaving — a duplicate arriving *after* subsequent messages from the same partition, with the original partition's ordering no longer applicable — produced a lost update that every simulated run passed, because the team's broker model could not express it.

The lesson is the one §12.1 states: **the fault was not in the model, so no seed count would ever have found it.** The finding is recorded in the honesty inventory as a *model gap with a production incident attached*, which is the most valuable kind of entry that inventory can hold. Nothing about the simulator was broken; the environment was imagined, and the imagination had a hole exactly where the broker's real behaviour differed from the documented reduction the team had written down.

### 14.7 CI and Reproduction

- **Per-merge:** a short sweep (a few hundred seeds, small clusters) as a required check. **Nightly:** a larger sweep (§14.8) plus the pinned-seed regression set, which grows by one entry every time the sweep fails.
- **Pinned seeds are permanent.** The regression set is a list of seeds; each entry re-runs an exact former failure against current code. Every boundary change, executor change or fault-model change is expected to keep the whole set passing.
- **The failure artifact** is the triple from §10.3. Ledger Core's rule is that a failing sweep job fails the build *and* prints the pasteable replay command in the job summary, so that a defect ticket contains everything needed.
- **The reproduction protocol** is §10.4 verbatim, with one local addition: because their boundary tracing is on by default in simulated runs, step 6 of the protocol usually ends the investigation — the boundary trace shows the exact sequence of deliveries, faults and flushes that produced the violation, in simulated order, and the code path follows from it.
- **The debugging payoff** is the one that convinced the sceptics on the team: a corrupted-ledger bug that would have been a multi-week production mystery was, in simulation, a breakpoint in a debugger at a specific simulated time, with the entire environment frozen and the whole run repeatable at will.

### 14.8 What It Costs Them — ILLUSTRATIVE ARITHMETIC

**Every figure in this subsection is invented for illustration and computed on assumed inputs. None of it is a quoted price, a vendor rate, or a measured cost.**

Assumptions: boundary work and simulator build: **0.5 FTE for two quarters** to reach a first useful sweep, then 0.25–0.5 FTE steady state. Nightly sweep: **1,500 seeds × 8 core-minutes**, on a concurrency of 200 cores, at an illustrative **US$0.04 per core-hour**.

| Item | Illustrative figure | Arithmetic shown |
|---|---|---|
| Nightly sweep compute | ≈ $8.00/night | 1,500 × 8 = 12,000 core-min = 200.0 core-hours; × $0.04 = $8.00 |
| Annual sweep compute | ≈ $2,920/yr | $8.00 × 365 |
| Sweep wall-clock duration | ≈ 1.0 hours/night | 12,000 core-min ÷ 200 cores ÷ 60 |
| Build phase (0.5 FTE, 2 quarters) | ≈ $45,000 | 0.5 × $180,000 × 0.5 |
| Steady state (0.5 FTE/yr) | ≈ $90,000/yr | 0.5 × $180,000 |
| Steady state (0.25 FTE/yr) | ≈ $45,000/yr | 0.25 × $180,000 |

The reading, and it is the section's real conclusion: **the sweep costs less than one percent of the engineering cost, and the engineering cost is what must be justified.** A team arguing about the cloud bill for the sweep has mis-priced the project by two orders of magnitude.

### 14.9 The Recommendation, Including a Decision *Not* to Simulate

Ledger Core's recommendation to its engineering leadership, in the shape such a recommendation should take:

1. **Simulate the ledger service.** The core, the protocol and the recovery logic are the bank's own code, the failure mode is silent corruption, and the payoff is a reproducible search over states the bank cannot reach otherwise. Approved, funded at 0.5 FTE build and 0.25–0.5 FTE steady state.
2. **Do not simulate the managed broker.** Instead, treat the broker as an external system under test: run **black-box, Jepsen-style verification against a real broker deployment** (§13.2) — real partitions, real kills, real rebalances, with an independent checker over the observed delivery-and-posting history — and feed whatever it finds back into the broker model as a *modelled* fault. This is the deliberate decision not to bring a dependency inside the model, and it is the honest response to §14.6.
3. **Do not simulate the settlement handshake; specify it.** The external settlement/clearing gateway is a stateful protocol with a counterparty, and its risks are design risks (what happens if the handshake is interrupted at each state; whether the guarantee the bank believes it has is the guarantee it has) rather than implementation races. Use **light-weight formal specification of the handshake states and transitions** — and escalate to a model checker if the review finds an ambiguity — because §12.3's limit applies in full: a simulator cannot find a specification bug, and this is a specification risk.
4. **Keep the two techniques in the loop permanently.** Feed production incidents into the honesty inventory as model gaps; keep real-environment checks running so the model does not drift; and re-derive the broker model from what black-box testing finds. Simulation to search, black-box to stay honest, specification to be right about the design.

---

## 15. The Anti-Patterns and the Claims Audit

### 15.1 The Anti-Pattern Catalogue

| # | Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **The boundary with a leak that lets real time in** | Runs are wall-clock bound; a seed reproduces sometimes; "the simulator is slow today" | One direct `sleep`, `now()` or syscall outside the boundary — often added "temporarily" for a library that needs it (§3.5) | Make the unseeded/ambient APIs unreachable from the deterministic core; lint or grep for them in CI; fail loudly on a simulated operation that is not implemented rather than no-op'ing it |
| 2 | **The global unseeded random call** | Rare seeds stop reproducing; the suite is "flaky" — in the one place flakiness should be impossible | A draw from a process-wide or OS RNG in behaviour-affecting code (§7.3) | No global RNG in the module graph; all entropy through injected, per-subsystem seeded streams; report draw counts per stream on failure so a shifted stream is visible |
| 3 | **The simulator with no assertions** | Green for months; findings are only "the cluster did not come up" | Invariants were never written, or were written as end-of-run outcome checks (§9.4) | Assert continuously over states, not outcomes; global cluster-level invariants as well as per-node; include anti-properties; require assertion density as a code-review standard; act on §14.4's sixth assertion |
| 4 | **The fault catalogue that only models clean crashes** | Recovery paths look exercised; production loses data anyway | Faults are injected only at operation boundaries; no torn writes, no flush-ordering faults, no corruption, no lost-but-acknowledged writes (§6.3) | Inject at arbitrary points, not boundaries; model write ordering and flushes explicitly; corrupt reads and writes; crash during recovery, not only during steady state |
| 5 | **The seed not captured in the failure artifact** | "It failed in CI last night and we cannot reproduce it"; the CI log has already rolled over | The run does not print seed + commit + config; or the config is a mutable reference rather than an inlined value (§10.3) | Make the artifact impossible not to produce; fail the build with the pasteable replay command in the summary; store the artifact with the defect, not only in logs |
| 6 | **The simulator that becomes a second product nobody maintains** | The model drifts from the system; new features are excluded from the simulation "for now"; the sweep is still green and no longer means anything | No named owner; no review path for model changes; no acceptance test that the model matches the system (§11.1, §12.4) | Name an owner; require simulator changes to be reviewed like production changes; make the honesty inventory a maintained document; treat a production incident as a model defect with a ticket |
| 7 | **Adopting DST instead of fixing a specification you know is wrong** | A large green sweep cited as evidence the design is sound | The design question was deferred and the implementation was verified against it (§12.3) | Fix the design first — light-weight specification or a model check — then simulate the implementation; never cite a green sweep as evidence about correctness of the design |
| 8 | **The simulated dependency that is not a model** | Simulated runs pass where the real dependency fails; the model was written at the interface, not from the dependency's behaviour | A fake that always acks, always orders, never duplicates (§3.5) | Derive the model from measured dependency behaviour; verify the model against the real dependency with a different technique (§14.9); record every reduction in the honesty inventory |
| 9 | **Fault probability treated as a constant to copy** | Either nothing ever happens or every run is chaos; the sweep's failure rate is meaningless | The band was never tuned, and probabilities were copied from another team's system (§8.4) | Tune the band as an experiment; randomise parameters per run (swarm-style); phase the schedule; measure what the sweep actually reaches |
| 10 | **Simulated time read as a performance result** | "It processed a simulated year in a minute" appears in a capacity deck | Simulated time is an accounting dimension, not a measurement (§4.5) | Keep performance claims out of simulation reports entirely; measure on real hardware; state the distinction in the report's own wording |

### 15.2 The Four Questions a Reviewer Should Ask

A short checklist that converts the catalogue above into a review procedure:

1. **Show me the honesty inventory** (§8.5) — what is not modelled, with an owner for each gap.
2. **Show me the assertions** (§9.4) — especially the global, cluster-level ones, and the anti-properties.
3. **Show me a failure artifact and reproduce it here** (§10.3, §10.4).
4. **Show me how the model is validated against reality** (§13.1) — production measurements, black-box verification, or incidents fed back into the model.

A programme that cannot answer all four has a simulator, but not a verification capability.

### 15.3 The Claims Audit

Every mechanism claimed in this guide, with its source, the date checked, and the quality of that source. **"First-party"** means the project documenting its own system; that is authoritative about *what the project does* and not independent evidence that it works. **"Project claim"** means a first-party performance or capability assertion. **"Conference talk"** means an abstract or talk, which is not a specification. **"Secondary"** means general literature or a third-party account.

| Claim in this guide | Source | Date checked | Quality |
|---|---|---|---|
| DST replaces the system's hardware with software "which fulfills the contracts of those systems, but whose state is completely transparent to the developer"; the same failure pattern can be replayed indefinitely | [thestrangeloop.com — Will Wilson, Strange Loop 2014 talk page](https://thestrangeloop.com/2014/testing-distributed-systems-w-slash-deterministic-simulation.html) | 2026-09-22 | **Conference talk** (abstract only; not a specification — relied on here for framing and vocabulary, not for mechanisms) |
| Simulation conducts "a *deterministic* simulation of an entire FoundationDB cluster within a single-threaded process"; determinism gives "perfect repeatability" | [FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html) | 2026-09-22 | **First-party** project documentation (mechanism) |
| Simulations "usually have about a 10-1 factor of real-to-simulated time" | [FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html) | 2026-09-22 | **Project claim / project estimate** of its own ratio — labelled as such in §4.1; not an independent benchmark |
| Simulation "runs tens of thousands of simulations every night, each one simulating large numbers of component failures" | [FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html) | 2026-09-22 | **Project claim** about its own practice; quoted verbatim, not converted into a cost or speed figure |
| Simulation models drive performance and drive space "and the possibility of the drive filling up"; models the network, "allowing a small amount of code to specify delivery of packets"; simulates network/machine/datacenter failures including "machines coming back from the dead" | [FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html) | 2026-09-22 | **First-party** (mechanism) |
| "swizzle-clogging" as the winning internal fault-design competition: clog a random subset of nodes' connections one by one, unclog in random order | [FoundationDB 7.4.7 docs, testing.html](https://apple.github.io/foundationdb/testing.html) | 2026-09-22 | **First-party**, and the page's own assessment that it "seems to be particularly good" is a **project claim**, not a measurement |
| Simulation is "enabled by and tightly integrated with Flow"; Flow brings actor-based concurrency to C++11, rewrites async functions into callbacks "to avoid blocking", and provides input to the simulation tool; `choose … when` waits "in a ordered and predictable way"; Flow's caveats (not C++, `state` scoping, IDE shims) | [FoundationDB 7.4.7 docs, flow.html](https://apple.github.io/foundationdb/flow.html) | 2026-09-22 | **First-party** (mechanism and its own language's costs) |
| "In the simulator, all non-deterministic parts of the system are stubbed out. This includes the clock, network, and disk operations." | [TigerBeetle docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) | 2026-09-22 | **First-party** (mechanism) |
| "Because our simulator is deterministic based on a *seed* number and the Git commit, we can perfectly reproduce any bugs discovered in testing for easy local debugging" | [TigerBeetle docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) | 2026-09-22 | **First-party project claim** about its own simulator |
| "Crucially, VOPR can speed up time arbitrarily. One minute of VOPR time is equivalent to days of real-world testing." | [TigerBeetle docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) | 2026-09-22 | **Project claim**, no measured multiple; deliberately not converted into a speed factor here. (VOPR's "1000x" and "1024 cores" figures are already flagged in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5 and are not restated) |
| "The VOPR uses a random seed to tune parameters for injecting different types of faults"; it may "drop and reorder packets, partition the network, or corrupt reads and writes to the 'disk'" | [TigerBeetle docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) | 2026-09-22 | **First-party** (mechanism and fault classes) |
| Storage checkers verify that replicas' data files are "byte-for-byte identical across caught-up nodes in the cluster"; "thousands of assertions… checking that all manner of invariants hold true"; assertions kept on in production | [TigerBeetle docs/internals/vopr.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/internals/vopr.md) | 2026-09-22 | **First-party** (mechanism and its stated design posture) |
| Safety mode injected faults "uniformly… on every tick it rolled a dice"; probabilistic faults heal, so a liveness bug was not found; liveness mode makes non-core failures permanent; the found example was a repair/round-robin load-balancing resonance; the property "if there's a quorum of replicas available, then the cluster as a whole remains available" | [TigerBeetle blog — Simulation Testing For Liveness](https://tigerbeetle.com/blog/2023-07-06-simulation-testing-for-liveness) | 2026-09-22 | **Project account of its own simulator** (blog; first-party about its own practice, not peer-reviewed) |
| Swarm testing as influence on per-run fault-parameter randomisation | [Swarm testing (Regehr et al.)](https://users.cs.utah.edu/~regehr/papers/swarm12.pdf), cited by TigerBeetle's blog as an inspiration | 2026-09-22 | **Secondary / academic literature** (the paper is primary for its own content; the linkage is the project's) |
| Assertions "downgrade catastrophic correctness bugs into liveness bugs"; minimum "two assertions per function"; "a fuzzer can prove only the presence of bugs, not their absence"; "zero dependencies" policy; static allocation only after init; explicit integer sizes | [TigerBeetle docs/TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md) | 2026-09-22 | **First-party** documentation of its own engineering rules |
| Jepsen "aims to improve the safety of distributed databases, queues, consensus systems", maintains "an open source library for safety testing" and publishes "in-depth analyses"; MariaDB Galera Cluster 12.1.2 findings (loss of committed transactions; no flush before acknowledgement) | [jepsen.io](https://jepsen.io/) | 2026-09-22 | **First-party** for Jepsen's self-description and for its own analyses of a product it does not own |
| Jepsen 0.3.10 "aimed at controllable entropy and support for running Jepsen inside Antithesis: a deterministic simulation testing environment"; pluggable random-value namespace enabling deterministic seeds or external entropy | [jepsen.io release notes](https://jepsen.io/) (Jepsen 0.3.10, dated 2025-12-01 on the project's news page) | 2026-09-22 | **First-party project release note** |
| TLA+ is "a high-level language for modeling programs and systems — especially concurrent and distributed ones", useful for "eliminating fundamental design errors, which are hard to find and expensive to correct in code"; tools TLC and TLAPS; TLA+ now under the TLA+ Foundation | [lamport.azurewebsites.net/tla/tla.html](https://lamport.azurewebsites.net/tla/tla.html) (page last modified 2025-10-13) | 2026-09-22 | **First-party** (author's own page) |
| QuickCheck introduced property-based testing with random inputs and shrinking a failing case to a minimal one | Claessen and Hughes, ICFP 2000 (named as general literature; paper not fetched this pass) | 2026-09-22 | **Secondary** — named as lineage, with no claim resting on details of the paper |
| Antithesis publishes Hegel (property-based testing) from the same team as its simulator | [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.3 (repo-internal, dated 2026-09-17 there) | 2026-09-22 | **Repo-internal cross-reference**; product specifics belong to [antithesis_guide.md](antithesis_guide.md) |
| All cost arithmetic in §11.4 and §14.8 | **Invented illustrative inputs** | 2026-09-22 | **Illustrative** — arithmetic on assumed rates, labelled as such, not a quoted price |
| The Cymbal Bank component, team, findings, incident and recommendation (§14) | **Fictional** | 2026-09-22 | **Illustrative fiction**, explicitly labelled |

---

## 16. What Could Not Be Verified

### 16.1 What Could Not Be Verified

Stated as a list, because a guide that hides its gaps is worth less than one that publishes them. Nothing in this section is an argument that a mechanism does not exist — it is a record of what a source could not confirm on 2026-09-22, and it is a to-do list for anyone who needs certainty on that point.

| # | Item that could not be verified | What this means, and how to resolve it |
|---|---|---|
| 1 | **The exact RNG-derivation scheme in either project.** Neither FoundationDB's public testing page nor TigerBeetle's VOPR page publishes how the root seed is turned into per-subsystem random streams. §7.2 presents the derivation discipline as generic engineering practice, and says so. | Read the projects' source, or derive your own scheme and document it. Do not present any scheme as "how FoundationDB does it". |
| 2 | **The simulated network's latency distribution function.** FoundationDB documents *that* the network is modelled and that code specifies packet delivery; the distribution, per-link parameters and correlation structure are not published on the page consulted. §5.5 says so. | Source code or a first-party engineering post. Until then, treat any specific distribution shape as unverified. |
| 3 | **A documented torn-write or bit-corruption model for FoundationDB's simulated disk.** The testing page documents drive *performance*, drive *space* and the drive filling up; TigerBeetle's VOPR page documents corruption of disk reads and writes. The former's corruption model was not verified. | Check the FoundationDB repository; do not attribute a torn-write model to it on the strength of this guide. |
| 4 | **Clock skew between simulated nodes as a modelled fault class.** Not found on either public documentation page consulted. Absence of a *documentation* mention is not absence of the fault. | Check the implementations' source or ask the projects. Do not cite this guide as evidence that neither models it. |
| 5 | **Resource-exhaustion faults (allocation failure, FD exhaustion, bounded-queue overflow) as modelled fault classes in either project.** | As above. |
| 6 | **Any independently measured speed-multiple or throughput figure for DST.** Every performance number encountered is a project's own claim about its own simulator: FoundationDB's ~10:1 ratio and its nightly-run count, and TigerBeetle's "arbitrarily" and "one minute… equivalent to days". The siblings' "1000x" and "1024 cores" claims are already flagged in [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5. **No independent benchmark was found, and none is asserted here.** | Measure your own system. A DST speed-multiple is workload-specific and a copied figure is worse than none. |
| 7 | **A defensible seed-sweep policy** (how many seeds, which seeds, what failure-rate target, what retention). Not found in any source; §10.7 flags it as OPEN. | Treat as an experiment with an owner. |
| 8 | **Automatic shrinking/minimisation for DST failures.** No implementation was found that both performs minimisation and documents a correctness guarantee for it; §10.6 flags it as OPEN. | Ask any tool vendor precisely what is minimised and whether the minimal case is guaranteed to fail for the same reason. |
| 9 | **How much coverage a given fault catalogue actually provides.** No method or measurement found; §8.4 flags it as OPEN. | Production measurement plus the two complements in §13. |
| 10 | **The QuickCheck paper text.** The lineage (property-based testing, shrinking) is named in §9.3 from general knowledge of the literature; the paper itself was not fetched on 2026-09-22. No mechanism in this guide rests on it. | Read Claessen and Hughes, ICFP 2000. |

### 16.2 The Glossary

| Term | Definition as used in this guide |
|---|---|
| **Assertion** | A check that must hold in every state; in a DST system, the mechanism that converts a reachable state into a reportable failure |
| **Black-box verification** | Testing a built artefact from outside, against a real deployment, with an independent oracle (Jepsen-style) |
| **Chaos engineering** | Deliberate fault injection into a real environment; the complement to DST ([chaos_engineering_guide.md](chaos_engineering_guide.md)) |
| **Clean shutdown** | A system told to stop; tests the shutdown path, not crash semantics |
| **Crash** | A system ceasing at an arbitrary instant with no notification; the state that makes recovery logic testable |
| **Deterministic executor** | The component that runs the system's tasks in a defined order in one thread, owning *when* code runs |
| **Durability contract** | The written statement of what survives a crash and what does not; the precondition for a meaningful crash test |
| **Environment (injected)** | The bundle of interfaces (clock, network, storage, entropy, lifecycle, observation) the system is constructed with |
| **Failure artifact** | The triple — seed + commit + configuration — that reproduces a run |
| **Fault catalogue** | The enumerated set of fault classes the simulator can inject |
| **Fault schedule** | The concrete ordered sequence of faults for one run, derived from the seed |
| **Honesty inventory** | The maintained list of what the simulator does not model |
| **I/O boundary** | The single narrow interface through which the system touches the outside world |
| **Invariant** | A property asserted to hold continuously in every reachable state |
| **Liveness** | A property that something good eventually happens, under stated assumptions |
| **Model-based checker** | A simplified reference model (or an independent computation) compared against the real implementation |
| **Monotonic time** | A non-decreasing counter used for durations and ordering; the clock the simulator genuinely owns |
| **Pinned seed** | A seed retained permanently in CI as a regression test for a fixed bug |
| **Replay** | Re-running the same revision with the same seed and configuration to obtain the identical execution |
| **Safety** | A property that something bad never happens |
| **Seed** | The integer input that, with the revision and the configuration, determines the run |
| **Shrinking / minimisation** | Reducing a failing case to a smaller failing case; OPEN for DST (§10.6) |
| **Simulated network** | The simulator-owned data structure holding in-flight messages with modelled delivery, delay, duplication, reorder, partition and loss |
| **Simulated time** | The counter the simulator advances; an accounting dimension, not a measurement |
| **Sub-seed** | A per-subsystem random stream derived deterministically from the root seed |
| **Swizzle-clogging** | FoundationDB's documented winning fault pattern: clog a random subset of nodes' connections one by one, then unclog in random order |
| **Torn write** | A write that landed incompletely, typically at a device block boundary |
| **Wall time** | The clock used for human- or counterparty-readable timestamps; an unseeded input unless derived from simulated time |

### 16.3 Cross-References

**Owns the definition, lineage, landscape and benefit case of DST:**

- [deterministic_engineering_guide.md](deterministic_engineering_guide.md) — **§5 "Toolkit II: Deterministic Simulation Testing"** (§5.1 definition; §5.2 the canonical source; §5.3 the implementations landscape with dated statuses; §5.4 the two benefits; §5.5 the five architectural requirements; §5.6 the costs; §5.7 simulation versus real-environment testing; and the claims audit that flags VOPR's "1000x"/"1024 cores"). Also its §3 for the removably-nondeterministic sources a simulator *cannot* fix, §4 for record-and-replay debugging, and §6 for durable-execution replay and workflow versioning — three adjacent techniques that are not DST.

**Sibling guides named in this guide:**

- [antithesis_guide.md](antithesis_guide.md) — owns the Antithesis product, company, pricing, market and history. This guide names Antithesis only as one implementation of the same architectural shape and cites its documentation nowhere as a mechanism source.
- [chaos_engineering_guide.md](chaos_engineering_guide.md) — the complement (§13.1): real-environment fault injection, its own anti-patterns (§13) and claims audit (§14).
- [turso_database_guide.md](turso_database_guide.md) — a pointer only; it mentions DST and Antithesis in passing (lines 226 and 641) and is not a DST source.
- [network_packet_capture_guide.md](network_packet_capture_guide.md) — the real-wire observation discipline; useful when validating a simulated network model against measured production traffic (§5.6, §13.1).

**Testing and verification guides in this repository:**

- [test_orchestration_guide.md](test_orchestration_guide.md) — coordinating suites and environments; the place a DST sweep has to fit alongside everything else in CI.
- [shell_testing_frameworks_comparison_guide.md](shell_testing_frameworks_comparison_guide.md) and [bash_script_testing_automation.md](bash_script_testing_automation.md) — determinism and testing discipline for the automation around the system.
- [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) — adversarial testing; note that a simulator with no adversary models no security property (§8.5).

**Where the Open questions live in this guide:**

- §8.4 — fault-probability tuning and per-run parameter randomisation (OPEN).
- §10.6 — shrinking and minimisation for DST failures (OPEN).
- §10.7 — the seed-sweep policy in CI, and fault-catalogue coverage (OPEN).
- §16.1 — ten specific unverified items, each with how to resolve it.

### 16.4 The Closing Summary

Six sentences, in the order the reader will need them.

A deterministic simulator is an engineering project, not a test library: it keeps the production code and replaces its environment, which means the environment becomes a second product with an owner, a backlog and a maintenance schedule ([§11](#11-what-it-costs-to-own), [deterministic_engineering_guide.md](deterministic_engineering_guide.md) §5.6). The expensive half of that project is not the simulator but the boundary — the restructuring that lets one narrow interface carry both the real and the simulated world, and the discipline that keeps a stray clock read or an unseeded random draw from silently destroying replay ([§3](#3-the-io-boundary), [§7](#7-the-deterministic-scheduler-and-the-seed-discipline)). What you buy for that cost is a search over the states production reaches only by accident: crashes at arbitrary points, torn writes, permanent partitions, and a virtual timeline in which leases, timeouts and backoff ladders are exercised in seconds ([§4](#4-simulated-time), [§8](#8-fault-injection-as-a-first-class-seeded-parameter)). What you are then obliged to write is the invariants — because a simulator with no assertions finds nothing, and search power multiplied by zero is zero ([§9](#9-writing-tests-that-find-bugs)). What you must never claim for it is what it cannot do: it tests the environment you imagined, it says nothing about real hardware or real timing, and it cannot find a specification bug ([§12](#12-where-it-stops-working)) — which is why simulation belongs beside black-box verification of the real system, real-environment fault injection, and formal specification of the design, rather than instead of any of them ([§13](#13-the-alternatives-and-the-complements)). The honest summary of the whole technique is one sentence about what a simulator's fidelity is ultimately a statement about: a deterministic simulator is only ever as good as the environment, not the system.
