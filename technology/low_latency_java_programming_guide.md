# Low-Latency Java Programming: The JVM on Your Terms

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Core Data Infrastructure / Architecture — Banking, Electronic Trading, Real-Time Systems, JVM Platform Engineering
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** September 2026

---

> The third language companion to a latency discipline already owned by the C/C++ guide. This document covers only the *JVM-specific delta*: the two runtime adversaries that a native stack does not have — the just-in-time compiler and the garbage collector — plus the weaker memory model, the allocation discipline that actually governs the pause, the off-heap toolkit, the measurement harness a managed runtime forces on you, and an honest ledger of where the runtime costs latency and where it repays it.

**Audience**: Solution architects, JVM platform engineers, and trading-technology leads who already understand the latency discipline and need to know what changes when the hot path runs on a managed runtime. This guide assumes you have read the repo's C/C++ sibling. Where the sibling owns a topic, this document cross-references it by section number and does not re-derive it. All system facts (release dates, JEP numbers, flag syntax and gating, tool and library maintenance status) were verified at primary sources and are date-stamped; anything that could not be established is stated as such in §16.

**Verification date for all version and stability claims: 21 September 2026.**

---

## Table of Contents

1. [The Overview, the Companion Boundary and the Decoder](#1-the-overview-the-companion-boundary-and-the-decoder)
2. [Why Java Is a Different Problem](#2-why-java-is-a-different-problem)
3. [The JIT and the Warmup Problem](#3-the-jit-and-the-warmup-problem)
4. [Garbage Collection](#4-garbage-collection)
5. [The Allocation Discipline](#5-the-allocation-discipline)
6. [The Memory Model and False Sharing](#6-the-memory-model-and-false-sharing)
7. [Concurrency and the Off-Heap Toolkit](#7-concurrency-and-the-off-heap-toolkit)
8. [Transport, as It Differs in Java](#8-transport-as-it-differs-in-java)
9. [Measurement: The Measured Zero](#9-measurement-the-measured-zero)
10. [The Runtime Configuration](#10-the-runtime-configuration)
11. [AOT, Native Images and the Alternative Runtimes](#11-aot-native-images-and-the-alternative-runtimes)
12. [The Platform Changes in Flight and Recently Delivered](#12-the-platform-changes-in-flight-and-recently-delivered)
13. [The Banking and Trading Angle](#13-the-banking-and-trading-angle)
14. [The Honest Assessment](#14-the-honest-assessment)
15. [The Anti-Patterns and the Claims Audit](#15-the-anti-patterns-and-the-claims-audit)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Companion Boundary and the Decoder

In Java the latency is not in your source code. It is in the runtime's decisions — the compiler's, about *when* to compile your loop and *what shape* to compile it in; the collector's, about *when* to stop your threads; and the hardware's, about *whether* a write your source code ordered sequentially is actually seen in that order. The discipline in Java is not to write C++ in Java. It is **to bring the runtime onto your terms** — to make the compiler's decisions predictable, the collector's pauses small and scheduled, and the memory model's guarantees explicit rather than assumed.

That sentence is the whole guide. Everything below is either a mechanism behind it or a guardrail around it.

### 1.1 The sibling relationship, stated explicitly

This repository already contains a complete treatment of the low-latency discipline. This document is a language companion to it, and the table below is the contract: where a sibling owns a topic, this guide cross-references it by path and section number and **does not re-derive it**.

| Topic | Owner | Owner's sections | This guide's treatment |
|---|---|---|---|
| The discipline itself: what low-latency development is, the latency hierarchy, latency vs throughput, jitter, determinism engineering | C/C++ guide | `low_latency_cpp_development_guide.md` §1 | Cross-reference only |
| The latency budget model, decomposition, percentile SLOs, measurement infrastructure | C/C++ guide | `…§2` | Cross-reference; §13 of this guide describes how a JVM workload *fits* such a budget, not how to build one |
| Hardware fundamentals: caches, cache lines, NUMA, TLB, huge pages, data-oriented design | C/C++ guide | `…§3` | Cross-reference; §6 of this guide covers only how the JVM *exposes or hides* those effects, and §10 the JVM flags that touch them |
| C++ language fundamentals for latency | C/C++ guide | `…§4` | Not applicable; replaced by §2–§6 of this guide |
| Concurrency, the C++ memory model, fences, lock-free queues | C/C++ guide | `…§5` | Cross-reference; §6 of this guide covers the Java Memory Model's *weaker* guarantees and the `java.util.concurrent` / `VarHandle` supply |
| The single-threaded event loop, `epoll`, `io_uring`, busy-polling | C/C++ guide | `…§6` | Cross-reference; §8 of this guide covers only what changes on the JVM |
| Networking, kernel bypass, RDMA, TCP tuning, NIC features | C/C++ guide | `…§7` | **Owned by the sibling.** §8 of this guide covers only the JVM-layer consequences: threading model, direct buffers, collector interaction |
| Message parsing, ITCH/SBE/FAST, fast parsing principles | C/C++ guide | `…§8` | Cross-reference; this guide notes only the JVM-side allocation consequences of the same parsers |
| The trading reference architecture | C/C++ guide | `…§9` | Cross-reference; §13 of this guide maps those components onto the paths where a JVM is the right runtime |
| Shared memory and IPC; FPGA and hardware acceleration | C/C++ guide | `…§10`, `…§11` | Cross-reference; §7 of this guide covers only the JVM's off-heap and memory-mapping libraries, and notes that FPGA vendor toolchains are not JVM projects |
| Profiling, tracing and benchmarking *practice*; clock sources, histograms, percentiles, outlier analysis | C/C++ guide | `…§12`, `…§13` | **Owned by the sibling.** §9 of this guide covers only the JVM-side harnesses and the specific traps a managed runtime adds |
| Build/tooling philosophy, the optimization workflow, pitfalls, real-world numbers | C/C++ guide | `…§14`–`…§17` | Cross-reference; §10 of this guide covers only JVM flags, §15 the *JVM-specific* anti-patterns |
| Low latency in banking: MiFID II RTS 6, MAS guidance, the economics of microseconds, over-engineering | C/C++ guide | `…§18` | Cross-reference; §13 of this guide covers only which bank paths are JVM-shaped and which are not |
| Practical exercises and projects | C/C++ guide | `…§19` | Not duplicated |
| The Rust delta: no collector, ownership as aliasing discipline, the crate ecosystem, the FFI seam | Rust companion | `low_latency_rust_programming_guide.md` §1–§17 | Cross-reference; §11 of this guide compares the JVM's AOT answer to Rust's ahead-of-time compilation story |
| GenAI serving latency: TTFT, TPOT, KV cache, batching, RAG | GenAI guide | `low_latency_genai_patterns_guide.md` §1–§12 | Cross-reference only; that guide owns the model-serving budget, not the JVM's |
| The reactive angle: operators, schedulers, backpressure as an API model | RxJava guide | `rxjava_guide.md` | Cross-reference only; §7 of this guide notes that a reactive scheduler is a thread allocator, not a latency strategy |
| The Java module platform: bundles, module layer, service layer, classloading | OSGi guide | `osgi_guide.md` | Cross-reference only; the JVM's own module work (Project Leyden's AOT cache, §11) is *not* OSGi's subject |
| Accelerator telemetry and GPU workloads | GPU / DCGM guides | `gpu_optimization_guide.md`, `nvidia_dcgm_guide.md` | Cross-reference only; a GPU is a throughput device and the sibling's §11.6 framing is the relevant one |
| **The managed-runtime delta: JIT and warmup, the collector, the Java Memory Model, the allocation-rate lever, the off-heap toolkit, JVM measurement and flags, AOT/Leyden/native image, the alternative JVMs** | **This guide** | — | §2–§12 |

**Disclaimed overlap.** Where this guide appears to restate a mechanism from the C/C++ sibling — the latency hierarchy, cache-line behaviour, tail statistics, the trading reference architecture, kernel bypass — treat the sibling as authoritative and this guide as a JVM annotation. Nothing here supersedes it.

### 1.2 The decoder

The vocabulary of JVM latency is small and load-bearing. These are the terms used without further definition for the rest of the document.

| Term | What it actually is | Why it matters here |
|---|---|---|
| **JIT (just-in-time compiler)** | The runtime component that translates bytecode into machine code *while the program runs*, using profile data it has collected so far | Your hot loop's machine code is decided at run time from evidence about the calls it has already seen. §3 |
| **Tiered compilation** | The JVM's staged path from interpreted execution, through a lightly optimising compiler that inserts profiling, to a heavily optimising compiler that uses the resulting profile | Explains why latency is a function of *how long the process has been running*. §3 |
| **Deoptimisation** | Discarding previously compiled machine code and falling back to a lower tier because an assumption the compiler made turned out false | A single deoptimisation can convert a microsecond path into a millisecond path, without any code change. §3 |
| **The profiler** | The runtime's own instrumentation layer: the counters and type feedback the JVM collects to drive compilation, plus the external tools that read JFR and the compilation log | The JVM is already instrumented; the question is whether you can read it. §3, §9 |
| **Collector families** | The selectable algorithms for reclaiming unreachable objects: Serial, Parallel, G1, ZGC, Shenandoah, Epsilon | The single largest runtime-owned latency risk. §4 |
| **The pause** | Any interval during which application threads cannot make progress because the runtime has stopped them (stop-the-world) or is holding them off a critical structure | The tails. §4 |
| **Allocation rate** | Bytes allocated per unit of time, and the distribution of how long those objects live | The lever the *application* controls, which determines how often the collector must act. §5 |
| **Java Memory Model (JMM)** | The specification of which values a thread may observe and in what order, given unsynchronised access | Weaker than the source suggests; the fix is explicit, and explicit costs something. §6 |
| **False sharing** | Cache-line invalidation traffic caused by two cores writing *different* variables that happen to share a cache line — the sibling's §3.2 mechanism, in a JVM setting | A JVM has an annotation for this, and it is off by default for application classes. §6 |
| **Off-heap** | Memory outside the garbage-collected Java heap, obtained from the OS (direct buffers, memory mapping, or the Foreign Function & Memory API) | Trades collector pressure for manual lifetime management. §5, §7 |
| **Warmup** | The interval between process start and the point at which the JIT, the class-loading subsystem and the runtime have reached their steady state | The operational consequence: a deploy is not at its steady-state latency. §3, §13 |
| **JEP** | A JDK Enhancement Proposal: the numbered design record for a change to the platform, with a Status and a Release field | The primary source for every platform claim in this guide. §12, §15 |

### 1.3 One-page orientation

If you read only this page, read this:

- **Java's biggest latency property is positive and structural**: there is no unbounded stop-the-world phase left in the modern collector set. Both concurrent collectors — ZGC since JDK 15 and Shenandoah since JDK 15 — have been production features for years, and the generational versions of both are either delivered or delivered-in-product. The correct statement is *bounded*, not *zero*.
- **Java's biggest latency *risk* is also structural, and it is a scheduling problem**: the collector, not the CPU, decides when your threads stop. A native stack has no component that can take the CPU away at a moment the application did not choose.
- **Java's second risk is a compilation problem**: the code that runs at second 3 is not the code that runs at second 300, and the code that runs at second 300 can be replaced by worse code at second 301 if an assumption breaks. §3.
- **The lever that actually moves the pause is allocation, not tuning.** A collector's pause is largely a function of the allocation the application makes and the lifetime of those objects. Tuning a collector to absorb an application's allocation rate is treating the symptom.
- **Measurement is not optional and not portable.** A hand-rolled Java benchmark without a warmup harness measures the compiler. The JVM ships the harness (JMH) precisely because this is a known trap. §9.
- **The platform is moving fast, which is both an opportunity and a governance cost.** Six-month feature releases; LTS every two years (21, 25, next 29); JDK 27 is current as of six days before this verification date. Anything you read about the JVM that is more than two years old is probably about a flag that no longer exists — §4.4 is four worked examples of exactly that.
- **The allocation rate is the number to internalise.** It is the application's own lever over the collector's behaviour, it is measurable, and it is the number that most JVM latency advice skips past on its way to a tuning flag.

---

## 2. Why Java Is a Different Problem

A native low-latency system's adversary is the machine: the cache, the TLB, the kernel, the network. Those adversaries are **absent from your control but constant in their behaviour** — a cache miss costs what it costs, and the sibling guide's §3 teaches you to avoid it. A managed runtime adds three adversaries that are **not constant, not visible in the source, and in two cases under your partial control**.

### 2.1 The three adversaries

| Adversary | What it is | Why it costs latency | What control the practitioner actually has |
|---|---|---|---|
| **The JIT** | Machine code is generated at run time from profiles collected so far, in tiers, and can be regenerated | A latency figure measured before steady state measures the compiler. A profiling tier is *slower* than an optimised tier, sometimes by a factor, and the transition is not instant. Assumptions can break and code can go backwards | Indirect and largely structural: keep the hot path monomorphic and stable so the compiler's guess is right and stays right; warm the process before it takes traffic; measure in steady state and also *measure the warmup itself* |
| **The collector** | A runtime subsystem that reclaims memory on its own schedule, using the application's threads or dedicated worker threads | Stop-the-world phases are unbounded from the application's point of view. Allocation stalls are a second, less-discussed mechanism: when a thread cannot allocate, it waits for the collector | Direct, in three levers: choose a collector appropriate to the pause requirement; reduce allocation rate and object lifetime so the collector has less to do; make the heap large enough that the collector is not under pressure |
| **The memory model** | A specification of permitted observations across threads that is *weaker* than sequential consistency | Code that looks correctly ordered in the source can be reordered by the compiler or the CPU; the bug appears as a stale read under load, and the fix is a synchronisation primitive with a real cost | Direct and explicit: `volatile`, the `java.util.concurrent.atomic` classes, `VarHandle` access modes. The discipline is to make every cross-thread assumption explicit, then measure the primitives you chose |

### 2.2 Why these are "different" and not merely "additional"

The sibling covers lock contention, allocation and cache behaviour for a native stack, and every one of those hazards exists in Java too — that material is owned by the sibling and is not repeated here. What is *specific to the JVM* is the shape of the failure:

- **The failure is time-dependent.** A native stack's latency profile is roughly stationary once the binary is running. The JVM's is not: it improves, and it can regress. "It was fast in the load test" and "it is slow in production" can both be true of the same binary.
- **The failure is not in the diff.** A code change that is latency-neutral can move the collector's behaviour by changing object lifetimes, or move the JIT's behaviour by adding a second implementation of an interface to a call site. Code review does not see either.
- **The failure is a configuration failure as often as a code failure.** The JVM's defaults are chosen for a broad population, not for your workload. Two applications running identical code with different flags can differ by an order of magnitude in tail latency.
- **The control surface is a specification, not an implementation.** You can read the JMM and the collector's JEP; you cannot read your vendor's build of the JIT the way you can read your own C++. This inverts the sibling's central practice of reading the generated assembly — you *can* read the JIT's log, and §3 and §9 say how, but the machinery producing it is not yours.

The third adversary arrives with a toolkit: because the memory model is specified rather than incidental, the platform ships a concurrency library whose correctness argument is written down, and §7 covers the parts of it that are appropriate for a latency-sensitive path. The parallel point about measurement is §9.

### 2.3 The control surface, summarised

The layers differ as §14.5's checklist sets out. Generated code, memory reclamation, cross-thread ordering, heap sizing and threads are yours in a native stack and either the runtime's or a flag's on the JVM; the kernel and network path beneath them is the same substrate the sibling's §6 and §7 own. And every section that follows answers one question: **what can I decide, and what has the runtime already decided for me?** Where the answer is "nothing — that is the collector's business", the guide points at the lever that *is* yours (allocation rate, §5); where it is "a flag, and the flag has a gate", it gives the exact syntax verified against the JDK source and names the gate (§10); where it is "a specification weaker than people assume", it states the assumption and the primitive that discharges it (§6).

---

## 3. The JIT and the Warmup Problem

This is the most transferable idea in this guide, because it is the part of the JVM's behaviour that a native-stack engineer has no prior model for. Read it even if you skip the collector section.

### 3.1 The mechanism: tiered compilation

The JVM does not compile your code once. It executes bytecode in an interpreter at first, collecting evidence about which methods run and with what types, and promotes methods to progressively more aggressive compilers as that evidence accumulates. The path has four conceptual stages:

| Stage | What runs | What the runtime learns | Latency characteristic |
|---|---|---|---|
| **Interpreted** | The bytecode interpreter | Which methods are hot; which call sites exist | Slowest stage, but each invocation also *collects* data |
| **Lightly optimising, with profiling** | A fast compiler that inserts counters and type checks into the generated code | Types observed at each call site; branch and loop frequencies; whether inlining candidates are monomorphic | Faster than interpreted, still slower than the final tier — the profiling instrumentation costs |
| **Lightly optimising, without profiling** | The same fast compiler with profiling removed | Nothing further | Useful for short-lived methods that will never get hot enough for the next stage |
| **Heavily optimising** | The aggressive compiler, consuming the profile | — | The steady-state code. Inlining, devirtualisation, loop transformations, escape analysis |

Two consequences follow immediately, and both are operational rather than theoretical:

1. **Latency is a function of process age.** The same request, issued at second 5 and at second 500, traverses different machine code. A latency budget validated on a warm process is not a validated budget for the first minutes after a deploy.
2. **The hot path is the *only* thing the aggressive tier sees.** Cold paths stay interpreted or lightly compiled — which is usually fine, and is a real cost when a path that is cold in normal operation becomes hot exactly when something has gone wrong (a recovery loop, a snapshot rebuild, a failover path). An error path that was never warmed is an error path at interpreter speed.

### 3.2 Deoptimisation: when the compiler's guess was wrong

The aggressive compiler optimises against *assumptions* derived from the profile. The canonical example: a call site that has only ever seen one implementation of an interface is compiled as a direct call, with a check. When a second implementation arrives, that assumption is false. The JVM does not simply keep a slow version — it **deoptimises**: it discards the compiled frame, falls back to a lower tier (typically the interpreter or the profiling tier), and recompiles later from the corrected profile.

The cost is twofold and the second part is the one people miss:

- **The immediate cost**: the executing method continues in the interpreter, at interpreter speed, in the middle of a live request.
- **The indirect cost**: the *recompilation* is queued behind other compiler work, so the method runs in a deoptimised state for a window that is not bounded by anything in your code. A deoptimisation storm — many call sites invalidated by one change in behaviour — turns a steady-state microsecond path into a millisecond path until the compiler catches up.

What makes this dangerous for a latency-sensitive system is that the trigger is usually an *uncommon* event arriving: a new message type on the feed, an admin operation, a failover causing a different connection class to be used, a JSON or log payload shape that was never seen at the warmed call site. The tail is exactly where deoptimisation lives — a p99.9 event can be a deoptimisation, not a cache miss.

### 3.3 Profile pollution

The profile belongs to the *call site*, not to your method, and it is accumulated over the life of the process from every caller that reaches it. This gives a failure mode with no analogue in a compiled language:

> An innocent early call — a metrics call, a startup-time enumeration, a serialisation framework asking "what type is this?" — can permanently teach the compiler that a hot call site is megamorphic, and it will then decline to inline or devirtualise that call site for the rest of the process's life.

The mechanism is worth being precise about, because it is the guide's central claim about the JIT: the compiler's optimisations are *global* to a call site and *cumulative*, so the profile's history is an input to the code your hot path runs. A hot loop does not merely get compiled — it gets compiled **using evidence contributed by everything else that ever called into it.** Contaminating evidence (a benchmark harness that runs before the service starts, a debug or tracing hook, a generic framework call in an initialisation path) is never washed out.

The practical guardrails:

- **Keep hot call sites monomorphic by construction.** Avoid interface indirection on the hot path where a concrete type or a sealed hierarchy will do; the sibling's §4.5 argument about virtual calls applies here with an extra twist, because the JVM's decision is *empirical* rather than declared.
- **Keep logging and metrics off the hot path's call graph**, not merely wrapped in a level check. A call that never executes still contributes to the profile if it executes *sometimes*.
- **Do not let a benchmark JVM be a production JVM** (and vice versa) — the profile is part of the runtime state.
- **Understand that the aggressive tier's decisions are visible.** The compilation log records inlining, devirtualisation, and "made not compilable"/"uncommon trap" events; §9 covers the tools that read it. This is the JVM's answer to reading the assembly in the sibling's §14 workflow.

### 3.4 Why a benchmark without a warmup harness measures the compiler

If you time "the first N executions" of a JVM method, you have measured a mixture of interpretation, profiling-compiled code, compilation pauses, class-loading, and finally the optimised code — in proportions that depend on N. Change N and the answer changes. Two further traps compound it:

- **Dead-code elimination of the entire measurement.** The heavily optimising tier is entitled to delete computations whose results are never observed. A microbenchmark whose result is unused can end up timing an empty loop. This is not a JVM bug; it is a correct optimiser taking a correct inference.
- **Constant folding of the inputs.** A loop over constants can be folded, so the measured cost is of something the compiler decided was invariant.

This is why **JMH exists**. JMH is the OpenJDK project's benchmarking harness, and its design purpose is precisely to neutralise these mechanisms: it runs the workload in a separate JVM, it runs warmup iterations before measurement iterations, it consumes the result in a way the optimiser cannot remove, and it reports the run-to-run distribution rather than a single mean. Its current status is verified in §9 (tag 1.37, last repository push 2026-07-14, active). The rule that follows is blunt and correct: **in Java, a hand-rolled benchmark is a measurement of the compiler, not of your code.**

### 3.5 Why the first requests after a deploy are not the steady state

Put §3.1–§3.4 together and the operational picture is unavoidable:

| Time after process start | What dominates | What a naive dashboard shows |
|---|---|---|
| 0–seconds | Class loading, verification, static initialisation, JVM bootstrapping | Latency far above steady state |
| Seconds–minutes | Interpretation and the profiling tier; the collector has touched a cold heap and no pages are pre-touched | A latency curve that falls steeply — usually dismissed as "JIT warmup" |
| Minutes–tens of minutes | The aggressive tier has compiled the main paths; the profile is still stabilising | Near steady state |
| Steady state | Optimised code; the profile is stable until something new arrives | The number you actually want to publish |
| Any later moment | A new message shape, a failover, an admin action → possible deoptimisation | An unexplained p99.9 spike, attributed to "the network" |

Two configuration consequences follow, and §10 gives the flags for both: the heap should be **pre-touched** at startup so that the warmup interval is not also paying for page faults, and the code cache the compiler writes into should be **sized deliberately** so that the process does not spend its warmup on compilation and then exhaust its space for compiled code — after which the JVM must stop compiling, and can even discard compiled methods, which is a latency cliff rather than a curve.

### 3.6 What a production warmup strategy actually involves

Five things, in order of value:

**Warm the process before it serves traffic** — replay a captured sample of real traffic (the sibling's §12 practice applies directly) or run an in-process synthetic driver against the real code paths, in the same JVM and configuration, so the *right* methods reach the aggressive tier before the first paying request. **Warm the error and recovery paths too** — they were never hot, they handle the incident, and this is the most commonly skipped step. **Keep the warmed state across restarts where the platform allows it** — Project Leyden's AOT work is the platform's own answer (§11): an AOT cache moves class-loading and profiling cost from deploy time to build time, and does not remove the need to think about warmup. **Make the load balancer aware** — take the warming instance out of the pool, warm it, verify a latency target with a probe, then admit it; a rolling restart that admits each new instance immediately is a daily scheduled tail-latency incident. And **measure the warmup itself**: time-to-steady-state is a first-class metric, because it is what sets the blast radius of a deploy, a failover or a restart under load.

### 3.7 The honest limits

You cannot control the JIT the way you control a compiler invocation: there is no `-O2` you can trust to mean the same thing across two JDK builds, and the vendor's implementation choices are not yours to audit. What you *can* do is shrink the space in which the heuristics make a bad decision — monomorphic call sites, stable types, no instrumentation in the hot call graph, warm before serving, and a log you actually read when the tail moves. That is the whole of the practitioner's control over the first adversary, and more than most teams exercise.

---

## 4. Garbage Collection

This is the section the guide exists for. Everything else in the document is either preparation for it or verification of it.

### 4.1 The pause taxonomy

Java's latency literature tends to discuss "the GC pause" as a single thing. It is at least four things, and only the first is what a collector's design goals are about.

| Kind | Mechanism | Who schedules it | Typical visibility |
|---|---|---|---|
| **Stop-the-world phase** | Application threads are parked while the runtime completes work that cannot be concurrent (root scanning, certain handshakes, weak-reference processing in some collectors) | The collector | The headline "pause". Its *duration* is what a collector's goals quantify |
| **Allocation stall** | A thread cannot allocate because the collector has not yet reclaimed or committed memory, so the runtime blocks the allocator until it can | The collector, *triggered by* the application's allocation rate | Often confused with a slow method. It is Java's most under-diagnosed latency event |
| **Concurrent-cycle CPU competition** | Collector worker threads compete for cores and memory bandwidth with application threads | The collector, continuously | Not a pause at all — a *broadening* of the latency distribution while a cycle runs |
| **Non-GC pauses with the same signature** | Time-to-safepoint (getting *all* threads to a point where the runtime may act on them), class unloading, deoptimisation, biased-style locking pathologies | The runtime, not the collector | Measured as latency; attributed to the collector; fixed by neither collector choice nor heap tuning |

The last row is not academic. Shenandoah's own design document states it explicitly: its non-goal is stated as *"The goal is not to fix all JVM pause issues. Pause times due to reasons other than GC like Time To Safe Point (TTSP) issues or monitor inflation are outside the scope of this JEP."* (JEP 189). A team that buys a concurrent collector and still sees multi-millisecond outliers has usually bought the wrong fix for a safepoint problem.

### 4.2 The collector families

| Collector | Selection | Shape | What it is for |
|---|---|---|---|
| **Serial** | `-XX:+UseSerialGC` | Single-threaded, stop-the-world | Small heaps and constrained environments: the collector of last resort when CPU or memory is scarcer than latency tolerance |
| **Parallel** | `-XX:+UseParallelGC` | Multi-threaded, stop-the-world | Throughput on multi-core hosts, where pause duration matters less than work per unit time |
| **G1** | `-XX:+UseG1GC` | Region-based, generational, concurrent marking with incremental evacuation; pause-target driven | The general-purpose default since JDK 9 (JEP 248) and, since JEP 523 (JDK 27), the default in *all* environments |
| **ZGC** | `-XX:+UseZGC` | Concurrent, region-based, compacting, NUMA-aware; generational since JDK 23 | Very large heaps where a sub-millisecond pause goal is a functional requirement |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Concurrent, region-based, compacting; generational mode available | Low pause times independent of heap size, with a different engineering trade from ZGC |
| **Epsilon** | `-XX:+UseEpsilonGC` (and it needs an unlock) | Allocates; never reclaims | Not a production collector. A *measurement* instrument: it establishes the no-GC latency baseline for a workload, which is exactly how JEP 318 proposes it be used |

Selection caveat first, because it is the most common way to lose an afternoon: **`-XX:+UseEpsilonGC` is an EXPERIMENTAL flag in the JDK 27 source**, so it requires `-XX:+UnlockExperimentalVMOptions` first. The other five are ordinary product flags and need no unlock. Everything in that sentence was read out of the JDK 27 source tree, not from a blog.

### 4.3 The verified availability and support gates

This table is the section's factual core. Each row was verified at the collector's own JEP record. **"Default" means the JVM selects it with no flag.**

| Collector | Available since | Production support since | Default? | Status as of 21 September 2026 |
|---|---|---|---|---|
| **Serial** | Pre-JDK 8 | Long-standing | Was the default in constrained environments until JDK 26; G1 took over in JDK 27 | Still supported. JEP 523 explicitly does **not** deprecate it |
| **Parallel** | Pre-JDK 8 | Long-standing | Never the general default | Still supported and improved: JEP 522 (JDK 26), "G1 GC: Improve Throughput by Reducing Synchronization" is the neighbouring throughput work |
| **CMS** | Pre-JDK 8 | Long-standing | Was the default before JDK 9 | **Removed.** Deprecated by JEP 291, removed by JEP 363 — Status Closed/Delivered, **Release 14** |
| **G1** | JDK 7 (JEP 248 context) | Long-standing | **Default since JDK 9** (JEP 248); JEP 523 (JDK 27, Delivered) makes G1 the default in *all* environments | Actively developed: JEP 475 "Late Barrier Expansion for G1" (JDK 24), JEP 522 (JDK 26) |
| **ZGC** | JDK 11, as an experimental feature (JEP 333, Release 11) | **JDK 15**, product (JEP 377, Release 15) | Not the default; G1 remains | Generational mode is the *only* mode: JEP 439 made generational ZGC available (JDK 21), JEP 474 made it the default (JDK 23), JEP 490 removed the non-generational mode (JDK 24) |
| **Shenandoah** | JDK 12, experimental (JEP 189, Release 12) | **JDK 15**, product (JEP 379, Release 15) | Not the default | Generational mode is a product feature: JEP 404 introduced it experimentally (JDK 24), JEP 521 made it product (JDK 25, Delivered). JEP 535 will make generational the default — Status *Targeted*, Release 28 |
| **Epsilon** | JDK 11 (JEP 318, Release 11) | Never — it is a diagnostic/experimental collector by design | Never | The flag is still EXPERIMENTAL in the JDK 27 source |

### 4.4 Four stale claims, corrected

The half-life of JVM latency advice is short, and these four errors are in circulation. Each correction below was verified against primary sources on 21 September 2026.

**Stale claim 1 — "add `-XX:+ZGenerational` to get the generational collector."** That flag does not exist in JDK 27. JEP 474 deprecated it in JDK 23 and JEP 490 obsoleted it in JDK 24; the identifier `ZGenerational` does not appear at all in the JDK 27 source file that defines ZGC's options (`gc/z/z_globals.hpp`). Passing it to a JDK 24-or-later JVM produces an **obsolete-option warning**, and the JVM continues. The correct modern invocation is simply `-XX:+UseZGC`: under JDK 24 and later that *is* the generational collector. A team that copied the old two-flag incantation from a tuning guide is running defaults while believing otherwise.

**Stale claim 2 — "ZGC is experimental."** ZGC was experimental in JDK 11 and nothing else. It became a product feature in **JDK 15** via JEP 377 (Release 15), four and a half years before this document. The misstatement usually comes from reading JEP 333's title — "A Scalable Low-Latency Garbage Collector (Experimental)" — and stopping there.

**Stale claim 3 — "Shenandoah's generational mode needs the experimental unlock."** True for one release: in JDK 24 it required `-XX:+UnlockExperimentalVMOptions -XX:ShenandoahGCMode=generational` (JEP 404). JEP 521 (JDK 25) made it a **product** feature, so the unlock is gone. What remains true and is easy to get wrong: in the JDK 27 source the default of `ShenandoahGCMode` is still the literal string `"satb"`, so generational mode is available and supported but **not yet the default** — that is JEP 535 in JDK 28 (Status Targeted as of this date).

**Stale claim 4 — "mark your field with `sun.misc.Contended`."** That identifier could not be located anywhere in the OpenJDK tree at any tag probed — `jdk-11-ga`, `jdk-17-ga`, `jdk-21-ga`, `jdk-25-ga`, `jdk-27-ga` — including the two plausible paths for it. Control probes against files that certainly exist at those tags succeeded, so the probe method is valid. The JDK's own annotation is different and is documented in-source as `@since 1.8`: **`jdk.internal.vm.annotation.Contended`**. The JDK uses it on itself — `ConcurrentHashMap` declares `@jdk.internal.vm.annotation.Contended static final class CounterCell`. Treat any guide recommending `sun.misc.Contended` as a signal to check that guide's other claims. See §6.4 for the gating that decides whether the annotation does anything.

### 4.5 The development line of the concurrent collectors

The concurrent collectors are not standing still, and the direction of travel matters for a team choosing one. Each change below is a numbered JEP with a status, verified at its own record.

| Collector | JEP | Release | Status | What it did |
|---|---|---|---|---|
| **ZGC** | **JEP 439** Generational ZGC | 21 | Closed / Delivered | Introduced young/old generations, so that young objects — which die young — are collected more frequently and cheaply |
| **ZGC** | **JEP 474** Generational Mode by Default | 23 | Delivered | Made generational the default mode; deprecated `-XX:+ZGenerational` |
| **ZGC** | **JEP 490** Remove the Non-Generational Mode | 24 | Delivered | Deleted the old mode entirely, which is why `-XX:+UseZGC` now means one thing |
| **Shenandoah** | **JEP 404** Generational Shenandoah (Experimental) | 24 | Delivered | Introduced generational mode behind `-XX:+UnlockExperimentalVMOptions -XX:ShenandoahGCMode=generational` |
| **Shenandoah** | **JEP 521** Generational Shenandoah | 25 | Closed / Delivered | Turned it into a product feature — no unlock needed |
| **Shenandoah** | **JEP 535** Generational Mode by Default | 28 | **Targeted** | Will make generational the default; **not yet delivered** as of 21 September 2026 |

The pattern is identical in both: a concurrent collector starts single-generation (all pauses bounded, but the collector pays to scan everything), gains generations (young objects collected cheaply, fewer allocation stalls, less CPU), then the non-generational mode is removed because maintaining two is not worth it.

### 4.6 What the sources actually claim about pause time — and whose claim it is

No latency figure appears in this guide that its source does not state, and every figure is labelled with whose claim it is. Two are quotable because they are in the collectors' own specification documents:

- **JEP 439 (Generational ZGC)**, in its Goals: *"Pause times should not exceed 1 millisecond"*, for heaps ranging *"from a few hundred megabytes up to many terabytes"*. Cite this as the JEP's goal, not as a measurement.
- **JEP 516 (AOT Object Caching with Any GC, JDK 26)**, describing ZGC: *"ZGC reclaims memory concurrently, never pausing application threads for more than a millisecond."* Again: the specification's own claim about the collector's design, not a promise about your workload.

JEP 439 also carries the only quantified example anywhere in this guide's sources, and it is worth reading slowly because of how easily it is misquoted. It reports an **Apache Cassandra** benchmark in which a generational configuration *"…a quarter of the heap size yet achieves four times the throughput compared to non-generational ZGC, while still keeping pause times under one millisecond"*. Three things are true about that sentence: it is from the JEP; it is about Cassandra, not about a trading system; and it says a quarter of the heap achieved four times the throughput — a result about *memory overhead and throughput*, with pause times held constant. Anyone citing it as "ZGC is four times faster than G1" has misread it.

### 4.7 The honest statement

**A collector's advertised pause is not the pause your workload will see.**

Every number in §4.6 is a goal or a property of the algorithm under conditions the collector's authors chose. The pause your application experiences is a function of at least five things the JEP does not know:

**Your allocation rate and object lifetime distribution** — the dominant term, and the one §5 says is your lever. **Your live-set size and the shape of the reference graph** — tracing a deep, wide graph costs more, and reference processing is explicitly outside generational ZGC's goals. **Your host's core count and memory bandwidth** — a concurrent collector needs cores to run concurrently with you, and a CPU-throttled container turns "concurrent" into "contended". **Safepoints and non-GC pauses** — JEP 189's non-goal is the standing warning that TTSP and monitor inflation are not collector problems. And **the rest of the process** — deoptimisation, class unloading, JNI transitions, and the OS jitter the sibling's §1.4 owns.

The correct posture is therefore: use the JEP figures to *shortlist* a collector, and use your own histogram, taken under your own load, over your own restarts, to decide. §10 gives the configuration and §9 the tools.

---

## 5. The Allocation Discipline

If §4 establishes that the collector is the second adversary, this section establishes the counter-intuitive consequence: **the collector's behaviour is mostly a function of decisions you make in ordinary application code, not of the collector's settings.**

### 5.1 The lever, restated

Three relationships determine how much work you hand the collector:

| Relationship | Why it holds | What it means for you |
|---|---|---|
| **Faster allocation → more frequent collection** | The collector's cycle is triggered by the heap filling, and the heap fills at your allocation rate | Halving the allocation rate roughly halves how often the collector has to act — which halves how many opportunities there are for a cycle to coincide with a latency-critical moment |
| **More garbage → more work per cycle** | A collection's cost scales with what it must trace, promote and copy | A per-message temporary is not free because it is "cheap"; it is a tax paid later, by a component you are not watching |
| **Longer-lived objects → promotion and old-generation pressure** | Objects that survive a young collection are promoted, and the old generation is collected less often and more expensively | A cache holding everything forever is not a cache; it is old-generation pressure that delays the collection that would have been cheap |

This is why the correct order of operations is **reduce allocation, then choose a collector, then tune it** — and why §15's anti-pattern list contains "the collector swapped without measuring the allocation rate" as its own entry. A collector swap is a one-line change that produces a measurable difference; an allocation-rate reduction is a code review; the second is where the durable improvement lives.

There is a wider truth the sibling already states and this guide only annotates: allocation is *cumulative* cost, and cumulative cost lands on the tail, because the tail is where an unlucky capacity growth, rehash or promotion coincides with a latency-critical request.

### 5.2 Reading an allocation profile before changing anything

Do not reason about allocation by reading code; Java's allocation sites are not visibly signposted and even a careful reader is wrong most of the time. Measure it, then fix what the measurement names.

- **allocation profiling** (async-profiler, §9.4) attributes allocation *bytes and counts* to call stacks. This is the single most valuable JVM-specific measurement in the whole discipline, because it converts "the collector is misbehaving" into "this method allocates 40% of the traffic".
- **Collector logging** (the `-Xlog:gc` family, §9.5 and §10) tells you whether the collector is actually the problem, and whether the pressure is young-generation, old-generation or allocation stalls. If the collector log shows no collection during the incident, the collector is not your cause — go to §3.2 and §4.1's fourth row.
- **The allocation rate itself is a publishable number.** Bytes allocated per second, and the mean/max lifetime of the objects allocated, are as much a part of a latency service's specification as its request rate. They are also the numbers that change silently when a dependency is upgraded.

### 5.3 Zero-allocation patterns

The patterns below are ordered by leverage. Each is a statement about *shape*, not about a library.

| Pattern | Mechanism | Where it applies |
|---|---|---|
| **Reuse a mutable buffer per thread** | Own one scratch object per thread (or per event-loop stage) and reset it rather than allocating a new one per message | Parsing, encoding, serialisation, any per-message working set |
| **Pre-size collections** | Avoid the growth path: a collection that grows reallocates and copies, and a map that grows rehashes | Any container with a knowable upper bound |
| **Write into a pre-allocated result** | Pass an "into" target instead of returning a newly built object | Serialisation, book construction, response building |
| **Slice instead of copy** | Hand out offsets and lengths into an existing region rather than materialising a new array or string | Parsing pipelines, quote and order-book views |
| **Primitive-typed layouts** | Parallel primitive arrays instead of arrays of boxed objects | Order books, price levels, per-instrument state — the sibling's §3.6 data-oriented argument, with the JVM's boxed-type penalty added |
| **Avoid per-call formatting** | A log line, a metrics label or a `String` concatenation allocates whether or not the level is enabled, unless the framework defers it | Every logging and metrics call in a hot call graph |
| **Avoid implicit boxing** | `Integer`/`Long`/`Double` in a generic collection, a lambda capturing a primitive into an object, a `Stream` of primitives | Any generic/JVM-collection interface on a hot path |
| **Avoid implicit iterators** | An enhanced `for` over a collection allocates an iterator unless the code is trivially optimisable; an index loop over an array does not | Loop-heavy hot paths |

Two JVM-specific observations on that table. First, the **boxing** row is where Java's ergonomics cost most: a design that reads beautifully with `Map<String, List<Order>>` can allocate three objects per lookup. Second, the **iterator** row is the one escape analysis most often rescues — see §5.4 — so it is a "measure before rewriting" row rather than a "rewrite on sight" row.

### 5.4 Escape analysis, and the common cases where it does not save you

The heavily optimising compiler performs **escape analysis**: it determines whether an allocated object can be observed outside the method that created it. If it provably cannot, the compiler may eliminate the allocation entirely (scalar replacement of the would-be object into registers and stack slots) or perform lock elimination on it. This is why a great deal of idiomatic Java allocates nothing at all in steady state.

Two honest qualifications, both important:

- **It is an optimisation, not a guarantee.** Nothing in the Java Language Specification requires escape analysis, and no configuration flag makes it certain for a given method. A pattern that allocates nothing on one JDK build may allocate on the next, because an inlining decision changed. `@HotSpotIntrinsicCandidate`-style contracts do not exist for this. So the only trustworthy statement about escape analysis in your service is an allocation profile (§5.2), not an argument.
- **It fails in well-known shapes.** An object escapes if it is stored in a field, returned, passed to a non-inlined method, captured by a lambda that escapes, thrown (a stack trace is an object graph), placed into a collection, or synchronised on in a way the compiler cannot eliminate. Iteration over a collection often *is* eliminated because the iterator's entire life is inside an inlined loop; the same iteration inside a lambdas-and-streams pipeline frequently is not, because the pipeline's intermediate objects cross non-inlined boundaries. And an exception thrown on the hot path allocates a stack trace — which is the JVM-side echo of the Rust companion's §3.5 argument about panic machinery, except here the cost is an allocation rather than unwind tables.

**Practical rule:** treat escape analysis as a gift that arrives when your code is simple enough for the inliner to see through it, and never as a budget line you have not measured.

### 5.5 Object pooling and its documented dangers

Pooling is the obvious answer to an allocation problem and the wrong default answer. The generational hypothesis on which every modern collector is built says that most objects die young, and young collections are cheap precisely *because* most objects are garbage. A pool fights that design in four ways:

**It trades cheap young collections for expensive old-generation pressure** — pooled objects are long-lived by construction, and the old generation is where the collector is least able to be cheap. **It leaks state** — an object whose release path forgot one field is a latency bug *and* a correctness bug, and it will be intermittent (§15). **It reintroduces contention** — a shared pool is a shared mutable structure, so you have replaced the allocator's problem with yours. And **it hides the real fix** — pooling lets a team keep the per-message allocation *pattern* and pay a different cost for it, where §5.3 changes the pattern instead.

**When pooling is right:** when the object is genuinely expensive to create (a large primitive array, a direct buffer, a native handle), when the pool is **per-thread** so that it needs no synchronisation, when the lifetime is naturally bounded by a request or a message, and when a reset-and-verify test exists. In that shape, the pool is a resource cache, which is a different and defensible thing.

### 5.6 The off-heap route

If the heap is the problem, one honest answer is to use less of it. Memory outside the garbage-collected heap is not traced, not moved, and does not contribute to the cycles §5.1 describes.

| Route | Mechanism | Cost |
|---|---|---|
| **Direct `ByteBuffer`** (`ByteBuffer.allocateDirect`) | OS-allocated buffer addressable from Java; not the heap | Manual lifetime. The buffer is freed when collected, so a leaked direct buffer consumes native memory until the collector happens to notice it; there is no "free" call |
| **Memory-mapped files** (`FileChannel.map`) | The OS page cache *is* your data structure; the mapping is a direct buffer view | The mapping is not part of the heap, so it survives independently; but access is a page-fault-bound path and the sibling's §10 shared-memory argument about NUMA placement applies unchanged |
| **The Foreign Function & Memory API** (JEP 454, final in JDK 22) | A supported API for allocating and addressing off-heap memory with explicit lifetimes and a defined deallocation point | The supported answer, and the one to prefer for new code. It also brings interop under the access-control regime JEP 472/498 introduced (§12) |
| **An off-heap library** (Chronicle, Agrona — §7.4) | A framework that owns the memory, the layout and the serialisation for you | A dependency with its own maintenance horizon; see §7.6 for the dated status of each |
| **A pauseless external store, consumed as a stream** | Move the state you cannot afford to have collected into a process whose lifetime model you control | A hop, a protocol and an operational surface |

The trade is always the same and worth stating once: **off-heap memory converts a latency risk you do not control into a lifetime risk you do.** That is usually a good trade for fixed-size structures known at start-up and usually a bad trade for per-message state that would have died young anyway.

**The locality cross-reference, in one paragraph.** Everything about *how the memory you use is laid out* — cache-line granularity, prefetching, NUMA placement, TLB and huge pages, structure-of-arrays versus array-of-structures — is the sibling's §3, and applies verbatim to Java data structures in the heap or off it. This guide adds only two JVM twists: boxed types silently add a pointer dereference to a layout you thought was flat, and an object header occupies space in front of every instance — a footprint cost that JEP 450 → JEP 519 → **JEP 534** (compact object headers by default in JDK 27) substantially reduces, so more of your working set fits in the cache lines the sibling's §3.2 is about.

---

## 6. The Memory Model and False Sharing

### 6.1 What the Java Memory Model guarantees, and where it is weaker than a reader assumes

The JMM is a specification of **what a thread is permitted to observe**, not of what the machine does. Its central promise is that correctly synchronised programs behave as if executed in some sequential order consistent with each thread's own program order — the "as-if-serial" property for data-race-free code. The corollary is the part that costs people production incidents:

- **Unsynchronised reads and writes have no timing guarantee.** A write by thread A may be observed by thread B immediately, later, or not at all until something forces the issue. There is no "it will be visible within a microsecond".
- **The compiler and the hardware may reorder independent memory operations.** The JIT reorders; the CPU reorders; the store buffer is per-core. Your source code's statement order is not a memory order.
- **Non-atomic reads and writes of wide values can tear.** A `long` or `double` field written without `volatile` is not guaranteed to be visible as a whole on all platforms.
- **A `final` field's visibility has a specific guarantee** tied to safe publication through the constructor, which is a *narrow* guarantee and not a general "initialised before it is read" promise.
- **Happens-before is the organising concept, and it is transitive** — but it must be *established*, by a monitor, a volatile access, an atomic operation, a thread start/join, or a supported library primitive. Two threads touching a field with no happens-before edge between them have a data race, and in the presence of a data race the model permits results that most engineers would call impossible.

The consequence for a latency-sensitive design is the same one the Rust companion draws about aliasing, from the opposite direction: **the sharing decision must be made explicit.** In Java you have the same four options — ownership split, single-threaded ownership, a synchronisation primitive, or a hand-built lock-free protocol — and the JMM makes the cost of the third and the risk of the fourth visible in the specification.

### 6.2 The visibility and ordering tools, with their costs

| Facility | Guarantee | Cost |
|---|---|---|
| **`volatile`** | A write is visible to a subsequent read; accesses are ordered with respect to each other (sequential consistency for that variable) | Prohibits caching the value in a register; a volatile write is a store that cannot be deferred, which on x86 typically means a store barrier. Cheap individually, expensive in a loop |
| **`synchronized`** | Mutual exclusion + a happens-before edge on entry/exit; the JVM can eliminate it for provably thread-local objects (lock elision) | The blocking cost the sibling's §5.1 owns. It also interacts with safepoints and thread state |
| **`java.util.concurrent.atomic`** | Per-operation atomicity with documented memory effects; the `get`/`set` forms are the "plain" accesses, the explicit `*Acquire`/`*Release` forms map to the weaker orderings | One instruction with lock semantics on x86 for `compareAndSet` loops; the *contention*, not the instruction, is the cost |
| **`VarHandle`** | The same access modes with a typed, checked interface, and several modes — plain, opaque, acquire/release, volatile, and the compare-and-exchange forms | The mode you choose *is* the cost; using the volatile mode where acquire/release would do is a self-inflicted barrier |
| **`LongAdder` / `LongAccumulator`** | Contention-spread counters: per-cell accumulation, summed on read | A read is not a single field load; a read is a reduce over cells. Correct for counters, wrong for state |
| **`StampedLock` / `ReentrantReadWriteLock`** | Optimistic or shared reads with explicit write exclusion | Better read scaling than `synchronized` under read-mostly access; a stamp you fail to validate is a silent correctness bug |
| **Immutable publication** | Safe through `final` fields and correct construction, and through a concurrent container's release semantics | Free at the read end; requires the object to genuinely be immutable, which is a design commitment rather than an annotation |

**The practitioner's rule:** choose the weakest ordering that the algorithm requires, then confirm with a profiler that the primitive is not appearing in the profile under load. A `volatile` in a spin loop, or a `compareAndSet` retried under contention, is a cache-line ping-pong exactly like the sibling's §5.2 lock-free hazards — the JMM merely tells you which of them are *legal*.

### 6.3 The atomics and `VarHandle`, as replacements for older idioms

The older JVM concurrency idioms — `synchronized` around a shared counter, a `Hashtable`, `Collections.synchronizedMap`, a `StringBuffer` — are correct and slow. The modern toolkit's shape:

- **`AtomicInteger`/`AtomicLong`/`AtomicReference`** for a single mutable value with atomic read-modify-write. These are the right tool for a state machine's transition, a sequence number, a one-slot publication.
- **`LongAdder`** where the value is a *statistic*. Latency instrumentation is the canonical case: a hundred threads incrementing one `AtomicLong` are a hundred threads fighting over one cache line; a `LongAdder` spreads the writes.
- **`VarHandle`** (and the `java.lang.invoke` family) where you want to *name* the access mode rather than inherit it: acquisition and release semantics where the algorithm permits them, `getAndAdd` without a full fence where it does not need one, and array-element access with the same modes.
- **`java.util.concurrent` containers** instead of wrapped ones: `ConcurrentHashMap` with its documented lock-free-read design, `ConcurrentLinkedQueue` and the `ArrayBlockingQueue`/`LinkedBlockingQueue` pair, `CopyOnWriteArrayList` for read-mostly configuration.

What the JVM added that matters most for latency, and is easy to overlook: **the containers implement their concurrency with the same cache-line awareness the sibling's §3.2 describes.** `ConcurrentHashMap`'s internal counter cells are marked `@jdk.internal.vm.annotation.Contended` in the JDK's own source — which is the natural bridge to the next subsection.

### 6.4 False sharing, and the annotation that addresses it

**The mechanism** is the sibling's §3.2, unchanged: two cores write two *different* variables that happen to occupy the same cache line, and the coherence protocol invalidates the line for each other, so each write costs a cache-line transfer instead of a store. The magnitude is the difference between a nanosecond and tens of nanoseconds per write, and it grows with core count.

**Why it is invisible in the source.** The layout of fields within an object, and the placement of objects relative to one another, is the runtime's decision. The sibling's advice — pad the structure so the hot fields do not share a line — requires a *packing control* that ordinary Java does not expose. Java's answer is an annotation, and the annotation is a different one from the one most guides name.

| Item | Verified value (JDK 27 source) |
|---|---|
| **The annotation** | `jdk.internal.vm.annotation.Contended` — an internal annotation, documented in-source as `@since 1.8`. The JDK uses it on its own classes |
| **`sun.misc.Contended`** | **Could not be located** anywhere in the OpenJDK tree at `jdk-11-ga`, `jdk-17-ga`, `jdk-21-ga`, `jdk-25-ga` or `jdk-27-ga`. Guides naming it should be treated with caution |
| **`-XX:ContendedPaddingWidth`** | `product(int, ..., 128)` with a documented range of 0–8192; described in-source as *"How many bytes to pad the fields/classes marked `@Contended` with"* |
| **`-XX:+EnableContended`** | `product(bool, ..., true)`; *"Enable `@Contended` annotation support"*. Default **on** |
| **`-XX:-RestrictContended`** | `product(bool, ..., true)`; *"Restrict `@Contended` to trusted classes"*. Because the default is **true**, application classes must be run with the flag *negated* — `-XX:-RestrictContended` — for `@Contended` to have any effect at all |

All three flags are ordinary product flags: **no `-XX:+UnlockExperimentalVMOptions` and no `-XX:+UnlockDiagnosticVMOptions` are required**, which is the opposite of what the older folklore about this feature says. The gating that matters is the last row: `-XX:-RestrictContended`. This is the guide's sixth anti-pattern in §15 — the annotation applied, the flag forgotten, the application unchanged, and a team convinced the fix worked because the code now looks different.

Two further honest notes. First, the annotation is an *internal* JDK API: depending on `jdk.internal.vm.annotation.Contended` from application code is reliance on an internal interface, with the usual compatibility exposure. The portable alternatives are to accept the false sharing, to restructure so the contended values live far apart in separate objects, or to use a design in which the contended counters are per-thread and are only *aggregated* occasionally — which removes the sharing instead of padding around it. Second, `-XX:ContendedPaddingWidth` with a default of 128 bytes is sized for a 64-byte cache line with room to spare; on hardware with a different line size or a different prefetcher, the default is an assumption rather than a measurement.

### 6.5 Detecting it

- **Contended-lock profiling.** async-profiler reports contended locks as one of its profiling modes (its own README lists *"Contended locks"* among what can be profiled). That catches monitor contention; it does not catch the case where two cores are merely writing different fields of the same line with no lock at all.
- **Allocation-layout awareness.** Two `int` counters in one object, two adjacent elements of an array of counters, or the first few fields of a per-thread state object are the usual offenders. When a profile shows a hot store with an unexplainably high cost that scales with thread count, look at what else occupies that line.
- **Symptom → cause.** The signature is a loop that does almost no work but shows high variance and high system-time under increased concurrency: exactly the same signature the sibling's §5.2 gives for a contended lock-free structure. Distinguish by checking whether the loop takes any lock at all.
- **The measurement, not the theory, decides.** Try the annotation with the gating flag, or restructure, and re-measure the same histogram. If the distribution does not move, the diagnosis was wrong, and you have gained that fact cheaply.

---

## 7. Concurrency and the Off-Heap Toolkit

This section is the *supply*: what is available on the JVM for the problems §5 and §6 create, what each is for, what it costs, and whether it is maintained. Maintenance status is part of the recommendation, and a dormant project is not recommended.

### 7.1 The reference concurrency toolkit

`java.util.concurrent` is the platform's library of documented, tested concurrency primitives, and for most latency-sensitive JVM work it is where the design should start — because its correctness argument is written down, and because a hand-written queue is a liability with no equivalent in a language whose standard library provides one.

The latency-relevant subset:

| Facility | What it answers | What it costs |
|---|---|---|
| **`ArrayBlockingQueue` / `LinkedBlockingQueue`** | Bounded producer-consumer handoff with blocking semantics and backpressure | Lock-based; fine for warm and cold paths, and for hot paths only when measured |
| **`ConcurrentLinkedQueue`** | A lock-free unbounded queue | Unbounded means it can grow without limit under backpressure — a latency problem deferred, not solved. Allocation per node |
| **`SynchronousQueue` / `TransferQueue`** | A rendezvous handoff with no queueing in the middle | The closest the library gets to a latency-shaped handoff; still general-purpose |
| **`ExecutorService` and friends** | A thread pool with a documented lifecycle and queueing policy | The *queue* is the latency risk: an unbounded work queue converts an overload into an ever-growing latency, and a bounded one with the right rejection policy converts it into a fast failure. Choose deliberately |
| **`CountDownLatch` / `Phaser` / `CyclicBarrier`** | Stage synchronisation | Cold-path tools; parking and unparking is not a hot-path mechanism. **`LongAdder` / `Atomic*` / `VarHandle`** are the §6.2 primitives, at the ordering you ask for |
| **`ConcurrentHashMap`**, `CopyOnWriteArrayList` | Concurrent containers with documented read semantics | `ConcurrentHashMap` reads are cheap and lock-free; iteration is weak-consistent, which is a semantics decision, not a latency one |

Two JVM-specific warnings that a native-stack engineer will not have internalised: an **unbounded** queue anywhere on a hot path is a latency incident waiting for a load spike, and a **common `ForkJoinPool`** quietly shared by unrelated libraries turns one component's parallelism into another component's tail.

### 7.2 The ring buffer and the reference implementation

The single-threaded-producer / single-consumer ring buffer — the sibling's §5.2 and §10.2 — is the canonical JVM low-latency handoff, and its most-cited implementation is **LMAX Disruptor**, whose design argument (pre-allocated entries, sequence barriers, mechanical sympathy with the cache hierarchy, no per-event allocation) is the reason it appears in almost every JVM low-latency architecture. It is also a worked example of why this guide dates every dependency.

| Project | Latest release (date) | Last repository push | Accurate status |
|---|---|---|---|
| **LMAX Disruptor** | 4.0.0 (2023-09-29) | 2025-04-02 | **Stable but low activity.** Not archived, not abandoned — its API is finished and its design is well understood. It is not an actively developed project, and this guide does not call it active |

That is the honest description, and it has a practical consequence worth stating plainly: a *finished* library can be a perfectly good dependency, but it is a **maintenance liability you are accepting deliberately**, not one you can assume someone else is managing. The relevant question is not "is it active?" but "who fixes it when the JDK changes underneath it, and have we established that it works on the JDK we are deploying?" — a question this document's own §12 (a platform that ships a feature release every six months) makes sharper than it is in a slower-moving ecosystem.

Written from scratch, the pattern is small enough to be worth owning for a critical path: two sequence counters, a pre-allocated array of plain objects or an off-heap region, and the §6.2 ordering primitives. Owning it means owning its cache-line padding, its publication discipline and its memory ordering — which is why the sibling's §5.2 discussion is the right prerequisite reading rather than a substitute.

### 7.3 JCTools

**JCTools** provides concurrent queue implementations that the JDK's general-purpose containers do not: bounded and unbounded, single- and multi-producer, single- and multi-consumer variants, with the *weakest* required ordering for each shape. Its value to this guide is exactly that mapping — the library's whole proposition is that the queue you actually need is not `ConcurrentLinkedQueue`, and that a queue specialised to your producer/consumer count is cheaper than a general one.

| Project | Latest release (date) | Last repository push | Status |
|---|---|---|---|
| **JCTools** | v4.0.7 (2026-08-18) | 2026-08-18 | Active |

### 7.4 Off-heap memory and structured data

| Option | What it gives you | What it costs |
|---|---|---|
| **Direct `ByteBuffer`** | Off-heap memory addressable from Java with a defined byte order and absolute/relative accessors | Manual lifetime; the buffer is released by the collector, not by you. Deallocation is therefore *scheduled by a heuristic*, which is an ironic property for a structure chosen to avoid collector influence |
| **`MappedByteBuffer`** (memory-mapped file) | The OS page cache as the data structure; the file is the durability boundary | Page faults on the read path; unmapping is not portable in the general case; the sibling's §10 NUMA-placement argument applies |
| **The Foreign Function & Memory API** (JEP 454, final in JDK 22) | Supported allocation, layout, addressing and explicit close; the modern answer, and the one to build new code on | Explicit lifetime discipline; the same JEP cluster that restricts `sun.misc.Unsafe` memory access (JEP 498, JDK 24) is steering the ecosystem here (§12) |
| **Chronicle Queue** (OpenHFT) | A persisted, memory-mapped, append-only queue with off-heap structured layout; the persisted-event-log pattern | A dependency with its own versioning and its own file format; its own performance claims should be re-measured |
| **Agrona** | The data-structure and buffer layer used by Aeron (below): off-heap buffers, ring buffers, atomics, and high-performance collection types | A library layer, not an architecture; its value is that it is the shared substrate of a well-maintained stack |

Dated status — the point of this table:

| Project | Latest release (date) | Last repository push | Status |
|---|---|---|---|
| **Chronicle Queue** (OpenHFT) | chronicle-queue-5.26.17 (2026-07-27) | 2026-09-20 | Active |
| **Agrona** | 2.6.1 (2026-09-17) | 2026-09-21 | Active |

### 7.5 Messaging and transport libraries

**Aeron** is the JVM ecosystem's mature answer to high-throughput, low-latency message transport: a reliable UDP-based messaging layer with an IPC path for same-host communication, designed around the same mechanical-sympathy principles as the ring buffer (pre-allocated buffers, flow control, no per-message allocation on the hot path, a shared-memory client library). It is the natural peer of the sibling's §7 transport material — but not its substitute: Aeron is a *transport library*, whereas the sibling's §7 is about the network stack, the NIC and the kernel; the two layers compose.

| Project | Latest release (date) | Last repository push | Status |
|---|---|---|---|
| **Aeron** | 1.53.2 (2026-09-18) | 2026-09-21 | Very active |
| **Agrona** (Aeron's data-structure layer) | 2.6.1 (2026-09-17) | 2026-09-21 | Active |

### 7.6 The maintenance ledger, and how to use it

| Project | Purpose in a latency-sensitive JVM system | Latest release (date) | Last push | Status |
|---|---|---|---|---|
| **Aeron** | Message transport, IPC and reliable UDP | 1.53.2 (2026-09-18) | 2026-09-21 | Very active |
| **Agrona** | Off-heap buffers, ring buffers, collections | 2.6.1 (2026-09-17) | 2026-09-21 | Active |
| **JCTools** | Specialised concurrent queues | v4.0.7 (2026-08-18) | 2026-08-18 | Active |
| **Chronicle Queue** | Persisted, memory-mapped off-heap log | chronicle-queue-5.26.17 (2026-07-27) | 2026-09-20 | Active |
| **LMAX Disruptor** | In-process ring buffer | 4.0.0 (2023-09-29) | 2025-04-02 | Stable but low activity — **not** active |

All rows were obtained from the projects' own GitHub API records and are dated 21 September 2026. Three rules follow for a bank's dependency governance, and they are the practical output of this section. **(1) Do not adopt a dormant project for a new latency-critical path** without an explicit decision to own it — the Disruptor row is the case study: excellent design, real production history, no active development. Adopting it is fine; assuming it is maintained is not. **(2) Date every dependency claim in the design document**, because a dependency status is a fact with a shelf life measured in months and "actively maintained" without a date is a claim about the past. **(3) Prefer the platform where the platform has an answer** — JEP 454's FFM API is final precisely to make off-heap access a supported part of the platform rather than a vendor's business, so ask whether the JDK's own API is sufficient before adding a third-party substrate, and use the library only where it demonstrably provides something the platform does not.

---

## 8. Transport, as It Differs in Java

**The network stack, kernel bypass, RDMA, TCP tuning, NIC features and their latency consequences are the sibling's §7, and this guide does not re-derive them.** Everything below is what *changes on the JVM* on top of that stack.

### 8.1 The four JVM-layer differences

| Difference | Mechanism | Consequence |
|---|---|---|
| **Threading model** | Java's default is thread-per-connection with blocking I/O, backed by OS threads; the JVM also offers non-blocking channels and, since JDK 21, virtual threads | The threading model determines how many threads contend for the collector's safepoints, how much per-connection state the heap holds, and how a slow dependency propagates |
| **Buffer management** | Sockets read into `ByteBuffer`s: heap-backed (a copy and an allocation per read) or direct (off-heap, but with the lifetime caveats of §7.4) | Buffer choice is a per-read allocation decision, which makes it an allocation-rate decision (§5) |
| **Collector interaction** | Every thread that must be brought to a safepoint is a thread the runtime must reach; every object the connection state holds is part of the live set | A thousand connections represented as a thousand heap objects and a thousand threads is a very different collector workload from a thousand connections in one event loop with off-heap state |
| **Instrumentation and copying** | The JVM's own framing (streams, buffers, codecs, logging) sits between the socket and your parser | Each layer is an invitation to copy and to allocate, and the copies are invisible in a stack trace that ends at `socketRead` |

### 8.2 Thread-per-connection: the costs that a native-stack engineer will underestimate

The sibling's §6.1 model — one thread per core, busy-polling, no blocking anywhere — has a JVM analogue, but the JVM's *defaults* are the opposite of it, and the cost of the default has three parts beyond the scheduling cost the sibling already covers:

- **Live-set cost.** Every connection's read buffer, write buffer, session state, decoder state and TLS context are heap objects. Ten thousand idle connections is ten thousand retained object graphs, all of which the collector must trace, and a substantial part of which is promoted to the old generation simply by surviving. That is a collection cost that scales with *concurrency*, not with throughput.
- **Safepoint cost.** Every thread — including every *idle* blocked thread — must reach a safepoint for the runtime to act. Idle threads parked in a blocking read are nominally cheap, but the safepoint machinery still has to account for them, and a thread that is inside a long-running JNI call or an uninterruptible transition can delay the safepoint for every other thread. This is the mechanism JEP 189's non-goal warns about (§4.1).
- **Context-switch cost under load.** A thousand platform threads on a machine with tens of cores is the sibling's §1.4 jitter list, now triggered by the JVM's own architecture rather than by the OS alone. The JVM also adds its own scheduling surface — the thread states, monitor states, and the safepoint protocol — on top of the OS's.

### 8.3 The non-blocking and asynchronous facilities

The JVM's own non-blocking substrate is the `java.nio` channel and selector layer: a single (or small number of) selector thread(s) driven by readiness events, with per-connection state held as a channel and a buffer rather than as a stack. This is the direct analogue of the sibling's §6 event loop, and the JVM-specific caveats are:

- **Readiness, not completion.** `java.nio` selectors are readiness-based, so the loop wakes, reads what is available, and must handle partial reads and partial writes explicitly. The sibling's §6.3 discussion of `io_uring`'s *completion* model is the relevant contrast, and the JVM's answer to that model lives in the libraries above the platform (Aeron's own I/O layer, §7.5) rather than in `java.nio` itself.
- **The selector's wake-up is a syscall boundary.** The loop's latency is bounded below by the poll syscall and by how promptly the kernel reports readiness; the sibling's §6.2 comparison of `epoll` against polling applies, including the case where busy-polling in user space is the honest answer for the very tightest budgets — and is achievable in Java by polling a channel in a dedicated pinned thread.
- **The async API layer is not the latency layer.** `CompletableFuture`, executor-backed async APIs and reactive schedulers (the RxJava guide's subject) are *composition* tools: they decide which thread runs a continuation, not how fast the work runs. A reactive pipeline with a scheduler hop per stage adds two thread handoffs to a path that was one call, and the handoffs are the latency. This guide's position is short: choose the threading model for the workload, and treat the async framework as a way to express that model, never as a source of speed.

### 8.4 Buffer management, and why direct buffers matter

| Buffer | Read path | Latency consequences |
|---|---|---|
| **Heap `ByteBuffer`** | The socket read copies into a byte array owned by the JVM heap | The copy costs bandwidth; the array is a heap object, so the read path allocates or retains per-connection buffers; and the copy means the data you parse is not the data the kernel wrote, so the parser's cache locality is over a *different* region |
| **Direct `ByteBuffer`** | The read fills an off-heap region directly | No copy, and the buffer is not part of the collector's workload — which is the real argument for direct buffers in a latency context, more than the copy alone. The cost is the §7.4 lifetime caveat: direct buffers are released by the collector's cleaning machinery, so their *deallocation* is heuristic even though their *use* is not |
| **A pooled set of direct buffers, assigned to a connection or a stage** | One allocation at start-up per slot, reused forever | The shape this guide recommends where a connection-oriented design is unavoidable: it removes the per-read allocation, keeps the buffers off the collector's books, and makes the memory footprint a function of the pool size rather than of the load |

The direct-buffer argument is worth stating in the discipline's own terms: **a direct buffer is an allocation-discipline decision (§5) before it is a zero-copy decision.** It reduces the collector's live set and eliminates per-read allocation. Its copy-avoidance is a bonus.

### 8.5 The threading model and the collector, considered together

This interaction is the one that a JVM architect must reason about explicitly and that no amount of network tuning addresses:

- **Thread count sets the safepoint population.** Stop-the-world phases must reach every thread. A design with thousands of platform threads is a design with a larger safepoint problem than the same workload on a handful of event-loop threads, and the difference shows up as a wider distribution even when the collector's own pauses are unchanged.
- **Allocation rate sets the collection frequency** (§5.1), and the threading model largely sets the allocation rate: one buffer and one decoder per connection allocate per *connection-event*; an event loop with a per-stage reusable buffer allocates per *stage invocation* and only when it must grow.
- **The pause interacts with the model asymmetrically.** A stop-the-world phase during a thread-per-connection workload harms the requests in flight and the ones queueing behind them. The same phase in an event-loop workload stops the loop, which stops *everything* — so the loop model concentrates the risk and the thread-per-connection model distributes it. Neither is universally right; the choice should follow from what the SLO is a percentile *of*.
- **The honest design goal** is the sibling's: one thread per core, no blocking, no per-message allocation, and the smallest possible set of heap-resident live objects — with the JVM's own contribution being that this goal also minimises the collector's and the safepoint machinery's opportunities to interfere.

### 8.6 Virtual threads: what they fix, and what they explicitly do not

Virtual threads were delivered final in **JDK 21 by JEP 444** (previews: JEP 425 in JDK 19, JEP 436 in JDK 20). The follow-up that matters most for mixed Java code is **JEP 491, "Synchronize Virtual Threads without Pinning" — Closed/Delivered, Release 24** — which arranges that virtual threads blocking inside `synchronized` methods and statements release their underlying platform thread, eliminating nearly all cases of pinning, and also improves the diagnostics for the remaining ones.

| Statement | Correct? |
|---|---|
| Virtual threads let a JVM service hold far more concurrent blocking operations than platform threads allow | **Yes** — that is the feature |
| Virtual threads make an individual request's latency lower | **No.** JEP 444's own goals are framed around *high-throughput* concurrent applications and using huge numbers of threads. Virtual threads are a **throughput** feature |
| Virtual threads remove the need to think about the collector | **No**, and the opposite direction is the risk: they make it *easier* to hold far more live state and far more in-flight work, which increases the live set and the allocation rate. A virtual-thread design without an allocation discipline is a bigger collector workload reached more quickly |
| Virtual threads remove the pinning penalty on `synchronized` | **Yes, since JDK 24** (JEP 491). The mechanism's own "Risks and Assumptions" section is honest that exiting a monitor may now have to queue a virtual thread to the scheduler, which "is currently not as efficient" as unparking a platform thread — a small cost paid for the scalability |
| Virtual threads make blocking I/O free | **No.** The blocking I/O still happens; what changes is which thread is idle while it happens. If the latency budget does not tolerate a blocking call, virtual threads do not make it tolerable |

The operational summary for a latency-sensitive service: adopt virtual threads to remove the *thread-count* constraint on a design that is otherwise correct — that is, one that has already controlled allocation and live set. Do not adopt them as a latency optimisation, and do not let them become a reason to skip §5.

---

## 9. Measurement: The Measured Zero

The repository's discipline is "the measured zero" — the principle that a claim about latency is worth exactly what its measurement is worth. Java makes this principle *harder to satisfy* than the sibling's stack does, because the runtime is a moving target. It also makes it easier, because the platform ships the instruments.

### 9.1 Why this tooling exists

The tooling below exists because of §3.4. Each entry answers three questions: *what question does it answer, what can it not answer, and is it maintained?*

### 9.2 JMH — the benchmark harness

- **What it is:** the OpenJDK project's Java Microbenchmark Harness, whose reason for existing is the whole of §3.4 — it runs the benchmark in a dedicated JVM, performs warmup iterations before measured iterations, consumes results so they cannot be optimised away, forks to control profile contamination, and reports a distribution.
- **Answers:** "what does this *method or small unit* cost, in steady state, in this JDK build, on this machine?" **Cannot answer:** end-to-end request latency under real load, collector behaviour under a real allocation mix, or anything about production traffic — a JMH number is a controlled comparison, not a prediction of a service SLO. **Dated status** (project's own repository API, 2026-09-21): tag **1.37**, last push **2026-07-14**, active; no GitHub "releases" objects are published for that tag.

### 9.3 JITWatch — seeing what the compiler actually did

- **What it is:** a log analyser and visualiser for the HotSpot JIT compiler. It reads the JVM's compilation log and shows which methods were compiled, which inlined into which, which were too large to inline, which were deoptimised, and why a call site was not optimised.
- **Answers:** "why is my hot path not optimised the way I assumed?" — the questions §3.1–§3.3 raise. It is the JVM's nearest equivalent to reading the generated assembly, and what makes profile pollution demonstrable rather than speculative. **Cannot answer:** anything about time. It shows code shape, not cost; a method that was inlined is not thereby fast, and one that was not is not thereby the bottleneck.
- **Dated status:** release **1.5.0 (2025-11-11)**; prior **1.4.9 (2024-01-06)**, **1.4.7 (2022-01-23)**; last push **2026-09-18**, active. **The repository has moved**: `AdoptOpenJDK/jitwatch` returns a 301 redirect to **`chriswhocodes/jitwatch`** — bookmark the new location. **Workflow, per the project's own README:** a HotSpot log analyser built on JavaFX (a Maven dependency for JDK 11+; pre-JDK 11 needs a runtime including JavaFX), so enable the JVM's compilation logging, run the workload, open the log.

### 9.4 async-profiler — the sampling profiler that survives safepoints

- **What it is:** a low-overhead sampling profiler for HotSpot-based JVMs which — in the project's own words — "does not suffer from the Safepoint bias problem", because it collects stack traces through HotSpot-specific mechanisms rather than by asking the JVM at safepoints. Its README lists what can be profiled: CPU time, allocations in the Java heap, native memory allocations and leaks, contended locks, and hardware and software counters such as cache misses, page faults and context switches — and it monitors non-Java threads (the GC and JIT compiler threads) and shows native and kernel frames in stack traces.
- **Answers:** where the CPU cycles and the allocated bytes are actually spent, including in the runtime's own threads that ordinary Java profilers cannot see; the allocation view is the single most useful measurement in this guide (§5.2). **Cannot answer:** *when*. A profile is an aggregate share of samples, not a latency histogram — it tells you where the time goes, not the p99.9, and it attributes a non-CPU stall only in the specific modes that look for one.
- **Dated status:** stable release **v4.5 (2026-07-20)**, last push **2026-09-21**, very active; Linux x64/arm64 and macOS x64/arm64 as officially maintained builds; requires **JDK 11+** and ships a converter that writes JFR, composing with the route below. **The safepoint-bias point, stated precisely:** a traditional profiler injects a sample by asking the JVM to stop, so it can only sample *at* safepoints — biasing the profile in exactly the direction that hides the runtime's own interference. A profiler that samples asynchronously is not a nicety here; it is the difference between measuring your code and measuring the sampling mechanism.

### 9.5 Collector-log analysis

- **What it is:** the JVM's own GC logging, the `-Xlog:gc` family in the modern unified logging framework. It appears twenty-two times in the JDK 27 launcher's own man page, which is the honest evidence that it is the supported route (§10).
- **Answers:** is the collector acting during my incident, and why — pause durations, cycle frequency, whether pressure is young-generation, old-generation or allocation-stall, whether the heap is being resized or uncommitted. **Cannot answer:** the application's latency. The most common error this guide sees is a team reading GC logs, finding nothing alarming, and concluding the problem is not the JVM's — when the correct conclusion is only that it is not the *collector's* (§4.1's fourth row, §3.2). **How to use it:** enable it in production at a level whose overhead you have measured, and keep it on; a collector log that can only be enabled during an incident is not there during the incident.

### 9.6 Flight Recorder and Mission Control

- **What they are:** the JVM's built-in event-recording and analysis facility. The Flight Recorder is a low-overhead recording facility enabled at the launcher with the `-XX:StartFlightRecording` family, which is present in the JDK 27 launcher's man page and documented there — a supported, product-level route rather than an experimental one. **JDK Mission Control** is the graphical analyser for the recordings.
- **Answers:** what happened, in order, across the whole runtime — allocation samples, GC events, JIT events, safepoint events, thread activity, file and socket activity — with a timeline. This is uniquely valuable for §3.2's unexplained tail spike: a recording spanning the spike can show a deoptimisation, a GC cycle and a safepoint landing in the same millisecond window, turning an attribution argument into an observation. **Cannot answer:** continuous production latency distributions, or the cause of a hot method (JITWatch's and async-profiler's job). Recording has a cost, so the technique is a low-overhead continuous recording with a rolling window, dumped when something goes wrong. **Dated status:** JDK Mission Control tag **9.1.2-ga**, last push **2026-09-13**, active; the Recorder itself is under active change — JEP 509 (Experimental), JEP 518 and JEP 520 in JDK 25, JEP 536 in JDK 27 — and in-process data redaction matters to a bank because it is a governance feature, not a convenience.

### 9.7 The honest point, and a minimum discipline

They compose rather than compete, and the split is clean: JMH measures cost, JITWatch shows code shape, async-profiler shows where cycles and bytes go, GC logs say whether the collector acted, and JFR with Mission Control gives the ordered timeline around an incident. None of them tells you *when* a latency event hit a request; the sibling's §13 owns that measurement, and the JVM changes its cost, not its necessity.

**Which tool you choose matters less than whether your team measures at all.** The failure mode this guide observes most often is not a team using the wrong profiler; it is a team with no histogram, tuning a collector from a blog post, and discovering the truth in a post-incident review. A correct tool used sporadically is worth less than a crude histogram that runs every day.

The minimum a team can adopt, and each item is deliberately cheap:

1. **One end-to-end latency histogram in production**, at percentiles, per deploy and per instance — the sibling's §13 discipline, not negotiable, and without it everything else is speculation.
2. **GC logging on, always**, at a level whose overhead has been measured once and recorded.
3. **A JMH benchmark for any component with a latency claim in its design document**, run in CI so that a regression is a failed build rather than a production surprise.
4. **A continuous low-overhead JFR recording with a rolling window**, dumped on incident, so that "what was the runtime doing?" has an answer that is not a guess.
5. **An allocation profile of the hot path, at least once per release** — §5.1 says that is the number governing the pause, and it changes silently.
6. **A warmup-time metric**, because §3.5 says your deploy is a latency event.

Six items, no exotic tooling, and a team that holds to them will out-perform a team with better tools and no cadence.

---

## 10. The Runtime Configuration

### 10.1 The principle

A JVM flag is a **hypothesis about your workload**, expressed in a syntax the runtime will accept whether or not the hypothesis is true. The JVM will not tell you that `-XX:MaxGCPauseMillis=5` is unachievable for your allocation rate; it will accept the flag and miss the target. The JVM will not tell you that `-XX:+UseG1GC` was already the default; it will accept a redundant flag. So the discipline is: **every flag in a production command line should be traceable to a measurement, and any flag that cannot be traced should be deleted.**

The tables below list only flags whose existence, exact syntax and gating were verified by reading the JDK 27 source tree or the JDK 27 launcher's own man page. Where a flag is not documented in the man page, that is noted, because the man page is what an operations team will consult.

### 10.2 Heap, memory and page behaviour

| Flag | Verified declaration and default | Gating | What a latency practitioner wants it for |
|---|---|---|---|
| `-XX:+AlwaysPreTouch` | `product(bool, ..., false)` | None — ordinary product flag | Pre-faults the whole heap at start-up so that ordinary allocation does not pay for page faults later. It converts a start-up cost into a steady-state saving, which is exactly the right trade for a service that must not have a long tail (§3.5) |
| `-XX:+AlwaysPreTouchStacks` | `product(bool, ..., false, DIAGNOSTIC)` | **Requires `-XX:+UnlockDiagnosticVMOptions`** | The stack analogue. Diagnostic gating means it is not intended as a routine production flag |
| `-XX:+UseTransparentHugePages` | `product(bool, ..., false)` declared in the **Linux-only** globals | None — ordinary product flag, Linux only | Asks the OS for huge pages, reducing TLB pressure on large working sets. The sibling's §3.4 owns the mechanism; the note here is that the JVM's flag only *expresses the request* — the host's own THP configuration and the `madvise`/`always` policy decide what happens |
| `-XX:LargePageSizeInBytes` / `-XX:+UseLargePages` | the former is in the JDK 27 launcher man page; the latter is declared `product_pd` (platform dependent, so the default differs by platform and build) | None | Explicit page sizing for hosts where THP is not the right mechanism. Not a portable prescription: verify on your platform and JDK build before relying on either |
| `-XX:+UseNUMA` / `-XX:+UseNUMAInterleaving` | both `product(bool, ..., false)` | None | The two NUMA placement policies — a *choice*, not an addition. The sibling's §3.3 owns the mechanism; the JVM's version is worth measuring rather than assuming, since its benefit depends on the host's topology and on whether the heap is the locality problem |
| `-XX:+UseCompactObjectHeaders` | `product(bool, ..., true)` — **on by default in JDK 27** | None | Smaller object headers, more objects per cache line, less heap pressure. Consistent with JEP 534 (§12.2). It is listed here as a flag you may now *not* need to set, and as a change that may explain a footprint or latency difference between JDK 24 and JDK 27 |
| `-XX:+UseCondCardMark` / `-XX:InitialCodeCacheSize` | the first `product(bool, ..., false)` and present in the man page; the second `product_pd, size_t`, platform-dependent | None | Cond-card marking is a write-barrier detail on a path that shows up in collector profiles — a documented knob, not a first-line tuning flag. The code-cache size matters more: a cache that fills stops compilation, and the JVM may then discard compiled methods — a latency cliff, not a slope (§3.5) |

### 10.3 Collector selection and its options

**Selection flags.** All five of the following are `product(bool, ..., false)` in the JDK 27 source, so none needs an unlock:

| Flag | Collector selected |
|---|---|
| `-XX:+UseSerialGC` | Serial |
| `-XX:+UseParallelGC` | Parallel |
| `-XX:+UseG1GC` | G1 (already the default since JDK 9, and the default in all environments since JEP 523 in JDK 27) |
| `-XX:+UseZGC` | ZGC, generational (the only mode since JEP 490) |
| `-XX:+UseShenandoahGC` | Shenandoah. In JDK 27 this is satb mode; generational mode is selected separately, and generational becomes the default only with JEP 535 in JDK 28 |

**The one selection flag that does need an unlock:**

| Flag | Verified declaration | Gating |
|---|---|---|
| `-XX:+UseEpsilonGC` | `product(bool, ..., false, EXPERIMENTAL)` | **Requires `-XX:+UnlockExperimentalVMOptions`** |

Use Epsilon only as JEP 318 proposes it — to establish the no-GC latency baseline for a workload, so that you can separate GC-induced outliers from the non-GC ones (§4.1's fourth row). It is not a production collector and it does not reclaim.

**General collector options (product flags):**

| Flag | Verified declaration and default | Note |
|---|---|---|
| `-XX:MaxGCPauseMillis` / `-XX:GCTimeRatio` | `product(uintx, ..., max_uintx - 1)` (effectively unset) and `product(uint, ..., 99)` | A *goal*, not a guarantee: naming a pause target does not make it achievable, and §4.7's argument applies in full. Setting both a pause goal and a throughput goal is a negotiation with the collector's heuristics, not two independent dials |

**ZGC options.** These are **not documented in the JDK 27 launcher man page**; the values below come from the JDK 27 source file that defines them, which is the appropriate reference when the man page is silent.

| Flag | Verified type and default |
|---|---|
| `-XX:ZAllocationSpikeTolerance` | `double`, default `2.0` — how much of an allocation-rate spike the collector should tolerate before reacting |
| `-XX:ZCollectionInterval` | Documented in-source as *"Backwards compatible alias for `ZCollectionIntervalMajor`"* — so the modern spelling is the one to use |
| `-XX:ZCollectionIntervalMajor` / `-XX:ZCollectionIntervalMinor` | `double`, default `-1` (disabled) — a forced collection interval rather than a reactive one |
| `-XX:ZCollectionIntervalOnly` | `bool`, default `false` |
| `-XX:ZUncommit` | `bool`, default `true` — returns unused memory to the OS |
| `-XX:ZUncommitDelay` | `uintx`, default `5*60` — the delay before uncommitting |
| `-XX:ZStatisticsInterval` | `uint`, default `10`, marked **DIAGNOSTIC** → requires `-XX:+UnlockDiagnosticVMOptions` |

**Shenandoah options:**

| Flag | Verified declaration and default | Gating |
|---|---|---|
| `-XX:ShenandoahGCMode` | `ccstr`, default the literal string `"satb"` in JDK 27 source | None for the flag; **generational mode is a product feature since JEP 521 (JDK 25) but is not the default until JEP 535 (JDK 28)** |
| `-XX:ShenandoahGCHeuristics` | `ccstr`, default `"adaptive"` | None |
| `-XX:ShenandoahRegionSize` | `size_t`, default `0`, marked **EXPERIMENTAL** | **Requires `-XX:+UnlockExperimentalVMOptions`** |

### 10.4 Diagnostics and logging that must be enabled to see anything

| Flag family | Verified status | Why it belongs on by default in a latency-sensitive deployment |
|---|---|---|
| `-Xlog:gc` and the wider unified-logging family, plus `-XX:StartFlightRecording` | `-Xlog:gc` appears **22 times** in the JDK 27 launcher man page; the recording option is present and documented there too — both ordinary, documented options needing no unlock | The §9.5 and §9.6 routes: without the log, a collector question can only be answered during an incident; without the recording, nothing tells you what the runtime was doing at the moment of the spike |
| `-XX:+UnlockDiagnosticVMOptions` | Appears **25 times** in the JDK 27 launcher man page | The gate in front of every DIAGNOSTIC flag, including `-XX:+DebugNonSafepoints` (§10.5). Many flags *listed* under it are not themselves diagnostic — the man page's repetition is an artefact of how options are grouped, not a per-flag statement |
| `-XX:+EnableContended`, `-XX:-RestrictContended`, `-XX:ContendedPaddingWidth` | all three `product` flags, none needing an unlock | The §6.4 gating, repeated here because these flags are **not** in the man page and are therefore exactly the kind a team forgets |

### 10.5 The two flags that are easy to get wrong

- **`-XX:+DebugNonSafepoints`** is declared `product(bool, ..., trueInDebug, DIAGNOSTIC)`. Two consequences: it is **off in a product build and on in a debug build**, and being DIAGNOSTIC means a product build must also pass `-XX:+UnlockDiagnosticVMOptions` before it can be set. It is the flag sampling profilers want, and the reason a profile taken on a debug or test build can look better than the same profile on production.
- **`-XX:+ZGenerational`** is **not a JDK 27 flag at all** (§4.4). It is obsolete, it prints a warning, and passing it changes nothing.

### 10.6 The honest warning

**A flag copied from a tuning guide, without understanding the workload, is worse than the default.**

Three mechanisms make that literally true rather than merely rude:

**The defaults are the product of measurement over a very broad population, and they are adaptive** — a pause goal, a time ratio, a heap that resizes. Overriding one with a hard value removes the adaptivity: `-XX:MaxGCPauseMillis` set optimistically does not produce short pauses, it makes the heuristics respond to an impossible goal and work harder. **A flag from a blog post is a flag from someone else's workload** — heap size, allocation rate, live-set shape, core count and JDK version all differ, and each changes the flag's effect. And **a copied flag is often a stale flag**: §4.4 is four examples in one guide's space, each correct at some point and each now wrong. A production command line assembled from search results accumulates these residuals, and nothing removes them.

The positive discipline, which is cheap: keep the production command line in version control with a comment on every line naming the measurement that justified it, re-derive the flags when the JDK major version changes, and delete any flag whose justification has expired.

### 10.7 The OS-level jitter the JVM cannot fix

The JVM is a process. It cannot prevent the OS from preempting it, cannot prevent an interrupt from landing on the core running the hot thread, cannot prevent the frequency governor from down-clocking, and cannot prevent a hypervisor from descheduling it. These are the sibling's §1.4 jitter sources, and the sibling's §3 hardware material is the place to solve them: CPU pinning and core isolation, NUMA-aware placement, interrupt affinity, and power-management settings.

Two JVM-specific observations are worth adding:

- **The JVM does not tell you when the OS is the cause.** Nothing in the collector log or a JFR recording says "a device interrupt delayed this thread". A JVM team diagnosing a tail must reach for the OS-level tooling the sibling describes, and the most useful reflex is to ask whether the tail spike correlates with the collector's log at all.
- **Two JVM-level settings interact with the OS layer even though they are not OS settings.** Thread count (each thread is a scheduling entity and a safepoint participant, §8.5) and heap pre-touching (`-XX:+AlwaysPreTouch`, which converts page faults into start-up work) are decisions that change how much the *OS* has to do to your process. They are the JVM's two contributions to the determinism agenda, and both are more effective than any collector tuning for the same purpose.

---

## 11. AOT, Native Images and the Alternative Runtimes

### 11.1 What ahead-of-time compilation of Java changes

Java's conventional execution model defers compilation to run time, which buys profile-guided optimisation for free and charges warmup for it. Every AOT approach to Java is an attempt to move part of that cost to build time, and every one of them pays for it in one of three currencies.

| What AOT can move earlier | What it gives up | The currency |
|---|---|---|
| Class loading, linking and verification | Nothing much — this is the least controversial win | Build time, and cache/cache-file management |
| Profiles and pre-compiled method code | The *accuracy* of a profile taken from a build-time run rather than from production traffic | Compile correctness under profile drift: a profile from a representative run is not a profile from your run |
| A fully closed-world native image | Reflection, dynamic class loading, JNI without configuration, and the JIT's ability to re-optimise based on live behaviour | Configuration complexity and a build pipeline that must be maintained per deployment target |

The trade in one sentence, and it is the fairest one available because it is the project's own: **GraalVM Native Image's documentation states that a native image "Starts in milliseconds" and "Delivers peak performance immediately, with no warmup"** — label this as the project's own claim, not as a measurement of your workload. What it does not say in those words is what §3 says at length: peak performance immediately means *the* peak the static compilation decided on, with no profile to improve it and no re-optimisation when the workload shifts.

### 11.2 The native-image toolchain, and its relationship to the JDK

| Fact | Verified value (21 September 2026) |
|---|---|
| Newest GraalVM CE release notes | **25.3.4.1**, then 25.2.4, 25.1.3, 25.0 |
| What that version line means | The versioning scheme is **"GraalVM 25.x for JDK 25"**, so the newest documented line targets **JDK 25 (LTS)**, not the current JDK 27 |
| The release-notes index | **Lists no JDK 26 or JDK 27 line** |
| Repository activity | Active (last repository push 2026-09-21) |

The honest reading of that table is the one a risk committee needs: **the native-image toolchain is not tracking the JDK feature-release clock.** Adopting GraalVM Native Image for a production component today means pinning to a JDK 25 line, which is defensible — JDK 25 is an LTS with years of support — but it means the AOT path and the JIT path are on different upgrade cadences, and a team running both has two platform lifecycles to manage.

### 11.3 The JDK's own AOT line: Project Leyden

The JDK does not leave AOT to a downstream project. The Leyden line, each step verified at its own JEP record:

| JEP | Release | Status | What it does |
|---|---|---|---|
| **JEP 483** Ahead-of-Time Class Loading & Linking | 24 | Delivered | Moves class loading and linking to build time by recording what the application loaded during a training run |
| **JEP 514** Ahead-of-Time Command-Line Ergonomics | 25 | Delivered | Makes the AOT workflow a single command rather than a bespoke pipeline |
| **JEP 515** Ahead-of-Time Method Profiling | 25 | Delivered | Records method profiles at build time so that the JIT starts from a profile instead of from nothing |
| **JEP 516** Ahead-of-Time Object Caching with Any GC | 26 | Delivered | Allows the cached objects to be used with a *low-latency collector*. Its own motivation section is an argument about tail latency — which is exactly the intersection this guide cares about |
| **JEP 544** Ahead-of-Time Code Compilation | 28 | **Proposed to Target** — not delivered | Would take the next step and pre-compile code, not just pre-load it |

The line to draw for a practitioner: **JEP 516 is the one that makes AOT compatible with the latency collector.** Before it, the AOT cache's object-caching behaviour was tied to collectors whose pause profile a latency-sensitive service had already rejected — which is precisely how a "nice optimisation" ends up unusable by the application that needs it most. If you are evaluating AOT for a latency-sensitive JVM service, JEP 516 (JDK 26 and later) is the minimum bar.

### 11.4 The alternative JVMs

| Runtime | What it is | Verified status (21 September 2026) |
|---|---|---|
| **Eclipse OpenJ9** | An alternative JVM implementation, contributed to Eclipse by IBM, and the VM under IBM's Java offering. Its distinguishing latency-relevant property is a different approach to memory management and a different JIT implementation from HotSpot's | **Eclipse OpenJ9 v0.62.0, released September 2026, supporting OpenJDK 27.** Verified from the project's own site |
| **IBM Semeru Runtimes** | The commercial-build family: described by IBM as free, enterprise-ready OpenJDK-based binaries that drop-in replace another runtime, with enterprise support available | Project page verified 2026-09-21. **A dated Semeru release number for a specific JDK was not established in this run** — see §16 |
| **Azul Platform Prime** | A commercial product whose central component is **Azul Zing Builds of OpenJDK**: an enhanced OpenJDK build offering, per Azul's own documentation, "low, consistent response latency" through the **C4 pauseless garbage collector**, higher throughput through the **Falcon JIT compiler**, and "faster warm-up … and less deviations from steady-state performance" through the **ReadyNow warmup optimizer** and Cloud Native Compiler | Azul's own documentation site (verified 2026-09-21). **These are the vendor's claims about its own product**, and the specific latency figures a procurement evaluation needs are not in that material |

Three rules for evaluating any of them, and they follow directly from this document's method:

**The vendor's claim is a hypothesis about your workload, like a flag (§10.1)** — "pauseless" and "faster warm-up" are properties of a design under conditions the vendor chose, and the only admissible evidence is your histogram, over your restart pattern. **A different VM is a different platform, not a faster HotSpot** — the alternative JVMs change the compiler and the collector, the two adversaries of §2, which means the flags, the diagnostics, the profilers and the collector logs all differ, and your team's HotSpot-shaped operational knowledge does not transfer. And **a commercial runtime adds a vendor relationship to the critical path**: support terms, release cadence, patch policy and a licence review belong in the same decision as the latency claim.

### 11.5 Which workloads suit which approach

| Workload shape | Reasonable choice | Reasoning |
|---|---|---|
| Long-running service, steady traffic, seconds-to-minutes of warmup tolerance | **HotSpot with a latency-appropriate collector**; add the Leyden AOT cache to shorten the warmup window | The JIT's profile-guided optimisation is the whole advantage of Java; a long-lived process gets it for free |
| Many short-lived processes, or serverless/scale-to-zero | **AOT/native image**, accepting the profile-guided-optimisation loss | The JIT never gets to use its profile, so the warmup is pure cost. This is AOT's real use case, and note that it is a *deployment-shape* argument, not a latency argument |
| Latency-critical path inside a long-running service | **HotSpot, warmed deliberately before traffic** (§3.6), with an AOT cache to shorten the warmup | You want the aggressive tier *and* you want to reach it sooner |
| A path where the tail is a legal or contractual exposure | **HotSpot with a concurrent collector, plus the measurement discipline of §9**; consider a commercial runtime only if your own histograms show the platform default is the constraint | The evidence requirement before changing platform implementations should be higher than the evidence requirement before changing a flag |

---

## 12. The Platform Changes in Flight and Recently Delivered

### 12.1 The release clock

| Fact | Verified value |
|---|---|
| Cadence | A feature release every **6 months** |
| LTS | Every two years. **JDK 21** (GA 19 September 2023) and **JDK 25** (GA 16 September 2025) are LTS; the next in the line is **JDK 29** |
| Current release | **JDK 27, GA 15 September 2026** — six days before this document's verification date |
| JDK 26 | GA 17 March 2026 |
| In development | **JDK 28**, with JEP 401 Value Objects at Status **Integrated**, JEP 535 Shenandoah generational-by-default **Targeted**, and JEP 544 AOT code compilation **Proposed to Target** |

Two consequences that a bank must budget for rather than discover: **a JVM platform is a living dependency with a six-month heartbeat**, and **the LTS line is where production belongs.** Between an LTS and the next, the feature releases are the place to evaluate, not to deploy — and a team that deploys feature releases must accept re-verifying its flags, its collector choice and its dependencies every six months.

### 12.2 The changes, with the latency verdict attached

Every row below carries a verified JEP number and status, or is explicitly flagged. The final column is this guide's judgement, and it is one sentence by construction.

| Change | Verified status | Does it change anything for a latency-sensitive workload? |
|---|---|---|
| **Virtual threads** | JEP 444, Closed/Delivered, Release 21. Previews: JEP 425 (JDK 19), JEP 436 (JDK 20). **JEP 491** "Synchronize Virtual Threads without Pinning", Closed/Delivered, Release 24, removes almost all pinning in `synchronized` | **Not as a latency feature**: JEP 444's own goals are about *high throughput*. JEP 491 removes a scalability penalty that also happened to be a latency cliff for mixed code. Read §8.6 and do not buy them for latency |
| **Structured concurrency** | **Still a preview at JDK 27**: JEP 453 (JDK 21, preview), JEP 499 (JDK 24, fourth), JEP 505 (JDK 25, fifth), JEP 525 (JDK 26, sixth), **JEP 533 (JDK 27, seventh preview)** | **Not yet.** A preview API is not something a latency-critical production path should depend on; its value is correctness of fan-out, not speed |
| **Scoped values** | **A different status from structured concurrency**: JEP 446 previewed in JDK 21, **JEP 506 is final in JDK 25** (listed without a preview suffix) | **Indirectly yes.** A final, low-overhead alternative to thread-local storage removes a map lookup and a lifetime coupling — a small but real win on any hot path that currently reads thread-locals |
| **Foreign Function & Memory API** | **JEP 454, Closed/Delivered, Release 22** (previews: JEP 442 JDK 21, JEP 434 JDK 20, JEP 424 JDK 19) | **Yes, positively.** A supported, specifiable off-heap API with explicit lifetimes is the route §7.4 recommends; it is what makes the off-heap discipline a platform decision rather than a vendor library's |
| **Restricting JNI and `sun.misc.Unsafe` memory access** | **JEP 472** "Prepare to Restrict the Use of JNI" (JDK 24) and **JEP 498** "Warn upon Use of Memory-Access Methods in `sun.misc.Unsafe`" (JDK 24) | **Yes, as a project cost.** Native interop is being brought under an explicit access regime, and unsafe memory access now warns. Any latency component that reaches into native code or `Unsafe` has a migration on its roadmap whether or not it has scheduled one |
| **Value objects** | **JEP 401, "Value Objects (Preview)", Status *Integrated*, Release 28 — NOT delivered.** Its dependency, JEP 539 "Strict Field Initialization in the JVM (Preview)", is likewise not delivered | **Not yet, and the date matters.** Value objects would remove the boxing and header overhead §5.3 and §5.6 complain about — a genuine latency-relevant win — but at Status *Integrated* with a Release of 28 it is a future JDK, not a design input today. An architecture that assumes value objects is an architecture sized for a platform that does not exist |
| **Compact object headers** | The chain: **JEP 450** experimental (JDK 24) → **JEP 519** product (JDK 25) → **JEP 534** "Compact Object Headers by Default" (JDK 27), with the source default verified `true` | **Yes, modestly and now.** JEP 519's own attributable evidence: SPECjbb2015 using **22% less heap space and 8% less CPU time**, with the *"number of garbage collections done by SPECjbb2015 … reduced by 15%"*. It also records that Amazon tested it on "hundreds of services in production, most of them using backports of the feature to JDK 21 and JDK 17". A smaller header means a smaller live set for the same logical state, which is §5.1's lever moved favourably by the platform rather than by your code. JEP 519 also notes that four object-header bits were "already reserved for Project Valhalla" |
| **Vector API** | **Still an incubator at JDK 27**: JEP 537 (twelfth), JEP 529 (eleventh, JDK 26), JEP 508 (tenth, JDK 25) | **Not yet as a stable API.** An incubating API is a re-compile obligation every six months. The sibling's §8.1 parsing argument is the relevant one, and the JVM's answer is not yet final |
| **Flight Recorder evolution** | **JEP 509** "JFR CPU-Time Profiling (Experimental)", **JEP 518** "JFR Cooperative Sampling" and **JEP 520** "JFR Method Timing & Tracing" (all JDK 25); **JEP 536** "JFR In-Process Data Redaction" (JDK 27) | **Yes, positively** — §9.6. Method timing and cooperative sampling are precisely the observability a latency team needs, and in-process redaction is what makes recording acceptable in a regulated environment |
| **32-bit x86 removal** | **JEP 503** (JDK 25) removed the 32-bit x86 port; the 32-bit Windows port went earlier with **JEP 479** (JDK 24) | **Not a latency change, an inventory change** — a long-lived platform with 32-bit components has a migration to do, and it is a portability risk rather than a performance one |

**How to read the table above.** Separate the four statuses — delivered-and-default (JEP 534), delivered-and-final-but-opt-in (JEP 454), preview (JEP 533), and Integrated-for-a-future-release (JEP 401) — because conflating them is how a design ends up depending on a preview API. Prefer platform mechanisms over library mechanisms where both exist, since the platform's are on the LTS upgrade path you already manage. Treat every incubator as a six-monthly maintenance obligation, the Vector API's twelve iterations being the standing evidence. And re-read the table at each LTS upgrade: its status column can be re-verified from the JEP records in an afternoon.

---

## 13. The Banking and Trading Angle

**The shared context — MiFID II RTS 6 timestamping, MAS guidance, the economics of microseconds, and the over-engineering argument — is the C/C++ sibling's §18, and this section does not re-derive it.** What follows is the JVM-specific mapping onto that context. No real bank is used as an example; where a worked illustration is needed, it is Cymbal Bank.

### 13.1 The paths, and where the JVM actually sits

| Path | Latency character | Is the JVM a good fit? |
|---|---|---|
| **Exchange gateway / order-entry session** | Hard latency-critical; the tightest budgets in the bank (sibling §18.1) | **Rarely the innermost layer.** The vendor SDKs, the kernel-bypass paths and the FPGA images are not JVM projects. A JVM sits *above* the gateway (session management, sequencing, order lifecycle) far more often than inside it — and that is the honest placement |
| **Market-data ingest and distribution** | Latency-critical for the input side, throughput-critical for the fan-out | **Yes, with discipline.** The feed handler's parse-and-publish path is where the JVM's allocation discipline (§5) has the highest payoff, and where a per-message allocation is most visible. The redistribution tier is genuinely throughput-shaped and is where the JVM's ecosystem strength is a better match than its latency properties |
| **Strategy / signal evaluation** | Latency-critical, but on a softer budget than the gateway — commonly sub-millisecond to low-millisecond | **Often yes.** The work is data-structure-bound and the shapes are controllable: primitive arrays, no boxing, pre-sized structures (§5.3). This is the path where a well-disciplined JVM implementation is genuinely competitive |
| **Pre-trade risk check** | Latency-relevant but correctness-dominated (sibling §18.4) | **Yes.** The JVM's containers, its testable concurrency primitives and its observability are a good match for a component whose invariant is "every order checked, every decision auditable" — and whose latency target is sub-millisecond rather than microsecond |
| **Order management and post-trade, wealth, custody, payments, back-office** | **Throughput-critical, not latency-critical**; sub-second targets | **Yes, unambiguously.** This is Java's home ground — durable, transactional, integration-heavy, staffed by a large talent pool, with a budget measured in milliseconds to seconds. The latency discipline here is a *cost* if applied, and the relevant sibling material is §18.7's softer-budget material; the JVM-specific additions are operational (§13.2) rather than architectural |
| **Fraud scoring, real-time limits** | Latency-relevant, millisecond budgets | **Yes.** This is where the JVM's ecosystem (in-memory stores, rule engines, stream libraries) is a genuine advantage and where the budget is comfortable enough that the runtime's costs are affordable |

The pattern in that table is worth naming: **the JVM moves up the stack as the budget tightens.** At the very tightest budgets the innermost layers belong to vendor SDKs and to the native stack the sibling owns; the JVM's territory is the tiers around them, and it is a large and load-bearing territory.

### 13.2 The operational consequences of the runtime choice

This is the part that is specific to a managed runtime, and it is what a design document must cover that a native-stack design document need not.

| Consequence | Mechanism | What the design must say |
|---|---|---|
| **Warmup on deploy** | §3.5: the first minutes after start are not steady state | How the instance warms (traffic replay, synthetic driver), how long it takes, how it is verified, and whether an AOT cache (§11.3) shortens it |
| **Rolling-restart behaviour** | Each restart re-enters warmup, and a rolling restart under peak load replaces warm instances with cold ones | The deployment runbook must sequence warmup before traffic admission, and the load balancer must honour a readiness signal that means *latency-ready*, not merely *listening* |
| **A collector that pauses under load** | §4.7: the pause is a function of your workload, and your workload is largest at the worst moment | Which collector, why, and — critically — the measured pause distribution at peak, not at average. A collector decision validated at 30% load is not validated |
| **Allocation-rate drift** | §5.1: a dependency upgrade or a feature change can move the allocation rate silently | A per-release allocation profile (§9.7 item 5), and a CI-level guard where the discipline permits |
| **AOT-cache regeneration, and the two lifecycles it creates** | An AOT cache is tied to the build, the JDK and the training run (§11.3), and the native-image toolchain is not synchronised with the JDK feature clock (§11.2) | Who regenerates the cache, at what point in the pipeline, and how its staleness is detected — an AOT cache trained on traffic that no longer resembles production is a warmup optimisation that quietly stopped working. And the platform plan must state which JDK line the AOT path is pinned to, and how that pin ages |
| **Flag provenance** | §10.6: a production command line accumulates stale flags | The command line in version control, with a justification per line, re-derived at each major JDK upgrade |

### 13.3 Which paths genuinely warrant this treatment

The sibling's §18.6 makes the over-engineering argument and this guide's answer is the same one, applied to a runtime that makes it easier to over-engineer by accident: **the JVM's default configuration is throughput-shaped, and applying latency discipline to a throughput path costs maintainability for nothing.**

A path warrants the treatment when all of the following hold:

**The business outcome depends on the tail**, not the mean — an execution path, a risk decision taken at the moment of decision, a market-data input; if a 10 ms outlier changes nothing, the discipline is not warranted. **The budget is stated as a percentile over a window** (the sibling's §2.4) and is one the platform can actually deliver after the warmup and collector work. And **the team can afford the specialist knowledge** — §14 makes that cost explicit, and it is permanent, because the platform moves every six months.

A path does **not** warrant it when the answer to "what does a tail event cost us?" is "nothing measurable" — which is the honest answer for most of a bank's estate, including most of its trading support estate. Applying §5 and §6 to a batch reconciliation job is not diligence; it is a maintenance liability with a latency claim attached.

### 13.4 The Cymbal Bank worked example, on paper

A Cymbal Bank design decision in this shape is worth writing down because it exercises the whole guide. The scenario: a **pre-trade risk gateway** that sits between a strategy process and the order router, must evaluate every order against limits before release, and must be auditable. The JVM is the chosen runtime because the rule evaluation is data-structure-bound, the team is Java-native, and the budget is sub-millisecond rather than microsecond.

What the design would have to say, and why:

| Design decision | The reasoning it must record | Guide reference |
|---|---|---|
| **Runtime and collector** | Long-running, steady-traffic service on an LTS JDK; a concurrent collector chosen because the SLO is a percentile, and because the allocation rate the risk evaluation creates is well understood | §4.3, §4.7 |
| **Budget and SLO** | Stated as percentiles over a window, with the warmup window excluded from the SLO's *measurement* but included in its *reporting* | §3.5, sibling §2.4 |
| **Allocation discipline** | Limits and positions in primitive arrays; no boxing; no per-order allocation in the evaluation itself; a per-thread scratch structure; a per-release allocation profile | §5.3, §5.4 |
| **Warmup, deployment and measurement** | Synthetic traffic driving the real evaluation paths *and the reject paths* before admission; a latency-readiness probe; a warmup-time metric per deploy; an end-to-end histogram per instance; GC logging on permanently; a rolling JFR window; a JMH benchmark for the evaluation core in CI | §3.6, §9.2–§9.6, §9.7, §13.2 |
| **Auditability** | The decision path must produce an auditable record without allocating on the hot path — the audit trail is written asynchronously, and the latency cost of that write is a line item in the budget | §5.1, sibling §9.4 |
| **What is deliberately not done** | No kernel bypass, no FPGA, no hand-written lock-free queue, no reactive layer: none of them is required by the budget, and each would add a maintenance obligation to a component whose value is correctness | sibling §18.6, §15 |

The last row is the most important one in the table. The discipline of this guide is not "apply every technique"; it is "trace every technique to the requirement that demands it, and skip the rest".

---

## 14. The Honest Assessment

This guide does not take a side beyond the evidence, and it does not present Java as either slow or fast. What it can do is state **where the managed runtime costs latency, where it repays it, and what must be configured differently from a native-code stack.** Those are three different questions and they have three different answers.

### 14.1 Where the managed runtime costs latency

| Cost | Mechanism | How it lands in practice |
|---|---|---|
| **Warmup** | §3: the code that runs in the first minutes is not the code that runs in steady state, and the transition is neither instant nor guaranteed to be monotone | An operational tax on every deploy, every restart, every failover, and every autoscale event. It is the cost with the largest organisational footprint, because it is paid by the people running the service, not by the people who wrote it |
| **Pause risk** | §4: a runtime component the application did not invoke decides to stop application threads or block an allocator | A tail risk that is bounded by design and not eliminable by configuration. The residual is real: allocation stalls, safepoint delays, and reference processing all persist in a concurrent collector |
| **Deoptimisation risk** | §3.2: a correct optimisation can be withdrawn, and the replacement code is slower | A latency cliff that appears without a deployment and is invisible in the application's own metrics. It is the cost that most surprises a native-stack engineer, because the mechanism has no analogue there |
| **Memory overhead** | Object headers, boxing, pointer-chased layouts, and a heap that must hold the working set plus promotion headroom | A footprint cost that is *reducing* (JEP 534's compact object headers are now the default in JDK 27, §12.2) but has not gone. It matters because it determines how much of a working set fits in cache, and the sibling's §3.1 makes cache residency the dominant term in the latency hierarchy |
| **Configuration surface** | §10: heap, pages, collector, code cache, diagnostics — each with a default that may or may not suit the workload | A permanent maintenance obligation, and — worse — an opportunity to make things worse by acting on a plausible-sounding flag |
| **Platform churn** | §12.1: a feature release every six months, an LTS every two years, incubators that must be recompiled, defaults that change between LTS lines | A governance cost, and the reason §10.6's flag-provenance discipline is not bureaucratic fussiness |
| **Two-adversary diagnosis** | A tail spike can be the collector, the compiler, the memory model, the OS, or the application — and the first three are invisible in the source | A triage cost. §9's tooling exists because the cause is not obvious from the code; a native stack's equivalent cost is lower because fewer of the causes are outside the team's code |

### 14.2 Where the managed runtime repays it

| Repayment | Mechanism | Why a native stack often lacks it |
|---|---|---|
| **A specification-level concurrency toolkit** | `java.util.concurrent` (JSR-166) gives queues, executors, atomics and synchronisers whose correctness arguments are documented, tested at platform scale, and usable by any engineer on the team | In a native stack, the same structures are either hand-written — with the audit burden the Rust companion's §2.3 describes — or sourced from third-party libraries whose maintenance status is on you |
| **A memory model that is written down** | The JMM is a specification with tooling consequences: static analysers, established patterns, and a documented answer to "what ordering do I need?" | The C++ model is equally specified and the sibling's §5.5 covers it; the Java advantage is the *library* built to it, which means the answer to a sharing problem is frequently a container rather than a protocol |
| **Runtime-driven optimisation for free** | Profile-guided optimisation without a build step, no PGO training data to curate, and — with JEP 515's AOT method profiling — the option to pre-seed it | A native stack typically needs a PGO pipeline to get the same profile-driven gains, and the sibling's §14.3 covers the cost of maintaining one |
| **Observability built into the platform** | JFR and JMC record the *runtime's own* decisions — GC events, JIT events, safepoints, allocation samples — in one timeline | Native tooling (`perf`, eBPF, VTune) is excellent and the sibling's §12 covers it, but "what did the runtime decide and when" is a question only a runtime that keeps its own event log can answer |
| **Development velocity and ecosystem** | A very large talent pool; mature libraries for persistence, messaging, integration, testing, metrics, and the operational tooling that goes with them | This is a productivity dividend that shows up in delivery time, onboarding and defect rates — not in nanoseconds. The Rust companion makes the identical point about Cargo, and the caution applies identically: **do not confuse the two** |
| **Collector work funded by the whole industry** | Multiple independent collector implementations, each with a design document and a public JEP history, and a default that improves every six months | A native stack gets an allocator, and the allocator's tuning is the application team's problem |
| **A bounded, specified pause** | §4.3: two production concurrent collectors, both generational, both with published goals | The honest advantage of manual memory management remains that the pause is *structurally absent*, and the Rust companion's §2.1 is the right place to read that argument. The Java counter-argument is that the absence is bought with the aliasing and lifetime discipline of §5 and §7, and that a team which cannot maintain that discipline will have a worse tail with the collector removed than with it present |

### 14.3 What a Java low-latency stack costs in complexity and specialist knowledge

This is the part that a vendor's benchmark does not price, and it is the reason §13.3 sets a bar before applying the discipline at all.

- **The knowledge is divided across roles and does not divide cleanly.** Someone must understand the memory model; someone must understand the collector; someone must be able to read a collector log and a JFR recording and to run JMH and JITWatch properly. In practice it is one or two people, and they are the ones called at 03:00.
- **The knowledge has a six-month shelf life.** §4.4 is four corrections in one document and will need re-verifying at the next LTS; there is no state in which the team "is done" learning the platform.
- **The measurement discipline is a standing cost, not a project** — §9.7's six items are cheap individually and a permanent commitment collectively, and the failure mode is that they lapse quietly after the incident that motivated them. The specialist knowledge also does not transfer out: it is scarce and non-portable, as the Rust companion says of systems expertise.
- **But the discipline can be acquired without a specialist.** That is the honest counterweight: allocation discipline (§5) and a measurement cadence (§9) are the two highest-leverage items in this document, and neither requires a collector expert. A team that adopts only those two, consistently, beats a team with a specialist and no cadence.

### 14.4 The conditions

**Java is the right runtime for a latency-sensitive system when all of the following hold:**

1. The budget is stated in **hundreds of microseconds to milliseconds of tail**, not single-digit microseconds — a budget the platform's concurrent collectors and a warmed JIT can plausibly meet (§4.7, §13.3).
2. The service is **long-running and steady-traffic**, so the JIT's profile-guided optimisation has time to pay for the warmup (§11.5).
3. The **hot path is data-structure-bound rather than vendor-SDK-bound** — your code operating on your state, not a call into a hardware vendor's C API (§13.1).
4. There is a **willingness to fund the discipline permanently** — the allocation hygiene of §5, the measurement cadence of §9.7 and the warmup work of §3.6 — rather than once. This includes staffing: the surrounding estate being Java-shaped and the team Java-native is part of the outcome, not a detail beside it.

**Java is not the right runtime when any of the following holds:** the budget is **single-digit microseconds end-to-end** on the innermost path, where the native stack and vendor SDKs the sibling owns are the only candidates and no JVM configuration changes that; the **process is short-lived** — one request per process, scale-to-zero — in which case warmup *is* the latency and the argument has become a deployment-shape argument about AOT (§11); the **hot path is a vendor API** whose contract is a C ABI or a kernel-bypass ring, where interop is possible (JEP 454) and adds a hop; the organisation **cannot fund the permanent discipline**, leaving the honest options of a native stack (which is not cheaper) or an SLO relaxed to what it will actually maintain; or the **organisation deploys feature releases without re-verifying**, because §12.1 turns that habit into a recurring surprise.

### 14.5 What must be configured differently from a native-code stack

A short checklist, and it is the practical summary of the whole document:

| Item | Native stack | Java |
|---|---|---|
| **Warmup** | Not a concept; the binary starts at its performance | A first-class operational procedure with a metric and a readiness gate (§3.6, §13.2) |
| **Memory sizing** | Allocator and OS tuning | Heap sizing, `-XX:+AlwaysPreTouch`, page settings, and a collector choice (§10.2–§10.3) |
| **Reclamation timing** | Yours, deterministic | The collector's; influenced only by allocation discipline and collector choice (§4, §5) |
| **Diagnostics** | Always available if you build it in | Requires explicit flags to be enabled, and some of them are gated (§10.4–§10.5) |
| **Compiled code** | Fixed at build time | Generated and re-generated at run time; sized through the code cache, inspected through the compilation log (§3, §9.3, §10.2) |
| **Load balancer semantics** | "Listening" is close to "ready" | "Ready" must mean *latency-ready*, which is a different and measurable signal (§13.2) |
| **Dependency risk** | Library versions | Library versions **and** the JDK line, on a six-month cadence (§12.1, §7.6) |
| **False sharing control** | Padding and alignment in the source | An internal annotation that is inert unless `-XX:-RestrictContended` is passed (§6.4) |

The output of all of this is a single observation. The managed runtime moves latency risk from the team's code into the team's operations. That is the single sentence that distinguishes this guide from the sibling. A native stack's latency is largely fixed when the binary is built; a JVM service's latency is partly decided every time it starts, on every deploy, under whatever load it meets. The discipline of this guide is the set of practices that makes those decisions predictable — and the honest assessment is that the practices are learnable, the platform supports them, and the cost of not doing them is paid in the tail.

---

## 15. The Anti-Patterns and the Claims Audit

### 15.1 The six anti-patterns

| Anti-pattern | Symptom | Cause | Guardrail |
|---|---|---|---|
| **The hand-rolled benchmark** | A design document cites a latency figure production never reproduces; a later optimisation produces no measurable production improvement | §3.4: a benchmark without warmup measures the compiler, and an unused result can be optimised away entirely | **JMH (§9.2) for anything carrying a latency claim** — warmup iterations, a consumed result, a reported distribution. A hand-rolled timer is admissible only for gross comparisons, never in a document people decide from |
| **The tuning flag copied from a blog** | A production command line of four or five `-XX:` flags whose provenance nobody can state; a JVM that behaves differently after a JDK upgrade although the code did not change | §10.6: a flag is a hypothesis about a workload, and the blog's workload was not yours. Worse, the flag may be obsolete — `-XX:+ZGenerational` and `sun.misc.Contended` are both in circulation and both wrong in JDK 27 (§4.4) | Every flag in version control with a one-line justification naming the measurement; re-derive the whole command line at each major JDK upgrade; delete any flag whose justification has expired. If the default is already correct, a redundant flag is a future hazard, not a precaution |
| **The collector swapped without measuring the allocation rate** | A collector change that improves a benchmark and not production; or improves it for a month and then regresses as a feature lands | §5.1: the pause is a function of allocation volume and object lifetime, and the allocation rate is a property of the application. Swapping the collector changes how the existing work is *charged*, not how much work there is | Measure the allocation profile before and after any collector change (§5.2), and publish the allocation rate as part of the service's specification. Order of operations: reduce allocation, then choose the collector, then tune it — never the reverse |
| **The warmup omitted, so the first requests after deploy carry the cost** | A daily tail-latency incident correlating with the deployment schedule; a p99.9 that is fine at 10:00 and terrible at 06:00; a rolling restart under peak load that is a visible latency event | §3.5 and §13.2: the instance is admitted as soon as it listens, so its warmup is served to real traffic. Interpretation, profiling compilation, an untouched heap and an unsized code cache are each on the critical path of a real request | Warm before admission, on a readiness signal that means *latency-ready*; warm the error and recovery paths too; measure and publish time-to-steady-state per deploy; use the Leyden AOT cache (JEP 483/514/515/516) to shorten the window |
| **The pooled object that leaks state** | An intermittent, load-dependent correctness bug in a system recently optimised for allocation; a field correct on an object's first use and wrong on reuse; a defect rate that rises after a "pure performance" change | §5.5: a pooled object is reset by a release path, and a reset path with one forgotten field produces a stale value on reuse. The symptom is intermittent because it depends on which instance the pool hands out | Pool only where the object is genuinely expensive to create and the pool is per-thread; write the reset as one explicit method assigning every field, paired with a test that fails when a field is added without being reset. Prefer §5.3's shape changes, which cannot fail this way |
| **The annotation applied without its gating flag** | A false-sharing fix in the source that produces no measurable change in production; a team convinced the problem is solved because the code now looks different | §6.4: `jdk.internal.vm.annotation.Contended` is inert for application classes unless the JVM is started with `-XX:-RestrictContended`, because the flag restricting the annotation to trusted classes defaults to **true** | Verify the flag is in the *running* command line, and verify the effect with the §6.5 measurement. Note the two adjacent traps: `sun.misc.Contended` could not be located in the OpenJDK tree at any probed tag, and the three `@Contended` flags are **not** in the launcher man page — so the person who reads the man page will not find them |

### 15.2 The claims audit

Every version gate, JEP number, flag and tool status this guide asserts, with its source and date. **J** = the JEP's own record on openjdk.org; **S** = the OpenJDK source tree at tag `jdk-25-ga`/`jdk-27-ga` as noted; **M** = the JDK 27 `java` launcher man page; **API** = the project's own GitHub API record; **V** = the vendor's or project's own documentation. All read **2026-09-21** unless stated.

| Claim | Value asserted | Source | Date |
|---|---|---|---|
| Release clock | JDK 27 GA 15 Sep 2026 (current); JDK 26 GA 17 Mar 2026; JDK 25 GA 16 Sep 2025 and an **LTS**; JDK 21 GA 19 Sep 2023 and the previous LTS; cadence 6 months, LTS every two years. JDK 28 in flight: JEP 401 **Integrated**, JEP 539 dependency, JEP 535 **Targeted**, JEP 544 **Proposed to Target** | J (`projects/jdk/25/`, `/27/`) | 2026-09-21 |
| G1 default | Default since JDK 9 (JEP 248); default in *all* environments per JEP 523, JDK 27, Delivered | J | 2026-09-21 |
| ZGC gate | Experimental JDK 11 (JEP 333, Release 11); product **JDK 15** (JEP 377, Release 15); not the default | J | 2026-09-21 |
| Shenandoah gate | Experimental JDK 12 (JEP 189, Release 12); product **JDK 15** (JEP 379, Release 15); not the default. JEP 189 itself states it is supported by Red Hat for aarch64 and amd64 | J | 2026-09-21 |
| CMS removal and Epsilon | CMS deprecated by JEP 291, removed by **JEP 363, Closed/Delivered, Release 14**. Epsilon **JEP 318, Closed/Delivered, Release 11**, its flag `product(bool, …, false, EXPERIMENTAL)` → needs `-XX:+UnlockExperimentalVMOptions` | J + S (`gc/shared/gc_globals.hpp`) | 2026-09-21 |
| ZGC generational line | JEP 439 (JDK 21), JEP 474 (JDK 23, generational by default), JEP 490 (JDK 24, non-generational mode removed) | J | 2026-09-21 |
| Shenandoah generational line | JEP 404 (JDK 24, experimental), JEP 521 (JDK 25, product), JEP 535 (JDK 28, **Targeted** — not delivered) | J | 2026-09-21 |
| Obsolete/default flag states | `-XX:+ZGenerational` is absent from the JDK 27 ZGC globals (deprecated by JEP 474, obsoleted by JEP 490); `-XX:ShenandoahGCMode` is a `ccstr` whose default is the literal string `"satb"` | S (`gc/z/z_globals.hpp`, `gc/shenandoah/shenandoah_globals.hpp`) + J | 2026-09-21 |
| Pause claims (attributed) | *"Pause times should not exceed 1 millisecond"* — **JEP 439's goal**; *"never pausing application threads for more than a millisecond"* — **JEP 516's text**; JEP 439's Cassandra example — a quarter of the heap, four times the throughput vs non-generational ZGC, pauses under 1 ms — **from the JEP** | J | 2026-09-21 |
| Shenandoah's stated non-goal | *"The goal is not to fix all JVM pause issues. Pause times due to reasons other than GC like Time To Safe Point (TTSP) issues or monitor inflation are outside the scope of this JEP."* | J (JEP 189) | 2026-09-21 |
| `@Contended` identity | `jdk.internal.vm.annotation.Contended`, used by java.base (e.g. `ConcurrentHashMap.java:2592`). **`sun.misc.Contended` could not be located** at `jdk-11-ga`, `jdk-17-ga`, `jdk-21-ga`, `jdk-25-ga`, `jdk-27-ga`; control probes succeeded | S | 2026-09-21 |
| The `@Contended` flags | `-XX:ContendedPaddingWidth` `product(int, …, 128)`, range 0–8192; `-XX:+EnableContended` `product(bool, …, true)`; `-XX:-RestrictContended` `product(bool, …, true)` — application classes need it **negated**. No unlock required for any of the three; none is in the man page | S (`runtime/globals.hpp`, lines ~823-829) + M (absence) | 2026-09-21 |
| GC flags | Selection: UseSerialGC / UseG1GC / UseParallelGC / UseZGC / UseShenandoahGC, all `product(bool, …, false)`. Others in the same file: `-XX:+AlwaysPreTouch` `product(bool,…,false)`; `-XX:+AlwaysPreTouchStacks` `product(bool,…,false, DIAGNOSTIC)` → needs `-XX:+UnlockDiagnosticVMOptions`; `-XX:MaxGCPauseMillis` `product(uintx, …, max_uintx - 1)`; `-XX:GCTimeRatio` `product(uint, …, 99)`; `-XX:+UseCondCardMark` `product(bool,…,false)` | S (`gc/shared/gc_globals.hpp`) | 2026-09-21 |
| ZGC and Shenandoah options | ZGC: `ZAllocationSpikeTolerance` 2.0; `ZCollectionInterval` an in-source alias for `ZCollectionIntervalMajor`; `ZUncommit` true; `ZUncommitDelay` 5*60; `ZCollectionIntervalMinor`/`Major` −1; `ZCollectionIntervalOnly` false; `ZStatisticsInterval` 10 (**DIAGNOSTIC**). Shenandoah: `-XX:ShenandoahGCHeuristics` `ccstr`, default `"adaptive"`; `-XX:ShenandoahRegionSize` `size_t`, default 0, **EXPERIMENTAL** → needs `-XX:+UnlockExperimentalVMOptions`. None of these is in the man page | S (`gc/z/z_globals.hpp`, `gc/shenandoah/shenandoah_globals.hpp`) + M (absence) | 2026-09-21 |
| Memory/page flags | `-XX:+UseTransparentHugePages` `product(bool,…,false)`, Linux-only globals; `-XX:+UseNUMA` and `-XX:+UseNUMAInterleaving` `product(bool,…,false)`; `-XX:+UseLargePages` and `-XX:InitialCodeCacheSize` `product_pd` (platform-dependent); `-XX:+UseCompactObjectHeaders` `product(bool, …, true)` — **on by default in JDK 27** | S (`os/linux/globals_linux.hpp:36`, `runtime/globals.hpp`) | 2026-09-21 |
| `-XX:+DebugNonSafepoints` | `product(bool, …, trueInDebug, DIAGNOSTIC)` — off in product builds, on in debug builds, and it needs `-XX:+UnlockDiagnosticVMOptions` in a product build; not in the man page | S (`runtime/globals.hpp:979`) + M (absence) | 2026-09-21 |
| Man-page presence | `-Xlog:gc` (22×), `-XX:StartFlightRecording`, `-XX:+UnlockDiagnosticVMOptions` (25×), `-XX:+AlwaysPreTouch`, `-XX:+UseTransparentHugePages`, `-XX:MaxGCPauseMillis`, `-XX:ShenandoahGCMode`, `-XX:+UseCompactObjectHeaders`, `-XX:+UseZGC`, `-XX:+UseShenandoahGC`, `-XX:InitialCodeCacheSize`, `-XX:+UseNUMA`, `-XX:+UseSerialGC`, `-XX:+UseParallelGC`, `-XX:+UseG1GC`, `-XX:LargePageSizeInBytes`, `-XX:+UseCondCardMark` | M (JDK 27) | 2026-09-21 |
| Measurement tooling | JMH tag **1.37**, last push **2026-07-14**, active, no GitHub "releases" object; async-profiler **v4.5 (2026-07-20)**, push **2026-09-21**, very active, JDK 11+, "does not suffer from the Safepoint bias problem", profiles CPU/allocations/native memory/contended locks/hardware counters; JITWatch **1.5.0 (2025-11-11)**, prior 1.4.9 (2024-01-06) and 1.4.7 (2022-01-23), push **2026-09-18**, active, repo moved by 301 redirect from `AdoptOpenJDK/jitwatch` to `chriswhocodes/jitwatch`, JavaFX-based; JMC tag **9.1.2-ga**, push **2026-09-13**, active | API + V (project READMEs) | 2026-09-21 |
| Libraries | LMAX Disruptor **4.0.0 (2023-09-29)**, push **2025-04-02**, **stable but low activity**, not archived; Aeron **1.53.2 (2026-09-18)**, push 2026-09-21, very active; Agrona **2.6.1 (2026-09-17)**, push 2026-09-21, active; JCTools **v4.0.7 (2026-08-18)**, active; Chronicle Queue **chronicle-queue-5.26.17 (2026-07-27)**, push 2026-09-20, active | API | 2026-09-21 |
| GraalVM | Newest release notes **25.3.4.1**, then 25.2.4, 25.1.3, 25.0; the "GraalVM 25.x for JDK 25" line; **the release-notes index lists no JDK 26 or 27 line**; repository active. Native Image's own claims: *"Starts in milliseconds"*, *"Delivers peak performance immediately, with no warmup"* | API + V | 2026-09-21 |
| Virtual threads | JEP 444, Closed/Delivered, Release 21; previews JEP 425 (19), JEP 436 (20); goals framed as high-throughput. **JEP 491**, Closed/Delivered, Release 24; its Risks section notes monitor exit may queue a virtual thread, "currently not as efficient" as unparking a platform thread | J | 2026-09-21 |
| Structured concurrency and scoped values | Structured concurrency is **still preview at JDK 27**: JEP 453 (21), 499 (24), 505 (25), 525 (26), **533 (27, seventh preview)**. Scoped values have a *different* status: JEP 446 preview (JDK 21) → **JEP 506 final in JDK 25**, listed without a preview suffix | J | 2026-09-21 |
| Foreign Function & Memory | **JEP 454, Closed/Delivered, Release 22**; previews JEP 442 (21), 434 (20), 424 (19) | J | 2026-09-21 |
| JNI and Unsafe restriction | JEP 472 "Prepare to Restrict the Use of JNI" (JDK 24); JEP 498 "Warn upon Use of Memory-Access Methods in `sun.misc.Unsafe`" (JDK 24) | J | 2026-09-21 |
| Value objects | JEP 401 "Value Objects (Preview)", Status **Integrated**, Release **28** — **NOT delivered**; JEP 539 is its dependency | J | 2026-09-21 |
| Compact object headers | JEP 450 experimental (24) → JEP 519 product (25) → **JEP 534 by default (27)**. JEP 519's evidence: SPECjbb2015 "22% less heap space and 8% less CPU time", GC count reduced "by 15%"; Amazon tested "hundreds of services in production"; four header bits "already reserved for Project Valhalla" | J (JEP 519) + S | 2026-09-21 |
| Vector API and JFR | Vector API: JEP 537 twelfth incubator (JDK 27), JEP 529 (26), JEP 508 (25). Flight Recorder: JEP 509 (Experimental), 518 and 520 (JDK 25); JEP 536 (JDK 27) | J | 2026-09-21 |
| Leyden / AOT | JEP 483 (JDK 24, Delivered); JEP 514 and 515 (JDK 25); JEP 516 (JDK 26, Delivered); JEP 544 (Proposed to Target, Release 28) | J | 2026-09-21 |
| 32-bit port removals | JEP 503 (JDK 25) x86; JEP 479 (JDK 24) Windows | J | 2026-09-21 |
| Alternative JVMs | Eclipse OpenJ9 **v0.62.0 released September 2026, supporting OpenJDK 27**; Azul Zing/Platform Prime — C4 pauseless GC, Falcon JIT, ReadyNow, Cloud Native Compiler, with the vendor's own "low, consistent response latency" and "faster warm-up" claims; IBM Semeru Runtimes described as free enterprise-ready OpenJDK-based binaries, but **no dated release for a specific JDK was established** | V | 2026-09-21 |

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 The search-backend limitation, recorded honestly

**`web_search` returned empty result sets for every query issued in this research run, across several unrelated topics, while `web_extract` on primary URLs worked normally and returned complete pages.** This is a **tool limitation of the search backend used for this run**, and **not** evidence of absence of any tool, library, flag, JEP or fact; no conclusion here rests on a search returning nothing. Every positive claim was obtained by direct extraction of a primary source — JEP records on openjdk.org, the OpenJDK source tree, the JDK launcher's own man page, project documentation and READMEs — or by direct query of a project's repository API, each date-stamped 21 September 2026.

### 16.2 Unverified items

Each of the following is stated in this guide as unverified, and each is a candidate for a follow-up verification rather than a silent assumption.

| Item | Status | Where it appears |
|---|---|---|
| **A dated IBM Semeru release for a specific JDK, and whether Oracle's own JDK binaries ship a functional Shenandoah** | Neither was established. For Shenandoah what *is* verified: JEP 189 states it "has been implemented and will be supported by Red Hat for aarch64 and for amd64"; JEP 379 (JDK 15) made it a product feature in mainline; and `-XX:+UseShenandoahGC` is in the JDK 27 launcher man page. Whether a given vendor's *binary* enables it is a build-vendor question this run did not resolve. Do not cite a Semeru version from this guide | §4.3, §11.4 |
| **Specific latency figures for any alternative JVM** | Not verified. Only the vendors' qualitative claims (C4 pauseless, Falcon, ReadyNow) were read from vendor documentation. No number in this guide comes from a vendor benchmark | §11.4 |
| **GraalVM's behaviour on JDK 26 or 27** | The release-notes index lists no such line as of 21 September 2026; whether an unreleased or non-indexed build supports a later JDK was not established. This is a gap in the vendor's public material, not a statement that no such build exists | §11.2 |
| **Escape-analysis specifics and the tier-level flag literals for controlling compilation** | §5.4 describes the standard mechanism (scalar replacement, lock elimination, the shapes in which escape analysis fails) as documented compiler behaviour, but it was **not** re-derived from the JDK source or a JLS/JVM-spec citation in this run, and it is presented as mechanism description rather than as a specification guarantee — which is the point of the subsection. The *path* in §3.1 is described and §9.3 covers reading the compilation log, but individual flag literals for pinning a tier or forcing recompilation were **not** verified against the JDK 27 source and are therefore not named in this guide | §3.1, §5.4, §10 |
| **A dated release for any web framework or reactive stack** | Not established, and not used. This guide names no framework as a latency recommendation; the RxJava guide owns the reactive angle, and §7's libraries are the ones whose status was verified | §7, §8.3 |

### 16.3 Glossary

| Term | Definition |
|---|---|
| **AOT (ahead-of-time compilation)** | Compiling and/or loading classes before the process runs, so part of the warmup cost is paid at build time. Project Leyden is the JDK's line of work; GraalVM Native Image is a separate, closed-world approach |
| **Allocation rate / allocation stall** | The rate at which bytes are allocated and the lifetime distribution of those objects — the application's principal lever over the collector. An *allocation stall* is a thread blocked because it cannot allocate until the collector frees or commits memory; it is not a stop-the-world pause and is frequently misdiagnosed as a slow method |
| **Code cache** | The region the JIT writes compiled code into. When it fills, compilation stops and compiled methods may be discarded — a latency cliff, not a slope |
| **Compact object headers** | An object header representation removing the separate class-pointer field: experimental in JDK 24, product in JDK 25, default in JDK 27 (JEP 450 → 519 → 534) |
| **Concurrent collector** | A collector doing most of its work without stopping application threads, leaving only short stop-the-world (STW) phases — root scanning is the irreducible part in ZGC and Shenandoah |
| **Deoptimisation** | Discarding compiled machine code because an assumption the compiler made no longer holds, and continuing in a lower tier until recompilation |
| **Escape analysis** | The compiler's determination that an allocated object cannot be observed outside its creating method, enabling scalar replacement and lock elimination — an optimisation, never a guarantee |
| **False sharing** | Two cores writing different variables that share a cache line, causing coherence traffic. Sibling §3.2 for the mechanism; the JVM's remedy is `@jdk.internal.vm.annotation.Contended` plus `-XX:-RestrictContended` |
| **G1** | The general-purpose, region-based, generational garbage-first collector. Default since JDK 9, and in all environments since JEP 523 (JDK 27) |
| **Happens-before** | The JMM's partial order, which must be *established* by a synchronisation action; without an edge, two accesses to a field are a data race, and in the presence of a data race the model permits results most engineers would call impossible |
| **Java Memory Model (JMM)** | The specification of permitted observations and orderings across threads — weaker than sequential consistency, and the fix is an explicit primitive (`volatile`, an atomic, a `VarHandle` mode) |
| **JIT (just-in-time compiler) / tiered compilation** | The runtime component generating machine code from bytecode at run time, driven by profiles it has collected; tiered compilation is the staged path from interpreted execution through a profiling compiler to the optimising compiler |
| **JFR / JMC** | The Java Flight Recorder (built-in low-overhead event recording) and JDK Mission Control (its analyser). The platform's ordered, runtime-wide timeline |
| **Off-heap / direct buffer** | Memory outside the garbage-collected heap — direct buffers, memory-mapped files, or the Foreign Function & Memory API. A direct buffer's storage is off-heap, but its *deallocation* is still the collector's business |
| **Pause** | Any interval in which application threads cannot progress because the runtime has stopped them. The taxonomy in §4.1 distinguishes stop-the-world phases, allocation stalls, concurrent CPU competition and non-GC safepoint delays |
| **Profile pollution** | A hot call site permanently mis-optimised because an early, unrepresentative caller contributed to its profile. §3.3 |
| **Safepoint** | A point at which all application threads are in a state where the runtime may act on them. Time-to-safepoint delays are *not* collector pauses and are not fixed by collector choice (JEP 189's own non-goal) |
| **Shenandoah / ZGC** | The two concurrent collectors. **Shenandoah**: region-based and compacting, with pause times independent of heap size — experimental in JDK 12, product in JDK 15, generational in product since JDK 25, generational by default targeted for JDK 28 (JEP 535). **ZGC**: region-based, compacting and NUMA-aware, production since JDK 15 and generational-only since JDK 24 (JEP 490), with a sub-millisecond pause goal stated in its own JEPs |
| **Stop-the-world (STW)** | A pause during which application threads are halted. Root scanning is the irreducible part in both concurrent collectors |
| **Tiered compilation** | The JVM's staged path from interpreted execution through a profiling compiler to the optimising compiler, driven by profile data |

### 16.4 Cross-references

**The discipline owners (read first):**

- [`low_latency_cpp_development_guide.md`](low_latency_cpp_development_guide.md) — §1 (what low-latency development is; the latency hierarchy; jitter), §2 (the latency budget approach and percentile SLOs), §3 (hardware: caches, false sharing, NUMA, huge pages, data-oriented design), §5 (concurrency and the C++ memory model), §6 (the event loop), §7 (networking, kernel bypass), §8 (market-data formats), §9 (the trading reference architecture), §10 (shared memory), §11 (FPGA), §12 (profiling practice), §13 (latency measurement — the source of §9.7's discipline here), §14 (build and tooling), §16 (pitfalls), §18 (low latency in banking — the source of §13's framing here).
- [`low_latency_rust_programming_guide.md`](low_latency_rust_programming_guide.md) — §1.1's ownership table is the model for §1.1 above; §2.1 (no garbage collector, and the correct way to state that advantage); §3.5 (panic machinery, the analogue of §5.4 here); §9 (the honest comparison); §11 (the regulated-institution angle); §14 (its honest-verification section, the model for §16 here).

**Adjacent companions (cross-referenced, not duplicated here):**

- [`low_latency_genai_patterns_guide.md`](low_latency_genai_patterns_guide.md) — the model-serving latency budget (TTFT, TPOT). Relevant to a JVM service only where the JVM is the orchestrator rather than the runtime under measurement.
- [`rxjava_guide.md`](rxjava_guide.md) — the reactive angle: operators, schedulers, backpressure. §8.3 here is the only place this guide touches it, and only to make the point that a scheduler is a thread allocator.
- [`osgi_guide.md`](osgi_guide.md) — the Java module platform, a different problem from the JVM's own module and AOT work (Project Leyden, §11.3), which is not OSGi's subject. [`gpu_optimization_guide.md`](gpu_optimization_guide.md) and [`nvidia_dcgm_guide.md`](nvidia_dcgm_guide.md) — accelerator work and its telemetry: a GPU is a throughput device, and the sibling's §11.6 framing applies.
- [`../banking/operational_resilience_framework_guide.md`](../banking/operational_resilience_framework_guide.md) — for the operational-resilience and third-party-risk framing behind §13.2's deployment consequences and §7.6's dependency governance.

**Primary sources cited in this guide:** `openjdk.org/jeps/<n>` (every JEP number above), `openjdk.org/projects/jdk/<n>/` (release pages and the release clock), the OpenJDK source tree at tags `jdk-25-ga` and `jdk-27-ga` (flag declarations, gating, defaults), the JDK 27 `java` launcher man page (documented flags), the GitHub REST API for each project's release and push dates, and the projects' own READMEs and documentation sites (JITWatch, async-profiler, GraalVM, Eclipse OpenJ9, Azul, IBM).

### 16.5 Closing summary

The JVM-specific delta in low-latency work is not a syntax question and not a speed question. It is a **scheduling** question: three components decide things on your behalf — the compiler decides what your code looks like, the collector decides when your threads stop, and the memory model decides what your threads may observe — and the discipline is the set of practices that make those decisions predictable enough to put a percentile SLO on.

**What the runtime costs you.** Warmup on every deploy and every restart (§3). A pause risk that is bounded by design but not eliminated by configuration (§4). A deoptimisation cliff that arrives without a deployment (§3.2). A footprint overhead that is shrinking but not gone (§12.2). A configuration surface whose defaults may not be yours, on a platform that ships a feature release every six months (§10, §12.1).

**What the runtime repays.** A concurrency toolkit whose correctness argument is written down (§7.1) and a memory model that is specified with a library built to it (§6). Profile-guided optimisation without a build pipeline, and an AOT path that shortens the warmup it otherwise charges (§11.3). Observability of the runtime's own decisions — JFR, JMC, collector logs — which no native stack records because no native stack has those decisions (§9). And the velocity and ecosystem that show up in delivery time rather than in nanoseconds (§14.2).

**What must change from a native-code stack.** The allocation rate is a first-class design number, because it is the lever that actually governs the pause (§5). The end-to-end histogram is non-negotiable, and so is a warmup metric beside it (§9.7). Every flag in the production command line must be traceable to a measurement, and the collection must be re-derived at each major JDK version (§10.6). And a claimed pause is not your pause — a collector's advertised behaviour is a hypothesis about a workload, and the only admissible evidence is your own distribution under your own load (§4.7).

The practitioners who get this right do not treat the runtime as an obstacle and do not trust it as a service. They measure the allocation rate before they touch the collector, they warm the process before it takes traffic, they keep the diagnostics on permanently, and they date every claim they make about the platform. That is not a smaller discipline than the C++ guide's — it is the same discipline with two more components to bring into line: the JVM on your terms.
